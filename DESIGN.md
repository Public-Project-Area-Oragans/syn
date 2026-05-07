# Design System -- Synapse

## Product Context
- **What this is:** PKM(노트) + SRS(카드 복습) + AI를 통합한 학습 SaaS 플랫폼
- **Who it's for:** 개발자, 대학원생, 자격증 준비자 (기술/학술 지식의 장기 기억이 필요한 사용자)
- **Space/industry:** 학습 도구 (PKM + SRS 교차 영역)
- **Project type:** Flutter Web SaaS 앱 + 랜딩 페이지
- **Memorable thing:** 노트를 쓰면 AI가 카드를 만들어주고, 복습하면 노트가 다시 살아난다

## Aesthetic Direction
- **Direction:** Warm Intellectual
- **Decoration level:** Intentional (미세한 종이 텍스처, 전체적으로 깨끗)
- **Mood:** 서점과 도서관의 느낌. 차분하지만 따뜻하고, 지적이지만 어렵지 않음. 학습이 즐거운 공간.
- **Competitors:** Obsidian(보라/다크), Anki(무미건조), RemNote(차가운 미니멀), Notion(비즈니스)
- **Differentiation:** 경쟁사는 모두 "도구"처럼 보인다. Synapse는 "학습 경험"처럼 느껴져야 한다.

## Sibling Projects (의도적 차별화)
| Project | Accent | Aesthetic | Display Font |
|---------|--------|-----------|-------------|
| StockPilot | Deep Green #0B7A3E | Operational | General Sans |
| DevPath | Cobalt Blue #0F6FEC | Industrial | Satoshi |
| StudyMate | Ink Blue #1B4965 | Structured Academic | Pretendard |
| **Synapse** | **Warm Amber #D97706** | **Warm Intellectual** | **Fraunces** |

## Typography
- **Display/Hero:** Fraunces (variable, optical sizing) -- 세리프. 지적이고 따뜻한 느낌. PKM 도구에서 세리프를 쓰는 곳은 거의 없어서 차별점.
- **Body:** Plus Jakarta Sans (400, 500, 600, 700) -- 따뜻한 geometric sans. 읽기 편하고 개성 있음.
- **UI/Labels:** Plus Jakarta Sans 500 (body와 동일, weight로 구분)
- **Data/Tables:** Geist Mono (tabular-nums 지원)
- **Code:** Geist Mono
- **Loading:** Google Fonts CDN (Flutter Web에서 google_fonts 패키지 사용)
- **Scale:**
  - Display XL: 48px / 1.1 (Fraunces)
  - Display: 36px / 1.2 (Fraunces)
  - H1: 30px / 1.3 (Fraunces)
  - H2: 24px / 1.35 (Plus Jakarta Sans 600)
  - H3: 20px / 1.4 (Plus Jakarta Sans 600)
  - Body: 16px / 1.6 (Plus Jakarta Sans 400)
  - Body Small: 14px / 1.5 (Plus Jakarta Sans 400)
  - Caption: 12px / 1.4 (Plus Jakarta Sans 500)
  - Code: 14px / 1.5 (Geist Mono)

## Color
- **Approach:** Restrained (1 액센트 + neutrals, 색상은 드물고 의미 있게)
- **Primary Accent:** Warm Amber #D97706 -- 따뜻함, 지식, 불빛의 연상. CTA, 선택 상태, 진행률에 사용.
- **Primary Hover:** #B45309
- **Primary Light:** #FEF3C7 (배경 하이라이트, 알림 배지)
- **Secondary:** Muted Teal #0D9488 -- 성공/완료, 그래프 하이라이트, SRS 정답 표시에 사용.
- **Neutrals (Warm Stone 계열):**
  - 50: #FAFAF9
  - 100: #F5F5F4
  - 200: #E7E5E4
  - 300: #D6D3D1
  - 400: #A8A29E
  - 500: #78716C
  - 600: #57534E
  - 700: #44403C
  - 800: #292524
  - 900: #1C1917
  - 950: #0C0A09
