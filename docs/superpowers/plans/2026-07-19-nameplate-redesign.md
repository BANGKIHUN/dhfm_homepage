# 「명판」 리디자인 구현 계획

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** 대하산업기계 홈페이지 23개 페이지의 비주얼 아이덴티티를 「명판」 방향으로 전면 교체한다. 메뉴 구성·페이지 목록·기존 JS 동작은 그대로 유지한다.

**Architecture:** 단일 `index.html`(SPA, `goto()` 페이지 전환)의 마크업을 교체하고, 인라인 `<style>`을 `styles.css`로 분리·재작성한다. 모든 화면은 **플레이트(명판)** 단일 컴포넌트의 변주로 구성한다. 빌드 단계 없음.

**Tech Stack:** 순수 HTML/CSS/JS. Google Fonts(Archivo, IBM Plex Sans KR, IBM Plex Mono). Formspree(폼 전송). 네이버 지도 API.

## Global Constraints

- 작업 브랜치: `redesign-nameplate` (부모 저장소 `D:/workspace/dhfm homepage/`)
- **페이지 추가·삭제·이동 금지.** `pages` 맵의 23개 키를 그대로 유지한다.
- **보존 필수 JS:** `goto`, `toggleMobileMenu`, `toggleMobileSub`, `gotoMobile`, `sendMsg`, `submitForm`, `sendConsultation`, `sendQuote`, `toggleFaq`, `switchTab`, `galleryData`, `buildGallery`, `switchGalleryTab`, `openLightbox`, `closeLightbox`, `lightboxMove`
- **Formspree 엔드포인트 변경 금지:** `https://formspree.io/f/meedkznw`
- **색 토큰 5개만 사용.** 하드코딩 색상값 금지.
  `--cast:#23262A` `--iron:#565D63` `--machined:#C9CDD0` `--plate:#ECEDEB` `--oxide:#B23A1E`
- **`--oxide` 사용처는 3곳뿐:** 견적요청 CTA / 현재 위치 표시 / 카테고리 마크
- **서체 3역할:** Display=Archivo 700–800 대문자 / Body=IBM Plex Sans KR 400·500 / Data=IBM Plex Mono 500 `font-variant-numeric: tabular-nums`
- **리벳은 홈 히어로 명판에만.** 다른 플레이트 금지.
- **그라디언트·그림자·질감 금지.** 경계선과 구조로만 명판감을 만든다.
- **CSS 선택자 중첩 깊이 2 이하.** 섹션 간격은 단일 유틸리티로 통일, 개별 재정의 금지.
- **`product-photo-01.jpg` 참조 금지.** 파일은 삭제하지 않는다.
- 모든 `<img>`: `alt` 필수, `loading="lazy"`, 명시적 `width`/`height`
- `prefers-reduced-motion: reduce`에서 모든 애니메이션 제거

## 검증 방식에 대한 참고

이 프로젝트에는 테스트 프레임워크가 없다(정적 HTML/CSS). 따라서 각 태스크의 검증은 **로컬 정적 서버 + 브라우저 실제 확인**으로 한다. 태스크마다 확인할 항목을 명시했다. 단위 테스트를 만들어내지 않는다 — 이 코드베이스에 맞지 않는다.

검증 서버 기동 (모든 태스크 공통, 한 번만):
```bash
cd "D:/workspace/dhfm homepage" && python -m http.server 8899
```
확인 주소: `http://localhost:8899/`

## File Structure

| 파일 | 책임 | 상태 |
|---|---|---|
| `index.html` | 마크업 23개 페이지 + 헤더/내비/푸터 + 인라인 JS | 수정 |
| `styles.css` | 전체 스타일. 토큰 → 기본 → 컴포넌트 → 템플릿 → 반응형 순 | 신규 |
| `images/` | 기존 유지 | 변경 없음 |

`styles.css` 내부 순서 (섹션 주석으로 구분, 이 순서를 지킬 것):
```
1. 토큰 (:root)
2. 리셋 · 기본 타이포
3. 레이아웃 유틸 (컨테이너, 섹션 간격)
4. 플레이트 컴포넌트 (.plate 계열)
5. 헤더 · 내비 · 모바일 메뉴
6. 푸터
7. 페이지 명판 (.page-plate)
8. T1 홈
9. T2 제품 상세
10. T3 사업분야
11. T4 문서
12. T5 데이터
13. T6 폼
14. T7 갤러리 + 라이트박스
15. T8 FAQ
16. 반응형 (@media)
17. 접근성 (focus-visible, prefers-reduced-motion)
```

---

## Task 1: 토대 — styles.css 분리, 토큰, 플레이트 컴포넌트

**Files:**
- Create: `styles.css`
- Modify: `index.html` (`<head>` 폰트 링크 교체, 인라인 `<style>` 블록 61–794행 제거, `<link>` 추가)

**Interfaces:**
- Produces: 이후 모든 태스크가 사용하는 클래스 —
  `.plate` `.plate--rivet` `.plate__band` `.plate__body` `.plate__foot`
  `.rule` `.rule--thin` `.data` `.data__key` `.data__val`
  `.container` `.section` `.btn` `.btn--quote` `.btn--ghost` `.mark`

- [ ] **Step 1: `<head>` 폰트 링크 교체**

`index.html` 59–60행의 기존 Google Fonts 링크를 아래로 교체:

```html
<link rel="preconnect" href="https://fonts.googleapis.com">
<link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
<link href="https://fonts.googleapis.com/css2?family=Archivo:wght@600;700;800&family=IBM+Plex+Sans+KR:wght@400;500;600&family=IBM+Plex+Mono:wght@400;500;600&display=swap" rel="stylesheet">
<link rel="stylesheet" href="styles.css">
```

- [ ] **Step 2: 인라인 `<style>` 제거**

`index.html`에서 `<style>`(61행)부터 `</style>`(794행 부근)까지 통째로 삭제한다. 삭제 후 `<head>`에 `<style>` 태그가 남아 있지 않은지 확인:

```bash
grep -c "<style>" index.html   # 기대: 0
```

- [ ] **Step 3: `styles.css` 생성 — 토큰과 기본**

```css
/* ══════ 1. 토큰 ══════ */
:root {
  --cast:     #23262A;
  --iron:     #565D63;
  --machined: #C9CDD0;
  --plate:    #ECEDEB;
  --oxide:    #B23A1E;

  --font-display: 'Archivo', system-ui, sans-serif;
  --font-body:    'IBM Plex Sans KR', system-ui, sans-serif;
  --font-data:    'IBM Plex Mono', ui-monospace, monospace;

  --gap:      clamp(48px, 7vw, 96px);
  --pad:      clamp(20px, 4vw, 40px);
  --measure:  68ch;
  --header-h: 72px;
}

/* ══════ 2. 리셋 · 기본 타이포 ══════ */
*, *::before, *::after { box-sizing: border-box; margin: 0; padding: 0; }
html { scroll-behavior: smooth; }

body {
  font-family: var(--font-body);
  font-weight: 400;
  color: var(--cast);
  background: var(--plate);
  line-height: 1.75;
  -webkit-font-smoothing: antialiased;
}

h1, h2, h3 { font-weight: 600; line-height: 1.25; letter-spacing: -0.01em; }

.display {
  font-family: var(--font-display);
  font-weight: 800;
  text-transform: uppercase;
  letter-spacing: -0.01em;
  line-height: 1.1;
}

.data, .num {
  font-family: var(--font-data);
  font-variant-numeric: tabular-nums;
}

a { color: inherit; }
img { max-width: 100%; height: auto; display: block; }

/* ══════ 3. 레이아웃 유틸 ══════ */
.container { width: 100%; max-width: 1180px; margin-inline: auto; padding-inline: var(--pad); }
.section   { padding-block: var(--gap); }
.measure   { max-width: var(--measure); }
```

