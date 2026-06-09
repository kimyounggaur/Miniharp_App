# 🎵 미니하프 연주 웹앱 — 통합 바이브코딩 프롬프트 (BEST · 15음 / 19음 / 21음)

> 키보드와 마우스/터치로 **미니하프(현)를 뜯어 연주**하는 인터랙티브 웹앱을 만들기 위한 **결정판 명세**입니다.
> 세 개의 초안(Claude · ChatGPT · Gemini)의 **장점만 골라 한 문서로 통합**했습니다.
>
> - **Claude 안에서** — 사용자 키 규칙을 그대로 지키는 **15/19/21 음역 설계(C3~B5, 포개짐 구조)**, **정수 배음 발현(pluck) 합성**, **포개짐을 검증하는 테스트**.
> - **ChatGPT 안에서** — 얇은 사선 현에 적합한 **SVG 라인 오버레이**(`pointer-events:stroke`), **곡선(베지어) 현 좌표 + 보정값 + 캘리브레이션**, **합성 임펄스 리버브**, **글리산도(드래그) 연주**, **MIDI 기반 주파수 계산**.
> - **Gemini 안에서** — Notion/Figma 톤의 **미니멀 UI 언어**와 **원본 이미지 무왜곡(`object-fit:contain`)** 강조.
>
> 추가로, 초안들에 있던 **두 가지 잠재 버그를 교정**했습니다.
> 1. SVG 오버레이는 세로형(포트레이트) 이미지에서 좌표가 어긋나므로 **`preserveAspectRatio="none"`** 으로 0~100 좌표를 화면 비율에 그대로 매핑합니다.
> 2. `vector-effect:non-scaling-stroke`에서는 `stroke-width`가 **화면 px** 단위이므로, 현 굵기를 `0.25`가 아니라 **px 스케일(현 1.6 / 글로우 2.4 / 히트 22)** 로 둡니다.
>
> **이번 요청을 최우선 규격으로 못 박았습니다.**
> 1. **화면 1(선택)** → 15음 / 19음 / 21음 미니하프 중 하나 선택
> 2. **화면 2(연주)** → 선택한 하프 원본 사진을 **9:16 화면에 무왜곡으로** 크게 띄우고, 그 위 **투명 SVG 현**과 키보드/마우스/터치로 연주
> 3. **입력 규칙(핵심 ★)**: `숫자키 단독` = 가운데(C4~B4) / `Shift`+숫자 = 높은음(C5 이상) / `Ctrl`+숫자 = 낮은음(B3 이하)
>
> **키 규칙에서 자연스럽게 도출되는 음역 설계 (★ 매우 중요):**
> 숫자키 `1~7` = **도·레·미·파·솔·라·시(C·D·E·F·G·A·B)**, 수식키 = **옥타브**(Ctrl=3옥타브, 단독=4옥타브, Shift=5옥타브).
> → 따라서 **연주 가능한 전체 범위는 정확히 3옥타브 = C3~B5 = 21음**입니다. (21 = 7음 × 3옥타브)
> → 15음 · 19음 · 21음을 **C3~B5 안에 포개지는(nested) 구조**로 확정합니다.
>
> | 하프 | 음역(C Major, 온음계) | 현 수 | Ctrl(낮은옥타브) | 숫자 단독(가운데) | Shift(높은옥타브) |
> |---|---|---|---|---|---|
> | **15음** | `G3 ~ G5` | 15 | ⌃5,⌃6,⌃7 → G3,A3,B3 | 1~7 → C4~B4 | ⇧1~⇧5 → C5~G5 |
> | **19음** | `E3 ~ B5` | 19 | ⌃3~⌃7 → E3~B3 | 1~7 → C4~B4 | ⇧1~⇧7 → C5~B5 |
> | **21음** | `C3 ~ B5` | 21 | ⌃1~⌃7 → C3~B3 | 1~7 → C4~B4 | ⇧1~⇧7 → C5~B5 |
>
> **포개짐:** 15음 ⊂ 19음 ⊂ 21음. **공통 규칙:** `같은 숫자 = 같은 계이름`(예: `5`는 어느 하프에서도 항상 '솔'). 선택한 하프에 **실제로 있는 음만** 소리납니다.

---

## 0. 이 문서 사용법 & 이미지 자산 준비 (먼저 읽기 ★)

1. 이 마크다운 **전체를 복사**해 AI 코딩 도구(Claude, Cursor, v0, Bolt, Lovable, Replit Agent, Gemini 등)에 붙여넣습니다.
2. 다음처럼 요청합니다.
   > 위 명세대로 **단일 `index.html` + `assets/` 폴더** 구조의 15/19/21음 미니하프 연주 웹앱을 완성해줘. 외부 라이브러리 없이 HTML/CSS/Vanilla JS와 Web Audio API만 사용해줘. 원본 하프 사진은 왜곡 없이 보여주고, SVG 투명 현 오버레이로 키보드·마우스·터치·글리산도 연주가 되게 해줘.
3. 결과를 받은 뒤 **17번 완성 체크리스트**로 검수하고, 어긋나는 항목을 다시 수정 요청합니다.

### 이미지 자산 파일명 규칙 (★ 반드시 영문 파일명으로 정리)

한글·공백·괄호·날짜가 들어간 파일명은 배포 환경에서 경로 오류가 잦습니다. 아래처럼 **단순 영문명**으로 바꿔 `assets/`에 넣고 코드에서는 영문명만 참조합니다.

| 업로드한 원본 파일 | 코드에서 쓸 파일명 | 용도 | 비고 |
|---|---|---|---|
| `15현_하프_2.jpg` (라미카 셀틱 미니하프) | `assets/harp-15.jpg` | 15음 선택 카드 + 연주 화면 | 실제 15현 사진, 그대로 사용 |
| `19현_하프_2.jpg` (라이어형, F3~C6 라벨) | `assets/harp-19.jpg` | 19음 선택 카드 + 연주 화면 | 사진 라벨은 F3~C6지만, 키 규칙상 앱은 **E3~B5**로 매핑(아래 ★ 주의 1) |
| `미니하프_크몽_실사_23현.JPG` (실사 렌더) | `assets/harp-21.jpg` | 21음 선택 카드 + 연주 화면 | 사진은 23현이지만 **21음**으로 사용(아래 ★ 주의 2) |
| 선택 아이콘 | `assets/icon.png` | 파비콘/PWA 아이콘 | 선택 사항 |

**중요 이미지 규칙**
- 사진은 앱의 핵심 비주얼이므로 새로 그리거나 대체하지 말고 **원본 사진을 최대한 그대로** 사용합니다.
- 모든 사진은 **종횡비를 절대 변경하지 않습니다.** `object-fit: contain`만 사용하고 `cover`로 잘라내지 않습니다. **왜곡·잘림 금지.**
- 연주 효과 때문에 **사진 자체를 흔들거나 확대하지 않습니다.** 현 글로우·진동·파동은 **오버레이 SVG에서만** 처리합니다.

### ★ 주의 1 — 19현 사진의 라벨(F3~C6)과 앱 음역(E3~B5)
업로드한 19현 사진에는 `F3 … C6`이 인쇄되어 있습니다. 그런데 **C6은 6옥타브**라 `Shift`(5옥타브)·`Ctrl`(3옥타브) 규칙만으로는 누를 수 없습니다. 그래서 이 앱의 19음은 **키 규칙을 우선**해 `E3~B5`로 매핑합니다(가운데·Shift를 꽉 채우는 가장 깔끔한 19음 포개짐). 만약 **사진 라벨(F3~C6)을 그대로 살리고 싶다면**, 숫자 위쪽 키(예: `-`,`=`)나 `Alt`+숫자를 **6옥타브 전용**으로 추가하면 됩니다(확장 아이디어 18번 참고).

