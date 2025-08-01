Python 코드(ScentSmart.py 등)와 C언어 펌웨어 파일들이 **서로 시리얼 통신을 통해 데이터를 주고받는 부분**은 양쪽 모두 명확히 구현되어 있고, 각 코드에서 다음과 같이 연결됩니다.

## 1. Python 쪽: 시리얼 데이터 송수신 위치

## ① **송신(데이터 보내기, PC→MCU)**

- **핵심 함수**:
    
    python
    
    `def write_data(self, wdata):     if dsSerial._is_open(self._serial):        self._serial.write(wdata)   # <--- 실제 데이터 전송!`
    
    - 여기서 `wdata`는 바이트로 만든 명령 패킷(향 번호, 시간, 명령 등 포함)
        
    - `dsSerial._is_open(self._serial)`은 포트가 열려있는지 체크
        
    - **핵심:** `.write(wdata)` 부분에서 실제 하드웨어로 데이터가 전송됨!
        
- 이 함수는 ScentSmart.py의 UiDlg 클래스에 있으며,
    
- 명령(향 분사, 세정, 정지, 상태 요청 등)이 발생하면 내부적으로 패킷을 바이너리로 만든 뒤 이 함수를 호출합니다.
    

## ② **수신(데이터 받기, MCU→PC)**

- **핵심 함수와 흐름**:
    
    - 별도의 쓰레드(`dsSerial.SerialReadThread`)가 시리얼 포트에서 데이터를 계속 읽음
        
    - 데이터(RX)가 오면 Signal을 통해
        
        python
        
        `def readSerialData(self, rdata):     ...  # rdata = 바이트 데이터(패킷)    self.parseReadData(rdata)`
        
    - 여기서 `rdata`는 펌웨어가 보낸 응답 패킷(상태, 결과 등)
        
- 실제 시리얼 포트 객체는 PySerial의 Serial 클래스를 씀
    

## 2. C/펌웨어 쪽: 시리얼 데이터 송수신 위치

## ① **수신(데이터 받기, PC→MCU)**

- UART(시리얼) 수신 인터럽트에서 한 바이트씩 읽어서 버퍼에 저장:
    
    cpp
    
    \`ISR(USART0_RX_vect) {      // (HostRun.cpp)     SerialHost._rx_complete_irq(c);    SerialHostEvent();     // 바이트 모으면 패킷화 }`\
    
- (`SerialHostEvent`에서)
    
    - STX~ETX까지 패킷 완성되면 플래그 Set → 이후 명령 파싱(RunSerialHostDataCheck), 분기(RunSerialHostCommand)
        

## ② **송신(데이터 보내기, MCU→PC)**

- 명령 처리 후 필요할 때 상태/응답 데이터를 패킷으로 만들어 전송:
    
    cpp
    
    `SerialHostSendData(SerialHostTxBuff, SerialHostTxCount); // (HostRun.cpp)`
    
    - 수신한 명령에 따라 확인, 상태, 결과, 버전 등 다양한 응답 패킷을 전송함
        

## 3. 실제 데이터 교환 "핵심 연결 지점"

- **파이썬 → MCU**
    
    > ScentSmart.py의 `write_data()` 함수 내부의  
    > `self._serial.write(wdata)`  
    > **→ (RS232/485 통신선을 거쳐...) →**  
    > MCU 펌웨어의 UART RX 인터럽트(ISR) 내부로 도착
    
- **MCU → 파이썬**
    
    > HostRun.cpp 등에서  
    > `SerialHostSendData()`(응답패킷)  
    > **→ (시리얼로 송신) →**  
    > PC에서 PySerial 수신 쓰레드가 읽어  
    > `readSerialData(rdata)` 핸들러에서 처리
    

## 4. 요약 (한 문장)

- **파이썬 ScentSmart.py의 `write_data()` 함수에서 `.write(wdata)`로 전송한 데이터가 펌웨어 HostRun.cpp의 시리얼 인터럽트(USART0_RX_vect)에서 수신 및 명령 분기/실행되고, 펌웨어가 명령 실행 후 `SerialHostSendData()`로 만든 응답 패킷을 시리얼로 보내면 PC 소프트웨어에서 다시 수신하여 로그 처리/UI 반영을 하게 됩니다.**
    

즉,

> **"시리얼 입출력이 실제 연결되는 위치는 파이썬의 `.write(wdata)`와 펌웨어의 인터럽트/패킷파싱 함수, 그리고 역방향 송수신 함수쌍입니다."**

양쪽 파일에서 위 함수 부분을 찾으면 실제로 데이터가 주고받아지는 "물리적 통로"와, 데이터 처리 시나리오 전체를 추적할 수 있습니다.

