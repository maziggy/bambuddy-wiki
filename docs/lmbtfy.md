---
title: Let me Bambuddy that for you
hide:
  - navigation
  - toc
  - footer
search:
  exclude: true
---

<style>
  .lmbtfy-stage {
    position: relative;
    margin: 2rem auto;
    max-width: 720px;
    padding: 2rem 1.5rem;
    border-radius: 0.5rem;
    background: var(--md-default-bg-color);
    border: 1px solid var(--md-default-fg-color--lightest);
    box-shadow: 0 4px 16px rgba(0,0,0,.08);
    font-family: var(--md-text-font-family, Roboto, sans-serif);
  }

  .lmbtfy-instructions {
    text-align: center;
    font-size: 0.95rem;
    color: var(--md-default-fg-color--light);
    margin-bottom: 1.25rem;
    min-height: 1.4em;
  }

  .lmbtfy-instructions strong {
    color: var(--md-primary-fg-color);
  }

  .lmbtfy-searchbar {
    position: relative;
    display: flex;
    align-items: center;
    height: 2.6rem;
    padding: 0 0.9rem;
    border-radius: 1.3rem;
    background: var(--md-default-fg-color--lightest);
    border: 1px solid transparent;
    transition: background .2s, border-color .2s, box-shadow .2s;
  }

  .lmbtfy-searchbar.is-focused {
    background: var(--md-default-bg-color);
    border-color: var(--md-primary-fg-color);
    box-shadow: 0 0 0 3px rgba(76, 175, 80, .15);
  }

  .lmbtfy-searchicon {
    width: 1.1rem;
    height: 1.1rem;
    margin-right: 0.6rem;
    flex-shrink: 0;
    opacity: 0.55;
  }

  .lmbtfy-query {
    flex: 1;
    font-size: 0.95rem;
    color: var(--md-default-fg-color);
    white-space: pre;
    overflow: hidden;
  }

  .lmbtfy-caret {
    display: inline-block;
    width: 1px;
    height: 1.05em;
    margin-left: 1px;
    background: var(--md-default-fg-color);
    vertical-align: text-bottom;
    animation: lmbtfy-blink 1s steps(2) infinite;
    opacity: 0;
  }

  .lmbtfy-searchbar.is-focused .lmbtfy-caret {
    opacity: 1;
  }

  @keyframes lmbtfy-blink {
    50% { opacity: 0; }
  }

  .lmbtfy-cursor {
    position: absolute;
    top: 0;
    left: 0;
    width: 22px;
    height: 22px;
    pointer-events: none;
    transition: transform 1.1s cubic-bezier(.22,.61,.36,1);
    transform: translate(40px, 220px);
    z-index: 5;
    filter: drop-shadow(0 1px 1px rgba(0,0,0,.25));
  }

  .lmbtfy-cursor.is-clicking {
    transform: translate(var(--target-x, 40px), var(--target-y, 60px)) scale(0.82);
  }

  .lmbtfy-cursor.is-resting {
    transform: translate(var(--target-x, 40px), var(--target-y, 60px));
  }

  .lmbtfy-ripple {
    position: absolute;
    top: 0;
    left: 0;
    width: 12px;
    height: 12px;
    border-radius: 50%;
    border: 2px solid var(--md-primary-fg-color);
    pointer-events: none;
    opacity: 0;
    transform: translate(-50%, -50%) scale(1);
  }

  .lmbtfy-ripple.is-pulsing {
    animation: lmbtfy-ripple .6s ease-out;
  }

  @keyframes lmbtfy-ripple {
    0%   { opacity: .9; transform: translate(-50%, -50%) scale(0.6); }
    100% { opacity: 0;  transform: translate(-50%, -50%) scale(3); }
  }

  .lmbtfy-finale {
    margin-top: 1.75rem;
    text-align: center;
    opacity: 0;
    transition: opacity .4s;
  }

  .lmbtfy-finale.is-visible {
    opacity: 1;
  }

  .lmbtfy-finale p {
    margin: 0 0 0.85rem;
    color: var(--md-default-fg-color--light);
    font-style: italic;
  }

  .lmbtfy-finale a.lmbtfy-button {
    display: inline-block;
    padding: 0.55rem 1.25rem;
    border-radius: 0.4rem;
    background: var(--md-primary-fg-color);
    color: var(--md-primary-bg-color);
    font-weight: 500;
    text-decoration: none;
    transition: background .2s;
  }

  .lmbtfy-finale a.lmbtfy-button:hover {
    background: var(--md-primary-fg-color--dark);
  }

  .lmbtfy-share {
    margin-top: 2rem;
    padding: 1rem;
    border-radius: 0.4rem;
    background: var(--md-default-fg-color--lightest);
    font-size: 0.85rem;
  }

  .lmbtfy-share code {
    word-break: break-all;
    background: transparent;
    padding: 0;
  }

  .lmbtfy-share-input {
    display: flex;
    gap: 0.5rem;
    margin-top: 0.5rem;
  }

  .lmbtfy-share-input input {
    flex: 1;
    padding: 0.5rem 0.7rem;
    border: 1px solid var(--md-default-fg-color--lightest);
    border-radius: 0.3rem;
    background: var(--md-default-bg-color);
    color: var(--md-default-fg-color);
    font-family: var(--md-code-font-family, monospace);
    font-size: 0.85rem;
  }

  .lmbtfy-share-input button {
    padding: 0.5rem 1rem;
    border: 0;
    border-radius: 0.3rem;
    background: var(--md-primary-fg-color);
    color: var(--md-primary-bg-color);
    cursor: pointer;
    font-size: 0.85rem;
  }

  .lmbtfy-empty {
    text-align: center;
    padding: 2rem 1rem;
  }

  .lmbtfy-empty h2 {
    margin-top: 0;
  }
