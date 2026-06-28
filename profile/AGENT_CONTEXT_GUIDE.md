# 🧠 AI Agent Context & Orchestration Guide (가명화 버전)

이 문서는 Team-Moong에 새롭게 투입된 코딩 에이전트(Coding Agent, e.g., Claude, Gemini 등)가 현재 저장소의 PR 상태를 읽고, 코드 리뷰 피드백을 적용하며, CI/CD 안정성을 재귀적으로 최적화할 때 활용하는 **자율 주행 개발(Agent-Driven Development) 컨텍스트 지침서**입니다.

---

## 📅 1. 실사례 기반 최근 변경 이력 (Git Log Context)

개발 및 테스트 트랙에서 최근 통과된 주요 패치 및 이슈 해결 이력입니다. 후속 작업 시 이 히스토리 흐름을 유지하며 리팩토링을 설계하십시오.

```bash
# moong-core-app 저장소 최근 커밋 로그 (가명화)
de63fca feat: upgrade AI reviewer script to use Pull Request Reviews API and target PR #1
1582775 style: AI 코드 리뷰어 하단 크레딧 서명 문구 제거
817f00c Optimize AI code reviewer with openrabbit-style rich walkthrough formatting
6d19281 ci: Flutter 포맷 불일치로 인한 빌드 중단 우회 조치 (|| true)
259ef20 ci: Ruff 및 Flutter 포맷 검사 명령어 최신 버전 호환성 수정
75df294 ci: AI 리뷰어 폴백 구현 및 openrabbit 로컬 이식 검증 수정
ff882bb ci: GitHub Actions 파이프라인 추가 (Flutter, Python, openrabbit)
e8085b8 refactor: 모델별 공식 docs 기반 PromptManager 모듈화 도입
```

---

## 🛠 2. AI 에이전트의 PR 피드백 재귀적 수용 방법

PR에 AI 코드 리뷰어의 피드백이 등록되면, 에이전트는 아래 절차에 따라 자동으로 해당 코드를 수정하고 검증해야 합니다.

### 1단계: PR 인라인 코멘트 데이터 크롤링
에이전트가 직접 `gh` 툴을 사용해 등록된 피드백의 `file` 경로, `line` 번호, `suggestion` 코드 블록을 추출합니다.
```bash
# REST API로 특정 PR의 인라인 코멘트 JSON 가져오기
gh api repos/Team-Moong/moong-core-app/pulls/1/comments -q ".[] | {path: .path, line: .line, body: .body}"
```

### 2단계: 로컬 코드 패치 (Patch)
에이전트는 검출된 버그나 비효율성을 로컬 워크스페이스에서 수정합니다.
* **실사례 (Korean Bold Markdown Concatenation Bug)**:
  - *현상*: LLM 응답을 스트리밍할 때 한글 볼드 마커가 연속되어 `**ㅇㅇ****ㅇㅇㅇ****` 형태로 깨지는 이슈 발생.
  - *해결*: [moong-core-app]의 `backend/app/utils/text_utils.py` 내에 아래와 같이 Regex 체인을 구현하여 스트리핑 및 중복 개행을 병합함.
    ```python
    import re

    def strip_markdown(text: str) -> str:
        if not text:
            return ""
        # 1. 볼드 및 이탤릭 마커 제거
        text = re.sub(r"\*\*([^*]+)\*\*", r"\1", text)
        text = re.sub(r"\*([^*]+)\*", r"\1", text)
        # 2. 연속된 중복 마커 및 공백 정리
        text = re.sub(r"\*{2,}", "", text)
        return text.strip()
    ```

### 3단계: 로컬 품질 검증
수정된 코드가 빌드나 포맷에 영향을 주지 않는지 반드시 로컬에서 자동화된 체크 커맨드를 돌린 후 푸시해야 합니다.
```bash
# Python 백엔드 포맷 및 린트 검증 (Ruff)
ruff format --check .
ruff check .

# Flutter 프론트엔드 포맷 검증
dart format --set-exit-if-changed .
```

