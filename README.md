<!doctype html>
<html lang="en">
<head>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width,initial-scale=1" />
  <title>Tic‑Tac‑Toe (XOX)</title>
  <style>
    :root{--bg:#0f1724;--card:#0b1220;--accent:#60a5fa;--muted:#94a3b8;--win:#22c55e}
    *{box-sizing:border-box}
    body{margin:0;font-family:Inter,ui-sans-serif,system-ui, -apple-system, "Segoe UI", Roboto, "Helvetica Neue", Arial;background:linear-gradient(180deg,#071024 0%,#041128 100%);color:#e6eef8;display:flex;align-items:center;justify-content:center;min-height:100vh;padding:24px}
    .container{width:100%;max-width:760px;background:linear-gradient(180deg,#071428 0%, rgba(255,255,255,0.02) 100%);border-radius:14px;padding:20px;box-shadow:0 8px 30px rgba(2,6,23,.6);}
    header{display:flex;align-items:center;justify-content:space-between;gap:12px}
    h1{font-size:20px;margin:0}
    .controls{display:flex;gap:8px;align-items:center}
    select, button{background:transparent;border:1px solid rgba(255,255,255,0.06);color:inherit;padding:8px 10px;border-radius:8px;cursor:pointer}
    .board{display:grid;grid-template-columns:repeat(3,1fr);gap:10px;margin-top:18px}
    .cell{aspect-ratio:1/1;background:linear-gradient(180deg,rgba(255,255,255,0.02),transparent);display:flex;align-items:center;justify-content:center;font-size:48px;border-radius:10px;user-select:none;cursor:pointer;transition:transform .12s, box-shadow .12s}
    .cell:active{transform:scale(.98)}
    .cell.disabled{cursor:not-allowed;opacity:.86}
    .meta{display:flex;justify-content:space-between;align-items:center;margin-top:14px;color:var(--muted)}
    .score{display:flex;gap:14px}
    .score div{background:rgba(255,255,255,0.03);padding:8px 12px;border-radius:10px}
    .message{margin-top:12px;padding:10px;border-radius:8px;background:rgba(255,255,255,0.02)}
    .line{position:relative}
    .winner{color:var(--win);font-weight:700}
    .highlight{box-shadow:0 6px 20px rgba(34,197,94,.18);border:2px solid rgba(34,197,94,.25)}
    footer{margin-top:14px;font-size:13px;color:var(--muted);text-align:center}
    @media (max-width:520px){.cell{font-size:36px}}
  </style>
</head>
<body>
  <div class="container" role="application" aria-label="Tic Tac Toe">
    <header>
      <h1>Tic‑Tac‑Toe (XOX)</h1>
      <div class="controls">
        <label>
          Mode
          <select id="mode">
            <option value="pvp">2‑player local</option>
            <option value="pvai">Play vs CPU</option>
          </select>
        </label>
        <label>
          Difficulty
          <select id="difficulty">
            <option value="easy">Easy (random)</option>
            <option value="hard">Hard (minimax)</option>
          </select>
        </label>
        <button id="restart">Restart</button>
      </div>
    </header>

    <div class="meta">
      <div class="score">
        <div id="scoreX">X: 0</div>
        <div id="scoreO">O: 0</div>
        <div id="ties">Ties: 0</div>
      </div>
      <div class="line">Turn: <strong id="turn">X</strong></div>
    </div>

    <div class="board" id="board" aria-label="Game board"></div>

    <div class="message" id="message">Click a cell to start.</div>

    <footer>Built with plain HTML/CSS/JS — responsive and keyboard accessible.</footer>
  </div>

  <script>
    // Game state
    const boardEl = document.getElementById('board');
    const messageEl = document.getElementById('message');
    const turnEl = document.getElementById('turn');
    const restartBtn = document.getElementById('restart');
    const modeSelect = document.getElementById('mode');
    const diffSelect = document.getElementById('difficulty');
    const scoreXEl = document.getElementById('scoreX');
    const scoreOEl = document.getElementById('scoreO');
    const tiesEl = document.getElementById('ties');

    let board = Array(9).fill(null);
    let current = 'X';
    let running = true;
    let scores = {X:0,O:0,ties:0};

    const wins = [
      [0,1,2],[3,4,5],[6,7,8],
      [0,3,6],[1,4,7],[2,5,8],
      [0,4,8],[2,4,6]
    ];

    function renderBoard(){
      boardEl.innerHTML = '';
      board.forEach((v,i)=>{
        const cell = document.createElement('div');
        cell.className = 'cell'+(v? ' disabled': '');
        cell.tabIndex = 0;
        cell.setAttribute('role','button');
        cell.setAttribute('aria-label', v? `Cell ${i+1} containing ${v}` : `Empty cell ${i+1}`);
        cell.innerHTML = v || '';
        cell.addEventListener('click', ()=>onCellClick(i));
        cell.addEventListener('keydown', (e)=>{ if(e.key === 'Enter' || e.key === ' ') { e.preventDefault(); onCellClick(i);} });
        boardEl.appendChild(cell);
      });
      updateTurnUI();
    }

    function updateTurnUI(){
      turnEl.textContent = current;
    }

    function onCellClick(index){
      if(!running || board[index]) return;
      makeMove(index, current);
      if(running && modeSelect.value === 'pvai' && current === 'O'){
        // CPU's turn (O)
        setTimeout(cpuMove, 220);
      }
    }

    function makeMove(index, player){
      board[index] = player;
      renderBoard();
      const res = checkWinner(board);
      if(res){
        handleResult(res);
      } else if(board.every(Boolean)){
        handleResult({winner: null});
      } else {
        current = (player === 'X')? 'O' : 'X';
        updateTurnUI();
      }
    }

    function handleResult(result){
      running = false;
      if(result.winner){
        messageEl.innerHTML = `<span class="winner">${result.winner}</span> wins!`;
        scores[result.winner]++;
        highlightWinningCells(result.line);
      } else {
        messageEl.textContent = 'Tie game.';
        scores.ties++;
      }
      updateScores();
    }

    function highlightWinningCells(line){
      if(!line) return;
      const cells = boardEl.querySelectorAll('.cell');
      line.forEach(i=>cells[i].classList.add('highlight'));
    }

    function updateScores(){
      scoreXEl.textContent = `X: ${scores.X}`;
      scoreOEl.textContent = `O: ${scores.O}`;
      tiesEl.textContent = `Ties: ${scores.ties}`;
    }

    function checkWinner(b){
      for(const line of wins){
        const [a,c,d] = line;
        if(b[a] && b[a] === b[c] && b[a] === b[d]){
          return {winner: b[a], line};
        }
      }
      return null;
    }

    // CPU logic
    function cpuMove(){
      if(!running) return;
      const diff = diffSelect.value;
      let idx;
      if(diff === 'easy'){
        const empties = board.map((v,i)=>v? -1:i).filter(i=>i>=0);
        idx = empties[Math.floor(Math.random()*empties.length)];
      } else {
        idx = bestMove(board, 'O');
      }
      makeMove(idx, 'O');
    }

    // Minimax
    function bestMove(b, player){
      const opponent = player === 'X' ? 'O' : 'X';
      // If immediate win available, take it.
      for(let i=0;i<9;i++){
        if(!b[i]){
          b[i] = player;
          if(checkWinner(b) && checkWinner(b).winner === player){ b[i] = null; return i; }
          b[i] = null;
        }
      }
      // If must block opponent immediate win.
      for(let i=0;i<9;i++){
        if(!b[i]){
          b[i] = opponent;
          if(checkWinner(b) && checkWinner(b).winner === opponent){ b[i] = null; return i; }
          b[i] = null;
        }
      }
      // Minimax search
      function minimax(boardState, depth, isMax){
        const winnerRes = checkWinner(boardState);
        if(winnerRes){
          if(winnerRes.winner === 'O') return 10 - depth;
          if(winnerRes.winner === 'X') return depth - 10;
        }
        if(boardState.every(Boolean)) return 0;
        let best = isMax ? -Infinity : Infinity;
        for(let i=0;i<9;i++){
          if(!boardState[i]){
            boardState[i] = isMax ? 'O' : 'X';
            const score = minimax(boardState, depth+1, !isMax);
            boardState[i] = null;
            if(isMax) best = Math.max(best, score); else best = Math.min(best, score);
          }
        }
        return best;
      }
      let move = null; let bestScore = -Infinity;
      for(let i=0;i<9;i++){
        if(!b[i]){
          b[i] = 'O';
          const s = minimax(b, 0, false);
          b[i] = null;
          if(s > bestScore){ bestScore = s; move = i; }
        }
      }
      return move === null ? board.findIndex(x=>!x) : move;
    }

    // Controls
    restartBtn.addEventListener('click', ()=>{
      resetBoard();
    });

    modeSelect.addEventListener('change', ()=>{
      resetBoard();
    });

    diffSelect.addEventListener('change', ()=>{
      resetBoard();
    });

    function resetBoard(){
      board = Array(9).fill(null);
      current = 'X';
      running = true;
      messageEl.textContent = 'Game reset. Click a cell to start.';
      const cells = boardEl.querySelectorAll('.cell');
      cells.forEach(c=>c.classList.remove('highlight'));
      renderBoard();
    }

    // Initialize
    function init(){
      // build grid
      for(let i=0;i<9;i++){
        const slot = document.createElement('div');
        slot.className = 'cell';
      }
      renderBoard();
      updateScores();
    }

    init();
  </script>
</body>
</html>