### ★ 주의 2 — 실사 사진은 23현, 요청은 21음
실사 렌더는 현이 **23개**입니다. 본 앱은 **21음**이므로, 기본 전략은 **21개 현 히트존을 균등/곡선 배치하고 남는 2현은 시각용으로 둡니다.** 정확히 맞추려면 (a) **21현 도면으로 교체**하거나 (b) 아래 `?calibrate=1` 도구로 실제 현 좌표를 찍어 `OVERRIDES`에 넣어 보정합니다.

### 하프 현 색상 관례 (시각 표시)
실제 하프는 음을 빨리 찾도록 **C현=빨강, F현=파랑(검정)**, 나머지=중립색으로 칠합니다. 오버레이의 시각용 현 색을 이 관례대로 줍니다.

---

## 1. 프로젝트 개요 & 핵심 요구사항

브라우저에서 **키보드와 마우스/터치로 미니하프를 연주**하는 인터랙티브 웹앱을 만듭니다.

- **화면 1 — 선택 화면**: `15음` / `19음` / `21음` 미니하프 중 하나를 카드로 선택합니다.
- **화면 2 — 연주 화면**: 선택한 하프 사진이 9:16 화면 안에 크게 표시되고, 사진 위에 **투명 SVG 현 히트라인**을 얹습니다.
- **입력 규칙(핵심 ★)**: `숫자키` 단독 = `C4~B4` / `Shift`+숫자 = `C5` 이상 / `Ctrl`+숫자 = `B3` 이하.
- **사운드**: Web Audio API 합성. mp3 없이 동작하되, 나중에 실제 하프 샘플로 쉽게 교체할 수 있도록 `playNote(noteId, opts)` 인터페이스로 분리합니다.
- **톤**: 원목 미니하프의 따뜻함, 맑은 현 울림, 부드러운 잔향, 명상적이고 동화적인 분위기. UI는 군더더기 없이 깔끔하고 모던하게.

### 확정 튜닝(기본 프리셋 · C Major 다이아토닉)
실제 상품마다 튜닝이 다를 수 있으므로 기본값은 **C Major 온음계**로 고정합니다. 보유 악기가 다르면 `NOTES_15/19/21`만 교체하면 됩니다.

| 모델 | 기본 음역 | 음 개수 | 음 배열(낮은음 → 높은음) |
|---|---:|---:|---|
| 15음 | G3~G5 | 15 | `G3 A3 B3 C4 D4 E4 F4 G4 A4 B4 C5 D5 E5 F5 G5` |
| 19음 | E3~B5 | 19 | `E3 F3 G3 A3 B3 C4 D4 E4 F4 G4 A4 B4 C5 D5 E5 F5 G5 A5 B5` |
| 21음 | C3~B5 | 21 | `C3 D3 E3 F3 G3 A3 B3 C4 D4 E4 F4 G4 A4 B4 C5 D5 E5 F5 G5 A5 B5` |

---

## 2. 기술 스택 & 결과물 형태

- **단일 `index.html`**: HTML + CSS + Vanilla JS를 한 파일에 작성합니다.
- **이미지 폴더**: `assets/harp-15.jpg`, `assets/harp-19.jpg`, `assets/harp-21.jpg`, 선택적으로 `assets/icon.png`.
- **사운드**: Web Audio API만 사용합니다 — `AudioContext`, `OscillatorNode`, `GainNode`, `BiquadFilterNode`, `ConvolverNode`(합성 임펄스 리버브), `StereoPannerNode`, 짧은 노이즈 버스트.
- **외부 라이브러리 금지**: React, jQuery, Tone.js, Howler.js 등 사용하지 않습니다. (부록 B에 React+TypeScript 선택 구조만 별도 제공.)
- **코드 품질 원칙**
  - 음/현/키 매핑/이미지/좌표는 `NOTE`, `NOTES_15/19/21`, `MAP`, `HARPS`, `OVERRIDES` 같은 **데이터 객체로 분리**합니다.
  - 키 입력 → 음 변환은 테스트 가능한 **순수 함수 `resolveShortcut()`** 으로 구현합니다.
  - 키보드·마우스·터치·글리산도는 모두 **동일한 `triggerNote(noteId)`** 경로를 사용합니다.
  - 접근성: focusable SVG 요소, `aria-label`, `aria-live`, 포커스 링을 반영합니다.

---

## 3. ★ 화면/레이아웃 규격 — 9:16 풀스크린

세로형 악기인 미니하프에 맞게 **9:16 비율의 한 화면**을 기준으로 합니다.

- 루트 `#app`은 항상 9:16 비율을 유지하며 뷰포트 안에서 **최대 크기로 중앙 배치**합니다.
- 모바일 세로에서는 화면을 거의 꽉 채우고, 데스크톱/가로에서는 9:16 패널이 가운데 배치되며 바깥은 우드/딥그린 계열 **레터박스 배경**으로 채웁니다.
- 모바일 주소창 대응으로 `100dvh`를 쓰고, 구형 폴백으로 `100vh`도 고려합니다.
- 화면 1·2는 `#app` 안에서 `position:absolute; inset:0;`으로 겹친 뒤 `hidden` 토글로 전환합니다.
- **빈 흰 여백 금지.** 남는 공간은 테마 배경으로 채웁니다.

```css
* { box-sizing: border-box; }
html, body { margin: 0; min-height: 100%; touch-action: manipulation; }
body {
  display: grid; place-items: center;
  min-height: 100dvh;
  background:
    radial-gradient(circle at 50% 0%, rgba(233,198,139,.20), transparent 38%),
    linear-gradient(180deg, #1d261d 0%, #0e1713 62%, #080d0b 100%);
  font-family: "Pretendard", "Apple SD Gothic Neo", system-ui, -apple-system, sans-serif;
  color: var(--ink);
  user-select: none;
}
#app {
  position: relative;
  width:  min(100vw, calc(100dvh * 9 / 16));
  height: min(100dvh, calc(100vw * 16 / 9));
  overflow: hidden; isolation: isolate;
  background:
    radial-gradient(circle at 50% 10%, rgba(255,222,163,.22), transparent 34%),
    linear-gradient(180deg, #273322 0%, #132018 55%, #0d1712 100%);
  box-shadow: 0 28px 80px rgba(0,0,0,.48);
}
.screen { position: absolute; inset: 0; display: flex; flex-direction: column; overflow: hidden; }
.screen[hidden] { display: none; }
```

---

## 4. ★ 원본 이미지 무왜곡 + SVG 현 오버레이 정렬 (가장 중요)

미니하프 현은 **얇고 대각선**으로 배치되므로, 원형 버튼보다 **SVG 라인 오버레이**가 정확합니다.

### 핵심 원칙
- 사진은 `object-fit: contain`으로 표시합니다.
- **사진과 같은 종횡비를 가진 `.harp-stage` 래퍼**를 만들고(`aspect-ratio: var(--harp-aspect)`), 그 위에 `.string-layer` SVG를 절대 배치합니다.
- SVG는 **`viewBox="0 0 100 100"` + `preserveAspectRatio="none"`** 으로 둡니다. 이렇게 하면 0~100 좌표가 스테이지의 **가로%·세로%에 그대로** 매핑되어, 세로형 사진에서도 현이 정확히 정렬됩니다. (`preserveAspectRatio`를 빼면 정사각 letterbox가 생겨 포트레이트에서 어긋납니다.)
- 현마다 SVG `line` 3개를 겹칩니다.
  - `.string-visible`: 실제 현 위에 살짝 겹치는 **시각용 선**. C현 빨강, F현 파랑, 나머지 아이보리.
  - `.string-glow`: 평소 숨김, 연주 시 **앰버 글로우**.
  - `.string-hit`: 투명하지만 두꺼운 **클릭/터치 히트라인**. `pointer-events: stroke`로 현 주변을 쉽게 누릅니다.
- 모든 선은 **`vector-effect: non-scaling-stroke`** 라서 화면 크기와 무관하게 굵기가 일정합니다. → `stroke-width`는 **화면 px** 로 해석되므로 px 스케일(현 1.6 / 글로우 2.4 / 히트 22)로 둡니다.
- **사진 자체는 절대 흔들거나 확대하지 않습니다.**

