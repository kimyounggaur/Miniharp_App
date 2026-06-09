# 🎶 미니하프 연주 웹앱 — 통합 바이브코딩 프롬프트 (FINAL · 15현/17현/21현)

> 이 문서는 첨부된 `텅드럼_연주_웹앱_바이브코딩_프롬프트_FINAL.md`의 장점인 **9:16 풀스크린**, **원본 이미지 무왜곡**, **선택 화면 → 연주 화면 구조**, **키보드 매핑 순수함수**, **Web Audio 합성 사운드**, **좌표 보정 도구**, **단계별 개발/검수 체크리스트**를 미니하프에 맞게 재구성한 최종 바이브코딩 프롬프트입니다.
>
> 사용자가 요청한 핵심 규칙을 최우선으로 반영합니다.
>
> 1. 첫 번째 웹페이지는 **15음 미니하프 / 17음 미니하프 / 21음 미니하프 중 하나를 선택하는 화면**.
> 2. 두 번째 웹페이지는 **선택된 미니하프를 연주하는 화면**.
> 3. `숫자키` 단독은 가운데 음역, **C5 이상의 높은음은 `Shift`+숫자키**, **B3 이하의 낮은음은 `Ctrl`+숫자키**로 연주.
> 4. 마우스/터치로 현을 직접 튕겨도 같은 음이 연주.
> 5. 원본 미니하프 사진은 **왜곡·잘림 없이 그대로 표시**하고, 그 위에 **SVG 투명 현 히트박스**를 얹는다.
>
> 질문에 적힌 `Shitf`는 키보드 수식키 `Shift`의 오타로 해석해 명세에 반영합니다.

---

## 0. 이 문서 사용법 & 이미지 자산 준비 (먼저 읽기 ★)

1. 이 마크다운 전체를 복사해 AI 코딩 도구(Claude, Cursor, v0, Bolt, Lovable, Replit Agent, Gemini 등)에 붙여넣습니다.
2. 다음처럼 요청합니다.

   > 위 명세대로 **단일 `index.html` + `assets/` 폴더** 구조의 15현/17현/21현 미니하프 연주 웹앱을 완성해줘. 외부 라이브러리 없이 HTML/CSS/Vanilla JS와 Web Audio API만 사용해줘. 원본 미니하프 이미지는 왜곡 없이 표시하고, SVG 투명 현 히트박스로 키보드·마우스·터치 연주가 되게 해줘.

3. 결과를 받은 뒤 **17번 완성 체크리스트**로 검수하고, 어긋나는 항목을 다시 수정 요청합니다.

### 이미지 자산 파일명 규칙 (★ 반드시 영문 파일명으로 정리)

한글·공백·괄호가 들어간 파일명은 배포 환경에서 경로 오류가 생기기 쉽습니다. 아래처럼 단순 영문명으로 바꿔 `assets/`에 넣고 코드에서는 영문명만 참조합니다.

| 업로드한 원본 파일 | 코드에서 쓸 파일명 | 용도 | 비고 |
|---|---|---|---|
| `15현 하프 2.jpg` | `assets/harp-15.jpg` | 15현 선택 카드 + 15현 연주 화면 | 실제 15현 사진 |
| `19현 하프 2.jpg` | `assets/harp-17.jpg` | 17현 선택 카드 + 17현 연주 화면 | 현재는 19현 사진이므로 **17현 실제 사진으로 교체 권장**. 프로토타입에서는 17현 좌표만 사용 |
| `미니하프(크몽)[실사]23현.JPG` | `assets/harp-21.jpg` | 21현 선택 카드 + 21현 연주 화면 | 현재는 23현 사진이므로 **21현 실제 사진으로 교체 권장**. 프로토타입에서는 21현 좌표만 사용 |
| 선택 아이콘 | `assets/icon.png` | 파비콘/PWA 아이콘 | 선택 사항 |

**중요 이미지 규칙**

- 사진은 앱의 핵심 비주얼이므로 새로 그리거나 대체하지 말고 **원본 사진을 최대한 그대로** 사용합니다.
- 모든 사진은 **종횡비를 절대 변경하지 않습니다.** `object-fit: contain`을 사용하고, `cover`로 잘라내지 않습니다.
- 연주 효과 때문에 사진 자체를 흔들거나 확대하지 않습니다. 현 글로우·진동·파동은 **오버레이 SVG**에서만 처리합니다.
- 17현/21현 실제 사진을 확보하면 파일명만 `harp-17.jpg`, `harp-21.jpg`로 교체하고, 필요 시 `GEOMETRY` 또는 `OVERRIDES` 좌표만 보정합니다.

---

## 1. 프로젝트 개요 & 핵심 요구사항

브라우저에서 **키보드와 마우스/터치로 미니하프를 연주**하는 인터랙티브 웹앱을 만듭니다.

- **화면 1 — 선택 화면**: `15현 미니하프` / `17현 미니하프` / `21현 미니하프` 중 하나를 카드로 선택합니다.
- **화면 2 — 연주 화면**: 선택한 미니하프 사진이 9:16 화면 안에 크게 표시되고, 사진 위에 투명 SVG 현 히트박스를 얹습니다.
- **입력 규칙(핵심 ★)**
  - `숫자키` 단독 = 가운데 음역 `C4~B4`.
  - `Shift`+숫자키 = 높은음 `C5` 이상.
  - `Ctrl`+숫자키 = 낮은음 `B3` 이하.
- **사운드**: Web Audio API 합성. mp3 없이 동작하되, 나중에 실제 하프 샘플로 쉽게 교체할 수 있도록 `playNote(noteId, opts)` 인터페이스로 분리합니다.
- **톤**: 원목 미니하프의 따뜻함, 맑은 현 울림, 부드러운 잔향, 명상적이고 동화적인 분위기.

### 확정 튜닝(기본 프리셋)

실제 상품마다 튜닝이 다를 수 있으므로, 이 웹앱의 기본값은 **C Major 다이아토닉**으로 고정합니다. 실제 보유 악기의 음역이 다르면 `NOTES_15/17/21`만 교체하면 됩니다.

| 모델 | 기본 음역 | 음 개수 | 음 배열(낮은음 → 높은음) |
|---|---:|---:|---|
| 15현 | G3~G5 | 15 | `G3 A3 B3 C4 D4 E4 F4 G4 A4 B4 C5 D5 E5 F5 G5` |
| 17현 | F3~A5 | 17 | `F3 G3 A3 B3 C4 D4 E4 F4 G4 A4 B4 C5 D5 E5 F5 G5 A5` |
| 21현 | F3~E6 | 21 | `F3 G3 A3 B3 C4 D4 E4 F4 G4 A4 B4 C5 D5 E5 F5 G5 A5 B5 C6 D6 E6` |

> 이 튜닝은 사용자가 요청한 `Ctrl = B3 이하`, `숫자 단독 = C4~B4`, `Shift = C5 이상` 구조에 맞게 세 음역이 자연스럽게 나뉘도록 설계한 기본 프리셋입니다.

---

## 2. 기술 스택 & 결과물 형태

- **단일 `index.html`**: HTML + CSS + Vanilla JS를 한 파일에 작성합니다.
- **이미지 폴더**: `assets/harp-15.jpg`, `assets/harp-17.jpg`, `assets/harp-21.jpg`, 선택적으로 `assets/icon.png`.
- **사운드**: Web Audio API만 사용합니다.
  - `AudioContext`
  - `OscillatorNode`
  - `GainNode`
  - `BiquadFilterNode`
  - `ConvolverNode`
  - `StereoPannerNode`
  - 짧은 노이즈 버스트 및 합성 임펄스 리버브
- **외부 라이브러리 금지**: React, jQuery, Tone.js, Howler.js 등 사용하지 않습니다. 단, 부록 B에 React+TypeScript 선택 구조를 별도로 제공합니다.
- **코드 품질 원칙**
  - 음/현/키 매핑/이미지/좌표는 `NOTE`, `NOTES_15/17/21`, `MAP`, `HARP_CONFIGS`, `GEOMETRY`, `OVERRIDES` 같은 데이터 객체로 분리합니다.
  - 키 입력 → 음 변환은 테스트 가능한 순수 함수 `resolveShortcut()`으로 구현합니다.
  - 키보드·마우스·터치·드래그 글리산도는 모두 동일한 `triggerNote(noteId)` 경로를 사용합니다.
  - 접근성: 실제 버튼 또는 focusable SVG 요소, `aria-label`, `aria-live`, 포커스 링을 반영합니다.

