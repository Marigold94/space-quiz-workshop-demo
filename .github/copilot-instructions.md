# 프로젝트

한국어 우주 탐사 퀴즈 COSMIC QUEST다. 앱 전체는 루트의 `index.html`에 있고 프로젝트 작업 규칙은 `.github/copilot-instructions.md`에 둔다.

# 실행과 검증

퀴즈는 빌드나 서버 없이 `index.html`을 직접 여는 방식이다. CSS와 JavaScript를 포함해 이 파일 하나로 유지하고 외부 라이브러리·폰트·네트워크 의존성을 추가하지 않는다.

| 작업 | 터미널 명령 |
| --- | --- |
| 퀴즈 열기: macOS | `open index.html` |
| 퀴즈 열기: Linux | `xdg-open index.html` |
| 퀴즈 열기: Windows PowerShell | `Start-Process .\index.html` |

자동 테스트 러너·개별 테스트 명령·린터는 설정되어 있지 않다.

코드 변경 후 브라우저에서 첫 문항 표시, 정답·오답 선택과 해설, 중복 답변 방지, 다음 문항 이동, 10문항 완료, 점수·진행률, 결과 화면, 다시 시작, 키보드 조작을 확인한다. 결과 점수 카운트업의 최종값과 다시 시작 시 취소, 라이트/다크 모드, 모션 감소 설정도 관련 변경 시 확인한다.

# 퀴즈 구조와 불변 조건

`index.html`의 `<style>` → 시맨틱 HTML → 일반 `<script>` 구조를 유지한다. 별도 시작 화면 없이 첫 문항에서 시작하며, 답변과 해설 → 다음 문항 → 결과 → 다시 시작 순서다.

- `questions`는 10개의 `{ question, answers, correct, explanation }` 객체다. 각 문항에는 보기 4개가 있고 `correct`는 보기 배열의 0부터 시작하는 인덱스다.
- `ui` 객체가 DOM 참조를 모은다. 상태는 `current`, `score`, `answered`, `scoreAnimation`으로 관리한다.
- `renderQuestion()`은 보기를 다시 만들고 답변 상태를 초기화한다. `selectAnswer()`는 `answered`로 재선택을 차단하고, 정답당 10점을 더한 뒤 해설을 표시한다.
- `updateProgress(completed)`의 진행률은 현재 문항 번호가 아니라 답변 완료 수다. 총점은 100점이다.
- `showResult()`는 퀴즈를 숨기고 점수별 반응을 표시한다. 점수 숫자는 `requestAnimationFrame`으로 올라가며, 다시 시작 시 예약된 프레임을 취소하고 상태를 초기화한다.
- 동적 문구는 `textContent`, `createElement`, `replaceChildren`으로 구성한다.

# 접근성과 화면 규칙

보기는 기본 `<button>`이고 Tab, Enter, Space로 조작한다. 답변 후 `aria-disabled="true"`로 상태를 알리되 실제 중복 실행 차단은 `answered`가 담당한다. 둘을 함께 유지한다.

초기 로딩에는 포커스를 강제로 옮기지 않는다. 이후 문항 전환과 다시 시작에서는 질문 제목, 답변 후에는 해설 영역, 결과에서는 결과 제목으로 포커스를 옮긴다. 해당 요소의 `tabindex="-1"`, 해설의 ARIA 연결, 점수 안내용 `role="status"`를 보존한다.

정오답은 색뿐 아니라 `✓ 정답` / `✕ 오답` 문구로도 표시한다. CSS 변수와 `prefers-color-scheme`으로 테마를 구성한다. `prefers-reduced-motion`은 CSS 애니메이션뿐 아니라 JavaScript 점수 카운트업에도 적용하며, 모션 감소 시 최종 점수를 즉시 표시한다.

문서와 사용자 화면 문구는 기존 한국어를 따른다.
