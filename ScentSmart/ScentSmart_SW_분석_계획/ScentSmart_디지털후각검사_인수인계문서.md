# 📄 디지털 후각 검사 소프트웨어 인수인계 문서

tags: #ScentSmart #인수인계 #디지털후각검사

## 1. 개요

- **명칭**: 디지털 후각 검사 소프트웨어
- **버전**: v1.0.0
- **개발언어**: Python 3.10+, PySide6
- **운영환경**: Windows 10 64bit 이상
- **문서 목적**: 유지보수·운영 가능하도록 인수인계

---

## 2. 시스템 구조 개요

- PySide6 GUI ↔ SQLite DB ↔ RS-232 시리얼 ↔ STM32 펌웨어
- 모듈 구성:
  - Login, Subject, Test, Report, Comm, Interface, Data Manager 등

---

## 3. 개발환경 및 라이브러리

- Python 3.10+
- VS Code
- 주요 라이브러리: PySide6, pyserial, xlsxwriter, winsound

---

## 4. 실행 방법

```bash
pip install pyside6 pyserial xlsxwriter
python main.py
```

---

## 5. 주요 기능 요약

- 로그인/사용자 관리
- 환자 정보 및 검사 결과 리포트
- 이미지/음성 안내, 발향 연동
- 결과 저장: Excel 리포트

---

## 6. DB 구조

- DS_TEST_SUBJECT: 환자 정보
- DS_TEST_ID: 검사 이력
- SQLite 파일 구조는 `dsTestDB` 기준

---

## 7. 유지보수 포인트

- 파라미터: `dsSetting.dsParam`
- 텍스트/음성: `dsText`, `guideSound`
- 보고서 양식: `xlsxwriter` 모듈

---

## 8. 오류 및 예외 처리

- 비밀번호 오류, 포트 오류, DB 조회 오류, 파일 저장 실패 등

---

## 9. UI 개선 평가

| 항목 | 평가 |
|------|------|
| GUI 품질 | 기본 Qt 스타일 |
| 반응형 | 없음 |
| 개선 난이도 | 중간 |
| 외주 여부 | 필요시 권장 |

---

## 10. 기타 참고

- 암호화: SHA256 + Fernet
- MODBUS 기반 통신
- `ds*.py` 중심으로 구성됨
