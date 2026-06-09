# 🎵 미니하프 연주 웹앱 — 통합 바이브코딩 프롬프트 (FINAL · 15음 / 19음 / 21음)

> 키보드와 마우스/터치로 **미니하프(현)를 뜯어 연주**하는 인터랙티브 웹앱을 만들기 위한 **결정판 명세**입니다. 첨부한 **15현·19현·23현(실사) 하프 이미지**와 **사용자 지정 키 규칙**을 그대로 반영했습니다.
>
> **이번 요청을 최우선 규격으로 못 박았습니다.**
> 1. **화면 1(선택)** → 15음 / 19음 / 21음 미니하프 중 하나 선택
> 2. **화면 2(연주)** → 선택한 하프 도면을 **9:16 화면에 무왜곡으로** 크게 띄우고, 그 위 **투명한 현(세로 히트존)** 과 키보드로 연주
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
   > 위 명세대로 **단일 `index.html` + `assets/` 폴더** 구조의 15/19/21음 미니하프 연주 웹앱을 완성해줘. 외부 라이브러리 없이 HTML/CSS/Vanilla JS와 Web Audio API만 사용해줘.
3. 결과를 받은 뒤 **16번 완성 체크리스트**로 검수하고, 어긋나는 항목을 다시 수정 요청합니다.

### 이미지 자산 파일명 규칙 (★ 반드시 이 매핑대로)

한글·공백·날짜가 들어간 파일명은 배포 환경에서 경로 오류가 잦으니, 아래처럼 **단순 영문명**으로 바꿔 `assets/`에 넣고 코드에서는 영문명만 참조합니다.

| 업로드한 원본 파일 | 코드에서 쓸 파일명 | 용도 |
|---|---|---|
| `15현_하프_2.jpg` (라미카 셀틱 미니하프) | `assets/play-15.png` | **연주 화면** 15음 본체 |
| `19현_하프_2.jpg` (라이어형, F3~C6 라벨) | `assets/play-19.png` | **연주 화면** 19음 본체 |
| `미니하프_크몽_실사_23현.JPG` (실사 렌더, 빨강/파랑 현) | `assets/play-21.png` | **연주 화면** 21음 본체 |
| (선택 카드용) `play-15/19/21.png` 재사용 또는 크롭 | `assets/char-15.png` 등 | **선택 화면** 카드 썸네일 |

> **선택 화면 카드 이미지(`char-*`)** 는 별도 캐릭터 아트가 없으므로 **연주용 본체 사진(`play-*`)을 그대로 재사용**해도 됩니다(나중에 전용 카드 이미지로 교체 가능). 코드에서는 `charImage`/`playImage`를 따로 두어 언제든 갈아끼울 수 있게 합니다.

### ⚠️ 이미지 ↔ 음역 매핑 주의사항 (반드시 확인)

- **(19현 라벨 불일치)** `play-19.png`에는 **`F3~C6`(19현)** 가 인쇄되어 있지만, 사용자 키 규칙(`Shift`=C5↑, `Ctrl`=B3↓, 단독=C4~B4)으로는 **6옥타브 `C6`를 지정할 수 없습니다**(Shift 키는 1~7뿐이라 C5~B5까지가 한계). 그래서 **앱의 19음 음역은 `E3~B5`로 매핑**합니다. 인쇄 라벨과 정확히 맞추고 싶다면 9번 `NOTES_19`/8번 `MAP`만 수정하거나, C6까지 쓰려면 `8`번 키 등 **별도 규칙을 추가**해야 합니다(기본 규칙에서 벗어남).
- **(23현 → 21음)** `play-21.png`에는 **현이 23개**로 보이지만 요청대로 **21음 하프**로 씁니다. 현 영역에 **21개 히트존을 균등 배치**(가운데 21현 기준)하고, 남는 2현은 비활성으로 둡니다. 정밀하게 맞추려면 10번 `?calibrate=1`로 21개 현을 클릭해 `xs` 좌표 배열을 만들거나, **21현 도면으로 교체**하세요. (정말 23음을 원하면 키 규칙상 21음이 최대이므로 확장 규칙이 필요합니다 — 17번 참조.)
- **(현이 사선으로 퍼짐)** 실사 렌더(`play-21`)와 셀틱형(`play-15`)은 현이 **부채꼴로 비스듬히** 퍼집니다. 10번 `field.tilt`로 히트존을 기울이거나 `calibrate`로 현 위치를 보정합니다.
- **(이미지 원본 보존)** 위 이미지들은 앱의 핵심 비주얼입니다. **새로 그리거나 대체하지 말고 원본 그대로** 사용합니다(4번 규칙). 종횡비를 바꾸지 않습니다.

### 🎨 하프 색상 관례 (지식)

실제 하프는 방향을 빠르게 찾도록 **도(C)현 = 빨강**, **파(F)현 = 파랑(또는 검정)** 으로 칠합니다. `play-21.png` 실사 렌더에도 빨강/파랑 현이 보입니다. 이 앱에서는 **투명 히트존 상단에 작은 빨강/파랑 표시점**을 찍어 같은 관례를 살립니다(7·10번).

---

## 1. 프로젝트 개요 & 핵심 요구사항

브라우저에서 **키보드와 마우스/터치로 미니하프 현을 뜯어 연주**하는 인터랙티브 웹앱을 만든다.

- **화면 1 — 선택 화면**: `15음` / `19음` / `21음` 미니하프 중 하나를 카드로 고른다.
- **화면 2 — 연주 화면**: 선택한 하프 **도면**이 9:16 화면에 크게 표시되고, 그 위 **투명한 세로 현(히트존)** 과 키보드로 연주한다.
- **입력 규칙(핵심 ★)**: `숫자키` 단독 = C4~B4 / `Shift`+숫자 = C5 이상 / `Ctrl`+숫자 = B3 이하. **1~7 = 도레미파솔라시**, 수식키 = 옥타브.
- **튜닝**: 위 확정값(15음 G3~G5, 19음 E3~B5, 21음 C3~B5, 모두 **C Major 온음계**).
- **사운드**: Web Audio API **합성**. mp3 없이 **현을 뜯는(plucked) 소리**가 나며, 추후 샘플 교체가 쉽게 구조화.
- **톤**: 맑고 우아한 하프 특유의 잔향. 따뜻한 우드/금색 현, 부드러운 글로우, 짧은 현 진동(strum) 연출.

---

## 2. 기술 스택 & 결과물 형태

- **단일 `index.html`** (HTML + CSS + Vanilla JS, 빌드 불필요) + `assets/` 이미지 폴더.
  - 더블클릭으로 바로 실행, 배포·공유 간단, 이미지 드롭만으로 교체.
- 사운드: **Web Audio API** (`AudioContext`, `OscillatorNode`, `GainNode`, `BiquadFilterNode`, `ConvolverNode`, `StereoPannerNode`, noise buffer).
- **외부 라이브러리 금지** (React, jQuery, Tone.js, Howler.js 등). 단, 부록 B에 React+TS 구조를 선택 옵션으로 제공.
- **코드 품질 원칙**
  - 음/현 데이터는 `NOTE`, `NOTES_15/19/21`, `MAP`, `HARPS` 같은 **데이터 객체로 분리**(하드코딩 분산 금지).
  - 키 입력 → 음 변환은 **테스트 가능한 순수 함수** `resolveShortcut()`로.
  - 클릭·터치·키보드는 모두 동일한 `triggerNote(noteId)` 경로를 사용한다.
  - 현 좌표는 **파라메트릭(field 기반 균등 분배)** 으로 자동 생성하고, 필요 시 `xs` 배열/`calibrate`로만 미세조정한다(앱 전체 코드 수정 금지).
  - **접근성**(`role`/`aria-label`/`aria-live`/포커스 링) 반영.

