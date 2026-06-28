# 뭉(Moong) Team

대화로 감정을 자각하게 돕는 감정 케어 반려 에이전트 — 2026 AI-Champion 본선작.

> [!IMPORTANT]
> **자율 주행 개발(Agent-Driven Development) 패러다임**을 바탕으로 다중 LLM 코드 리뷰와 CI/CD 테스트 안정성을 100% 자동 순환 수렴 구조로 구축하여 유지합니다.

---

## 👥 팀 구성 (3트랙)
*   **research-eng** — 모델 파인튜닝, 정성적/정량적 평가, 대화 데이터 정제 및 실험 자동화
*   **app-eng** — FastAPI 백엔드 API, 도메인 비즈니스 설계, Flutter 하이브리드 모바일 웹앱 구현
*   **devsecops** — 자율 순환형 CI/CD 빌드 최적화, 다중 LLM 자동 코드 리뷰 인프라, 보안 크레덴셜 제어

---

## 🤝 협업 방식
*   **기본 흐름**: 이슈 생성 ➔ 브랜치 분기 ➔ 작업 ➔ PR 오픈 ➔ AI & Peer 코드 리뷰 ➔ 머지
*   **브랜치 규칙**: 
    - `main` : 상시 릴리즈 및 서비스 배포 가능한 절대 안정 브랜치
    - `develop` : 기능 통합 브랜치
    - `feat/이슈번호-설명` : 신규 기능 개발 단위
    - `fix/이슈번호-설명` : 버그 및 오류 수정 단위
*   **커밋 컨벤션**: `type: 내용` 규칙 준수 (`feat`, `fix`, `docs`, `refactor`, `test`, `chore`)
*   **에이전트 협업**: `/handoff` 명세 및 `claude.md` / `agent.md` 기반 맥락 전파 체계

---

## 📖 핵심 인프라 & 가이드북

> [!TIP]
> 아래 문서를 통해 Team-Moong의 자동화 체계 및 AI 에이전트 오케스트레이션 방식을 상세하게 파악할 수 있습니다.

*   🤖 [**AI 코드 리뷰 & CI/CD 인프라 가이드북**](./AI_CODE_REVIEW_INFRA.md)
    *   API Key 기반 다중 LLM(Solar/EXAONE) 동적 라우팅 및 OIDCBedrock 수준의 정량 평가(Confidence Gate) 구현 세부 정보
*   🧠 [**AI 에이전트 자율 주행 협업 지침서 (ADD)**](./AGENT_CONTEXT_GUIDE.md)
    *   GitHub CLI와 MCP를 활용한 에이전트의 PR 자가 치유(Self-Healing) 기법 및 Flutter 비동기 테스트 타이머 누수 제어 가이드
*   📋 [**개발 프로세스 및 에이전트 이슈 트래킹 가이드**](./ISSUE_TEMPLATE_GUIDE.md)
    *   `dev` 통합 지향 브랜치 전략(Mermaid 흐름도) 및 에이전트와 개발자가 준수해야 할 표준 이슈/PR 본문 템플릿 규격 제공


