# AWS Summit Seoul 2026 — 세션 리뷰 페이지 제작 PRD

> **작성자**: 물류플랫폼팀 (jenny.by)
> **대상**: daniel.007 (Day 1), zero.002 (Day 2)
> **목적**: 컨퍼런스 현장 메모 + 사진 + 동영상 → GitHub Pages 공유 가능한 세션 리뷰 페이지

---

## 0. 전체 워크플로우

```
[Claude Code에 업로드]               [산출물 저장 위치]

  SESSION_PRD.md          ─┐
  SESSION_TEMPLATE.html   ─┤         conference_review/
  현장 사진 (JPG)          ─┤  →     └── github_page/
  메모 텍스트              ─┤             ├── main_index.html   ← 메인 허브
  동영상 (MP4/MOV, 선택)   ─┘             └── aws/
                                               ├── index.html   ← AWS 허브
                                               └── 01-session-name/
                                                   ├── index.html   ← 세션 리뷰 페이지
                                                   └── videos/      ← 동영상 (있는 경우)

[개인 원본 자료 — push 제외]
  workfiles/aws-summit-2026/[본인이름]/   ← 사진, 메모, 녹음 등
  (.gitignore에 의해 자동 제외됨)

[GitHub Pages 배포]
  github_page/ 폴더째로 push → https://brad-and.github.io/conference_review/aws/01-session-name/
```

---

## 1. 산출물 폴더 구조

### 세션 폴더 위치

```
conference_review/github_page/aws/
  01-session-name/
    index.html         ← 세션 리뷰 메인 페이지 (이미지 base64 embed)
    videos/            ← 동영상 파일 (있는 경우만)
      session-clip.mp4
```

### 폴더명 규칙

| 규칙 | 예시 |
|------|------|
| `숫자두자리-영문키워드` 형식 | `01-bedrock-rag` |
| 소문자 + 하이픈만 | `02-eks-upgrade` |
| 한글·공백·특수문자 금지 | `03-ai-leadership` |

### 핵심 원칙

- **이미지**: base64 embed → `index.html` 안에 완전 포함, GitHub 경로 이슈 없음
- **동영상**: `videos/` 폴더에 파일 복사 → `<video src="./videos/파일명.mp4">` 상대경로
- **모든 경로**: 상대경로만 사용. `file://` 또는 `/Users/...` 절대경로 금지
- **파일 크기**: `index.html`은 25MB 미만 유지

---

## 2. 디자인 시스템

### CSS 변수 (반드시 사용)

```css
:root {
  --bg-base:     #0d0d0d;
  --bg-surface:  #161616;
  --bg-elevated: #1f1f1f;
  --bg-card:     #1a1a1a;

  --green:       #1ed760;   /* 강조 / 완료 */
  --aws:         #FF9900;   /* AWS 오렌지 */
  --orange:      #ffa42b;   /* jenny.by */
  --blue:        #539df5;   /* brad.and */
  --purple:      #b39ddb;   /* daniel.007 */
  --teal:        #4ecdc4;   /* zero.002 */
  --pink:        #f3727f;
  --yellow:      #f5c842;

  --text-primary:   #ffffff;
  --text-secondary: #888888;
  --border:         #252525;

  --radius:      10px;
  --radius-pill: 9999px;
}
```

### 참석자 뱃지

```html
<!-- daniel.007 (Day 1) — purple -->
<span class="badge-attendee daniel">● daniel.007</span>

<!-- zero.002 (Day 2) — teal -->
<span class="badge-attendee zero">● zero.002</span>
```

```css
.badge-attendee {
  font-size: 11px; font-weight: 700;
  padding: 4px 12px; border-radius: var(--radius-pill);
  letter-spacing: 0.06em; display: inline-flex; align-items: center; gap: 5px;
}
.badge-attendee.daniel {
  background: rgba(179,157,219,0.18); color: var(--purple);
  border: 1px solid rgba(179,157,219,0.4);
}
.badge-attendee.zero {
  background: rgba(78,205,196,0.18); color: var(--teal);
  border: 1px solid rgba(78,205,196,0.4);
}
```

---

## 3. 세션 페이지 구조