---

## 3. ★ 화면/레이아웃 규격 — 9:16 풀스크린

앱은 **세로 9:16 비율의 한 화면**을 기준으로 하고, **뷰포트를 가득 채운다.** (하프는 세로로 길어 9:16과 잘 맞는다.)

- 루트(`#app`)는 **항상 9:16 비율**을 유지하며 화면에 맞춰 최대 크기로 중앙 배치.
  - 모바일 세로: 화면 전체를 채움.
  - 데스크톱/가로: 9:16 패널이 화면 높이에 맞춰 중앙에 크게, 바깥은 어두운 우드톤 배경(레터박스).
- 모바일 주소창 대응으로 **`100dvh`** 사용(폴백 `100vh`).
- 화면 1·2는 `#app` 안에서 `position:absolute; inset:0;`로 **9:16를 가득** 채우고 `hidden`으로 토글. 빈 흰 여백 금지(테마 배경으로 채움).

**레퍼런스 CSS**
```css
* { box-sizing: border-box; }
html, body { margin: 0; height: 100%; touch-action: manipulation; }
body {
  display: grid; place-items: center;
  min-height: 100dvh;
  background: radial-gradient(circle at top, #2a1d14 0%, #120c08 64%); /* 데스크톱 레터박스 */
  font-family: "Pretendard", "Apple SD Gothic Neo", system-ui, sans-serif;
  color: var(--ink);
  user-select: none;
}
#app {
  position: relative;
  width:  min(100vw, calc(100dvh * 9 / 16));
  height: min(100dvh, calc(100vw * 16 / 9));
  overflow: hidden;
  background:
    radial-gradient(circle at 50% 14%, rgba(255,224,138,.16), transparent 36%),
    linear-gradient(180deg, #241a12 0%, var(--bg) 56%, #15100b 100%);
  box-shadow: 0 20px 60px rgba(0,0,0,.46);
  isolation: isolate;
}
.screen { position: absolute; inset: 0; display: flex; flex-direction: column; overflow: hidden; }
.screen[hidden] { display: none; }
```

> 모든 폰트·간격·이미지 크기는 `#app` 크기에 비례하도록 `%`, `vmin`, `clamp()`, `cqw`, `min()/max()`로 잡아 **어떤 화면에서도 비율이 깨지지 않게** 한다.

---

## 4. ★ 이미지 무왜곡 + 세로 현(히트존) 정렬

**0번 표의 원본 이미지를 변형 없이 그대로 표시한다.**

- 모든 이미지는 **종횡비를 절대 변경하지 않는다**(가로/세로 따로 늘이기 금지).
- 표시는 **`object-fit: contain`** → 이미지가 **잘리거나 찌그러지지 않고 전체가** 보인다. (`cover` 금지.)
- 연주 화면 도면은 **9:16 안에서 최대한 크게** 키우되 비율 유지. 남는 공간은 테마 배경으로 채운다.
- 연주 효과(현 진동/글로우) 때문에 **이미지 자체가 흔들리거나 확대/변형되면 안 된다.** 반응은 오버레이 현에서만.

**사진 + 현 오버레이 정렬 방법(중요)**
도면 위에 투명 현을 정확히 얹기 위해, **이미지와 같은 비율의 래퍼(`.harp-stage`)** 를 만들고 그 안에서 **백분율 좌표**로 세로 히트존을 배치한다. 래퍼가 이미지와 동일 비율이므로 화면 크기가 바뀌어도 현이 이미지와 함께 정확히 스케일된다.

```css
.play-main { flex: 1 1 auto; min-height: 0; display: grid; place-items: center; padding: clamp(8px,3cqw,18px); }
.harp-stage {
  position: relative;
  height: min(96%, 78vh);
  aspect-ratio: 540 / 760;   /* 선택된 하프의 HARPS[type].aspect로 교체 (이미지 실제 비율) */
  max-width: 100%;
  margin: auto;
}
.harp-photo {                /* 원본 무왜곡 */
  width: 100%; height: 100%;
  object-fit: contain;
  display: block; user-select: none; -webkit-user-drag: none;
  filter: drop-shadow(0 18px 22px rgba(0,0,0,.30));
}
.string-layer { position: absolute; inset: 0; }   /* 투명 현 레이어 */
```

> 도면마다 현이 놓인 영역과 기울기가 다르므로, 히트존은 **`HARPS[type].field`(현 영역 사각형 + 기울기) 한 곳에서만** 조정한다. 사진이 바뀌어도 앱 전체 코드를 고치지 말고 `field`(또는 `xs`/`calibrate`)만 수정한다(10번).

---

## 5. 공통 디자인 시스템

미니하프의 **맑고 우아한 공명 + 따뜻한 우드** 느낌을 살린다. 선택 화면은 친근하고 또렷하게, 연주 화면은 차분하고 몰입감 있게.

**색상(CSS 변수)**

| 토큰 | 값 | 용도 |
|---|---|---|
| `--bg` | `#1a1410` | 앱 배경(따뜻한 다크 우드) |
| `--panel` | `#271d16` | 카드/패널 배경 |
| `--gold` | `#D9B45B` | 현(금색)/주요 버튼 |
| `--gold-dark` | `#B5923F` | hover/강조 |
| `--glow` | `#FFE08A` | 연주 하이라이트(앰버) |
| `--string-c` | `#D8504A` | **도(C) 현** 표시점(빨강·하프 관례) |
| `--string-f` | `#4A7FD8` | **파(F) 현** 표시점(파랑·하프 관례) |
| `--rose` | `#C77B52` | **15음** 포인트(로즈 코퍼) |
| `--teal` | `#4FA39A` | **19음** 포인트(틸 그린) |
| `--violet` | `#8E6FC0` | **21음** 포인트(애미시스트) |
| `--ink` | `#F3ECDD` | 본문 텍스트(소프트 크림) |
| `--muted` | `#B7A48C` | 보조 텍스트 |

**스타일 원칙**: 큰 라운드(18~28px), 부드러운 확산 그림자, **우아하고 잔잔한** 마이크로 인터랙션. 화면 전환 0.3~0.4s 페이드/슬라이드. 클릭 시 `scale(.98)`, 연주 시 현이 짧게 떨리며(strum) 앰버 글로우. `prefers-reduced-motion`이면 큰 움직임을 줄이고 opacity 변화만 남긴다.

> **세 하프를 색으로 구분**: 15음=로즈, 19음=틸, 21음=바이올렛. 선택·연주 화면에서 일관 적용.

---

## 6. [화면 1] 선택 화면 — 15음 / 19음 / 21음 카드

**레이아웃(9:16 세로 가득)**
- 상단: 앱 타이틀(예: "🎵 미니하프 연주하기") + 한 줄 안내("연주할 미니하프를 골라주세요").
- 본문: **세로로 3개의 카드**(9:16 높이를 고르게 분할; 넓은 화면이면 가로 3열로 `flex-wrap`).
  - **15음 카드**: `assets/char-15.png`(무왜곡, `object-fit:contain`) + 로즈 라벨 **"15음 미니하프"** + 부제 "G3~G5 · C Major · 가볍게".
  - **19음 카드**: `assets/char-19.png` + 틸 라벨 **"19음 미니하프"** + 부제 "E3~B5 · C Major · 표준".
  - **21음 카드**: `assets/char-21.png` + 바이올렛 라벨 **"21음 미니하프"** + 부제 "C3~B5 · C Major · 넓은 음역".