</style>

<div id="lmbtfy-root"></div>

<script>
(function () {
  'use strict';

  var params = new URLSearchParams(window.location.search);
  // Use `lmbtfy=` (not `q=`) so the intermediate page doesn't trigger Material's
  // search.share handler, which hijacks any URL containing `?q=`.
  var rawQuery = (params.get('lmbtfy') || '').trim();
  var root = document.getElementById('lmbtfy-root');
  if (!root) return;

  if (!rawQuery) {
    root.innerHTML = [
      '<div class="lmbtfy-stage lmbtfy-empty">',
      '  <h2>Let me Bambuddy that for you</h2>',
      '  <p>Generate a polite-but-pointed link that walks someone through searching the wiki.</p>',
      '  <div class="lmbtfy-share-input">',
      '    <input id="lmbtfy-build-query" type="text" placeholder="what they should have searched for">',
      '    <button id="lmbtfy-build-button" type="button">Build link</button>',
      '  </div>',
      '  <p id="lmbtfy-build-output" style="margin-top:1rem; word-break:break-all;"></p>',
      '</div>'
    ].join('\n');

    var input = document.getElementById('lmbtfy-build-query');
    var btn = document.getElementById('lmbtfy-build-button');
    var out = document.getElementById('lmbtfy-build-output');
    function build() {
      var q = (input.value || '').trim();
      if (!q) { out.textContent = ''; return; }
      var url = window.location.origin + window.location.pathname + '?lmbtfy=' + encodeURIComponent(q);
      out.innerHTML = '<a href="' + url + '">' + url + '</a>';
    }
    btn.addEventListener('click', build);
    input.addEventListener('keydown', function (e) { if (e.key === 'Enter') build(); });
    return;
  }

  // Display query — collapse plus-signs to spaces for friendlier reading.
  var displayQuery = rawQuery.replace(/\+/g, ' ');
  var searchUrl = '/?q=' + encodeURIComponent(displayQuery);

  root.innerHTML = [
    '<div class="lmbtfy-stage" id="lmbtfy-stage">',
    '  <div class="lmbtfy-instructions" id="lmbtfy-instructions">Step 1: open the wiki search.</div>',
    '  <div class="lmbtfy-searchbar" id="lmbtfy-searchbar">',
    '    <svg class="lmbtfy-searchicon" viewBox="0 0 24 24" fill="currentColor"><path d="M15.5 14h-.79l-.28-.27A6.471 6.471 0 0 0 16 9.5 6.5 6.5 0 1 0 9.5 16c1.61 0 3.09-.59 4.23-1.57l.27.28v.79l5 4.99L20.49 19l-4.99-5zm-6 0C7.01 14 5 11.99 5 9.5S7.01 5 9.5 5 14 7.01 14 9.5 11.99 14 9.5 14z"/></svg>',
    '    <span class="lmbtfy-query" id="lmbtfy-query"></span><span class="lmbtfy-caret" id="lmbtfy-caret"></span>',
    '  </div>',
    '  <svg class="lmbtfy-cursor" id="lmbtfy-cursor" viewBox="0 0 24 24">',
    '    <path d="M5.5 3.21V20.79c0 .45.54.67.85.35l4.86-4.86 2.39 5.34c.11.25.4.36.65.25l1.87-.83c.25-.11.36-.4.25-.65l-2.39-5.34h6.87c.45 0 .67-.54.35-.85L6.35 2.85a.5.5 0 0 0-.85.36z" fill="#222"/>',
    '    <path d="M6.5 5.04V18.4l3.78-3.78 3.34 7.45.95-.42-3.4-7.6h5.34L6.5 5.04z" fill="#fff"/>',
    '  </svg>',
    '  <span class="lmbtfy-ripple" id="lmbtfy-ripple"></span>',
    '  <div class="lmbtfy-finale" id="lmbtfy-finale">',
    '    <p>Was that so hard?</p>',
    '    <a href="' + searchUrl + '" class="lmbtfy-button" id="lmbtfy-go">Take me to the search results &rarr;</a>',
    '  </div>',
    '</div>',
    '<div class="lmbtfy-share">',
    '  Next time, share this with them so they can learn:',
    '  <div class="lmbtfy-share-input">',
    '    <input id="lmbtfy-share-url" type="text" readonly>',
    '    <button id="lmbtfy-share-copy" type="button">Copy</button>',
    '  </div>',
    '</div>'
  ].join('\n');

  var stage = document.getElementById('lmbtfy-stage');
  var instructions = document.getElementById('lmbtfy-instructions');
  var searchbar = document.getElementById('lmbtfy-searchbar');
  var queryEl = document.getElementById('lmbtfy-query');
  var cursor = document.getElementById('lmbtfy-cursor');
  var ripple = document.getElementById('lmbtfy-ripple');
  var finale = document.getElementById('lmbtfy-finale');
  var goButton = document.getElementById('lmbtfy-go');
  var shareInput = document.getElementById('lmbtfy-share-url');
  var shareCopy = document.getElementById('lmbtfy-share-copy');

  shareInput.value = window.location.href;
  shareCopy.addEventListener('click', function () {
    shareInput.select();
    try {
      navigator.clipboard.writeText(shareInput.value);
      shareCopy.textContent = 'Copied!';
      setTimeout(function () { shareCopy.textContent = 'Copy'; }, 1500);
    } catch (e) {
      document.execCommand('copy');
    }
  });

  function wait(ms) {
    return new Promise(function (resolve) { setTimeout(resolve, ms); });
  }

  function moveCursorTo(el) {
    var stageRect = stage.getBoundingClientRect();
    var targetRect = el.getBoundingClientRect();
    var x = (targetRect.left - stageRect.left) + 18;
    var y = (targetRect.top - stageRect.top) + (targetRect.height / 2) - 11;
    cursor.style.setProperty('--target-x', x + 'px');
    cursor.style.setProperty('--target-y', y + 'px');
    cursor.classList.add('is-resting');
  }

  function clickPulse() {
    var cursorRect = cursor.getBoundingClientRect();
    var stageRect = stage.getBoundingClientRect();
    ripple.style.left = (cursorRect.left - stageRect.left + 6) + 'px';
    ripple.style.top = (cursorRect.top - stageRect.top + 6) + 'px';
    ripple.classList.remove('is-pulsing');
    void ripple.offsetWidth;
    ripple.classList.add('is-pulsing');
    cursor.classList.add('is-clicking');
    setTimeout(function () { cursor.classList.remove('is-clicking'); }, 180);
  }

  async function typeQuery(text) {
    for (var i = 0; i < text.length; i++) {
      queryEl.textContent += text[i];
      var delay = 70 + Math.random() * 60;
      if (text[i] === ' ') delay += 50;
      await wait(delay);
    }
  }

  async function run() {
    await wait(450);
    moveCursorTo(searchbar);
    await wait(1150);
    clickPulse();
    searchbar.classList.add('is-focused');
    await wait(420);
    instructions.innerHTML = 'Step 2: type <strong>' + displayQuery.replace(/[&<>"']/g, function (c) {
      return { '&': '&amp;', '<': '&lt;', '>': '&gt;', '"': '&quot;', "'": '&#39;' }[c];
    }) + '</strong>.';
    await wait(350);
    await typeQuery(displayQuery);
    await wait(550);
    instructions.innerHTML = 'Step 3: hit <strong>Enter</strong>.';
    await wait(800);
    finale.classList.add('is-visible');
    await wait(2500);
    window.location.href = searchUrl;
  }

  // Slight delay so MkDocs's instant-load transition settles first.
  setTimeout(run, 150);
})();
</script>