### 필수 섹션 순서

```
[1] NAV          브레드크럼 네비게이션
[2] HERO         세션 제목, 발표자, 현장 사진 배경, 메타 정보
[3] STATS STRIP  핵심 숫자 3~4개
[4] SECTIONS     노트 섹션들 (아래 컴포넌트 조합)
[5] FOOTER       페이지 하단
```

### [1] NAV

세션 파일 위치: `github_page/aws/01-session-name/index.html`

```html
<nav>
  <a href="../../index.html" class="nav-back">← 아카이브</a>
  <span class="nav-sep">/</span>
  <a href="../index.html" class="nav-back">AWS Summit Seoul 2026</a>
  <span class="nav-sep">/</span>
  <span class="nav-current">세션명</span>
</nav>
```

```css
nav {
  position: sticky; top: 0; z-index: 100;
  background: rgba(13,13,13,0.92); backdrop-filter: blur(12px);
  border-bottom: 1px solid var(--border);
  padding: 14px 48px;
  display: flex; align-items: center; gap: 12px;
}
.nav-back    { font-size: 13px; color: var(--text-secondary); text-decoration: none; transition: color .2s; }
.nav-back:hover { color: var(--text-primary); }
.nav-sep     { color: var(--border); font-size: 16px; }
.nav-current { font-size: 13px; color: var(--text-secondary); }
```

### [2] HERO

```html
<div class="hero">
  <div class="hero-bg">
    <img src="data:image/jpeg;base64,..." alt="세션 현장" />
  </div>
  <div class="container">
    <div class="hero-eyebrow">
      <span class="badge-aws">AWS</span>
      <span class="badge-outline">트랙명</span>
      <span class="badge-attendee daniel">daniel.007</span>
    </div>
    <h1>세션 제목<br><span class="accent">강조 키워드</span></h1>
    <p class="hero-desc">한 줄 요약 메시지</p>
    <div class="hero-meta-row">
      <div class="hero-meta-item">
        <div class="label">발표자</div>
        <div class="value">이름 · 소속</div>
      </div>
      <div class="hero-meta-item">
        <div class="label">날짜</div>
        <div class="value">2026년 5월 20일 (Day 1)</div>
      </div>
      <div class="hero-meta-item">
        <div class="label">트랙</div>
        <div class="value">트랙명</div>
      </div>
    </div>
  </div>
</div>
```

### [3] STATS STRIP

```html
<div class="container">
  <div class="stats-strip">
    <div class="stat-cell">
      <div class="number">숫자</div>
      <div class="desc">설명</div>
    </div>
    <!-- 3~4개 반복 -->
  </div>
</div>
```

### [4] 노트 섹션 컴포넌트

#### 기본 텍스트 섹션
```html
<div class="container">
  <section class="border-top">
    <div class="section-label">LABEL</div>
    <h2 class="section-title">섹션 제목</h2>
    <p class="section-sub">요약</p>
    <!-- 내용 -->
  </section>
</div>
```

#### Before / After 비교
```html
<div class="problem-grid">
  <div class="problem-card before">
    <div class="tag">기존</div>
    <h3>기존 방식</h3>
    <p>설명</p>
  </div>
  <div class="problem-card after">
    <div class="tag">변화</div>
    <h3>새로운 방식</h3>
    <p>설명</p>
  </div>
</div>
```

#### 핵심 수치 강조
```html
<div class="insight-box">
  <div>
    <div class="big-num">38%</div>
    <div class="big-label">지표 설명</div>
  </div>
  <div class="divider"></div>
  <div class="conclusion">결론 텍스트. <span>핵심 키워드</span></div>
</div>
```

#### 인라인 슬라이드 사진
```html
<div class="slide-inline">
  <img src="data:image/jpeg;base64,..." alt="슬라이드 설명" loading="lazy">
  <div class="slide-inline-cap">캡션</div>
</div>
```

#### 인용 박스
```html
<div class="reaction-box">
  <div class="reaction-text">"인상적인 발언"</div>
  <div class="reaction-sub">— 발표자 이름 / 컨텍스트</div>
</div>
```

