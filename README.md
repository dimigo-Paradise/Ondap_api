# Ondap_api

## 폴더 구조

```text
Ondap_api/
├─ src/ondap_api/
│  ├─ api/v1/routers/
│  │  ├─ senior/                  # 어르신 앱용 API
│  │  ├─ carer/                   # 보호자 앱용 API
│  │  └─ internal/                # Ondap_server가 작업 결과를 보고하는 내부 API (외부 비공개)
│  ├─ api/dependencies/           # 사용자 인증·요청별 권한 확인
│  ├─ core/                       # 설정·로깅·공통 예외·보안 유틸리티
│  ├─ db/                         # SQLAlchemy 연결·세션·모델 등록
│  ├─ modules/                    # 기능별 모델·스키마·서비스·저장소
│  │  ├─ accounts/                # 어르신·보호자 계정·로그인·세션
│  │  ├─ tenants/                 # 운영 주체 (지금은 자체 서비스 1건, 모든 테이블에 tenant_id)
│  │  ├─ families/                # 가족·구성원·초대 코드
│  │  ├─ consents/                # 공유 범위·원문 보관 동의·철회
│  │  ├─ stories/                 # 어르신 녹음 원본·전사·요약 (원문 보관)
│  │  ├─ replies/                 # 보호자 음성 답장·버전·승인
│  │  ├─ policies/                # 승인 상태·금지 주제·전달 시간/횟수 검사
│  │  ├─ deliveries/              # policies를 통과한 답장의 전달·재생 기록
│  │  ├─ notifications/           # 앱 푸시 토큰·발송 기록
│  │  ├─ jobs/                    # Ondap_server에 맡긴 작업의 상태 추적
│  │  ├─ audit/                   # 열람·승인·삭제 기록 (수정 불가)
│  │  └─ privacy/                 # 보관기한·삭제 요청·원문 접근 통제
│  └─ integrations/               # 외부 서비스 어댑터
│     ├─ storage/                 # 비공개 S3 업로드·다운로드 주소 발급
│     ├─ queue/                   # Ondap_server로 작업 발행 (Redis)
│     └─ push/                    # FCM(Android)·APNs(iOS) 발송
├─ alembic/versions/              # DB 변경 이력 (DB 스키마는 이 저장소만 소유)
├─ contracts/
│  ├─ openapi/                    # 앱용 API 명세. FastAPI에서 자동 생성, 손으로 고치지 않음
│  ├─ jobs/                       # Ondap_api ↔ Ondap_server 작업 메시지·결과 보고 형식
│  └─ audio/                      # 앱 녹음 형식·길이·용량 규격
├─ tests/
│  ├─ unit/                       # 서비스·정책 단위 테스트
│  ├─ api/                        # 엔드포인트 테스트
│  └─ integration/                # 실제 DB·Redis·S3 연동 테스트
├─ deploy/                        # Docker Compose (PostgreSQL·Redis·S3 호환 저장소)·Nginx 설정
├─ scripts/                       # 개발 실행·명세 내보내기 도구
├─ docs/                          # 설계 결정·운영 문서
└─ .github/workflows/             # 검사·빌드·배포
```

## 필요해질 때 추가할 것

| 위치 | 내용 | 추가 시점 |
|---|---|---|
| `api/v1/routers/device/`, `modules/devices/` | 스피커 기기 등록·인증·상태 | 기기 파일럿 시작 |
| `modules/partners/` | 사업자·API 키·이벤트 수신 | 첫 파트너 연동 |
| `modules/firmware/` | 기기 펌웨어 배포·OTA | 기기 배포 전 |

## 온답 저장소 구성

| 저장소 | 역할 |
|---|---|
| [Ondap_app_senior](https://github.com/dimigo-Paradise/Ondap_app_senior) | 어르신 앱 (Android·iOS). 이야기 녹음, 가족 답장 듣기. **MVP의 어르신 쪽 접점** |
| [Ondap_app_carer](https://github.com/dimigo-Paradise/Ondap_app_carer) | 보호자 앱 (Android·iOS). 요약 확인, 답장 녹음·승인 |
| [Ondap_api](https://github.com/dimigo-Paradise/Ondap_api) | HTTP API 서버 (Python·FastAPI). **DB를 소유하는 유일한 저장소** |
| [Ondap_server](https://github.com/dimigo-Paradise/Ondap_server) | 백그라운드 작업 (Python·Celery). STT·마스킹·요약·오디오 변환. DB 직접 접근 없음 |
| [Ondap_embed](https://github.com/dimigo-Paradise/Ondap_embed) | 스피커 기기 펌웨어 (ESP-IDF). MVP 이후 |

```text
senior app ─┐
            ├─ HTTPS ─▶ Ondap_api ─▶ PostgreSQL · S3 · FCM/APNs
carer app  ─┘              │  ▲
                           ▼  │   작업 발행(Redis) ↓ / 결과 보고(내부 API) ↑
                       Ondap_server ─▶ STT · 요약 API
```