---

## 3. ★ 화면/레이아웃 규격 — 9:16 풀스크린

앱은 세로형 악기인 미니하프에 맞게 **9:16 비율의 한 화면**을 기준으로 합니다.

- 루트 `#app`은 항상 9:16 비율을 유지하며 뷰포트 안에서 최대 크기로 중앙 배치합니다.
- 모바일 세로 화면에서는 화면을 거의 꽉 채우고, 데스크톱/가로 화면에서는 9:16 패널이 가운데 배치되며 바깥은 우드/딥그린 계열 레터박스 배경으로 채웁니다.
- 모바일 주소창 대응으로 `100dvh`를 사용하고, 구형 브라우저 폴백으로 `100vh`도 고려합니다.
- 화면 1·2는 `#app` 안에서 `position:absolute; inset:0;`으로 겹친 뒤 `hidden` 토글로 전환합니다.
- 빈 흰 여백 금지. 남는 공간은 테마 배경으로 채웁니다.

**레퍼런스 CSS**

```css
* { box-sizing: border-box; }
html, body { margin: 0; min-height: 100%; touch-action: manipulation; }
body {
  display: grid;
  place-items: center;
  min-height: 100dvh;
  background:
    radial-gradient(circle at 50% 0%, rgba(233, 198, 139, .20), transparent 38%),
    linear-gradient(180deg, #1d261d 0%, #0e1713 62%, #080d0b 100%);
  font-family: "Pretendard", "Apple SD Gothic Neo", system-ui, -apple-system, sans-serif;
  color: var(--ink);
  user-select: none;
}
#app {
  position: relative;
  width:  min(100vw, calc(100dvh * 9 / 16));
  height: min(100dvh, calc(100vw * 16 / 9));
  overflow: hidden;
  isolation: isolate;
  background:
    radial-gradient(circle at 50% 10%, rgba(255, 222, 163, .22), transparent 34%),
    linear-gradient(180deg, #273322 0%, #132018 55%, #0d1712 100%);
  box-shadow: 0 28px 80px rgba(0, 0, 0, .48);
}
.screen {
  position: absolute;
  inset: 0;
  display: flex;
  flex-direction: column;
  overflow: hidden;
}
.screen[hidden] { display: none; }
```

---

## 4. ★ 원본 이미지 무왜곡 + SVG 현 오버레이 정렬

미니하프는 현이 얇고 대각선으로 배치되어 있으므로, 텅드럼처럼 원형 버튼을 두는 방식보다 **SVG 라인 오버레이**가 적합합니다.

### 핵심 원칙

- 사진은 `object-fit: contain`으로 표시합니다.
- 사진과 같은 종횡비를 가진 `.harp-stage` 래퍼를 만들고, 그 위에 `viewBox="0 0 100 100"` SVG를 절대 배치합니다.
- SVG 안에 현마다 2개의 선을 둡니다.
  - `.string-visible`: 실제 현 위에 살짝 겹치는 시각용 선. C현은 빨강, F현은 파랑, 나머지는 은은한 아이보리.
  - `.string-hit`: 투명하지만 두꺼운 클릭/터치 히트라인. `pointer-events: stroke`로 현 주변을 쉽게 누를 수 있게 합니다.
- 연주 시 `.is-playing` 클래스로 해당 현에 글로우와 짧은 떨림 효과를 줍니다.
- 사진 자체는 절대 흔들거나 확대하지 않습니다.

**레퍼런스 CSS**

```css
.play-main {
  flex: 1 1 auto;
  min-height: 0;
  display: grid;
  place-items: center;
  padding: clamp(6px, 2.5cqw, 18px);
}
.harp-stage {
  position: relative;
  width: min(96%, 62vh);
  max-width: 100%;
  aspect-ratio: var(--harp-aspect, 596 / 842);
  margin: auto;
}
.harp-photo {
  width: 100%;
  height: 100%;
  object-fit: contain;
  display: block;
  user-select: none;
  -webkit-user-drag: none;
  filter: drop-shadow(0 20px 24px rgba(0, 0, 0, .34));
}
.string-layer {
  position: absolute;
  inset: 0;
  width: 100%;
  height: 100%;
  overflow: visible;
}
.string-item { outline: none; }
.string-visible {
  stroke: var(--string-color, rgba(255, 252, 235, .58));
  stroke-width: .25;
  stroke-linecap: round;
  vector-effect: non-scaling-stroke;
  opacity: .58;
  pointer-events: none;
}
.string-hit {
  stroke: rgba(255, 255, 255, .001);
  stroke-width: 22;
  stroke-linecap: round;
  vector-effect: non-scaling-stroke;
  cursor: pointer;
  pointer-events: stroke;
}
.string-glow {
  stroke: var(--glow);
  stroke-width: 1.4;
  stroke-linecap: round;
  vector-effect: non-scaling-stroke;
  opacity: 0;
  filter: drop-shadow(0 0 9px rgba(255, 213, 122, .95));
  pointer-events: none;
  transition: opacity .14s ease, stroke-width .18s ease;
}
.string-item.is-playing .string-glow {
  opacity: .95;
  stroke-width: 2.3;
}
.string-item:focus-visible .string-visible,
.string-item:focus-visible .string-glow {
  opacity: 1;
  stroke-width: 2.2;
}
@media (prefers-reduced-motion: reduce) {
  .string-glow { transition: opacity .01ms linear; }
}
```

---

## 5. 디자인 시스템

원목 미니하프의 따뜻함과 현악기의 맑은 울림을 살립니다. 전체 톤은 **따뜻한 나무색 + 크림색 + 딥그린 + 금빛 글로우**입니다.

**CSS 변수**

```css
:root {
  --bg: #132018;
  --panel: rgba(39, 51, 34, .78);
  --panel-strong: rgba(45, 37, 26, .88);
  --wood: #B9773D;
  --wood-dark: #6E3F20;
  --gold: #DCA85C;
  --glow: #FFD77A;
  --ink: #FFF5E4;
  --muted: #C9BDA7;
  --green: #5E8067;
  --red-string: #D8483E;
  --blue-string: #4E7EC8;
  --plain-string: rgba(255, 250, 232, .68);
  --shadow: rgba(0, 0, 0, .42);
}
```

**스타일 원칙**

- 카드와 패널은 `border-radius: 22px~30px`.
- 배경은 딥그린/브라운 그라데이션, 카드 안쪽은 반투명 글래스 느낌.
- 15현/17현/21현은 포인트 색으로 구분합니다.
  - 15현: 밝은 골드 `#DCA85C`
  - 17현: 따뜻한 오렌지 우드 `#C7793D`
  - 21현: 깊은 로즈우드 `#A95742`
- 클릭/연주 효과는 강한 바운스보다 **잔잔한 글로우와 짧은 현 떨림**.
- `prefers-reduced-motion` 사용자는 움직임을 최소화하고 글로우만 표시합니다.

---

## 6. [화면 1] 선택 화면 — 15현 / 17현 / 21현 카드

### 레이아웃

- 상단 16~22%: 앱 타이틀과 설명.
  - 제목: `🎶 미니하프 연주하기`
  - 부제: `숫자키로 현을 튕기고, Shift/Ctrl로 높은음과 낮은음을 연주해요.`
- 본문 70~78%: 3개의 카드.
  - 모바일 세로: 카드 3개를 세로 스택으로 배치하되 각 카드가 너무 작아지지 않게 이미지 영역을 `minmax(0, 1fr)`로 나눕니다.
  - 데스크톱/넓은 화면: 카드 3개를 가로 또는 2+1 그리드로 배치해도 되지만 `#app` 9:16 안에서 넘치지 않게 합니다.
- 하단 5~8%: 짧은 안내 텍스트.

### 카드 내용

