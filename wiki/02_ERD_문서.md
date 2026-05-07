# 2. ERD 문서

> **프로젝트명**: Synapse — 통합 학습-지식 그래프 SaaS
> **버전**: v1.0
> **작성일**: 2026-05-07
> **기술 스택**: Spring Boot 4, Flutter 3.x, FastAPI, PostgreSQL 16, Redis, Elasticsearch, Kafka, K8s

---

## 2.1 데이터베이스 설계 원칙

### 멀티테넌시 전략

- **모델**: Pool (단일 DB, 공유 스키마)
- **격리**: Row Level Security (RLS) + 애플리케이션 레벨 tenant_id 강제 필터
- **인덱스 규칙**: 모든 인덱스는 `tenant_id`를 prefix로 포함

### 공통 컬럼 규약

| 컬럼 | 타입 | 설명 |
|------|------|------|
| id | UUID (v7) | PK, 시간 순서 보장 |
| tenant_id | UUID | FK → tenants.id, NOT NULL |
| created_at | TIMESTAMPTZ | 생성 시각, DEFAULT now() |
| updated_at | TIMESTAMPTZ | 수정 시각, 트리거 자동 갱신 |
| deleted_at | TIMESTAMPTZ | 소프트 삭제, NULL = 활성 |

---

## 2.2 ERD 다이어그램

### 2.2.1 테넌시/빌링 도메인

```mermaid
erDiagram
    tenants {
        uuid id PK
        varchar(100) name
        varchar(20) plan "free|pro|team|enterprise"
        varchar(20) status "active|suspended|deleted"
        jsonb settings
        timestamptz created_at
        timestamptz updated_at
    }

    tenant_members {
        uuid id PK
        uuid tenant_id FK
        uuid user_id FK
        varchar(20) role "owner|admin|member"
        timestamptz joined_at
        timestamptz created_at
    }

    plan_quotas {
        uuid id PK
        varchar(20) plan_code PK
        varchar(50) resource_type "notes|cards|ai_generations|storage_bytes"
        bigint max_value
        varchar(20) period "monthly|total"
    }

    subscriptions {
        uuid id PK
        uuid tenant_id FK
        varchar(20) plan_code
        varchar(50) stripe_subscription_id
        varchar(20) status "active|past_due|canceled|trialing"
        timestamptz current_period_start
        timestamptz current_period_end
        timestamptz canceled_at
        timestamptz created_at
    }

    usage_counters {
        uuid id PK
        uuid tenant_id FK
        varchar(50) resource_type
        bigint current_value
        bigint max_value
        varchar(20) period
        date period_start
        timestamptz updated_at
    }

    tenants ||--o{ tenant_members : "has"
    tenants ||--o| subscriptions : "subscribes"
    tenants ||--o{ usage_counters : "tracks"
    plan_quotas }o--|| subscriptions : "defines limits"
```

### 2.2.2 인증/사용자 도메인

```mermaid
erDiagram
    users {
        uuid id PK
        varchar(255) email UK
        varchar(100) display_name
        varchar(512) avatar_url
        varchar(20) status "active|suspended|deleted"
        varchar(10) locale "ko|en|ja"
        timestamptz email_verified_at
        timestamptz last_login_at
        timestamptz created_at
        timestamptz updated_at
    }

    oauth_identities {
        uuid id PK
        uuid user_id FK
        varchar(20) provider "google|github|apple|microsoft"
        varchar(255) provider_user_id
        varchar(512) access_token_enc
        varchar(512) refresh_token_enc
        timestamptz expires_at
        timestamptz created_at
    }

    mfa_credentials {
        uuid id PK
        uuid user_id FK
        varchar(20) type "totp"
        varchar(512) secret_enc
        boolean is_active
        timestamptz verified_at
        timestamptz created_at
    }

    refresh_tokens {
        uuid id PK
        uuid user_id FK
        varchar(512) token_hash
        varchar(50) device_fingerprint
        inet ip_address
        timestamptz expires_at
        timestamptz created_at
    }

    user_settings {
        uuid id PK
        uuid user_id FK
        varchar(20) theme "light|dark|system"
        varchar(10) language "ko|en|ja"
        jsonb notification_prefs
        jsonb review_prefs
        timestamptz updated_at
    }

    data_export_jobs {
        uuid id PK
        uuid user_id FK
        uuid tenant_id FK
        varchar(20) status "pending|processing|completed|failed"
        varchar(20) format "json|markdown|csv"
        varchar(512) download_url
        timestamptz expires_at
        timestamptz completed_at
        timestamptz created_at
    }

    users ||--o{ oauth_identities : "authenticates via"
    users ||--o{ mfa_credentials : "secures with"
    users ||--o{ refresh_tokens : "sessions"
    users ||--|| user_settings : "configures"
    users ||--o{ data_export_jobs : "exports"
    tenant_members }o--|| users : "belongs to"
```

