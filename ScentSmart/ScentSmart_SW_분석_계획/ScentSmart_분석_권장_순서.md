# 🧠 ScentSmart 분석 권장 순서

tags: #ScentSmart #분석전략 #코드읽기

## 1. 전체 워크플로 파악

- 메인 클래스: `UiDlg(QWidget)`
- 메뉴 → 검사/훈련/설정 → 저장 흐름 구성 파악
- UI-로직-시리얼 흐름 전체 큰 구조 이해

---

## 2. UI 흐름 정리

- `uiDlgStart()`, `uiDlgLogin()`, `uiDlgMain()` 등
- 사용자 동작 → 버튼 → 함수 연결 확인
- `.ui` 파일 구조도 병행 관찰

---

## 3. 검사 시나리오별 구조

- 검사 종류: 역치, 식별, 인지
- 각 검사 흐름 함수 이름 규칙:
  - `startTest`, `waitReady`, `testProceed`, `setResponseUi`, `sequential*`
- 상태 흐름 분기 파악

---

## 4. 시리얼 통신 흐름

- `requestScentWithValues` → dsComm 메시지 생성 → `write_data()`
- 수신은 스레드 emit → lambda → readSerialData

---

## 5. 저장 및 설정 흐름

- 검사 결과: Excel (`saveTestData*`)
- 설정: json 기반 (`loadSettingsFile`)
- 음성/텍스트: 폴더 구조 탐색

---

## 6. 유틸리티 및 구조 패턴

- 다이얼로그 전환, 사운드, 테이블 등 공통 UI 처리 구조
- dsText, dsSetting 등의 역할 분담 확인

---

## 분석 전략 요약

1. **메인 구조 → 기능별 흐름 → 시리얼/저장 분석**
2. 주요 호출 함수만 먼저 흐름 따라가면서 이해
3. 각 단계 핵심 함수 시각화 (다이어그램 or 트리)