**상호작용**
- hover/focus: `translateY(-4px) scale(1.025)` + 떠오르는 그림자.
- 클릭/Enter/Space: 선택 카드 강조 → 나머지 카드 흐리게(`opacity:.45`) → 클릭 지점 물결(ripple) → 0.3~0.4s 후 **연주 화면으로 페이드 전환**(선택값 `"15"|"19"|"21"` 전달).
- 접근성: 카드는 `role="button"`, `tabindex="0"`, `aria-label="15음 미니하프 선택"`. 또렷한 포커스 링.

---

## 7. [화면 2] 연주 화면 — 도면 + 투명 세로 현

**레이아웃(9:16 세로 가득)**
- **상단 바(슬림)**: 좌측 `← 다시 선택`(화면 1 복귀), 가운데 악기명("19음 미니하프"), 우측 **볼륨 슬라이더**.
- **메인(가장 크게)**: `.harp-stage` 안에 선택된 도면(`play-15/19/21.png`)을 **무왜곡·최대 크기**(4번)로 표시하고, 그 위 `.string-layer`에 **투명 세로 현 N개**를 배치.
- **하단(접이식 가능)**: **키 안내 패널** — "숫자=가운데 · ⇧+숫자=높은음 · ⌃+숫자=낮은음" 요약 + 매핑 표 토글. **최근 연주음 배지**(예: "솔 · G4 · 5") 잠깐 표시.

**오버레이 렌더링 규칙**
- `HARPS[type].notes`의 모든 음에 대해, `field` 기준으로 **왼쪽(최저음) → 오른쪽(최고음)** 순서로 투명 세로 `<button>`(현)을 균등 배치한다(10번 `renderStrings`).
- 도면에 이미 현/라벨이 그려져 있으므로 **버튼은 투명**(hover/focus 시 옅은 윤곽만 허용). 누르면 `.is-playing`으로 **현이 짧게 떨리며 앰버 글로우**.
- **도(C)현은 빨강, 파(F)현은 파랑** 표시점을 상단에 찍어 위치를 쉽게 찾게 한다.
- **도면 이미지는 절대 흔들거나 확대하지 않는다.** 연주 효과는 현 오버레이에서만.
- 접근성: 각 버튼 `aria-label="<계이름>, <음명>, 단축키 <키표시>"`(예: "솔, G4, 단축키 5" / "낮은 미, E3, 단축키 Ctrl+3"). 마우스·터치·키보드 모두 같은 `triggerNote(noteId)`.

> 현 개수는 하프 종류와 일치해야 한다(15/19/21). **키보드 연주가 1차 수단**이며 어떤 경우에도 정확히 동작해야 한다. 히트존이 도면의 실제 현과 픽셀 단위로 완벽 정합하지 않아도 된다(10번 보정).

---

## 8. 키보드 입력 규칙 (핵심 ★)

세 옥타브를 **수식키**로 구분한다.

| 입력 | 음역 | 예시 |
|---|---|---|
| `숫자키` 단독 | 가운데 4옥타브 C4~B4 | `1`=C4, `5`=G4, `7`=B4 |
| `Shift`+숫자 | 높은 5옥타브 C5 이상 | `⇧1`=C5, `⇧5`=G5, `⇧7`=B5 |
| `Ctrl`+숫자 | 낮은 3옥타브 B3 이하 | `⌃1`=C3, `⌃5`=G3, `⌃7`=B3 |

**설계 원칙 — "같은 숫자키 = 같은 계이름(도레미), 수식키 = 옥타브"**
> 1~7 = 도·레·미·파·솔·라·시(C·D·E·F·G·A·B). 수식키가 옥타브를 결정: 단독=가운데(4), `Shift`=위(5), `Ctrl`=아래(3).
> 예) **5번 키 = 항상 '솔'** → `Ctrl+5=G3`, `5=G4`, `Shift+5=G5`.
> **선택한 하프에 실제로 있는 음만 소리난다.** 없는 음 조합은 무음.

**모델별 유효 키**

#### 15음 (`G3 A3 B3 C4 D4 E4 F4 G4 A4 B4 C5 D5 E5 F5 G5`)
| 입력 | 음 | | 입력 | 음 |
|---|---|---|---|---|
| `1`~`7` | C4~B4 | | `⇧1`~`⇧5` | C5~G5 |
| `⌃5` | G3 | | `⌃6` | A3 |
| `⌃7` | B3 | | | |

- `⇧6`,`⇧7`(A5/B5)·`⌃1`~`⌃4`(C3~F3)는 15음에 없으므로 **무음**. `8/9/0` 단독 무음.

#### 19음 (`E3 F3 G3 A3 B3 C4 … B4 C5 … B5`)
| 입력 | 음 | | 입력 | 음 |
|---|---|---|---|---|
| `1`~`7` | C4~B4 | | `⇧1`~`⇧7` | C5~B5 |
| `⌃3` | E3 | | `⌃4` | F3 |
| `⌃5` | G3 | | `⌃6` | A3 |
| `⌃7` | B3 | | | |

- `⌃1`,`⌃2`(C3/D3)는 19음에 없으므로 **무음**. `8/9/0` 단독 무음.

#### 21음 (`C3 D3 E3 F3 G3 A3 B3 C4 … B4 C5 … B5`) — 모든 조합 사용
| 입력 | 음 | | 입력 | 음 |
|---|---|---|---|---|
| `1`~`7` | C4~B4 | | `⇧1`~`⇧7` | C5~B5 |
| `⌃1`~`⌃7` | C3~B3 | | | |

- `8/9/0` 단독·`Alt`/`Meta`·`Ctrl+Shift` 동시만 **무음**. 그 외 7×3=21음 전부 발음.

---

## 9. 음 데이터 (15음 / 19음 / 21음)

> **C Major 온음계 기준.** 왼쪽(긴 현)이 가장 낮은 음, 오른쪽(짧은 현)으로 갈수록 높아짐.

**음 배열**
```js
// 15음 C Major: G3~G5 (15음)
const NOTES_15 = ["G3","A3","B3","C4","D4","E4","F4","G4","A4","B4","C5","D5","E5","F5","G5"];

// 19음 C Major: E3~B5 (19음)  ※ 15음에 저음(E3 F3)·고음(A5 B5)을 더한 구조
const NOTES_19 = ["E3","F3","G3","A3","B3","C4","D4","E4","F4","G4","A4","B4","C5","D5","E5","F5","G5","A5","B5"];

// 21음 C Major: C3~B5 (21음)  ※ 3옥타브 풀레인지. 모든 키 조합 사용
const NOTES_21 = ["C3","D3","E3","F3","G3","A3","B3","C4","D4","E4","F4","G4","A4","B4","C5","D5","E5","F5","G5","A5","B5"];
```