- [ ] **Step 4: 플레이트 컴포넌트 추가**

`styles.css`에 이어서 작성. **이 컴포넌트가 전체 디자인의 단일 기본 단위다.**

```css
/* ══════ 4. 플레이트 ══════ */
.plate {
  position: relative;
  background: var(--plate);
  border: 1px solid var(--machined);
  box-shadow: inset 0 0 0 1px rgba(255,255,255,0.65);
}

/* 리벳 — 홈 히어로 명판에만 사용 */
.plate--rivet::before, .plate--rivet::after,
.plate--rivet > .rivet-b::before, .plate--rivet > .rivet-b::after {
  content: ''; position: absolute;
  width: 7px; height: 7px; border-radius: 50%;
  background: var(--machined);
}
.plate--rivet::before            { top: 9px;    left: 9px; }
.plate--rivet::after             { top: 9px;    right: 9px; }
.plate--rivet > .rivet-b::before { bottom: 9px; left: 9px; }
.plate--rivet > .rivet-b::after  { bottom: 9px; right: 9px; }

.plate__band {
  display: flex; justify-content: space-between; align-items: center; gap: 12px;
  padding: 10px var(--pad);
  border-bottom: 1px solid var(--machined);
  font-family: var(--font-data);
  font-size: 0.66rem; letter-spacing: 0.18em; text-transform: uppercase;
  color: var(--iron);
}

.plate__body { padding: var(--pad); }

.plate__foot {
  display: flex; justify-content: space-between; align-items: center;
  gap: 16px; flex-wrap: wrap;
  padding: 16px var(--pad);
  border-top: 1px solid var(--machined);
}

/* 각인선 */
.rule       { height: 2px; background: var(--cast); margin-top: 18px; }
.rule--thin { height: 1px; background: var(--machined); margin-bottom: 16px; }

/* 명판 데이터행 */
.data-table { width: 100%; border-collapse: collapse; font-family: var(--font-data);
              font-size: 0.8rem; font-variant-numeric: tabular-nums; }
.data-table td { padding: 7px 0; border-bottom: 1px solid var(--machined); vertical-align: top; }
.data-table td:first-child { width: 92px; color: var(--iron);
                             font-size: 0.66rem; letter-spacing: 0.1em; padding-top: 9px; }
.data-table td:last-child  { color: var(--cast); font-weight: 500; }

/* 카테고리 마크 — oxide 허용처 3/3 */
.mark { display: inline-block; width: 6px; height: 6px; background: var(--oxide);
        margin-right: 9px; vertical-align: middle; }

/* ══════ 버튼 ══════ */
.btn {
  display: inline-flex; align-items: center; gap: 8px;
  padding: 13px 26px; border: 1px solid var(--cast); background: none;
  font-family: var(--font-data); font-size: 0.78rem; font-weight: 600;
  letter-spacing: 0.12em; color: var(--cast); cursor: pointer;
  transition: background 0.15s, color 0.15s;
}
.btn:hover { background: var(--cast); color: var(--plate); }

/* 견적요청 — oxide 허용처 1/3 */
.btn--quote { background: var(--oxide); border-color: var(--oxide); color: #fff; }
.btn--quote:hover { background: var(--cast); border-color: var(--cast); color: #fff; }

.btn--ghost { border-color: var(--machined); color: var(--iron); }
.btn--ghost:hover { background: var(--iron); border-color: var(--iron); color: var(--plate); }
```

- [ ] **Step 5: 접근성·모션 기본 추가**

`styles.css` 맨 끝에 작성:

```css
/* ══════ 17. 접근성 ══════ */
:focus-visible {
  outline: 2px solid var(--oxide);
  outline-offset: 2px;
}

@media (prefers-reduced-motion: reduce) {
  *, *::before, *::after {
    animation-duration: 0.01ms !important;
    animation-iteration-count: 1 !important;
    transition-duration: 0.01ms !important;
    scroll-behavior: auto !important;
  }
}
```

- [ ] **Step 6: 검증**

```bash
cd "D:/workspace/dhfm homepage" && python -m http.server 8899
```

브라우저에서 `http://localhost:8899/` 확인. 이 시점에는 **스타일이 대부분 벗겨진 상태가 정상이다** (기존 CSS를 지웠고 새 템플릿은 아직 없음). 확인할 것:
- 콘솔에 404 없음 (`styles.css`, 폰트 정상 로드)
- `goto()` 클릭 시 페이지 전환이 여전히 동작
- 폰트가 Archivo / IBM Plex로 적용됨 (개발자도구 Computed 확인)

- [ ] **Step 7: 커밋**

```bash
git add styles.css index.html
git commit -m "refactor: CSS를 styles.css로 분리하고 명판 토큰·플레이트 컴포넌트 도입"
```

---

## Task 2: 헤더 · 내비 · 모바일 메뉴 · 푸터

**Files:**
- Modify: `index.html` (헤더 795–852행, 모바일 메뉴 855–908행, 푸터)
- Modify: `styles.css` (섹션 5, 6)

**Interfaces:**
- Consumes: Task 1의 `.plate` `.btn--quote` `.mark` `.data`
- Produces: `.site-header` `.nav` `.nav__item` `.nav__drop` `.mobile-menu` `.site-footer`

- [ ] **Step 1: 헤더 마크업 교체**

로고를 명판 어법으로. `index.html` 795–799행의 `.logo` 블록을 교체:

```html
<a class="logo" href="#" onclick="goto('home')">
  <span class="logo__ko">대하산업기계</span>
  <span class="logo__en data">DHFM CO., LTD.</span>
</a>
```

내비 구조(800–847행)는 **`onclick="goto(...)"` 호출을 그대로 두고** 클래스만 부여한다:
`<nav>` → `<nav class="nav">`, `.nav-item` → `.nav__item`, `.dropdown` → `.nav__drop`,
`.contact-btn` → `class="nav__item nav__cta"`.

**`INTRANET` 링크는 추가하지 않는다** (제거된 상태 유지).

- [ ] **Step 2: 헤더 스타일**

```css
/* ══════ 5. 헤더 ══════ */
.site-header {
  position: fixed; inset: 0 0 auto 0; z-index: 1000;
  background: var(--plate);
  border-bottom: 1px solid var(--machined);
}
.header-main {
  display: flex; align-items: center; justify-content: space-between; gap: 24px;
  height: var(--header-h);
  max-width: 1180px; margin-inline: auto; padding-inline: var(--pad);
}
.logo { text-decoration: none; display: flex; flex-direction: column; line-height: 1.15; }
.logo__ko { font-weight: 600; font-size: 1.05rem; letter-spacing: -0.02em; }
.logo__en { font-size: 0.6rem; letter-spacing: 0.2em; color: var(--iron); }

.nav { display: flex; align-items: center; gap: 4px; }
.nav__item {
  position: relative; padding: 24px 14px; cursor: pointer;
  font-size: 0.86rem; font-weight: 500;
}
.nav__item::after {
  content: ''; position: absolute; left: 14px; right: 14px; bottom: 18px;
  height: 2px; background: var(--oxide);
  transform: scaleX(0); transition: transform 0.15s;
}
.nav__item:hover::after { transform: scaleX(1); }   /* oxide 허용처 2/3 */

.nav__drop {
  position: absolute; top: 100%; left: 0; min-width: 180px;
  background: var(--plate); border: 1px solid var(--machined);
  opacity: 0; visibility: hidden; transition: opacity 0.12s;
}
.nav__item:hover .nav__drop, .nav__item:focus-within .nav__drop { opacity: 1; visibility: visible; }
.nav__drop a {
  display: block; padding: 11px 16px; font-size: 0.82rem; text-decoration: none;
  border-bottom: 1px solid var(--machined);
}
.nav__drop a:last-child { border-bottom: 0; }
.nav__drop a:hover { background: var(--cast); color: var(--plate); }

.nav__cta {
  margin-left: 8px; padding: 11px 20px;
  background: var(--oxide); color: #fff;      /* oxide 허용처 1/3 */
  font-family: var(--font-data); font-size: 0.74rem; letter-spacing: 0.1em;
}
.nav__cta::after { display: none; }

body { padding-top: var(--header-h); }
```

