DOCTYPE html>
<html lang="ru">
<head>
<meta charset="utf-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0, user-scalable=no">
<title>Шахматы от Jenia</title>
<style>
* {
  box-sizing: border-box;
  margin: 0;
  padding: 0;
}
body {
  background: #0d131f;
  color: #fff;
  font-family: sans-serif;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  min-height: 100vh;
  padding: 8px;
  touch-action: none;
  user-select: none;
}
h2 {
  margin-bottom: 10px;
  color: #ffd700;
  font-size: 22px;
  text-align: center;
}
#board-wrap {
  padding: 5px;
  background: #d4af37;
  border-radius: 10px;
}
#board {
  display: grid;
  grid-template-columns: repeat(8, 1fr);
  grid-template-rows: repeat(8, 1fr);
  width: 92vw;
  height: 92vw;
  max-width: 440px;
  max-height: 440px;
  border-radius: 4px;
  overflow: hidden;
}
.sq {
  position: relative;
  display: flex;
  justify-content: center;
  align-items: center;
  font-size: 9vw;
  cursor: pointer;
}
@media (min-width: 460px) {
  .sq { font-size: 42px; }
}
.light { background: #f3e5ab; }
.dark { background: #165b49; }
.selected { background: #fbc02d !important; }
.last-move {
  box-shadow: inset 0 0 0 3px #00e676;
}
.hint::after {
  content: "";
  width: 26%;
  height: 26%;
  background: rgba(255, 215, 0, 0.85);
  border-radius: 50%;
}
.piece-w {
  color: #ffffff;
  text-shadow: 0 2px 3px #000, 0 0 8px #00e676;
  pointer-events: none;
}
.piece-b {
  color: #111622;
  text-shadow: 0 0 2px #ffd700, 0 2px 4px #000;
  pointer-events: none;
}
.coord-rank {
  position: absolute;
  top: 2px;
  left: 3px;
  font-size: 10px;
  font-weight: bold;
  pointer-events: none;
}
.coord-file {
  position: absolute;
  bottom: 1px;
  right: 3px;
  font-size: 10px;
  font-weight: bold;
  pointer-events: none;
}
.light .coord-rank,
.light .coord-file { color: #0d3b2e; }
.dark .coord-rank,
.dark .coord-file { color: #f3e5ab; }
#drag-ghost {
  position: fixed;
  pointer-events: none;
  z-index: 9999;
  font-size: 46px;
  transform: translate(-50%, -50%);
  display: none;
}
#status {
  margin-top: 14px;
  font-size: 17px;
  color: #f3e5ab;
  font-weight: bold;
  min-height: 24px;
}
button {
  margin-top: 12px;
  padding: 12px 26px;
  font-size: 16px;
  border: none;
  border-radius: 8px;
  background: #ffd700;
  color: #090d16;
  font-weight: bold;
  cursor: pointer;
}
</style>
</head>
<body>
  <h2>♔ Шахматы от Jenia ♚</h2>
  <div id="board-wrap">
    <div id="board"></div>
  </div>
  <div id="drag-ghost"></div>
  <div id="status">Ваш ход (Белые)</div>
  <button onclick="resetGame()">Новая партия</button>

<script>
const PIECES = {
  P:'♙', N:'♘', B:'♗', R:'♖', Q:'♕', K:'♔',
  p:'♟', n:'♞', b:'♝', r:'♜', q:'♛', k:'♚'
};
const VALS = {p:10, n:32, b:33, r:50, q:90, k:900};
let board = [];
let sel = null;
let hints = [];
let last = null;
let turn = 'w';
let drag = null;

function initBoard() {
  board = [
    ['r','n','b','q','k','b','n','r'],
    ['p','p','p','p','p','p','p','p'],
    ['','','','','','','',''],
    ['','','','','','','',''],
    ['','','','','','','',''],
    ['','','','','','','',''],
    ['P','P','P','P','P','P','P','P'],
    ['R','N','B','Q','K','B','N','R']
  ];
  sel = null;
  hints = [];
  last = null;
  turn = 'w';
  document.getElementById('status').textContent = 'Ваш ход (Белые)';
  draw();
}

function isW(p) { return p !== '' && p === p.toUpperCase(); }
function isB(p) { return p !== '' && p === p.toLowerCase(); }

function getMoves(r, c, b) {
  const p = b[r][c];
  if (!p) return [];
  const res = [];
  const w = isW(p);
  const t = p.toLowerCase();

  function add(nr, nc) {
    if (nr < 0 || nr > 7 || nc < 0 || nc > 7) return false;
    const tgt = b[nr][nc];
    if (tgt === '') {
      res.push({r: nr, c: nc});
      return true;
    }
    if (w !== isW(tgt)) {
      res.push({r: nr, c: nc});
    }
    return false;
  }

  if (t === 'p') {
    const dir = w ? -1 : 1;
    const st = w ? 6 : 1;
    if (r + dir >= 0 && r + dir <= 7 && b[r + dir][c] === '') {
      res.push({r: r + dir, c: c});
      if (r === st && b[r + 2 * dir][c] === '') {
        res.push({r: r + 2 * dir, c: c});
      }
    }
    const dcList = [-1, 1];
    for (let i = 0; i < dcList.length; i++) {
      const nr = r + dir;
      const nc = c + dcList[i];
      if (nr >= 0 && nr <= 7 && nc >= 0 && nc <= 7) {
        if (b[nr][nc] !== '' && w !== isW(b[nr][nc])) {
          res.push({r: nr, c: nc});
        }
      }
    }
  } else if (t === 'n') {
    add(r - 2, c - 1); add(r - 2, c + 1);
    add(r - 1, c - 2); add(r - 1, c + 2);
    add(r + 1, c - 2); add(r + 1, c + 2);
    add(r + 2, c - 1); add(r + 2, c + 1);
  } else if (t === 'k') {
    for (let dr = -1; dr <= 1; dr++) {
      for (let dc = -1; dc <= 1; dc++) {
        if (dr !== 0 || dc !== 0) add(r + dr, c + dc);
      }
    }
  } else {
    const drs = [];
    const dcs = [];
    if (t === 'r' || t === 'q') {
      drs.push(-1, 1, 0, 0);
      dcs.push(0, 0, -1, 1);
    }
    if (t === 'b' || t === 'q') {
      drs.push(-1, -1, 1, 1);
      dcs.push(-1, 1, -1, 1);
    }
    for (let i = 0; i < drs.length; i++) {
      let nr = r + drs[i];
      let nc = c + dcs[i];
      while (add(nr, nc)) {
        nr += drs[i];
        nc += dcs[i];
      }
    }
  }
  return res;
}

function draw() {
  const el = document.getElementById('board');
  el.innerHTML = '';
  const files = ['a','b','c','d','e','f','g','h'];
  for (let r = 0; r < 8; r++) {
    for (let c = 0; c < 8; c++) {
      const sq = document.createElement('div');
      const isLight = (r + c) % 2 === 0;
      sq.className = 'sq ' + (isLight ? 'light' : 'dark');
      sq.dataset.r = r;
      sq.dataset.c = c;

      if (sel && sel.r === r && sel.c === c) {
        sq.classList.add('selected');
      }
      if (last) {
        const isFrom = (last.sr === r && last.sc === c);
        const isTo = (last.r === r && last.c === c);
        if (isFrom || isTo) sq.classList.add('last-move');
      }
      for (let h = 0; h < hints.length; h++) {
        if (hints[h].r === r && hints[h].c === c) {
          sq.classList.add('hint');
        }
      }

      if (c === 0) {
        const rk = document.createElement('span');
        rk.className = 'coord-rank';
        rk.textContent = 8 - r;
        sq.appendChild(rk);
      }
      if (r === 7) {
        const fl = document.createElement('span');
        fl.className = 'coord-file';
        fl.textContent = files[c];
        sq.appendChild(fl);
      }

      const p = board[r][c];
      const isDragged = (drag && drag.r === r && drag.c === c);
      if (p !== '' && !isDragged) {
        const span = document.createElement('span');
        span.className = isW(p) ? 'piece-w' : 'piece-b';
        span.textContent = PIECES[p];
        sq.appendChild(span);
      }

      sq.addEventListener('pointerdown', function(e) {
        onDown(e, r, c);
      });
      el.appendChild(sq);
    }
  }
}

function onDown(e, r, c) {
  if (turn !== 'w') return;
  const p = board[r][c];

  if (sel) {
    let legal = false;
    for (let i = 0; i < hints.length; i++) {
      if (hints[i].r === r && hints[i].c === c) legal = true;
    }
    if (legal) {
      playTurn(sel.r, sel.c, r, c);
      return;
    }
  }

  if (isW(p)) {
    sel = {r: r, c: c};
    hints = getMoves(r, c, board);
    drag = {r: r, c: c};
    const ghost = document.getElementById('drag-ghost');
    ghost.textContent = PIECES[p];
    ghost.className = 'piece-w';
    ghost.style.left = e.clientX + 'px';
    ghost.style.top = e.clientY + 'px';
    ghost.style.display = 'block';
    draw();
  } else {
    sel = null;
    hints = [];
    draw();
  }
}

window.addEventListener('pointermove', function(e) {
  if (!drag) return;
  const ghost = document.getElementById('drag-ghost');
  ghost.style.left = e.clientX + 'px';
  ghost.style.top = e.clientY + 'px';
});

window.addEventListener('pointerup', function(e) {
  if (!drag) return;
  const ghost = document.getElementById('drag-ghost');
  ghost.style.display = 'none';
  const sr = drag.r;
  const sc = drag.c;
  drag = null;

  const tgt = document.elementFromPoint(e.clientX, e.clientY);
  if (tgt && tgt.classList.contains('sq')) {
    const tr = parseInt(tgt.dataset.r, 10);
    const tc = parseInt(tgt.dataset.c, 10);
    if (tr !== sr || tc !== sc) {
      let legal = false;
      for (let i = 0; i < hints.length; i++) {
        if (hints[i].r === tr && hints[i].c === tc) legal = true;
      }
      if (legal) {
        playTurn(sr, sc, tr, tc);
        return;
      }
    }
  }
  draw();
});

function playTurn(sr, sc, r, c) {
  movePiece(sr, sc, r, c);
  last = {sr: sr, sc: sc, r: r, c: c};
  sel = null;
  hints = [];
  turn = 'b';
  draw();
  document.getElementById('status').textContent = 'Соперник думает...';
  setTimeout(aiMove, 300);
}

function movePiece(sr, sc, r, c) {
  let p = board[sr][sc];
  if (p === 'P' && r === 0) p = 'Q';
  if (p === 'p' && r === 7) p = 'q';
  board[r][c] = p;
  board[sr][sc] = '';
}

function aiMove() {
  let all = [];
  for (let r = 0; r < 8; r++) {
    for (let c = 0; c < 8; c++) {
      if (isB(board[r][c])) {
        const list = getMoves(r, c, board);
        for (let i = 0; i < list.length; i++) {
          const nr = list[i].r;
          const nc = list[i].c;
          const tgt = board[nr][nc];
          const val = tgt !== '' ? VALS[tgt.toLowerCase()] : 0;
          const score = val + Math.random() * 2;
          all.push({sr: r, sc: c, r: nr, c: nc, score: score});
        }
      }
    }
  }
  if (all.length === 0) {
    document.getElementById('status').textContent = 'Партия окончена!';
    return;
  }
  all.sort(function(a, b) { return b.score - a.score; });
  const best = all[0];
  const cap = board[best.r][best.c];
  movePiece(best.sr, best.sc, best.r, best.c);
  last = {sr: best.sr, sc: best.sc, r: best.r, c: best.c};
  turn = 'w';
  draw();
  if (cap === 'K') {
