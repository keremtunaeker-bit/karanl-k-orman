<!DOCTYPE html>
<html lang="tr">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no" />
  <title>Karanlık Orman - 3D Zombi Hayatta Kalma</title>
  <style>
    * {
      box-sizing: border-box;
      user-select: none;
      -webkit-user-select: none;
      -webkit-touch-callout: none;
    }
    html, body {
      margin: 0;
      padding: 0;
      width: 100%;
      height: 100%;
      overflow: hidden;
      background: #070d09;
      font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
      color: #f3f7f4;
      touch-action: none;
    }
    canvas {
      display: block;
      width: 100%;
      height: 100%;
    }

    /* HUD */
    #hud {
      position: fixed;
      inset: 0;
      pointer-events: none;
      display: none;
      z-index: 10;
    }
    .hud-top {
      position: absolute;
      top: 14px;
      left: 14px;
      right: 14px;
      display: flex;
      justify-content: space-between;
      align-items: flex-start;
      gap: 12px;
      flex-wrap: wrap;
    }
    .stat-box {
      background: rgba(10, 18, 13, 0.78);
      border: 1px solid rgba(130, 220, 150, 0.25);
      backdrop-filter: blur(6px);
      padding: 10px 14px;
      border-radius: 10px;
      box-shadow: 0 6px 20px rgba(0,0,0,0.45);
      font-size: 14px;
      line-height: 1.45;
    }
    .money { color: #ffd147; font-weight: 800; }
    .hp { color: #ff6b6b; font-weight: 800; }
    .wave { color: #7ef9a9; font-weight: 800; }
    .bombs { color: #ff9f43; font-weight: 800; }

    #crosshair {
      position: fixed;
      left: 50%;
      top: 50%;
      width: 20px;
      height: 20px;
      transform: translate(-50%, -50%);
      pointer-events: none;
    }
    #crosshair::before, #crosshair::after {
      content: "";
      position: absolute;
      background: rgba(255, 255, 255, 0.85);
      box-shadow: 0 0 6px rgba(0,0,0,0.9);
    }
    #crosshair::before {
      left: 9px;
      top: 0;
      width: 2px;
      height: 20px;
    }
    #crosshair::after {
      top: 9px;
      left: 0;
      width: 20px;
      height: 2px;
    }

    #toast {
      position: fixed;
      top: 84px;
      left: 50%;
      transform: translateX(-50%);
      background: rgba(20, 32, 24, 0.92);
      border: 1px solid rgba(255, 209, 71, 0.5);
      color: #ffe28a;
      padding: 8px 16px;
      border-radius: 999px;
      font-size: 13px;
      font-weight: 700;
      opacity: 0;
      transition: opacity 0.25s ease;
      pointer-events: none;
      z-index: 25;
      white-space: nowrap;
    }
    #toast.show { opacity: 1; }

    /* Ortak Ekranlar (Lobi, Dükkan, Ölüm) */
    .overlay {
      position: fixed;
      inset: 0;
      background: radial-gradient(circle at center, rgba(16, 32, 21, 0.88), rgba(4, 8, 6, 0.96));
      display: flex;
      align-items: center;
      justify-content: center;
      z-index: 30;
      padding: 16px;
    }
    .panel {
      width: 100%;
      max-width: 640px;
      background: rgba(12, 22, 16, 0.95);
      border: 1px solid rgba(120, 230, 150, 0.3);
      border-radius: 16px;
      padding: 24px;
      box-shadow: 0 20px 60px rgba(0, 0, 0, 0.75);
      max-height: 92vh;
      overflow-y: auto;
    }
    h1 {
      margin: 0 0 6px;
      font-size: 32px;
      letter-spacing: 0.5px;
      color: #8affb3;
    }
    .sub {
      margin: 0 0 18px;
      color: #b5c9bc;
      font-size: 14px;
    }

    .mode-switch {
      display: grid;
      grid-template-columns: 1fr 1fr;
      gap: 10px;
      margin: 14px 0 18px;
    }
    .mode-btn {
      padding: 14px 12px;
      border-radius: 12px;
      border: 2px solid rgba(255,255,255,0.14);
      background: rgba(255,255,255,0.04);
      color: #dce8e0;
      font-weight: 800;
      font-size: 15px;
      cursor: pointer;
      transition: all 0.15s ease;
    }
    .mode-btn.active {
      border-color: #5cff9d;
      background: rgba(92, 255, 157, 0.18);
      color: #ffffff;
      box-shadow: 0 0 18px rgba(92, 255, 157, 0.25);
    }

    .btn {
      width: 100%;
      padding: 13px 18px;
      border: none;
      border-radius: 11px;
      font-size: 15px;
      font-weight: 800;
      cursor: pointer;
      transition: transform 0.1s ease, opacity 0.15s ease;
    }
    .btn:active { transform: scale(0.98); }
    .btn-primary {
      background: linear-gradient(135deg, #2ecc71, #1ea858);
      color: #06140c;
    }
    .btn-secondary {
      background: rgba(255, 255, 255, 0.1);
      color: #f3f7f4;
      border: 1px solid rgba(255,255,255,0.16);
      margin-top: 10px;
    }

    .records {
      display: grid;
      grid-template-columns: repeat(3, 1fr);
      gap: 10px;
      margin-bottom: 16px;
    }
    .rec-card {
      background: rgba(255,255,255,0.04);
      border: 1px solid rgba(255,255,255,0.08);
      border-radius: 10px;
      padding: 10px;
      text-align: center;
    }
    .rec-card span {
      display: block;
      font-size: 11px;
      color: #9ab3a3;
    }
    .rec-card strong {
      font-size: 18px;
      color: #ffd147;
    }

    /* Dükkan */
    #shopOverlay { display: none; }
    .shop-grid {
      display: grid;
      grid-template-columns: repeat(auto-fit, minmax(180px, 1fr));
      gap: 12px;
      margin: 14px 0 18px;
    }
    .shop-item {
      background: rgba(255,255,255,0.05);
      border: 1px solid rgba(255,255,255,0.12);
      border-radius: 12px;
      padding: 12px;
      display: flex;
      flex-direction: column;
      justify-content: space-between;
      gap: 8px;
    }
    .shop-item h3 { margin: 0; font-size: 15px; color: #fff; }
    .shop-item p { margin: 0; font-size: 12px; color: #b5c9bc; }
    .shop-btn {
      padding: 8px 10px;
      border-radius: 8px;
      border: none;
      font-weight: 700;
      cursor: pointer;
      background: #ffd147;
      color: #191302;
    }
    .shop-btn.owned {
      background: #2ecc71;
      color: #06140c;
    }

    #deathOverlay { display: none; }

    /* Mobil Kontroller */
    #mobileControls {
      display: none;
      position: fixed;
      inset: 0;
      pointer-events: none;
      z-index: 20;
    }
    #mobileControls.active {
      display: block;
    }
    .touch-btn {
      pointer-events: auto;
      position: fixed;
      border-radius: 14px;
      border: 1px solid rgba(255, 255, 255, 0.28);
      background: rgba(15, 28, 20, 0.68);
      backdrop-filter: blur(4px);
      color: #ffffff;
      font-weight: 800;
      display: flex;
      align-items: center;
      justify-content: center;
      text-align: center;
      box-shadow: 0 6px 18px rgba(0,0,0,0.45);
    }
    .touch-btn:active, .touch-btn.pressed {
      background: rgba(92, 255, 157, 0.42);
      border-color: #7ef9a9;
      transform: scale(0.95);
    }
    #btnForward { left: 22px; bottom: 102px; width: 76px; height: 64px; }
    #btnBackward { left: 22px; bottom: 26px; width: 76px; height: 64px; }
    #btnJump { left: 112px; bottom: 64px; width: 78px; height: 64px; font-size: 13px; }

    #btnShopMobile { right: 118px; bottom: 178px; width: 84px; height: 48px; font-size: 12px; background: rgba(255, 209, 71, 0.25); border-color: rgba(255, 209, 71, 0.6); }
    #btnBombMobile { right: 22px; bottom: 178px; width: 84px; height: 48px; font-size: 12px; background: rgba(255, 120, 50, 0.28); border-color: rgba(255, 159, 67, 0.65); }
    #btnFireMobile { right: 22px; bottom: 42px; width: 104px; height: 104px; border-radius: 999px; font-size: 15px; background: rgba(235, 64, 52, 0.36); border-color: rgba(255, 107, 107, 0.75); }

    #lookZone {
      pointer-events: auto;
      position: fixed;
      top: 72px;
      left: 0;
      right: 0;
      bottom: 235px;
      z-index: 15;
    }
  </style>