### 2.2.3 노트 도메인

```mermaid
erDiagram
    notes {
        uuid id PK
        uuid tenant_id FK
        uuid user_id FK
        varchar(500) title
        text content_md
        text content_plain "검색용 평문"
        varchar(20) status "active|archived|trashed"
        int word_count
        jsonb metadata
        timestamptz created_at
        timestamptz updated_at
        timestamptz deleted_at
    }

    note_versions {
        uuid id PK
        uuid note_id FK
        uuid tenant_id FK
        int version_number
        text content_md
        varchar(500) change_summary
        timestamptz created_at
    }

    note_links {
        uuid id PK
        uuid tenant_id FK
        uuid source_note_id FK
        uuid target_note_id FK
        varchar(50) link_type "wikilink|reference|embed"
        varchar(500) context_snippet
        timestamptz created_at
    }

    note_chunks {
        uuid id PK
        uuid note_id FK
        uuid tenant_id FK
        int chunk_index
        text chunk_text
        vector(1536) embedding
        int token_count
        timestamptz created_at
    }

    tags {
        uuid id PK
        uuid tenant_id FK
        varchar(100) name
        varchar(7) color
        timestamptz created_at
    }

    note_tags {
        uuid note_id FK
        uuid tag_id FK
        timestamptz created_at
    }

    attachments {
        uuid id PK
        uuid note_id FK
        uuid tenant_id FK
        varchar(255) filename
        varchar(100) content_type
        bigint size_bytes
        varchar(512) s3_key
        timestamptz created_at
    }

    bookmarks {
        uuid id PK
        uuid user_id FK
        uuid tenant_id FK
        uuid note_id FK
        timestamptz created_at
    }

    notes ||--o{ note_versions : "versioned"
    notes ||--o{ note_links : "links from"
    notes ||--o{ note_links : "links to"
    notes ||--o{ note_chunks : "chunked"
    notes ||--o{ note_tags : "tagged"
    notes ||--o{ attachments : "attaches"
    notes ||--o{ bookmarks : "bookmarked"
    tags ||--o{ note_tags : "applied to"
```

### 2.2.4 카드/SRS 도메인

```mermaid
erDiagram
    card_decks {
        uuid id PK
        uuid tenant_id FK
        uuid user_id FK
        varchar(200) name
        text description
        varchar(7) color
        varchar(20) status "active|archived"
        int card_count
        int new_cards_per_day "기본 20"
        timestamptz created_at
        timestamptz updated_at
    }

    cards {
        uuid id PK
        uuid tenant_id FK
        uuid user_id FK
        uuid deck_id FK
        uuid source_note_id FK "nullable"
        uuid source_chunk_id FK "nullable"
        varchar(20) card_type "basic|cloze|reverse"
        text front_content
        text back_content
        varchar(20) status "new|learning|review|suspended"
        float easiness_factor "SM-2 EF, 기본 2.5"
        int interval_days
        int repetitions
        int lapses
        timestamptz due_date
        timestamptz last_reviewed_at
        timestamptz created_at
        timestamptz updated_at
    }

    card_reviews {
        uuid id PK
        uuid tenant_id FK
        uuid card_id FK
        uuid session_id FK
        int rating "1=again, 2=hard, 3=good, 4=easy"
        int time_spent_ms
        float prev_ef
        float new_ef
        int prev_interval
        int new_interval
        timestamptz reviewed_at
    }

    review_sessions {
        uuid id PK
        uuid tenant_id FK
        uuid user_id FK
        uuid deck_id FK
        int total_cards
        int completed_cards
        int correct_count
        int again_count
        bigint total_time_ms
        varchar(20) status "in_progress|completed|abandoned"
        timestamptz started_at
        timestamptz ended_at
    }

    card_decks ||--o{ cards : "contains"
    cards ||--o{ card_reviews : "reviewed"
    cards }o--o| notes : "sourced from"
    review_sessions ||--o{ card_reviews : "includes"
    card_decks ||--o{ review_sessions : "session for"
```

### 2.2.5 AI/RAG 도메인

```mermaid
erDiagram
    semantic_cache {
        uuid id PK
        uuid tenant_id FK
        varchar(64) query_hash "SHA-256"
        text query_text
        vector(1536) query_embedding
        jsonb result_data
        int hit_count
        timestamptz expires_at
        timestamptz created_at
    }

    llm_usage_logs {
        uuid id PK
        uuid tenant_id FK
        uuid user_id FK
        varchar(50) model "gpt-4o|gpt-4o-mini|text-embedding-3-small"
        varchar(50) operation "card_generate|semantic_search|qa|summarize"
        int input_tokens
        int output_tokens
        int total_tokens
        numeric(10_4) cost_usd
        int latency_ms
        varchar(20) status "success|error|timeout"
        jsonb metadata
        timestamptz created_at
    }

    llm_feedback {
        uuid id PK
        uuid tenant_id FK
        uuid user_id FK
        uuid usage_log_id FK
        int rating "1-5"
        text comment
        varchar(20) feedback_type "quality|relevance|accuracy"
        timestamptz created_at
    }

    llm_usage_logs ||--o{ llm_feedback : "receives"
```

