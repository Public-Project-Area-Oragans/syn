# 4. API 명세서

> **프로젝트명**: Synapse — 통합 학습-지식 그래프 SaaS
> **버전**: v1.0
> **작성일**: 2026-05-07
> **기술 스택**: Spring Boot 4, Flutter 3.x, FastAPI, PostgreSQL 16, Redis, Elasticsearch, Kafka, K8s

---

## 4.1 공통 규약

### Base URL

```
Production: https://api.synapse.app/api/v1
Staging:    https://api-staging.synapse.app/api/v1
```

### 인증

- **방식**: JWT Bearer Token (httpOnly Cookie 자동 전송)
- **헤더**: `Authorization: Bearer <access_token>` (Cookie 불가 시)
- **테넌트**: `X-Tenant-Id` 헤더 (Gateway 자동 주입)

### 공통 응답 형식

```json
{
  "success": true,
  "data": { ... },
  "meta": {
    "timestamp": "2026-05-07T10:00:00Z",
    "requestId": "req-uuid"
  }
}
```

### 에러 응답

```json
{
  "success": false,
  "error": {
    "code": "NOTE_NOT_FOUND",
    "message": "요청한 노트를 찾을 수 없습니다.",
    "details": []
  },
  "meta": {
    "timestamp": "2026-05-07T10:00:00Z",
    "requestId": "req-uuid"
  }
}
```

### HTTP 상태 코드

| 코드 | 의미 | 사용 |
|------|------|------|
| 200 | OK | 조회/수정 성공 |
| 201 | Created | 생성 성공 |
| 204 | No Content | 삭제 성공 |
| 400 | Bad Request | 유효성 검증 실패 |
| 401 | Unauthorized | 인증 실패 |
| 403 | Forbidden | 권한 부족 / 할당량 초과 |
| 404 | Not Found | 리소스 없음 |
| 409 | Conflict | 중복 리소스 |
| 422 | Unprocessable | 비즈니스 규칙 위반 |
| 429 | Too Many Requests | Rate Limit 초과 |
| 500 | Internal Error | 서버 오류 |

### 페이지네이션

```
GET /notes?cursor={lastId}&limit=20&sort=updated_at:desc
```

```json
{
  "data": [...],
  "pagination": {
    "cursor": "next-cursor-value",
    "hasMore": true,
    "totalCount": 150
  }
}
```

### Rate Limiting 헤더

```
X-RateLimit-Limit: 1000
X-RateLimit-Remaining: 999
X-RateLimit-Reset: 1620000000
```

---

## 4.2 Auth 도메인

### POST /auth/signup

회원가입 (이메일)

```json
// Request
{
  "email": "user@example.com",
  "password": "SecureP@ss1!",
  "displayName": "홍길동",
  "locale": "ko"
}

// Response 201
{
  "data": {
    "userId": "uuid",
    "tenantId": "uuid",
    "email": "user@example.com",
    "displayName": "홍길동"
  }
}
```

### POST /auth/login

```json
// Request
{
  "email": "user@example.com",
  "password": "SecureP@ss1!"
}

// Response 200 (MFA 미설정 시)
{
  "data": {
    "accessToken": "jwt...",
    "expiresIn": 900,
    "user": { "id": "uuid", "email": "...", "displayName": "..." }
  }
}
// Set-Cookie: refresh_token=...; HttpOnly; Secure; SameSite=Strict
```

### POST /auth/oauth/{provider}

OAuth 2.0 로그인 시작 (provider: google|github|apple|microsoft)

```json
// Response 200
{
  "data": {
    "redirectUrl": "https://accounts.google.com/o/oauth2/auth?..."
  }
}
```

### POST /auth/oauth/{provider}/callback

OAuth 콜백 처리

### POST /auth/mfa/setup

TOTP MFA 설정

```json
// Response 200
{
  "data": {
    "secret": "BASE32SECRET",
    "qrCodeUrl": "otpauth://totp/Synapse:user@example.com?...",
    "qrCodeImage": "data:image/png;base64,..."
  }
}
```

### POST /auth/mfa/verify

MFA 코드 검증

```json
// Request
{ "code": "123456" }
```

### POST /auth/refresh

Access Token 갱신 (Refresh Token Cookie 사용)

### POST /auth/logout

로그아웃 (Refresh Token 무효화)

---

## 4.3 Tenant 도메인

### GET /tenant/context

현재 테넌트 정보 조회

```json
{
  "data": {
    "tenantId": "uuid",
    "name": "My Workspace",
    "plan": "pro",
    "role": "owner",
    "quotas": {
      "notes": { "used": 45, "max": -1 },
      "aiGenerations": { "used": 120, "max": 500 }
    }
  }
}
```