- **Semantic:**
  - Success: #16A34A (SRS 정답, 복습 완료)
  - Warning: #F59E0B (한도 80% 도달, 주의)
  - Error: #DC2626 (오류, SRS 오답, 삭제)
  - Info: #0EA5E9 (안내, 팁, 새 기능)
- **Dark mode strategy:**
  - Background: Stone-900 #1C1917 (차가운 순흑 대신 따뜻한 다크)
  - Surface: Stone-800 #292524
  - Elevated: Stone-700 #44403C
  - 액센트 채도 10-15% 감소
  - 텍스트: Stone-100 #F5F5F4

## Spacing
- **Base unit:** 8px
- **Density:** Comfortable (학습은 여유로운 공간이 필요)
- **Scale:**
  - 2xs: 2px
  - xs: 4px
  - sm: 8px
  - md: 16px
  - lg: 24px
  - xl: 32px
  - 2xl: 48px
  - 3xl: 64px
- **Content padding:** md(16px) mobile, lg(24px) tablet, xl(32px) desktop

## Layout
- **Approach:** Hybrid (앱은 grid-disciplined, 랜딩은 editorial)
- **Grid:** 4 columns (mobile) / 8 columns (tablet) / 12 columns (desktop)
- **Max content width:** 1280px
- **Sidebar width:** 240px (collapsed: 56px)
- **Border radius:**
  - sm: 4px (inputs, badges)
  - md: 8px (cards, buttons)
  - lg: 12px (panels, dialogs)
  - xl: 16px (hero sections)
  - full: 9999px (avatars, pills)
- **Breakpoints:**
  - Mobile: < 640px
  - Tablet: 640-1024px
  - Desktop: > 1024px

## Motion
- **Approach:** Intentional (의미 있는 전환만, 장식적 모션은 없음)
- **Easing:**
  - Enter: ease-out (요소 등장)
  - Exit: ease-in (요소 퇴장)
  - Move: ease-in-out (위치 이동)
  - Spring: cubic-bezier(0.34, 1.56, 0.64, 1) (카드 뒤집기, 축하)
- **Duration:**
  - Micro: 50-100ms (호버, 포커스)
  - Short: 150-250ms (토글, 드롭다운)
  - Medium: 250-400ms (패널 전환, 카드 뒤집기)
  - Long: 400-700ms (페이지 전환, 그래프 레이아웃)
- **Key animations:**
  - SRS 카드 뒤집기: Y축 3D 회전, 300ms, spring easing
  - 복습 완료 축하: 파티클 효과, 600ms
  - 그래프 노드 등장: scale 0 -> 1, 200ms, staggered
  - 사이드바 토글: width 전환, 200ms, ease-in-out

## Component Patterns
- **Buttons:** md border-radius, Primary(Amber filled), Secondary(Stone outlined), Ghost(text only)
- **Cards:** md border-radius, Stone-100 배경, subtle shadow(0 1px 3px rgba(0,0,0,0.1))
- **Inputs:** sm border-radius, Stone-300 border, Amber focus ring
- **SRS Card:** lg border-radius, elevated shadow, 가운데 정렬, 큰 텍스트
- **Graph Node:** 원형, Amber fill(활성), Stone-400 fill(비활성), Teal fill(정답률 높음)

## Decisions Log
| Date | Decision | Rationale |
|------|----------|-----------|
| 2026-05-07 | Warm Amber #D97706 액센트 | StockPilot(녹색)/DevPath(파란색)/StudyMate(잉크블루)와 의도적 차별화. 학습 도구에 따뜻한 색상이 부재. |
| 2026-05-07 | Fraunces 세리프 디스플레이 폰트 | PKM 도구에서 세리프를 쓰는 곳이 거의 없음. "서점/도서관" 느낌으로 학습 공간 차별화. |
| 2026-05-07 | Plus Jakarta Sans 본문 폰트 | Inter/Roboto를 피하면서 따뜻한 geometric sans. DM Sans보다 개성 있음. |
| 2026-05-07 | Warm Stone neutrals | Cool gray 대신 warm gray. "차가운 도구"가 아닌 "따뜻한 학습 공간" 느낌. |
| 2026-05-07 | Comfortable spacing density | 학습은 정보 밀도보다 여유로운 공간이 중요. WMS와 반대 방향. |