### 2.2.6 감사 도메인

```mermaid
erDiagram
    audit_logs {
        uuid id PK
        uuid tenant_id FK
        uuid user_id FK
        varchar(50) action "note.create|card.review|auth.login|billing.subscribe"
        varchar(50) resource_type "note|card|deck|user|subscription"
        uuid resource_id
        jsonb old_value
        jsonb new_value
        inet ip_address
        varchar(512) user_agent
        timestamptz created_at
    }

    processed_events {
        uuid id PK
        varchar(100) event_id UK "Kafka 메시지 ID"
        varchar(50) topic
        varchar(20) status "processed|failed|skipped"
        int retry_count
        timestamptz processed_at
        timestamptz created_at
    }
```

---

## 2.3 인덱스 설계 규칙

### 네이밍 컨벤션

```
idx_{table}_{columns}
uq_{table}_{columns}
```

### 핵심 인덱스

| 테이블 | 인덱스 | 컬럼 | 타입 |
|--------|--------|------|------|
| notes | idx_notes_tenant_user | (tenant_id, user_id, deleted_at) | B-tree |
| notes | idx_notes_tenant_status | (tenant_id, status, updated_at DESC) | B-tree |
| note_links | idx_note_links_target | (tenant_id, target_note_id) | B-tree |
| note_chunks | idx_note_chunks_embedding | (embedding) | IVFFlat (lists=100) |
| cards | idx_cards_tenant_due | (tenant_id, user_id, status, due_date) | B-tree |
| cards | idx_cards_deck | (tenant_id, deck_id, status) | B-tree |
| card_reviews | idx_card_reviews_card | (tenant_id, card_id, reviewed_at DESC) | B-tree |
| audit_logs | idx_audit_tenant_time | (tenant_id, created_at DESC) | B-tree |
| semantic_cache | idx_semantic_cache_hash | (tenant_id, query_hash) | B-tree |
| semantic_cache | idx_semantic_cache_vec | (query_embedding) | HNSW (m=16, ef=64) |

### 파티셔닝 전략

```sql
-- audit_logs: 월별 파티셔닝
CREATE TABLE audit_logs (
    ...
) PARTITION BY RANGE (created_at);

CREATE TABLE audit_logs_2026_01 PARTITION OF audit_logs
    FOR VALUES FROM ('2026-01-01') TO ('2026-02-01');

-- card_reviews: 월별 파티셔닝
CREATE TABLE card_reviews (
    ...
) PARTITION BY RANGE (reviewed_at);
```

---

## 2.4 RLS 정책 예시

### 기본 RLS 정책

```sql
-- notes 테이블 RLS
ALTER TABLE notes ENABLE ROW LEVEL SECURITY;

CREATE POLICY notes_tenant_isolation ON notes
    USING (tenant_id = current_setting('app.current_tenant_id')::uuid);

CREATE POLICY notes_user_access ON notes
    FOR ALL
    USING (
        tenant_id = current_setting('app.current_tenant_id')::uuid
        AND (
            user_id = current_setting('app.current_user_id')::uuid
            OR current_setting('app.current_role') = 'admin'
        )
    );
```

### 테넌트 컨텍스트 설정

```sql
-- 요청마다 Gateway에서 설정
SET LOCAL app.current_tenant_id = 'tenant-uuid-here';
SET LOCAL app.current_user_id = 'user-uuid-here';
SET LOCAL app.current_role = 'member';
```

---

## 2.5 데이터 흐름 요약

```
노트 작성 → notes INSERT
         → note_versions INSERT (비동기)
         → note_links UPSERT (위키링크 파싱)
         → note_chunks INSERT (청킹 + 임베딩, 비동기)
         → Elasticsearch 인덱싱 (Kafka)

카드 생성 → cards INSERT
         → card_decks.card_count UPDATE

복습 제출 → card_reviews INSERT
         → cards UPDATE (SM-2 계산)
         → review_sessions UPDATE
         → usage_counters INCREMENT (Kafka)
```

---

## 2.6 마이그레이션 전략

- **도구**: Flyway 10.x
- **네이밍**: `V{version}__{description}.sql`
- **규칙**:
  - DDL과 DML 분리
  - 모든 마이그레이션 되돌리기 가능하도록 작성
  - 대용량 테이블 변경 시 `CREATE INDEX CONCURRENTLY` 사용
  - RLS 정책은 별도 마이그레이션 파일로 관리
