# Tungstun
Official TUNGSTUN terminal teaser – explore the hidden archives of OVH and uncover the lore behind ALB:01, an upcoming album shrouded in mystery, rebellion, and coded resonance.

<!doctype html>
<html lang="en">
<head>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width,initial-scale=1" />
  <title>Tungstun — OVH Archive</title>
  <link rel="preconnect" href="https://fonts.googleapis.com">
  <link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
  <link href="https://fonts.googleapis.com/css2?family=VT323&family=Share+Tech+Mono&display=swap" rel="stylesheet">
  <style>
    :root{
      --bg:#000;
      --red:#ff1f1f;
      --glow:0 0 14px rgba(255,31,31,0.22),0 0 34px rgba(255,10,10,0.08);
      --mono: 'VT323', 'Share Tech Mono', 'Courier New', monospace;
    }
    *{box-sizing:border-box;margin:0;padding:0}
    html,body{height:100%;background:var(--bg);color:var(--red);font-family:var(--mono);overflow:hidden;text-shadow:var(--glow)}

    /* CRT overlays */
    .crt::before{content:'';position:fixed;inset:0;background:repeating-linear-gradient(180deg,rgba(255,0,0,0.03)0px,rgba(0,0,0,0)2px);mix-blend-mode:overlay;opacity:.5;pointer-events:none;animation:scan 6s linear infinite}
    .grain{position:fixed;inset:0;background-image:radial-gradient(rgba(255,0,0,0.06)0.5px,transparent 1px);background-size:3px 3px;opacity:.08;mix-blend-mode:overlay;pointer-events:none;animation:grainAnim .5s steps(2) infinite}
    @keyframes scan{0%{transform:translateY(-100%)}100%{transform:translateY(100%)}}@keyframes grainAnim{0%,100%{opacity:.08}50%{opacity:.14}}

    .flicker{animation:flickerAnim 7s infinite}
    @keyframes flickerAnim{0%{opacity:1}2%{opacity:0.85}6%{opacity:1}8%{opacity:0.8}12%{opacity:1}100%{opacity:1}}

    /* Boot */
    #boot{display:flex;align-items:center;justify-content:center;height:100vh;flex-direction:column;gap:12px;font-size:18px;letter-spacing:2px;position:relative;z-index:10}
    #bootProgress{font-family:var(--mono);font-size:16px}
    #matrixBg{position:fixed;inset:0;overflow:hidden;z-index:1}
    #matrixCanvas{width:100%;height:100%;display:block}


    /* Terminal */
    #terminal{display:none;flex-direction:column;height:100vh;padding:28px;gap:10px;position:relative}
    .logo{font-size:18px}
    .logo .mark{font-size:34px;margin-bottom:6px}
    .screen{flex:1;overflow:auto}
    pre{white-space:pre-wrap;word-break:break-word;font-size:15px;line-height:1.45}
    .prompt{display:flex;align-items:flex-start;gap:10px}
    .label{color:rgba(255,31,31,0.65);min-width:110px}
    .cmdline{flex:1;outline:none;border:none;background:transparent;color:var(--red);font-family:var(--mono);caret-color:var(--red);min-height:20px}
    .cursor{display:inline-block;width:8px;height:18px;background:var(--red);margin-left:6px;animation:blink 1s steps(1) infinite}
    @keyframes blink{50%{opacity:0}}

    /* error overlay */
    .error-screen{position:fixed;inset:0;display:flex;align-items:center;justify-content:center;flex-direction:column;text-align:center;backdrop-filter:blur(2px);pointer-events:auto}
    .error-box{max-width:820px;padding:28px;border-radius:8px;text-align:center}
    .links a{color:var(--red);text-decoration:none;display:inline-block;margin:6px 12px;padding:8px 12px;border-radius:4px;border:1px solid rgba(255,31,31,0.06);box-shadow:0 0 14px rgba(255,31,31,0.06)}

    @media (max-width:680px){.logo .mark{font-size:26px}.label{min-width:90px}}
  </style>
