<!DOCTYPE html>
<html lang="ru">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <title>Improved Monopoly</title>
  <style>
    :root {
      --bg1: #dbeafe;
      --bg2: #ecfeff;
      --panel: #f8fafc;
      --panel-border: #cbd5e1;
      --text: #0f172a;
      --blue: #2563eb;
      --green: #10b981;
      --red: #ef4444;
      --yellow: #facc15;
      --purple: #8b5cf6;
      --board-green: #4ade80;
      --board-border: #15803d;
    }

    * { box-sizing: border-box; }
    html, body { margin: 0; height: 100%; }
    body {
      font-family: "Segoe UI", Tahoma, sans-serif;
      background: linear-gradient(135deg, var(--bg1), var(--bg2));
      color: var(--text);
      display: flex;
      align-items: center;
      justify-content: center;
      min-height: 100vh;
      padding: 16px;
    }

    .game {
      width: min(1400px, 100%);
      height: min(900px, 92vh);
      display: grid;
      grid-template-columns: 860px 440px;
      background: white;
      border: 1px solid #dbe2ea;
      border-radius: 18px;
      overflow: hidden;
      box-shadow: 0 18px 45px rgba(15,23,42,.15);
    }

    .board-wrap {
      background: linear-gradient(135deg, #dcfce7, #dbeafe);
      padding: 10px;
      display: flex;
      align-items: center;
      justify-content: center;
      position: relative;
    }

    .board {
      position: relative;
      width: 100%;
      height: 100%;
      max-width: 820px;
      max-height: 820px;
      background: var(--board-green);
      border: 6px solid var(--board-border);
      display: grid;
      grid-template-columns: repeat(11, 1fr);
      grid-template-rows: repeat(11, 1fr);
      gap: 2px;
      overflow: hidden;
    }

    .cell {
      position: relative;
      background: #f8fafc;
      border: 1px solid #cbd5e1;
      display: flex;
      flex-direction: column;
      justify-content: space-between;
      align-items: center;
      padding: 4px 2px;
      font-weight: 700;
      font-size: 9px;
      text-align: center;
      overflow: hidden;
      user-select: none;
      min-height: 66px;
    }

    .cell .name {
      line-height: 1.1;
      max-width: 100%;
      padding: 0 3px;
    }

    .cell .price {
      font-size: 8px;
      color: #334155;
    }

    .cell .owner {
      font-size: 7px;
      color: #475569;
      min-height: 12px;
      margin-top: 2px;
    }

    .cell .bar {
      width: 100%;
      height: 12px;
      border-bottom: 1px solid #dbe2ea;
      margin-bottom: 2px;
      background: transparent;
    }

    .cell.special {
      background: #e2e8f0;
      color: #0f172a;
    }

    .special .bar { display: none; }

    .board-center {
      grid-column: 2 / 11;
      grid-row: 2 / 11;
      background: #f8fafc;
      display: flex;
      flex-direction: column;
      align-items: center;
      justify-content: center;
      gap: 18px;
      padding: 14px;
      border: 2px solid #e2e8f0;
      z-index: 0;
    }

    .logo {
      font-size: 32px;
      letter-spacing: 2px;
      font-weight: 900;
      color: #1e293b;
      text-transform: uppercase;
      border-bottom: 5px solid var(--red);
      padding-bottom: 8px;
    }

    .dice-box {
      display: flex;
      gap: 18px;
      align-items: center;
    }

    .die {
      width: 54px;
      height: 54px;
      border-radius: 10px;
      border: 2px solid #334155;
      background: white;
      display: flex;
      align-items: center;
      justify-content: center;
      font-size: 26px;
      font-weight: 900;
      box-shadow: 0 6px 18px rgba(17,24,39,.12);
      transition: transform .25s ease, box-shadow .25s ease;
    }

    .die.rolling {
      transform: rotate(20deg) scale(1.05);
      box-shadow: 0 12px 26px rgba(17,24,39,.2);
    }

    .cell-token-layer {
      position: absolute;
      inset: 0;
      pointer-events: none;
      z-index: 5;
    }

    .token {
      position: absolute;
      width: 16px;
      height: 16px;
      border-radius: 50%;
      border: 2px solid rgba(255,255,255,.9);
      box-shadow: 0 3px 10px rgba(15,23,42,.3);
      transition: transform .35s ease, left .35s ease, top .35s ease;
    }

    .token.red { background: #ef4444; }
    .token.blue { background: #2563eb; }
    .token.yellow { background: #facc15; }
    .token.green { background: #22c55e; }

    .side {
      background: var(--panel);
      border-left: 1px solid var(--panel-border);
      padding: 18px 16px;
      display: flex;
      flex-direction: column;
      gap: 16px;
    }

    .status {
      display: grid;
      grid-template-columns: repeat(2, minmax(0,1fr));
      gap: 10px;
    }

    .player-card {
      padding: 10px 12px;
      border-radius: 10px;
      border: 2px solid #dfe7ee;
      background: #fff;
      font-size: 12px;
      line-height: 1.5;
      transition: .2s;
    }

    .player-card.active {
      border-color: #ef4444;
      background: #fff1f2;
      box-shadow: 0 6px 18px rgba(239,68,68,.12);
    }

    .player-card.bankrupt {
      opacity: .45;
      text-decoration: line-through;
      background: #e2e8f0;
    }

    .controls {
      display: grid;
      grid-template-columns: 1fr;
      gap: 10px;
    }

    button {
      appearance: none;
      border: none;
      border-radius: 10px;
      padding: 12px 14px;
      background: var(--blue);
      color: white;
      font-weight: 700;
      cursor: pointer;
      font-size: 15px;
      transition: transform .15s ease, filter .15s ease;
    }

    button:hover { filter: brightness(1.05); }
    button:active { transform: translateY(1px); }

    button.secondary { background: #0f172a; }
    button.warning { background: #ef4444; }
    button.success { background: #10b981; }

    button:disabled {
      background: #cbd5e1;
      cursor: not-allowed;
      color: #64748b;
    }

    .info-box {
      background: #0f172a;
      border-radius: 10px;
      color: #7dd3fc;
      padding: 12px;
      min-height: 180px;
      max-height: 280px;
      overflow: auto;
      font-family: Consolas, monospace;
      font-size: 11px;
      line-height: 1.55;
    }

    .info-box .msg { margin-bottom: 4px; }
    .info-box .msg strong { color: #f8fafc; }

    .selected {
      background: #dbeafe;
      border: 1px solid #93c5fd;
      border-radius: 10px;
      padding: 10px 12px;
      font-size: 12px;
      min-height: 56px;
      display: flex;
      align-items: center;
    }

    @media (max-width:1100px) {
      .game {
        grid-template-columns: 1fr;
        height: auto;
      }
      .board-wrap { order: 1; }
      .side { order: 2; }
    }
  </style>
</head>
<body>
  <div class="game">
    <div class="board-wrap">
      <div id="board" class="board"></div>
    </div>

    <div class="side">
      <h3 style="margin:0;color:#0f172a;">Статус игроков</h3>
      <div id="status" class="status"></div>

      <div class="controls">
        <button id="rollBtn">Бросить кубики</button>
        <button id="buyBtn" class="success" disabled>Купить</button>
        <button id="auctionBtn" class="secondary">Аукцион</button>
        <button id="mortgageBtn">Залог</button>
        <button id="tradeBtn" class="warning">Обмен</button>
        <button id="endTurnBtn" class="secondary" disabled>Завершить ход</button>
      </div>

      <div class="selected" id="selectedCell">Выберите клетку на поле.</div>
      <div class="info-box" id="logBox"></div>
    </div>
  </div>

  <script>
    const boardData = [
      { name: "СТАРТ", type: "start", price: 0, color: "#fff", group: "special" },
      { name: "Средиземное авеню", type: "property", price: 60, rent: 2, color: "#955436", group: "brown" },
      { name: "Общественная казна", type: "chance", price: 0, color: "#fff", group: "special" },
      { name: "Балтийское авеню", type: "property", price: 60, rent: 4, color: "#955436", group: "brown" },
      { name: "Подоходный налог", type: "tax", price: 50, color: "#fff", group: "special" },
      { name: "Железная дорога Чтение", type: "railroad", price: 200, rent: 25, color: "#9ca3af", group: "railroad" },
      { name: "Восточное авеню", type: "property", price: 100, rent: 6, color: "#00a2e8", group: "cyan" },
      { name: "Шанс", type: "chance", price: 0, color: "#fff", group: "special" },
      { name: "Коммунальное авеню", type: "property", price: 100, rent: 6, color: "#00a2e8", group: "cyan" },
      { name: "Коннектикут авеню", type: "property", price: 120, rent: 8, color: "#00a2e8", group: "cyan" },
      { name: "ТЮРЬМА", type: "jail", price: 0, color: "#fff", group: "special" },
      { name: "Святой Чарльз Плейс", type: "property", price: 140, rent: 10, color: "#c9328a", group: "pink" },
      { name: "Электрическая компания", type: "utility", price: 150, rent: 10, color: "#9ca3af", group: "utility" },
      { name: "Штаты авеню", type: "property", price: 140, rent: 10, color: "#c9328a", group: "pink" },
      { name: "Вирджиния авеню", type: "property", price: 160, rent: 12, color: "#c9328a", group: "pink" },
      { name: "Пенсильванская ж/д", type: "railroad", price: 200, rent: 25, color: "#9ca3af", group: "railroad" },
      { name: "Сан-Джеймс Плейс", type: "property", price: 180, rent: 14, color: "#ff7f27", group: "orange" },
      { name: "Общественная казна", type: "chance", price: 0, color: "#fff", group: "special" },
      { name: "Теннесси авеню", type: "property", price: 180, rent: 14, color: "#ff7f27", group: "orange" },
      { name: "Нью-Йорк авеню", type: "property", price: 200, rent: 16, color: "#ff7f27", group: "orange" },
      { name: "Бесплатная стоянка", type: "freeparking", price: 0, color: "#fff", group: "special" },
      { name: "Кентукки авеню", type: "property", price: 220, rent: 18, color: "#ed1c24", group: "red" },
      { name: "Шанс", type: "chance", price: 0, color: "#fff", group: "special" },
      { name: "Индиана авеню", type: "property", price: 220, rent: 18, color: "#ed1c24", group: "red" },
      { name: "Иллинойс авеню", type: "property", price: 240, rent: 20, color: "#ed1c24", group: "red" },
      { name: "Ж/д B&O", type: "railroad", price: 200, rent: 25, color: "#9ca3af", group: "railroad" },
      { name: "Атлантик авеню", type: "property", price: 260, rent: 22, color: "#fff200", group: "yellow" },
      { name: "Вентнор авеню", type: "property", price: 260, rent: 22, color: "#fff200", group: "yellow" },
      { name: "Водопровод", type: "utility", price: 150, rent: 10, color: "#9ca3af", group: "utility" },
      { name: "Марвин Гарденс", type: "property", price: 280, rent: 24, color: "#fff200", group: "yellow" },
      { name: "ИДИ В ТЮРЬМУ", type: "gotojail", price: 0, color: "#fff", group: "special" },
      { name: "Тихоокеанское авеню", type: "property", price: 300, rent: 26, color: "#22b14c", group: "green" },
      { name: "Северная Каролина", type: "property", price: 300, rent: 26, color: "#22b14c", group: "green" },
      { name: "Общественная казна", type: "chance", price: 0, color: "#fff", group: "special" },
      { name: "Пенсильвания авеню", type: "property", price: 320, rent: 28, color: "#22b14c", group: "green" },
      { name: "Короткая линия", type: "railroad", price: 200, rent: 25, color: "#9ca3af", group: "railroad" },
      { name: "Шанс", type: "chance", price: 0, color: "#fff", group: "special" },
      { name: "Парк Плейс", type: "property", price: 350, rent: 35, color: "#3f48cc", group: "blue" },
      { name: "Сверхналог", type: "tax", price: 100, color: "#fff", group: "special" },
      { name: "Бродвей", type: "property", price: 400, rent: 50, color: "#3f48cc", group: "blue" }
    ];

    const players = [
      { id: 0, name: "Красный", color: "red", money: 1500, position: 0, inJail: false, bankrupt: false, ai: false, properties: [] },
      { id: 1, name: "Синий", color: "blue", money: 1500, position: 0, inJail: false, bankrupt: false, ai: true, properties: [] },
      { id: 2, name: "Желтый", color: "yellow", money: 1500, position: 0, inJail: false, bankrupt: false, ai: true, properties: [] },
      { id: 3, name: "Зеленый", color: "green", money: 1500, position: 0, inJail: false, bankrupt: false, ai: true, properties: [] }
    ];

    const board = boardData.map((cell, idx) => ({
      ...cell,
      id: idx,
      owner: null,
      mortgaged: false,
      houses: 0
    }));

    const state = {
      current: 0,
      gameOver: false,
      waitForAction: false,
      lastRoll: [1, 1],
      log: []
    };

    const boardEl = document.getElementById("board");
    const statusEl = document.getElementById("status");
    const logBox = document.getElementById("logBox");
    const selectedCellEl = document.getElementById("selectedCell");

    function log(msg) {
      state.log.push(msg);
      if (state.log.length > 80) state.log.shift();
      logBox.innerHTML = state.log.map(m => `<div class="msg">${m}</div>`).join("");
      logBox.scrollTop = logBox.scrollHeight;
    }

    function buildBoard() {
      boardEl.innerHTML = "";

      const center = document.createElement("div");
      center.className = "board-center";
      center.innerHTML = `
        <div class="logo">Монополия</div>
        <div class="dice-box">
          <div id="die1" class="die">1</div>
          <div id="die2" class="die">1</div>
        </div>
      `;
      boardEl.appendChild(center);

      for (let i = 0; i < board.length; i++) {
        const cell = board[i];
        const div = document.createElement("div");
        div.className = "cell " + (["property","railroad","utility"].includes(cell.type) ? "" : "special") + " cell-" + i;
        div.dataset.index = String(i);

        if (cell.color && cell.color !== "#fff") {
          const bar = document.createElement("div");
          bar.className = "bar";
          bar.style.background = cell.color;
          div.appendChild(bar);
        }

        const name = document.createElement("div");
        name.className = "name";
        name.textContent = cell.name;
        div.appendChild(name);

        if (cell.price > 0) {
          const price = document.createElement("div");
          price.className = "price";
          price.textContent = `$${cell.price}`;
          div.appendChild(price);
        }

        const owner = document.createElement("div");
        owner.className = "owner";
        owner.id = "owner-" + i;
        div.appendChild(owner);

        const layer = document.createElement("div");
        layer.className = "cell-token-layer";
        layer.id = "tokens-" + i;
        div.appendChild(layer);

        div.addEventListener("click", () => {
          selectedCellEl.textContent = `Клетка: ${cell.name} — ${cell.type} — цена ${cell.price ? "$" + cell.price : "—"}`;
        });

        boardEl.appendChild(div);
      }
    }

    function renderBoardTokens() {
      for (let i = 0; i < board.length; i++) {
        const layer = document.getElementById("tokens-" + i);
        if (!layer) continue;
        layer.innerHTML = "";
        players.forEach((player, pIndex) => {
          if (player.position !== i || player.bankrupt) return;

          const t = document.createElement("div");
          t.className = "token " + player.color;
          const offsetX = 8 + (pIndex % 2) * 7 + (pIndex >= 2 ? 3 : 0);
          const offsetY = 8 + Math.floor(pIndex / 2) * 7;
          t.style.left = offsetX + "px";
          t.style.top = offsetY + "px";
          t.title = player.name;
          layer.appendChild(t);
        });
      }
    }

    function renderOwners() {
      board.forEach((cell, index) => {
        const el = document.getElementById("owner-" + index);
        if (!el) return;
        if (cell.owner !== null) {
          el.textContent = "Владелец: " + players[cell.owner].name;
        } else {
          el.textContent = cell.mortgaged ? "Заложено" : "";
        }
      });
    }

    function renderStatus() {
      statusEl.innerHTML = "";
      players.forEach((player, idx) => {
        const card = document.createElement("div");
        const active = idx === state.current && !state.gameOver;
        card.className = "player-card" + (active ? " active" : "") + (player.bankrupt ? " bankrupt" : "");
        const jailTxt = player.inJail ? " (тюрьма)" : "";
        const props = player.properties.length ? "Недвижимость: " + player.properties.length : "Недвижимость: 0";
        card.innerHTML = `
          <strong>${player.name}</strong><br>
          Деньги: $${player.money}<br>
          Положение: ${player.position}<br>
          ${props}${jailTxt}
        `;
        statusEl.appendChild(card);
      });
    }

    function updateDice(a, b) {
      document.getElementById("die1").textContent = a;
      document.getElementById("die2").textContent = b;
    }

    function getCurrentPlayer() {
      return players[state.current];
    }

    function nextAlivePlayer(indexStart = state.current) {
      for (let i = 0; i < players.length; i++) {
        const p = players[(indexStart + i) % players.length];
        if (!p.bankrupt) return p.id;
      }
      return state.current;
    }

    function setButtons() {
      const p = getCurrentPlayer();
      const canAct = !state.gameOver && !p.bankrupt && !state.waitForAction;
      const hasBuy = canBuyCurrentProperty();
      const canMortgage = playerHasMortgagableProperty(p);

      document.getElementById("rollBtn").disabled = !canAct || (p.inJail && !p.ai);
      document.getElementById("buyBtn").disabled = !canAct || !hasBuy;
      document.getElementById("auctionBtn").disabled = !canAct;
      document.getElementById("mortgageBtn").disabled = !canAct || !canMortgage;
      document.getElementById("tradeBtn").disabled = !canAct || p.ai;
      document.getElementById("endTurnBtn").disabled = !(canAct && state.waitForAction);
    }

    function playerHasMortgagableProperty(player) {
      return player.properties.some(index => board[index].owner === player.id && !board[index].mortgaged);
    }

    function randomDie() {
      return Math.floor(Math.random() * 6) + 1;
    }

    function checkWin() {
      const alive = players.filter(p => !p.bankrupt);
      if (alive.length <= 1) {
        state.gameOver = true;
        log(`🎉 Игра окончена! Победитель: <strong>${alive[0]?.name || "Никто"}</strong>`);
        document.getElementById("rollBtn").disabled = true;
        document.getElementById("buyBtn").disabled = true;
        document.getElementById("auctionBtn").disabled = true;
        document.getElementById("mortgageBtn").disabled = true;
        document.getElementById("tradeBtn").disabled = true;
        document.getElementById("endTurnBtn").disabled = true;
      }
    }

    function bankruptPlayer(player) {
      player.bankrupt = true;
      player.properties.forEach(index => {
        board[index].owner = null;
        board[index].mortgaged = false;
      });
      player.properties = [];
      player.money = 0;
      renderStatus();
      renderOwners();
      renderBoardTokens();
      checkWin();
    }

    function canBuyCurrentProperty() {
      const p = getCurrentPlayer();
      const cell = board[p.position];
      if (!["property","railroad","utility"].includes(cell.type)) return false;
      return cell.owner === null && p.money >= cell.price;
    }

    function buyCurrentProperty() {
      const p = getCurrentPlayer();
      const cell = board[p.position];
      if (!canBuyCurrentProperty()) return;

      p.money -= cell.price;
      cell.owner = p.id;
      p.properties.push(p.position);
      state.waitForAction = false;

      log(`🏠 ${p.name} купил ${cell.name} за $${cell.price}`);
      renderStatus();
      renderOwners();
      renderBoardTokens();
      setButtons();
    }

    function payRent(player, cell) {
      const owner = players[cell.owner];
      if (!owner || owner.id === player.id) return;

      let rent = cell.rent ?? 0;

      if (cell.type === "utility") rent = rent * 2;
      if (cell.type === "railroad") rent = rent * 1;

      const sameGroup = board.filter(item => item.group === cell.group && item.owner === cell.owner).length;
      if (sameGroup >= 2 && ["property", "railroad", "utility"].includes(cell.type)) {
        rent *= 2;
      }

      if (player.money >= rent) {
        player.money -= rent;
        owner.money += rent;
        log(`💸 ${player.name} платит аренду ${owner.name}: $${rent}`);
      } else {
        owner.money += player.money;
        player.money = 0;
        log(`💀 ${player.name} не может оплатить аренду и банкротится!`);
        bankruptPlayer(player);
      }
    }

    function resolveCellEffect(player) {
      const cell = board[player.position];

      if (cell.type === "tax") {
        if (player.money >= cell.price) {
          player.money -= cell.price;
          log(`💸 ${player.name} уплатил налог $${cell.price}`);
        } else {
          log(`💀 ${player.name} не может оплатить налог и банкротится`);
          bankruptPlayer(player);
          return;
        }
      }

      if (cell.type === "chance") {
        const events = [
          { text: "Случайная награда: +100", money: 100 },
          { text: "Штраф: -75", money: -75 },
          { text: "Переместились на старт: +200", money: 200, moveTo: 0 },
          { text: "Иди в тюрьму", moveTo: 10, jail: true }
        ];
        const e = events[Math.floor(Math.random() * events.length)];
        log(`🎴 ${player.name}: ${e.text}`);

        if (e.money > 0) player.money += e.money;
        if (e.money < 0) player.money += e.money;

        if (e.moveTo !== undefined) {
          player.position = e.moveTo;
          if (e.jail) {
            player.inJail = true;
            log(`🚓 ${player.name} отправляется в тюрьму`);
          } else {
            player.money += 200;
            log(`💰 ${player.name} получил $200 за старт`);
          }
        }
      }

      if (cell.type === "gotojail") {
        player.position = 10;
        player.inJail = true;
        log(`🚓 ${player.name} отправляется в тюрьму`);
      }

      if (["property", "railroad", "utility"].includes(cell.type)) {
        if (cell.owner === null) {
          if (player.money >= cell.price) {
            if (player.ai) {
              player.money -= cell.price;
              cell.owner = player.id;
              player.properties.push(player.position);
              log(`🤖 ${player.name} автоматически купил ${cell.name} за $${cell.price}`);
            } else {
              state.waitForAction = true;
              log(`📌 ${player.name} может купить ${cell.name} за $${cell.price}`);
            }
          } else {
            log(`⚠️ ${player.name} не может купить ${cell.name}: недостаточно денег`);
          }
        } else if (cell.owner !== player.id) {
          payRent(player, cell);
        }
      }

      renderStatus();
      renderOwners();
      renderBoardTokens();
      setButtons();
    }

    function processMove(player, totalSteps) {
      const start = player.position;
      const end = (start + totalSteps) % 40;

      if (end < start) {
        player.money += 200;
        log(`💰 ${player.name} прошёл старт и получил $200`);
      }

      player.position = end;
      renderBoardTokens();

      setTimeout(() => {
        if (state.gameOver) return;
        resolveCellEffect(player);
      }, 250);
    }

    function endTurn() {
      if (state.gameOver) return;

      state.waitForAction = false;
      let next = (state.current + 1) % players.length;
      let attempts = 0;

      while (players[next].bankrupt && attempts < 10) {
        next = (next + 1) % players.length;
        attempts++;
      }

      state.current = next;
      log(`➡️ Ход переходит к ${players[state.current].name}`);
      renderStatus();
      setButtons();

      checkWin();

      if (!state.gameOver && players[state.current].ai) {
        setTimeout(() => {
          if (!state.gameOver) {
            doAITurn(players[state.current]);
          }
        }, 600);
      }
    }

    function doRoll(player, d1, d2) {
      const total = d1 + d2;
      state.lastRoll = [d1, d2];
      updateDice(d1, d2);
      log(`🎲 ${player.name} бросил ${d1} и ${d2} (всего ${total})`);

      if (player.inJail) {
        if (d1 === d2) {
          player.inJail = false;
          log(`🔓 ${player.name} выбросил дубль и вышел из тюрьмы`);
        } else {
          log(`🚫 ${player.name} не выбросил дубль и остаётся в тюрьме`);
          endTurn();
          return;
        }
      }

      state.waitForAction = false;
      processMove(player, total);
      renderStatus();
      setButtons();
    }

    function rollTurn() {
      if (state.gameOver) return;
      const p = getCurrentPlayer();
      if (p.bankrupt) return;

      if (p.ai) {
        doAITurn(p);
        return;
      }

      const d1 = randomDie();
      const d2 = randomDie();
      doRoll(p, d1, d2);
    }

    function doAITurn(player) {
      if (state.gameOver || player.bankrupt) return;

      const d1 = randomDie();
      const d2 = randomDie();
      doRoll(player, d1, d2);
    }

    function startAuction() {
      if (state.gameOver) return;
      const p = getCurrentPlayer();
      const cell = board[p.position];

      if (cell.owner !== null) {
        log("⚠️ Нельзя начать аукцион: клетка уже принадлежит кому-то.");
        return;
      }

      const participants = players.filter(pl => !pl.bankrupt && pl.money > 0);

      let highestBid = { playerId: p.id, amount: 0 };
      participants.forEach(pl => {
        if (pl.ai) {
          const aiOffer = Math.max(10, Math.floor(Math.random() * (pl.money * 0.35)));
          if (aiOffer > highestBid.amount) {
            highestBid = { playerId: pl.id, amount: aiOffer };
          }
        }
      });

      const humanInput = window.prompt(`Аукцион: ${cell.name}\nВведите ставку (минимум 10):`, String(Math.max(10, highestBid.amount + 10)));
      const humanBid = Number(humanInput);
      if (!Number.isNaN(humanBid) && humanBid >= 10 && humanBid > highestBid.amount) {
        highestBid = { playerId: p.id, amount: humanBid };
      }

      if (highestBid.amount <= 0) {
        log("❌ Аукцион закончился без ставок.");
        return;
      }

      const winner = players[highestBid.playerId];
      winner.money -= highestBid.amount;
      cell.owner = winner.id;
      winner.properties.push(winner.position);

      log(`🏷️ Аукцион выигран ${winner.name} за $${highestBid.amount}`);
      renderStatus();
      renderOwners();
      renderBoardTokens();
      setButtons();
    }

    function mortgageSelectedProperty() {
      const p = getCurrentPlayer();
      const available = p.properties.filter(index => board[index].owner === p.id && !board[index].mortgaged);

      if (!available.length) {
        log("🚫 Нет доступных залогов.");
        return;
      }

      const chosen = Number(window.prompt(
        `Выберите клетку для залога: ${available.map(i => `${i}:${board[i].name}`).join(" | ")}`,
        String(available[0])
      ));

      if (!Number.isInteger(chosen) || !available.includes(chosen)) {
        log("⚠️ Неверный выбор залога.");
        return;
      }

      const cell = board[chosen];
      const amount = Math.floor(cell.price / 2);
      p.money += amount;
      cell.mortgaged = true;

      log(`🏦 ${p.name} заложил ${cell.name} и получил $${amount}`);
      renderStatus();
      renderOwners();
      setButtons();
    }

    function tradeProperty() {
      const p = getCurrentPlayer();
      if (p.ai) return;

      if (!p.properties.length) {
        log("У вас нет собственности для обмена.");
        return;
      }

      const targetPlayerId = Number(window.prompt("С кем меняем? 1=Синий, 2=Желтый, 3=Зеленый", "1"));
      const target = players[targetPlayerId];
      if (!target || target.id === p.id || target.bankrupt) {
        log("Неверный игрок для обмена.");
        return;
      }

      const ownIndexes = p.properties;
      const propIndex = Number(window.prompt("Какую клетку передать? " + ownIndexes.join(","), String(ownIndexes[0])));
      if (!ownIndexes.includes(propIndex)) {
        log("Это не ваша собственность.");
        return;
      }

      const targetOwn = target.properties;
      if (!targetOwn.length) {
        log(`${target.name} ничего не может отдать в обмен.`);
        return;
      }

      const chooseTargetProp = Number(window.prompt("Какую клетку получить от " + target.name + "? " + targetOwn.join(","), String(targetOwn[0])));
      if (!targetOwn.includes(chooseTargetProp)) {
        log("Цель обмена не принадлежит целевому игроку.");
        return;
      }

      const indexInP = p.properties.indexOf(propIndex);
      const indexInT = target.properties.indexOf(chooseTargetProp);

      p.properties.splice(indexInP, 1);
      target.properties.splice(indexInT, 1);

      p.properties.push(chooseTargetProp);
      target.properties.push(propIndex);

      board[propIndex].owner = target.id;
      board[chooseTargetProp].owner = p.id;

      log(`🤝 ${p.name} обменял ${board[propIndex].name} на ${board[chooseTargetProp].name} с ${target.name}`);
      renderOwners();
      renderStatus();
      renderBoardTokens();
      setButtons();
    }

    function init() {
      buildBoard();
      renderStatus();
      renderBoardTokens();
      renderOwners();
      setButtons();
      log("🎮 Игра началась. Ходит Красный.");
      log("💡 Важные механики: аукцион, залог, обмен, ИИ, банкротство.");
    }

    document.getElementById("rollBtn").addEventListener("click", rollTurn);
    document.getElementById("buyBtn").addEventListener("click", buyCurrentProperty);
    document.getElementById("auctionBtn").addEventListener("click", startAuction);
    document.getElementById("mortgageBtn").addEventListener("click", mortgageSelectedProperty);
    document.getElementById("tradeBtn").addEventListener("click", tradeProperty);
    document.getElementById("endTurnBtn").addEventListener("click", endTurn);

    init();
  </script>
</body>
</html>