**주파수 · 숫자악보(자음보) · 한글 계이름 — 공통 음표 테이블**
```js
const NOTE = {
  // noteId: { freq, midi, num(자음보: .숫자=낮은옥타브, 숫자'=높은옥타브), ko }
  // 낮은 옥타브(3) — Ctrl 영역
  C3:{ freq:130.81, midi:48, num:".1", ko:"낮은 도" },
  D3:{ freq:146.83, midi:50, num:".2", ko:"낮은 레" },
  E3:{ freq:164.81, midi:52, num:".3", ko:"낮은 미" },
  F3:{ freq:174.61, midi:53, num:".4", ko:"낮은 파" },
  G3:{ freq:196.00, midi:55, num:".5", ko:"낮은 솔" },
  A3:{ freq:220.00, midi:57, num:".6", ko:"낮은 라" },
  B3:{ freq:246.94, midi:59, num:".7", ko:"낮은 시" },
  // 가운데 옥타브(4) — 숫자 단독
  C4:{ freq:261.63, midi:60, num:"1",  ko:"도" },
  D4:{ freq:293.66, midi:62, num:"2",  ko:"레" },
  E4:{ freq:329.63, midi:64, num:"3",  ko:"미" },
  F4:{ freq:349.23, midi:65, num:"4",  ko:"파" },
  G4:{ freq:392.00, midi:67, num:"5",  ko:"솔" },
  A4:{ freq:440.00, midi:69, num:"6",  ko:"라" },
  B4:{ freq:493.88, midi:71, num:"7",  ko:"시" },
  // 높은 옥타브(5) — Shift 영역
  C5:{ freq:523.25, midi:72, num:"1'", ko:"높은 도" },
  D5:{ freq:587.33, midi:74, num:"2'", ko:"높은 레" },
  E5:{ freq:659.25, midi:76, num:"3'", ko:"높은 미" },
  F5:{ freq:698.46, midi:77, num:"4'", ko:"높은 파" },
  G5:{ freq:783.99, midi:79, num:"5'", ko:"높은 솔" },
  A5:{ freq:880.00, midi:81, num:"6'", ko:"높은 라" },
  B5:{ freq:987.77, midi:83, num:"7'", ko:"높은 시" }
};
const freqFromMidi = m => 440 * Math.pow(2, (m - 69) / 12);
```

---

## 10. 하프 데이터 모델 & 현 좌표 (파라메트릭 + 보정 도구)

하프는 현이 **왼쪽→오른쪽으로 일렬 배치**되므로, 텅드럼처럼 현마다 좌표를 손으로 찍지 않고 **`field`(현 영역 사각형 + 기울기)에 균등 분배**한다. 도면이 바뀌면 `field`(또는 `xs`/`calibrate`)만 고친다.

**데이터 모델**
```js
const MAP = {
  MID:  { 1:"C4", 2:"D4", 3:"E4", 4:"F4", 5:"G4", 6:"A4", 7:"B4" }, // 숫자 단독(4옥타브)
  HIGH: { 1:"C5", 2:"D5", 3:"E5", 4:"F5", 5:"G5", 6:"A5", 7:"B5" }, // Shift(5옥타브)
  LOW:  { 1:"C3", 2:"D3", 3:"E3", 4:"F3", 5:"G3", 6:"A3", 7:"B3" }  // Ctrl(3옥타브)
};

const HARPS = {
  "15": {
    type:"15", name:"15음 미니하프", range:"G3~G5", scale:"C Major",
    charImage:"assets/char-15.png", playImage:"assets/play-15.png",
    accent:"var(--rose)", aspect:"190 / 400", notes:NOTES_15,
    // 현이 놓인 영역(.harp-stage 대비 %). left=최저음 쪽, right=최고음 쪽
    field:{ left:30, right:74, top:14, bottom:88, tilt:0 }
    // xs: [..]  // (선택) 현 중심 x% 직접 지정 시 균등분배 대신 이 값 사용
  },
  "19": {
    type:"19", name:"19음 미니하프", range:"E3~B5", scale:"C Major",
    charImage:"assets/char-19.png", playImage:"assets/play-19.png",
    accent:"var(--teal)", aspect:"540 / 760", notes:NOTES_19,
    field:{ left:17, right:80, top:20, bottom:82, tilt:0 }
  },
  "21": {
    type:"21", name:"21음 미니하프", range:"C3~B5", scale:"C Major",
    charImage:"assets/char-21.png", playImage:"assets/play-21.png",
    accent:"var(--violet)", aspect:"600 / 800", notes:NOTES_21,
    // 실사 렌더는 현이 사선으로 퍼짐 → tilt로 히트존을 살짝 기울임
    field:{ left:10, right:68, top:24, bottom:92, tilt:6 }
  }
};
```

**현 좌표 자동 생성 + 렌더링**
```js
// 현 i(0-based, 0=최저음 왼쪽)의 중심 x%
function stringX(cfg, i, n){
  if (cfg.xs && cfg.xs[i] != null) return cfg.xs[i];       // 직접 지정값 우선
  const t = n > 1 ? i / (n - 1) : 0.5;
  return cfg.field.left + t * (cfg.field.right - cfg.field.left); // 균등 분배
}

function renderStrings(type){
  const cfg = HARPS[type];
  const layer = document.querySelector(".string-layer");
  layer.innerHTML = "";
  const n = cfg.notes.length, f = cfg.field;
  const slotW = (f.right - f.left) / n;                    // 현 1개가 차지하는 가로 폭
  cfg.notes.forEach((noteId, i) => {
    const btn = document.createElement("button");
    btn.type = "button"; btn.className = "string-hit"; btn.dataset.note = noteId;
    btn.style.left = `${stringX(cfg, i, n)}%`;
    btn.style.top  = `${(f.top + f.bottom) / 2}%`;          // 세로 중앙 기준
    btn.style.setProperty("--h", `${f.bottom - f.top}%`);   // 현 길이(세로)
    btn.style.setProperty("--w", `${Math.max(3.4, slotW * 0.92)}%`); // 클릭 폭(현 간격에 맞춤)
    btn.style.setProperty("--rot", `${f.tilt || 0}deg`);
    const pc = noteId.replace(/[0-9]/g, "");                // 음이름(C,D,…)
    if (pc === "C") btn.classList.add("is-c");              // 도 = 빨강 표시점
    if (pc === "F") btn.classList.add("is-f");              // 파 = 파랑 표시점
    btn.setAttribute("aria-label", `${labelOf(noteId)}, 단축키 ${shortcutLabelOf(noteId)}`);
    btn.addEventListener("pointerdown", ev => { ev.preventDefault(); triggerNote(noteId); });
    layer.appendChild(btn);
  });
}

function labelOf(noteId){ const n = NOTE[noteId]; return n ? `${n.ko} (${noteId}, ${n.num})` : noteId; }
function shortcutLabelOf(noteId){
  for (const d of [1,2,3,4,5,6,7]){
    if (MAP.MID[d]  === noteId) return `${d}`;
    if (MAP.HIGH[d] === noteId) return `Shift+${d}`;
    if (MAP.LOW[d]  === noteId) return `Ctrl+${d}`;
  }
  return "";
}
```