---

## 🤝 3. 병렬 에이전트 오케스트레이션 (Parallel Agent Orchestration)

작업 규모가 크거나 여러 리포지토리의 싱크가 필요할 때, 메인 에이전트는 하위 서브에이전트(Subagents)들을 병렬로 가동하여 작업을 분배합니다.

```mermaid
sequenceDiagram
    autonumber
    participant M as Main Agent (Orchestrator)
    participant S1 as Subagent-App (App/Backend)
    participant S2 as Subagent-RAG (RAG/Data)
    participant GH as GitHub Repo

    M->>S1: "moong-core-app의 프론트엔드 린트 교정 요청"
    M->>S2: "moong-wellness-rag의 데이터 셋 정규화 스크립트 작성 요청"
    par S1 to GH
        S1->>S1: 로컬 'dart format' 검증
        S1->>GH: git push origin feat/format-fix
    and S2 to GH
        S2->>S2: 로컬 'pytest' 검증
        S2->>GH: git push origin feat/dataset-normalize
    end
    Note over GH: GitHub Actions 트리거<br/>AI Code Reviewer 자동 리뷰 시작
    GH-->>M: PR Reviews 완료 알림 (Success/Fail)
    M->>M: 피드백 재귀 분석 후 재분배
```

### 📋 서브에이전트 위임 가이드라인
* **독립된 워크스페이스(Branch mode)**: 서브에이전트를 생성할 때는 메인 브랜치의 충돌을 막기 위해 반드시 `branch` 모드로 가동할 것을 권장합니다.
* **단일 책임 원칙**: `Subagent-App`은 컴포넌트 린팅과 포맷 오류 해결에 집중하고, `Subagent-RAG`는 RAG 데이터셋 유효성 검증 테스트 작성에 전념하도록 역할을 완전히 격리하십시오.

---

## ⚡ 4. Flutter 테스트 환경 비동기 타이머 누수 및 위젯 테스트 가이드

Flutter 위젯 테스트 실행 시 비동기 타이머(Timer)나 애니메이션이 정리(dispose)되지 않아 `A Timer is still pending even after the widget tree was disposed.` 에러가 발생하는 경우가 많습니다. 에이전트는 이를 방지하기 위해 다음 패턴을 준수해야 합니다.

### 1) 비동기 API 통신(Dio 등) 우회 (Mock 주입)
위젯의 `initState` 단계에서 API 통신(`DioClient` 호출 등)이 발생하는 경우, 테스트 환경에서도 실제 비동기 처리가 시작되어 누수를 발생시킵니다.
* **해결 패턴**: 위젯 생성자에 선택적 리스트(e.g., `items`, `wellnessMissions`) 파라미터를 추가하고, 테스트 빌드 시에는 빈 배열(`[]`) 또는 mock 리스트를 명시적으로 주입하여 네트워크 조회를 건너뛰게 설계합니다.
  ```dart
  // 위젯 구현부
  class InventoryScreen extends StatefulWidget {
    final List<InventoryItem>? items; // 테스트 주입용
    const InventoryScreen({super.key, this.items});
    ...
  }

  // initState 분기 처리
  @override
  void initState() {
    super.initState();
    if (widget.items != null) {
      _items = widget.items!;
      _loading = false;
    } else {
      _fetchItems(); // 실제 Dio 네트워크 호출
    }
  }
  ```

### 2) SnackBar/애니메이션 타이머 소비 (`pumpAndSettle`)
스낵바(`SnackBar`) 노출 등 애니메이션을 동반하는 위젯 인터랙션을 테스트한 후에는, 애니메이션 타이머가 만료되기 전에 테스트가 종료되어 에러가 발생합니다.
* **해결 패턴**: 스낵바를 띄우는 인터랙션(`tester.tap(...)` 등) 직후에 반드시 `await tester.pumpAndSettle();`을 호출하여 모든 애니메이션과 대기 타이머를 끝까지 대기(drain)시킵니다.
  ```dart
  testWidgets('준비중 탭 -> 스낵바 노출', (tester) async {
    await tester.pumpWidget(const MyWidget());
    await tester.tap(find.text('준비중'));
    await tester.pump();
    expect(find.text('아직 준비중이에요'), findsOneWidget);
    
    // 필수: 스낵바 페이드아웃 및 닫힘 타이머가 끝날 때까지 대기
    await tester.pumpAndSettle();
  });
  ```

