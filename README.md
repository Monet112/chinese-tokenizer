---
layout: default
title: 汉字分词 · Chinese Segmenter
<style>
  @import url('https://fonts.googleapis.com/css2?family=Noto+Serif+SC:wght@400;700&family=Noto+Sans+SC:wght@300;400&family=IM+Fell+English+SC&display=swap');

  *, *::before, *::after { box-sizing: border-box; margin: 0; padding: 0; }

  body {
    background: #f5f0e8;
    min-height: 100vh;
    font-family: 'Noto Sans SC', sans-serif;
    color: #1a1208;
    position: relative;
    overflow-x: hidden;
  }

  /* Ink-wash paper texture overlay */
  body::before {
    content: '';
    position: fixed;
    inset: 0;
    background-image:
      radial-gradient(ellipse at 20% 80%, rgba(180,140,80,0.08) 0%, transparent 60%),
      radial-gradient(ellipse at 80% 10%, rgba(120,80,40,0.06) 0%, transparent 50%);
    pointer-events: none;
    z-index: 0;
  }

  .page-wrapper {
    position: relative;
    z-index: 1;
    max-width: 820px;
    margin: 0 auto;
    padding: 3rem 2rem 5rem;
  }

  /* Vertical red accent bar */
  .accent-bar {
    width: 4px;
    height: 64px;
    background: #c0392b;
    margin-bottom: 1.5rem;
  }

  h1 {
    font-family: 'Noto Serif SC', serif;
    font-size: clamp(2rem, 5vw, 3.2rem);
    font-weight: 700;
    letter-spacing: -0.01em;
    line-height: 1.1;
    color: #1a1208;
  }

  h1 .sub {
    display: block;
    font-family: 'IM Fell English SC', serif;
    font-size: clamp(0.85rem, 2vw, 1rem);
    font-weight: 400;
    letter-spacing: 0.25em;
    color: #7a6040;
    margin-top: 0.4rem;
  }

  .divider {
    margin: 2rem 0;
    border: none;
    border-top: 1px solid #c8b89a;
    position: relative;
  }

  .divider::after {
    content: '◆';
    position: absolute;
    left: 50%;
    top: 50%;
    transform: translate(-50%, -50%);
    background: #f5f0e8;
    padding: 0 0.6rem;
    color: #c0392b;
    font-size: 0.6rem;
  }

  .status-bar {
    display: flex;
    align-items: center;
    gap: 0.6rem;
    margin-bottom: 1.8rem;
    font-size: 0.78rem;
    letter-spacing: 0.05em;
    color: #7a6040;
    font-family: 'IM Fell English SC', serif;
  }

  .status-dot {
    width: 8px; height: 8px;
    border-radius: 50%;
    background: #aaa;
    flex-shrink: 0;
    transition: background 0.4s;
  }
  .status-dot.loading { background: #e8a020; animation: pulse 1s infinite; }
  .status-dot.ready   { background: #27ae60; }
  .status-dot.error   { background: #c0392b; }

  @keyframes pulse {
    0%, 100% { opacity: 1; }
    50%       { opacity: 0.3; }
  }

  label {
    display: block;
    font-family: 'Noto Serif SC', serif;
    font-size: 0.8rem;
    letter-spacing: 0.15em;
    color: #7a6040;
    margin-bottom: 0.6rem;
    text-transform: uppercase;
  }

  textarea {
    width: 100%;
    min-height: 140px;
    padding: 1rem 1.1rem;
    background: #fffdf7;
    border: 1px solid #c8b89a;
    border-left: 3px solid #c0392b;
    border-radius: 2px;
    font-family: 'Noto Serif SC', serif;
    font-size: 1.1rem;
    line-height: 1.8;
    color: #1a1208;
    resize: vertical;
    transition: border-color 0.2s, box-shadow 0.2s;
    outline: none;
  }

  textarea:focus {
    border-color: #8b1a12;
    box-shadow: 0 0 0 3px rgba(192,57,43,0.08);
  }

  textarea::placeholder { color: #b8a888; }

  .btn-row {
    display: flex;
    align-items: center;
    gap: 1rem;
    margin-top: 1.1rem;
  }

  button {
    padding: 0.65rem 2.2rem;
    background: #1a1208;
    color: #f5f0e8;
    border: none;
    border-radius: 2px;
    font-family: 'IM Fell English SC', serif;
    font-size: 0.9rem;
    letter-spacing: 0.12em;
    cursor: pointer;
    transition: background 0.2s, transform 0.1s;
  }

  button:hover:not(:disabled) { background: #c0392b; }
  button:active:not(:disabled) { transform: scale(0.98); }
  button:disabled { opacity: 0.45; cursor: not-allowed; }

  .clear-btn {
    background: transparent;
    color: #7a6040;
    border: 1px solid #c8b89a;
  }
  .clear-btn:hover:not(:disabled) { background: #ede7d9; color: #1a1208; }

  /* Output */
  .output-section {
    margin-top: 2.5rem;
    opacity: 0;
    transform: translateY(8px);
    transition: opacity 0.4s, transform 0.4s;
  }
  .output-section.visible { opacity: 1; transform: none; }

  .segments-container {
    display: flex;
    flex-wrap: wrap;
    gap: 0.5rem 0.4rem;
    padding: 1.2rem 1.1rem;
    background: #fffdf7;
    border: 1px solid #c8b89a;
    min-height: 60px;
  }

  .seg {
    display: inline-flex;
    align-items: center;
    padding: 0.2rem 0.55rem;
    border: 1px solid #d4c4a0;
    border-radius: 1px;
    font-family: 'Noto Serif SC', serif;
    font-size: 1rem;
    background: #f5f0e8;
    position: relative;
    animation: fadeIn 0.3s ease both;
  }

  @keyframes fadeIn {
    from { opacity: 0; transform: scale(0.9); }
    to   { opacity: 1; transform: scale(1); }
  }

  /* colour-code by character count (visual rhythm) */
  .seg[data-len="1"] { border-color: #c0392b33; }
  .seg[data-len="2"] { border-color: #1a120833; }
  .seg[data-len="3"] { border-color: #e8a02055; }

  .raw-output {
    margin-top: 1rem;
    padding: 0.9rem 1.1rem;
    background: #1a1208;
    color: #c8d8c0;
    font-family: 'Courier New', monospace;
    font-size: 0.82rem;
    line-height: 1.7;
    word-break: break-all;
    border-radius: 2px;
  }

  .raw-label {
    font-family: 'IM Fell English SC', serif;
    font-size: 0.72rem;
    letter-spacing: 0.15em;
    color: #7a6040;
    margin-top: 1.4rem;
    margin-bottom: 0.4rem;
  }

  .count-note {
    margin-top: 0.7rem;
    font-size: 0.75rem;
    color: #7a6040;
    letter-spacing: 0.05em;
  }

  .error-msg {
    color: #c0392b;
    font-size: 0.82rem;
    margin-top: 0.6rem;
    display: none;
  }
<div class="page-wrapper">
<div class="accent-bar"></div>
<h1>
    汉字分词
    <span class="sub">Chinese Word Segmentation</span>
  </h1>
<hr class="divider">
<div class="status-bar">
    <div class="status-dot loading" id="statusDot"></div>
    <span id="statusText">Loading Python runtime…</span>
  </div>
<label for="inputText">输入中文文本 · Enter Chinese Text</label>
<textarea
    id="inputText"
    placeholder="例如：我来到北京清华大学"
    disabled
<div class="error-msg" id="errorMsg"></div>
<div class="btn-row">
    <button id="segBtn" disabled>Segment 分词</button>
    <button class="clear-btn" id="clearBtn">Clear 清除</button>
  </div>
<div class="output-section" id="outputSection">
    <label>分词结果 · Segmented Tokens</label>
    <div class="segments-container" id="segmentsContainer"></div>
    <p class="raw-label">Raw output · 原始输出</p>
    <div class="raw-output" id="rawOutput"></div>
    <p class="count-note" id="countNote"></p>
</div>
</div>
<!-- Pyodide -->
<script src="https://cdn.jsdelivr.net/pyodide/v0.27.4/full/pyodide.js"></script>
<script>
(async () => {
  const dot        = document.getElementById('statusDot');
  const statusText = document.getElementById('statusText');
  const inputText  = document.getElementById('inputText');
  const segBtn     = document.getElementById('segBtn');
  const clearBtn   = document.getElementById('clearBtn');
  const outputSec  = document.getElementById('outputSection');
  const segCont    = document.getElementById('segmentsContainer');
  const rawOut     = document.getElementById('rawOutput');
  const countNote  = document.getElementById('countNote');
  const errorMsg   = document.getElementById('errorMsg');

  let pyodide = null;

  // ── Boot Pyodide + install jieba ──────────────────────────────────────────
  try {
    pyodide = await loadPyodide();
    statusText.textContent = 'Installing jieba…';
    await pyodide.loadPackage('micropip');
    await pyodide.runPythonAsync(`
import micropip
await micropip.install('jieba')
import jieba
# Warm up (suppresses first-run initialisation logs)
list(jieba.cut('测试'))
    `);
    dot.className       = 'status-dot ready';
    statusText.textContent = 'Ready — jieba loaded';
    inputText.disabled  = false;
    segBtn.disabled     = false;
  } catch (err) {
    dot.className       = 'status-dot error';
    statusText.textContent = 'Failed to load runtime';
    console.error(err);
  }

  // ── Segment ───────────────────────────────────────────────────────────────
  segBtn.addEventListener('click', async () => {
    const text = inputText.value.trim();
    errorMsg.style.display = 'none';

    if (!text) {
      errorMsg.textContent   = '请输入中文文本 · Please enter some Chinese text.';
      errorMsg.style.display = 'block';
      return;
    }

    segBtn.disabled     = true;
    segBtn.textContent  = '分词中…';
    outputSec.classList.remove('visible');

    try {
      // Pass the text safely via pyodide globals
      pyodide.globals.set('_input_text', text);
      const result = await pyodide.runPythonAsync(`
import jieba, json
segments = list(jieba.cut(_input_text, cut_all=False))
json.dumps(segments, ensure_ascii=False)
      `);

      const segments = JSON.parse(result);

      // Render token chips
      segCont.innerHTML = segments
        .map((s, i) => {
          const len = [...s].length; // unicode-aware length
          return `<span class="seg" data-len="${Math.min(len, 3)}" style="animation-delay:${i * 30}ms">${s}</span>`;
        })
        .join('');

      // Raw slash-delimited output
      rawOut.textContent = segments.join(' / ');

      countNote.textContent =
        `${segments.length} tokens from ${[...text].length} characters`;

      outputSec.classList.add('visible');
    } catch (err) {
      errorMsg.textContent   = '分词时出错 · Error during segmentation: ' + err.message;
      errorMsg.style.display = 'block';
    }

    segBtn.disabled    = false;
    segBtn.textContent = 'Segment 分词';
  });

  // ── Clear ─────────────────────────────────────────────────────────────────
  clearBtn.addEventListener('click', () => {
    inputText.value = '';
    outputSec.classList.remove('visible');
    errorMsg.style.display = 'none';
    inputText.focus();
  });

  // Allow Ctrl/Cmd + Enter to segment
  inputText.addEventListener('keydown', e => {
    if ((e.ctrlKey || e.metaKey) && e.key === 'Enter') segBtn.click();
  });
})();