#### 번호 목록
```html
<div class="obstacle-list">
  <div class="obstacle-item">
    <div class="obstacle-num">① 단계명</div>
    <p>설명</p>
  </div>
</div>
```

#### 동영상 (로컬 파일)
```html
<div class="video-wrap">
  <video controls preload="metadata">
    <source src="./videos/session-clip.mp4" type="video/mp4">
    <source src="./videos/session-clip.mov" type="video/quicktime">
    <p>이 브라우저는 동영상을 지원하지 않습니다.</p>
  </video>
</div>
<p class="video-caption">영상 설명</p>
```

```css
.video-wrap {
  border-radius: var(--radius); overflow: hidden;
  background: #000; border: 1px solid var(--border);
  margin: 24px 0;
}
.video-wrap video { width: 100%; display: block; }
.video-caption { font-size: 13px; color: var(--text-secondary); margin-top: 8px; }
```

#### 동영상 (YouTube / Google Drive embed)
```html
<!-- YouTube -->
<div class="video-wrap iframe-wrap">
  <iframe src="https://www.youtube.com/embed/VIDEO_ID"
    title="세션명" frameborder="0"
    allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture"
    allowfullscreen></iframe>
</div>

<!-- Google Drive -->
<div class="video-wrap iframe-wrap">
  <iframe src="https://drive.google.com/file/d/FILE_ID/preview"
    title="세션명" allow="autoplay" allowfullscreen></iframe>
</div>
```

```css
.iframe-wrap { position: relative; padding-top: 56.25%; }
.iframe-wrap iframe { position: absolute; inset: 0; width: 100%; height: 100%; border: 0; }
```

---

## 4. 이미지 처리

| 방식 | 사용 | 비고 |
|------|------|------|
| `src="C:/Users/.../photo.jpg"` | ❌ 금지 | 다른 컴퓨터에서 안 보임 |
| `src="file:///Users/.../photo.jpg"` | ❌ 금지 | 로컬 전용 |
| `src="data:image/jpeg;base64,..."` | ✅ 권장 | 파일에 embed, 완전 독립 |

**Claude Code에 사진 파일을 올리면 자동으로 base64로 변환해서 HTML에 embed합니다.**

압축 기준: 최대 1200px, JPEG 75% — Claude Code가 자동 처리

---

## 5. 동영상 처리

### 로컬 파일을 그대로 올리기 (권장)

Claude Code에 동영상 파일을 함께 제공하면:
1. `github_page/aws/세션폴더/videos/` 에 복사
2. HTML에 `<video src="./videos/파일명.mp4">` 상대경로 삽입
3. GitHub에 올리면 그대로 재생

### GitHub 파일 크기 제한

| 파일 크기 | 처리 |
|-----------|------|
| 100MB 미만 | ✅ 그냥 push 가능 |
| 100MB ~ 2GB | ⚠️ Git LFS 필요 |
| 2GB 초과 | ❌ 외부 embed 필요 |

100MB 초과 시 Claude Code가 자동으로 경고하고 ffmpeg 압축 명령어 안내:
```bash
ffmpeg -i input.mp4 -vcodec h264 -crf 28 -acodec aac output.mp4
```

---

## 6. Claude Code 프롬프트 템플릿