```css
.play-main { flex: 1 1 auto; min-height: 0; display: grid; place-items: center; padding: clamp(6px, 2.5vw, 18px); }
.harp-stage {
  position: relative;
  width: min(96%, 62vh);
  max-width: 100%;
  aspect-ratio: var(--harp-aspect, 3 / 5);
  margin: auto;
}
.harp-photo {
  width: 100%; height: 100%;
  object-fit: contain; display: block;
  user-select: none; -webkit-user-drag: none;
  filter: drop-shadow(0 20px 24px rgba(0,0,0,.34));
}
.string-layer { position: absolute; inset: 0; width: 100%; height: 100%; overflow: visible; }
.string-item { outline: none; }
.string-visible {
  stroke: var(--string-color, rgba(255,252,235,.55));
  stroke-width: 1.6;            /* non-scaling-stroke → 화면 ~1.6px */
  stroke-linecap: round;
  vector-effect: non-scaling-stroke;
  opacity: .55; pointer-events: none;
}
.string-hit {
  stroke: rgba(255,255,255,.001);  /* 투명하지만 클릭/터치 가능 */
  stroke-width: 22;                /* 손가락용 넉넉한 히트 폭(px) */
  stroke-linecap: round;
  vector-effect: non-scaling-stroke;
  cursor: pointer; pointer-events: stroke;
}
.string-glow {
  stroke: var(--glow);
  stroke-width: 2.4; stroke-linecap: round;
  vector-effect: non-scaling-stroke;
  opacity: 0;
  filter: drop-shadow(0 0 9px rgba(255,213,122,.95));
  pointer-events: none;
  transition: opacity .14s ease, stroke-width .18s ease;
}
.string-item.is-playing .string-glow { opacity: .95; stroke-width: 3.4; animation: strum .5s ease-out; }
@keyframes strum { 0%{transform:translateX(.6px)} 35%{transform:translateX(-.6px)} 70%{transform:translateX(.3px)} 100%{transform:translateX(0)} }
/* C현=빨강, F현=파랑 표시점 */
.string-dot { pointer-events: none; }
.string-dot.is-c { fill: var(--red-string); }
.string-dot.is-f { fill: var(--blue-string); }
.string-item:focus-visible .string-visible,
.string-item:focus-visible .string-glow { opacity: 1; stroke-width: 3; }
@media (prefers-reduced-motion: reduce) {
  .string-glow { transition: opacity .01ms linear; }
  .string-item.is-playing .string-glow { animation: none; }
}
```

---

## 5. 디자인 시스템

원목 미니하프의 따뜻함과 현악기의 맑은 울림을 살립니다. 전체 톤은 **따뜻한 나무색 + 크림색 + 딥그린 + 금빛 글로우**, 형태는 **부드러운 곡선과 옅은 그림자**(Notion/Figma 톤).

```css
:root {
  --bg: #1a1410;
  --panel: rgba(39,51,34,.78);
  --panel-strong: rgba(45,37,26,.88);
  --wood: #B9773D;
  --wood-dark: #6E3F20;
  --gold: #D9B45B;
  --glow: #FFE08A;
  --ink: #F3ECDD;
  --muted: #C9BDA7;
  --green: #5E8067;
  --red-string: #D8504A;   /* C현 */
  --blue-string: #4A7FD8;  /* F현 */
  --string-color: rgba(255,250,232,.62);
  --shadow: rgba(0,0,0,.42);
  /* 하프별 포인트 색 */
  --rose: #C77B52;    /* 15음 */
  --teal: #4FA39A;    /* 19음 */
  --violet: #8E6FC0;  /* 21음 */
}
```

**스타일 원칙**
- 카드/패널은 `border-radius: 22px~30px`, 안쪽은 반투명 글래스 느낌.
- 15음/19음/21음은 포인트 색으로 구분: 15음 `--rose`, 19음 `--teal`, 21음 `--violet`.
- 클릭/연주 효과는 강한 바운스보다 **잔잔한 글로우 + 짧은 현 떨림**.
- `prefers-reduced-motion`이면 움직임 최소화, 글로우만.

---

## 6. [화면 1] 선택 화면 — 15음 / 19음 / 21음 카드

### 레이아웃
- 상단 16~22%: 타이틀/설명.
  - 제목: `🎵 미니하프 연주하기`
  - 부제: `숫자키로 현을 튕기고, Shift/Ctrl로 높은음·낮은음을 연주해요.`
- 본문 70~78%: 카드 3개. 모바일 세로는 세로 스택, 데스크톱은 가로/2+1 그리드(단, `#app` 9:16 안에서 넘치지 않게).
- 하단 5~8%: 짧은 안내.

### 카드 내용
| 카드 | 이미지 | 라벨 | 부제 |
|---|---|---|---|
| 15음 | `assets/harp-15.jpg` | `15음 미니하프` | `G3~G5 · C Major · 가볍게 시작` |
| 19음 | `assets/harp-19.jpg` | `19음 미니하프` | `E3~B5 · C Major · 균형 잡힌 음역` |
| 21음 | `assets/harp-21.jpg` | `21음 미니하프` | `C3~B5 · C Major · 3옥타브 풀레인지` |

### 상호작용/접근성
- hover/focus: `translateY(-4px) scale(1.02)` + 부드러운 그림자.
- 클릭/Enter/Space: 선택 카드 글로우 → 0.3~0.4초 후 연주 화면으로 전환.
- 카드는 `<button class="select-card">` 또는 `role="button" tabindex="0"`, `aria-label="15음 미니하프 선택"`.

---

## 7. [화면 2] 연주 화면 — 선택된 미니하프 연주

### 레이아웃
- **상단 바**: 좌측 `← 다시 선택`, 가운데 악기명(예 `21음 미니하프`), 우측 볼륨 슬라이더(또는 아이콘+접이식).
- **메인**: `.harp-stage` 안에 사진(`.harp-photo`) + SVG 오버레이(`.string-layer`). 현 클릭/터치로 재생, **현 위 드래그로 글리산도**(14번).
- **하단 패널**: 요약 `숫자 = C4~B4 · Shift+숫자 = C5↑ · Ctrl+숫자 = B3↓`, 최근 연주음 배지(예 `높은 도 · C5 · Shift+1`), `키표 보기` 토글(모델별 유효 키 표).

```js
function renderHarp(type) {
  const cfg = HARPS[type];
  state.currentType = type;
  const stage = document.querySelector('.harp-stage');
  stage.style.setProperty('--harp-aspect', cfg.aspect);
  stage.style.setProperty('--accent', cfg.accent);
  const img = document.querySelector('.harp-photo');
  img.src = cfg.image; img.alt = cfg.name;
  document.querySelector('#currentTitle').textContent = cfg.name;
  renderStrings(type);
  renderKeyboardGuide(type);
}
```

---

## 8. 키보드 입력 규칙 (핵심 ★)

### 기본 원칙
숫자 `1~7` = 계이름(도레미파솔라시), 수식키 = 옥타브.

| 입력 | 옥타브 | 음역 | 매핑 |
|---|---|---|---|
| `Ctrl`+숫자 | 3옥타브 | `B3` 이하 | `1=C3,2=D3,3=E3,4=F3,5=G3,6=A3,7=B3` |
| `숫자키` 단독 | 4옥타브 | `C4~B4` | `1=C4,2=D4,3=E4,4=F4,5=G4,6=A4,7=B4` |
| `Shift`+숫자 | 5옥타브 | `C5` 이상 | `1=C5,2=D5,3=E5,4=F5,5=G5,6=A5,7=B5` |