</head>
<body>
  <!-- LOBİ -->
  <div id="lobbyOverlay" class="overlay">
    <div class="panel">
      <h1>🌲 KARANLIK ORMAN</h1>
      <p class="sub">Sisli ormanda hayatta kal, zombi başına <strong>1 TL</strong> kazan, turuncu üstlü hızlı bebek zombilere dikkat et!</p>

      <div class="records">
        <div class="rec-card"><span>EN YÜKSEK DALGA</span><strong id="recWave">1</strong></div>
        <div class="rec-card"><span>TOPLAM LEŞ</span><strong id="recKills">0</strong></div>
        <div class="rec-card"><span>EN FAZLA PARA</span><strong id="recMoney">0 TL</strong></div>
      </div>

      <div style="font-size:13px; font-weight:700; color:#b5c9bc;">KONTROL MODU SEÇ:</div>
      <div class="mode-switch">
        <button id="modePC" class="mode-btn active" type="button">💻 PC MODU</button>
        <button id="modeMobile" class="mode-btn" type="button">📱 MOBİL MOD</button>
      </div>

      <p id="controlHint" class="sub" style="margin-bottom:14px;">
        PC: <strong>W/S</strong> İleri-Geri • <strong>A/D</strong> Dönüş • <strong>Space</strong> Zıpla • <strong>Sol Tık</strong> Vur • <strong>G</strong> Bomba • <strong>F</strong> Dükkan
      </p>

      <button id="startBtn" class="btn btn-primary" type="button">OYUNA BAŞLA</button>
    </div>
  </div>

  <!-- DÜKKAN -->
  <div id="shopOverlay" class="overlay">
    <div class="panel">
      <div style="display:flex; justify-content:space-between; align-items:center;">
        <h2 style="margin:0; color:#ffd147;">🛒 SİLAH MARKETİ</h2>
        <div style="font-weight:800; color:#ffd147;">Bakiye: <span id="shopMoney">0 TL</span></div>
      </div>
      <div id="shopGrid" class="shop-grid"></div>
      <button id="closeShopBtn" class="btn btn-primary" type="button">OYUNA DÖN</button>
    </div>
  </div>

  <!-- ÖLÜM EKRANI -->
  <div id="deathOverlay" class="overlay">
    <div class="panel" style="text-align:center;">
      <h2 style="margin:0 0 8px; color:#ff6b6b; font-size:28px;">💀 ORMANA YENİLDİN</h2>
      <p id="deathStats" class="sub">Dalga: 1 • Leş: 0 • Para: 0 TL</p>
      <button id="retryBtn" class="btn btn-primary" type="button">TEKRAR OYNA</button>
      <button id="lobbyBtn" class="btn btn-secondary" type="button">LOBİYE DÖN</button>
    </div>
  </div>

  <!-- OYUN HUD -->
  <div id="hud">
    <div class="hud-top">
      <div class="stat-box">
        <div>Can: <span id="hudHp" class="hp">100</span></div>
        <div>Para: <span id="hudMoney" class="money">0 TL</span></div>
        <div>Bomba: <span id="hudBombs" class="bombs">2</span></div>
      </div>
      <div class="stat-box" style="text-align:right;">
        <div>Dalga: <span id="hudWave" class="wave">1</span></div>
        <div>Silah: <span id="hudWeapon" style="font-weight:800;">Tabanca</span></div>
        <div>Leş: <span id="hudKills" style="font-weight:800;">0</span></div>
      </div>
    </div>
    <div id="crosshair"></div>
  </div>

  <!-- MOBİL TUŞLAR -->
  <div id="mobileControls">
    <div id="lookZone"></div>
    <div id="btnForward" class="touch-btn">İLERİ</div>
    <div id="btnBackward" class="touch-btn">GERİ</div>
    <div id="btnJump" class="touch-btn">ZIPLA</div>
    <div id="btnShopMobile" class="touch-btn">DÜKKAN</div>
    <div id="btnBombMobile" class="touch-btn">BOMBA</div>
    <div id="btnFireMobile" class="touch-btn">VUR</div>
  </div>

  <div id="toast"></div>

  <script type="module">
    import * as THREE from 'https://unpkg.com/three@0.160.0/build/three.module.js';

    // Kalıcı Kayıt
    const STORAGE_KEY = 'karanlik_orman_save_v1';
    const saved = JSON.parse(localStorage.getItem(STORAGE_KEY) || '{}');
    const records = {
      bestWave: saved.bestWave || 1,
      totalKills: saved.totalKills || 0,
      bestMoney: saved.bestMoney || 0
    };

    function saveRecords() {
      localStorage.setItem(STORAGE_KEY, JSON.stringify(records));
      updateLobbyRecords();
    }

    function updateLobbyRecords() {
      document.getElementById('recWave').textContent = records.bestWave;
      document.getElementById('recKills').textContent = records.totalKills;
      document.getElementById('recMoney').textContent = `${records.bestMoney} TL`;
    }
    updateLobbyRecords();

    // UI Elemanları
    const lobbyOverlay = document.getElementById('lobbyOverlay');
    const shopOverlay = document.getElementById('shopOverlay');
    const deathOverlay = document.getElementById('deathOverlay');
    const hud = document.getElementById('hud');
    const mobileControls = document.getElementById('mobileControls');
    const toastEl = document.getElementById('toast');

    let controlMode = 'PC'; // 'PC' | 'MOBILE'
    let gameState = 'LOBBY'; // 'LOBBY' | 'PLAYING' | 'SHOP' | 'DEAD'

    function showToast(msg) {
      toastEl.textContent = msg;
      toastEl.classList.add('show');
      clearTimeout(showToast._t);
      showToast._t = setTimeout(() => toastEl.classList.remove('show'), 1800);
    }

    const modePCBtn = document.getElementById('modePC');
    const modeMobileBtn = document.getElementById('modeMobile');
    const controlHint = document.getElementById('controlHint');

    function setControlMode(mode) {
      controlMode = mode;
      if (mode === 'PC') {
        modePCBtn.classList.add('active');
        modeMobileBtn.classList.remove('active');
        controlHint.innerHTML = 'PC: <strong>W/S</strong> İleri-Geri • <strong>A/D</strong> Dönüş • <strong>Space</strong> Zıpla • <strong>Sol Tık</strong> Vur • <strong>G</strong> Bomba • <strong>F</strong> Dükkan';
      } else {
        modeMobileBtn.classList.add('active');
        modePCBtn.classList.remove('active');
        controlHint.innerHTML = 'MOBİL: Ekranda <strong>İLERİ, GERİ, ZIPLAMA, VUR, DÜKKAN, BOMBA</strong> tuşları açılır, üst alanda sürükleyerek bakabilirsin.';
      }
    }
    modePCBtn.onclick = () => setControlMode('PC');
    modeMobileBtn.onclick = () => setControlMode('MOBILE');

    // Three.js Sahne
    const scene = new THREE.Scene();
    scene.background = new THREE.Color(0x09130d);
    scene.fog = new THREE.FogExp2(0x09130d, 0.036);

    const camera = new THREE.PerspectiveCamera(72, window.innerWidth / window.innerHeight, 0.1, 140);
    const renderer = new THREE.WebGLRenderer({ antialias: true, powerPreference: 'high-performance' });
    renderer.setSize(window.innerWidth, window.innerHeight);
    renderer.setPixelRatio(Math.min(window.devicePixelRatio, 2));
    renderer.shadowMap.enabled = true;
    document.body.prepend(renderer.domElement);

    // Işıklandırma
    const ambient = new THREE.AmbientLight(0x527a5f, 1.15);
    scene.add(ambient);

    const dirLight = new THREE.DirectionalLight(0xa8d5ba, 1.4);
    dirLight.position.set(18, 30, 12);
    dirLight.castShadow = true;
    scene.add(dirLight);

    const flashLight = new THREE.SpotLight(0xfff1c4, 3.2, 42, Math.PI / 5, 0.5);
    flashLight.position.set(0, 0, 0);
    flashLight.target.position.set(0, 0, -10);
    camera.add(flashLight);
    camera.add(flashLight.target);
    scene.add(camera);

    // Zemin ve Orman Ağaçları
    const ground = new THREE.Mesh(
      new THREE.PlaneGeometry(140, 140),
      new THREE.MeshStandardMaterial({ color: 0x132418, roughness: 0.95 })
    );
    ground.rotation.x = -Math.PI / 2;
    ground.receiveShadow = true;
    scene.add(ground);

    const trunkMat = new THREE.MeshStandardMaterial({ color: 0x382417, roughness: 0.9 });
    const leafMat = new THREE.MeshStandardMaterial({ color: 0x163822, roughness: 0.85 });

    for (let i = 0; i < 95; i++) {
      const angle = Math.random() * Math.PI * 2;
      const dist = 9 + Math.random() * 48;
      const x = Math.cos(angle) * dist;
      const z = Math.sin(angle) * dist;
      const h = 4 + Math.random() * 5;

      const tree = new THREE.Group();
      const trunk = new THREE.Mesh(new THREE.CylinderGeometry(0.28, 0.42, h, 7), trunkMat);
      trunk.position.y = h / 2;
      trunk.castShadow = true;
      tree.add(trunk);

      const crown = new THREE.Mesh(new THREE.ConeGeometry(1.8 + Math.random() * 0.8, 3.8, 7), leafMat);
      crown.position.y = h + 1.2;
      crown.castShadow = true;
      tree.add(crown);

      tree.position.set(x, 0, z);
      scene.add(tree);
    }

    // 3D Silah Modelleri
    const gunGroup = new THREE.Group();
    gunGroup.position.set(0.32, -0.28, -0.55);
    camera.add(gunGroup);

    const weapons = {
      pistol: {
        id: 'pistol',
        name: 'Tabanca',
        price: 0,
        damage: 34,
        fireRate: 0.24,
        auto: false,
        color: 0x4b5563
      },
      shotgun: {
        id: 'shotgun',
        name: 'Pompalı',
        price: 12,
        damage: 24,
        pellets: 5,
        fireRate: 0.65,
        auto: false,
        color: 0x7c2d12
      },
      rifle: {
        id: 'rifle',
        name: 'Taarruz Tüfeği',
        price: 25,
        damage: 28,
        fireRate: 0.12,
        auto: true,
        color: 0x2f4858
      },
      minigun: {
        id: 'minigun',
        name: 'Minigun',
        price: 55,
        damage: 22,
        fireRate: 0.055,
        auto: true,
        color: 0xd97706
      }
    };

    let ownedWeapons = ['pistol'];
    let currentWeaponId = 'pistol';

    function rebuildGunModel() {
      while (gunGroup.children.length > 0) {
        gunGroup.remove(gunGroup.children[0]);
      }
      const w = weapons[currentWeaponId];
      const bodyMat = new THREE.MeshStandardMaterial({ color: w.color, metalness: 0.5, roughness: 0.35 });
      const darkMat = new THREE.MeshStandardMaterial({ color: 0x1f2428, metalness: 0.7, roughness: 0.3 });

      if (currentWeaponId === 'minigun') {
        const housing = new THREE.Mesh(new THREE.BoxGeometry(0.16, 0.16, 0.42), bodyMat);
        gunGroup.add(housing);
        for (let i = -1; i <= 1; i++) {
          for (let j = -1; j <= 1; j++) {
            const barrel = new THREE.Mesh(new THREE.CylinderGeometry(0.015, 0.015, 0.62, 6), darkMat);
            barrel.rotation.x = Math.PI / 2;
            barrel.position.set(i * 0.035, j * 0.035, -0.35);
            gunGroup.add(barrel);
          }
        }
      } else {
        const len = currentWeaponId === 'pistol' ? 0.32 : 0.56;
        const body = new THREE.Mesh(new THREE.BoxGeometry(0.09, 0.11, len), bodyMat);
        body.position.z = -len * 0.35;
        gunGroup.add(body);

        const barrel = new THREE.Mesh(new THREE.CylinderGeometry(0.02, 0.02, len * 0.75, 8), darkMat);
        barrel.rotation.x = Math.PI / 2;
        barrel.position.set(0, 0.02, -len * 0.7);
        gunGroup.add(barrel);
      }
    }
    rebuildGunModel();

    // Oyuncu & Fizik Durumu
    const player = {
      pos: new THREE.Vector3(0, 1.75, 0),
      velY: 0,
      yaw: 0,
      pitch: 0,
      hp: 100,
      maxHp: 100,
      money: 0,
      bombs: 2,
      kills: 0,
      wave: 1,
      fireCooldown: 0,
      recoil: 0
    };

    const zombies = [];
    const projectiles = []; // Mermi ve bomba
    const effects = [];

    // Turuncu Üstlü Hızlı Bebek Zombi Oluşturucu
    function spawnZombie(isBaby = false) {
      const zGroup = new THREE.Group();
      const scale = isBaby ? 0.56 : 1.0;

      const skinMat = new THREE.MeshStandardMaterial({
        color: isBaby ? 0x86c46b : 0x5c8d58,
        roughness: 0.75
      });
      const orangeShirtMat = new THREE.MeshStandardMaterial({
        color: 0xff6a00,
        emissive: 0x4a1800,
        roughness: 0.6
      });
      const pantsMat = new THREE.MeshStandardMaterial({ color: 0x262b36, roughness: 0.8 });

      // Gövde (Bebek zombide parlak TURUNCU üst!)
      const torso = new THREE.Mesh(
        new THREE.BoxGeometry(0.68, 0.82, 0.38),
        isBaby ? orangeShirtMat : pantsMat
      );
      torso.position.y = 1.05;
      torso.castShadow = true;
      zGroup.add(torso);

      // Kafa
      const head = new THREE.Mesh(new THREE.BoxGeometry(0.56, 0.56, 0.56), skinMat);
      head.position.y = 1.76;
      head.castShadow = true;
      zGroup.add(head);

      // Kollar
      const armGeo = new THREE.BoxGeometry(0.22, 0.72, 0.22);
      const leftArm = new THREE.Mesh(armGeo, isBaby ? orangeShirtMat : skinMat);
      leftArm.position.set(-0.46, 1.08, 0.22);
      leftArm.rotation.x = -Math.PI / 3;
      zGroup.add(leftArm);

      const rightArm = new THREE.Mesh(armGeo, isBaby ? orangeShirtMat : skinMat);
      rightArm.position.set(0.46, 1.08, 0.22);
      rightArm.rotation.x = -Math.PI / 3;
      zGroup.add(rightArm);

      // Bacaklar
      const legGeo = new THREE.BoxGeometry(0.25, 0.65, 0.25);
      const leftLeg = new THREE.Mesh(legGeo, pantsMat);
      leftLeg.position.set(-0.2, 0.32, 0);
      zGroup.add(leftLeg);

      const rightLeg = new THREE.Mesh(legGeo, pantsMat);
      rightLeg.position.set(0.2, 0.32, 0);
      zGroup.add(rightLeg);

      zGroup.scale.setScalar(scale);

      const angle = Math.random() * Math.PI * 2;
      const dist = 18 + Math.random() * 18;
      zGroup.position.set(
        player.pos.x + Math.cos(angle) * dist,
        0,
        player.pos.z + Math.sin(angle) * dist
      );

      scene.add(zGroup);

      const baseSpeed = isBaby ? (7.6 + player.wave * 0.25) : (2.7 + player.wave * 0.14);
      zombies.push({
        mesh: zGroup,
        leftLeg,
        rightLeg,
        isBaby,
        hp: isBaby ? (45 + player.wave * 6) : (85 + player.wave * 18),
        speed: baseSpeed,
        velY: 0,
        leapCooldown: 1.2 + Math.random() * 1.4,
        attackCooldown: 0,
        animPhase: Math.random() * Math.PI * 2
      });
    }

    function spawnWave(waveNum) {
      const total = 5 + waveNum * 3;
      const babyCount = Math.max(1, Math.floor(total * 0.3));
      for (let i = 0; i < total; i++) {
        spawnZombie(i < babyCount);
      }
      showToast(`Dalga ${waveNum}! Turuncu üstlü hızlı bebek zombiler geliyor!`);
    }

    // Ateş & Bomba
    function shootWeapon() {
      if (gameState !== 'PLAYING' || player.fireCooldown > 0) return;
      const w = weapons[currentWeaponId];
      player.fireCooldown = w.fireRate;
      player.recoil = 0.08;

      const pellets = w.pellets || 1;
      for (let p = 0; p < pellets; p++) {
        const dir = new THREE.Vector3(0, 0, -1);
        const spread = w.pellets ? 0.09 : (currentWeaponId === 'minigun' ? 0.045 : 0.015);
        dir.x += (Math.random() - 0.5) * spread;
        dir.y += (Math.random() - 0.5) * spread;
        dir.normalize().applyQuaternion(camera.quaternion);

        const bullet = new THREE.Mesh(
          new THREE.SphereGeometry(0.06, 6, 6),
          new THREE.MeshBasicMaterial({ color: 0xffe066 })
        );
        bullet.position.copy(camera.position).addScaledVector(dir, 0.8);
        scene.add(bullet);

        projectiles.push({
          type: 'bullet',
          mesh: bullet,
          vel: dir.multiplyScalar(58),
          damage: w.damage,
          life: 1.4
        });
      }
    }

    function throwBomb() {
      if (gameState !== 'PLAYING') return;
      if (player.bombs <= 0) {
        showToast('Bomban bitti! Dükkandan alabilirsin.');
        return;
      }
      player.bombs--;
      updateHud();

      const dir = new THREE.Vector3(0, 0.28, -1).normalize().applyQuaternion(camera.quaternion);
      const bombMesh = new THREE.Mesh(
        new THREE.SphereGeometry(0.18, 10, 10),
        new THREE.MeshStandardMaterial({ color: 0x222222, emissive: 0x551100 })
      );
      bombMesh.position.copy(camera.position).addScaledVector(dir, 0.8);
      scene.add(bombMesh);

      projectiles.push({
        type: 'bomb',
        mesh: bombMesh,
        vel: dir.multiplyScalar(17),
        life: 1.6
      });
      showToast('Bomba atıldı!');
    }

    function explodeAt(pos, radius = 7.5, dmg = 140) {
      const blast = new THREE.Mesh(
        new THREE.SphereGeometry(radius, 14, 14),
        new THREE.MeshBasicMaterial({ color: 0xff7b00, transparent: true, opacity: 0.55 })
      );
      blast.position.copy(pos);
      scene.add(blast);
      effects.push({ mesh: blast, life: 0.25, maxLife: 0.25 });

      for (let i = zombies.length - 1; i >= 0; i--) {
        const z = zombies[i];
        if (z.mesh.position.distanceTo(pos) <= radius) {
          damageZombie(i, dmg);
        }
      }
    }

    function damageZombie(index, amount) {
      const z = zombies[index];
      if (!z) return;
      z.hp -= amount;
      if (z.hp <= 0) {
        scene.remove(z.mesh);
        zombies.splice(index, 1);
        player.money += 1; // Zombi başına 1 TL ödül
        player.kills += 1;
        records.totalKills += 1;
        if (player.money > records.bestMoney) records.bestMoney = player.money;
        saveRecords();
        updateHud();
      }
    }

    // Dükkan Sistemi
    const shopItems = [
      { id: 'hp', title: '+40 Can', desc: 'Sağlığını yeniler.', cost: 8, action: () => { player.hp = Math.min(player.maxHp, player.hp + 40); } },
      { id: 'bomb', title: '+2 Bomba', desc: 'Patlayıcı bomba ekler.', cost: 10, action: () => { player.bombs += 2; } },
      { id: 'shotgun', title: 'Pompalı (3D)', desc: 'Yakından 5 saçma atar.', cost: 12, weapon: 'shotgun' },
      { id: 'rifle', title: 'Taarruz Tüfeği (3D)', desc: 'Hızlı otomatik atış.', cost: 25, weapon: 'rifle' },
      { id: 'minigun', title: 'Minigun (3D)', desc: 'Aşırı hızlı namlu yağmuru!', cost: 55, weapon: 'minigun' }
    ];

    function renderShop() {
      const grid = document.getElementById('shopGrid');
      document.getElementById('shopMoney').textContent = `${player.money} TL`;
      grid.innerHTML = '';

      shopItems.forEach((item) => {
        const card = document.createElement('div');
        card.className = 'shop-item';

        const isWeapon = Boolean(item.weapon);
        const alreadyOwned = isWeapon && ownedWeapons.includes(item.weapon);
        const isEquipped = isWeapon && currentWeaponId === item.weapon;

        let btnText = `${item.cost} TL Al`;
        let btnClass = 'shop-btn';
        if (isEquipped) {
          btnText = 'Kuşanıldı';
          btnClass += ' owned';
        } else if (alreadyOwned) {
          btnText = 'Kuşan';
          btnClass += ' owned';
        }

        card.innerHTML = `
          <div>
            <h3>${item.title}</h3>
            <p>${item.desc}</p>
          </div>
          <button class="${btnClass}" type="button">${btnText}</button>
        `;

        card.querySelector('button').onclick = () => {
          if (isWeapon && alreadyOwned) {
            currentWeaponId = item.weapon;
            rebuildGunModel();
            updateHud();
            renderShop();
            showToast(`${weapons[currentWeaponId].name} kuşandı!`);
            return;
          }
          if (player.money < item.cost) {
            showToast('Yeterli TL yok!');
            return;
          }
          player.money -= item.cost;
          if (isWeapon) {
            ownedWeapons.push(item.weapon);
            currentWeaponId = item.weapon;
            rebuildGunModel();
          } else if (item.action) {
            item.action();
          }
          if (player.money > records.bestMoney) records.bestMoney = player.money;
          saveRecords();
          updateHud();
          renderShop();
          showToast(`${item.title} alındı!`);
        };

        grid.appendChild(card);
      });
    }

    function openShop() {
      if (gameState !== 'PLAYING') return;
      gameState = 'SHOP';
      if (document.pointerLockElement) document.exitPointerLock();
      mobileControls.classList.remove('active');
      renderShop();
      shopOverlay.style.display = 'flex';
    }

    function closeShop() {
      if (gameState !== 'SHOP') return;
      shopOverlay.style.display = 'none';
      gameState = 'PLAYING';
      if (controlMode === 'MOBILE') {
        mobileControls.classList.add('active');
      } else {
        renderer.domElement.requestPointerLock?.();
      }
    }

    document.getElementById('closeShopBtn').onclick = closeShop;

    // HUD Güncelleme
    function updateHud() {
      document.getElementById('hudHp').textContent = Math.max(0, Math.ceil(player.hp));
      document.getElementById('hudMoney').textContent = `${player.money} TL`;
      document.getElementById('hudBombs').textContent = player.bombs;
      document.getElementById('hudWave').textContent = player.wave;
      document.getElementById('hudWeapon').textContent = weapons[currentWeaponId].name;
      document.getElementById('hudKills').textContent = player.kills;
    }

    // Oyun Başlatma / Ölüm / Lobi
    function startGame() {
      // Temizlik
      zombies.forEach((z) => scene.remove(z.mesh));
      zombies.length = 0;
      projectiles.forEach((p) => scene.remove(p.mesh));
      projectiles.length = 0;

      player.pos.set(0, 1.75, 0);
      player.velY = 0;
      player.yaw = 0;
      player.pitch = 0;
      player.hp = 100;
      player.money = 0;
      player.bombs = 2;
      player.kills = 0;
      player.wave = 1;
      ownedWeapons = ['pistol'];
      currentWeaponId = 'pistol';
      rebuildGunModel();
      updateHud();

      lobbyOverlay.style.display = 'none';
      deathOverlay.style.display = 'none';
      shopOverlay.style.display = 'none';
      hud.style.display = 'block';
      gameState = 'PLAYING';

      if (controlMode === 'MOBILE') {
        mobileControls.classList.add('active');
      } else {
        mobileControls.classList.remove('active');
        renderer.domElement.requestPointerLock?.();
      }

      spawnWave(1);
    }

    function handleDeath() {
      gameState = 'DEAD';
      if (document.pointerLockElement) document.exitPointerLock();
      mobileControls.classList.remove('active');
      hud.style.display = 'none';
      document.getElementById('deathStats').textContent =
        `Dalga: ${player.wave} • Leş: ${player.kills} • Para: ${player.money} TL`;
      deathOverlay.style.display = 'flex';
    }

    function returnToLobby() {
      gameState = 'LOBBY';
      if (document.pointerLockElement) document.exitPointerLock();
      mobileControls.classList.remove('active');
      hud.style.display = 'none';
      shopOverlay.style.display = 'none';
      deathOverlay.style.display = 'none';
      lobbyOverlay.style.display = 'flex';
      updateLobbyRecords();
    }

    document.getElementById('startBtn').onclick = startGame;
    document.getElementById('retryBtn').onclick = startGame;
    document.getElementById('lobbyBtn').onclick = returnToLobby;

    // Klavye & Fare Kontrolleri (PC)
    const keys = {};
    let mouseDown = false;

    window.addEventListener('keydown', (e) => {
      keys[e.code] = true;
      if (e.code === 'KeyF') {
        if (gameState === 'PLAYING') openShop();
        else if (gameState === 'SHOP') closeShop();
      }
      if (e.code === 'KeyG' && gameState === 'PLAYING') {
        throwBomb();
      }
    });

    window.addEventListener('keyup', (e) => {
      keys[e.code] = false;
    });

    renderer.domElement.addEventListener('mousedown', (e) => {
      if (gameState !== 'PLAYING' || controlMode !== 'PC') return;
      if (!document.pointerLockElement) {
        renderer.domElement.requestPointerLock?.();
      }
      if (e.button === 0) {
        mouseDown = true;
        shootWeapon();
      }
    });

    window.addEventListener('mouseup', () => {
      mouseDown = false;
    });

    window.addEventListener('mousemove', (e) => {
      if (gameState !== 'PLAYING' || controlMode !== 'PC') return;
      if (document.pointerLockElement === renderer.domElement) {
        player.yaw -= e.movementX * 0.0024;
        player.pitch -= e.movementY * 0.0022;
        player.pitch = Math.max(-Math.PI / 2.2, Math.min(Math.PI / 2.2, player.pitch));
      }
    });

    // Mobil Dokunmatik Tuşlar (İleri, Geri, Zıplama, Vurma, Dükkan, Bomba + Bakış)
    const touchState = {
      forward: false,
      backward: false,
      jump: false,
      fire: false
    };

    function bindHoldButton(id, prop) {
      const el = document.getElementById(id);
      const start = (e) => {
        e.preventDefault();
        touchState[prop] = true;
        el.classList.add('pressed');
      };
      const end = (e) => {
        e.preventDefault();
        touchState[prop] = false;
        el.classList.remove('pressed');
      };
      el.addEventListener('touchstart', start, { passive: false });
      el.addEventListener('touchend', end, { passive: false });
      el.addEventListener('touchcancel', end, { passive: false });
    }

    bindHoldButton('btnForward', 'forward');
    bindHoldButton('btnBackward', 'backward');
    bindHoldButton('btnJump', 'jump');
    bindHoldButton('btnFireMobile', 'fire');

    document.getElementById('btnShopMobile').addEventListener('touchstart', (e) => {
      e.preventDefault();
      openShop();
    }, { passive: false });

    document.getElementById('btnBombMobile').addEventListener('touchstart', (e) => {
      e.preventDefault();
      throwBomb();
    }, { passive: false });

    const lookZone = document.getElementById('lookZone');
    let lastTouchX = null;
    let lastTouchY = null;

    lookZone.addEventListener('touchstart', (e) => {
      const t = e.changedTouches[0];
      lastTouchX = t.clientX;
      lastTouchY = t.clientY;
    }, { passive: true });

    lookZone.addEventListener('touchmove', (e) => {
      if (gameState !== 'PLAYING' || lastTouchX === null) return;
      const t = e.changedTouches[0];
      const dx = t.clientX - lastTouchX;
      const dy = t.clientY - lastTouchY;
      lastTouchX = t.clientX;
      lastTouchY = t.clientY;

      player.yaw -= dx * 0.0065;
      player.pitch -= dy * 0.0055;
      player.pitch = Math.max(-Math.PI / 2.2, Math.min(Math.PI / 2.2, player.pitch));
    }, { passive: true });

    lookZone.addEventListener('touchend', () => {
      lastTouchX = null;
      lastTouchY = null;
    });

    // Ana Döngü
    let lastTime = performance.now();

    function animate(now) {
      requestAnimationFrame(animate);
      const dt = Math.min(0.05, (now - lastTime) / 1000);
      lastTime = now;

      if (gameState === 'PLAYING') {
        player.fireCooldown = Math.max(0, player.fireCooldown - dt);
        player.recoil = Math.max(0, player.recoil - dt * 0.45);

        // Dönüş (PC A/D)
        if (controlMode === 'PC') {
          if (keys['KeyA']) player.yaw += 2.2 * dt;
          if (keys['KeyD']) player.yaw -= 2.2 * dt;
        }

        // İleri / Geri Hareket
        let moveDir = 0;
        if (keys['KeyW'] || touchState.forward) moveDir += 1;
        if (keys['KeyS'] || touchState.backward) moveDir -= 1;

        const forwardVec = new THREE.Vector3(-Math.sin(player.yaw), 0, -Math.cos(player.yaw));
        const moveSpeed = 7.2;
        player.pos.addScaledVector(forwardVec, moveDir * moveSpeed * dt);

        // Sınır
        const maxRadius = 56;
        if (player.pos.length() > maxRadius) {
          player.pos.setLength(maxRadius);
        }

        // Zıplama & Yerçekimi
        const wantsJump = keys['Space'] || touchState.jump;
        if (wantsJump && player.pos.y <= 1.76) {
          player.velY = 7.8;
        }
        player.velY -= 21 * dt;
        player.pos.y += player.velY * dt;
        if (player.pos.y < 1.75) {
          player.pos.y = 1.75;
          player.velY = 0;
        }

        camera.position.copy(player.pos);
        const euler = new THREE.Euler(player.pitch, player.yaw, 0, 'YXZ');
        camera.quaternion.setFromEuler(euler);

        // Silah Geri Tepmesi
        gunGroup.position.z = -0.55 + player.recoil;
        gunGroup.position.y = -0.28 + player.recoil * 0.35;

        // Sürekli Ateş
        const w = weapons[currentWeaponId];
        if ((mouseDown || touchState.fire) && w.auto) {
          shootWeapon();
        }

        // Mermi & Bomba Güncelleme
        for (let i = projectiles.length - 1; i >= 0; i--) {
          const p = projectiles[i];
          p.life -= dt;
          if (p.type === 'bomb') {
            p.vel.y -= 18 * dt;
          }
          p.mesh.position.addScaledVector(p.vel, dt);

          if (p.type === 'bomb' && p.mesh.position.y <= 0.18) {
            explodeAt(p.mesh.position);
            scene.remove(p.mesh);
            projectiles.splice(i, 1);
            continue;
          }

          let hit = false;
          if (p.type === 'bullet') {
            for (let j = zombies.length - 1; j >= 0; j--) {
              const z = zombies[j];
              const hitRadius = z.isBaby ? 0.72 : 1.05;
              const zCenter = z.mesh.position.clone().add(new THREE.Vector3(0, z.isBaby ? 0.65 : 1.1, 0));
              if (p.mesh.position.distanceTo(zCenter) < hitRadius) {
                damageZombie(j, p.damage);
                hit = true;
                break;
              }
            }
          }

          if (hit || p.life <= 0) {
            scene.remove(p.mesh);
            projectiles.splice(i, 1);
          }
        }

        // Zombi Hareketi (Turuncu Üstlü Hızlı Bebek Zombi Zıplama Özelliği)
        for (let i = zombies.length - 1; i >= 0; i--) {
          const z = zombies[i];
          const toPlayer = new THREE.Vector3(
            player.pos.x - z.mesh.position.x,
            0,
            player.pos.z - z.mesh.position.z
          );
          const dist = toPlayer.length();
          if (dist > 0.01) {
            toPlayer.normalize();
            z.mesh.position.addScaledVector(toPlayer, z.speed * dt);
            z.mesh.rotation.y = Math.atan2(toPlayer.x, toPlayer.z);
          }

          // Bebek zombi ekstra çevik: arada zıplayarak üstüne atlar
          if (z.isBaby) {
            z.leapCooldown -= dt;
            if (z.leapCooldown <= 0 && z.mesh.position.y <= 0.02 && dist < 11) {
              z.velY = 6.6;
              z.leapCooldown = 1.6 + Math.random() * 1.1;
            }
          }

          z.velY -= 20 * dt;
          z.mesh.position.y += z.velY * dt;
          if (z.mesh.position.y < 0) {
            z.mesh.position.y = 0;
            z.velY = 0;
          }

          // Bacak animasyonu
          z.animPhase += dt * (z.isBaby ? 17 : 8);
          z.leftLeg.rotation.x = Math.sin(z.animPhase) * 0.6;
          z.rightLeg.rotation.x = -Math.sin(z.animPhase) * 0.6;

          // Oyuncuya Hasar
          z.attackCooldown = Math.max(0, z.attackCooldown - dt);
          const hitDist = z.isBaby ? 1.25 : 1.55;
          if (dist < hitDist && z.attackCooldown <= 0) {
            player.hp -= z.isBaby ? 11 : 14;
            z.attackCooldown = z.isBaby ? 0.65 : 1.0;
            updateHud();
            if (player.hp <= 0) {
              handleDeath();
              break;
            }
          }
        }

        // Yeni Dalga Kontrolü
        if (gameState === 'PLAYING' && zombies.length === 0) {
          player.wave += 1;
          if (player.wave > records.bestWave) {
            records.bestWave = player.wave;
            saveRecords();
          }
          updateHud();
          spawnWave(player.wave);
        }
      }

      // Efektler
      for (let i = effects.length - 1; i >= 0; i--) {
        const ef = effects[i];
        ef.life -= dt;
        ef.materialOpacity = Math.max(0, ef.life / ef.maxLife);
        ef.mesh.material.opacity = ef.life / ef.maxLife;
        if (ef.life <= 0) {
          scene.remove(ef.mesh);
          effects.splice(i, 1);
        }
      }

      renderer.render(scene, camera);
    }

    requestAnimationFrame(animate);

    window.addEventListener('resize', () => {
      camera.aspect = window.innerWidth / window.innerHeight;
      camera.updateProjectionMatrix();
      renderer.setSize(window.innerWidth, window.innerHeight);
    });
  </script>
</body>
</html>
