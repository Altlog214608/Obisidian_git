# ✅ `uiDlgProtocol` 함수 완전 정리

`uiDlgProtocol(self, uiLoader)` 함수는 프로그램에서 **시리얼 통신 설정 및 프로토콜 UI**를 초기화하고 연결하는 핵심 함수입니다.

---

## 📌 1. UI 로드 및 위젯 초기화

```python
self.ui_data_protocol_dlg = uiLoader.load("./ui/ui_data_protocol.ui")
```

- QUiLoader로 `.ui` 파일을 로드해 위젯 인스턴스를 생성
- 시리얼 연결 및 각종 프로토콜 테스트 UI 포함

---

## 📌 2. 시리얼 포트 및 통신 파라미터 콤보박스 설정

```python
self.available_port_list = dsSerial._get_available_ports()
self.ui_data_protocol_dlg.comboBox_port.insertItems(0, [x.portName() for x in self.available_port_list])
```

- 현재 사용 가능한 시리얼 포트를 검색하여 포트 콤보박스에 추가
- 각종 통신 설정 파라미터(Baudrate, Databits, Parity 등)를 콤보박스에 삽입

---

## 📌 3. 버튼 이벤트 연결 (Signal → Slot)

```python
self.ui_data_protocol_dlg.pushButton_connect.clicked.connect(self.pushButton_connect_clicked)
```

- 시리얼 연결/발향/세정/정지/온도/압력 등 모든 버튼에 대응 함수 연결

---

## 📌 4. 윈도우 설정 적용

```python
self.setWindowBySetting(self.ui_data_protocol_dlg)
```

- 프레임리스 창, Always-on-top 등 공통 윈도우 속성 적용

---

## 📌 5. 시리얼 콤보박스 기본값 설정

```python
self.ui_data_protocol_dlg.comboBox_baudrate.setCurrentIndex(0)
self.ui_data_protocol_dlg.comboBox_databits.setCurrentIndex(3)
```

- Baudrate: 기본 9600bps
- Databits: 기본 8bit

---

## 📌 6. 콘솔 텍스트 에디트 연결

```python
self.setSerialConsole(self.ui_data_protocol_dlg.textEdit_console)
```

- 수신 로그를 표시할 텍스트 콘솔을 내부 멤버로 연결

---

## 📌 7. 자동 시리얼 포트 연결 시도

```python
if len(self.available_port_list) > 0:
    dsSerial._connect_default(self._serial, self._serial_read_thread, port_name=self.available_port_list[0].portName())
```

- 포트가 존재하면 자동으로 첫 포트에 기본 설정으로 연결 시도

---

## 📌 8. 연결 상태 텍스트 업데이트

```python
self.ui_data_protocol_dlg.pushButton_connect.setText(
    {False: dsText.serialText['status_connect'], True: dsText.serialText['status_disconnect']}[dsSerial._is_open(self._serial)]
)
```

- 시리얼 포트 상태에 따라 버튼 텍스트 갱신 (Connect ↔ Disconnect)

---

## ✅ 요약 역할 정리

| 구분         | 설명 |
|--------------|------|
| UI 로드       | `ui_data_protocol.ui`를 로드해 창 구성 |
| 포트 설정     | 사용 가능한 포트 검색 → 콤보박스 설정 |
| 파라미터 설정  | Baudrate, Databits 등 설정 값 표시 |
| 버튼 연결     | 연결/발향/세정/온도/압력 등 시그널 연결 |
| 콘솔 연결     | 로그용 QTextEdit 연결 |
| 자동 연결 시도 | 첫 포트에 자동 연결, 상태 반영 |
| 상태 업데이트 | 연결 버튼 텍스트 실시간 갱신 |

---

> `uiDlgProtocol`은 사용자가 직접 하드웨어 통신을 실험·설정·제어할 수 있는 **시리얼 통신 전용 테스트 패널 UI의 초기화 중심 함수**입니다.