</head>
<body class="crt">
  <div id="matrixBg">
    <canvas id="matrixCanvas"></canvas>
  </div>
  
  <div id="boot">
    <div>INITIALIZING OVH ARCHIVE...</div>
    <div id="bootProgress">[..........]</div>
    <div style="font-size:12px;opacity:0.7">Press any key to enable audio</div>
  </div>

  <div class="grain"></div>

  <div id="terminal" class="flicker">
    <div class="logo"><span class="mark">(\/v)  TUNGSTUN — OVH</span><div style="font-size:12px;color:rgba(255,31,31,0.6)">Order of Vanished Horizons — Archive Terminal</div></div>
    <div class="screen" id="screen"><pre id="output"></pre></div>
    <div class="prompt"><div class="label">OVH@archives</div><div contenteditable="true" id="input" class="cmdline" spellcheck="false" aria-label="terminal input"></div><div class="cursor" id="cursor"></div></div>
  </div>

  <div id="errorScreen" class="error-screen" style="display:none">
    <div class="error-box" id="errorBox"></div>
  </div>

  <script>
    // ---------- Matrix Rain Effect ----------
    const matrixCanvas = document.getElementById('matrixCanvas');
    const matrixCtx = matrixCanvas.getContext('2d');
    matrixCanvas.width = window.innerWidth;
    matrixCanvas.height = window.innerHeight;

    const chars = 'TUNGSTUN01#$%^&*()_-+=[]{}|;:,.<>?/~`';
    const targetWord = 'TUNGSTUN';
    const fontSize = 14;
    const columns = Math.floor(matrixCanvas.width / fontSize);
    const drops = Array(columns).fill(1);
    
    // Calculate center position for TUNGSTUN
    const wordFontSize = 48;
    const wordX = (matrixCanvas.width - targetWord.length * wordFontSize * 0.6) / 2;
    const wordY = matrixCanvas.height / 2;
    
    // Track which letters have been revealed
    const revealedLetters = Array(targetWord.length).fill(false);
    let startFormingTime = 0;
    let matrixStartTime = Date.now();

    function drawMatrix() {
      matrixCtx.fillStyle = 'rgba(0, 0, 0, 0.05)';
      matrixCtx.fillRect(0, 0, matrixCanvas.width, matrixCanvas.height);
      
      const elapsed = Date.now() - matrixStartTime;
      const shouldStartForming = elapsed > 7000; // Start forming at 7 seconds
      
      if (shouldStartForming && startFormingTime === 0) {
        startFormingTime = Date.now();
      }
      
      // Draw falling matrix
      matrixCtx.fillStyle = '#ff1f1f';
      matrixCtx.font = fontSize + 'px monospace';
      
      for (let i = 0; i < drops.length; i++) {
        const text = chars[Math.floor(Math.random() * chars.length)];
        matrixCtx.fillText(text, i * fontSize, drops[i] * fontSize);
        
        if (drops[i] * fontSize > matrixCanvas.height && Math.random() > 0.975) {
          drops[i] = 0;
        }
        drops[i]++;
      }
      
      // Draw forming TUNGSTUN
      if (shouldStartForming) {
        const formingElapsed = Date.now() - startFormingTime;
        matrixCtx.font = `bold ${wordFontSize}px monospace`;
        
        for (let i = 0; i < targetWord.length; i++) {
          const letterX = wordX + i * wordFontSize * 0.6;
          const revealDelay = i * 200; // Each letter reveals 200ms after previous
          
          if (formingElapsed > revealDelay) {
            if (!revealedLetters[i]) {
              revealedLetters[i] = true;
              if(audioCtx) playTone(800 + i * 50, 0.08, 'sine', 0.06);
            }
            
            // Glitch effect before solidifying
            if (formingElapsed < revealDelay + 400) {
              const glitchChar = chars[Math.floor(Math.random() * chars.length)];
              matrixCtx.fillStyle = `rgba(255, 31, 31, ${0.3 + Math.random() * 0.4})`;
              matrixCtx.fillText(glitchChar, letterX, wordY);
            } else {
              // Solid letter with glow
              const alpha = Math.min(1, (formingElapsed - revealDelay - 400) / 300);
              matrixCtx.shadowBlur = 20;
              matrixCtx.shadowColor = 'rgba(255, 31, 31, 0.8)';
              matrixCtx.fillStyle = `rgba(255, 31, 31, ${alpha})`;
              matrixCtx.fillText(targetWord[i], letterX, wordY);
              matrixCtx.shadowBlur = 0;
            }
          }
        }
      }
    }

    let matrixInterval = setInterval(drawMatrix, 50);

    // ---------- Audio (WebAudio self-contained) ----------
    let audioCtx = null;
    function initAudio(){
      if(audioCtx) return; audioCtx = new (window.AudioContext || window.webkitAudioContext)();
    }

    function playTone(freq=440, time=0.08, type='sine', gain=0.08){
      if(!audioCtx) return; const o = audioCtx.createOscillator(); const g = audioCtx.createGain(); o.type = type; o.frequency.value = freq; g.gain.value = gain; o.connect(g); g.connect(audioCtx.destination); o.start(); g.gain.exponentialRampToValueAtTime(0.0001, audioCtx.currentTime + time); setTimeout(()=>{ try{ o.stop(); }catch(e){} }, (time+0.02)*1000);
    }

    function playClick(){ if(!audioCtx) return; const o = audioCtx.createOscillator(); const g = audioCtx.createGain(); o.type='square'; o.frequency.value=1200; g.gain.value=0.04; o.connect(g); g.connect(audioCtx.destination); o.start(); g.gain.exponentialRampToValueAtTime(0.0001, audioCtx.currentTime+0.06); setTimeout(()=>{try{o.stop()}catch(e){}},120); }

    function playTick(){ if(!audioCtx) return; playTone(1000,0.06,'sawtooth',0.05); }

    // glitch noise: uses buffer source with bandpass and waveshaper for distortion
    function startGlitch(duration=2.0){ if(!audioCtx) return; const bufferSize = audioCtx.sampleRate * duration; const buffer = audioCtx.createBuffer(1, bufferSize, audioCtx.sampleRate); const data = buffer.getChannelData(0); for(let i=0;i<bufferSize;i++) data[i] = (Math.random()*2-1)*0.8; const src = audioCtx.createBufferSource(); src.buffer = buffer; const bp = audioCtx.createBiquadFilter(); bp.type='bandpass'; bp.frequency.value = 220; bp.Q.value = 1.5; const sh = audioCtx.createWaveShaper(); const curve = new Float32Array(65536); for(let i=0;i<65536;i++){ const x = i/65536*2-1; curve[i] = Math.tanh(x*6); } sh.curve = curve; sh.oversample='4x'; const g = audioCtx.createGain(); g.gain.value = 0.6; src.connect(bp); bp.connect(sh); sh.connect(g); g.connect(audioCtx.destination); src.start(); setTimeout(()=>{ try{ src.stop(); }catch(e){} }, duration*1000); }

    // ---------- Boot animation (visual + audio) ----------
    const boot = document.getElementById('boot'); 
    const progress = document.getElementById('bootProgress'); 
    const terminal = document.getElementById('terminal'); 
    const matrixBg = document.getElementById('matrixBg');
    let bootIndex = 0;
    
    function bootAnim(){ 
      if(bootIndex < 10){ 
        bootIndex++; 
        progress.textContent = '[' + '#'.repeat(bootIndex) + '.'.repeat(10-bootIndex) + ']'; 
        if(audioCtx) playTone(900,0.06,'square',0.05); 
        setTimeout(bootAnim, 900); 
      } else { 
        // Wait for TUNGSTUN to fully form (about 2 seconds after it starts)
        setTimeout(()=>{ 
          clearInterval(matrixInterval);
          boot.style.display='none'; 
          matrixBg.style.display='none';
          terminal.style.display='flex'; 
          startTerminal(); 
        }, 2500); 
      } 
    }

    // Wait for user interaction to enable audio
    function enableAudioOnce(){ if(!audioCtx){ initAudio(); playTone(800,0.06,'sine',0.06); } window.removeEventListener('keydown', enableAudioOnce); window.removeEventListener('pointerdown', enableAudioOnce); }
    window.addEventListener('keydown', enableAudioOnce); window.addEventListener('pointerdown', enableAudioOnce);

    // start boot anim (visual); actual tones will play only after gesture enables audioCtx
    setTimeout(bootAnim, 240);

    // ---------- Terminal / interaction ----------
    const output = document.getElementById('output'); const input = document.getElementById('input'); const screen = document.getElementById('screen'); const cursor = document.getElementById('cursor');

    const initialLines = [
      '-- ACCESS OVH ARCHIVE --',
      'RETRIEVING: ALB:01...',
      '',
      '[OVH:SECURE] The Order records echoes where the world has forgotten to listen.',
      'Tungstun is the signal between silence and sound.',
      'ALB:01 — designation only.',
      'LOCATION: Archive Sector R/RED',
      'RELEASE: BLOOD MOON // CYCLE 103126',
      '-- END --',
      ''
    ];

    function typeIntro(i=0){ if(i>=initialLines.length){ append('\n> '); focusInput(); return; } append(initialLines[i]); if(audioCtx) playClick(); setTimeout(()=> typeIntro(i+1), 140 + Math.random()*90); }

    function startTerminal(){ typeIntro(); }

    function append(text){ output.textContent += (output.textContent.length? '\n' : '') + text; screen.scrollTop = screen.scrollHeight; }
    function focusInput(){ input.focus(); placeCaretAtEnd(input); }

    function placeCaretAtEnd(el){ el.focus(); if(window.getSelection && document.createRange){ const range = document.createRange(); range.selectNodeContents(el); range.collapse(false); const sel = window.getSelection(); sel.removeAllRanges(); sel.addRange(range); }}

    // commands
    function showHelp(){ append('AVAILABLE COMMANDS:\n  help — show this message\n  status — archive status\n  ovh.log — selected logs\n  access ALB:01 — attempt to open archive slot\n  clear — clear the screen'); if(audioCtx) playTone(1200,0.06,'sine',0.04); }
    function showStatus(){ append('[STATUS] ARCHIVE: ALB:01\n  STATE: CLASSIFIED\n  RELEASE: BLOOD MOON // CYCLE 103126\n  NOTE: DECRYPTION KEY DISTRIBUTION RESTRICTED'); if(audioCtx) playTone(700,0.06,'triangle',0.05); }
    function showLog(){ append('-- OVH LOG // SELECTED --\n[42] // 17:23:11 // "Signal anomaly recorded at sector R/RED."\n[77] // 01:02:04 // "Transmission contains audio pattern: tungsten hum."\n[99] // 08:11:51 // "ALB:01 indexed and sealed under Class V."\n-- END LOG --'); if(audioCtx) playTone(900,0.06,'sine',0.04); }

    // access sequence (with lore)
    let seqTriggered = false; function accessAlb(){ if(seqTriggered) return; seqTriggered = true; append('ACCESSING CLASSIFIED ARCHIVE...'); if(audioCtx) playTone(520,0.12,'sawtooth',0.06); setTimeout(()=> runAccessFragments(), 600); }

    function runAccessFragments(){ append('\n>> ALERT: PROTOCOL OVH-7 INTERRUPTED'); append('>> LOG FRAGMENTS RECOVERED:'); append('\n[LOG_12-OVH] :: containment sequence failure.'); append('[IDENTIFIERS] :: ASH_15H // C4P-01'); append('[PROTOCOL] :: Vanished Horizon Containment'); append('[STATUS] :: SYSTEM RED // OVH collapse imminent'); append('\n:: "They were just kids... they crossed a signal that was never meant to exist."'); append(':: "The frequencies—they didn\'t just echo... they resonated."'); append(':: "Initiate red veil... silence the horizon..."'); append(':: "Only this node remains."'); append('\nTRANSMISSION FLAGGED FOR SELF-TERMINATION.'); if(audioCtx) playTone(380,0.08,'sine',0.05); setTimeout(()=> startCountdown(20), 420); }

    // countdown
    let countdownTimer = null; function startCountdown(sec){ append('[SELF-TERMINATION SEQUENCE INITIATED]'); let t = sec; append('[TIMER] ' + t + 's'); if(audioCtx) tickOnce(); countdownTimer = setInterval(()=>{ t--; // replace last timer line
      const lines = output.textContent.split('\n'); for(let i=lines.length-1;i>=0;i--){ if(lines[i].startsWith('[TIMER]')){ lines[i] = '[TIMER] ' + t + 's'; break; } }
      output.textContent = lines.join('\n'); screen.scrollTop = screen.scrollHeight; if(audioCtx) tickOnce(); if(t<=0){ clearInterval(countdownTimer); beginHeavyShutdown(); } }, 1000); }

    function tickOnce(){ playTick(); }

    // heavy glitch + audio
    function beginHeavyShutdown(){ input.contentEditable='false'; append('\n[SELF-TERMINATION: EXECUTING]'); if(audioCtx) startGlitch(3.2);
      // visual heavy glitch cycles
      let cycles = 0; const total = 40; const inter = setInterval(()=>{
        cycles++; const chars='█▓▒░<>01#$_-/%@*~'; let out=''; const rows = Math.max(12, Math.floor(window.innerHeight/16)); const cols = 90;
        for(let r=0;r<rows;r++){ let row=''; for(let c=0;c<cols;c++){ row += chars[Math.floor(Math.random()*chars.length)]; } out += row + '\n'; }
        // sprinkle fragments
        if(Math.random()<0.5) out = out.replace(out.split('\n')[Math.floor(Math.random()*Math.min(6,out.split('\n').length))], ':: "RED VEIL ENGAGED — ASH_15H // C4P-01"');
        screen.innerHTML = '<pre style="font-family:var(--mono);">'+out+'</pre>'; terminal.style.transform = 'translate('+(Math.random()*12-6)+'px,'+(Math.random()*12-6)+'px)'; terminal.style.filter = 'contrast('+(1+Math.random()*0.6)+') blur('+(Math.random()*1.2)+'px)';
        if(cycles>total){ clearInterval(inter); finalizeError(); }
      }, 60);
    }

    function finalizeError(){ // wind down audio and show links
      if(audioCtx){ /* subtle final tone */ playTone(120,0.4,'sine',0.08); }
      screen.innerHTML = '';
      const errorBox = document.getElementById('errorBox'); errorBox.innerHTML = `<h1 style="color:var(--red);">SYSTEM FAILURE</h1><p style="font-family:var(--mono)">ERROR CODE: [NULL-15H]</p><p style="opacity:0.85">archive node terminated — only remnants remain.</p><div class="links"><a href="https://www.youtube.com/c/Tungstun" target="_blank">YOUTUBE // TUNGSTUN</a><a href="https://instagram.com/tungstunmusic" target="_blank">INSTAGRAM // TUNGSTUN</a></div>`;
      document.getElementById('errorScreen').style.display = 'flex'; terminal.style.transform='none'; terminal.style.filter='none';
    }

    // input handling
    input.addEventListener('keydown', (e)=>{
      if(e.key === 'Enter'){
        e.preventDefault(); const raw = input.textContent.trim(); if(!raw) { input.textContent = ''; return; }
        const cmd = raw.toLowerCase(); append('> ' + raw); input.textContent = '';
        if(!audioCtx) { initAudio(); playClick(); }
        // basic commands
        if(cmd === 'help') { showHelp(); }
        else if(cmd === 'status') { showStatus(); }
        else if(cmd === 'ovh.log' || cmd === 'ovh log' || cmd === 'log') { showLog(); }
        else if(cmd.startsWith('access')){ if(cmd.includes('alb:01') || cmd.includes('alb 01') || cmd.includes('alb01')) { accessAlb(); } else { append('[ERROR] Unknown target. Try: access ALB:01'); if(audioCtx) playTone(400,0.06,'sawtooth',0.04); } }
        else if(cmd === 'clear') { output.textContent = ''; }
        else { append('[ERROR] Unknown command — type help'); if(audioCtx) playTone(300,0.06,'sine',0.03); }
      }
    });

    // focus/cursor niceties
    window.addEventListener('click', ()=> input.focus()); setTimeout(()=> input.focus(), 1200);

    // accessibility scroll
    const observer = new MutationObserver(()=>{ screen.scrollTop = screen.scrollHeight; }); observer.observe(output, { childList:true, subtree:true, characterData:true });
  </script>
</body>
</html>