**중요(엣지케이스)**
- 선택한 악기에 **실제로 있는 음만** 소리납니다. 없는 조합·`8/9/0`은 **무음**.
- `Alt` 또는 `Meta(⌘)`가 눌리면 OS/브라우저 단축키 보호를 위해 **무음**.
- `Ctrl+Shift` 동시 **무음**.
- `event.repeat`(꾹 누름) **무시**. 손으로 빠르게 여러 번 누르면 매번 재생.
- 반드시 **`event.code`** 사용(`event.key`는 `Shift+1`이 `!`로 바뀌어 깨짐). **`Digit1~0`과 `Numpad1~0` 모두** 지원.
- 매핑된 **유효 키에서만 `preventDefault()`**(Ctrl+숫자 탭 전환·저장 차단).

### 모델별 유효 키
#### 15음 `G3~G5`
| 입력 | 음 | 입력 | 음 | 입력 | 음 |
|---|---|---|---|---|---|
| `Ctrl+5` | G3 | `1` | C4 | `Shift+1` | C5 |
| `Ctrl+6` | A3 | `2` | D4 | `Shift+2` | D5 |
| `Ctrl+7` | B3 | `3` | E4 | `Shift+3` | E5 |
|  |  | `4` | F4 | `Shift+4` | F5 |
|  |  | `5` | G4 | `Shift+5` | G5 |
|  |  | `6` | A4 |  |  |
|  |  | `7` | B4 |  |  |

#### 19음 `E3~B5`
| 입력 | 음 | 입력 | 음 | 입력 | 음 |
|---|---|---|---|---|---|
| `Ctrl+3` | E3 | `1` | C4 | `Shift+1` | C5 |
| `Ctrl+4` | F3 | `2` | D4 | `Shift+2` | D5 |
| `Ctrl+5` | G3 | `3` | E4 | `Shift+3` | E5 |
| `Ctrl+6` | A3 | `4` | F4 | `Shift+4` | F5 |
| `Ctrl+7` | B3 | `5` | G4 | `Shift+5` | G5 |
|  |  | `6` | A4 | `Shift+6` | A5 |
|  |  | `7` | B4 | `Shift+7` | B5 |

#### 21음 `C3~B5` (모든 조합 사용)
| 입력 | 음 | 입력 | 음 | 입력 | 음 |
|---|---|---|---|---|---|
| `Ctrl+1` | C3 | `1` | C4 | `Shift+1` | C5 |
| `Ctrl+2` | D3 | `2` | D4 | `Shift+2` | D5 |
| `Ctrl+3` | E3 | `3` | E4 | `Shift+3` | E5 |
| `Ctrl+4` | F3 | `4` | F4 | `Shift+4` | F5 |
| `Ctrl+5` | G3 | `5` | G4 | `Shift+5` | G5 |
| `Ctrl+6` | A3 | `6` | A4 | `Shift+6` | A5 |
| `Ctrl+7` | B3 | `7` | B4 | `Shift+7` | B5 |

---

## 9. 음 데이터 (15음 / 19음 / 21음)

> **C Major 온음계.** 왼쪽(긴 현)이 가장 낮은 음, 오른쪽(짧은 현)으로 갈수록 높아짐.

```js
// 15음 C Major: G3~G5
const NOTES_15 = ["G3","A3","B3","C4","D4","E4","F4","G4","A4","B4","C5","D5","E5","F5","G5"];
// 19음 C Major: E3~B5  (15음에 저음 E3·F3, 고음 A5·B5 추가)
const NOTES_19 = ["E3","F3","G3","A3","B3","C4","D4","E4","F4","G4","A4","B4","C5","D5","E5","F5","G5","A5","B5"];
// 21음 C Major: C3~B5  (3옥타브 풀레인지, 모든 키 조합 사용)
const NOTES_21 = ["C3","D3","E3","F3","G3","A3","B3","C4","D4","E4","F4","G4","A4","B4","C5","D5","E5","F5","G5","A5","B5"];
```

```js
// 주파수는 MIDI에서 계산(오차·중복 방지). num=자음보(.숫자=낮은옥타브, 숫자'=높은옥타브)
const freqFromMidi = m => +(440 * Math.pow(2, (m - 69) / 12)).toFixed(2);

const NOTE = {
  // 낮은 옥타브(3) — Ctrl 영역
  C3:{ midi:48, num:".1", ko:"낮은 도" }, D3:{ midi:50, num:".2", ko:"낮은 레" },
  E3:{ midi:52, num:".3", ko:"낮은 미" }, F3:{ midi:53, num:".4", ko:"낮은 파" },
  G3:{ midi:55, num:".5", ko:"낮은 솔" }, A3:{ midi:57, num:".6", ko:"낮은 라" },
  B3:{ midi:59, num:".7", ko:"낮은 시" },
  // 가운데 옥타브(4) — 숫자 단독
  C4:{ midi:60, num:"1", ko:"도" }, D4:{ midi:62, num:"2", ko:"레" },
  E4:{ midi:64, num:"3", ko:"미" }, F4:{ midi:65, num:"4", ko:"파" },
  G4:{ midi:67, num:"5", ko:"솔" }, A4:{ midi:69, num:"6", ko:"라" },
  B4:{ midi:71, num:"7", ko:"시" },
  // 높은 옥타브(5) — Shift 영역
  C5:{ midi:72, num:"1'", ko:"높은 도" }, D5:{ midi:74, num:"2'", ko:"높은 레" },
  E5:{ midi:76, num:"3'", ko:"높은 미" }, F5:{ midi:77, num:"4'", ko:"높은 파" },
  G5:{ midi:79, num:"5'", ko:"높은 솔" }, A5:{ midi:81, num:"6'", ko:"높은 라" },
  B5:{ midi:83, num:"7'", ko:"높은 시" }
};
Object.values(NOTE).forEach(n => { n.freq = freqFromMidi(n.midi); });
// 참고 Hz: C3 130.81 · C4 261.63 · A4 440 · C5 523.25 · B5 987.77

// 옥타브 1:1 매핑(Ctrl=3, 단독=4, Shift=5)
const MAP = {
  LOW:  { 1:"C3", 2:"D3", 3:"E3", 4:"F3", 5:"G3", 6:"A3", 7:"B3" },
  MID:  { 1:"C4", 2:"D4", 3:"E4", 4:"F4", 5:"G4", 6:"A4", 7:"B4" },
  HIGH: { 1:"C5", 2:"D5", 3:"E5", 4:"F5", 5:"G5", 6:"A5", 7:"B5" }
};
```

---

## 10. 현 좌표/오버레이 모델 (곡선 베지어 + 보정)

미니하프 현은 보통 **낮은음=왼쪽**, 높은음=오른쪽이며, 상단 핀에서 사운드보드/브리지로 비스듬히 내려갑니다. 사진마다 기울기·프레임이 달라서, 좌표를 **생성식(상단 곡선 + 하단 곡선) + 보정값** 구조로 만듭니다.

### 하프 설정 객체 (config + geometry 통합)
```js
const HARPS = {
  "15": {
    type:"15", name:"15음 미니하프", range:"G3~G5", scale:"C Major",
    image:"assets/harp-15.jpg", accent:"var(--rose)",
    aspect:"190 / 400",   // ★ 실제 이미지 비율로 측정해 교체
    notes: NOTES_15,
    geo: { // .harp-stage 0~100% 좌표. t=0(최저음·왼쪽) → t=1(최고음·오른쪽)
      top:    { x0:30, y0:14, cx:51, cy:11, x1:72, y1:20 }, // 상단 핀 라인
      bottom: { x0:42, y0:86, cx:57, cy:64, x1:70, y1:44 }  // 하단 브리지 라인
    }
  },
  "19": {
    type:"19", name:"19음 미니하프", range:"E3~B5", scale:"C Major",
    image:"assets/harp-19.jpg", accent:"var(--teal)",
    aspect:"540 / 760",
    notes: NOTES_19,
    geo: { // 라이어형은 현이 거의 수직·평행 → top/bottom x 분포를 비슷하게
      top:    { x0:15, y0:13, cx:49, cy:9,  x1:85, y1:14 },
      bottom: { x0:17, y0:86, cx:50, cy:84, x1:83, y1:82 }
    }
  },
  "21": {
    type:"21", name:"21음 미니하프", range:"C3~B5", scale:"C Major",
    image:"assets/harp-21.jpg", accent:"var(--violet)",
    aspect:"600 / 800",
    notes: NOTES_21,
    geo: { // 실사 렌더는 현이 사선으로 퍼짐(부채꼴)
      top:    { x0:34, y0:22, cx:52, cy:18, x1:70, y1:26 },
      bottom: { x0:30, y0:76, cx:53, cy:58, x1:76, y1:40 }
    }
  }
};

// 특정 현만 어긋날 때만 해당 음을 덮어씀: { x1,y1(상단), x2,y2(하단) }
const OVERRIDES = { "15": {}, "19": {}, "21": {} };
```