**현(히트존) CSS — 투명 클릭 영역 + 진동 + 색상 표시점**
```css
.string-layer { position:absolute; inset:0; }
.string-hit {
  position:absolute;
  transform: translate(-50%, -50%) rotate(var(--rot, 0deg));
  width: var(--w); height: var(--h);
  border:0; padding:0; border-radius:999px;
  background: rgba(255,255,255,.001);   /* 투명하지만 클릭/터치 가능 */
  cursor:pointer; touch-action:manipulation;
}
/* 뜯을 때 빛나는 가는 세로 현 */
.string-hit::before{
  content:""; position:absolute; left:50%; top:0; bottom:0; width:2px;
  transform:translateX(-50%); transform-origin:center; border-radius:999px; opacity:0;
  background: linear-gradient(180deg, transparent, var(--glow), transparent);
  box-shadow: 0 0 12px var(--glow), 0 0 26px rgba(255,224,138,.55);
  transition: opacity .14s ease;
}
.string-hit.is-playing::before{ opacity:1; animation: strum .5s ease-out; }
@keyframes strum{
  0%   { transform:translateX(-50%) scaleX(2.6); }
  40%  { transform:translateX(-58%) scaleX(1.7); }
  70%  { transform:translateX(-46%) scaleX(1.2); }
  100% { transform:translateX(-50%) scaleX(1); }
}
/* 도(C)=빨강, 파(F)=파랑 표시점 (실제 하프 색상 관례) */
.string-hit.is-c::after, .string-hit.is-f::after{
  content:""; position:absolute; left:50%; top:0;
  width:9px; height:9px; transform:translate(-50%,-50%); border-radius:50%; opacity:.85;
}
.string-hit.is-c::after{ background:var(--string-c); box-shadow:0 0 8px var(--string-c); }
.string-hit.is-f::after{ background:var(--string-f); box-shadow:0 0 8px var(--string-f); }
.string-hit:focus-visible{ outline:3px solid var(--glow); outline-offset:2px; }
@media (prefers-reduced-motion: reduce){ .string-hit.is-playing::before{ animation:none; } }
```

**좌표 보정 도구 `calibrate()` (개발 편의, 선택)**
연주 화면에서 `?calibrate=1`이면 활성화. **최저음 현부터 순서대로** 클릭하면 각 지점의 `x%`가 누적된 `xs` 배열로 콘솔에 출력되어, 현이 비스듬하거나 간격이 불규칙한 도면에 빠르게 맞출 수 있다.
```js
function enableCalibrate(){
  const stage = document.querySelector(".harp-stage");
  const xs = [];
  stage.addEventListener("click", e => {
    const r = stage.getBoundingClientRect();
    const x = +(((e.clientX - r.left) / r.width) * 100).toFixed(1);
    xs.push(x);
    console.log(`현 ${xs.length}: x=${x}%   →  xs:[${xs.join(", ")}]`);
  });
  console.log("[calibrate] 최저음(맨 왼쪽) 현부터 순서대로 클릭하세요. HARPS[type].xs 에 붙여넣으면 됩니다.");
}
if (new URLSearchParams(location.search).get("calibrate")) enableCalibrate();
```

> `field`로 대부분 맞고, 안 맞는 도면만 `calibrate`로 `xs`를 만들어 붙인다. 도면의 `aspect`(HARPS의 값)가 실제 이미지 비율과 같은지 먼저 확인한다.

---

## 11. 키 입력 처리 로직 (순수 함수 + 엣지케이스 + 테스트)

**반드시 지킬 규칙**
1. **`event.code` 사용**(`event.key` 금지). `Shift+1`은 일부 키보드에서 `key`가 `"!"`로 바뀌어 숫자 판별이 깨진다. **`Digit1~0`과 `Numpad1~0` 둘 다** 지원.
2. **`Alt` 또는 `Meta`(⌘)** 가 눌리면 → 무음(OS/브라우저 단축키 충돌 방지).
3. **`Ctrl`+`Shift` 동시** → 무음.
4. **`event.repeat`(꾹 누름) 무시.** 단, 같은 키를 손으로 빠르게 여러 번 누르면 매번 재생.
5. 매핑된 **유효 키에서만 `event.preventDefault()`** 호출(Ctrl+숫자로 탭 전환·저장 차단).
6. 포커스가 `input/textarea/select/[contenteditable]` 안이면 연주하지 않음.
7. `resolveShortcut()`은 **선택된 하프의 `notes`에 실제로 존재하는 음만** 반환한다.

**순수 함수 레퍼런스 구현**
```js
function digitOf(code){
  const m = /^(?:Digit|Numpad)([0-9])$/.exec(code);
  return m ? m[1] : null;               // "0".."9" | null
}
function isEditableTarget(t){
  return !!(t instanceof Element && t.closest('input, textarea, select, [contenteditable="true"]'));
}
function resolveShortcut(e, type){
  if (e.altKey || e.metaKey) return null;
  if (e.shiftKey && e.ctrlKey) return null;
  const d = digitOf(e.code); if (d === null) return null;
  let note = e.shiftKey ? MAP.HIGH[d] : e.ctrlKey ? MAP.LOW[d] : MAP.MID[d];
  if (!note) return null;
  return HARPS[type].notes.includes(note) ? note : null;   // 하프에 없는 음은 무음
}
```

**전역 keydown**
```js
addEventListener("keydown", e => {
  if (e.repeat || isEditableTarget(e.target) || !state.currentType) return;
  const note = resolveShortcut(e, state.currentType);
  if (!note) return;
  e.preventDefault();
  triggerNote(note);
});
```

**필수 테스트 케이스 (`console.assert`)**
```js
const M=(code,mods,t)=>resolveShortcut({code,shiftKey:!!mods.s,ctrlKey:!!mods.c,altKey:!!mods.a,metaKey:!!mods.m},t);
// 숫자 단독 = 4옥타브 (모든 하프 공통)
console.assert(M("Digit1",{},"15")==="C4");
console.assert(M("Digit5",{},"19")==="G4");
console.assert(M("Digit7",{},"21")==="B4");
console.assert(M("Digit8",{},"21")===null);
console.assert(M("Digit0",{},"15")===null);
// Shift = 5옥타브
console.assert(M("Digit1",{s:1},"15")==="C5");
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
// 같은 숫자 = 같은 계이름 (5 = 항상 '솔')
console.assert(M("Digit5",{c:1},"21")==="G3" && M("Digit5",{},"21")==="G4" && M("Digit5",{s:1},"21")==="G5");
// 무효 조합
console.assert(M("Digit4",{s:1,c:1},"21")===null);
console.assert(M("Digit1",{a:1},"21")===null);
console.assert(M("Digit1",{m:1},"21")===null);
console.assert(M("Numpad1",{},"15")==="C4");
// 데이터 무결성
console.assert(NOTES_15.length===15 && NOTES_19.length===19 && NOTES_21.length===21);
console.assert([...NOTES_15,...NOTES_19,...NOTES_21].every(n=>NOTE[n]));
console.assert(NOTES_15.every(n=>NOTES_19.includes(n)) && NOTES_19.every(n=>NOTES_21.includes(n))); // 포개짐
```

---

## 12. 사운드 엔진 — Web Audio 합성 (하프 = 뜯는 현 음색)

**목표 음색**: 손끝으로 **현을 뜯는 빠른 어택** + 맑고 **정수 배음(harmonic)** 의 따뜻한 울림 + 자연스러운 잔향. 낮은 현일수록 길게 울린다. 종/메탈 느낌의 비배음(텅드럼)과 달리 **하프는 정수 배음**이 핵심. **폴리포니**(여러 현 동시·아르페지오). 소리는 자연 감쇠(끊지 않음).

