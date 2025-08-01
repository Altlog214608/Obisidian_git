아래는 **Qt Designer(및 PySide6/PyQt)**에서 자주 쓰이고, 기본적으로 제공되는 **위젯 종류(클래스)** 목록과 각 위젯의 대표적 용도, 그리고 실제 Python에서 어떤 클래스 이름으로 쓰는지, 어떤 상황에 주로 사용하는지 쉽고 직관적으로 정리한 표입니다.

# 1. Qt 기본 위젯 종류 (클래스별 용도)

|Qt 클래스 이름|PySide6/PyQt 클래스|대표 용도·설명|
|---|---|---|
|**QPushButton**|QPushButton|“버튼” (클릭, 제출, 확인, 취소 등 모든 기본 버튼)|
|**QLabel**|QLabel|고정 텍스트, 이미지, 안내문 등 “글자 또는 그림 표시”|
|**QLineEdit**|QLineEdit|한 줄짜리 입력박스 (이름, 숫자, 텍스트 등 입력)|
|**QTextEdit**|QTextEdit|여러 줄 입력(메모, 로그, 에디터 등)|
|**QPlainTextEdit**|QPlainTextEdit|QTextEdit과 유사, “문장 스타일 없는 단순 텍스트 에디터”|
|**QCheckBox**|QCheckBox|체크/해제 상태 입력 (on/off, 설정 등 토글 용)|
|**QRadioButton**|QRadioButton|여러 선택지 중 한 가지만 선택(예: 남/여, 옵션 등)|
|**QComboBox**|QComboBox|드롭다운 리스트 (여러 값 중 하나 선택)|
|**QSpinBox**|QSpinBox|“숫자 증감” 입력 (화살표 버튼으로 숫자 조절)|
|**QDoubleSpinBox**|QDoubleSpinBox|소수점 숫자 증감 입력|
|**QSlider**|QSlider|슬라이더(막대)로 값 조절 (볼륨, 세기, 사이즈 등)|
|**QProgressBar**|QProgressBar|“진행 상태” 시각화 (파일 다운로드, 실행 상태 등)|
|**QListWidget**|QListWidget|한 줄씩 항목(리스트) 보여주기 (선택, 목록 기반 UI)|
|**QTableWidget**|QTableWidget|표(테이블) 형태로 데이터 표시 및 편집 (엑셀 유사)|
|**QTreeWidget**|QTreeWidget|트리구조 폴더/디렉터리, 계층형 데이터 표시|
|**QTabWidget**|QTabWidget|탭(여러 페이지 전환처럼) 인터페이스|
|**QDateEdit**|QDateEdit|날짜 입력/선택 위젯|
|**QTimeEdit**|QTimeEdit|시간 입력/시계|
|**QCalendarWidget**|QCalendarWidget|달력에서 날짜 선택|
|**QGroupBox**|QGroupBox|관련 위젯 묶기(테두리/제목 표시)|
|**QFrame**|QFrame|단순한 구분선, 박스 등(구조적 레이아웃 정리)|
|**QWidget**|QWidget|“모든 위젯의 기본 단위”, 직접적으로 아무 역할 없이 컨테이너 용|
|**QScrollArea**|QScrollArea|스크롤이 필요한 영역 지정|
|**QHBoxLayout / QVBoxLayout / QGridLayout**|QHBoxLayout 등|레이아웃 배치 전용, 여러 위젯 자동 정렬|

# 2. 창/다이얼로그 관련 주요 클래스

|Qt 이름|Python 클래스|용도|
|---|---|---|
|**QMainWindow**|QMainWindow|프로그램 메인 창, 메뉴바/툴바/상태바 포함|
|**QDialog**|QDialog|대화창(모달팝업 등, 예: 설정, 알림, 팝업)|
|**QMessageBox**|QMessageBox|간단한 메시지/경고/확인창 (OK/Cancel 등)|
|**QFileDialog**|QFileDialog|파일 열기/저장/폴더 선택 다이얼로그|
|**QColorDialog**|QColorDialog|색상 선택기|
|**QFontDialog**|QFontDialog|폰트 선택 팝업|

# 3. 주요 파일/이미지/미디어 위젯

|Qt 이름|Python 클래스|용도|
|---|---|---|
|**QPixmap**|QPixmap|이미지 저장(버튼 배경, 라벨용 이미지 등)|
|**QImage**|QImage|이미지 처리(픽셀조작 등 기능 포함)|
|**QMovie**|QMovie|gif/애니메이션 이미지 표시|
|**QMediaPlayer**|QMediaPlayer (PySide6)|동영상/음악 재생(멀티미디어)|

# 4. 이벤트·타이머·스레드·툴 기능

|Qt 이름|Python 클래스|용도|
|---|---|---|
|**QTimer**|QTimer|일정 시간마다 실행(초시계, 애니메이션용)|
|**QThread**|QThread|백그라운드 스레드(병렬 실행, 시리얼 통신 등)|
|**QShortcut**|QShortcut|단축키 할당|
|**QAction**|QAction|메뉴/툴바 클릭액션(단축키, 트리거 등)|

# 5. PySide6/PyQt에서 흔히 쓰는 패키지별 "탑레벨 클래스 구조"

- **PySide6.QtWidgets**  
    → 모든 기본 위젯(QPushButton, QLabel, QMainWindow 등)
    
- **PySide6.QtCore**  
    → QTimer, QObject, Signal/Slot, QThread 등
    
- **PySide6.QtGui**  
    → QPixmap, QImage, QFont, QPalette 등
    

## 실전에서 자주 등장하는 코드 패턴 예시

python

`from PySide6.QtWidgets import (     QApplication, QMainWindow, QPushButton, QLabel, QLineEdit,    QDialog, QVBoxLayout, QHBoxLayout, QMessageBox, QProgressBar,    QListWidget, QTableWidget, QTabWidget ) from PySide6.QtCore import QTimer, QThread, Qt from PySide6.QtGui import QPixmap, QFont`

# 정리 요약

- **QPushButton, QLabel, QLineEdit, QTextEdit, QMainWindow, QDialog** 등은 가장 많이 쓰는 '기본 위젯'
    
- **위젯 class는 고정**: Qt 공식 클래스 이름을 반드시 써야 하며, `<widget class="QPushButton" name="btn1">`처럼 표시됨
    
- **name은 자유**: 각 위젯을 코드에서 구별하기 위한 고유 이름(식별자)
    
- **PySide6/PyQt는 Qt 지원하는 모든 GUI 관련 클래스를 거의 1:1 매칭**
    

---
