내장 시그널(빌트인 시그널, built-in signal)이란  
**PyQt/PySide에서 각 위젯 클래스가 “클래스 정의 내부에 이미 포함”하고 있는,  
특정 상황(이벤트)이 발생할 때 자동으로 발생하는 Signal 객체**를 말합니다.  
즉, 개발자가 따로 pyqtSignal로 선언할 필요 없이 바로 사용할 수 있는 Qt 시스템 내 기본 이벤트입니다.

## 대표적인 내장 시그널 정리 (주요 위젯별)

## 1. QPushButton (버튼)

- **clicked()**: 버튼이 클릭(press & release)될 때
    
- **pressed()**: 버튼을 누르는 순간
    
- **released()**: 버튼에서 손을 떼는 순간
    
- **toggled(checked:bool)**: 체크 가능한 버튼이 상태를 바꿀 때[pythonguis](https://www.pythonguis.com/docs/qpushbutton/)
    

## 2. QSlider, QDial (슬라이더/다이얼)

- **valueChanged(int)**: 값이 바뀔 때
    
- **sliderMoved(int)**: 슬라이더가 사용자의 마우스/키 조작으로 움직일 때
    
- **sliderPressed()**: 핸들을 누르기 시작할 때
    
- **sliderReleased()**: 핸들을 떼었을 때[pythonguis](https://www.pythonguis.com/tutorials/pyqt-basic-widgets/)
    

## 3. QLineEdit (한 줄 텍스트 입력)

- **textChanged(str)**: 텍스트가 바뀔 때(프로그램적으로든 사용자가든)
    
- **textEdited(str)**: 사용자가 텍스트를 직접 수정할 때(입력)
    
- **returnPressed()**: 사용자가 Enter(리턴)키를 눌렀을 때
    
- **editingFinished()**: 편집이 끝난 뒤 포커스가 바뀌면
    

## 4. QComboBox (콤보박스/드롭다운)

- **currentIndexChanged(int/str)**: 선택 항목이 바뀔 때(인덱스 또는 텍스트)
    
- **activated(int/str)**: 아이템이 선택·활성화될 때
    
- **highlighted(int/str)**: 목록에서 마우스로 하이라이트될 때
    

## 5. QCheckBox (체크박스)

- **stateChanged(int)**: 체크 상태가 바뀔 때 (0=해제, 2=체크)
    
- **clicked(bool)**: 클릭될 때
    

## 6. QRadioButton (라디오버튼)

- **toggled(bool)**: 체크·해제 상태 바뀔 때
    

## 7. QSpinBox/QDoubleSpinBox

- **valueChanged(int/float)**: 값이 바뀔 때
    
- **editingFinished()**: 입력/수정 끝났을 때
    

## 8. QTabWidget

- **currentChanged(int)**: 현재 탭이 바뀔 때
    

## 9. QMainWindow/QDialog 등 창

- **accepted()**, **rejected()**: 다이얼로그, OK/취소 등 액션 시
    
- **closeEvent** (이건 이벤트 처리 메서드지만, 대화상자에서 signal로 emit 가능)
    

## 내장 시그널의 특징

- 각 위젯의 주요 상호작용(클릭, 값 변경, 입력 등)에 대해 미리 정의되어 있음
    
- connect(시그널, 슬롯) 형식으로 바로 연결해 사용
    
    
    
    ```python
    self.button.clicked.connect(self.on_button_clicked) 
    self.slider.valueChanged.connect(self.on_slider_moved)
    ```
    
- 시그널마다 전달하는 인자가 다름(클릭: 없음, 값변경: int 등)
    
- **직접 pyqtSignal로 만들 필요 없음**
    

## 주요 메서드(내장 슬롯, 즉 반응 함수)

아래는 각 위젯에서 기본적으로 지원하는 대표적 “내장 메서드(슬롯)”입니다:

- **setText(str)**: QLabel, QPushButton, QLineEdit 등
    
- **setValue(int/float)**: QSlider, QSpinBox 등
    
- **setChecked(bool)**: QCheckBox 등
    
- **setEnabled(bool), setVisible(bool)**: 거의 모든 QWidget 파생
    
- **clear()**: QLineEdit, QTextEdit 등
    

이 메서드(slot)는 signal과 직접 연결해서 동작시킬 수 있습니다.

## 참고/활용 팁

- 각 위젯마다 시그널 이름·종류가 다르므로, 공식 문서나 PythonGUIs/위키독스 등에서 자주 찾으면 좋습니다.
    
- 슬롯(메서드)는 **내장 메서드 or 개발자 함수 무엇이든 연결 가능**
    
- 예: QLineEdit의 textChanged를 QLabel의 setText와 바로 연결!
    

**요약**

- 내장 시그널은 위젯마다 이미 내장되어 “값 변화, 클릭, 입력 등” 주요 이벤트 때 호출됨
    
- 따로 pyqtSignal 없이 곧바로 connect해서 쓸 수 있음
    
- 주로 .clicked, .valueChanged, .textChanged 등 자주 활용
    