**필수 구조**
- `AudioContext`는 1회 생성·재사용. **첫 사용자 입력에서 `resume()`**.
- **마스터 게인 + 볼륨 슬라이더(0~1)**. 현 위치에 따른 **스테레오 팬**(낮은음=왼쪽, 높은음=오른쪽).
- 음 = 기본음 + **정수 배음 파셜(1×,2×,3×…)** + **손끝 플럭 트랜지언트(밴드패스 노이즈)** + **밝게 시작해 닫히는 로우패스**.
- (권장) **합성 임펄스 리버브(ConvolverNode)** 로 맑은 잔향(작은 홀). mp3 불필요.
- 추후 샘플 교체 쉽게: 외부에서는 항상 `playNote(noteId, opts)`만 호출.

**레퍼런스 구현**
```js
let ctx=null, master=null;
function ensureAudio(){
  if(!ctx){
    ctx = new (window.AudioContext || window.webkitAudioContext)();
    master = ctx.createGain(); master.gain.value = 0.85; master.connect(ctx.destination);
    // 맑은 잔향(합성 임펄스 리버브)
    const conv = ctx.createConvolver(); conv.buffer = makeIR(ctx, 1.9, 2.6);
    const wet  = ctx.createGain(); wet.gain.value = 0.18;          // 잔향 양 0~0.35 권장
    master.connect(wet); wet.connect(conv); conv.connect(ctx.destination);
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

// 손끝으로 뜯는 짧은 트랜지언트(밴드패스 노이즈) — 하프 특유의 '띵' 어택
function makePluckNoise(now, dest, f, amt=0.05){
  const dur=0.018, n=Math.max(1,Math.floor(ctx.sampleRate*dur));
  const buf=ctx.createBuffer(1,n,ctx.sampleRate); const d=buf.getChannelData(0);
  for(let i=0;i<n;i++) d[i]=Math.random()*2-1;
  const src=ctx.createBufferSource(); src.buffer=buf;
  const bp=ctx.createBiquadFilter(); bp.type="bandpass"; bp.frequency.value=Math.min(8000,f*4); bp.Q.value=0.6;
  const g=ctx.createGain(); g.gain.setValueAtTime(amt,now); g.gain.exponentialRampToValueAtTime(0.0001, now+dur);
  src.connect(bp).connect(g).connect(dest); src.start(now); src.stop(now+dur+0.01);
}

function playNote(noteId, opts={}){
  ensureAudio();
  const info = NOTE[noteId]; if(!info) return;
  const now = ctx.currentTime, f = info.freq, midi = info.midi;
  const decay = Math.max(1.6, 4.0 - (midi - 48) * 0.065);          // 낮을수록 길게(약 1.6~4.0s)

  const voice = ctx.createGain();
  voice.gain.setValueAtTime(0.0001, now);
  voice.gain.exponentialRampToValueAtTime(0.9, now + 0.004);       // 빠른 플럭 어택(~4ms)
  voice.gain.exponentialRampToValueAtTime(0.0001, now + decay);    // 긴 지수 감쇠

  // 밝게 시작해 서서히 닫히는 로우패스(맑게 → 따뜻하게)
  const lp = ctx.createBiquadFilter(); lp.type="lowpass"; lp.Q.value=0.5;
  lp.frequency.setValueAtTime(Math.min(13000, f*16), now);
  lp.frequency.exponentialRampToValueAtTime(Math.max(800, f*3), now + decay);

  // 현 위치 기반 스테레오 팬
  let out = lp;
  if(ctx.createStereoPanner && typeof opts.pan === "number"){
    const pan = ctx.createStereoPanner();
    pan.pan.value = Math.max(-0.7, Math.min(0.7, opts.pan));
    lp.connect(pan); out = pan;
  }
  out.connect(master);

  // 하프 = 정수 배음(harmonic). 높은 배음일수록 더 빨리 사라짐(밝은 어택→따뜻한 여운)
  [[1,1.00],[2,0.55],[3,0.34],[4,0.20],[5,0.13],[6,0.08],[7,0.05]].forEach(([h,g])=>{
    if(f*h > 15000) return;
    const o = ctx.createOscillator(); o.type = "sine"; o.frequency.value = f*h;
    const og = ctx.createGain(); og.gain.setValueAtTime(g, now);
    og.gain.exponentialRampToValueAtTime(0.0001, now + decay * (1/Math.sqrt(h)));
    o.connect(og).connect(voice); o.start(now); o.stop(now + decay + 0.08);
  });
  voice.connect(lp);
  makePluckNoise(now, voice, f);
}
```
> 더 사실적인 소리는 ① 음별 샘플(.wav/.mp3)을 로드해 `playNote`만 갈아끼우거나, ② **Karplus-Strong**(튜닝된 딜레이 + 로우패스 피드백) 물리 모델링을 `AudioWorklet`으로 추가한다(부록·17번). 기본 합성만으로도 충분히 하프답게 동작한다.

---

## 13. 상호작용 & 시각 효과

**공통 트리거 함수 — 키보드·마우스·터치 모두 동일 경로**
```js
function triggerNote(noteId){
  const type = state.currentType;
  const notes = HARPS[type].notes;
  const idx = notes.indexOf(noteId), n = notes.length;
  const pan = n > 1 ? (idx / (n - 1)) * 1.4 - 0.7 : 0;   // 왼(낮음) ~ 오른(높음)
  playNote(noteId, { pan });
  flashString(noteId);
  showRecentNote(noteId);
  updateAriaLive(noteId);
}
function flashString(noteId){
  const el = document.querySelector(`.string-hit[data-note="${noteId}"]`);
  if(!el) return;
  el.classList.remove("is-playing"); void el.offsetWidth;  // 애니메이션 재시작
  el.classList.add("is-playing");
  setTimeout(()=>el.classList.remove("is-playing"), 520);
}
```

- **연주 피드백**: 해당 현이 `.is-playing`(짧은 strum 진동 + 앰버 글로우) 약 0.5s. **도면 이미지는 변형 금지**(현 오버레이만 반응).
- `keydown`에서 소리+효과, 시각 효과는 타이머/`keyup`으로 해제. **소리는 끊지 않고 자연 감쇠**.
- **최근 연주음 배지**(예: "솔 · G4 · 5")를 1초 정도 표시 후 페이드아웃.
- (접근성) 별도 `aria-live="polite"` 영역에 최근 음 이름을 갱신.
- 움직임은 **잔잔하게**. `prefers-reduced-motion`이면 글로우만 켜고 strum 진동은 생략.

---

## 14. 접근성 & 반응형

- 모든 카드·현은 키보드 포커스 가능 + 또렷한 포커스 링. 실제 `<button>` 우선.
- 이미지 `alt`(예: `alt="19음 미니하프"`), 현 `aria-label`(계이름+음명+단축키), `aria-live`로 최근 음.
- 터치 타깃 최소 36px 확보(현 간격이 좁으면 `--w`로 클릭 폭을 충분히). 작은 화면에서 하단 안내 패널은 접이식으로 본체를 가리지 않게.
- 가로/세로·다양한 해상도에서 **9:16 비율 + 도면 무왜곡** 유지.

```css
:focus-visible { outline: 3px solid var(--glow); outline-offset: 3px; }
.sr-only { position:absolute; width:1px; height:1px; padding:0; margin:-1px; overflow:hidden; clip:rect(0,0,0,0); white-space:nowrap; border:0; }
@media (prefers-reduced-motion: reduce){ *,*::before,*::after{ animation-duration:.01ms!important; transition-duration:.01ms!important; } }
```