### 좌표 계산 + SVG 렌더링
```js
const SVG_NS = "http://www.w3.org/2000/svg";
const svgEl = (name, attrs={}) => { const el=document.createElementNS(SVG_NS,name);
  Object.entries(attrs).forEach(([k,v])=>el.setAttribute(k,String(v))); return el; };

function quadPoint(c, t){ const m=1-t; return {
  x: m*m*c.x0 + 2*m*t*c.cx + t*t*c.x1,
  y: m*m*c.y0 + 2*m*t*c.cy + t*t*c.y1 }; }

function makeStringPositions(type){
  const cfg = HARPS[type], last = cfg.notes.length - 1;
  return cfg.notes.map((noteId, i) => {
    const t = last === 0 ? 0 : i/last;
    const top = quadPoint(cfg.geo.top, t), bot = quadPoint(cfg.geo.bottom, t);
    const o = OVERRIDES[type]?.[noteId] || {};
    return { noteId, i,
      x1: o.x1 ?? top.x, y1: o.y1 ?? top.y,
      x2: o.x2 ?? bot.x, y2: o.y2 ?? bot.y };
  });
}

function stringColor(noteId){
  const pc = noteId[0];
  if (pc === "C") return "var(--red-string)";   // 하프 관례: C현 빨강
  if (pc === "F") return "var(--blue-string)";   // F현 파랑
  return "var(--string-color)";
}

function renderStrings(type){
  const svg = document.querySelector(".string-layer");
  svg.innerHTML = "";
  svg.setAttribute("viewBox", "0 0 100 100");
  svg.setAttribute("preserveAspectRatio", "none");   // ★ 포트레이트 정렬 핵심
  svg.setAttribute("aria-label", `${HARPS[type].name} 현 영역`);

  makeStringPositions(type).forEach(pos => {
    const { noteId, x1, y1, x2, y2 } = pos;
    const g = svgEl("g", { class:"string-item", tabindex:"0", role:"button",
      "data-note": noteId, "aria-label": `${labelOf(noteId)}, 단축키 ${shortcutLabelOf(noteId)}` });
    g.style.setProperty("--string-color", stringColor(noteId));

    g.append(
      svgEl("line", { class:"string-visible", x1,y1,x2,y2 }),
      svgEl("line", { class:"string-glow",    x1,y1,x2,y2 }),
      svgEl("line", { class:"string-hit",     x1,y1,x2,y2 })
    );
    // C/F 현 색상 표시점(상단)
    const pc = noteId[0];
    if (pc === "C" || pc === "F")
      g.append(svgEl("circle", { class:`string-dot ${pc==="C"?"is-c":"is-f"}`, cx:x1, cy:y1, r:1.4 }));

    const play = ev => { ev.preventDefault(); triggerNote(noteId, { source:"pointer" }); };
    g.addEventListener("pointerdown", play);
    g.addEventListener("keydown", ev => { if (ev.code==="Enter"||ev.code==="Space") play(ev); });
    svg.appendChild(g);
  });
}
```

### 좌표 보정 도구 `?calibrate=1`
연주 화면을 `index.html?calibrate=1`로 열면 클릭 좌표(0~100%)를 콘솔에 출력합니다. `preserveAspectRatio="none"` 덕분에 화면 클릭 위치가 곧 SVG 좌표라 그대로 붙여넣으면 됩니다.
```js
function enableCalibrate(){
  const stage = document.querySelector(".harp-stage");
  let pts = [];
  stage.addEventListener("pointerdown", e => {
    const r = stage.getBoundingClientRect();
    const x = +(((e.clientX - r.left)/r.width )*100).toFixed(1);
    const y = +(((e.clientY - r.top )/r.height)*100).toFixed(1);
    pts.push({x,y});
    console.log(`(${pts.length}) x:${x}, y:${y}`, pts);
  });
  console.log("[calibrate] 현마다 상단→하단 2점씩 클릭. geo.top/bottom 또는 OVERRIDES에 반영하세요.");
}
if (new URLSearchParams(location.search).get("calibrate")) enableCalibrate();
```
**보정 순서**: ① `?calibrate=1`로 열기 → ② 최저음 현의 상단·하단, 최고음 현의 상단·하단 클릭 → ③ `geo.top.x0/y0/x1/y1`, `geo.bottom...`에 반영 → ④ 가운데가 휘면 `cx/cy` 조정 → ⑤ 몇 개만 어긋나면 `OVERRIDES`에 해당 음만 추가. (먼저 `aspect`가 실제 이미지 비율과 같은지 확인.)

---

## 11. 키 입력 처리 로직 (순수 함수 + 테스트)

```js
function digitOf(code){ const m=/^(?:Digit|Numpad)([0-9])$/.exec(code); return m ? m[1] : null; }
function isEditableTarget(t){ return !!(t instanceof Element && t.closest('input, textarea, select, [contenteditable="true"]')); }

function resolveShortcut(e, type){
  if (!HARPS[type]) return null;
  if (e.altKey || e.metaKey) return null;
  if (e.shiftKey && e.ctrlKey) return null;
  const d = digitOf(e.code); if (d === null) return null;
  const note = e.shiftKey ? MAP.HIGH[d] : e.ctrlKey ? MAP.LOW[d] : MAP.MID[d];
  if (!note) return null;
  return HARPS[type].notes.includes(note) ? note : null;  // 하프에 없는 음은 무음
}

addEventListener("keydown", e => {
  if (e.repeat || isEditableTarget(e.target) || !state.currentType) return;
  const note = resolveShortcut(e, state.currentType);
  if (!note) return;
  e.preventDefault();              // 유효 키에서만 기본동작 차단
  triggerNote(note, { source:"keyboard" });
});
```

**필수 테스트 (`console.assert` — 개발 중 전부 통과)**
```js
const M=(code,mods,t)=>resolveShortcut({code,shiftKey:!!mods.s,ctrlKey:!!mods.c,altKey:!!mods.a,metaKey:!!mods.m},t);
// 숫자 단독 = 4옥타브(공통)
console.assert(M("Digit1",{},"15")==="C4");
console.assert(M("Digit7",{},"21")==="B4");
console.assert(M("Digit8",{},"21")===null);
console.assert(M("Numpad1",{},"19")==="C4");
// Shift = 5옥타브
console.assert(M("Digit5",{s:1},"15")==="G5");   // 15음 최고음
console.assert(M("Digit6",{s:1},"15")===null);   // 15음엔 A5 없음
console.assert(M("Digit7",{s:1},"19")==="B5");
console.assert(M("Digit7",{s:1},"21")==="B5");
// Ctrl = 3옥타브
console.assert(M("Digit5",{c:1},"15")==="G3");   // 15음 최저음
console.assert(M("Digit4",{c:1},"15")===null);   // 15음엔 F3 없음
console.assert(M("Digit3",{c:1},"19")==="E3");   // 19음 최저음
console.assert(M("Digit2",{c:1},"19")===null);   // 19음엔 D3 없음
console.assert(M("Digit1",{c:1},"21")==="C3");   // 21음 최저음
console.assert(M("Digit7",{c:1},"21")==="B3");
// 같은 숫자 = 같은 계이름 (5는 항상 '솔')
console.assert(M("Digit5",{c:1},"21")==="G3" && M("Digit5",{},"21")==="G4" && M("Digit5",{s:1},"21")==="G5");
// 무효 조합
console.assert(M("Digit4",{s:1,c:1},"21")===null);
console.assert(M("Digit1",{a:1},"21")===null);
console.assert(M("Digit1",{m:1},"21")===null);
// 데이터 무결성 + 포개짐(15 ⊂ 19 ⊂ 21)
console.assert(NOTES_15.length===15 && NOTES_19.length===19 && NOTES_21.length===21);
console.assert([...NOTES_15,...NOTES_19,...NOTES_21].every(n=>NOTE[n]));
console.assert(NOTES_15.every(n=>NOTES_19.includes(n)) && NOTES_19.every(n=>NOTES_21.includes(n)));
```