### 3) `fakeAsync`를 활용한 가상 시간(Virtual Time) 흐름 제어
반복 주기적인 타이머(`Timer.periodic`)나 수 초 이상 지속되는 복잡한 지연(Delay)이 동반되는 위젯을 검증할 때는 `fakeAsync` 패키지를 사용하여 실시간 대기 없이 비동기 제어를 완료해야 타이머 누수를 원천 차단할 수 있습니다.
* **해결 패턴**: `fakeAsync` 클로저 내에서 테스트 코드를 래핑하고 `async.elapse()`를 통해 가상 시간을 강제로 흐르게 만들어 대기 중인 콜백을 동기적으로 소진합니다.
  ```dart
  import 'package:flutter_test/flutter_test.dart';
  import 'package:fake_async/fake_async.dart';

  testWidgets('Timer.periodic 타이머 누수 fakeAsync 해결 패턴', (WidgetTester tester) async {
    fakeAsync((async) {
      // 1. 가상 비동기 스코프 내에서 위젯 마운트
      tester.pumpWidget(const TimerStatusWidget());
      
      // 2. 가상 시간 10초 흘려보내기
      async.elapse(const Duration(seconds: 10));
      tester.pump(); // 상태 변경에 따른 프레임 갱신
      
      // 3. 기대 상태 단언(Assert)
      expect(find.text('진행율: 100%'), findsOneWidget);
      
      // 4. 리소스 정리 검증을 위해 위젯 언마운트 및 남은 마이크로태스크 소진
      tester.pumpWidget(const SizedBox.shrink());
      async.flushMicrotasks();
    });
  });
  ```

### 4) `Timer.periodic` 및 `Stream.periodic` 명시적 취소(Cancel) 보장
비동기 루프가 위젯 소멸(`dispose`) 시에도 해제되지 않으면 백그라운드 누수가 발생합니다. State 클래스의 `dispose` 생명주기 메서드에서 타이머 인스턴스를 확실히 취소하고 null 처리하십시오.
  ```dart
  Timer? _pollingTimer;

  @override
  void dispose() {
    _pollingTimer?.cancel();
    _pollingTimer = null;
    super.dispose();
  }
  ```

---

## 🧪 5. 실전 ADD/TDD 루프 상세: `strip_markdown` 개발 사례 (Red-Green-Refactor)

Team-Moong 프로젝트의 모든 비즈니스 로직 및 파싱 유틸리티 개선은 테스트 주도 개발(TDD) 흐름을 따릅니다. 아래는 LLM 응답 내 한글 볼드 마커 깨짐 문제를 해결하기 위한 **`strip_markdown` 함수 구현의 3단계 TDD 실전 절차**입니다.

### 🔄 ADD/TDD 전체 개발 루프 (TDD Cycle)
```mermaid
graph TD
    A["1. RED: 실패하는 테스트 케이스 작성"] --> B["2. GREEN: 최소한의 코드로 테스트 통과"]
    B --> C["3. REFACTOR: 성능 최적화 & 구조 개선"]
    C --> D["4. VERIFY: 전체 테스트 슈트 실행 및 린트 검증"]
    D -->|통과| E["5. GIT PUSH: 원격 저장소 제출"]
    D -->|실패| B
```

### 1단계: RED (실패하는 테스트 설계)
구현하려는 함수의 스펙을 정의하고 극단적인 에지 케이스(Edge Cases)를 포함하는 테스트 코드를 먼저 작성합니다. 이 시점에는 실제 함수가 없거나 오동작하여 테스트가 실패(Red)해야 합니다.

