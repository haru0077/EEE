from IPython.display import HTML

# Final HTML for V9 without "최종" in title
html_content = """<!DOCTYPE html>
<html lang="ko">
<head>
  <meta charset="UTF-8">
  <title>원자 시뮬레이터 V9</title>
  <style>
    body { font-family: Arial; background: #f4f4f4; text-align: center; padding: 20px; }
    .container { display: flex; justify-content: space-around; }
    .sim { background: white; padding: 15px; border-radius: 10px; width: 45%; box-sizing: border-box; }
    .info { text-align: left; margin-top: 10px; }
    .info p { margin: 4px 0; }
    .orbital { text-align: left; margin-top: 10px; }
    .orb-line { margin: 6px 0; }
    .orb-cell { display: inline-block; width: 30px; height: 40px; margin: 2px; border: 1px solid #000; position: relative; }
    .up { position: absolute; top: 50%; left: 20%; transform: translate(-50%, -50%); font-size:18px; }
    .down { position: absolute; top: 50%; left: 80%; transform: translate(-50%, -50%); font-size:18px; }
    canvas { background: white; border: 1px solid #ccc; margin-top: 10px; display: block; margin-left: auto; margin-right: auto; }
  </style>
</head>
<body>
  <h2>🌟 원자 시뮬레이터 비교 V9 🌟</h2>
  <div class="container">
    <div class="sim">
      <h3>원소 A</h3>
      <label>원소 선택:
        <select id="sel1"></select>
      </label>
      <label><input type="checkbox" id="ion1"> 이온 모드</label>
      <canvas id="cv1" width="300" height="300"></canvas>
      <div class="info">
        <p>기호: <span id="symbol1">-</span></p>
        <p>전자배치: <span id="config1">-</span></p>
        <p>전하: <span id="charge1">-</span></p>
        <p>주기: <span id="period1">-</span></p>
        <p>족: <span id="group1">-</span></p>
        <p>반지름: <span id="radius1">-</span> pm</p>
      </div>
      <div class="orbital" id="orb1"><h4>오비탈</h4></div>
    </div>
    <div class="sim">
      <h3>원소 B</h3>
      <label>원소 선택:
        <select id="sel2"></select>
      </label>
      <label><input type="checkbox" id="ion2"> 이온 모드</label>
      <canvas id="cv2" width="300" height="300"></canvas>
      <div class="info">
        <p>기호: <span id="symbol2">-</span></p>
        <p>전자배치: <span id="config2">-</span></p>
        <p>전하: <span id="charge2">-</span></p>
        <p>주기: <span id="period2">-</span></p>
        <p>족: <span id="group2">-</span></p>
        <p>반지름: <span id="radius2">-</span> pm</p>
      </div>
      <div class="orbital" id="orb2"><h4>오비탈</h4></div>
    </div>
  </div>
  <button onclick="drawAll()">비교 보기</button>

  <script>
    const elems = [
      {"Z":1,"symbol":"H","orb":[1],"radius":53},
      {"Z":2,"symbol":"He","orb":[2],"radius":31},
      {"Z":3,"symbol":"Li","orb":[2,1],"radius":167},
      {"Z":4,"symbol":"Be","orb":[2,2],"radius":112},
      {"Z":5,"symbol":"B","orb":[2,2,1],"radius":87},
      {"Z":6,"symbol":"C","orb":[2,2,2],"radius":67},
      {"Z":7,"symbol":"N","orb":[2,2,3],"radius":56},
      {"Z":8,"symbol":"O","orb":[2,2,4],"radius":48},
      {"Z":9,"symbol":"F","orb":[2,2,5],"radius":42},
      {"Z":10,"symbol":"Ne","orb":[2,2,6],"radius":38},
      {"Z":11,"symbol":"Na","orb":[2,2,6,1],"radius":186},
      {"Z":12,"symbol":"Mg","orb":[2,2,6,2],"radius":160},
      {"Z":13,"symbol":"Al","orb":[2,2,6,2,1],"radius":143},
      {"Z":14,"symbol":"Si","orb":[2,2,6,2,2],"radius":118},
      {"Z":15,"symbol":"P","orb":[2,2,6,2,3],"radius":110},
      {"Z":16,"symbol":"S","orb":[2,2,6,2,4],"radius":104},
      {"Z":17,"symbol":"Cl","orb":[2,2,6,2,5],"radius":99},
      {"Z":18,"symbol":"Ar","orb":[2,2,6,2,6],"radius":71},
      {"Z":19,"symbol":"K","orb":[2,2,6,2,6,1],"radius":227},
      {"Z":20,"symbol":"Ca","orb":[2,2,6,2,6,2],"radius":197}
    ];
    const labels = ["1s","2s","2p","3s","3p","4s"];

    window.onload = () => {
      [1,2].forEach(i => {
        const sel = document.getElementById(`sel${i}`);
        elems.forEach(e => {
          const opt = document.createElement('option');
          opt.value = e.Z;
          opt.textContent = `${e.symbol} (${e.Z})`;
          sel.appendChild(opt);
        });
      });
    };

    function drawCanvas(idx, e0, e, charge) {
      const cv = document.getElementById(`cv${idx}`);
      const ctx = cv.getContext('2d');
      ctx.clearRect(0, 0, cv.width, cv.height);
      const base = 30, gap = 25;
      // Neutral orbits
      ctx.strokeStyle = "#ccc";
      e0.orb.forEach((_,i) => {
        ctx.beginPath();
        ctx.arc(cv.width/2, cv.height/2, base + i*gap, 0, 2*Math.PI);
        ctx.stroke();
      });
      // Actual orbits and electrons
      ctx.strokeStyle = "#000";
      e.orb.forEach((_,i) => {
        const factor = charge>0 ? 0.8 : (charge<0 ? 1.2 : 1);
        const r = (base + i*gap) * factor;
        ctx.beginPath();
        ctx.arc(cv.width/2, cv.height/2, r, 0, 2*Math.PI);
        ctx.stroke();
        for(let j=0; j<e.orb[i]; j++){
          const ang = 2*Math.PI*j/e.orb[i];
          const x = cv.width/2 + r*Math.cos(ang);
          const y = cv.height/2 + r*Math.sin(ang);
          ctx.beginPath();
          ctx.arc(x, y, 4, 0, 2*Math.PI);
          ctx.fill();
        }
      });
    }

    function drawSim(idx) {
      const Z = +document.getElementById(`sel${idx}`).value;
      const ion = document.getElementById(`ion${idx}`).checked;
      let e0 = elems.find(x => x.Z === Z);
      let e = JSON.parse(JSON.stringify(e0));
      let charge = 0;
      if (ion) {
        const last = e.orb.length - 1;
        if ([11,12,13,19,20].includes(Z)) { e.orb[last]--; charge = +1; }
        if ([9,16,17,8].includes(Z))       { e.orb[last]++; charge = -1; }
        if (Z === 1)                        { e.orb[0]--;  charge = +1; }
      }
      const period = e.orb.length;
      const group  = e.orb[e.orb.length - 1];
      const radius = e.radius;

      document.getElementById(`symbol${idx}`).innerText = e.symbol + (charge ? (charge>0? `+${charge}`:charge) : "");
      document.getElementById(`config${idx}`).innerHTML  = e.orb.map((v,i)=>v?`${labels[i]}<sup>${v}</sup>`:"").filter(x=>x).join(" ");
      document.getElementById(`charge${idx}`).innerText  = charge===0? "중성":(charge>0? `+${charge}`:charge);
      document.getElementById(`period${idx}`).innerText  = period;
      document.getElementById(`group${idx}`).innerText   = group;
      document.getElementById(`radius${idx}`).innerText  = radius;

      drawCanvas(idx, e0, e, charge);

      const orbDiv = document.getElementById(`orb${idx}`);
      orbDiv.innerHTML = "<h4>오비탈</h4>";
      e.orb.forEach((cnt,i) => {
        const line = document.createElement('div');
        line.className = 'orb-line';
        line.innerHTML = `<strong>${labels[i]}:</strong> `;
        const cells = labels[i].endsWith('p') ? 3 : 1;
        const arr = Array(cells).fill(0);
        for(let j=0;j<cnt;j++){
          arr[j<cells?j:j-cells]++;
        }
        arr.forEach(v=>{
          const cell = document.createElement('div');
          cell.className = 'orb-cell';
          if(v>=1){
            const up = document.createElement('div');
            up.className = 'up'; up.textContent = '↑';
            cell.appendChild(up);
          }
          if(v>=2){
            const down = document.createElement('div');
            down.className = 'down'; down.textContent = '↓';
            cell.appendChild(down);
          }
          line.appendChild(cell);
        });
        orbDiv.appendChild(line);
      });
    }

    function drawAll() {
      drawSim(1);
      drawSim(2);
    }
  </script>
</body>
</html>"""

path = "/mnt/data/원자_시뮬레이터_V9.html"
with open(path, "w", encoding="utf-8") as f:
    f.write(html_content)

HTML(f'<a href="sandbox:{path}" target="_blank">원자_시뮬레이터_V9.html 다운로드</a>')
