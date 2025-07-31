# ✅ ScentSmart 인수인계 문서 작성 구조안

tags: #인수인계 #ScentSmart #구조안

## 1. 개요 (Overview)

- 소프트웨어 명칭 및 버전
- 주요 기능 요약
- 개발 언어 및 플랫폼 (Python 3.10+, PySide6 기반)
- 문서 목적: 유지보수·커스터마이징·운영을 위한 기술 인수인계

---

## 2. 시스템 구조 요약

- 전체 아키텍처 요약 (도식 + 간단 설명)
  - 예: PySide6 GUI ↔ DataManager ↔ DB ↔ Firmware 통신 (MODBUS, RS-232)
- 주요 모듈 목록 및 역할
  - Login-Manager, Subject-Manager, Test-Manager, Interface, Comm-Manager 등

---

## 3. 개발 환경 및 구성요소

- Python 3.10+
- 주요 라이브러리:
  - `pyside6`, `pyserial`, `xlsxwriter`
- 운영환경: Windows 10 이상, IDE: VS Code 권장

---

## 4. 모듈별 기능 및 흐름 요약

- Login-Manager: 로그인 및 인증
- Subject-Manager: 환자/검사 이력 관리
- Test-Manager: 후각 검사 단계 흐름 제어
- Interface: UI 창 전환 및 음성 안내
- Comm-Manager: 발향/세정 시리얼 명령 송신
- Firmware: 하드웨어 수신/제어 응답

---

## 5. DB 구조 요약

- 주요 테이블: DS_TEST_SUBJECT, DS_TEST_ID
- 필드/관계 및 키 요약

---

## 6. 실행 방법 및 배포

- 설치, 가상환경, 라이브러리 설치 방법
- 실행 명령: `python main.py`
- 배포: pyinstaller 또는 cx_Freeze 가능

---

## 7. 자주 발생하는 에러 및 대응

- 비밀번호 오류
- 장치 연결 오류
- DB 손상 또는 저장 실패 등

---

## 8. UI 리뉴얼 판단

| 항목 | 판단 |
|------|------|
| 디자인 | 기본 Qt 스타일 (현대성 낮음) |
| 반응형 | 없음 |
| 유지보수 난이도 | 낮음 |
| 외주 필요성 | 시각 중심 기능 확장 시 권장 |

---

## 9. 향후 커스터마이징 예시

- 검색 필터, PDF 출력, WebSocket, 테마 추가 등

---

## 10. 담당자 메모 / 주의사항

- 암호화/설정 모듈 위치
- 펌웨어 테스트 방법
- 데이터 백업 위치