| 카드 | 이미지 | 라벨 | 부제 |
|---|---|---|---|
| 15현 | `assets/harp-15.jpg` | `15현 미니하프` | `G3~G5 · C Major · 가볍게 시작` |
| 17현 | `assets/harp-17.jpg` | `17현 미니하프` | `F3~A5 · C Major · 균형 잡힌 음역` |
| 21현 | `assets/harp-21.jpg` | `21현 미니하프` | `F3~E6 · C Major · 넓은 고음` |

### 상호작용

- hover/focus: `translateY(-4px) scale(1.018)` + 부드러운 그림자.
- 클릭/Enter/Space: 선택 카드 글로우 → 반대 카드 살짝 흐림 → 0.3~0.4초 후 연주 화면으로 전환.
- 접근성:
  - 카드 자체를 `<button class="select-card">`로 만들거나 `role="button" tabindex="0"`를 사용합니다.
  - `aria-label="15현 미니하프 선택"`처럼 구체적으로 작성합니다.

---

## 7. [화면 2] 연주 화면 — 선택된 미니하프 연주

### 레이아웃

- **상단 바**
  - 좌측: `← 다시 선택`
  - 가운데: 현재 악기명 예) `21현 미니하프`
  - 우측: 볼륨 슬라이더 또는 볼륨 아이콘 + 접이식 슬라이더
- **메인 영역**
  - `.harp-stage` 안에 현재 악기 사진 표시.
  - 사진 위에 `.string-layer` SVG 오버레이.
  - 현을 클릭/터치하면 해당 음 재생.
  - 현 위를 드래그하면 여러 현이 순서대로 울리는 **글리산도(glissando)**를 지원합니다. 단, 같은 현이 너무 빠르게 중복 발음되지 않도록 현별 70~100ms 쿨다운을 둡니다.
- **하단 패널**
  - 요약: `숫자 = C4~B4 · Shift+숫자 = C5↑ · Ctrl+숫자 = B3↓`
  - 최근 연주음 배지: 예) `높은 도 · C5 · Shift+1`
  - `키표 보기` 토글: 모델별 유효 키 매핑 표 표시.

### 렌더링 레퍼런스

```js
function renderHarp(type) {
  const cfg = HARP_CONFIGS[type];
  state.currentType = type;

  const stage = document.querySelector('.harp-stage');
  const img = document.querySelector('.harp-photo');
  const title = document.querySelector('#currentTitle');

  stage.style.setProperty('--harp-aspect', cfg.aspect);
  img.src = cfg.image;
  img.alt = cfg.name;
  title.textContent = cfg.name;

  renderStrings(type);
  renderKeyboardGuide(type);
}
```

---

## 8. 키보드 입력 규칙 (핵심 ★)

### 기본 원칙

세 음역을 수식키로 나눕니다.

| 입력 | 음역 | 설명 |
|---|---|---|
| `숫자키` 단독 | `C4~B4` | 가운데 옥타브. `1=C4`, `2=D4`, ..., `7=B4` |
| `Shift`+숫자 | `C5` 이상 | 높은음. `Shift+1=C5`, ..., `Shift+7=B5`, `Shift+8=C6`, `Shift+9=D6`, `Shift+0=E6` |
| `Ctrl`+숫자 | `B3` 이하 | 낮은음. `Ctrl+4=F3`, `Ctrl+5=G3`, `Ctrl+6=A3`, `Ctrl+7=B3` |

**중요**

- 선택한 악기에 실제로 있는 음만 소리납니다.
- 없는 조합은 무음입니다.
- 15현은 `F3`, `A5`, `B5`, `C6`, `D6`, `E6`이 없으므로 해당 조합은 무음입니다.
- 17현은 `B5`, `C6`, `D6`, `E6`이 없으므로 해당 조합은 무음입니다.
- 21현은 위 표의 모든 기본 조합 중 `F3~E6` 범위에 해당하는 음을 사용합니다.
- `Alt` 또는 `Meta(⌘)`가 눌리면 OS/브라우저 단축키 보호를 위해 무음입니다.
- `Ctrl+Shift` 동시는 무음입니다.
- `event.repeat`은 무시해서 키를 꾹 눌러도 연타되지 않게 합니다.
- `event.code`를 사용해야 합니다. `Shift+1`에서 `event.key`가 `!`로 바뀌는 문제를 피하기 위함입니다.
- `Digit1~Digit0`과 `Numpad1~Numpad0`을 모두 지원합니다.

### 모델별 유효 키

#### 15현 `G3 A3 B3 C4 D4 E4 F4 G4 A4 B4 C5 D5 E5 F5 G5`

| 입력 | 음 | 입력 | 음 |
|---|---|---|---|
| `Ctrl+5` | G3 | `1` | C4 |
| `Ctrl+6` | A3 | `2` | D4 |
| `Ctrl+7` | B3 | `3` | E4 |
|  |  | `4` | F4 |
|  |  | `5` | G4 |
|  |  | `6` | A4 |
|  |  | `7` | B4 |
| `Shift+1` | C5 | `Shift+4` | F5 |
| `Shift+2` | D5 | `Shift+5` | G5 |
| `Shift+3` | E5 |  |  |

#### 17현 `F3 G3 A3 B3 C4 D4 E4 F4 G4 A4 B4 C5 D5 E5 F5 G5 A5`

| 입력 | 음 | 입력 | 음 |
|---|---|---|---|
| `Ctrl+4` | F3 | `1` | C4 |
| `Ctrl+5` | G3 | `2` | D4 |
| `Ctrl+6` | A3 | `3` | E4 |
| `Ctrl+7` | B3 | `4` | F4 |
|  |  | `5` | G4 |
|  |  | `6` | A4 |
|  |  | `7` | B4 |
| `Shift+1` | C5 | `Shift+4` | F5 |
| `Shift+2` | D5 | `Shift+5` | G5 |
| `Shift+3` | E5 | `Shift+6` | A5 |

#### 21현 `F3 G3 A3 B3 C4 D4 E4 F4 G4 A4 B4 C5 D5 E5 F5 G5 A5 B5 C6 D6 E6`

| 입력 | 음 | 입력 | 음 |
|---|---|---|---|
| `Ctrl+4` | F3 | `1` | C4 |
| `Ctrl+5` | G3 | `2` | D4 |
| `Ctrl+6` | A3 | `3` | E4 |
| `Ctrl+7` | B3 | `4` | F4 |
|  |  | `5` | G4 |
|  |  | `6` | A4 |
|  |  | `7` | B4 |
| `Shift+1` | C5 | `Shift+6` | A5 |
| `Shift+2` | D5 | `Shift+7` | B5 |
| `Shift+3` | E5 | `Shift+8` | C6 |
| `Shift+4` | F5 | `Shift+9` | D6 |
| `Shift+5` | G5 | `Shift+0` | E6 |

---

## 9. 음 데이터 모델

### 음 배열

```js
const NOTES_15 = [
  "G3", "A3", "B3",
  "C4", "D4", "E4", "F4", "G4", "A4", "B4",
  "C5", "D5", "E5", "F5", "G5"
];

const NOTES_17 = [
  "F3", "G3", "A3", "B3",
  "C4", "D4", "E4", "F4", "G4", "A4", "B4",
  "C5", "D5", "E5", "F5", "G5", "A5"
];

const NOTES_21 = [
  "F3", "G3", "A3", "B3",
  "C4", "D4", "E4", "F4", "G4", "A4", "B4",
  "C5", "D5", "E5", "F5", "G5", "A5", "B5",
  "C6", "D6", "E6"
];
```

### 공통 음표 테이블

`freq`는 직접 적어도 되지만, 오차와 중복을 줄이기 위해 `midi`에서 계산합니다.