- [ ] **Step 3: 모바일 메뉴 스타일**

기존 마크업(855–908행)의 클래스명은 JS가 참조하므로 **`.mobile-menu` `.mobile-menu-item` `.mobile-menu-cat` `.mobile-sub` `.open` 을 그대로 유지**한다. 외형만 교체:

```css
.hamburger { display: none; }

@media (max-width: 900px) {
  .nav { display: none; }
  .hamburger {
    display: flex; flex-direction: column; justify-content: center; gap: 5px;
    width: 44px; height: 44px; background: none; border: 0; cursor: pointer;
  }
  .hamburger span { display: block; height: 2px; background: var(--cast); transition: transform 0.2s, opacity 0.2s; }
  .hamburger.open span:nth-child(1) { transform: translateY(7px) rotate(45deg); }
  .hamburger.open span:nth-child(2) { opacity: 0; }
  .hamburger.open span:nth-child(3) { transform: translateY(-7px) rotate(-45deg); }
}

.mobile-menu {
  position: fixed; inset: var(--header-h) 0 0 0; z-index: 999;
  background: var(--plate); overflow-y: auto;
  transform: translateX(100%); transition: transform 0.22s;
}
.mobile-menu.open { transform: translateX(0); }
.mobile-menu-cat {
  display: flex; justify-content: space-between; align-items: center;
  padding: 17px var(--pad); border-bottom: 1px solid var(--machined);
  font-weight: 500; cursor: pointer;
}
.mobile-menu-cat .arrow { transition: transform 0.18s; color: var(--iron); }
.mobile-menu-item.open .mobile-menu-cat .arrow { transform: rotate(90deg); }
.mobile-sub { display: none; background: rgba(0,0,0,0.02); }
.mobile-menu-item.open .mobile-sub { display: block; }
.mobile-sub a {
  display: block; padding: 13px var(--pad) 13px calc(var(--pad) + 16px);
  font-size: 0.86rem; text-decoration: none; border-bottom: 1px solid var(--machined);
}
.mobile-contact-btn {
  display: block; width: calc(100% - var(--pad) * 2); margin: var(--pad);
  padding: 15px; border: 0; background: var(--oxide); color: #fff;
  font-family: var(--font-data); font-size: 0.8rem; letter-spacing: 0.1em; cursor: pointer;
}
```

- [ ] **Step 4: 푸터를 명판으로 교체**

푸터 마크업을 아래로 교체 (기존 푸터 위치는 `</div>` 페이지 블록들 뒤, `<script>` 앞):

```html
<footer class="site-footer">
  <div class="container">
    <div class="footer-grid">
      <div>
        <div class="logo__ko">대하산업기계 주식회사</div>
        <div class="logo__en data">DAEHA INDUSTRIAL MACHINERY CO., LTD.</div>
      </div>
      <table class="data-table">
        <tr><td>ADDR.</td><td>인천광역시 서구 봉수대로 1568번길 63-20</td></tr>
        <tr><td>TEL.</td><td>032-567-9087</td></tr>
        <tr><td>FAX.</td><td>032-567-9088</td></tr>
        <tr><td>MAIL.</td><td>master@dhfm.co.kr</td></tr>
      </table>
    </div>
    <div class="footer-base data">
      <span>© 2016 DHFM CO., LTD.</span>
      <span>
        <a href="#" onclick="goto('privacy')">개인정보 보호정책</a>
        <a href="#" onclick="goto('terms')">이용약관</a>
      </span>
    </div>
  </div>
</footer>
```

```css
/* ══════ 6. 푸터 ══════ */
.site-footer { background: var(--cast); color: var(--machined); padding-block: var(--gap) 24px; margin-top: var(--gap); }
.site-footer .logo__ko { color: var(--plate); font-size: 1rem; }
.site-footer .data-table td { border-bottom-color: rgba(255,255,255,0.12); }
.site-footer .data-table td:last-child { color: var(--machined); }
.footer-grid { display: grid; grid-template-columns: 1fr 1fr; gap: var(--gap); }
.footer-base {
  display: flex; justify-content: space-between; flex-wrap: wrap; gap: 12px;
  margin-top: 40px; padding-top: 20px; border-top: 1px solid rgba(255,255,255,0.12);
  font-size: 0.68rem; letter-spacing: 0.08em; color: var(--iron);
}
.footer-base a { margin-left: 18px; text-decoration: none; }
.footer-base a:hover { color: var(--plate); }
@media (max-width: 700px) { .footer-grid { grid-template-columns: 1fr; gap: 32px; } }
```

- [ ] **Step 5: 검증**

`http://localhost:8899/` 에서 확인:
- 데스크톱: 6개 내비 카테고리 hover 시 드롭다운 표시, 하단에 oxide 밑줄
- 키보드: Tab으로 내비 이동 시 드롭다운이 `focus-within`으로 열리고 포커스 링이 보임
- 900px 이하로 줄이면 햄버거 표시 → 클릭 시 메뉴 슬라이드, 카테고리 클릭 시 아코디언 열림
- 모바일 메뉴 열린 상태에서 body 스크롤 잠김
- 푸터의 주소·전화가 고정폭으로 정렬됨

- [ ] **Step 6: 커밋**

```bash
git add index.html styles.css
git commit -m "feat: 헤더·내비·모바일 메뉴·푸터를 명판 어법으로 교체"
```

---

## Task 3: 페이지 명판 (22개 하위 페이지 공통 헤더)

**Files:**
- Modify: `index.html` (`.page-banner` 22곳)
- Modify: `styles.css` (섹션 7)

**Interfaces:**
- Consumes: `.mark` `.rule` `.container`
- Produces: `.page-plate` — 홈을 제외한 모든 페이지의 진입부

- [ ] **Step 1: 스타일 작성**

```css
/* ══════ 7. 페이지 명판 ══════ */
.page-plate { border-bottom: 1px solid var(--machined); padding-block: clamp(40px, 6vw, 72px) 0; }
.page-plate__cat {
  font-family: var(--font-data); font-size: 0.66rem;
  letter-spacing: 0.18em; text-transform: uppercase; color: var(--iron);
}
.page-plate__title { font-family: var(--font-display); font-weight: 800;
                     font-size: clamp(1.9rem, 4.5vw, 2.9rem); margin-top: 10px; }
.page-plate__ko { font-size: 0.92rem; color: var(--iron); margin-top: 6px; }
.page-plate .rule { margin-bottom: 0; }
```

- [ ] **Step 2: 22개 `.page-banner`를 교체**

각 페이지의 기존 배너를 아래 패턴으로 바꾼다. 예시는 CEO 인사말(`#page-ceo`):

```html
<div class="page-plate">
  <div class="container">
    <div class="page-plate__cat"><span class="mark"></span>회사소개 / ABOUT</div>
    <div class="page-plate__title">CEO 인사말</div>
    <div class="page-plate__ko">Message from the CEO</div>
    <div class="rule"></div>
  </div>
</div>
```

