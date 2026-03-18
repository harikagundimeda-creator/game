# game
<!doctype html>
<html lang="en" class="h-full">
 <head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Number Guessing Game</title>
  <script src="https://cdn.tailwindcss.com/3.4.17"></script>
  <script src="https://cdn.jsdelivr.net/npm/lucide@0.263.0/dist/umd/lucide.min.js"></script>
  <script src="/_sdk/element_sdk.js"></script>
  <link href="https://fonts.googleapis.com/css2?family=Outfit:wght@300;400;500;600;700;800&amp;family=JetBrains+Mono:wght@400;600;700&amp;display=swap" rel="stylesheet">
  <style>
    * { margin: 0; padding: 0; box-sizing: border-box; }

    @keyframes float {
      0%, 100% { transform: translateY(0px); }
      50% { transform: translateY(-8px); }
    }

    @keyframes pulseGlow {
      0%, 100% { box-shadow: 0 0 20px rgba(var(--accent-rgb), 0.2); }
      50% { box-shadow: 0 0 40px rgba(var(--accent-rgb), 0.4); }
    }

    @keyframes slideUp {
      from { opacity: 0; transform: translateY(20px); }
      to { opacity: 1; transform: translateY(0); }
    }

    @keyframes shake {
      0%, 100% { transform: translateX(0); }
      20% { transform: translateX(-8px); }
      40% { transform: translateX(8px); }
      60% { transform: translateX(-5px); }
      80% { transform: translateX(5px); }
    }

    @keyframes celebrate {
      0% { transform: scale(1); }
      25% { transform: scale(1.1) rotate(-2deg); }
      50% { transform: scale(1.15) rotate(2deg); }
      75% { transform: scale(1.1) rotate(-1deg); }
      100% { transform: scale(1); }
    }

    @keyframes confetti-fall {
      0% { transform: translateY(-100%) rotate(0deg); opacity: 1; }
      100% { transform: translateY(800%) rotate(720deg); opacity: 0; }
    }

    .hint-slide { animation: slideUp 0.3s ease-out; }
    .shake { animation: shake 0.5s ease-in-out; }
    .celebrate { animation: celebrate 0.6s ease-in-out; }

    .confetti-piece {
      position: absolute;
      width: 8px;
      height: 8px;
      top: -10px;
      border-radius: 2px;
      animation: confetti-fall 2.5s ease-in forwards;
    }

    .number-input::-webkit-outer-spin-button,
    .number-input::-webkit-inner-spin-button {
      -webkit-appearance: none;
      margin: 0;
    }
    /* .number-input {  -moz-animation: alternate-reverse;} */

    .guess-history-item {
      transition: all 0.3s ease;
    }
  </style>
  <style>body { box-sizing: border-box; }</style>
  <script src="/_sdk/data_sdk.js" type="text/javascript"></script>
 </head>
 <body class="h-full">
  <div id="app" class="h-full w-full overflow-auto" style="background: #0f172a; font-family: 'Outfit', sans-serif;"><!-- Confetti container -->
   <div id="confetti-container" class="fixed inset-0 pointer-events-none z-50 overflow-hidden"></div><!-- Background pattern -->
   <div class="fixed inset-0 opacity-[0.03]" style="background-image: radial-gradient(circle at 1px 1px, white 1px, transparent 0); background-size: 40px 40px;"></div>
   <div class="relative z-10 flex flex-col items-center justify-center min-h-full px-4 py-8"><!-- Title -->
    <div class="text-center mb-8" style="animation: slideUp 0.5s ease-out;">
     <div class="inline-flex items-center gap-3 mb-3">
      <div class="w-10 h-10 rounded-xl flex items-center justify-center" style="background: #f59e0b;"><i data-lucide="hash" style="width: 22px; height: 22px; color: #0f172a;"></i>
      </div>
      <h1 id="game-title" class="text-3xl md:text-4xl font-800 tracking-tight" style="color: #f1f5f9; font-family: 'JetBrains Mono', monospace;">Guess The Number</h1>
     </div>
     <p id="instruction-text" class="text-base font-300" style="color: #64748b;">I'm thinking of a number between 1 and 100. Can you find it?</p>
    </div><!-- Game Card -->
    <div id="game-card" class="w-full max-w-md rounded-2xl p-6 md:p-8 relative overflow-hidden" style="background: #1e293b; border: 1px solid #334155;"><!-- Attempts & Range -->
     <div class="flex justify-between items-center mb-6">
      <div class="flex items-center gap-2"><i data-lucide="target" style="width: 16px; height: 16px; color: #f59e0b;"></i> <span class="text-sm font-500" style="color: #94a3b8;">Attempts: <span id="attempts" class="font-700" style="color: #f59e0b;">0</span></span>
      </div>
      <div class="flex items-center gap-2"><i data-lucide="activity" style="width: 16px; height: 16px; color: #f59e0b;"></i> <span class="text-sm font-500" style="color: #94a3b8;">Range: <span id="range-display" class="font-700" style="color: #f59e0b;">1–100</span></span>
      </div>
     </div><!-- Input area -->
     <div id="input-area">
      <div class="flex gap-3 mb-4"><input type="number" id="guess-input" class="number-input flex-1 rounded-xl px-5 py-4 text-xl font-600 outline-none transition-all duration-200 focus:ring-2" style="background: #0f172a; border: 2px solid #334155; color: #f1f5f9; font-family: 'JetBrains Mono', monospace; --tw-ring-color: #f59e0b;" placeholder="Enter your guess..." min="1" max="100" autocomplete="off"> <button id="guess-btn" class="rounded-xl px-6 py-4 font-600 text-lg transition-all duration-200 hover:scale-105 active:scale-95" style="background: #f59e0b; color: #0f172a;" onclick="makeGuess()"> <i data-lucide="arrow-right" style="width: 22px; height: 22px;"></i> </button>
      </div>
     </div><!-- Hint Display -->
     <div id="hint-container" class="mb-4 hidden">
      <div id="hint-box" class="rounded-xl px-5 py-4 flex items-center gap-3"><i id="hint-icon" data-lucide="info" style="width: 20px; height: 20px; flex-shrink: 0;"></i> <span id="hint-text" class="font-500 text-base"></span>
      </div>
     </div><!-- Win Display -->
     <div id="win-display" class="hidden text-center py-4">
      <div class="text-5xl mb-3">
       🎉
      </div>
      <h2 class="text-2xl font-800 mb-1" style="color: #4ade80; font-family: 'JetBrains Mono', monospace;">You Got It!</h2>
      <p id="win-message" class="text-base mb-5" style="color: #94a3b8;"></p><button id="play-again-btn" class="rounded-xl px-8 py-3 font-600 text-base transition-all duration-200 hover:scale-105 active:scale-95 inline-flex items-center gap-2" style="background: #f59e0b; color: #0f172a;" onclick="resetGame()"> <i data-lucide="refresh-cw" style="width: 18px; height: 18px;"></i> Play Again </button>
     </div><!-- Guess History -->
     <div id="history-section" class="hidden mt-5 pt-5" style="border-top: 1px solid #334155;">
      <h3 class="text-xs font-600 uppercase tracking-widest mb-3" style="color: #64748b;">Your Guesses</h3>
      <div id="history-list" class="flex flex-wrap gap-2"></div>
     </div>
    </div><!-- Keyboard hint -->
    <p class="mt-4 text-xs font-400" style="color: #475569;"><span class="inline-flex items-center gap-1"> <kbd class="px-1.5 py-0.5 rounded text-[10px] font-500" style="background: #1e293b; border: 1px solid #334155; color: #64748b;">Enter</kbd> to submit your guess </span></p>
   </div>
  </div>
  <script>
    // Game state
    let secretNumber = generateNumber();
    let attempts = 0;
    let guessHistory = [];
    let rangeLow = 1;
    let rangeHigh = 100;
    let gameOver = false;

    function generateNumber() {
      return Math.floor(Math.random() * 100) + 1;
    }

    const guessInput = document.getElementById('guess-input');
    const attemptsEl = document.getElementById('attempts');
    const rangeDisplay = document.getElementById('range-display');
    const hintContainer = document.getElementById('hint-container');
    const hintBox = document.getElementById('hint-box');
    const hintText = document.getElementById('hint-text');
    const hintIcon = document.getElementById('hint-icon');
    const winDisplay = document.getElementById('win-display');
    const winMessage = document.getElementById('win-message');
    const inputArea = document.getElementById('input-area');
    const historySection = document.getElementById('history-section');
    const historyList = document.getElementById('history-list');
    const gameCard = document.getElementById('game-card');

    guessInput.addEventListener('keydown', (e) => {
      if (e.key === 'Enter') {
        e.preventDefault();
        makeGuess();
      }
    });

    function makeGuess() {
      if (gameOver) return;

      const val = parseInt(guessInput.value);
      if (isNaN(val) || val < 1 || val > 100) {
        showHint('Please enter a number between 1 and 100', 'alert-circle', '#ef4444', 'rgba(239,68,68,0.1)');
        gameCard.classList.add('shake');
        setTimeout(() => gameCard.classList.remove('shake'), 500);
        return;
      }

      attempts++;
      attemptsEl.textContent = attempts;
      guessHistory.push({ value: val, direction: null });

      if (val === secretNumber) {
        // Win!
        guessHistory[guessHistory.length - 1].direction = 'correct';
        gameOver = true;
        hintContainer.classList.add('hidden');
        inputArea.style.display = 'none';
        winDisplay.classList.remove('hidden');
        winMessage.textContent = `The number was ${secretNumber}. You found it in ${attempts} attempt${attempts > 1 ? 's' : ''}!`;
        gameCard.classList.add('celebrate');
        setTimeout(() => gameCard.classList.remove('celebrate'), 600);
        launchConfetti();
      } else if (val < secretNumber) {
        guessHistory[guessHistory.length - 1].direction = 'low';
        rangeLow = Math.max(rangeLow, val + 1);
        rangeDisplay.textContent = `${rangeLow}–${rangeHigh}`;
        showHint(`Too Low! Go higher ↑`, 'arrow-up', '#38bdf8', 'rgba(56,189,248,0.1)');
        gameCard.classList.add('shake');
        setTimeout(() => gameCard.classList.remove('shake'), 500);
      } else {
        guessHistory[guessHistory.length - 1].direction = 'high';
        rangeHigh = Math.min(rangeHigh, val - 1);
        rangeDisplay.textContent = `${rangeLow}–${rangeHigh}`;
        showHint(`Too High! Go lower ↓`, 'arrow-down', '#f472b6', 'rgba(244,114,182,0.1)');
        gameCard.classList.add('shake');
        setTimeout(() => gameCard.classList.remove('shake'), 500);
      }

      renderHistory();
      guessInput.value = '';
      guessInput.focus();
    }

    function showHint(text, icon, color, bgColor) {
      hintContainer.classList.remove('hidden');
      hintBox.style.background = bgColor;
      hintBox.style.border = `1px solid ${color}33`;
      hintText.textContent = text;
      hintText.style.color = color;
      hintIcon.setAttribute('data-lucide', icon);
      hintIcon.style.color = color;
      lucide.createIcons();
      hintBox.classList.remove('hint-slide');
      void hintBox.offsetWidth;
      hintBox.classList.add('hint-slide');
    }

    function renderHistory() {
      historySection.classList.remove('hidden');
      historyList.innerHTML = '';
      guessHistory.forEach((g, i) => {
        const chip = document.createElement('div');
        chip.className = 'guess-history-item rounded-lg px-3 py-1.5 text-sm font-600 flex items-center gap-1.5';
        chip.style.fontFamily = "'JetBrains Mono', monospace";
        chip.style.animationDelay = `${i * 0.05}s`;

        let bg, color, arrow;
        if (g.direction === 'low') {
          bg = 'rgba(56,189,248,0.15)'; color = '#38bdf8'; arrow = '↑';
        } else if (g.direction === 'high') {
          bg = 'rgba(244,114,182,0.15)'; color = '#f472b6'; arrow = '↓';
        } else {
          bg = 'rgba(74,222,128,0.15)'; color = '#4ade80'; arrow = '✓';
        }

        chip.style.background = bg;
        chip.style.color = color;
        chip.textContent = `${g.value} ${arrow}`;
        historyList.appendChild(chip);
      });
    }

    function resetGame() {
      secretNumber = generateNumber();
      attempts = 0;
      guessHistory = [];
      rangeLow = 1;
      rangeHigh = 100;
      gameOver = false;

      attemptsEl.textContent = '0';
      rangeDisplay.textContent = '1–100';
      hintContainer.classList.add('hidden');
      winDisplay.classList.add('hidden');
      inputArea.style.display = '';
      historySection.classList.add('hidden');
      historyList.innerHTML = '';
      guessInput.value = '';
      guessInput.focus();
    }

    function launchConfetti() {
      const container = document.getElementById('confetti-container');
      const colors = ['#f59e0b', '#4ade80', '#38bdf8', '#f472b6', '#a78bfa', '#fb923c'];
      for (let i = 0; i < 60; i++) {
        const piece = document.createElement('div');
        piece.className = 'confetti-piece';
        piece.style.left = Math.random() * 100 + '%';
        piece.style.background = colors[Math.floor(Math.random() * colors.length)];
        piece.style.animationDelay = Math.random() * 1 + 's';
        piece.style.animationDuration = (2 + Math.random() * 1.5) + 's';
        piece.style.width = (5 + Math.random() * 6) + 'px';
        piece.style.height = (5 + Math.random() * 6) + 'px';
        piece.style.borderRadius = Math.random() > 0.5 ? '50%' : '2px';
        container.appendChild(piece);
      }
      setTimeout(() => { container.innerHTML = ''; }, 4000);
    }

    // Element SDK
    const defaultConfig = {
      game_title: 'Guess The Number',
      instruction_text: "I'm thinking of a number between 1 and 100. Can you find it?",
      background_color: '#0f172a',
      surface_color: '#1e293b',
      text_color: '#f1f5f9',
      accent_color: '#f59e0b',
      muted_color: '#64748b',
      font_family: 'Outfit',
      font_size: 16
    };

    function applyConfig(config) {
      const bg = config.background_color || defaultConfig.background_color;
      const surface = config.surface_color || defaultConfig.surface_color;
      const text = config.text_color || defaultConfig.text_color;
      const accent = config.accent_color || defaultConfig.accent_color;
      const muted = config.muted_color || defaultConfig.muted_color;
      const font = config.font_family || defaultConfig.font_family;
      const baseSize = config.font_size || defaultConfig.font_size;

      document.getElementById('app').style.background = bg;
      document.getElementById('game-card').style.background = surface;

      document.getElementById('game-title').textContent = config.game_title || defaultConfig.game_title;
      document.getElementById('game-title').style.color = text;

      document.getElementById('instruction-text').textContent = config.instruction_text || defaultConfig.instruction_text;
      document.getElementById('instruction-text').style.color = muted;

      // Font
      const fontStack = `${font}, 'Outfit', sans-serif`;
      document.getElementById('app').style.fontFamily = fontStack;

      // Font sizes
      document.getElementById('game-title').style.fontSize = `${baseSize * 2}px`;
      document.getElementById('instruction-text').style.fontSize = `${baseSize}px`;

      // Accent color on buttons and highlights
      const guessBtn = document.getElementById('guess-btn');
      guessBtn.style.background = accent;

      const playAgainBtn = document.getElementById('play-again-btn');
      playAgainBtn.style.background = accent;

      document.querySelectorAll('[style*="color: #f59e0b"]').forEach(el => {
        // Only update the specific accent-colored spans
      });
      attemptsEl.style.color = accent;
      rangeDisplay.style.color = accent;

      const iconContainer = document.querySelector('.w-10.h-10');
      if (iconContainer) iconContainer.style.background = accent;
    }

    if (window.elementSdk) {
      window.elementSdk.init({
        defaultConfig,
        onConfigChange: async (config) => {
          applyConfig(config);
        },
        mapToCapabilities: (config) => ({
          recolorables: [
            {
              get: () => config.background_color || defaultConfig.background_color,
              set: (v) => { config.background_color = v; window.elementSdk.setConfig({ background_color: v }); }
            },
            {
              get: () => config.surface_color || defaultConfig.surface_color,
              set: (v) => { config.surface_color = v; window.elementSdk.setConfig({ surface_color: v }); }
            },
            {
              get: () => config.text_color || defaultConfig.text_color,
              set: (v) => { config.text_color = v; window.elementSdk.setConfig({ text_color: v }); }
            },
            {
              get: () => config.accent_color || defaultConfig.accent_color,
              set: (v) => { config.accent_color = v; window.elementSdk.setConfig({ accent_color: v }); }
            },
            {
              get: () => config.muted_color || defaultConfig.muted_color,
              set: (v) => { config.muted_color = v; window.elementSdk.setConfig({ muted_color: v }); }
            }
          ],
          borderables: [],
          fontEditable: {
            get: () => config.font_family || defaultConfig.font_family,
            set: (v) => { config.font_family = v; window.elementSdk.setConfig({ font_family: v }); }
          },
          fontSizeable: {
            get: () => config.font_size || defaultConfig.font_size,
            set: (v) => { config.font_size = v; window.elementSdk.setConfig({ font_size: v }); }
          }
        }),
        mapToEditPanelValues: (config) => new Map([
          ['game_title', config.game_title || defaultConfig.game_title],
          ['instruction_text', config.instruction_text || defaultConfig.instruction_text]
        ])
      });
    }

    // Init icons
    lucide.createIcons();
    guessInput.focus();
  </script>
 <script>(function(){function c(){var b=a.contentDocument||a.contentWindow.document;if(b){var d=b.createElement('script');d.innerHTML="window.__CF$cv$params={r:'9de241cc91a74001',t:'MTc3MzgxNjE0My4wMDAwMDA='};var a=document.createElement('script');a.nonce='';a.src='/cdn-cgi/challenge-platform/scripts/jsd/main.js';document.getElementsByTagName('head')[0].appendChild(a);";b.getElementsByTagName('head')[0].appendChild(d)}}if(document.body){var a=document.createElement('iframe');a.height=1;a.width=1;a.style.position='absolute';a.style.top=0;a.style.left=0;a.style.border='none';a.style.visibility='hidden';document.body.appendChild(a);if('loading'!==document.readyState)c();else if(window.addEventListener)document.addEventListener('DOMContentLoaded',c);else{var e=document.onreadystatechange||function(){};document.onreadystatechange=function(b){e(b);'loading'!==document.readyState&&(document.onreadystatechange=e,c())}}}})();</script></body>
</html>