```js
const freqFromMidi = midi => +(440 * Math.pow(2, (midi - 69) / 12)).toFixed(2);

const NOTE = {
  F3: { midi: 53, num: ".4",   ko: "낮은 파" },
  G3: { midi: 55, num: ".5",   ko: "낮은 솔" },
  A3: { midi: 57, num: ".6",   ko: "낮은 라" },
  B3: { midi: 59, num: ".7",   ko: "낮은 시" },

  C4: { midi: 60, num: "1",    ko: "도" },
  D4: { midi: 62, num: "2",    ko: "레" },
  E4: { midi: 64, num: "3",    ko: "미" },
  F4: { midi: 65, num: "4",    ko: "파" },
  G4: { midi: 67, num: "5",    ko: "솔" },
  A4: { midi: 69, num: "6",    ko: "라" },
  B4: { midi: 71, num: "7",    ko: "시" },

  C5: { midi: 72, num: "1'",   ko: "높은 도" },
  D5: { midi: 74, num: "2'",   ko: "높은 레" },
  E5: { midi: 76, num: "3'",   ko: "높은 미" },
  F5: { midi: 77, num: "4'",   ko: "높은 파" },
  G5: { midi: 79, num: "5'",   ko: "높은 솔" },
  A5: { midi: 81, num: "6'",   ko: "높은 라" },
  B5: { midi: 83, num: "7'",   ko: "높은 시" },

  C6: { midi: 84, num: "1''",  ko: "아주 높은 도" },
  D6: { midi: 86, num: "2''",  ko: "아주 높은 레" },
  E6: { midi: 88, num: "3''",  ko: "아주 높은 미" }
};

Object.values(NOTE).forEach(n => { n.freq = freqFromMidi(n.midi); });
```

### 키 매핑 데이터

```js
const MAP = {
  MID: {
    1: "C4", 2: "D4", 3: "E4", 4: "F4", 5: "G4", 6: "A4", 7: "B4"
  },
  LOW: {
    4: "F3", 5: "G3", 6: "A3", 7: "B3"
  },
  HIGH: {
    1: "C5", 2: "D5", 3: "E5", 4: "F5", 5: "G5", 6: "A5", 7: "B5",
    8: "C6", 9: "D6", 0: "E6"
  }
};
```

### 하프 설정 객체

```js
const HARP_CONFIGS = {
  "15": {
    type: "15",
    name: "15현 미니하프",
    range: "G3~G5",
    scale: "C Major",
    image: "assets/harp-15.jpg",
    accent: "#DCA85C",
    aspect: "269 / 420",   // 업로드 이미지 기준. 교체하면 실제 비율로 변경.
    notes: NOTES_15
  },
  "17": {
    type: "17",
    name: "17현 미니하프",
    range: "F3~A5",
    scale: "C Major",
    image: "assets/harp-17.jpg",
    accent: "#C7793D",
    aspect: "537 / 780",   // 현재 19현 참조 이미지 기준. 실제 17현으로 교체하면 변경.
    notes: NOTES_17
  },
  "21": {
    type: "21",
    name: "21현 미니하프",
    range: "F3~E6",
    scale: "C Major",
    image: "assets/harp-21.jpg",
    accent: "#A95742",
    aspect: "596 / 842",   // 현재 23현 참조 이미지 기준. 실제 21현으로 교체하면 변경.
    notes: NOTES_21
  }
};
```

---

## 10. 현 좌표/오버레이 모델

미니하프의 현은 보통 **낮은음이 왼쪽**, 높은음이 오른쪽에 있고, 상단 핀에서 사운드보드/브리지 방향으로 비스듬히 내려갑니다. 사진마다 기울기와 프레임이 다르므로, 좌표는 아래처럼 **생성식 + 보정값** 구조로 만듭니다.

### 기본 아이디어

- `GEOMETRY[type]`은 상단 핀 곡선과 하단 브리지 선을 정의합니다.
- `makeStringPositions(type)`이 `notes.length`만큼 현의 시작점/끝점을 계산합니다.
- 특정 현만 어긋나면 `OVERRIDES[type][noteId]`로 해당 현만 덮어씁니다.
- 실제 앱을 완성할 때는 `?calibrate=1`을 켜고 좌표를 조정합니다.

### 시작 좌표값(업로드 이미지 기반 초기값)

아래 좌표는 `.harp-stage` 기준 0~100% 좌표입니다. 실제 이미지와 약간 어긋날 수 있으므로, 최종 구현에서 사진에 맞게 보정합니다.

```js
const GEOMETRY = {
  "15": {
    // 15현 하프 2.jpg: 269x420, 세로형. 상단 핀은 살짝 곡선, 브리지는 오른쪽 대각선.
    top:    { x0: 27.0, y0: 12.0, x1: 78.0, y1: 22.5, cx: 53.0, cy: 8.5 },
    bottom: { x0: 45.0, y0: 83.0, x1: 75.0, y1: 40.0, cx: 59.0, cy: 61.0 },
    hitWidth: 22,
    visualWidth: .28
  },
  "17": {
    // 현재는 19현 하프 2.jpg를 17현 프로토타입 이미지로 쓰는 기준.
    top:    { x0: 7.0,  y0: 7.5,  x1: 82.0, y1: 11.5, cx: 45.0, cy: 2.8 },
    bottom: { x0: 48.0, y0: 92.0, x1: 85.5, y1: 38.5, cx: 66.0, cy: 64.0 },
    hitWidth: 20,
    visualWidth: .24
  },
  "21": {
    // 현재는 23현 실사 이미지를 21현 프로토타입 이미지로 쓰는 기준.
    top:    { x0: 34.0, y0: 23.0, x1: 70.0, y1: 26.0, cx: 52.0, cy: 19.0 },
    bottom: { x0: 31.5, y0: 75.5, x1: 75.0, y1: 39.0, cx: 53.5, cy: 58.0 },
    hitWidth: 18,
    visualWidth: .22
  }
};

const OVERRIDES = {
  "15": {
    // 예: G3: { x1: 27.5, y1: 12.3, x2: 45.4, y2: 82.1 }
  },
  "17": {},
  "21": {}
};
```

### 좌표 계산 함수

```js
function quadPoint(curve, t) {
  const mt = 1 - t;
  return {
    x: mt * mt * curve.x0 + 2 * mt * t * curve.cx + t * t * curve.x1,
    y: mt * mt * curve.y0 + 2 * mt * t * curve.cy + t * t * curve.y1
  };
}

function makeStringPositions(type) {
  const cfg = HARP_CONFIGS[type];
  const geo = GEOMETRY[type];
  const last = cfg.notes.length - 1;

  return cfg.notes.map((noteId, index) => {
    const t = last === 0 ? 0 : index / last;
    const top = quadPoint(geo.top, t);
    const bottom = quadPoint(geo.bottom, t);
    const o = OVERRIDES[type]?.[noteId];

    return {
      noteId,
      index,
      x1: o?.x1 ?? top.x,
      y1: o?.y1 ?? top.y,
      x2: o?.x2 ?? bottom.x,
      y2: o?.y2 ?? bottom.y,
      hitWidth: o?.hitWidth ?? geo.hitWidth,
      visualWidth: o?.visualWidth ?? geo.visualWidth
    };
  });
}

function stringColor(noteId) {
  const letter = noteId[0];
  if (letter === "C") return "var(--red-string)";   // 하프 관례: C현 빨강
  if (letter === "F") return "var(--blue-string)";  // 하프 관례: F현 파랑/검정 계열
  return "var(--plain-string)";
}
```

### SVG 현 렌더링

```js
const SVG_NS = "http://www.w3.org/2000/svg";

function svgEl(name, attrs = {}) {
  const el = document.createElementNS(SVG_NS, name);
  Object.entries(attrs).forEach(([k, v]) => el.setAttribute(k, String(v)));
  return el;
}

function renderStrings(type) {
  const svg = document.querySelector(".string-layer");
  svg.innerHTML = "";
  svg.setAttribute("viewBox", "0 0 100 100");
  svg.setAttribute("aria-label", `${HARP_CONFIGS[type].name} 현 영역`);

  const positions = makeStringPositions(type);

  positions.forEach(pos => {
    const { noteId, x1, y1, x2, y2 } = pos;
    const label = `${labelOf(noteId)}, 단축키 ${shortcutLabelOf(noteId, type)}`;

    const group = svgEl("g", {
      class: "string-item",
      tabindex: "0",
      role: "button",
      "aria-label": label,
      "data-note": noteId
    });
    group.style.setProperty("--string-color", stringColor(noteId));

    const visible = svgEl("line", {
      class: "string-visible",
      x1, y1, x2, y2
    });
    visible.style.strokeWidth = pos.visualWidth;

    const glow = svgEl("line", {
      class: "string-glow",
      x1, y1, x2, y2
    });

    const hit = svgEl("line", {
      class: "string-hit",
      x1, y1, x2, y2
    });
    hit.style.strokeWidth = pos.hitWidth;

    group.append(visible, glow, hit);

    const play = ev => {
      ev.preventDefault();
      triggerNote(noteId, { source: "pointer" });
    };

    group.addEventListener("pointerdown", play);
    group.addEventListener("keydown", ev => {
      if (ev.code === "Enter" || ev.code === "Space") play(ev);
    });

    svg.appendChild(group);
  });
}
```