---

## 15. 단계별 개발 순서 (Phase 1~8)

각 단계가 끝날 때마다 브라우저에서 직접 확인하고 다음으로 넘어간다.

1. **뼈대 & 9:16 화면**: `index.html` + `#app`(9:16, 3번 CSS) + `#selectScreen`/`#playScreen` 토글(`showSelect()`, `showPlay(type)`) + `← 다시 선택`. `const state={currentType:null}`. 데스크톱 레터박스·모바일 풀 확인.
2. **데이터 모델**: `NOTE`, `NOTES_15/19/21`, `MAP`, `HARPS`(9·10번). 15/19/21개·`NOTE` 존재·포개짐 검증.
3. **키 입력 순수함수 + 테스트**: `digitOf`, `isEditableTarget`, `resolveShortcut`(11번) + `console.assert` 전부 통과.
4. **사운드 엔진**: `ensureAudio`/`setVolume`/`playNote`(+리버브·플럭 노이즈·팬, 12번). 첫 입력 `resume`, 폴리포니, 낮은 현이 더 길게 울리는지, 어택이 '뜯는' 느낌인지 확인.
5. **선택 화면**: `char-15/19/21.png` 무왜곡 카드 3개 + 라벨/부제 + hover/ripple/페이드 전환(6번).
6. **연주 화면 + 이미지**: 상단 바(뒤로/악기명/볼륨) + `.harp-stage`에 `play-15/19/21.png` **원본 무왜곡 최대 표시**(4번). `aspect`가 실제 이미지와 맞는지 확인.
7. **투명 현 오버레이**: `renderStrings(type)`로 `field` 균등 분배 투명 버튼 생성 → pointer/터치/키보드 모두 `triggerNote()` → strum 글로우 + 도(빨강)/파(파랑) 표시점 + 최근음 배지 + `aria-live`. 도면과 어긋나면 `?calibrate=1`로 `xs` 보정(10번).
8. **마감·검수**: 키 안내 패널/매핑표 토글, 접근성(포커스/aria), `prefers-reduced-motion`, 다양한 화면에서 9:16·무왜곡 확인, 콘솔 오류 제거.

---

## 16. 완성 체크리스트

**레이아웃 / 이미지**
- [ ] 앱이 **9:16 비율**로 뷰포트를 가득 채운다(모바일 풀, 데스크톱 중앙 레터박스). 빈 흰 여백 없음.
- [ ] 선택 화면 카드 이미지 3개가 **왜곡 없이**(contain) 표시된다.
- [ ] 연주 화면 15/19/21음 도면이 **왜곡·잘림 없이** 9:16 안에서 **최대 크기**로 표시된다.
- [ ] 투명 현이 도면과 함께 정확히 스케일된다(화면 크기 바뀌어도 어긋나지 않음).
- [ ] 도면의 현과 히트존 위치가 대체로 일치한다(`field`/`xs`/`calibrate`로 맞춤). 도(C)=빨강·파(F)=파랑 표시점이 보인다.

**연주 / 매핑**
- [ ] 15음=`G3~G5`, 19음=`E3~B5`, 21음=`C3~B5`만 재생된다.
- [ ] `1~7`→C4~B4. `⇧`=5옥타브(15음은 `⇧1~5`만), `⌃`=3옥타브(15음은 `⌃5~7`만, 19음은 `⌃3~7`만, 21음은 `⌃1~7` 전부).
- [ ] `같은 숫자=같은 계이름`(`⌃5=G3`, `5=G4`, `⇧5=G5`).
- [ ] `8/9/0` 단독·하프에 없는 `Shift`/`Ctrl` 조합·`Ctrl+Shift`·`Alt`·`Meta` 무음. `Numpad`도 동작.
- [ ] 키 꾹 눌러도 연타 안 됨, 빠른 반복은 매번 재생. `Ctrl+숫자`로 브라우저 탭 안 바뀜.
- [ ] 키보드·마우스·터치 모두 같은 `triggerNote()`. 폴리포니(동시·아르페지오). 첫 입력에서 정상 발음(resume).
- [ ] 누른 현 즉시 strum 진동/글로우 + 최근음 배지. 도면 본체는 변형되지 않음.

**사운드 / 품질**
- [ ] 텅드럼처럼 둥근 게 아니라 **현을 뜯는 빠른 어택 + 정수 배음의 맑은 울림**. 은은한 잔향(리버브).
- [ ] 낮은 현이 더 길고, 높은 현은 너무 날카롭지 않게. 낮은음=왼쪽/높은음=오른쪽 팬. 볼륨 슬라이더 즉시 반영.
- [ ] 순수함수 테스트·데이터 개수/존재/포개짐 검증 통과. 포커스 링/`aria-label`/`aria-live`/`prefers-reduced-motion` 반영.

---

## 17. 확장 아이디어

녹음/재생, 숫자악보 입력→자동 연주, 동요·명상곡 연습 모드(반짝반짝 작은별 등), **아르페지오/글리산도**(여러 현 쓸어내리기) 제스처, **Karplus-Strong 물리모델 음색**(AudioWorklet), 리버브/딜레이 양 슬라이더, 메트로놈, 실물 mp3 샘플 교체, **레버(샵/플랫) 토글**로 다른 조성 연주, 숫자악보 라벨 표시 토글, **현 좌표 편집 모드**(`calibrate` 확장: 드래그로 위치 조정 후 `xs` JSON 내보내기), 23음 등 확장 음역(키 규칙 확장: `8/9/0` 또는 `Q~P` 행 추가), PWA 설치, 다크/연주회 모드.

---

## 부록 A. 그대로 복붙하는 압축 프롬프트