---

## 12. 사운드 엔진 — Web Audio 미니하프 합성 (뜯는 현)

**목표 음색**: 손끝으로 현을 뜯는 **빠른 어택** + 맑고 따뜻한 **정수 배음(harmonic)** 의 울림 + 은은한 잔향. 낮은 현일수록 길게 울림. (종·금속 느낌의 비배음인 텅드럼과 달리 **하프는 정수 배음**이 핵심.) **폴리포니**(여러 현 동시·아르페지오), 키를 떼도 **자연 감쇠**(강제로 끊지 않음).

**필수 구조**
- `AudioContext`는 **1회 생성·재사용**, **첫 사용자 입력에서 `resume()`**.
- **마스터 게인 + 볼륨 슬라이더(0~1)** + **합성 임펄스 리버브(ConvolverNode)**. 현 위치 기반 **스테레오 팬**(낮은음=왼쪽, 높은음=오른쪽).
- 음 = 기본음 + **정수 배음 파셜(1×~7×)**(높은 배음일수록 빨리 감쇠) + **손끝 플럭 트랜지언트(밴드패스 노이즈)** + **밝게 시작해 닫히는 로우패스**.
- 추후 실제 샘플로 교체할 땐 `playNote(noteId, opts)` 내부만 갈아끼웁니다.

```js
let ctx=null, master=null, reverbSend=null;

function ensureAudio(){
  if(!ctx){
    ctx = new (window.AudioContext || window.webkitAudioContext)();
    master = ctx.createGain(); master.gain.value = 0.85; master.connect(ctx.destination);
    const conv = ctx.createConvolver(); conv.buffer = makeIR(ctx, 1.9, 2.6);
    reverbSend = ctx.createGain(); reverbSend.gain.value = 0.18;   // 잔향 양 0~0.35
    reverbSend.connect(conv).connect(ctx.destination);
  }
  if(ctx.state === "suspended") ctx.resume();                       // ★ autoplay 해제
}
function makeIR(ctx, sec, decay){
  const rate=ctx.sampleRate, len=Math.max(1,Math.floor(rate*sec));
  const buf=ctx.createBuffer(2,len,rate);
  for(let ch=0;ch<2;ch++){ const d=buf.getChannelData(ch);
    for(let i=0;i<len;i++) d[i]=(Math.random()*2-1)*Math.pow(1-i/len, decay); }
  return buf;
}
function setVolume(v){ ensureAudio(); master.gain.setTargetAtTime(Math.max(0,Math.min(1,+v)), ctx.currentTime, 0.015); }

// 손끝 트랜지언트(밴드패스 노이즈) — 하프 특유의 '띵' 어택
function makePluckNoise(now, dest, f, amt=0.05){
  const dur=0.018, n=Math.max(1,Math.floor(ctx.sampleRate*dur));
  const buf=ctx.createBuffer(1,n,ctx.sampleRate), d=buf.getChannelData(0);
  for(let i=0;i<n;i++) d[i]=(Math.random()*2-1)*(1-i/n);
  const src=ctx.createBufferSource(); src.buffer=buf;
  const bp=ctx.createBiquadFilter(); bp.type="bandpass"; bp.frequency.value=Math.min(8000,f*4); bp.Q.value=0.7;
  const g=ctx.createGain(); g.gain.setValueAtTime(amt,now); g.gain.exponentialRampToValueAtTime(0.0001, now+dur);
  src.connect(bp).connect(g).connect(dest); src.start(now); src.stop(now+dur+0.01);
}

function playNote(noteId, opts={}){
  ensureAudio();
  const info = NOTE[noteId]; if(!info) return;
  const now = ctx.currentTime, f = info.freq, midi = info.midi;
  const decay = Math.max(1.6, 4.2 - (midi - 48) * 0.07);            // 낮을수록 길게(~1.6~4.2s)

  const voice = ctx.createGain();
  voice.gain.setValueAtTime(0.0001, now);
  voice.gain.exponentialRampToValueAtTime(0.9, now + 0.004);        // 빠른 플럭 어택(~4ms)
  voice.gain.exponentialRampToValueAtTime(0.0001, now + decay);     // 긴 지수 감쇠

  const lp = ctx.createBiquadFilter(); lp.type="lowpass"; lp.Q.value=0.5;
  lp.frequency.setValueAtTime(Math.min(13000, f*16), now);
  lp.frequency.exponentialRampToValueAtTime(Math.max(800, f*3), now + decay);

  let out = lp;
  if(ctx.createStereoPanner && typeof opts.pan === "number"){
    const pan = ctx.createStereoPanner();
    pan.pan.value = Math.max(-0.7, Math.min(0.7, opts.pan));
    lp.connect(pan); out = pan;
  }
  out.connect(master); out.connect(reverbSend);

  // 하프 = 정수 배음. 높은 배음일수록 더 빨리 사라짐(밝은 어택 → 따뜻한 여운)
  [[1,1.00],[2,0.55],[3,0.34],[4,0.20],[5,0.13],[6,0.08],[7,0.05]].forEach(([h,g])=>{
    if(f*h > 15000) return;
    const o = ctx.createOscillator(); o.type="sine"; o.frequency.value=f*h;
    const og = ctx.createGain(); og.gain.setValueAtTime(g, now);
    og.gain.exponentialRampToValueAtTime(0.0001, now + decay * (1/Math.sqrt(h)));
    o.connect(og).connect(voice); o.start(now); o.stop(now + decay + 0.08);
  });
  voice.connect(lp);
  makePluckNoise(now, voice, f);
}
```
> 더 사실적인 소리는 ① 음별 샘플(.wav/.mp3)을 로드해 `playNote`만 교체하거나, ② **Karplus-Strong**(튜닝된 딜레이 + 로우패스 피드백) 물리 모델링을 `AudioWorklet`으로 추가합니다(18번). 기본 합성만으로도 충분히 하프답습니다.

---

## 13. 공통 트리거 & 시각 피드백

키보드·마우스·터치·글리산도를 **`triggerNote(noteId)` 하나로** 모읍니다.