### 좌표 보정 도구 `calibrate()`

개발 중 `index.html?calibrate=1`로 열면 클릭 위치를 콘솔에 출력합니다. 현 시작/끝 좌표를 맞출 때 사용합니다.

```js
function enableCalibrate() {
  const stage = document.querySelector(".harp-stage");
  stage.addEventListener("pointerdown", e => {
    if (!new URLSearchParams(location.search).get("calibrate")) return;
    const r = stage.getBoundingClientRect();
    const x = ((e.clientX - r.left) / r.width * 100).toFixed(1);
    const y = ((e.clientY - r.top) / r.height * 100).toFixed(1);
    console.log(`x:${x}, y:${y}`);
  });
}

enableCalibrate();
```

**권장 보정 순서**

1. `?calibrate=1`로 열고, 가장 낮은 현의 상단 핀과 하단 브리지 좌표를 찍습니다.
2. 가장 높은 현의 상단 핀과 하단 브리지 좌표를 찍습니다.
3. `GEOMETRY[type].top.x0/y0/x1/y1`, `bottom.x0/y0/x1/y1`에 반영합니다.
4. 가운데 현이 휘어져 어긋나면 `cx/cy`를 조정합니다.
5. 특정 몇 개 현만 어긋나면 `OVERRIDES`에 해당 음만 넣습니다.

---

## 11. 키 입력 처리 로직 (순수 함수 + 엣지케이스 + 테스트)

### 순수 함수 레퍼런스

```js
function digitOf(code) {
  const m = /^(?:Digit|Numpad)([0-9])$/.exec(code);
  return m ? m[1] : null;  // "0".."9" | null
}

function isEditableTarget(target) {
  return !!(
    target instanceof Element &&
    target.closest('input, textarea, select, [contenteditable="true"]')
  );
}

function resolveShortcut(e, type) {
  if (!HARP_CONFIGS[type]) return null;
  if (e.altKey || e.metaKey) return null;
  if (e.shiftKey && e.ctrlKey) return null;

  const d = digitOf(e.code);
  if (d === null) return null;

  const note = e.shiftKey ? MAP.HIGH[d]
             : e.ctrlKey  ? MAP.LOW[d]
             :              MAP.MID[d];

  if (!note) return null;
  return HARP_CONFIGS[type].notes.includes(note) ? note : null;
}
```

### 전역 키다운 처리

```js
window.addEventListener("keydown", e => {
  if (!state.currentType) return;
  if (e.repeat) return;
  if (isEditableTarget(e.target)) return;

  const note = resolveShortcut(e, state.currentType);
  if (!note) return;

  // 유효한 연주 키에서만 기본 동작 차단: Ctrl+숫자 브라우저 탭 전환 방지
  e.preventDefault();
  triggerNote(note, { source: "keyboard" });
});
```

### 필수 테스트 케이스

개발 중 콘솔에서 모두 통과해야 합니다.

```js
function testKey(code, mods, type) {
  return resolveShortcut({
    code,
    shiftKey: !!mods.shift,
    ctrlKey: !!mods.ctrl,
    altKey: !!mods.alt,
    metaKey: !!mods.meta
  }, type);
}

console.assert(NOTES_15.length === 15, "15현 개수 오류");
console.assert(NOTES_17.length === 17, "17현 개수 오류");
console.assert(NOTES_21.length === 21, "21현 개수 오류");
console.assert(NOTES_15.every(n => NOTE[n]), "15현 NOTE 누락");
console.assert(NOTES_17.every(n => NOTE[n]), "17현 NOTE 누락");
console.assert(NOTES_21.every(n => NOTE[n]), "21현 NOTE 누락");

console.assert(testKey("Digit1", {}, "15") === "C4");
console.assert(testKey("Digit7", {}, "21") === "B4");
console.assert(testKey("Digit8", {}, "21") === null);
console.assert(testKey("Numpad1", {}, "17") === "C4");

console.assert(testKey("Digit5", { ctrl: true }, "15") === "G3");
console.assert(testKey("Digit4", { ctrl: true }, "15") === null);    // 15현에는 F3 없음
console.assert(testKey("Digit4", { ctrl: true }, "17") === "F3");
console.assert(testKey("Digit7", { ctrl: true }, "21") === "B3");
console.assert(testKey("Digit3", { ctrl: true }, "21") === null);

console.assert(testKey("Digit1", { shift: true }, "15") === "C5");
console.assert(testKey("Digit5", { shift: true }, "15") === "G5");
console.assert(testKey("Digit6", { shift: true }, "15") === null);   // 15현에는 A5 없음
console.assert(testKey("Digit6", { shift: true }, "17") === "A5");
console.assert(testKey("Digit7", { shift: true }, "17") === null);   // 17현에는 B5 없음
console.assert(testKey("Digit7", { shift: true }, "21") === "B5");
console.assert(testKey("Digit8", { shift: true }, "21") === "C6");
console.assert(testKey("Digit9", { shift: true }, "21") === "D6");
console.assert(testKey("Digit0", { shift: true }, "21") === "E6");

console.assert(testKey("Digit4", { shift: true, ctrl: true }, "21") === null);
console.assert(testKey("Digit1", { alt: true }, "21") === null);
console.assert(testKey("Digit1", { meta: true }, "21") === null);
```

---

## 12. 사운드 엔진 — Web Audio 미니하프 합성

### 목표 음색

- 짧고 선명한 플럭 어택.
- 금속성보다 **원목 현악기 느낌**이 강한 맑은 하프 톤.
- 낮은 음은 더 길고 따뜻하게 울리고, 높은 음은 너무 날카롭지 않게 필터링합니다.
- 폴리포니: 여러 현이 겹쳐 울릴 수 있어야 합니다.
- 자연 감쇠: 키를 떼도 소리를 강제로 끊지 않습니다.
- 은은한 리버브: 방 안에서 울리는 작은 하프처럼 부드럽게 퍼집니다.

### 구조

- `ensureAudio()`
  - `AudioContext` 1회 생성.
  - 첫 사용자 입력에서 `resume()`.
  - 마스터 게인, 리버브, 출력 연결.
- `playNote(noteId, opts)`
  - 기본음 + 배음 파셜.
  - 짧은 플럭 노이즈.
  - 빠른 어택, 긴 지수 감쇠.
  - 음 높이에 따라 감쇠 시간과 필터 컷오프 조절.
  - 현 위치 기반 스테레오 팬.
- `setVolume(v)`
  - 볼륨 슬라이더와 연결.

### 레퍼런스 구현