### POST /tenant/switch

테넌트 전환 (복수 테넌트 멤버)

### GET /tenant/usage

사용량 현황 조회

### GET /tenant/members

멤버 목록 조회 (Team 플랜 이상)

### POST /tenant/members/invite

멤버 초대

---

## 4.4 Billing 도메인

### GET /billing/plans

사용 가능한 플랜 목록

```json
{
  "data": [
    {
      "code": "pro",
      "name": "Pro",
      "price": 9.99,
      "currency": "USD",
      "interval": "month",
      "features": ["unlimited_notes", "500_ai_month", "10gb_storage"]
    }
  ]
}
```

### POST /billing/checkout

Stripe Checkout 세션 생성

```json
// Request
{ "planCode": "pro", "interval": "month" }

// Response 200
{
  "data": {
    "checkoutUrl": "https://checkout.stripe.com/..."
  }
}
```

### POST /billing/portal

Stripe Customer Portal URL 생성

### GET /billing/subscription

현재 구독 상태 조회

### POST /billing/webhooks (Internal)

Stripe Webhook 수신 (내부 전용)

---

## 4.5 GDPR 도메인

### POST /gdpr/data-export

데이터 내보내기 요청

```json
// Request
{ "format": "json" }

// Response 202
{
  "data": {
    "jobId": "uuid",
    "status": "pending",
    "estimatedMinutes": 5
  }
}
```

### GET /gdpr/data-export/{jobId}

내보내기 상태 조회

### DELETE /gdpr/account

계정 삭제 요청 (30일 유예)

---

## 4.6 Notes 도메인

### POST /notes

노트 생성

```json
// Request
{
  "title": "학습 노트 제목",
  "content": "# 내용\n\n[[다른노트]] 참조...",
  "tags": ["학습", "프로그래밍"]
}

// Response 201
{
  "data": {
    "id": "uuid",
    "title": "학습 노트 제목",
    "content": "# 내용\n\n[[다른노트]] 참조...",
    "tags": ["학습", "프로그래밍"],
    "wordCount": 25,
    "links": [{ "targetTitle": "다른노트", "targetId": "uuid" }],
    "createdAt": "2026-05-07T10:00:00Z"
  }
}
```

### GET /notes

노트 목록 조회 (페이지네이션)

### GET /notes/{id}

노트 상세 조회

### PUT /notes/{id}

노트 수정 (위키링크 자동 갱신)

### DELETE /notes/{id}

노트 삭제 (소프트 삭제)

### GET /notes/{id}/backlinks

특정 노트의 백링크 목록

```json
{
  "data": [
    {
      "noteId": "uuid",
      "title": "참조하는 노트",
      "contextSnippet": "...[[현재노트]] 관련 내용..."
    }
  ]
}
```

### GET /notes/{id}/versions

노트 버전 이력

### POST /notes/{id}/attachments

첨부파일 업로드 (Presigned URL 반환)

### GET /notes/search

키워드 검색 (Elasticsearch)

```
GET /notes/search?q=머신러닝&tags=AI&sort=relevance
```

---

## 4.7 Cards/Decks 도메인

### POST /decks

덱 생성

### GET /decks

덱 목록 조회

### PUT /decks/{id}

덱 수정

### DELETE /decks/{id}

덱 삭제

### POST /decks/{deckId}/cards

카드 생성

```json
// Request
{
  "cardType": "basic",
  "frontContent": "TCP와 UDP의 차이점은?",
  "backContent": "TCP는 연결 지향적, 신뢰성 보장...",
  "sourceNoteId": "uuid"
}
```

### GET /decks/{deckId}/cards

덱 내 카드 목록

### PUT /cards/{id}

카드 수정

### DELETE /cards/{id}

카드 삭제

### POST /cards/batch

카드 일괄 생성

```json
// Request
{
  "deckId": "uuid",
  "cards": [
    { "cardType": "basic", "frontContent": "...", "backContent": "..." },
    { "cardType": "cloze", "frontContent": "{{c1::답}}은 ...", "backContent": "" }
  ]
}
```

---

## 4.8 SRS/Reviews 도메인

### GET /reviews/queue

오늘의 복습 큐 조회

```json
{
  "data": {
    "totalDue": 25,
    "newCards": 5,
    "reviewCards": 15,
    "learningCards": 5,
    "cards": [
      {
        "cardId": "uuid",
        "cardType": "basic",
        "frontContent": "...",
        "deckName": "프로그래밍",
        "status": "review"
      }
    ]
  }
}
```

### POST /reviews/sessions

복습 세션 시작

