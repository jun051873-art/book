<!DOCTYPE html>
<html lang="zh-TW">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>我的專屬魔法閱讀器</title>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/epub.js/0.3.88/epub.min.js"></script>
    <style>
        :root { --panel-bg: #fff9f0; --accent: #6c5ce7; }
        body { margin: 0; background: var(--panel-bg); font-family: "PingFang TC", sans-serif; height: 100vh; overflow: hidden; }
        
        /* 啟動書架 */
        #setup { position: fixed; inset: 0; display: flex; flex-direction: column; align-items: center; justify-content: center; z-index: 100; background: #fff9f0; }
        .upload-box { padding: 40px; border: 3px dashed var(--accent); border-radius: 20px; text-align: center; background: white; cursor: pointer; }

        /* 閱讀器容器 */
        #viewer { width: 100vw; height: 100vh; visibility: hidden; }

        /* 底部控制選單 (仿截圖風格) */
        #menu-panel {
            position: fixed; bottom: 0; width: 100%; background: var(--panel-bg);
            border-radius: 24px 24px 0 0; box-shadow: 0 -10px 30px rgba(0,0,0,0.15);
            padding: 25px; box-sizing: border-box; transform: translateY(100%); 
            transition: transform 0.4s ease; z-index: 1000;
        }
        #menu-panel.active { transform: translateY(0); }

        .menu-row { display: flex; align-items: center; justify-content: space-between; margin-bottom: 20px; font-size: 16px; color: #444; }
        .icon-bar { display: flex; justify-content: space-around; font-size: 22px; margin-bottom: 20px; border-bottom: 1px solid #ddd; padding-bottom: 10px; }
        
        .color-dot { width: 35px; height: 35px; border-radius: 50%; border: 2px solid white; cursor: pointer; box-shadow: 0 2px 5px rgba(0,0,0,0.1); }
        button.main-btn { background: var(--accent); color: white; border: none; padding: 10px 20px; border-radius: 12px; font-weight: bold; }
    </style>
</head>
<body>

<div id="setup">
    <div class="upload-box" onclick="document.getElementById('f').click()">
        <h1>✨ 魔法書架</h1>
        <p>點擊這裡導入你的 EPUB 電子書</p>
    </div>
</div>
<input type="file" id="f" accept=".epub" hidden>

<div id="viewer"></div>

<div id="menu-panel">
    <div class="icon-bar">
        <span>Aa</span> <span>版面</span> <span>🎨</span> <span>行為</span> <span>🐜</span>
    </div>

    <div class="menu-row">
        <span>主題配色</span>
        <div style="display: flex; gap: 10px;">
            <div class="color-dot" style="background:#ffffff;" onclick="setTheme('#ffffff', '#333')"></div>
            <div class="color-dot" style="background:#fdf6e3;" onclick="setTheme('#fdf6e3', '#5b4636')"></div>
            <div class="color-dot" style="background:#1a1a2e;" onclick="setTheme('#1a1a2e', '#e0e0e0')"></div>
        </div>
    </div>

    <div class="menu-row">
        <span>自然語音</span>
        <select id="vSelect" style="width: 50%; padding: 5px; border-radius: 5px;"></select>
        <button class="main-btn" onclick="runSpeech()" id="spkBtn">🔊 播放</button>
    </div>

    <div class="menu-row">
        <span>字體大小</span>
        <input type="range" min="16" max="32" value="20" oninput="setStyle('fontSize', this.value + 'px')">
    </div>

    <button onclick="toggleM()" style="width: 100%; padding: 12px; border-radius: 15px; border: none; background: #eee;">關閉設定</button>
</div>

<script>
    let b, r, s = window.speechSynthesis, speaking = false;

    document.getElementById('f').onchange = function(e) {
        const file = e.target.files[0];
        if (!file) return;
        
        const reader = new FileReader();
        reader.onload = function(evt) {
            document.getElementById('setup').style.display = 'none';
            document.getElementById('viewer').style.visibility = 'visible';
            
            b = ePub(evt.target.result);
            r = b.renderTo("viewer", { width: "100%", height: "100%", flow: "paginated" });
            r.display();

            r.on("click", (ev) => {
                const w = window.innerWidth;
                if (ev.clientX < w * 0.3) r.prev();
                else if (ev.clientX > w * 0.7) r.next();
                else toggleM();
            });
            loadVoices();
        };
        reader.readAsArrayBuffer(file);
    };

    function toggleM() { document.getElementById('menu-panel').classList.toggle('active'); }

    function setTheme(bg, fg) {
        if (!r) return;
        document.getElementById('viewer').style.background = bg;
        r.themes.default({ body: { background: `${bg} !important`, color: `${fg} !important` } });
    }

    function setStyle(prop, val) {
        let s = {}; s["p"] = {}; s["p"][prop] = `${val} !important`;
        r.themes.default(s);
    }

    function loadVoices() {
        const sel = document.getElementById('vSelect');
        sel.innerHTML = s.getVoices().filter(v => v.lang.includes('zh')).map(v => `<option value="${v.name}">${v.name.includes('Online') ? '✨ ' + v.name : v.name}</option>`).join('');
    }
    s.onvoiceschanged = loadVoices;

    function runSpeech() {
        if (speaking) { s.cancel(); speaking = false; document.getElementById('spkBtn').innerText = "🔊 播放"; return; }
        const ifr = document.querySelector('iframe');
        if (ifr) {
            const u = new SpeechSynthesisUtterance(ifr.contentDocument.body.innerText);
            u.voice = s.getVoices().find(v => v.name === document.getElementById('vSelect').value);
            u.onend = () => { speaking = false; document.getElementById('spkBtn').innerText = "🔊 播放"; };
            s.speak(u);
            speaking = true;
            document.getElementById('spkBtn').innerText = "🛑 停止";
        }
    }
</script>
</body>
</html>