1. [https://ppl-ai-file-upload.s3.amazonaws.com/web/direct-files/attachments/72041696/d454c19b-06fb-4a6d-9714-a94ec292d75d/ScentSmart.py](https://ppl-ai-file-upload.s3.amazonaws.com/web/direct-files/attachments/72041696/d454c19b-06fb-4a6d-9714-a94ec292d75d/ScentSmart.py)
2. [https://community.st.com/t5/stm32-mcus-products/stm32-to-python/td-p/119889](https://community.st.com/t5/stm32-mcus-products/stm32-to-python/td-p/119889)
3. [https://forums.raspberrypi.com/viewtopic.php?t=335210](https://forums.raspberrypi.com/viewtopic.php?t=335210)
4. [https://stackoverflow.com/questions/6316584/access-usb-serial-ports-using-python-and-pyserial](https://stackoverflow.com/questions/6316584/access-usb-serial-ports-using-python-and-pyserial)
5. [https://electronics.stackexchange.com/questions/168766/sending-info-from-python-to-avr-c](https://electronics.stackexchange.com/questions/168766/sending-info-from-python-to-avr-c)
6. [https://www.instructables.com/Python-Serial-Port-Communication-Between-PC-and-Ar/](https://www.instructables.com/Python-Serial-Port-Communication-Between-PC-and-Ar/)
7. [https://forum.arduino.cc/t/arduino-pyserial-only-reading-data-on-first-python-execution/1268053](https://forum.arduino.cc/t/arduino-pyserial-only-reading-data-on-first-python-execution/1268053)
8. [https://forum.arduino.cc/t/esp32-help-with-software-serial/1127108](https://forum.arduino.cc/t/esp32-help-with-software-serial/1127108)
9. [https://forum.arduino.cc/t/serial-communication-with-python-beginner-qestion/944780](https://forum.arduino.cc/t/serial-communication-with-python-beginner-qestion/944780)
10. [https://randomnerdtutorials.com/esp32-uart-communication-serial-arduino/](https://randomnerdtutorials.com/esp32-uart-communication-serial-arduino/)
11. [https://stackoverflow.com/questions/12463605/two-port-receive-using-software-serial-on-arduino](https://stackoverflow.com/questions/12463605/two-port-receive-using-software-serial-on-arduino)
12. [https://github.com/jcomas/CM1106_UART](https://github.com/jcomas/CM1106_UART)
13. [https://forums.raspberrypi.com/viewtopic.php?t=379066](https://forums.raspberrypi.com/viewtopic.php?t=379066)
14. [https://www.instructables.com/Serial-Port-Programming-With-NET/](https://www.instructables.com/Serial-Port-Programming-With-NET/)
15. [https://stackoverflow.com/questions/19231465/how-to-make-a-serial-port-sniffer-sniffing-physical-port-using-a-python](https://stackoverflow.com/questions/19231465/how-to-make-a-serial-port-sniffer-sniffing-physical-port-using-a-python)
16. [https://stackoverflow.com/questions/62668941/how-to-read-serial-with-interrupt-serial](https://stackoverflow.com/questions/62668941/how-to-read-serial-with-interrupt-serial)
17. [https://forums.ni.com/t5/LabVIEW/Communication-with-serial-device/td-p/3791600](https://forums.ni.com/t5/LabVIEW/Communication-with-serial-device/td-p/3791600)
18. [https://www.educative.io/answers/how-to-establish-a-serial-communication-channel-in-python](https://www.educative.io/answers/how-to-establish-a-serial-communication-channel-in-python)
19. [https://github.com/pyserial/pyserial/issues/163](https://github.com/pyserial/pyserial/issues/163)
20. [https://marlinfw.org/docs/setting/serial.html](https://marlinfw.org/docs/setting/serial.html)
21. [https://www.engineersgarage.com/articles-raspberry-pi-serial-communication-uart-protocol-serial-linux-devices/](https://www.engineersgarage.com/articles-raspberry-pi-serial-communication-uart-protocol-serial-linux-devices/)
22. [https://stackoverflow.com/questions/67388438/im-trying-to-read-sensor-data-through-serial-communication-but-when-i-try-to-re](https://stackoverflow.com/questions/67388438/im-trying-to-read-sensor-data-through-serial-communication-but-when-i-try-to-re)
23. [https://www.geeksforgeeks.org/cpp/serial-port-connection-in-cpp/](https://www.geeksforgeeks.org/cpp/serial-port-connection-in-cpp/)
24. [https://device-programming-course.readthedocs.io/en/latest/serial-communication.html](https://device-programming-course.readthedocs.io/en/latest/serial-communication.html)
25. [https://forum.repetier.com/discussion/9532/serial-communication](https://forum.repetier.com/discussion/9532/serial-communication)
26. [https://forum.arduino.cc/t/serial-communication-problem-python-c/354466](https://forum.arduino.cc/t/serial-communication-problem-python-c/354466)
27. [https://stackoverflow.com/questions/24074914/python-to-arduino-serial-read-write](https://stackoverflow.com/questions/24074914/python-to-arduino-serial-read-write)
28. [https://forums.balena.io/t/raspberry-pi-4-uart-link-fails-after-approx-5s/306933](https://forums.balena.io/t/raspberry-pi-4-uart-link-fails-after-approx-5s/306933)
29. [https://robotics.stackexchange.com/questions/39565/python-or-c-for-serial-communication](https://robotics.stackexchange.com/questions/39565/python-or-c-for-serial-communication)
30. [https://arduino.stackexchange.com/questions/38052/serial-comm-timing-issue-between-arduino-and-pyserial](https://arduino.stackexchange.com/questions/38052/serial-comm-timing-issue-between-arduino-and-pyserial)
31. [https://docs.qualcomm.com/bundle/publicresource/topics/80-70015-253/ubuntu_host.html](https://docs.qualcomm.com/bundle/publicresource/topics/80-70015-253/ubuntu_host.html)
32. [https://stackoverflow.com/questions/16043079/serial-communication-read-line-c](https://stackoverflow.com/questions/16043079/serial-communication-read-line-c)
33. [https://www.abelectronics.co.uk/kb/article/1112/pyserial-rs232-serial-communication](https://www.abelectronics.co.uk/kb/article/1112/pyserial-rs232-serial-communication)
34. [https://stackoverflow.com/questions/990269/data-structure-for-storing-serial-port-data-in-firmware](https://stackoverflow.com/questions/990269/data-structure-for-storing-serial-port-data-in-firmware)
35. [https://github.com/wjwwood/serial](https://github.com/wjwwood/serial)
36. [https://www.flamingspork.com/blog/2017/10/20/zmodem-saves-the-day-or-why-my-firmware-for-a-machine-with-a-cpu-from-2017-contains-a-serial-file-transfer-protocol-from-the-1980s/](https://www.flamingspork.com/blog/2017/10/20/zmodem-saves-the-day-or-why-my-firmware-for-a-machine-with-a-cpu-from-2017-contains-a-serial-file-transfer-protocol-from-the-1980s/)
37. [https://www.youtube.com/watch?v=Kr1RyK6WENQ](https://www.youtube.com/watch?v=Kr1RyK6WENQ)
38. [https://makersportal.com/blog/2018/2/25/python-datalogger-reading-the-serial-output-from-arduino-to-analyze-data-using-pyserial](https://makersportal.com/blog/2018/2/25/python-datalogger-reading-the-serial-output-from-arduino-to-analyze-data-using-pyserial)
39. [https://lucidar.me/en/serialib/read-and-write-strings-on-serial-port-in-c-cpp/](https://lucidar.me/en/serialib/read-and-write-strings-on-serial-port-in-c-cpp/)
40. [https://ww1.microchip.com/downloads/aemDocuments/documents/OTH/ProductDocuments/ReferenceManuals/70005145b.pdf](https://ww1.microchip.com/downloads/aemDocuments/documents/OTH/ProductDocuments/ReferenceManuals/70005145b.pdf)
41. [https://developer.tuya.com/en/docs/iot/tuya-cloud-universal-serial-port-access-protocol?id=K9hhi0xxtn9cb](https://developer.tuya.com/en/docs/iot/tuya-cloud-universal-serial-port-access-protocol?id=K9hhi0xxtn9cb)
42. [https://www.beanair.com/files/UM-RF-08-ModBus-Messaging.pdf](https://www.beanair.com/files/UM-RF-08-ModBus-Messaging.pdf)
43. [https://robotics.stackexchange.com/questions/11897/how-to-read-and-write-data-with-pyserial-at-same-time](https://robotics.stackexchange.com/questions/11897/how-to-read-and-write-data-with-pyserial-at-same-time)
44. [https://stackoverflow.com/questions/6947413/how-to-open-read-and-write-from-serial-port-in-c](https://stackoverflow.com/questions/6947413/how-to-open-read-and-write-from-serial-port-in-c)
45. [https://github.com/sensemore/SMCom](https://github.com/sensemore/SMCom)
46. [https://community.st.com/t5/stm32-mcus-products/how-to-flash-stm32-mcu-from-yocto-linux-using-uart/td-p/600495](https://community.st.com/t5/stm32-mcus-products/how-to-flash-stm32-mcu-from-yocto-linux-using-uart/td-p/600495)
47. [https://ki-it.com/xml/12865/12865.pdf](https://ki-it.com/xml/12865/12865.pdf)
48. [https://discuss.python.org/t/pyserial-not-sending-string-to-serial-port-of-arduino/19696?page=2](https://discuss.python.org/t/pyserial-not-sending-string-to-serial-port-of-arduino/19696?page=2)
49. [https://www.youtube.com/watch?v=aipVmfXCBag](https://www.youtube.com/watch?v=aipVmfXCBag)
50. [https://patents.google.com/patent/US20190045396A1/en](https://patents.google.com/patent/US20190045396A1/en)
51. [https://community.st.com/t5/stm32cubemonitor-mcus/stm32cubemonitor-reading-from-a-serial-port/td-p/245136](https://community.st.com/t5/stm32cubemonitor-mcus/stm32cubemonitor-reading-from-a-serial-port/td-p/245136)
52. [https://koreascience.kr/article/JAKO202116047278079.view?orgId=kocon](https://koreascience.kr/article/JAKO202116047278079.view?orgId=kocon)
53. [https://python-forum.io/thread-14530.html](https://python-forum.io/thread-14530.html)
54. [https://alknemeyer.github.io/embedded-comms-with-python/](https://alknemeyer.github.io/embedded-comms-with-python/)
55. [https://stackoverflow.com/questions/59609133/not-able-to-write-and-read-from-serial-port-using-pyserial-python-script](https://stackoverflow.com/questions/59609133/not-able-to-write-and-read-from-serial-port-using-pyserial-python-script)
56. [https://stackoverflow.com/questions/77076050/pyserial-gets-no-response-from-the-bootloader-in-embedded-device](https://stackoverflow.com/questions/77076050/pyserial-gets-no-response-from-the-bootloader-in-embedded-device)
57. [https://forum.arduino.cc/t/serial-port-communication-between-python-and-arduino-in-embedded-c/449072](https://forum.arduino.cc/t/serial-port-communication-between-python-and-arduino-in-embedded-c/449072)
58. [https://www.xanthium.in/linux-serial-port-programming-using-python-pyserial-and-arduino-avr-pic-microcontroller](https://www.xanthium.in/linux-serial-port-programming-using-python-pyserial-and-arduino-avr-pic-microcontroller)
59. [https://electronics.stackexchange.com/questions/326659/serial-communication-between-embedded-system-and-pc-with-only-usb-ports-availabl](https://electronics.stackexchange.com/questions/326659/serial-communication-between-embedded-system-and-pc-with-only-usb-ports-availabl)
60. [https://projecthub.arduino.cc/ansh2919/serial-communication-between-python-and-arduino-663756](https://projecthub.arduino.cc/ansh2919/serial-communication-between-python-and-arduino-663756)
61. [https://forum.arduino.cc/t/trouble-with-serial-output-on-c/1068591](https://forum.arduino.cc/t/trouble-with-serial-output-on-c/1068591)
62. [https://forum.rakwireless.com/t/rak4631-firmware-update-over-uart-port-using-external-device/5528](https://forum.rakwireless.com/t/rak4631-firmware-update-over-uart-port-using-external-device/5528)
63. [https://stackoverflow.com/questions/466474/how-do-i-use-datareceived-event-of-the-serialport-port-object-in-c](https://stackoverflow.com/questions/466474/how-do-i-use-datareceived-event-of-the-serialport-port-object-in-c)
64. [https://github.com/espressif/arduino-esp32/issues/6727](https://github.com/espressif/arduino-esp32/issues/6727)
65. [https://ddtxrx.tistory.com/entry/Python-Serial-Communication](https://ddtxrx.tistory.com/entry/Python-Serial-Communication)
66. [https://stackoverflow.com/questions/75782057/how-to-transfer-a-string-through-serial-communication-and-extract-the-number-fro](https://stackoverflow.com/questions/75782057/how-to-transfer-a-string-through-serial-communication-and-extract-the-number-fro)
67. [https://chigun.tistory.com/22](https://chigun.tistory.com/22)
68. [https://forum.arduino.cc/t/how-to-send-data-continuously-on-1-serial-port-and-communicate-on-another/588366](https://forum.arduino.cc/t/how-to-send-data-continuously-on-1-serial-port-and-communicate-on-another/588366)
69. [https://forum.arduino.cc/t/esp8266-serial-not-working-cant-flash-firmware/1107122](https://forum.arduino.cc/t/esp8266-serial-not-working-cant-flash-firmware/1107122)
70. [https://stackoverflow.com/questions/52037166/in-the-python-serial-port-program-what-should-i-do-with-b](https://stackoverflow.com/questions/52037166/in-the-python-serial-port-program-what-should-i-do-with-b)
71. [https://learn.sparkfun.com/tutorials/serial-communication/all](https://learn.sparkfun.com/tutorials/serial-communication/all)
72. [https://stackoverflow.com/questions/7712408/c-ensuring-full-serial-response](https://stackoverflow.com/questions/7712408/c-ensuring-full-serial-response)
73. [https://predictabledesigns.com/firmware-programming-how-to-format-serial-communication-data/](https://predictabledesigns.com/firmware-programming-how-to-format-serial-communication-data/)