```js
let ctx = null;
let master = null;
let reverbSend = null;

function ensureAudio() {
  if (!ctx) {
    ctx = new (window.AudioContext || window.webkitAudioContext)();

    master = ctx.createGain();
    master.gain.value = 0.82;
    master.connect(ctx.destination);

    const convolver = ctx.createConvolver();
    convolver.buffer = makeImpulseResponse(ctx, 2.4, 2.6);

    reverbSend = ctx.createGain();
    reverbSend.gain.value = 0.20;
    reverbSend.connect(convolver).connect(ctx.destination);
  }

  if (ctx.state === "suspended") ctx.resume();
}

function makeImpulseResponse(ctx, seconds = 2.4, decay = 2.6) {
  const rate = ctx.sampleRate;
  const length = Math.max(1, Math.floor(rate * seconds));
  const buffer = ctx.createBuffer(2, length, rate);

  for (let ch = 0; ch < 2; ch++) {
    const data = buffer.getChannelData(ch);
    for (let i = 0; i < length; i++) {
      data[i] = (Math.random() * 2 - 1) * Math.pow(1 - i / length, decay);
    }
  }
  return buffer;
}

function setVolume(value) {
  ensureAudio();
  const v = Math.max(0, Math.min(1, Number(value)));
  master.gain.setTargetAtTime(v, ctx.currentTime, 0.015);
}

function makePluckNoise(now, dest, freq) {
  const dur = 0.038;
  const len = Math.max(1, Math.floor(ctx.sampleRate * dur));
  const buffer = ctx.createBuffer(1, len, ctx.sampleRate);
  const data = buffer.getChannelData(0);

  for (let i = 0; i < len; i++) {
    const fade = 1 - i / len;
    data[i] = (Math.random() * 2 - 1) * fade;
  }

  const src = ctx.createBufferSource();
  src.buffer = buffer;

  const bp = ctx.createBiquadFilter();
  bp.type = "bandpass";
  bp.frequency.value = Math.min(7000, Math.max(900, freq * 3.2));
  bp.Q.value = 2.2;

  const g = ctx.createGain();
  g.gain.setValueAtTime(0.055, now);
  g.gain.exponentialRampToValueAtTime(0.0001, now + dur);

  src.connect(bp).connect(g).connect(dest);
  src.start(now);
  src.stop(now + dur + 0.02);
}

function playNote(noteId, opts = {}) {
  ensureAudio();
  const info = NOTE[noteId];
  if (!info) return;

  const now = ctx.currentTime;
  const freq = info.freq;
  const midi = info.midi;

  // 낮은 음일수록 더 오래 울림
  const decay = Math.max(1.6, 4.6 - (midi - 53) * 0.075);

  const voice = ctx.createGain();
  voice.gain.setValueAtTime(0.0001, now);
  voice.gain.exponentialRampToValueAtTime(0.72, now + 0.006);
  voice.gain.exponentialRampToValueAtTime(0.0001, now + decay);

  const lp = ctx.createBiquadFilter();
  lp.type = "lowpass";
  lp.Q.value = 0.62;
  lp.frequency.setValueAtTime(Math.min(9000, freq * 9), now);
  lp.frequency.exponentialRampToValueAtTime(Math.max(900, freq * 3.1), now + decay * 0.75);

  let out = lp;
  if (ctx.createStereoPanner && typeof opts.pan === "number") {
    const panner = ctx.createStereoPanner();
    panner.pan.value = Math.max(-0.58, Math.min(0.58, opts.pan));
    lp.connect(panner);
    out = panner;
  }

  out.connect(master);
  out.connect(reverbSend);

  // 하프 느낌: 기본음은 부드러운 sine, 배음은 약한 triangle/sine 혼합
  const partials = [
    [1.00, 1.00, "sine"],
    [2.00, 0.34, "triangle"],
    [3.00, 0.18, "sine"],
    [4.02, 0.08, "triangle"],
    [5.01, 0.045, "sine"]
  ];

  partials.forEach(([mul, gain, type], i) => {
    const osc = ctx.createOscillator();
    osc.type = type;
    osc.frequency.value = freq * mul;
    osc.detune.value = i === 0 ? 0 : (i * 1.2);

    const g = ctx.createGain();
    g.gain.setValueAtTime(gain, now);
    g.gain.exponentialRampToValueAtTime(0.0001, now + decay * (i === 0 ? 1 : 0.72));

    osc.connect(g).connect(voice);
    osc.start(now);
    osc.stop(now + decay + 0.08);
  });

  makePluckNoise(now, voice, freq);
  voice.connect(lp);
}
```

> 나중에 실제 현 샘플을 쓰고 싶으면 `playNote(noteId, opts)` 내부만 샘플 재생 방식으로 교체하고, UI/키보드/좌표 로직은 그대로 유지합니다.

---

## 13. 공통 트리거와 시각 피드백

키보드·마우스·터치·글리산도는 모두 `triggerNote(noteId)` 하나로 모읍니다.

```js
const state = {
  currentType: null,
  lastPlayedAt: new Map()
};

function getStringPosition(type, noteId) {
  return makeStringPositions(type).find(p => p.noteId === noteId);
}

function triggerNote(noteId, opts = {}) {
  const type = state.currentType;
  if (!type) return;
  if (!HARP_CONFIGS[type].notes.includes(noteId)) return;

  const pos = getStringPosition(type, noteId);
  const midX = pos ? (pos.x1 + pos.x2) / 2 : 50;
  const pan = (midX - 50) / 50;

  playNote(noteId, { pan });
  flashString(noteId);
  showRecentNote(noteId);
  updateAriaLive(noteId);
}

function flashString(noteId) {
  const el = document.querySelector(`.string-item[data-note="${noteId}"]`);
  if (!el) return;
  el.classList.remove("is-playing");
  void el.getBoundingClientRect();
  el.classList.add("is-playing");
  setTimeout(() => el.classList.remove("is-playing"), 230);
}

function labelOf(noteId) {
  const n = NOTE[noteId];
  return n ? `${n.ko} (${noteId}, ${n.num})` : noteId;
}

function updateAriaLive(noteId) {
  const live = document.querySelector("#ariaLive");
  if (live) live.textContent = `${labelOf(noteId)} 연주`;
}
```

### 최근 연주음 배지

```js
function showRecentNote(noteId) {
  const badge = document.querySelector("#recentNote");
  if (!badge) return;

  const shortcut = shortcutLabelOf(noteId, state.currentType);
  const n = NOTE[noteId];
  badge.textContent = `${n.ko} · ${noteId} · ${shortcut}`;
  badge.classList.remove("show");
  void badge.offsetWidth;
  badge.classList.add("show");
}
```

### 단축키 라벨 함수

```js
function shortcutLabelOf(noteId, type) {
  const letter = noteId[0];
  const octave = Number(noteId.slice(1));
  const degree = { C: "1", D: "2", E: "3", F: "4", G: "5", A: "6", B: "7" }[letter];

  if (octave === 3) return `Ctrl+${degree}`;
  if (octave === 4) return degree;
  if (octave === 5) return `Shift+${degree}`;

  // 21현 고음 확장: C6/D6/E6
  if (octave === 6) {
    const highExtra = { C: "8", D: "9", E: "0" }[letter];
    return highExtra ? `Shift+${highExtra}` : "";
  }
  return "";
}
```

---

## 14. 글리산도(드래그 연주) 지원 — 선택이지만 강력 권장

하프다운 연주감을 위해, 현 위를 손가락/마우스로 쓸면 지나간 현들이 순서대로 울리게 합니다.

### 구현 원칙

- `pointerdown`에서 `state.dragging = true`.
- `pointermove`에서 현재 포인터가 지나가는 `.string-hit`를 감지합니다.
- 같은 현이 70~100ms 안에 중복 발음되지 않게 합니다.
- `pointerup`, `pointercancel`, `pointerleave`에서 드래그 종료.
- 키보드 연주에는 쿨다운을 적용하지 않습니다.

```js
const GLISS_COOLDOWN = 85;
state.dragging = false;

function canGliss(noteId) {
  const now = performance.now();
  const last = state.lastPlayedAt.get(noteId) || 0;
  if (now - last < GLISS_COOLDOWN) return false;
  state.lastPlayedAt.set(noteId, now);
  return true;
}

function setupGlissando() {
  const layer = document.querySelector(".string-layer");

  layer.addEventListener("pointerdown", () => {
    state.dragging = true;
  });

  layer.addEventListener("pointermove", e => {
    if (!state.dragging) return;
    const el = document.elementFromPoint(e.clientX, e.clientY);
    const group = el?.closest?.(".string-item");
    const noteId = group?.dataset?.note;
    if (noteId && canGliss(noteId)) {
      triggerNote(noteId, { source: "glissando" });
    }
  });

  ["pointerup", "pointercancel", "pointerleave"].forEach(evt => {
    layer.addEventListener(evt, () => { state.dragging = false; });
  });
}
```

---

## 15. 접근성 & 반응형