전체 22개의 카테고리·제목·영문 대응표:

| 페이지 ID | 카테고리 | 제목 | 영문 |
|---|---|---|---|
| `page-ceo` | 회사소개 / ABOUT | CEO 인사말 | Message from the CEO |
| `page-history` | 회사소개 / ABOUT | 회사연혁 | History |
| `page-policy` | 회사소개 / ABOUT | 경영방침 | Management Policy |
| `page-organization` | 회사소개 / ABOUT | 조직도 | Organization |
| `page-certification` | 회사소개 / ABOUT | 인증서 | Certifications |
| `page-ringblower` | 제품정보 / PRODUCTS | 링블로워 | Ring Blower |
| `page-plungerpump` | 제품정보 / PRODUCTS | 고압 플런저펌프 | High Pressure Plunger Pump |
| `page-washer` | 제품정보 / PRODUCTS | 고압 세척기 | High Pressure Washer |
| `page-washingsystem` | 제품정보 / PRODUCTS | 고압 세척시스템 | High Pressure Washing System |
| `page-blowerbs` | 사업분야 / BUSINESS | 블로워사업 | Blower Business |
| `page-pumpbs` | 사업분야 / BUSINESS | 펌프사업 | Pump Business |
| `page-pumpsystembs` | 사업분야 / BUSINESS | 펌프시스템사업 | Pump System Business |
| `page-faq` | 고객지원 / SUPPORT | FAQ | Frequently Asked Questions |
| `page-consultation` | 고객지원 / SUPPORT | 온라인 상담 | Online Consultation |
| `page-quote` | 고객지원 / SUPPORT | 견적요청 | Request a Quote |
| `page-privacy` | 고객지원 / SUPPORT | 개인정보 보호정책 | Privacy Policy |
| `page-terms` | 고객지원 / SUPPORT | 이용약관 | Terms of Service |
| `page-productimg` | 홍보센터 / MEDIA | 제품사진 | Product Gallery |
| `page-recruit` | 인재채용 / CAREERS | 채용안내 | Recruitment |
| `page-recruitfigure` | 인재채용 / CAREERS | 채용인재상 | Who We Look For |
| `page-welfare` | 인재채용 / CAREERS | 복리후생 | Benefits |
| `page-contact` | CONTACT | Contact Us | 오시는 길 · 문의 |

- [ ] **Step 3: 검증**

22개 페이지를 내비로 하나씩 열어 확인:
- 모든 페이지가 동일한 리듬으로 시작하는가
- 카테고리 마크(oxide 사각형)가 각 페이지에 정확히 1개
- 제목이 Archivo, 카테고리가 Plex Mono로 렌더되는가

```bash
grep -c "page-plate__title" index.html   # 기대: 22
grep -c "page-banner" index.html          # 기대: 0
```

- [ ] **Step 4: 커밋**

```bash
git add index.html styles.css
git commit -m "feat: 22개 하위 페이지에 공통 페이지 명판 헤더 적용"
```

---

## Task 4: 홈 — 회사 명판 히어로 + 각인 모션

**Files:**
- Modify: `index.html` (`#page-home` 히어로, 912–926행)
- Modify: `styles.css` (섹션 8)

**Interfaces:**
- Consumes: `.plate--rivet` `.plate__band` `.plate__body` `.plate__foot` `.data-table` `.btn--quote`
- Produces: `.hero` `.hero__plate` `.hero-row` (모션 대상)

- [ ] **Step 1: 히어로 마크업 교체**

기존 `.hero` 블록 전체를 교체. **`product-photo-01.jpg`는 참조하지 않는다.**

```html
<div class="hero section">
  <div class="container">
    <div class="plate plate--rivet hero__plate">
      <span class="rivet-b"></span>
      <div class="plate__band">
        <span>Nameplate</span>
        <span>DHFM · Incheon KR</span>
      </div>
      <div class="plate__body">
        <h1 class="hero__ko">대하산업기계<br>주식회사</h1>
        <div class="hero__en data">DAEHA INDUSTRIAL MACHINERY CO., LTD.</div>
        <div class="rule"></div><div class="rule--thin"></div>
        <table class="data-table">
          <tr class="hero-row"><td>MFG.</td><td>링블로워 · 고압플런저펌프 · 고압세척시스템</td></tr>
          <tr class="hero-row"><td>EST.</td><td>2016</td></tr>
          <tr class="hero-row"><td>LOC.</td><td>인천광역시 서구 봉수대로 1568번길 63-20</td></tr>
          <tr class="hero-row"><td>CERT.</td><td>ISO 9001</td></tr>
          <tr class="hero-row"><td>ORIGIN</td><td>100% 자체 설계 · 국내 생산</td></tr>
        </table>
      </div>
      <div class="plate__foot">
        <span class="data hero__serial">EST. 2016 — INCHEON</span>
        <span class="hero__btns">
          <button class="btn btn--quote" onclick="goto('quote')">견적요청 →</button>
          <button class="btn btn--ghost" onclick="goto('ringblower')">제품 보기</button>
        </span>
      </div>
    </div>
  </div>
</div>
```

- [ ] **Step 2: 히어로 스타일 + 각인 모션**

```css
/* ══════ 8. T1 홈 — 히어로 ══════ */
.hero__plate { max-width: 760px; margin-inline: auto; }
.hero__ko { font-size: clamp(2rem, 5.5vw, 3.1rem); font-weight: 600; letter-spacing: -0.03em; line-height: 1.12; }
.hero__en { font-size: 0.7rem; letter-spacing: 0.2em; color: var(--iron); margin-top: 10px; }
.hero__serial { font-size: 0.64rem; letter-spacing: 0.14em; color: var(--iron); }
.hero__btns { display: flex; gap: 10px; flex-wrap: wrap; }

/* 각인 모션 — 총 600ms, 이 사이트의 유일한 오케스트레이션 */
.hero-row { opacity: 0; animation: engrave 0.28s ease-out forwards; }
.hero-row:nth-child(1) { animation-delay: 0.08s; }
.hero-row:nth-child(2) { animation-delay: 0.19s; }
.hero-row:nth-child(3) { animation-delay: 0.30s; }
.hero-row:nth-child(4) { animation-delay: 0.41s; }
.hero-row:nth-child(5) { animation-delay: 0.52s; }

@keyframes engrave {
  from { opacity: 0; transform: translateY(-4px); }
  to   { opacity: 1; transform: none; }
}

@media (prefers-reduced-motion: reduce) {
  .hero-row { opacity: 1; animation: none; }
}
```

- [ ] **Step 3: 검증**

`http://localhost:8899/` 새로고침하여 확인:
- 데이터 5행이 위에서부터 순차로 나타나고 약 600ms에 완료
- OS 설정에서 "동작 줄이기"를 켜면 애니메이션 없이 즉시 전부 표시
- 리벳 4개가 명판 모서리에 보임
- `견적요청` 버튼이 화면에서 유일한 강한 색
- 375px 폭에서 가로 스크롤 없음

```bash
grep -c "product-photo-01" index.html   # 기대: 0
```

- [ ] **Step 4: 커밋**

```bash
git add index.html styles.css
git commit -m "feat: 홈 히어로를 회사 명판으로 교체하고 각인 모션 추가"
```

---

## Task 5: 홈 — 나머지 5개 섹션

**Files:**
- Modify: `index.html` (`#page-home` 928–1049행)
- Modify: `styles.css` (섹션 8 이어서)

**Interfaces:**
- Consumes: `.plate` `.btn--quote` `.data`
- Produces: `.prod-grid` `.prod-plate` `.proof-strip` `.biz-strip` `.quote-band` `.contact-grid`