```python
# File: backend/app/tests/test_text_utils.py
import pytest
from backend.app.utils.text_utils import strip_markdown

def test_strip_markdown_null_and_empty():
    """Null값 및 빈 문자열 입력에 대한 안전성 검증"""
    assert strip_markdown("") == ""
    assert strip_markdown(None) == ""

def test_strip_markdown_basic_formats():
    """기본 마크다운 볼드/이탤릭 마커 제거 기능 검증"""
    assert strip_markdown("안녕하세요 **Team-Moong**입니다.") == "안녕하세요 Team-Moong입니다."
    assert strip_markdown("이것은 *이탤릭* 텍스트입니다.") == "이것은 이탤릭 텍스트입니다."

def test_strip_markdown_korean_concatenation_bugs():
    """한글 스트리밍 시 발생하는 연속된 볼드 마커 깨짐 에지 케이스 검증"""
    # 1. 연속 볼드 마커 병합
    assert strip_markdown("**ㅇㅇ****ㅇㅇㅇ**") == "ㅇㅇㅇㅇㅇ"
    # 2. 후행 및 비정상적인 다중 별표 기호
    assert strip_markdown("**ㅇㅇ****ㅇㅇㅇ****") == "ㅇㅇㅇㅇㅇ"
    assert strip_markdown("별표 기호가 많은 **테스트*** 문장") == "별표 기호가 많은 테스트 문장"
```

### 2단계: GREEN (테스트 통과를 위한 최소 구현)
테스트를 빠르게 통과할 수 있도록 단순하고 직관적인 코드를 우선 작성합니다. 예외가 나지 않고 모든 테스트 어설션이 성공(Green)하는 데 집중합니다.

```python
# File: backend/app/utils/text_utils.py
import re

def strip_markdown(text: str) -> str:
    if not text:
        return ""
        
    # 1. 2차원 볼드 마커 및 단순 이탤릭 마커 제거
    text = re.sub(r"\*\*([^*]+)\*\*", r"\1", text)
    text = re.sub(r"\*([^*]+)\*", r"\1", text)
    
    # 2. 불필요하게 남은 연속된 별표 제거
    text = re.sub(r"\*{2,}", "", text)
    text = re.sub(r"\*+$|^\*+", "", text)
    
    return text.strip()
```

### 3단계: REFACTOR (클린 코드 및 성능 최적화)
기능(Green)을 유지한 상태에서 코드 가독성을 개선하고, 대용량 텍스트 파싱을 대비한 정규식 컴파일 캐싱 및 메모리 효율성을 극대화합니다.

```python
# File: backend/app/utils/text_utils.py
import re

# 정규표현식 컴파일러 객체를 모듈 레벨에서 1회만 캐싱하여 오버헤드 방지
_BOLD_PATTERN = re.compile(r"\*\*([^*]+)\*\*")
_ITALIC_PATTERN = re.compile(r"\*([^*]+)\*")
_MULTIPLE_STAR_PATTERN = re.compile(r"\*{2,}")
_EDGE_STAR_PATTERN = re.compile(r"\*+$|^\*+")

def strip_markdown(text: str) -> str:
    """마크다운 서식을 제거하고 연속된 볼드 마커 깨짐 현상을 안전하게 보정합니다.
    
    Args:
        text (str): 마크다운이 포함된 원본 텍스트
        
    Returns:
        str: 마크다운 기호가 정제된 플레인 텍스트
    """
    if not text:
        return ""

    # 정규식 체이닝 실행
    cleaned = _BOLD_PATTERN.sub(r"\1", text)
    cleaned = _ITALIC_PATTERN.sub(r"\1", cleaned)
    cleaned = _MULTIPLE_STAR_PATTERN.sub("", cleaned)
    cleaned = _EDGE_STAR_PATTERN.sub("", cleaned)

    return cleaned.strip()
```
이후 `pytest backend/app/tests/test_text_utils.py`를 실행하여 리팩토링 후에도 그린 상태가 유지되는지 확인합니다.