```text
키보드와 마우스/터치로 연주하는 "미니하프 웹앱"을 외부 라이브러리 없이 단일 index.html(HTML+CSS+Vanilla JS) + assets/ 이미지 폴더로 만들어줘. 사운드는 Web Audio API 합성(현을 뜯는 plucked 음색).

[화면 흐름]
- 화면1(선택): 15음/19음/21음 미니하프 카드 3개. assets/char-15.png, char-19.png, char-21.png 사용. hover 확대, 클릭 ripple, 0.3~0.4s 페이드 전환으로 선택값("15"|"19"|"21") 전달.
- 화면2(연주): 선택한 하프 도면 표시. assets/play-15.png(15현), play-19.png(19현), play-21.png(실사 23현 렌더) 사용. 상단 바(← 다시 선택 / 악기명 / 볼륨), 하단 키 안내 + 최근음 배지.

[★ 9:16 풀스크린]
- #app은 항상 9:16 비율로 뷰포트 안에서 최대 크기 중앙 배치. 모바일은 꽉 차게, 데스크톱/가로는 중앙 9:16 패널 + 어두운 우드톤 레터박스. 100dvh, overflow:hidden, 빈 흰 여백 금지(테마 배경).
- #app { width:min(100vw, calc(100dvh*9/16)); height:min(100dvh, calc(100vw*16/9)); position:relative; overflow:hidden; }
- .screen { position:absolute; inset:0; } 로 전환.

[★ 이미지 원본 무왜곡 + 현 오버레이]
- 모든 이미지 종횡비 유지 + object-fit:contain (cover/찌그러짐/잘림 금지). 도면은 9:16 안에서 최대 크기. 남는 공간은 테마 배경.
- 연주 화면: 이미지와 같은 비율의 .harp-stage 안에 도면을 깔고, 그 위 .string-layer(position:absolute; inset:0)에 투명 세로 현(button.string-hit)을 HARPS[type].field 기준으로 왼쪽(최저음)→오른쪽(최고음) 균등 배치. 누르면 strum 진동+앰버 글로우만(도면 자체 변형 금지). 도(C)현엔 빨강, 파(F)현엔 파랑 표시점.
- 도면마다 현 위치/기울기가 다르니 field{left,right,top,bottom,tilt}(또는 xs 배열)만 고쳐 맞춤. ?calibrate=1 이면 stage 클릭 좌표를 xs로 출력.
- 주의: play-19는 F3~C6 라벨이지만 키 규칙상 6옥타브 C6 불가 → 19음 음역은 E3~B5로 매핑. play-21은 현이 23개로 보이나 21음으로 쓰고 21개 히트존 균등 배치(나머지 비활성).

[튜닝 — C Major 온음계, 키 규칙에서 도출된 음역]
- 15음(15): G3 A3 B3 C4 D4 E4 F4 G4 A4 B4 C5 D5 E5 F5 G5
- 19음(19): E3 F3 G3 A3 B3 C4 D4 E4 F4 G4 A4 B4 C5 D5 E5 F5 G5 A5 B5
- 21음(21): C3 D3 E3 F3 G3 A3 B3 C4 D4 E4 F4 G4 A4 B4 C5 D5 E5 F5 G5 A5 B5  (15음 ⊂ 19음 ⊂ 21음)
- NOTE 객체에 freq/midi/num(자음보)/ko(한글계이름). freq=440*2^((midi-69)/12). C3=130.81 … A4=440.00 … B5=987.77.

[키 입력 규칙 — 핵심]
- 숫자 1~7 = 계이름(1=도C,2=레D,3=미E,4=파F,5=솔G,6=라A,7=시B). 수식키=옥타브. 같은 숫자=같은 계이름.
- 단독=4옥타브(C4~B4) / Shift=5옥타브(C5~B5) / Ctrl=3옥타브(C3~B3). 선택한 하프에 실제 있는 음만 소리(15음: ⇧1~5, ⌃5~7 / 19음: ⇧1~7, ⌃3~7 / 21음: ⇧1~7, ⌃1~7). 그 외 무음. 8/9/0 단독 무음.
- 반드시 event.code(Digit1~0 + Numpad1~0)와 shiftKey/ctrlKey로 판별(Shift+1은 key가 "!"). Alt/Meta 또는 Ctrl+Shift 동시 → 무음. event.repeat 무시. 매핑된 유효 키만 preventDefault. input/textarea 포커스 시 연주 안 함.
- 순수함수 resolveShortcut({code,shiftKey,ctrlKey,altKey,metaKey}, "15"|"19"|"21") -> noteId|null. HARPS[type].notes.includes로 필터. console.assert 테스트 포함: Digit1→C4, Digit8→null, Shift+Digit5(15)→G5, Shift+Digit6(15)→null, Ctrl+Digit5(15)→G3, Ctrl+Digit4(15)→null, Ctrl+Digit3(19)→E3, Ctrl+Digit2(19)→null, Ctrl+Digit1(21)→C3, Ctrl+Shift+Digit4→null, Alt/Meta+Digit1→null, Numpad1→C4.

[현 좌표]
- HARPS["15"|"19"|"21"].field {left,right,top,bottom,tilt}(%로 현 영역). renderStrings(type)가 notes를 왼쪽(최저)→오른쪽(최고)으로 균등 분배해 투명 button.string-hit 생성, --w/--h/--rot 설정, aria-label에 계이름+음명+단축키. C현 is-c(빨강점)/F현 is-f(파랑점). pointerdown과 keydown 모두 triggerNote(noteId). 어긋나면 field/xs만 수정(또는 ?calibrate=1).

[사운드 — 하프(뜯는 현) 음색]
- AudioContext 1회 생성·재사용, 첫 입력 resume(). master gain + 볼륨(0~1) + 현 위치 기반 StereoPanner(낮은음 왼쪽, 높은음 오른쪽).
- playNote: 빠른 플럭 어택(~4ms) + 정수 배음 파셜(1,2,3,4,5,6,7배, 높은 배음일수록 빨리 감쇠) + 밝게 시작해 닫히는 lowpass + 짧은 bandpass 노이즈로 손끝 어택 + 긴 지수 감쇠(낮을수록 길게 1.6~4.0s). 폴리포니, 소리는 끊지 말고 자연 감쇠. 합성 임펄스 ConvolverNode 리버브(wet~0.18)로 맑은 잔향.

[상호작용/접근성]
- 누르면 해당 현 strum 진동+앰버 글로우(도면 변형 금지), 최근음 배지("솔·G4·5") 잠깐 표시, aria-live 갱신. role=button + aria-label. 포커스 링. prefers-reduced-motion 반영.

색: 배경 따뜻한 다크 우드 그라데이션(#1a1410~), 현 금색(#D9B45B), 글로우 앰버(#FFE08A), 15음 로즈(#C77B52)/19음 틸(#4FA39A)/21음 바이올렛(#8E6FC0), 도현 빨강(#D8504A)/파현 파랑(#4A7FD8), 텍스트 크림(#F3ECDD). 맑고 우아한 톤. 완료 후 주요 구현/실행법/체크리스트를 요약해줘.
```

---

## 부록 B. (선택) React + TypeScript 구조로 만들 경우

단일 파일 대신 유지보수성을 높이려면 아래 구조를 따른다. 튜닝·키 매핑·9:16·이미지 무왜곡 규칙은 본문과 동일하게 지킨다.

```text
src/
  data/harps.ts             # NOTE, NOTES_15/19/21, MAP, HARPS, 타입
  utils/keyboard.ts         # digitOf, isEditableTarget, resolveShortcut (+ keyboard.test.ts)
  audio/HarpAudioEngine.ts  # ensureAudio, setVolume, playNote(+리버브/플럭노이즈/팬)
  pages/SelectScreen.tsx    # 15/19/21 카드
  pages/PlayScreen.tsx      # 선택 하프 도면 + 투명 현 오버레이
  components/HarpStage.tsx, StringButton.tsx, KeyboardGuide.tsx
  styles/global.css         # 9:16 풀스크린, .harp-stage, .string-hit 등
public/assets/              # char-15/19/21.png, play-15/19/21.png, icon.png
```
- 라우팅: `/`, `/play/15`, `/play/19`, `/play/21`(URL이 source of truth, 잘못된 값은 선택 화면으로).
- `resolveShortcut`는 React 상태에 의존하지 않는 순수 함수로 유지, **Vitest**로 부록 A 테스트 검증.
- `AudioContext`는 싱글턴으로, 리렌더마다 새로 만들지 않는다. `HARPS`의 `field`/`xs`는 좌표 편집을 위해 별도 파일로.

---

*이 명세대로 만들면 9:16 화면을 가득 채우고, 15/19/21음 미니하프 도면을 왜곡 없이 그대로 보여주며, 그 위 투명한 세로 현과 숫자키/Shift/Ctrl 매핑(1~7=도레미파솔라시, 수식키=옥타브)으로 정확히 연주되고, 빠른 어택·정수 배음·맑은 잔향의 하프다운 소리가 나는 우아한 연주 웹앱이 완성됩니다.* 🎵🎶