- [ ] **Step 1: 제품 대표 수치 판독**

`images/` 내 카탈로그 이미지를 열어 4개 제품의 범위값을 판독한다.

| 제품 | 판독 대상 파일 |
|---|---|
| 링블로워 | `ringblower-04-specs.jpg` |
| 플런저펌프 | `plungerpump-large-specs.png`, `plungerpump-medium-specs.png`, `plungerpump-small-specs.png` |
| 고압세척기 | `washer-specs.png` |
| 세척시스템 | `washingsystem-specs1.png` |

각 제품마다 **최대 압력 / 최대 유량 / 출력 범위** 3개 값을 뽑는다.
확인된 예시: 플런저펌프 F5000/F6000 계열 — `200 bar` / `1717 L/min` / `530 kW`.

**판독한 값은 아래 형식으로 별도 기록하여 대표님 검수 요청한다. 검수 전에는 main 병합 금지.**

```
docs/superpowers/specs/2026-07-19-제품수치-검수요청.md
```

- [ ] **Step 2: 제품 3종 그리드 마크업**

```html
<div class="section">
  <div class="container">
    <div class="prod-grid">
      <a class="plate prod-plate" href="#" onclick="goto('ringblower')">
        <div class="plate__band"><span>01 / Ring Blower</span><span>DH Series</span></div>
        <img src="images/gallery/rb-01.jpg" alt="대하산업기계 링블로워 DH 시리즈"
             width="676" height="960" loading="lazy">
        <div class="plate__body">
          <h3 class="prod-plate__ko">링블로워</h3>
          <table class="data-table">
            <tr><td>PRESS.</td><td>[Step 1 판독값]</td></tr>
            <tr><td>FLOW</td><td>[Step 1 판독값]</td></tr>
            <tr><td>POWER</td><td>[Step 1 판독값]</td></tr>
          </table>
        </div>
      </a>
      <!-- 플런저펌프: images/gallery/pump-01.jpg, 세척시스템: images/gallery/syspump-01.jpg 로 동일 구조 반복 -->
    </div>
  </div>
</div>
```

```css
.prod-grid { display: grid; grid-template-columns: repeat(3, 1fr); gap: 20px; }
.prod-plate { text-decoration: none; display: block; transition: border-color 0.15s; }
.prod-plate:hover { border-color: var(--cast); }
.prod-plate img { border-block: 1px solid var(--machined); background: #fff;
                  aspect-ratio: 4/3; object-fit: contain; padding: 20px; }
.prod-plate__ko { font-size: 1.15rem; margin-bottom: 14px; }
@media (max-width: 860px) { .prod-grid { grid-template-columns: 1fr; } }
```

- [ ] **Step 3: 근거 4항 띠**

**`A/S 신속대응` → `5축가공·유동해석` 교체 확정 사항을 반영한다.**

```html
<div class="container">
  <div class="plate proof-strip">
    <div class="proof"><span class="num">2016</span><span>설립</span></div>
    <div class="proof"><span class="num">ISO 9001</span><span>품질경영 인증</span></div>
    <div class="proof"><span class="num">100%</span><span>자체 설계·국내 생산</span></div>
    <div class="proof"><span class="num">5-AXIS</span><span>5축 가공 · 유동해석 설계</span></div>
  </div>
</div>
```

```css
.proof-strip { display: grid; grid-template-columns: repeat(4, 1fr); }
.proof { padding: 26px var(--pad); border-right: 1px solid var(--machined);
         display: flex; flex-direction: column; gap: 6px; }
.proof:last-child { border-right: 0; }
.proof .num { font-family: var(--font-data); font-size: 1.35rem; font-weight: 600; letter-spacing: -0.01em; }
.proof span:last-child { font-size: 0.78rem; color: var(--iron); line-height: 1.5; }
@media (max-width: 860px) {
  .proof-strip { grid-template-columns: repeat(2, 1fr); }
  .proof:nth-child(2n) { border-right: 0; }
  .proof:nth-child(-n+2) { border-bottom: 1px solid var(--machined); }
}
```

- [ ] **Step 4: 사업분야 띠 + 견적요청 밴드**

```html
<div class="section container">
  <div class="plate biz-strip">
    <a href="#" onclick="goto('blowerbs')"><span class="data">01</span>블로워사업</a>
    <a href="#" onclick="goto('pumpbs')"><span class="data">02</span>펌프사업</a>
    <a href="#" onclick="goto('pumpsystembs')"><span class="data">03</span>펌프시스템사업</a>
  </div>
</div>

<div class="quote-band">
  <div class="container quote-band__inner">
    <div>
      <h2 class="quote-band__title">필요한 사양을 알려주시면 회신드립니다</h2>
      <p class="quote-band__sub">모델·용도·수량만 있으면 검토 가능합니다. 특수 사양 주문 제작도 상담해 드립니다.</p>
    </div>
    <button class="btn btn--quote" onclick="goto('quote')">견적요청 →</button>
  </div>
</div>
```

```css
.biz-strip { display: grid; grid-template-columns: repeat(3, 1fr); }
.biz-strip a { display: flex; align-items: center; gap: 14px; padding: 24px var(--pad);
               text-decoration: none; border-right: 1px solid var(--machined); transition: background 0.15s; }
.biz-strip a:last-child { border-right: 0; }
.biz-strip a:hover { background: var(--cast); color: var(--plate); }
.biz-strip .data { font-size: 0.7rem; color: var(--iron); letter-spacing: 0.1em; }
.biz-strip a:hover .data { color: var(--machined); }

.quote-band { background: var(--cast); color: var(--plate); padding-block: var(--gap); }
.quote-band__inner { display: flex; align-items: center; justify-content: space-between; gap: 32px; flex-wrap: wrap; }
.quote-band__title { font-size: clamp(1.2rem, 2.6vw, 1.7rem); font-weight: 600; }
.quote-band__sub { color: var(--machined); font-size: 0.88rem; margin-top: 8px; max-width: 46ch; }
@media (max-width: 860px) { .biz-strip { grid-template-columns: 1fr; }
  .biz-strip a { border-right: 0; border-bottom: 1px solid var(--machined); } }
```

- [ ] **Step 5: 연락처 + 지도 — 폼 제거**

**홈의 문의 폼(1034–1044행)을 삭제한다.**

`sendMsg()` 함수 제거 여부는 **먼저 확인한 뒤 판단한다.** 이 함수는 `h-*` ID와 `.btn-send` 선택자를 쓰는데, Contact 페이지에도 `.btn-send` 클래스 버튼이 있을 수 있다. 다음으로 확인한다:

```bash
grep -n "sendMsg\|btn-send" index.html
```

- 홈 폼 삭제 후 `sendMsg` 호출부가 0개면 → 함수 정의도 제거한다.
- Contact 페이지가 `sendMsg`를 공유하고 있으면 → **함수를 남기고**, Contact 폼의 ID를 그대로 둔 채 홈 마크업만 제거한다.

확인 없이 삭제하면 Contact 페이지의 전송이 깨진다.

지도는 클라이언트 ID가 플레이스홀더라 렌더되지 않으므로 **대체 표시를 넣는다**:

