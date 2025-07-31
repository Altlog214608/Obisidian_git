# ✅ ScentSmart.py 전체 함수 구조 정리 (역할 중심)

ScentSmart.py는 PySide6 기반 GUI와 시리얼 통신을 결합하여 **디지털 후각 검사 시스템**을 구현한 메인 애플리케이션입니다. 아래는 모든 함수들을 기능별로 정리한 문서입니다.

---

## 1. 📡 시리얼 통신 / 하드웨어 제어

| 함수명 | 역할 |
|--------|------|
| `setSerialReadThread()` | 시리얼 포트 연결 및 수신 스레드 초기화, 시그널 연결 |
| `readSerialData(rdata)` | 수신된 데이터 콘솔에 기록, `parseReadData`로 전달 |
| `write_data(wdata)` | 바이트 패킷을 시리얼 포트에 송신하고 로그 남김 |
| `parseReadData(data)` | 받은 데이터를 프로토콜에 따라 해석 |
| `requestFrequency(f)` | 주파수 설정 명령 생성 및 전송 |
| `requestScentNo/AndTime/WithValues()` | 발향, 세정 관련 명령 생성 및 전송 |
| `requestTempPress()` | 온도 및 압력 요청 패킷 전송 |
| `progressBarScentAndClean/ForTrainST/ForTrainID()` | 발향/세정 동작과 UI 진행바, 대기, 사운드 처리 |

---

## 2. 🧩 UI 이벤트 / 창 전환 및 상태 제어

| 함수명 | 역할 |
|--------|------|
| `eventFilter()` | 키보드 이벤트 필터링 (Enter/Tab 커스터마이징) |
| `setWindowBySetting(dialog)` | 윈도우 상단 바 및 고정 옵션 설정 |
| `uiDlgChange/Show/Hide/ShowUp/ChangeWithDlg()` | 여러 다이얼로그 전환 및 애니메이션/표시 효과 |
| `setSerialConsole()` | 외부에서 콘솔 텍스트 위젯을 주입 |
| `pushButton_*_clicked()` | 프로토콜 UI 버튼 이벤트 → 시리얼 명령 실행 |

---

## 3. 🔐 로그인/사용자 인증

| 함수명 | 설명 |
|--------|------|
| `uiDlgLoginStart`, `clearPWEdit`, `checkPW`, `countErrorPW` | 로그인 절차, 오류횟수 관리 |
| `loadPWFile`, `savePWFile` | 비밀번호 로컬 저장/불러오기 |
| `uiDlgLoginResetPW`, `uiDlgLoginResetPWReset` | 비밀번호 재설정 프로세스 처리 |

---

## 4. 🛠 설정/메뉴/검사 선택 진입

| 함수명 | 설명 |
|--------|------|
| `uiMenuBtn*` | 메뉴 내 검사/훈련/결과/설정 선택 시 다이얼로그 연결 |
| `uiDlgSettings`, `updateSettingsUI`, `uiSettingUpdateSettings` | 설정화면 로딩 및 값 동기화 |
| `loadSettingsFile`, `saveSettingsFile` | 설정 파일 로딩/저장(JSON) |

---

## 5. 🧪 검사 및 훈련 시나리오

### 역치(Threshold), 식별(Discrimination), 인지(Identification)

- `initTest*` : 초기 변수/화면 설정
- `uiTest*Start/Resume/Proceed/GuidePictureBtnBack/Forward/TryScent`
- `waitTest*Ready`, `sequential*`
- `uiTest*ResponseRetry/Next/Choice1/2/3/4`
- `checkResponse*`, `confirmResponse*`
- `select/unselectResponse*`
- `saveTestData*`, `makeTestResults*`

**공통 특징**:
- 다단계 프로세스 (가이드 → 준비 → 응답 → 저장)
- 사용자 입력 처리, 사운드 안내, 데이터 누적
- 공통 구조를 다수의 검사 유형에 맞게 재활용

---

## 6. 🧠 훈련 (ST/ID)

| 함수명 | 설명 |
|--------|------|
| `uiTrainST*`, `TrainSTProceed`, `sequentialTrainST`, `rateTrainST`, `waitTrainSTReady` | 순차 훈련 시나리오 |
| `uiTrainID*`, `TrainIDProceed` 등 | 인지 훈련용 시나리오 |

---

## 7. ⏱ 타이머/점수

| 함수명 | 설명 |
|--------|------|
| `uiDlgTimer`, `testTimerTimeout`, `updateUiTimes` | 검사 시간 측정 및 진행률 표시 |
| `set*Score`, `setTestsScores` | 점수 계산 및 UI에 적용 |

---

## 8. 📁 DB/엑셀 저장

| 함수명 | 설명 |
|--------|------|
| `uiDlgDB` | DB 테이블 초기화 및 표시 |
| `saveDataThreshold/Discrimination/Identification/ResultsTemp` | 결과 파일 엑셀 저장 (xlsxwriter 사용), 차트 포함 |

---

## 9. 📦 기타 유틸/헬퍼 함수

| 설명 |
|------|
| select/unselect UI 항목, 사운드 재생, 테이블 업데이트, 값 초기화 등 다수 보조 함수 포함 |

---

## ✅ 결론 요약

ScentSmart.py는 다음 5가지 기능 축을 중심으로 구성되어 있습니다:

1. **UI 흐름 제어** – 다이얼로그 간 전환, 이벤트 필터, 시각적 제어
2. **검사/훈련 시나리오** – 검사별 로직을 단계적으로 구현, 사용자 입력과 저장 처리
3. **시리얼 통신** – 장치 제어 명령 생성 및 응답 수신
4. **결과 저장 및 설정 관리** – JSON, Excel 저장 및 UI 반영
5. **타이머 및 점수 계산** – 검사 시간 측정과 결과 점수 계산

> 전반적으로 *UI 제어 ↔ 하드웨어 통신 ↔ 사용자 데이터 처리*가 긴밀하게 엮인 구조로 되어 있으며, 유지보수성과 기능 분리가 잘 고려된 설계입니다.
