# 뭉(Moong) Team

대화로 감정을 자각하게 돕는 감정 케어 반려 에이전트 — 2026 AI-Champion 본선작.

## 팀 (3트랙)
- **research-eng** — 모델·평가·데이터·실험
- **app-eng** — API·도메인·웹앱
- **devsecops** — CI/CD·배포·보안

## 협업 방식

### 기본 흐름
이슈 생성 → 브랜치 분기 → 작업 → PR 오픈 → 코드 리뷰 → 머지

### 브랜치 규칙
- `main` — 항상 배포 가능한 상태 유지
- `develop` — 통합 브랜치
- `feat/이슈번호-설명` — 기능 개발
- `fix/이슈번호-설명` — 버그 수정

### 이슈 & PR
- 모든 작업은 **이슈 먼저** 등록 후 진행
- PR은 이슈와 연결 (`Closes #이슈번호`)
- PR 머지 전 최소 **1명 리뷰 승인** 필수
- 리뷰어는 48시간 내 응답 권고

### 커밋 컨벤션
`type: 내용` 형식 — `feat` / `fix` / `docs` / `refactor` / `test` / `chore`

### 에이전트 협업
- `/handoff` 및 `claude.md`&`agent.md` 기반 맥락 공유 협업
- 🤖 [**Team-Moong AI 코드 리뷰 & CI/CD 인프라 상세 가이드 보기**](./AI_CODE_REVIEW_INFRA.md)
- 🧠 [**Team-Moong AI 에이전트 협업 및 오케스트레이션 지침서 보기**](./AGENT_CONTEXT_GUIDE.md)