- 선택 카드와 SVG 현은 포커스 가능해야 합니다.
- 키보드 사용자는 숫자키 연주뿐 아니라, Tab으로 현을 선택하고 Enter/Space로 연주할 수 있어야 합니다.
- 각 현은 `aria-label="높은 도 (C5, 1'), 단축키 Shift+1"`처럼 음명과 단축키를 포함합니다.
- 최근 연주음은 `aria-live="polite"` 영역에 갱신합니다.
- 하단 안내 패널은 작은 화면에서 너무 커지면 접이식으로 만듭니다.
- 터치 타깃은 얇은 현보다 훨씬 넓게 잡습니다. SVG `.string-hit`의 `stroke-width`는 18~24 정도로 시작합니다.
- `prefers-reduced-motion`를 반영합니다.

```css
:focus-visible {
  outline: 3px solid rgba(255, 215, 122, .95);
  outline-offset: 3px;
}
.sr-only {
  position: absolute;
  width: 1px;
  height: 1px;
  padding: 0;
  margin: -1px;
  overflow: hidden;
  clip: rect(0, 0, 0, 0);
  white-space: nowrap;
  border: 0;
}
@media (prefers-reduced-motion: reduce) {
  *, *::before, *::after {
    animation-duration: .01ms !important;
    transition-duration: .01ms !important;
  }
}
```

---

## 16. 단계별 개발 순서 (Phase 1~9)

각 단계가 끝날 때마다 브라우저에서 직접 확인하고 다음 단계로 넘어갑니다.

### Phase 1 — 프로젝트 뼈대와 9:16 화면

- `index.html` 생성.
- `#app`, `#selectScreen`, `#playScreen` 구성.
- 9:16 CSS 적용.
- `showSelect()`, `showPlay(type)` 화면 전환 함수 작성.
- 데스크톱/모바일에서 빈 흰 여백 없이 보이는지 확인.

### Phase 2 — 이미지 자산 연결

- `assets/harp-15.jpg`, `assets/harp-17.jpg`, `assets/harp-21.jpg` 연결.
- 선택 화면 카드에서 이미지가 왜곡 없이 보이는지 확인.
- 연주 화면 `.harp-stage`의 `aspect-ratio`가 이미지 실제 비율과 맞는지 확인.

### Phase 3 — 데이터 모델 작성

- `NOTE`, `NOTES_15`, `NOTES_17`, `NOTES_21`, `MAP`, `HARP_CONFIGS` 작성.
- 모든 `NOTES_*` 음이 `NOTE`에 존재하는지 검증.
- 모델별 음 개수 15/17/21 검증.

### Phase 4 — 키 입력 순수함수와 테스트

- `digitOf()`, `isEditableTarget()`, `resolveShortcut()` 작성.
- 필수 `console.assert` 테스트 통과.
- `event.code` 기반으로 `Digit`/`Numpad` 모두 지원.
- `Ctrl+Shift`, `Alt`, `Meta`, 없는 조합은 무음.

### Phase 5 — 사운드 엔진

- `ensureAudio()`, `makeImpulseResponse()`, `setVolume()`, `playNote()` 작성.
- 첫 클릭/키 입력에서 `AudioContext.resume()`이 정상 작동하는지 확인.
- 여러 현을 빠르게 눌러도 폴리포니가 되는지 확인.
- 낮은 음이 더 길고, 높은 음이 너무 날카롭지 않은지 확인.

### Phase 6 — 선택 화면 완성

- 15/17/21 카드 배치.
- hover/focus/클릭 전환 애니메이션.
- 카드 `aria-label`과 키보드 선택(Enter/Space) 반영.

### Phase 7 — 연주 화면 레이아웃

- 상단 바: 뒤로가기, 악기명, 볼륨.
- 메인: `.harp-stage` + `.harp-photo` + `.string-layer`.
- 하단: 키 안내, 최근 연주음, 키표 토글.

### Phase 8 — SVG 현 오버레이와 좌표 보정

- `GEOMETRY`, `OVERRIDES`, `makeStringPositions()`, `renderStrings()` 작성.
- 현마다 클릭/터치/Enter/Space로 `triggerNote(noteId)` 호출.
- `?calibrate=1`로 실제 사진에 맞춰 좌표 조정.
- C현 빨강, F현 파랑, 나머지 현 아이보리 색으로 표시.

### Phase 9 — 마감/검수

- 글리산도 구현.
- `aria-live`, 포커스 링, `prefers-reduced-motion` 확인.
- 콘솔 오류 제거.
- 다양한 화면 크기에서 이미지 왜곡, 오버레이 어긋남, 하단 패널 겹침 확인.
- 17번 체크리스트 전체 통과.

---

## 17. 완성 체크리스트

### 화면 / 이미지

- [ ] 첫 화면에서 15현/17현/21현 미니하프 카드 3개가 보인다.
- [ ] 카드를 클릭하면 두 번째 연주 화면으로 전환된다.
- [ ] `← 다시 선택`으로 첫 화면으로 돌아간다.
- [ ] 앱이 9:16 비율로 뷰포트를 최대한 채운다.
- [ ] 데스크톱/가로 화면에서 중앙 9:16 패널과 레터박스 배경이 자연스럽다.
- [ ] 미니하프 사진이 `object-fit: contain`으로 왜곡·잘림 없이 표시된다.
- [ ] 사진 자체는 연주 중 흔들리거나 확대되지 않는다.
- [ ] SVG 현 오버레이가 화면 크기가 바뀌어도 사진과 함께 정확히 스케일된다.

### 키보드 매핑

- [ ] 15현은 `G3~G5` 15개 음만 난다.
- [ ] 17현은 `F3~A5` 17개 음만 난다.
- [ ] 21현은 `F3~E6` 21개 음만 난다.
- [ ] 숫자 단독 `1~7`은 항상 `C4~B4`.
- [ ] `Shift+1~7`은 `C5~B5` 중 선택 모델에 있는 음만 난다.
- [ ] 21현에서 `Shift+8=C6`, `Shift+9=D6`, `Shift+0=E6`이 동작한다.
- [ ] `Ctrl+4=F3`, `Ctrl+5=G3`, `Ctrl+6=A3`, `Ctrl+7=B3` 중 선택 모델에 있는 음만 난다.
- [ ] 15현에서 `Ctrl+4`는 무음이다.
- [ ] 17현에서 `Shift+7`은 무음이다.
- [ ] 단독 `8/9/0`, 없는 `Shift`/`Ctrl` 조합, `Ctrl+Shift`, `Alt`, `Meta`는 무음이다.
- [ ] `Numpad1~0`도 동일하게 동작한다.
- [ ] 키를 꾹 눌러도 `event.repeat`으로 연타되지 않는다.
- [ ] `Ctrl+숫자` 유효 키에서 브라우저 탭 전환이 일어나지 않는다.

### 마우스 / 터치 / 글리산도

- [ ] 현을 클릭하거나 터치하면 해당 음이 난다.
- [ ] 얇은 현 주변을 눌러도 잘 인식된다.
- [ ] 현 위를 드래그하면 글리산도가 자연스럽게 난다.
- [ ] 같은 현이 드래그 중 너무 과하게 중복 발음되지 않는다.
- [ ] 누른 현에 금빛 글로우가 잠깐 표시된다.

### 사운드

- [ ] 첫 입력에서 오디오가 정상 시작된다.
- [ ] 여러 음이 겹쳐 울리는 폴리포니가 된다.
- [ ] 낮은 음은 길고 따뜻하게, 높은 음은 맑지만 너무 날카롭지 않다.
- [ ] 볼륨 슬라이더가 즉시 반영된다.
- [ ] 은은한 리버브가 있다.
- [ ] 소리가 키업에서 끊기지 않고 자연 감쇠한다.

### 접근성 / 품질

- [ ] 선택 카드와 현에 포커스 링이 보인다.
- [ ] 현마다 `aria-label`에 계이름, 음명, 단축키가 포함된다.
- [ ] 최근 연주음이 `aria-live`에 갱신된다.
- [ ] `prefers-reduced-motion`에서 움직임이 최소화된다.
- [ ] 콘솔 오류가 없다.
- [ ] 순수함수 테스트가 모두 통과한다.

---

## 18. 확장 아이디어