```
첨부한 SESSION_PRD.md와 SESSION_TEMPLATE.html을 참고해서
AWS Summit Seoul 2026 세션 리뷰 페이지를 만들어줘.

세션 정보:
- 세션명: [세션 제목]
- 발표자: [이름 · 소속]
- 트랙: [트랙명]
- 날짜: [Day 1 (5/20) / Day 2 (5/21)]
- 참석자: [daniel.007 / zero.002]

내 노트:
[여기에 메모 붙여넣기]

첨부 파일:
- 사진: [사진 파일들 업로드]
- 동영상: [동영상 파일 업로드 — 있는 경우]

원본 자료 폴더 (내 개인 작업 폴더):
- workfiles/aws-summit-2026/[본인이름]/

산출물 저장 위치:
- HTML: conference_review/github_page/aws/[세션폴더명]/index.html
- 동영상 있으면: 같은 폴더 안에 videos/ 서브폴더 생성
  (예: conference_review/github_page/aws/01-bedrock-rag/index.html)

요구사항:
1. 이미지는 base64로 HTML에 embed (로컬 경로 금지)
2. 동영상이 있으면:
   - github_page/aws/[세션폴더명]/videos/ 에 복사
   - <video src="./videos/파일명"> 상대경로 참조
   - 100MB 초과 시 경고 후 ffmpeg 압축 명령어 안내
3. GitHub Pages 상대경로만 사용 (../../index.html → main_index.html로 리다이렉트, ../index.html → AWS 허브)
4. nav 브레드크럼: ← 아카이브 / AWS Summit Seoul 2026 / 세션명
5. 완성 파일 크기가 25MB를 넘지 않도록 이미지 압축
6. 원본 자료 폴더(workfiles/aws-summit-2026/[본인이름]/)를 .gitignore에 추가
   (이미 있으면 스킵, github_page/.gitignore 파일에 추가)
7. GitHub에 올라가는 것은 index.html과 videos/ 파일만임을 확인
```

---

## 7. 개인 작업 폴더 관리 (Claude Code 자동 처리)

### 폴더 구조

각자 본인 이름으로 폴더를 만들어 원본 자료를 관리한다.

```
github_page/workfiles/aws-summit-2026/
  daniel/           ← daniel.007 개인 폴더
    사진/
    메모.txt
    현장녹음.m4a
  zero/             ← zero.002 개인 폴더
    사진/
    메모.txt
```

### gitignore 자동 추가 규칙

Claude Code가 프롬프트에서 **원본 자료 폴더**를 감지하면:

1. `github_page/.gitignore`를 열어 해당 경로가 있는지 확인
2. 없으면 자동 추가:
   ```
   # [본인이름] 개인 작업 폴더 — push 제외
   workfiles/aws-summit-2026/[본인이름]/
   ```
3. 있으면 스킵

### GitHub에 올라가는 파일 (최소 공유 단위)

```
github_page/aws/[세션폴더명]/
  index.html     ← 세션 리뷰 (이미지 base64 embed 완료)
  videos/        ← 동영상 파일 (있는 경우만)
```

원본 사진·메모·녹음 파일은 절대 push되지 않는다.

---

## 8. AWS 허브 페이지 업데이트 (수동)

세션 페이지 완성 후 `github_page/aws/index.html` 에서 해당 세션 카드를 수동으로 업데이트:

```html
<!-- WIP → 완성 상태로 변경 -->
<a href="./01-session-name/index.html" class="session-card done">
  <div class="card-top">
    <span class="track-badge">트랙명</span>
    <span class="status-badge done">완료</span>
  </div>
  <h3>세션 제목</h3>
  <p>한 줄 설명</p>
  <div class="card-footer">
    <span class="badge-attendee daniel">daniel.007</span>
    <span class="card-time">50분</span>
  </div>
</a>
```

허브 페이지에서 수정할 항목:
- `href`: `"#"` → `"./세션폴더명/index.html"` 으로 교체
- `status-badge`: `wip` → `done` 으로 변경
- 세션 수 카운트 업데이트

---

## 9. 완성 체크리스트

- [ ] `github_page/aws/세션폴더명/index.html` 파일 생성됨
- [ ] `index.html` 크기 25MB 미만
- [ ] 이미지 경로에 `file://` 또는 절대경로 없음 (전부 base64)
- [ ] nav 브레드크럼 경로 확인: `../../index.html`(→ main_index.html 리다이렉트), `../index.html`(→ AWS 허브)
- [ ] CSS 변수 (`--aws`, `--purple`, `--teal` 등) 정의됨
- [ ] 참석자 뱃지 색상 올바름 (daniel → purple / zero → teal)
- [ ] 동영상 있는 경우: `videos/` 폴더 생성 + 상대경로 참조
- [ ] 동영상 100MB 미만 확인
- [ ] `github_page/.gitignore`에 개인 작업 폴더 경로 추가됨
- [ ] `github_page/aws/index.html` 허브 카드 업데이트 완료
- [ ] git push: `cd conference_review && git push origin main`
- [ ] 모바일 뷰 확인 (반응형)