```html
<div class="section container">
  <div class="contact-grid">
    <div class="plate">
      <div class="plate__band"><span>Location</span><span>Incheon</span></div>
      <div class="plate__body">
        <table class="data-table">
          <tr><td>ADDR.</td><td>인천광역시 서구 봉수대로 1568번길 63-20<br>(서구 금곡동 273-38)</td></tr>
          <tr><td>TEL.</td><td>032-567-9087</td></tr>
          <tr><td>FAX.</td><td>032-567-9088</td></tr>
          <tr><td>MAIL.</td><td>master@dhfm.co.kr</td></tr>
        </table>
        <a class="btn btn--ghost" style="margin-top:20px"
           href="https://map.naver.com/p/search/인천광역시 서구 봉수대로 1568번길 63-20"
           target="_blank" rel="noopener">네이버지도에서 보기 →</a>
      </div>
    </div>
    <div id="naver-map" class="plate map-slot"></div>
  </div>
</div>
```

```css
.contact-grid { display: grid; grid-template-columns: 1fr 1.3fr; gap: 20px; align-items: start; }
.map-slot { min-height: 380px; background: var(--machined); }
@media (max-width: 860px) { .contact-grid { grid-template-columns: 1fr; } }
```

- [ ] **Step 6: 검증**

- 홈에 문의 폼이 없다: `grep -c "h-name" index.html` → 기대 `0`
- 제품 카드 3개의 수치가 고정폭으로 자릿수 정렬됨
- 근거 4항 마지막이 `5-AXIS / 5축 가공 · 유동해석 설계`
- 견적요청 밴드가 홈에서 유일하게 어두운 배경
- 375px에서 모든 그리드가 1열로 접히고 가로 스크롤 없음

- [ ] **Step 7: 커밋**

```bash
git add index.html styles.css docs/
git commit -m "feat: 홈 나머지 섹션 재구성 — 제품 그리드·근거 4항·견적 밴드·연락처, 홈 폼 제거"
```

---

## Task 6: T2 제품 상세 4종

**Files:**
- Modify: `index.html` (`#page-ringblower` `#page-plungerpump` `#page-washer` `#page-washingsystem`)
- Modify: `styles.css` (섹션 9)

**Interfaces:**
- Consumes: `.plate` `.page-plate` `.btn--quote` `.data-table`
- Produces: `.prod-detail` `.spec-figure` — Task 7이 카탈로그 이미지 프레임에 재사용

- [ ] **Step 1: 카탈로그 이미지 프레임 + 라이트박스 연동 스타일**

```css
/* ══════ 9. T2 제품 상세 ══════ */
.prod-detail { display: grid; grid-template-columns: 1fr 1fr; gap: 20px; align-items: start; }
.prod-hero-img { background: #fff; padding: 32px; aspect-ratio: 1; object-fit: contain; }

.spec-figure { margin-top: 20px; }
.spec-figure img { cursor: zoom-in; background: #fff; }
.spec-figure figcaption {
  font-family: var(--font-data); font-size: 0.66rem; letter-spacing: 0.14em;
  text-transform: uppercase; color: var(--iron);
  padding: 10px var(--pad); border-top: 1px solid var(--machined);
  display: flex; justify-content: space-between; gap: 12px;
}
@media (max-width: 860px) { .prod-detail { grid-template-columns: 1fr; } }

/* 라이트박스 기본 — Task 9의 갤러리보다 먼저 필요하므로 여기서 정의한다.
   Task 9은 갤러리 전용 스타일만 추가하고 이 블록을 중복 정의하지 않는다. */
.lightbox { display: none; position: fixed; inset: 0; z-index: 2000;
            background: rgba(35,38,42,0.94); place-items: center; padding: 40px; }
.lightbox.open { display: grid; }
.lightbox-img { max-width: 92vw; max-height: 88vh; object-fit: contain; }
.lightbox--single .lightbox-prev, .lightbox--single .lightbox-next { display: none; }
```

- [ ] **Step 2: 링블로워 페이지를 완성형으로 작성**

나머지 3개 제품이 따를 기준이다.

```html
<div class="section container">
  <div class="prod-detail">
    <div class="plate">
      <div class="plate__band"><span>Ring Blower</span><span>DH Series</span></div>
      <img class="prod-hero-img" src="images/gallery/rb-01.jpg"
           alt="링블로워 DH 시리즈 본체" width="676" height="960" loading="lazy">
    </div>
    <div class="plate">
      <div class="plate__band"><span>Specification</span><span>제원</span></div>
      <div class="plate__body">
        <table class="data-table">
          <tr><td>PRESS.</td><td>[Task 5 Step 1 판독값]</td></tr>
          <tr><td>FLOW</td><td>[Task 5 Step 1 판독값]</td></tr>
          <tr><td>POWER</td><td>[Task 5 Step 1 판독값]</td></tr>
          <tr><td>MODEL</td><td>DH-5.5-H ~ DH-37-H</td></tr>
        </table>
      </div>
      <div class="plate__foot">
        <span class="data" style="font-size:.64rem;color:var(--iron)">특수 사양 주문 제작 가능</span>
        <button class="btn btn--quote" onclick="goto('quote')">이 제품 견적요청 →</button>
      </div>
    </div>
  </div>

  <!-- 특징: 기존 본문 유지, 플레이트로 감싸기 -->
  <div class="plate" style="margin-top:20px">
    <div class="plate__band"><span>Features</span><span>특징</span></div>
    <div class="plate__body measure">
      <!-- 기존 #page-ringblower 의 특징 본문을 그대로 이동 -->
    </div>
  </div>

  <!-- 카탈로그 이미지: 프레임 + 라이트박스 + lazy -->
  <figure class="plate spec-figure">
    <img src="images/ringblower-04-specs.jpg" alt="링블로워 제원표"
         width="1400" height="1980" loading="lazy"
         onclick="openCatalog(this.src, '링블로워 제원표')">
    <figcaption><span>Catalog — 제원표</span><span>클릭하여 확대</span></figcaption>
  </figure>
</div>
```

- [ ] **Step 3: 카탈로그 라이트박스 함수 추가**

기존 갤러리 라이트박스를 재사용하는 얇은 진입점을 `<script>`에 추가:

```javascript
function openCatalog(src, alt) {
  const box = document.getElementById('lightbox');
  const img = document.getElementById('lightbox-img');
  img.src = src; img.alt = alt || '';
  box.classList.add('open');
  document.body.style.overflow = 'hidden';
  // 카탈로그는 단일 이미지이므로 이전/다음 화살표를 숨긴다
  box.classList.add('lightbox--single');
}
```

`.lightbox--single` 스타일은 Step 1에서 이미 정의했다.

**`closeLightbox`를 수정한다.** 기존 함수 본문에서 `classList.remove('open')` 호출 바로 다음 줄에 아래를 추가하여, 카탈로그 라이트박스를 닫은 뒤 갤러리 라이트박스의 좌우 화살표가 복구되도록 한다:

```javascript
box.classList.remove('lightbox--single');
```

이 한 줄이 없으면 카탈로그 확대를 한 번 연 뒤부터 갤러리에서 이전/다음 이동이 영구히 사라진다. Task 6 Step 5에서 반드시 확인할 것.

- [ ] **Step 4: 나머지 3개 제품 페이지에 동일 구조 적용**

| 페이지 | 히어로 이미지 | 카탈로그 이미지 |
|---|---|---|
| `page-plungerpump` | `images/gallery/pump-01.jpg` | `plungerpump-large-specs.png`, `plungerpump-medium-specs.png`, `plungerpump-small-specs.png` |
| `page-washer` | `images/gallery/pump-03.jpg` | `washer-specs.png`, `washer-photo.jpg`(노즐·호스 선정표로 캡션 표기) |
| `page-washingsystem` | `images/gallery/syspump-01.jpg` | `washingsystem-specs1.png`, `washingsystem-specs2.png`, `washingsystem-specs3.png` |

