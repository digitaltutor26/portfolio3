# portfolio3 — 법원 감정 케이스 관리 시스템 (MCP 연동)

이 저장소는 법원 감정 업무를 위한 로컬 웹 애플리케이션과 향후 MCP(Model Context Protocol) 기반 AI 서비스 연동을 목표로 구성된 포트폴리오입니다.

---

## 저장소 구성

```
portfolio3/
├── README.md            ← 이 파일
└── court-cases.zip      ← 법원 감정 케이스 관리 시스템 소스 코드 (압축)
```

### court-cases.zip 내부 구조

압축을 풀면 아래 구조가 나타납니다.

```
court-cases/
├── client/
│   └── index.html          # 단일 파일 프론트엔드 (HTML + CSS + Vanilla JS)
│                           # 케이스 목록/등록/수정/검색, 문서 업로드 UI 포함
│
├── server/
│   ├── package.json        # 의존성: express, better-sqlite3, multer, typescript 등
│   ├── tsconfig.json       # TypeScript 컴파일 설정 (ESM 모드)
│   └── src/
│       ├── index.ts        # 서버 진입점 — Express 앱 설정, 정적 파일 서빙, 포트 바인딩
│       ├── db.ts           # SQLite 연결 및 초기화, 스키마 자동 적용, ID 생성 유틸
│       ├── types.ts        # 공유 타입 정의 (Case, CaseDocument, CaseEvent 인터페이스)
│       └── routes/
│           ├── cases.ts    # 케이스 CRUD REST API (/api/cases)
│           └── docs.ts     # 문서 업로드/다운로드 API (/api/cases/:id/documents)
│
├── scripts/
│   └── schema.sql          # SQLite 스키마 (cases / documents / events 테이블)
│
├── db/                     # SQLite DB 파일 위치 (실행 시 자동 생성)
│   └── cases.db            # (자동 생성됨)
│
├── files/                  # 케이스별 첨부 문서 저장 폴더 (실행 시 자동 생성)
│   └── <case_id>/          # (케이스 ID별 하위 폴더 자동 생성)
│
├── README.md               # 빠른 시작 가이드
├── SETUP.md                # 상세 설치·실행 가이드 (초보자용)
└── MIGRATION.md            # 향후 클라우드 이전 체크리스트 (PostgreSQL, S3/R2)
```

---

## 주요 파일 설명

### 백엔드 (Node.js + Express + TypeScript)

| 파일 | 역할 |
|------|------|
| `server/src/index.ts` | Express 앱 초기화, CORS·JSON 미들웨어, API 라우터 연결, 정적 파일 서빙, 0.0.0.0 포트 바인딩 |
| `server/src/db.ts` | `better-sqlite3` 연결, WAL 모드·외래키 활성화, 스키마 자동 적용, 프로세스 종료 시 안전한 DB 닫기 |
| `server/src/types.ts` | `Case`, `CaseDocument`, `CaseEvent` 인터페이스 및 `CaseStatus` 유니언 타입 정의 |
| `server/src/routes/cases.ts` | 케이스 목록 조회(검색/필터), 단건 조회, 등록, 수정, 삭제 REST 엔드포인트 |
| `server/src/routes/docs.ts` | `multer`를 사용한 문서 업로드, 문서 목록 조회, 다운로드, 삭제 엔드포인트 |

### 데이터베이스 스키마 (SQLite)

| 테이블 | 설명 |
|--------|------|
| `cases` | 사건번호, 법원, 재판부, 감정 유형, 진행상태, 감정료, 쟁점, 결과, 비고 등 케이스 메타데이터 |
| `documents` | 케이스별 첨부 파일 정보 (원본 파일명, 저장 경로, 문서 유형, MIME 타입 등) |
| `events` | 케이스 진행 이력 (감정신청/현장조사/감정서제출/판결 등 날짜별 이벤트) |

### 프론트엔드

| 파일 | 설명 |
|------|------|
| `client/index.html` | 단일 HTML 파일로 구성된 SPA. 케이스 목록·검색·필터, 등록/수정 폼, 문서 업로드/다운로드 UI 포함 |

### 문서

| 파일 | 내용 |
|------|------|
| `README.md` | 프로젝트 개요 및 빠른 시작 |
| `SETUP.md` | Node.js 설치부터 외부 접속(Tailscale) 설정, 백업, 자주 쓰는 명령까지 단계별 안내 |
| `MIGRATION.md` | SQLite → PostgreSQL, 로컬 파일 → AWS S3/Cloudflare R2, 호스팅 이전 체크리스트 |

---

## 기술 스택

| 구분 | 기술 |
|------|------|
| 백엔드 | Node.js, Express, TypeScript (ESM) |
| 데이터베이스 | SQLite (`better-sqlite3`) |
| 파일 처리 | Multer (로컬 디스크 저장) |
| 프론트엔드 | HTML + CSS + Vanilla JavaScript (빌드 도구 없음) |
| 외부 접속 | Tailscale (사설 VPN) |

---

## 시스템 API 개요

```
GET    /api/health                          # 서버 상태 확인
GET    /api/cases                           # 케이스 목록 (검색·필터 쿼리 파라미터 지원)
GET    /api/cases/:id                       # 케이스 단건 조회
POST   /api/cases                           # 케이스 등록
PUT    /api/cases/:id                       # 케이스 수정
DELETE /api/cases/:id                       # 케이스 삭제

GET    /api/cases/:caseId/documents         # 문서 목록
POST   /api/cases/:caseId/documents         # 문서 업로드 (multipart/form-data)
GET    /api/documents/:docId/download       # 문서 다운로드
DELETE /api/documents/:docId                # 문서 삭제
```

---

## 향후 계획 (MCP 연동)

현재 업로드된 MCP를 기반으로 아래 기능 확장을 검토 중입니다.

- **AI 자연어 케이스 검색**: "지난달 제출 완료된 건축 감정 건 목록 보여줘" 형태의 질의
- **문서 자동 분류**: 업로드된 PDF에서 문서 유형(감정서/판결문/도면 등) 자동 인식
- **감정서 초안 생성**: 케이스 정보 기반 감정서 양식 자동 작성 보조
- **클라우드 이전**: PostgreSQL + S3/R2 기반으로 전환 후 외부 배포

---

## 빠른 시작

```bash
# 1. 압축 해제
unzip court-cases.zip

# 2. 의존성 설치
cd court-cases/server
npm install

# 3. 실행
npm run dev

# 4. 브라우저 접속
# http://localhost:3000
```

자세한 설치 방법은 `court-cases/SETUP.md`를 참고하세요.