- 숫자악보 입력 → 자동 연주.
- 반짝반짝 작은별, 작은별 변주, 생일축하곡 등 연습 모드.
- 현 라벨 표시/숨김 토글.
- 초보자 모드: 다음에 칠 현을 빛나게 표시.
- 녹음/재생.
- 메트로놈.
- 리버브/딜레이/톤 밝기 슬라이더.
- 실물 하프 샘플 교체.
- 튜닝 프리셋: C Major, G Major, F Major, Pentatonic, Lullaby mode.
- PWA 설치 지원.
- 좌표 편집 모드: 현을 드래그해 좌표 조정 후 JSON 내보내기.
- 양손 모드: 왼손/오른손 색상 가이드.
- 모바일 전체화면 안내.

---

## 부록 A. 그대로 복붙하는 압축 프롬프트

```text
키보드와 마우스/터치로 연주하는 "미니하프 웹앱"을 외부 라이브러리 없이 단일 index.html(HTML+CSS+Vanilla JS) + assets/ 이미지 폴더로 만들어줘. 사운드는 Web Audio API 합성으로 구현해줘.

[화면 흐름]
- 화면1: 15현 미니하프 / 17현 미니하프 / 21현 미니하프 중 하나를 선택하는 카드 화면.
- 화면2: 선택된 미니하프를 연주하는 화면. 상단에는 ← 다시 선택 / 악기명 / 볼륨, 중앙에는 미니하프 사진과 SVG 현 오버레이, 하단에는 키 안내와 최근 연주음 배지.

[이미지 자산]
- assets/harp-15.jpg = 15현 하프 이미지.
- assets/harp-17.jpg = 17현 하프 이미지. 현재 19현 참조 이미지가 있다면 프로토타입용으로 쓰되 실제 17현 이미지로 교체 가능하게 해줘.
- assets/harp-21.jpg = 21현 하프 이미지. 현재 23현 참조 이미지가 있다면 프로토타입용으로 쓰되 실제 21현 이미지로 교체 가능하게 해줘.
- 모든 이미지는 object-fit: contain으로 원본 비율 유지. cover 금지. 왜곡/잘림 금지. 연주 중 사진 자체는 흔들거나 확대하지 말고, 효과는 SVG 오버레이에만 줘.

[9:16 풀스크린]
- #app은 항상 9:16 비율로 뷰포트 안에서 최대 크기 중앙 배치. 모바일 세로는 꽉 차게, 데스크톱/가로는 중앙 9:16 패널 + 딥그린/우드톤 레터박스. 100dvh 사용.
- #app { width:min(100vw, calc(100dvh*9/16)); height:min(100dvh, calc(100vw*16/9)); position:relative; overflow:hidden; }
- .screen { position:absolute; inset:0; } 로 전환.

[튜닝]
- 15현: G3 A3 B3 C4 D4 E4 F4 G4 A4 B4 C5 D5 E5 F5 G5
- 17현: F3 G3 A3 B3 C4 D4 E4 F4 G4 A4 B4 C5 D5 E5 F5 G5 A5
- 21현: F3 G3 A3 B3 C4 D4 E4 F4 G4 A4 B4 C5 D5 E5 F5 G5 A5 B5 C6 D6 E6
- NOTE 객체는 midi에서 freq 계산: freq=440*2^((midi-69)/12). 한글 계이름과 숫자악보(num) 포함.

[키 입력 규칙]
- 숫자키 단독: 1=C4,2=D4,3=E4,4=F4,5=G4,6=A4,7=B4.
- Shift+숫자: Shift+1=C5,2=D5,3=E5,4=F5,5=G5,6=A5,7=B5,8=C6,9=D6,0=E6.
- Ctrl+숫자: Ctrl+4=F3,5=G3,6=A3,7=B3.
- 선택한 악기에 실제로 있는 음만 연주. 없는 조합은 무음. 단독 8/9/0 무음. Ctrl+Shift 동시, Alt, Meta는 무음.
- 반드시 event.code(Digit1~0, Numpad1~0)를 사용하고 event.key에 의존하지 마. event.repeat 무시. input/textarea/select/contenteditable 포커스 중에는 연주하지 마. 유효 키에서만 preventDefault.
- resolveShortcut(e,type)->noteId|null 순수함수로 만들고 console.assert 테스트 포함.

[현 오버레이]
- .harp-stage는 이미지 실제 비율의 래퍼. 그 안에 img.harp-photo와 svg.string-layer(viewBox 0 0 100 100)를 겹쳐.
- 각 현은 SVG group + line 3개로 렌더링: visible line, glow line, 투명 hit line. hit line은 stroke-width 18~24, pointer-events: stroke.
- C현은 빨강, F현은 파랑, 나머지는 아이보리. 클릭/터치/Enter/Space로 triggerNote(noteId). 현 위 드래그 글리산도 지원.
- GEOMETRY(type별 top/bottom 곡선) + OVERRIDES(note별 보정) 구조로 좌표를 관리. ?calibrate=1에서 클릭 좌표를 콘솔에 출력.

[사운드]
- AudioContext는 1회 생성, 첫 사용자 입력에서 resume(). master gain + 볼륨 슬라이더 + 리버브 send.
- playNote(noteId,{pan})는 기본 sine + 약한 배음 partials + 짧은 pluck noise + lowpass + 지수 감쇠. 낮은 음은 더 오래 울리고 높은 음은 맑게. StereoPanner로 현 위치 기반 좌우감. 키업에서 소리 끊지 않고 자연 감쇠. 폴리포니 지원.

[상호작용/접근성]
- triggerNote(noteId)는 키보드/마우스/터치/글리산도 모두 공통 사용.
- 누른 현은 금빛 글로우. 최근음 배지 "높은 도 · C5 · Shift+1" 표시. aria-live 갱신.
- 선택 카드와 현은 포커스 가능, aria-label에 계이름/음명/단축키 포함. prefers-reduced-motion 반영.

[마감]
- 완료 후 실행 방법, 주요 구현, 검수 체크리스트를 요약해줘. 콘솔 오류 없이 동작해야 해.
```

---

## 부록 B. 선택: React + TypeScript 구조로 만들 경우

단일 파일 대신 유지보수형 프로젝트로 만들 경우 아래 구조를 따릅니다. 단, 위의 튜닝·키 매핑·9:16·이미지 무왜곡·SVG 현 오버레이 규칙은 그대로 지킵니다.

```text
src/
  data/harps.ts                 # NOTE, NOTES_15/17/21, MAP, HARP_CONFIGS, GEOMETRY, OVERRIDES
  utils/keyboard.ts             # digitOf, isEditableTarget, resolveShortcut
  utils/geometry.ts             # quadPoint, makeStringPositions, stringColor
  audio/MiniHarpAudioEngine.ts  # ensureAudio, setVolume, playNote
  pages/SelectScreen.tsx        # 15/17/21 선택 카드
  pages/PlayScreen.tsx          # 현재 하프 연주 화면
  components/HarpStage.tsx      # 이미지 + SVG 현 오버레이
  components/StringLine.tsx     # 개별 현
  components/KeyboardGuide.tsx  # 키 안내/표
  components/RecentNote.tsx     # 최근음 배지
  styles/global.css             # 9:16, 디자인 시스템
public/assets/
  harp-15.jpg
  harp-17.jpg
  harp-21.jpg
  icon.png
```

- 라우팅은 `/`, `/play/15`, `/play/17`, `/play/21`.
- URL이 source of truth가 되게 하고, 잘못된 타입이면 선택 화면으로 보냅니다.
- `resolveShortcut()`은 React 상태에 의존하지 않는 순수 함수로 유지하고 Vitest로 테스트합니다.
- `AudioContext`는 싱글턴 클래스로 관리해 리렌더마다 새로 만들지 않습니다.
- 좌표 보정용 `calibrate` 모드는 dev-only로 둡니다.

---

*이 명세대로 만들면 15현/17현/21현 중 하나를 선택한 뒤, 선택된 미니하프 사진을 원본 비율 그대로 보여주고, 숫자키·Shift·Ctrl 조합과 마우스/터치/글리산도로 따뜻한 하프 음색을 연주할 수 있는 웹앱을 구현할 수 있습니다.* 🎶