**주의:** `plungerpump-photo.jpg`와 `washer-photo.jpg`는 이름과 달리 사진이 아니라 표다. 히어로 이미지로 쓰지 말고 `spec-figure`에 캡션을 정확히 달아 배치한다.

- [ ] **Step 5: 검증**

4개 제품 페이지에서 각각 확인:
- 히어로 이미지가 흰 배경에 여백을 두고 표시
- 제원 플레이트의 수치가 고정폭 정렬
- 카탈로그 이미지 클릭 시 라이트박스 확대, 좌우 화살표 없음, ESC/배경 클릭으로 닫힘
- 라이트박스 닫은 뒤 갤러리 페이지의 라이트박스가 정상 동작 (화살표 복구)
- 네트워크 탭에서 카탈로그 이미지가 초기 로드에 포함되지 않음 (lazy 동작)
- `이 제품 견적요청` 버튼이 견적요청 페이지로 이동

- [ ] **Step 6: 커밋**

```bash
git add index.html styles.css
git commit -m "feat: 제품 상세 4종을 명판 레이아웃으로 교체, 카탈로그 이미지 라이트박스·lazy 적용"
```

---

## Task 7: T3 사업분야 3종

**Files:**
- Modify: `index.html` (`#page-blowerbs` `#page-pumpbs` `#page-pumpsystembs`)
- Modify: `styles.css` (섹션 10)

**Interfaces:**
- Consumes: `.plate` `.spec-figure` `openCatalog()`

- [ ] **Step 1: 이미지 스택 스타일**

```css
/* ══════ 10. T3 사업분야 ══════ */
.biz-stack { display: flex; flex-direction: column; gap: 20px; }
.biz-stack .spec-figure { margin-top: 0; }
```

- [ ] **Step 2: 3개 페이지의 이미지들을 `spec-figure`로 감싸기**

각 페이지의 기존 `<img>`를 전부 `figure.plate.spec-figure` 패턴으로 교체한다.
대상 이미지: `blowerbs-01~05.png`, `pumpbs-01~07.jpg`, `pumpsystembs-01~03.jpg`

패턴:
```html
<figure class="plate spec-figure">
  <img src="images/blowerbs-01.png" alt="블로워사업 — [내용 설명]"
       width="[실제값]" height="[실제값]" loading="lazy"
       onclick="openCatalog(this.src, '블로워사업 자료')">
  <figcaption><span>Blower Business — 01</span><span>클릭하여 확대</span></figcaption>
</figure>
```

**`alt` 텍스트는 이미지를 실제로 열어보고 내용에 맞게 작성한다.** "이미지1" 같은 무의미한 alt 금지.
`width`/`height`는 실제 픽셀 크기를 확인해 넣는다.

- [ ] **Step 3: 검증**

- 3개 페이지 모두 이미지가 명판 프레임 안에 표시
- 각 이미지에 의미 있는 `alt` 존재
- 클릭 시 확대, 초기 로드에 미포함
- 375px에서 이미지가 넘치지 않음

- [ ] **Step 4: 커밋**

```bash
git add index.html styles.css
git commit -m "feat: 사업분야 3종 이미지를 명판 프레임 + 라이트박스로 교체"
```

---

## Task 8: T4 문서형 7종 + T5 데이터형 3종

**Files:**
- Modify: `index.html` (`#page-ceo` `#page-policy` `#page-privacy` `#page-terms` `#page-recruit` `#page-recruitfigure` `#page-welfare` `#page-history` `#page-organization` `#page-certification`)
- Modify: `styles.css` (섹션 11, 12)

**Interfaces:**
- Consumes: `.plate` `.measure` `.rule` `.data`

- [ ] **Step 1: 문서형 스타일**

```css
/* ══════ 11. T4 문서 ══════ */
.doc { max-width: var(--measure); }
.doc h2 { font-size: 1.25rem; margin-top: 40px; padding-bottom: 10px;
          border-bottom: 2px solid var(--cast); }
.doc h2:first-child { margin-top: 0; }
.doc h3 { font-size: 1rem; margin-top: 28px; color: var(--iron); }
.doc p  { margin-top: 14px; }
.doc ul, .doc ol { margin: 14px 0 0 20px; }
.doc li { margin-top: 7px; }
.doc strong { font-weight: 600; }
```

- [ ] **Step 2: 7개 문서 페이지 본문을 `.doc.measure`로 감싸기**

**본문 텍스트는 한 글자도 바꾸지 않는다.** 감싸는 구조만 교체:

```html
<div class="section container">
  <div class="doc">
    <!-- 기존 본문 그대로 -->
  </div>
</div>
```

- [ ] **Step 3: 회사연혁 — 고정폭 연도 정렬**

```css
/* ══════ 12. T5 데이터 ══════ */
.history { border-top: 1px solid var(--machined); }
.history__row { display: grid; grid-template-columns: 120px 1fr; gap: 24px;
                padding: 18px 0; border-bottom: 1px solid var(--machined); }
.history__year { font-family: var(--font-data); font-weight: 600; font-size: 1.05rem;
                 font-variant-numeric: tabular-nums; }
@media (max-width: 700px) { .history__row { grid-template-columns: 1fr; gap: 4px; } }
```

기존 연혁 항목을 `.history__row` 패턴으로 재조립한다. 연도는 `.history__year`로 감싼다.

- [ ] **Step 4: 조직도·인증서 — 플레이트 그리드**

```css
.plate-grid { display: grid; grid-template-columns: repeat(auto-fit, minmax(240px, 1fr)); gap: 20px; }
.cert-figure img { background: #fff; padding: 20px; aspect-ratio: 3/4; object-fit: contain; }
```

인증서 이미지(`ISO 9001_kor.jpg`)는 `spec-figure` 패턴 + `openCatalog` 적용.

- [ ] **Step 5: 검증**

10개 페이지 확인:
- 문서 본문 폭이 읽기 좋은 범위(약 68자)로 제한됨
- 본문 내용이 변경되지 않았음: `git diff` 로 텍스트 노드 변경이 없는지 확인
- 연혁의 연도가 세로로 자릿수 정렬됨
- 인증서 클릭 시 확대

- [ ] **Step 6: 커밋**

```bash
git add index.html styles.css
git commit -m "feat: 문서형 7종·데이터형 3종 페이지 레이아웃 교체"
```

---

## Task 9: T6 폼 3종 + T7 갤러리 + T8 FAQ

**Files:**
- Modify: `index.html` (`#page-quote` `#page-consultation` `#page-contact` `#page-productimg` `#page-faq`)
- Modify: `styles.css` (섹션 13, 14, 15)

**Interfaces:**
- Consumes: `.plate` `.btn--quote`
- **보존 필수:** `cf-*` ID(`c-name` `q-name` 등), `ct-*` ID, `switchTab` `toggleFaq` `buildGallery` `switchGalleryTab` `openLightbox` 호출부

- [ ] **Step 1: 폼 스타일**

```css
/* ══════ 13. T6 폼 ══════ */
.cf-group { margin-bottom: 18px; }
.cf-label { display: block; font-family: var(--font-data); font-size: 0.68rem;
            letter-spacing: 0.12em; text-transform: uppercase; color: var(--iron); margin-bottom: 7px; }
.cf-input, .cf-textarea, .cf-select {
  width: 100%; padding: 13px 14px;
  background: #fff; border: 1px solid var(--machined);
  font-family: var(--font-body); font-size: 0.92rem; color: var(--cast);
}
.cf-input:focus, .cf-textarea:focus, .cf-select:focus {
  outline: 2px solid var(--oxide); outline-offset: -1px; border-color: var(--oxide);
}
.cf-textarea { min-height: 160px; resize: vertical; }
.cf-row { display: grid; grid-template-columns: 1fr 1fr; gap: 18px; }
.cf-req { color: var(--oxide); }
.cf-success { display: none; margin-top: 16px; padding: 14px;
              border: 1px solid var(--cast); font-size: 0.88rem; }
@media (max-width: 700px) { .cf-row { grid-template-columns: 1fr; } }
```

