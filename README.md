<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>CTP/Obelisk Gambling Simulator</title>
  <style>
    body {
      font-family: sans-serif;
      text-align: center;
      background: #111;
      color: #fff;
    }

    button {
      padding: 10px 20px;
      margin: 10px;
      cursor: pointer;
    }

    .grid {
      display: flex;
      flex-wrap: wrap;
      justify-content: center;
      gap: 10px;
      margin-top: 20px;
    }

    .card {
      width: 120px;
      height: 160px;
      perspective: 1000px;
    }

    .inner {
      width: 100%;
      height: 100%;
      transition: transform 0.6s;
      transform-style: preserve-3d;
      position: relative;
    }

    .flipped .inner {
      transform: rotateY(180deg);
    }

    .front, .back {
      position: absolute;
      width: 100%;
      height: 100%;
      backface-visibility: hidden;
      border: 2px solid #333;
      border-radius: 8px;
    }

    .front {
      background: #444;
    }

    .back {
      background: #222;
      transform: rotateY(180deg);
    }

    .back img {
      width: 100%;
      height: 100%;
      object-fit: cover;
      border-radius: 8px;
    }

    #result, #history {
      margin: 20px auto;
      max-width: 90%;
    }
  </style>
</head>
<body>
  <h1>CTP / Obelisk Pull Simulator</h1>
  <p>Gems Spent: <span id="gem-count">0</span></p>
  <button onclick="pull(10)">Open 10x</button>
  <button onclick="clearHistory()">Clear History</button>

  <div id="result"></div>
  <div id="controls"></div>
  <h2>History</h2>
  <div id="history"></div>

  <script>
    const gemPerPull = 600;
    const gemDisplay = document.getElementById("gem-count");
    const resultDiv = document.getElementById("result");
    const historyDiv = document.getElementById("history");
    const controlsDiv = document.getElementById("controls");

    const items = [
      { name: "1★ Obelisk", chance: 59.5, img: "https://media.discordapp.net/attachments/1375554664795213910/1377043680828850206/Screenshot_20250528_005633_Future_Fight.jpg?ex=6837875d&is=683635dd&hm=6c952222f0c39daaf1f4fcd4d6772ac7f5c9563b6d974f0c2786e55fa1b1715f&=&format=webp" },
      { name: "2★ Obelisk", chance: 30, img: "https://media.discordapp.net/attachments/1375554664795213910/1377043518991630376/Screenshot_20250528_005705_Future_Fight.jpg?ex=68378737&is=683635b7&hm=4373707acc96f80beeeaf8b54e97344ca694459c092e063b93934edd7a219ee1&=&format=webp" },
      { name: "3★ Obelisk", chance: 5, img: "https://media.discordapp.net/attachments/1375554664795213910/1377043519247749231/Screenshot_20250528_005707_Future_Fight.jpg?ex=68378737&is=683635b7&hm=dcadcd91702de0c2099857d926c8d0adcafceb320f2870456c1395cf2afac3ff&=&format=webp" },
      { name: "4★ Obelisk", chance: 2.5, img: "https://media.discordapp.net/attachments/1375554664795213910/1377043519524307004/Screenshot_20250528_005709_Future_Fight.jpg?ex=68378737&is=683635b7&hm=f6ddb908e64b2a47c5d3a4a5d6ce0488d29a972a7ee536dc6bdbbfef58d6a664&=&format=webp" },
      { name: "5★ Obelisk", chance: 1.25, img: "https://media.discordapp.net/attachments/1375554664795213910/1377043519725768816/Screenshot_20250528_005710_Future_Fight.jpg?ex=68378737&is=683635b7&hm=a6b5f4011f9809818743315a542943227318f79713db0d1a93a06a00d500dde8&=&format=webp" },
      { name: "6★ Obelisk", chance: 0.25, img: "https://media.discordapp.net/attachments/1375554664795213910/1377043519939674184/Screenshot_20250528_005712_Future_Fight.jpg?ex=68378737&is=683635b7&hm=3f76132a0bdbc643e115b6e8d35e00b514e08e6a2640ecf5c320a88fdb145098&=&format=webp" },
      { name: "CTP Of Authority", chance: 0.05, img: "https://thanosvibs.money/static/assets/items/ctp_authority.png" },
      { name: "CTP Of Energy", chance: 0.05, img: "https://thanosvibs.money/static/assets/items/ctp_energy.png" },
      { name: "CTP Of Destruction", chance: 0.05, img: "https://thanosvibs.money/static/assets/items/ctp_destruction.png" },
      { name: "CTP Of Refinement", chance: 0.05, img: "https://thanosvibs.money/static/assets/items/ctp_refinement.png" },
      { name: "CTP Of Regeneration", chance: 0.05, img: "https://thanosvibs.money/static/assets/items/ctp_regeneration.png" },
      { name: "CTP Of Rage", chance: 0.05, img: "https://thanosvibs.money/static/assets/items/ctp_rage.png" },
      { name: "CTP Of Judgement", chance: 0.05, img: "https://thanosvibs.money/static/assets/items/ctp_judgement.png" },
      { name: "CTP Of Greed", chance: 0.05, img: "https://thanosvibs.money/static/assets/items/ctp_greed.png" },
      { name: "CTP Of Insight", chance: 0.05, img: "https://thanosvibs.money/static/assets/items/ctp_insight.png" },
      { name: "CTP Of Conquest", chance: 0.05, img: "https://thanosvibs.money/static/assets/items/ctp_conquest.png" },
      { name: "CTP Of Liberation", chance: 0.05, img: "https://thanosvibs.money/static/assets/items/ctp_liberation.png" },
      { name: "CTP Of Patience", chance: 0.05, img: "https://thanosvibs.money/static/assets/items/ctp_patience.png" },
      { name: "CTP Of Transcendence", chance: 0.05, img: "https://thanosvibs.money/static/assets/items/ctp_transcendence.png" }
    ];

    let gemSpent = parseInt(localStorage.getItem("gems")) || 0;
    let history = JSON.parse(localStorage.getItem("history") || "[]");

    function updateUI() {
      gemDisplay.textContent = gemSpent;
      historyDiv.innerHTML = history.map(h => `<div>${h}</div>`).join("");
      localStorage.setItem("gems", gemSpent);
      localStorage.setItem("history", JSON.stringify(history));
    }

    function getRandomItem() {
      const rand = Math.random() * 100;
      let sum = 0;
      for (const item of items) {
        sum += item.chance;
        if (rand <= sum) return item;
      }
      return items[0];
    }

    function pull(count) {
      let pulls = [];
      for (let i = 0; i < count; i++) pulls.push(getRandomItem());

      // Guarantee at least one 5★ or higher
      if (!pulls.some(p => p.name.includes("5★") || p.name.includes("6★") || p.name.includes("CTP"))) {
        pulls[Math.floor(Math.random() * count)] = items.find(i => i.name === "5★ Obelisk");
      }

      gemSpent += gemPerPull * count;
      updateUI();

      resultDiv.innerHTML = `<div class="grid">${pulls.map((item, i) => `
        <div class="card" id="card-${i}">
          <div class="inner">
            <div class="front"></div>
            <div class="back"><img src="${item.img}" alt="${item.name}"></div>
          </div>
        </div>
      `).join("")}</div>`;

      controlsDiv.innerHTML = `
        <button onclick="skipReveal()">Skip</button>
        <button onclick="closeResult()" style="display:none" id="closeBtn">Close</button>
      `;

      // Flip animation
      let i = 0;
      const interval = setInterval(() => {
        const el = document.getElementById(`card-${i}`);
        if (el) el.classList.add("flipped");
        i++;
        if (i >= pulls.length) {
          clearInterval(interval);
          document.getElementById("closeBtn").style.display = "inline-block";
        }
      }, 300);

      history.push("Pull: " + pulls.map(p => p.name).join(", "));
      updateUI();
    }

    function skipReveal() {
      for (let i = 0; i < 10; i++) {
        const el = document.getElementById(`card-${i}`);
        if (el) el.classList.add("flipped");
      }
      document.getElementById("closeBtn").style.display = "inline-block";
    }

    function closeResult() {
      resultDiv.innerHTML = "";
      controlsDiv.innerHTML = "";
    }

    function clearHistory() {
      if (confirm("Clear all history and gem count?")) {
        localStorage.clear();
        gemSpent = 0;
        history = [];
        updateUI();
        resultDiv.innerHTML = "";
        controlsDiv.innerHTML = "";
      }
    }

    updateUI();
  </script>
</body>
</html>
