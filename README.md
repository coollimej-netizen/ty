<!DOCTYPE html>
<html lang="ko">
<head>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1" />
  <title>동영상 보기 (3개)</title>
  <style>
    :root { --bg:#0b1220; --card:#111a2b; --muted:#8aa0bf; --text:#e6eefc; --accent:#6aa5ff; }
    * { box-sizing: border-box; }
    body { margin: 0; font-family: system-ui, -apple-system, "Segoe UI", Roboto, "Apple SD Gothic Neo", "Noto Sans KR", sans-serif; background: linear-gradient(180deg, #0b1220 0%, #0f1a30 100%); color: var(--text); }
    header { padding: 18px 20px; border-bottom: 1px solid rgba(255,255,255,0.06); position: sticky; top: 0; backdrop-filter: blur(6px); background: rgba(11,18,32,0.6); }
    header h1 { margin: 0; font-size: 18px; font-weight: 700; letter-spacing: 0.2px; }
    .wrap { display: grid; grid-template-columns: 280px 1fr; gap: 16px; padding: 16px; max-width: 1200px; margin: 0 auto; }
    .panel { background: var(--card); border: 1px solid rgba(255,255,255,0.07); border-radius: 16px; box-shadow: 0 10px 30px rgba(0,0,0,0.25); }
    .list { padding: 12px; }
    .list h2 { font-size: 14px; color: var(--muted); font-weight: 600; margin: 4px 6px 8px; }
    .video-btn { width: 100%; padding: 10px 12px; border: 1px solid rgba(255,255,255,0.08); background: #0f1726; color: var(--text); border-radius: 12px; text-align: left; cursor: pointer; transition: transform .06s ease, border-color .2s ease, background .2s ease; display: flex; align-items: center; gap: 10px; }
    .video-btn + .video-btn { margin-top: 8px; }
    .video-btn:hover { transform: translateY(-1px); border-color: rgba(255,255,255,0.18); }
    .video-btn.active { background: #14203a; border-color: var(--accent); }
    .dot { width: 8px; height: 8px; border-radius: 999px; background: var(--accent); box-shadow: 0 0 18px rgba(106,165,255,0.8); }
    .title { font-size: 14px; font-weight: 700; letter-spacing: .2px; }
    .muted { font-size: 12px; color: var(--muted); }

    .player { padding: 14px; display: grid; grid-template-rows: auto 1fr; gap: 12px; }
    .player-head { display: flex; align-items: center; justify-content: space-between; padding: 8px 10px; border: 1px solid rgba(255,255,255,0.07); border-radius: 12px; background: #0f1726; }
    .now-title { font-weight: 800; font-size: 16px; letter-spacing: .3px; }
    .src-hint { font-size: 12px; color: var(--muted); }

    video { width: 100%; max-height: 70vh; border-radius: 16px; border: 1px solid rgba(255,255,255,0.08); background: #090f1a; }

    .help { margin-top: 10px; font-size: 12px; color: var(--muted); }
    code { background: #0e1627; padding: 2px 6px; border-radius: 6px; border: 1px solid rgba(255,255,255,0.06); }

    @media (max-width: 860px) {
      .wrap { grid-template-columns: 1fr; }
    }
  </style>
</head>
<body>
  <header>
    <h1>동영상 보기 · 3개</h1>
  </header>

  <div class="wrap">
    <!-- 왼쪽: 동영상 목록 -->
    <aside class="panel list">
      <h2>동영상 선택</h2>
      <div id="videoList"></div>
      <p class="help">💡 아래 <code>videos</code> 배열의 <code>title</code>과 <code>src</code>를 코드에서 원하는 이름/경로로 바꿔주세요. (mp4/webm 권장)</p>
    </aside>

    <!-- 오른쪽: 플레이어 -->
    <main class="panel player">
      <div class="player-head">
        <div class="now-title" id="nowTitle">동영상을 선택하세요</div>
        <div class="src-hint" id="srcHint"></div>
      </div>
      <video id="viewer" controls preload="metadata" poster="" playsinline></video>
    </main>
  </div>

  <script>
    // ▼▼▼ 여기만 수정해서 사용하세요 ▼▼▼
    // 동영상 3개: title(이름)과 src(파일/URL)
    const videos = [
      { title: "콜라", title: "20250815_220249.mp4" },
      { title: "자동차", title: "20250816_122836.mp4" },
      { title: "이빨", title: "Generated File August 16, 2025 - 3_32PM.mp4" },
    ];
    // ▲▲▲ 여기만 수정해서 사용하세요 ▲▲▲

    const listEl = document.getElementById('videoList');
    const viewer = document.getElementById('viewer');
    const nowTitle = document.getElementById('nowTitle');
    const srcHint = document.getElementById('srcHint');

    let currentIndex = -1;

    // 목록 렌더링
    function renderList() {
      listEl.innerHTML = '';
      videos.forEach((v, i) => {
        const btn = document.createElement('button');
        btn.className = 'video-btn' + (i === currentIndex ? ' active' : '');
        btn.innerHTML = `
          <span class="dot" aria-hidden="true"></span>
          <span class="title">${escapeHTML(v.title || '제목 없음')}</span>
        `;
        btn.addEventListener('click', () => selectVideo(i));
        listEl.appendChild(btn);
      });
    }

    // 동영상 선택
    function selectVideo(index) {
      const v = videos[index];
      if (!v) return;
      currentIndex = index;

      nowTitle.textContent = v.title || '제목 없음';
      srcHint.textContent = v.src || '';

      // 비디오 소스 갱신
      viewer.pause();
      viewer.removeAttribute('src');
      viewer.innerHTML = '';

      const source = document.createElement('source');
      source.src = v.src;
      source.type = guessTypeFromSrc(v.src);
      viewer.appendChild(source);

      viewer.load();
      viewer.play().catch(() => {/* 자동재생 실패 시 무시 */});

      // 버튼 active 상태 반영
      Array.from(listEl.children).forEach((el, i) => {
        el.classList.toggle('active', i === currentIndex);
      });
    }

    // 파일 확장자 기반 타입 추정
    function guessTypeFromSrc(src) {
      const lower = (src || '').toLowerCase();
      if (lower.endsWith('.mp4')) return 'video/mp4';
      if (lower.endsWith('.webm')) return 'video/webm';
      if (lower.endsWith('.ogg') || lower.endsWith('.ogv')) return 'video/ogg';
      return 'video/mp4';
    }

    // XSS 방지를 위한 간단한 escape
    function escapeHTML(str) {
      return String(str)
        .replaceAll('&', '&amp;')
        .replaceAll('<', '&lt;')
        .replaceAll('>', '&gt;')
        .replaceAll('"', '&quot;')
        .replaceAll("'", '&#039;');
    }

    // 초기 렌더
    renderList();
    // 첫 번째 자동 선택 (원치 않으면 주석 처리)
    if (videos.length) selectVideo(0);
  </script>
</body>
</html>
<!DOCTYPE html>
<html lang="ko">
<head>
    <meta charset="UTF-8">
    <title>메뉴 버튼 예제</title>
    <style>
        body {
            font-family: Arial, sans-serif;
            text-align: center;
            margin-top: 50px;
        }
        .menu {
            display: flex;
            justify-content: center;
            gap: 20px;
        }
        .menu button {
            padding: 10px 20px;
            font-size: 16px;
            border: none;
            background-color: #4CAF50;
            color: white;
            cursor: pointer;
            border-radius: 5px;
            transition: background-color 0.3s;
        }
        .menu button:hover {
            background-color: #45a049;
        }
    </style>
</head>
<body>
    <h1>메뉴</h1>
    <div class="menu">
        <button onclick="location.href='https://coollimej-netizen.github.io/my-cat/'">애용공식홈</button>
    </div>
</body>
</html>

