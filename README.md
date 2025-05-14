![image](https://github.com/user-attachments/assets/1ad58a1d-7f49-463e-980d-0fc6d2b98354)# ChessCom

Attack-map.js - description

This small native JavaScript module adds a dynamic “attack map” to your chessboard, i.e. it highlights all fields that are potentially attacked by pieces of the selected side.


```js
;(function(){
  // ─── CONFIG ─────────────────────────────────────────────────────────────
  // Change this to "white" or "black"
  let opponentColor = "black";

  // ─── SETUP ──────────────────────────────────────────────────────────────
  if(window.attackMapInterval) {
    clearInterval(window.attackMapInterval);
    document.querySelectorAll('.attack-map-overlay').forEach(el=>el.remove());
  }

  // Helpers
  const oppPfx = () => opponentColor[0];                // "w" or "b"
  const sqCls  = f=>`square-${f[0]}${f[1]}`;
  const inB    = n=>n>=1 && n<=8;
  const knightD= [[1,2],[2,1],[2,-1],[1,-2],[-1,-2],[-2,-1],[-2,1],[-1,2]];
  const kingD  = [[1,0],[1,1],[0,1],[-1,1],[-1,0],[-1,-1],[0,-1],[1,-1]];
  const diagD  = [[1,1],[1,-1],[-1,1],[-1,-1]];
  const straightD = [[1,0],[-1,0],[0,1],[0,-1]];

  // Canvas for overlays
  const boardEl = document.querySelector('wc-chess-board.board');
  boardEl.style.position = 'relative';
  const overlayLayer = document.createElement('div');
  overlayLayer.style.cssText = `
    position:absolute; top:0; left:0; width:100%; height:100%;
    pointer-events:none; z-index:5;
  `;
  overlayLayer.classList.add('attack-map-overlay');
  boardEl.appendChild(overlayLayer);

  // Convert file/rank to %
  function posToPct([f,r]){
    // files a=1→left, so x = (f-1)*12.5%
    // ranks 1=bottom→y = (8-r)*12.5%
    return {
      x: (f-1)*12.5 + '%',
      y: (8-r)*12.5 + '%'
    };
  }

  // Compute all attack targets
  function computeTargets(){
    const pcs = Array.from(
      document.querySelectorAll(`.piece.${oppPfx()}p, .piece.${oppPfx()}r, .piece.${oppPfx()}n, .piece.${oppPfx()}b, .piece.${oppPfx()}q, .piece.${oppPfx()}k`)
    );
    const occ = {};
    document.querySelectorAll('.piece').forEach(p=>{
      p.classList.forEach(c=> c.startsWith('square-') && (occ[c]=true));
    });

    const hits = new Set();
    pcs.forEach(p=>{
      let [f,r] = [0,0];
      p.classList.forEach(c=>{
        const m = c.match(/^square-([1-8])([1-8])$/);
        if(m) [f,r] = [+m[1], +m[2]];
      });
      if(!f) return;
      const tag = p.classList;
      // knight
      if(tag.contains(`${oppPfx()}n`))
        knightD.forEach(d=>{ const t=[f+d[0],r+d[1]]; 
           inB(t[0])&&inB(t[1]) && hits.add(t+','+t)});
      // king
      if(tag.contains(`${oppPfx()}k`))
        kingD.forEach(d=>{ const t=[f+d[0],r+d[1]]; 
           inB(t[0])&&inB(t[1]) && hits.add(t+','+t)});
      // pawn
      if(tag.contains(`${oppPfx()}p`)){
        const dir = oppPfx()==='w'? 1:-1;
        [[1,dir],[-1,dir]].forEach(d=>{
          const t=[f+d[0],r+d[1]];
          inB(t[0])&&inB(t[1]) && hits.add(t+','+t);
        });
      }
      // bishop or queen (diagonals)
      if(tag.contains(`${oppPfx()}b`)||tag.contains(`${oppPfx()}q`))
        diagD.forEach(d=>{
          for(let i=1;i<=8;i++){
            const tf=f+d[0]*i, tr=r+d[1]*i;
            if(!inB(tf)||!inB(tr)) break;
            hits.add(tf+','+tr);
            // stop if blocked
            if(occ[ sqCls([tf,tr]) ]) break;
          }
        });
      // rook or queen (straights)
      if(tag.contains(`${oppPfx()}r`)||tag.contains(`${oppPfx()}q`))
        straightD.forEach(d=>{
          for(let i=1;i<=8;i++){
            const tf=f+d[0]*i, tr=r+d[1]*i;
            if(!inB(tf)||!inB(tr)) break;
            hits.add(tf+','+tr);
            if(occ[ sqCls([tf,tr]) ]) break;
          }
        });
    });
    return Array.from(hits).map(s=>s.split(',').map(Number));
  }

  // Render overlays
  function draw(){
    overlayLayer.innerHTML = '';
    computeTargets().forEach(rc=>{
      const {x,y} = posToPct(rc);
      const d = document.createElement('div');
      d.style.cssText = `
        position:absolute;
        width:12.5%; height:12.5%;
        left:${x}; top:${y};
        background:rgba(255,0,0,0.3);
      `;
      overlayLayer.appendChild(d);
    });
  }

  // Kick‑off interval
  window.attackMapInterval = setInterval(draw, 500);

  // Expose stop & reconfig functions
  window.stopAttackMap = ()=>{ 
    clearInterval(window.attackMapInterval);
    overlayLayer.remove();
    delete window.attackMapInterval;
  };
  window.setOpponentColor = c=>{
    if(c!=='white'&&c!=='black') throw 'Use "white" or "black"';
    opponentColor = c;
  };

  console.log(
    'Attack‑map running. Use setOpponentColor("white"/"black") to switch;\n',
    'stopAttackMap() to clear.'
  );
})();

```
![image](https://github.com/user-attachments/assets/2e419e2b-e3f5-4ff3-8ead-f3f502c20681)
