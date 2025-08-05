# PySide6 기초 및 구조
tags: #PySide6 #Qt #기초

## ✅ PySide6란?

| 항목    | 설명                                                    |
| ----- | ----------------------------------------------------- |
| 정식 명칭 | **PySide6 (Qt for Python)**                           |
| 기반    | Qt 6 (C++ 기반)                                         |
| 개발사   | The Qt Company                                        |
| 라이선스  | **LGPL** 또는 상용 라이선스                           |

## ✅ 주요 모듈

| 모듈        | 주요 클래스 예시                                     |
| --------- | --------------------------------------------- |
| QtCore    | `QObject`, `QThread`, `QTimer`, `QDateTime` 등 |
| QtWidgets | `QWidget`, `QDialog`, `QPushButton` 등         |
| QtGui     | `QPixmap`, `QImage`, `QColor`, `QFont` 등      |

# ✅ PySide6 한줄 정의

  

> **PySide6는 Qt6 기반의 Python GUI 툴킷으로, 윈도우, 버튼, 다이얼로그, 차트, 웹뷰 등을 만들 수 있는 라이브러리입니다.**

  

---

  

## 🔧 기본 정보

  

| 항목 | 내용 |

|------|------|

| 이름 | PySide6 |

| 기반 | Qt 6 (C++ GUI 프레임워크) |

| 제공사 | The Qt Company (공식) |

| Python 버전 | Python 3.6 이상 |

| 설치 | `pip install PySide6` |

| 대안 | PyQt6 (비슷하지만 라이선스 다름) |

| GUI 구성 방식 | 코드 기반, 또는 `.ui` 파일 (Qt Designer 사용 가능) |

  

---

  

## 🧩 주요 모듈 구성

  

| 모듈 | 설명 |

|------|------|

| `PySide6.QtWidgets` | 기본 GUI 위젯 (윈도우, 버튼, 레이아웃 등) |

| `PySide6.QtCore` | 신호-슬롯 시스템, 타이머, 쓰레드, 시간 등 |

| `PySide6.QtGui` | 이미지, 드래그앤드롭, 키보드 등 |

| `PySide6.QtWebEngineWidgets` | 웹브라우저 뷰 (크롬 엔진 기반) |

| `PySide6.QtCharts` | 차트 및 그래프 |

| `PySide6.QtMultimedia` | 오디오/비디오 재생 |

| `PySide6.QtSql` | SQL 연동 |

  

---

  

## 🖼️ 기본 예제 코드

  

```python

from PySide6.QtWidgets import QApplication, QLabel, QWidget, QVBoxLayout

import sys

  

app = QApplication(sys.argv)

  

window = QWidget()

window.setWindowTitle("PySide6 예제")

  

layout = QVBoxLayout()

label = QLabel("안녕하세요! PySide6입니다.")

layout.addWidget(label)

window.setLayout(layout)

  

window.show()

app.exec()

```

  

> 실행 시 간단한 텍스트가 있는 창이 나타납니다.

  

---

  

## 🎨 UI 디자인 방식

  

| 방법 | 설명 |

|------|------|

| 코드 작성 | `QWidget`, `QPushButton` 등을 직접 작성 |

| Qt Designer 사용 | `.ui` 파일 생성 후 `pyside6-uic`로 `.py` 변환 |

  

```bash

pyside6-uic design.ui -o design_ui.py

```

  

---

  

## ⚔️ PyQt6와의 차이

  

| 항목 | PySide6 | PyQt6 |

|------|---------|--------|

| 소유 | The Qt Company (공식) | Riverbank Computing |

| 라이선스 | LGPL (상업적 사용 가능) | GPL 또는 상용 라이선스 |

| API | 거의 동일 | 거의 동일 |

| 추천 용도 | 기업/상업 프로젝트 | 오픈소스 프로젝트 |

  

---

  

## 📦 설치 방법

  

```bash

pip install PySide6

```

  

> Qt Designer 사용 시:

> - Windows: [qt.io/download](https://www.qt.io/download)

> - Mac/Linux: Homebrew, apt 등으로 `qttools` 설치 필요

  

---

  

## 🧠 tkinter vs PySide6 비교

  

| 항목 | tkinter (`Canvas`, `TableWidget`) | PySide6 (`QTableWidget`, `UiDlg`) |

| 앱 초기화 | `root = tk.Tk()` | `app = QApplication(sys.argv)` |

| 메인 윈도우 | `Frame`, `Tk()` | `QMainWindow`, `QDialog`, `QWidget` |

| 이벤트 루프 | `root.mainloop()` | `app.exec()` |

| 위젯 배치 | `.pack()`, `.grid()` | `QVBoxLayout`, `.addWidget()` |

| 키 바인딩 | `.bind()` | `signal.connect()`, `keyPressEvent` |

| 테이블 구성 | 직접 그리기, 입력 위젯 추가 | `QTableWidget`, `QTableView` |

| 셀 편집 | Entry 위젯 따로 구성 | 기본적으로 셀 편집 가능 |

| 체크박스 | 캔버스에 직접 그림 | `QCheckBox` 기본 지원 |

  

---

  

## 🔍 tkinter 예제 흐름

  

1. `Canvas`에 직접 셀 그리기  

2. 키보드 입력 수동 처리  

3. 셀 클릭 → Entry 전환  

4. 직접 데이터 삽입/삭제/업데이트  

5. 수작업으로 레이아웃/좌표 관리

  

---

  

## 💡 PySide6 대체 방식

  

```python

from PySide6.QtWidgets import QTableWidget, QTableWidgetItem

  

table = QTableWidget()

table.setRowCount(3)

table.setColumnCount(4)

table.setItem(0, 0, QTableWidgetItem("Data A"))

```

  

- 셀 편집/정렬/체크박스 등 기본 지원  

- 캔버스 그리기, 좌표 계산 필요 없음  

- 유지보수 및 확장성 우수

  

---

  

## ✅ 핵심 비교 요약

  

| 항목 | tkinter | PySide6 |

|------|---------|---------|

| 난이도 | 직접 구현 필요 | 위젯 사용으로 간단 |

| 고급 기능 | 직접 그림 | 기본 제공 |

| 확장성 | 제한적 | 높음 |

| 시각적 품질 | 단순함 | 세련됨 (네이티브 UI) |

| 이벤트 처리 | `.bind()` | `signal.connect()` |

  

---

  

## 🏁 결론

  

> PySide6는 고급 위젯, 시그널/슬롯 구조, 시각적 품질 측면에서 tkinter보다 더 생산적이고 실용적인 GUI 개발 프레임워크입니다.