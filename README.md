# Kowallet-Home-Page
<!DOCTYPE html>
<html>
<head>
  <meta charset="UTF-8">
  <title>Token Launch Simulator</title>
  <style>
    body { font-family: Arial; max-width: 800px; margin: 40px auto; padding: 0 20px; background: #f4f4f4; text-align: center; }
    header { background: #2563eb; color: white; padding: 20px 0; margin-bottom: 20px; }
    .card { background: white; border-radius: 8px; padding: 15px; margin: 15px 0; border: 1px solid #ddd; }
    input { padding: 5px; margin: 5px; }
    button { padding: 8px 15px; margin-top: 10px; background: #10b981; color: white; border: none; border-radius: 5px; cursor: pointer; }
    h2 { margin-bottom: 10px; }
  </style>
</head>
<body>
  <header>
    <h1>Token Launch Simulator</h1>
    <div>Credits: <span id="credits">1000</span></div>
  </header>

  <div class="card">
    <h2>Create Token</h2>
    <input id="tokenName" placeholder="Token Name"/>
    <input id="tokenSym" placeholder="SYM" style="width:80px"/>
    <button id="createTokenBtn">Create Token (Cost: 10 credits)</button>
    <div id="createMsg" style="margin-top:10px;"></div>
  </div>

  <div class="card">
    <h2>Active Launches</h2>
    <div id="launches"></div>
  </div>

  <div class="card">
    <h2>Leaderboard (Recent Results)</h2>
    <div id="leaderboard"></div>
  </div>

  <script>
    // Load credits from localStorage or start at 1000
    let credits = Number(localStorage.getItem('credits') || 1000);
    document.getElementById('credits').innerText = credits;

    // Load launches and leaderboard from localStorage
    let launches = JSON.parse(localStorage.getItem('launches') || '[]');
    let leaderboard = JSON.parse(localStorage.getItem('leaderboard') || '[]');

    function saveState() {
      localStorage.setItem('credits', credits);
      localStorage.setItem('launches', JSON.stringify(launches));
      localStorage.setItem('leaderboard', JSON.stringify(leaderboard));
      document.getElementById('credits').innerText = credits;
    }

    function renderLaunches() {
      const container = document.getElementById('launches');
      container.innerHTML = '';
      launches.forEach((l, i) => {
        const div = document.createElement('div');
        div.innerHTML = `<strong>${l.name} (${l.sym})</strong> — Target: ${l.target} — Raised: ${l.raised} 
          <button data-index="${i}">Buy (10 credits)</button>`;
        container.appendChild(div);
      });

      // Add click events
      document.querySelectorAll('#launches button').forEach(btn => {
        btn.onclick = e => {
          const i = e.target.dataset.index;
          if (credits < 10) { alert('Not enough credits!'); return; }
          credits -= 10;
          launches[i].raised += 10;

          // Random chance of rug (12%) or success if target reached
          if (launches[i].raised >= launches[i].target) {
            resolveSuccess(i);
          } else if (Math.random() < 0.12) {
            resolveRug(i);
          }

          saveState();
          renderLaunches();
          renderLeaderboard();
        }
      });
    }

    function resolveSuccess(i) {
      const l = launches[i];
      leaderboard.push({token: l.sym, result: 'Moon!', profit: l.target});
      alert(`${l.sym} successfully launched! You earned ${l.target} credits.`);
      launches.splice(i,1);
      credits += l.target; // reward points
      saveState();
    }

    function resolveRug(i) {
      const l = launches[i];
      leaderboard.push({token: l.sym, result: 'Rug!', profit: -Math.floo