**필수 표시:** 기존 `성함 *` 형태의 `*`를 `<span class="cf-req">*</span>`로 감싼다.

- [ ] **Step 2: 3개 폼 페이지를 플레이트로 감싸기**

```html
<div class="section container">
  <div class="plate" style="max-width:760px;margin-inline:auto">
    <div class="plate__band"><span>Request a Quote</span><span>견적요청</span></div>
    <div class="plate__body">
      <!-- 기존 폼 마크업 그대로. ID·onclick 변경 금지 -->
    </div>
  </div>
</div>
```

- [ ] **Step 3: 갤러리 + 라이트박스 스타일**

```css
/* ══════ 14. T7 갤러리 ══════ */
.gallery-tabs { display: flex; gap: 0; border-bottom: 1px solid var(--machined); margin-bottom: 24px; }
.gallery-tabs button, .faq-tabs button {
  padding: 14px 22px; background: none; border: 0; cursor: pointer;
  font-family: var(--font-data); font-size: 0.76rem; letter-spacing: 0.1em; color: var(--iron);
  border-bottom: 2px solid transparent;
}
.gallery-tabs button.active, .faq-tabs button.active {
  color: var(--cast); border-bottom-color: var(--oxide);   /* oxide 허용처 2/3 */
}
.gallery-panel { display: none; }
.gallery-panel.active { display: grid; grid-template-columns: repeat(auto-fill, minmax(220px, 1fr)); gap: 16px; }
.gallery-item { background: #fff; border: 1px solid var(--machined); padding: 16px;
                cursor: zoom-in; aspect-ratio: 1; }
.gallery-item img { width: 100%; height: 100%; object-fit: contain; }

/* 라이트박스 기본 스타일은 Task 6 Step 1에서 이미 정의했다. 여기서 중복 정의하지 않는다. */

/* ══════ 15. T8 FAQ ══════ */
.faq-item { border-bottom: 1px solid var(--machined); }
.faq-q { display: flex; justify-content: space-between; gap: 16px; align-items: center;
         padding: 20px 0; cursor: pointer; font-weight: 500; }
.faq-a { display: none; padding: 0 0 22px; color: var(--iron); max-width: var(--measure); }
.faq-item.open .faq-a { display: block; }
.faq-item.open .faq-q { color: var(--oxide); }
```

- [ ] **Step 4: 검증**

- 갤러리 4개 탭 전환 동작, 각 탭의 이미지가 그리드로 표시
- 이미지 클릭 → 라이트박스, 좌우 화살표로 이동, ESC/배경 클릭으로 닫힘
- FAQ 3개 탭 전환 + 아코디언 열림/닫힘
- 3개 폼에서 필수 항목 비우고 제출 → 기존 alert 정상 동작
- 폼 입력란 포커스 시 oxide 아웃라인
- `grep -c "formspree.io/f/meedkznw" index.html` → 기대 `3` (홈 폼 제거로 4→3)

- [ ] **Step 5: 커밋**

```bash
git add index.html styles.css
git commit -m "feat: 폼 3종·갤러리·FAQ 외형 교체 (JS 동작 보존)"
```

---

## Task 10: 반응형 · 접근성 · 최종 검수

**Files:**
- Modify: `styles.css` (섹션 16)
- Modify: `index.html` (SEO 메타 점검)

- [ ] **Step 1: SEO 메타 점검**

`og:image`가 삭제 대상인 스톡 사진을 가리키지 않는지 확인하고, 실제 제품 이미지로 교체:

```html
<meta property="og:image" content="https://www.dhfm.co.kr/images/gallery/rb-01.jpg">
```

JSON-LD의 `logo` 필드도 동일하게 점검한다.

- [ ] **Step 2: 23개 페이지 전수 확인**

데스크톱(1280px)과 모바일(375px) 양쪽에서 23개 페이지를 모두 연다. 각 페이지에서:

| 확인 항목 | 기준 |
|---|---|
| 가로 스크롤 | 375px에서 발생하지 않음 |
| 페이지 명판 | 카테고리 마크 1개, 제목, 각인선 존재 |
| oxide 사용 | 페이지당 CTA·활성표시·카테고리마크 외 등장하지 않음 |
| 이미지 | 전부 `alt` 보유, 프레임 안에 수납 |
| 텍스트 대비 | 본문 4.5:1 이상 |

- [ ] **Step 3: 키보드 전수 확인**

Tab만으로 다음이 가능해야 한다:
- 헤더 내비 6개 카테고리 진입 → 드롭다운 항목 도달
- 모든 버튼·링크·입력란에 보이는 포커스 링
- 라이트박스 열린 상태에서 ESC로 닫기

- [ ] **Step 4: 리벳 최종 판단**

홈 히어로의 리벳 4개를 실제 화면에서 본다. 스큐어모픽하게 읽히면 `.plate--rivet` 적용을 제거한다. 이것은 구현자의 시각 판단 사항이며, 제거하는 쪽이 기본값이다.

- [ ] **Step 5: 잔여물 정리**

```bash
grep -c "product-photo-01" index.html   # 기대: 0
grep -c "page-banner"      index.html   # 기대: 0
grep -c "<style>"          index.html   # 기대: 0
grep -c "navy\|#0a1628\|#c8922a" index.html styles.css   # 기대: 0
```

- [ ] **Step 6: 커밋**

```bash
git add index.html styles.css
git commit -m "chore: 반응형·접근성 최종 검수 및 SEO 메타 정리"
```

---

## Task 11: 검수 요청 문서 작성

**Files:**
- Create: `docs/superpowers/specs/2026-07-19-제품수치-검수요청.md`

- [ ] **Step 1: 판독 수치 정리**

Task 5 Step 1에서 카탈로그 이미지로부터 판독한 모든 수치를 아래 형식으로 정리한다:

```markdown
# 제품 대표 수치 검수 요청

아래 수치는 `images/` 내 카탈로그 이미지에서 판독한 값입니다.
**게시 전 확인 부탁드립니다. 틀린 값이 있으면 알려주시면 수정합니다.**

| 제품 | 항목 | 판독값 | 출처 파일 | 확인 |
|---|---|---|---|---|
| 링블로워 | 최대 압력 | ... | ringblower-04-specs.jpg | ☐ |
| ... | ... | ... | ... | ☐ |

## 그 밖에 확인이 필요한 사항
- 네이버 지도 Client ID: 현재 `YOUR_CLIENT_ID` 플레이스홀더 상태로 지도가 표시되지 않습니다.
  네이버 클라우드 플랫폼에서 발급받은 ID가 필요합니다.
```

- [ ] **Step 2: 커밋**

```bash
git add docs/
git commit -m "docs: 제품 대표 수치 검수 요청서 작성"
```

---

## 완료 조건

- [ ] 23개 페이지 전부 데스크톱·모바일에서 확인 완료
- [ ] 기존 JS 함수 16개 전부 정상 동작
- [ ] `--oxide`가 허용된 3곳 외에 등장하지 않음
- [ ] 375px에서 가로 스크롤 없음
- [ ] 키보드만으로 전체 탐색 가능
- [ ] `prefers-reduced-motion`에서 애니메이션 없음
- [ ] 제품 수치 검수 요청서 전달 완료
- [ ] **대표님 수치 검수 완료 전까지 main 병합하지 않음**
