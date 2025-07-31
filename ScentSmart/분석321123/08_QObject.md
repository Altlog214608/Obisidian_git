# ✅ QObject를 상속하는 이유

  

## 1. **시그널(Signal)/슬롯(Slot)** 기능 사용 가능

  

* Qt에서 `Signal`은 `QObject`을 상속받은 클래스에서만 동작합니다.

* `Signal` 자체도 `QObject`의 내부 매커니즘에 의존해 작동합니다.

* 만약 `QObject`을 상속하지 않으면 `.emit()`이나 `.connect()`가 동작하지 않아요.

  

```python

class MyObject(QObject):

    my_signal = Signal(str)  # ✅ 가능

  

class NotQObject:

    my_signal = Signal(str)  # ❌ 오류 발생 (QObject 미상속)

```

  

---

  

## 2. **이벤트 시스템(event loop)과 통합 가능**

  

* `QObject`은 Qt의 **이벤트 루프 시스템**의 기본 단위입니다.

* 예: 타이머(`QTimer`), UI 이벤트, 커스텀 이벤트 등은 모두 `QObject` 기반에서 작동합니다.

  

---

  

## 3. **메모리 관리 (parent-child 구조)**

  

* `QObject`은 **부모-자식 객체 트리 구조**를 지원합니다.

* 부모가 삭제되면 자식도 자동 삭제 → 메모리 누수 방지

* UI 위젯, 타이머, 쓰레드 등에서 매우 중요합니다.

  

```python

parent = QObject()

child = QObject(parent)

# parent가 delete되면 child도 자동 delete

```

  

---

  

## 4. **시그널 차단(block), 연결 해제(disconnect), 객체 추적(objectName) 등**

  

* `QObject`을 상속하면 다음과 같은 유틸 기능도 사용 가능:

  

  * `blockSignals(bool)`

  * `signalsBlocked()`

  * `setObjectName("이름")`

  * `findChild()`, `findChildren()` 등

  

---

  

## ✅ 요약 정리

  

| 기능            | QObject 상속이 필요한 이유                    |

| ------------- | ------------------------------------- |

| Signal / Slot | `QObject`이 내부적으로 관리함                  |

| 이벤트 루프        | `QObject` 기반 구조                       |

| 메모리 관리        | 부모-자식 관계로 자동 삭제                       |

| 기타 유틸 기능      | `objectName`, signal blocking 등 사용 가능 |