```js
const state = { currentType: null, lastPlayedAt: new Map(), dragging: false };

function triggerNote(noteId, opts={}){
  const type = state.currentType; if(!type) return;
  const notes = HARPS[type].notes;
  if(!notes.includes(noteId)) return;
  const idx = notes.indexOf(noteId), n = notes.length;
  const pan = n > 1 ? (idx/(n-1))*1.4 - 0.7 : 0;   // 왼(낮음) ~ 오른(높음)
  playNote(noteId, { pan });
  flashString(noteId);
  showRecentNote(noteId);
  updateAriaLive(noteId);
}

function flashString(noteId){
  const el = document.querySelector(`.string-item[data-note="${noteId}"]`);
  if(!el) return;
  el.classList.remove("is-playing"); void el.getBoundingClientRect();  // 애니메이션 재시작
  el.classList.add("is-playing");
  setTimeout(()=>el.classList.remove("is-playing"), 520);
}

function labelOf(noteId){ const n=NOTE[noteId]; return n ? `${n.ko} (${noteId}, ${n.num})` : noteId; }
function shortcutLabelOf(noteId){
  const deg = { C:"1",D:"2",E:"3",F:"4",G:"5",A:"6",B:"7" }[noteId[0]];
  const oct = +noteId.slice(1);
  return oct===3 ? `Ctrl+${deg}` : oct===5 ? `Shift+${deg}` : `${deg}`;
}
function updateAriaLive(noteId){ const l=document.querySelector("#ariaLive"); if(l) l.textContent=`${labelOf(noteId)} 연주`; }
function showRecentNote(noteId){
  const b=document.querySelector("#recentNote"); if(!b) return;
  const n=NOTE[noteId];
  b.textContent = `${n.ko} · ${noteId} · ${shortcutLabelOf(noteId)}`;
  b.classList.remove("show"); void b.offsetWidth; b.classList.add("show");
}
```
- 연주 피드백: 해당 현이 `.is-playing`(짧은 strum 떨림 + 앰버 글로우) 약 0.5s. **사진은 변형 금지**(현 오버레이만 반응).
- 소리는 **키업에서 끊지 않고 자연 감쇠**. 움직임은 잔잔하게, `prefers-reduced-motion`이면 글로우만.

---

## 14. 글리산도(드래그 연주) — 강력 권장

하프다운 연주감을 위해 현 위를 손가락/마우스로 쓸면 지나간 현들이 차례로 울립니다.

```js
const GLISS_COOLDOWN = 85;   // 같은 현 중복 발음 방지(ms)

function canGliss(noteId){
  const now = performance.now(), last = state.lastPlayedAt.get(noteId) || 0;
  if(now - last < GLISS_COOLDOWN) return false;
  state.lastPlayedAt.set(noteId, now); return true;
}
function setupGlissando(){
  const layer = document.querySelector(".string-layer");
  layer.addEventListener("pointerdown", () => { state.dragging = true; });
  layer.addEventListener("pointermove", e => {
    if(!state.dragging) return;
    const el = document.elementFromPoint(e.clientX, e.clientY);
    const noteId = el?.closest?.(".string-item")?.dataset?.note;
    if(noteId && canGliss(noteId)) triggerNote(noteId, { source:"glissando" });
  });
  ["pointerup","pointercancel","pointerleave"].forEach(evt =>
    layer.addEventListener(evt, () => { state.dragging = false; }));
}
```
- `pointerdown`에서 드래그 시작, `pointermove`에서 `elementFromPoint`로 지나는 현 감지.
- 키보드 연주에는 쿨다운을 적용하지 않습니다(매 입력마다 발음).

---

## 15. 접근성 & 반응형

- 선택 카드와 SVG 현은 **포커스 가능**(Tab 이동, Enter/Space 연주).
- 각 현 `aria-label="높은 도 (C5, 1'), 단축키 Shift+1"`처럼 계이름·음명·단축키 포함.
- 최근 연주음은 `aria-live="polite"`에 갱신.
- 터치 타깃은 얇은 현보다 훨씬 넓게(`.string-hit` `stroke-width` 18~24px).
- 하단 안내 패널은 작은 화면에서 접이식으로.
- `prefers-reduced-motion` 반영.

```css
:focus-visible { outline: 3px solid var(--glow); outline-offset: 3px; }
.sr-only { position:absolute; width:1px; height:1px; padding:0; margin:-1px; overflow:hidden; clip:rect(0,0,0,0); white-space:nowrap; border:0; }
@media (prefers-reduced-motion: reduce){
  *,*::before,*::after { animation-duration:.01ms !important; transition-duration:.01ms !important; }
}
```

---

## 16. 단계별 개발 순서 (Phase 1~9)

각 단계가 끝날 때마다 브라우저에서 직접 확인하고 넘어갑니다.

1. **뼈대 & 9:16** — `index.html`, `#app`/`#selectScreen`/`#playScreen`, 9:16 CSS, `showSelect()`/`showPlay(type)`. 빈 흰 여백 없는지 확인.
2. **이미지 자산** — `harp-15/19/21.jpg` 연결. 선택 카드·`.harp-stage`에서 왜곡 없이 보이는지, `aspect-ratio`가 실제 비율과 맞는지 확인.
3. **데이터 모델** — `NOTE`/`NOTES_15/19/21`/`MAP`/`HARPS` 작성. 음 개수 15/19/21·포개짐 검증.
4. **키 순수함수 + 테스트** — `digitOf`/`isEditableTarget`/`resolveShortcut`, `console.assert` 전부 통과. `Digit`/`Numpad` 둘 다, 무효 조합 무음.
5. **사운드 엔진** — `ensureAudio`/`makeIR`/`setVolume`/`playNote`. 첫 입력 `resume()`, 폴리포니, 낮은음 길게/높은음 맑게.
6. **선택 화면** — 카드 배치 + hover/focus/전환 + `aria-label`/키보드 선택.
7. **연주 화면 레이아웃** — 상단바(뒤로/악기명/볼륨), 메인(`.harp-stage`+`.harp-photo`+`.string-layer`), 하단(키 안내/최근음/키표 토글).
8. **SVG 현 오버레이 + 보정** — `makeStringPositions`/`renderStrings`, `preserveAspectRatio="none"`, 클릭/터치/Enter로 `triggerNote`, `?calibrate=1`로 좌표 맞춤, C현 빨강/F현 파랑.
9. **마감/검수** — 글리산도, `aria-live`/포커스 링/`prefers-reduced-motion`, 콘솔 오류 0, 다양한 화면에서 왜곡·오버레이 어긋남·패널 겹침 확인, 17번 체크리스트 전체 통과.

---

## 17. 완성 체크리스트

### 화면 / 이미지
- [ ] 첫 화면에서 15음/19음/21음 카드 3개가 보인다.
- [ ] 카드를 클릭하면 연주 화면으로 전환되고, `← 다시 선택`으로 돌아간다.
- [ ] 앱이 9:16 비율로 뷰포트를 최대한 채운다(데스크톱은 중앙 패널+레터박스).
- [ ] 하프 사진이 `object-fit: contain`으로 **왜곡·잘림 없이** 표시된다.
- [ ] 연주 중 사진 자체는 흔들리거나 확대되지 않는다.
- [ ] SVG 현 오버레이가 화면 크기가 바뀌어도 사진과 정확히 함께 스케일된다(포트레이트 포함).

### 키보드 매핑
- [ ] 15음은 `G3~G5` 15개, 19음은 `E3~B5` 19개, 21음은 `C3~B5` 21개만 난다.
- [ ] 숫자 단독 `1~7`은 항상 `C4~B4`.
- [ ] `Shift+1~7`은 `C5~B5` 중 모델에 있는 음만, `Ctrl+1~7`은 `C3~B3` 중 모델에 있는 음만 난다.
- [ ] 15음 `Ctrl+4`(F3)·`Shift+6`(A5)은 무음, 19음 `Ctrl+2`(D3)는 무음.
- [ ] `같은 숫자=같은 계이름`(예: `5`는 어디서나 '솔') 이 지켜진다.
- [ ] 단독 `8/9/0`, `Ctrl+Shift`, `Alt`, `Meta`, 없는 조합은 무음. `Numpad`도 동일.
- [ ] 꾹 눌러도 연타 안 됨(`event.repeat`). 유효 키에서만 탭 전환이 차단된다.

### 마우스 / 터치 / 글리산도
- [ ] 현을 클릭/터치하면 해당 음이 나고, 얇은 현 주변을 눌러도 잘 인식된다.
- [ ] 현 위를 드래그하면 글리산도가 자연스럽게 나고, 같은 현이 과하게 중복 발음되지 않는다.
- [ ] 누른 현에 금빛 글로우/떨림이 잠깐 표시된다. C현 빨강·F현 파랑 점이 보인다.

