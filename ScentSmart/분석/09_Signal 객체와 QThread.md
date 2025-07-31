  

## ✅ `Signal` 객체에서 자주 쓰는 메서드 정리

  

Qt에서 **Signal**은 `PySide6.QtCore.Signal` 또는 `PyQt5.QtCore.pyqtSignal`로 선언하며, 이벤트 발생 알림용입니다.

Signal 자체는 보통 클래스를 선언할 때 정의하고, 다음 메서드들을 주로 사용합니다:

  

| 메서드                     | 설명                      |

| ----------------------- | ----------------------- |

| `connect(slot)`         | 시그널과 슬롯 함수(또는 메서드)를 연결  |

| `emit(*args)`           | 시그널을 발생시킴 (연결된 슬롯이 실행됨) |

| `disconnect(slot=None)` | 연결을 해제함 (전체 또는 특정 슬롯)   |

| `blockSignals(bool)`    | True일 경우 시그널 전달을 잠시 막음  |

| `signalsBlocked()`      | 시그널이 현재 차단되었는지 확인       |

  

### 🔸 예시

  

```python

from PySide6.QtCore import QObject, Signal

  

class MyObject(QObject):

    my_signal = Signal(str)

  

    def do_something(self):

        self.my_signal.emit("Hello")

  

def on_signal_received(value):

    print("시그널 받음:", value)

  

obj = MyObject()

obj.my_signal.connect(on_signal_received)

obj.do_something()  # → "시그널 받음: Hello"

```

  

---

  

## ✅ 3. `QThread.start(priority)`와 `QThread.Priority`

  

### 📌 `QThread.start(priority=None)`

  

* `QThread`는 Qt에서 쓰레드를 나타냅니다.

* `.start()`는 쓰레드를 실행시키는 메서드입니다.

* `priority` 인자를 주면 실행 우선순위를 정할 수 있어요.

  

### 📌 `QtCore.QThread.Priority`

  

`QThread.Priority`는 실행 우선순위를 나타내는 열거형(enum)입니다.

  

| 우선순위                   | 설명                 |

| ---------------------- | ------------------ |

| `IdlePriority`         | 가장 낮은 우선순위         |

| `LowestPriority`       |                    |

| `LowPriority`          |                    |

| `NormalPriority`       | 기본값                |

| `HighPriority`         |                    |

| `HighestPriority`      | 가장 높은 우선순위         |

| `TimeCriticalPriority` | 매우 긴급한 작업          |

| `InheritPriority`      | 부모 쓰레드로부터 우선순위를 상속 |

  

👉 즉,

  

```python

self._serial_read_thread.start(QtCore.QThread.Priority.HighestPriority)

```

  

는 **"이 쓰레드를 매우 높은 우선순위로 실행시켜라"** 라는 의미입니다.

  

---

  

## 🧠 요약

  

| 개념                        | 내용                                      |

| ------------------------- | --------------------------------------- |

| `start()` 호출 주체           | `QThread` 객체임 (`Signal` 아님)             |

| `Signal`의 주요 메서드          | `connect()`, `emit()`, `disconnect()` 등 |

| `QThread.start(priority)` | 쓰레드를 시작하며 우선순위 설정 가능                    |

| `QThread.Priority`        | 쓰레드 실행 우선순위 Enum                        |