```json
// Request
{ "deckId": "uuid" }

// Response 201
{
  "data": {
    "sessionId": "uuid",
    "totalCards": 25,
    "startedAt": "2026-05-07T10:00:00Z"
  }
}
```

### POST /reviews/sessions/{sessionId}/submit

카드 복습 결과 제출

```json
// Request
{
  "cardId": "uuid",
  "rating": 3,
  "timeSpentMs": 5000
}

// Response 200
{
  "data": {
    "nextInterval": 7,
    "newEF": 2.6,
    "nextDueDate": "2026-05-14",
    "sessionProgress": { "completed": 10, "total": 25 }
  }
}
```

### PUT /reviews/sessions/{sessionId}/complete

세션 완료

---

## 4.9 Graph 도메인

### GET /graph/data

그래프 시각화 데이터

```json
{
  "data": {
    "nodes": [
      { "id": "uuid", "title": "노트1", "linkCount": 5, "pageRank": 0.85 }
    ],
    "edges": [
      { "source": "uuid1", "target": "uuid2", "type": "wikilink" }
    ]
  }
}
```

### GET /graph/neighbors/{noteId}

특정 노트의 N-hop 이웃 조회

### GET /graph/clusters

자동 감지된 클러스터 목록

---

## 4.10 AI 도메인

### POST /ai/cards/generate

AI 카드 자동 생성

```json
// Request
{
  "noteId": "uuid",
  "cardType": "basic",
  "count": 5,
  "difficulty": "medium"
}

// Response 200
{
  "data": {
    "cards": [
      {
        "frontContent": "생성된 질문...",
        "backContent": "생성된 답변...",
        "confidence": 0.92
      }
    ],
    "usage": { "inputTokens": 500, "outputTokens": 300 }
  }
}
```

### POST /ai/search/semantic

시맨틱 검색

```json
// Request
{
  "query": "머신러닝에서 과적합을 방지하는 방법",
  "limit": 10,
  "threshold": 0.7
}

// Response 200
{
  "data": {
    "results": [
      {
        "noteId": "uuid",
        "title": "정규화 기법",
        "chunkText": "과적합을 방지하기 위해...",
        "score": 0.89
      }
    ]
  }
}
```

### POST /ai/search/hybrid

하이브리드 검색 (시맨틱 + BM25 RRF)

### POST /ai/qa

RAG 기반 Q&A (스트리밍)

```json
// Request
{
  "question": "내 노트에서 정규화 기법은 무엇이라고 설명하고 있나요?",
  "stream": true
}

// Response: Server-Sent Events
data: {"type": "chunk", "content": "노트에 따르면"}
data: {"type": "chunk", "content": " 정규화 기법은..."}
data: {"type": "sources", "noteIds": ["uuid1", "uuid2"]}
data: {"type": "done", "usage": {"inputTokens": 800, "outputTokens": 200}}
```

---

## 4.11 Stats 도메인

### GET /stats/overview

사용자 학습 통계 대시보드

```json
{
  "data": {
    "today": { "reviewed": 25, "correct": 20, "streak": 7 },
    "weekly": { "reviewed": 150, "newCards": 30 },
    "totalNotes": 89,
    "totalCards": 450,
    "retentionRate": 0.85
  }
}
```

### GET /stats/heatmap

학습 히트맵 (GitHub 스타일)

### GET /stats/retention

리텐션 커브 차트 데이터

---

## 4.12 Import/Export 도메인

### POST /import/markdown

Markdown 파일 가져오기 (Obsidian Vault)

### POST /import/anki

Anki .apkg 파일 가져오기

### POST /export/markdown

Markdown 형식 내보내기

### POST /export/anki

Anki .apkg 형식 내보내기

---

## 4.13 Admin 도메인 (관리자 전용)

### GET /admin/tenants

테넌트 목록 관리

### GET /admin/tenants/{id}/usage

특정 테넌트 사용량 상세

### PUT /admin/tenants/{id}/status

테넌트 상태 변경 (suspend/activate)

### GET /admin/stats/system

시스템 전체 통계

### GET /admin/audit-logs

감사 로그 검색

```
GET /admin/audit-logs?tenantId=uuid&action=note.create&from=2026-05-01&to=2026-05-07
```

---

## 4.14 Audit 도메인

### GET /audit/logs

현재 테넌트 감사 로그 조회 (Admin/Owner 전용)

```json
{
  "data": [
    {
      "id": "uuid",
      "action": "note.create",
      "userId": "uuid",
      "resourceType": "note",
      "resourceId": "uuid",
      "ipAddress": "1.2.3.4",
      "createdAt": "2026-05-07T10:00:00Z"
    }
  ]
}
```