### 사운드
- [ ] 첫 입력에서 오디오가 시작되고, 폴리포니(겹침)가 된다.
- [ ] 낮은음은 길고 따뜻하게, 높은음은 맑지만 너무 날카롭지 않다. 은은한 리버브가 있다.
- [ ] 볼륨 슬라이더가 즉시 반영되고, 키업에서 소리가 끊기지 않고 자연 감쇠한다.

### 접근성 / 품질
- [ ] 선택 카드·현에 포커스 링이 보이고, 현 `aria-label`에 계이름/음명/단축키가 포함된다.
- [ ] 최근 연주음이 `aria-live`에 갱신된다. `prefers-reduced-motion`에서 움직임이 최소화된다.
- [ ] 콘솔 오류 0, 순수함수 테스트 전부 통과.

---

## 18. 확장 아이디어
- 숫자악보 입력 → 자동 연주(반짝반짝 작은별·생일축하 등 연습 모드).
- **6옥타브 확장**: 19현 사진 라벨(F3~C6)을 살리려면 `-`,`=` 또는 `Alt`+숫자를 6옥타브 전용으로 추가.
- 현 라벨 표시/숨김, 초보자 모드(다음 칠 현 하이라이트), 녹음/재생, 메트로놈.
- 리버브/딜레이/톤 밝기 슬라이더, 실물 하프 샘플 교체, **Karplus-Strong** 물리 모델링.
- 튜닝 프리셋(C/G/F Major, Pentatonic, Lullaby), PWA 설치, 좌표 편집 모드(드래그 후 JSON 내보내기), 양손 색상 가이드.

---

## 부록 A. 그대로 복붙하는 압축 프롬프트

```text
키보드와 마우스/터치로 연주하는 "미니하프 웹앱"을 외부 라이브러리 없이 단일 index.html(HTML+CSS+Vanilla JS) + assets/ 이미지 폴더로 만들어줘. 사운드는 Web Audio API 합성.

[화면]
- 화면1: 15음/19음/21음 미니하프 카드 선택.
- 화면2: 선택 하프 사진 + SVG 현 오버레이. 상단=← 다시 선택/악기명/볼륨, 하단=키 안내+최근 연주음 배지+키표 토글.

[이미지] assets/harp-15.jpg, harp-19.jpg, harp-21.jpg. 모두 object-fit:contain으로 원본 비율 유지(cover/왜곡/잘림 금지). 연주 중 사진은 흔들거나 확대하지 말고 효과는 SVG 오버레이에만.

[9:16] #app { position:relative; width:min(100vw,calc(100dvh*9/16)); height:min(100dvh,calc(100vw*16/9)); overflow:hidden; } .screen{position:absolute;inset:0;}로 전환. 데스크톱은 중앙 9:16 패널+딥그린/우드 레터박스.

[음역(★ 키 규칙에서 도출, 포개짐)]
- 숫자 1~7 = 도레미파솔라시(C~B), 수식키 = 옥타브: Ctrl=3옥타브, 단독=4옥타브, Shift=5옥타브 → 전체 C3~B5(21음).
- 15음 = G3~G5, 19음 = E3~B5, 21음 = C3~B5 (15 ⊂ 19 ⊂ 21). 같은 숫자=같은 계이름.
- MAP.LOW{1:C3..7:B3}, MAP.MID{1:C4..7:B4}, MAP.HIGH{1:C5..7:B5}. NOTE는 midi에서 freq=440*2^((midi-69)/12) 계산, 한글 계이름·자음보 포함.

[키 입력] 숫자 단독=C4~B4, Shift+=C5~B5, Ctrl+=C3~B3. 선택 악기에 있는 음만 연주(없으면 무음). 단독 8/9/0·Ctrl+Shift·Alt·Meta 무음. 반드시 event.code(Digit/Numpad 둘 다), event.repeat 무시, input/textarea/contenteditable 포커스 중 연주 금지, 유효 키에서만 preventDefault. resolveShortcut(e,type)->noteId|null 순수함수 + console.assert로 포개짐(15⊂19⊂21)과 "5는 항상 솔" 검증.

[현 오버레이] .harp-stage(aspect-ratio=이미지 비율) 안에 img.harp-photo + svg.string-layer(viewBox 0 0 100 100, preserveAspectRatio="none"). 현마다 g + line 3개(visible/glow/투명 hit). hit는 pointer-events:stroke, 모든 선 vector-effect:non-scaling-stroke(stroke-width는 px: 현1.6/글로우2.4/히트22). C현 빨강·F현 파랑. 좌표는 geo.top/bottom 베지어 곡선을 음 index로 보간 + OVERRIDES 보정, ?calibrate=1에서 클릭 좌표를 콘솔 출력. 클릭/터치/Enter로 triggerNote, 현 위 드래그 글리산도(현별 ~85ms 쿨다운).

[사운드] AudioContext 1회 생성, 첫 입력 resume(). master gain+볼륨 슬라이더+ConvolverNode 합성 리버브. playNote(noteId,{pan})=정수 배음 파셜(1×~7×, 높은 배음일수록 빨리 감쇠)+짧은 밴드패스 pluck noise+밝게 시작해 닫히는 lowpass+빠른 어택(~4ms)/긴 지수 감쇠. 낮은음 더 길게, StereoPanner로 낮음=왼쪽/높음=오른쪽. 폴리포니, 키업에서 자연 감쇠.

[상호작용/접근성] triggerNote가 키보드/마우스/터치/글리산도 공통. 누른 현 금빛 글로우+짧은 떨림(사진은 변형 금지). 최근음 배지 "높은 도 · C5 · Shift+1", aria-live 갱신. 카드·현 포커스 가능, aria-label에 계이름/음명/단축키. prefers-reduced-motion 반영.

[마감] 완료 후 실행 방법·주요 구현·검수 체크리스트 요약. 콘솔 오류 없이 동작.
```

---

## 부록 B. (선택) React + TypeScript 구조

단일 파일 대신 유지보수형으로 만들 경우. 음역·키 매핑·9:16·이미지 무왜곡·SVG 현 오버레이 규칙은 본문과 동일하게 지킵니다.

```text
src/
  data/harps.ts                 # NOTE, NOTES_15/19/21, MAP, HARPS(geo 포함), OVERRIDES, 타입
  utils/keyboard.ts             # digitOf, isEditableTarget, resolveShortcut (+ keyboard.test.ts)
  utils/geometry.ts             # quadPoint, makeStringPositions, stringColor
  audio/MiniHarpAudioEngine.ts  # ensureAudio, setVolume, playNote(리버브/플럭/팬)
  pages/SelectScreen.tsx        # 15/19/21 선택 카드
  pages/PlayScreen.tsx          # 선택 하프 사진 + SVG 현 오버레이
  components/HarpStage.tsx, StringLine.tsx, KeyboardGuide.tsx, RecentNote.tsx
  styles/global.css             # 9:16 풀스크린, .harp-stage, .string-* 등
public/assets/                  # harp-15.jpg, harp-19.jpg, harp-21.jpg, icon.png
```
- 라우팅 `/`, `/play/15`, `/play/19`, `/play/21` (URL이 source of truth, 잘못된 값은 선택 화면으로).
- `resolveShortcut`는 React 상태에 의존하지 않는 순수 함수로 유지, **Vitest**로 부록의 테스트 검증.
- `AudioContext`는 싱글턴으로(리렌더마다 새로 만들지 않음). `geo`/`OVERRIDES`는 좌표 편집을 위해 데이터 파일로 분리.

---

*이 명세대로 만들면 9:16 화면을 가득 채우고, 15/19/21음 미니하프 사진을 왜곡 없이 그대로 보여주며, 그 위 투명 SVG 현과 숫자키/Shift/Ctrl 매핑(1~7=도레미파솔라시, 수식키=옥타브)·마우스·터치·글리산도로 정확히 연주되고, 빠른 어택·정수 배음·맑은 잔향의 하프다운 소리가 나는 우아한 연주 웹앱이 완성됩니다.* 🎵🎶
