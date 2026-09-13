<!DOCTYPE html>
<html lang="id">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Petualangan Rakit Komputer: Misi Perangkat Keras SMPN 12</title>
  <style>
    * {
      box-sizing: border-box;
      margin: 0;
      padding: 0;
      font-family: Arial, Helvetica, sans-serif;
    }
    body {
      background-color: #0f172a;
      color: #f8fafc;
      display: flex;
      justify-content: center;
      align-items: center;
      min-height: 100vh;
      line-height: 1.5;
      padding: 16px;
    }
    #game-container {
      width: 100%;
      max-width: 1280px;
      height: 720px;
      background-color: #1e293b;
      border-radius: 16px;
      box-shadow: 0 20px 25px -5px rgba(0, 0, 0, 0.5);
      position: relative;
      overflow: hidden;
      display: flex;
      flex-direction: column;
    }
    header {
      background-color: #0f172a;
      padding: 12px 24px;
      display: flex;
      justify-content: space-between;
      align-items: center;
      border-bottom: 2px solid #334155;
    }
    header .title {
      font-size: 18px;
      font-weight: bold;
      color: #38bdf8;
    }
    header .stats {
      display: flex;
      gap: 16px;
      font-weight: bold;
    }
    .badge {
      background-color: #334155;
      padding: 6px 14px;
      border-radius: 20px;
      color: #f1f5f9;
      font-size: 14px;
    }
    .screen {
      display: none;
      flex: 1;
      padding: 24px;
      flex-direction: column;
      align-items: center;
      justify-content: center;
      position: relative;
      overflow-y: auto;
    }
    .screen.active {
      display: flex;
    }
    
    /* Asset Frame Contracts */
    .asset-frame {
      position: relative;
      width: 100%;
      aspect-ratio: var(--slot-ratio, 16 / 9);
      overflow: hidden;
      border-radius: 8px;
    }
    .asset-frame > .asset-fallback,
    .asset-frame > img {
      position: absolute;
      inset: 0;
      display: block;
      width: 100%;
      height: 100%;
    }
    .asset-frame > .asset-fallback {
      display: grid;
      place-items: center;
      background: #334155;
    }
    .asset-frame > .asset-fallback svg {
      display: block;
      width: 100%;
      height: 100%;
    }
    .asset-frame > img {
      opacity: 0;
      transition: opacity 0.3s ease;
    }
    .asset-frame.asset-contain > img {
      object-fit: contain;
      object-position: center;
    }
    .asset-frame.asset-cover > img {
      object-fit: cover;
      object-position: center;
    }
    .asset-frame.is-loaded > img {
      opacity: 1;
    }
    .asset-frame.is-loaded > .asset-fallback {
      display: none;
    }
    .asset-frame.is-missing > img {
      opacity: 0;
    }

    /* Buttons & Controls */
    .btn {
      background-color: #0284c7;
      color: white;
      border: none;
      padding: 12px 24px;
      font-size: 16px;
      font-weight: bold;
      border-radius: 8px;
      cursor: pointer;
      min-width: 44px;
      min-height: 44px;
      transition: all 0.2s ease;
      display: inline-flex;
      align-items: center;
      justify-content: center;
    }
    .btn:hover {
      background-color: #0369a1;
      transform: translateY(-2px);
    }
    .btn-secondary {
      background-color: #475569;
    }
    .btn-secondary:hover {
      background-color: #334155;
    }

    /* S1 Landing Page */
    #s1 {
      text-align: center;
      gap: 16px;
    }
    #s1 h1 {
      font-size: 30px;
      color: #38bdf8;
    }
    #s1 p {
      color: #94a3b8;
      max-width: 620px;
    }

    /* Modal */
    .modal {
      display: none;
      position: absolute;
      inset: 0;
      background: rgba(15, 23, 42, 0.85);
      z-index: 50;
      justify-content: center;
      align-items: center;
    }
    .modal.active {
      display: flex;
    }
    .modal-content {
      background: #1e293b;
      padding: 32px;
      border-radius: 12px;
      max-width: 520px;
      border: 1px solid #475569;
    }

    /* Mission Grids */
    .game-grid {
      display: grid;
      grid-template-columns: 1fr 1fr;
      gap: 20px;
      width: 100%;
      max-width: 900px;
      margin-top: 12px;
    }
    .drag-items, .drop-targets {
      display: flex;
      flex-direction: column;
      gap: 10px;
      justify-content: center;
    }
    .card {
      background: #334155;
      padding: 12px 16px;
      border-radius: 8px;
      cursor: pointer;
      border: 2px solid transparent;
      display: flex;
      align-items: center;
      gap: 12px;
      text-align: left;
      color: #f8fafc;
      font-size: 15px;
      min-height: 44px;
      transition: all 0.2s ease;
    }
    .card:hover {
      background: #475569;
    }
    .card.selected {
      border-color: #38bdf8;
      background: #0f172a;
    }
    .card.correct {
      border-color: #22c55e;
      background: #14532d;
      cursor: default;
    }
    .feedback-box {
      margin-top: 16px;
      padding: 12px;
      border-radius: 8px;
      background: #334155;
      text-align: center;
      min-height: 48px;
      width: 100%;
      max-width: 700px;
      color: #e2e8f0;
    }

    /* Questions / S5 */
    .options-list {
      display: flex;
      flex-direction: column;
      gap: 12px;
      width: 100%;
      max-width: 600px;
      margin-top: 16px;
    }
  </style>
</head>
<body>

  <div id="game-container">
    <header>
      <div class="title">SMPN 12 Semarang - Gim Perangkat Keras</div>
      <div class="stats">
        <span class="badge" id="mission-progress">Misi: 0/3</span>
        <span class="badge" id="score-display">Skor: 0</span>
      </div>
    </header>

    <!-- S1: LANDING PAGE -->
    <div id="s1" class="screen active">
      <div id="asset-slot-A1" class="asset-frame asset-contain" style="--slot-ratio: 16 / 9; max-width: 420px;" data-asset="A1" data-asset-slot="asset-slot-A1">
        <div class="asset-fallback" aria-hidden="true">
          <svg viewBox="0 0 400 225">
            <rect width="400" height="225" fill="#1e293b"/>
            <circle cx="200" cy="85" r="45" fill="#38bdf8"/>
            <rect x="130" y="140" width="140" height="65" rx="10" fill="#0284c7"/>
            <text x="200" y="177" fill="#ffffff" font-size="14" font-weight="bold" text-anchor="middle">Teknisi SMPN 12</text>
          </svg>
        </div>
        <img class="asset-image" src="./assets/images/hero-teknisi-komputer.png" alt="Hero Teknisi Komputer SMPN 12" decoding="async">
      </div>
      <h1>Petualangan Rakit Komputer</h1>
      <p>Bantu perbaiki komputer laboratorium SMPN 12 Semarang dengan mempelajari jenis dan fungsi perangkat keras!</p>
      <div style="display: flex; gap: 12px; margin-top: 8px;">
        <button class="btn btn-secondary" onclick="openModal()">Tujuan Pembelajaran</button>
        <button class="btn" onclick="goToScreen('s2')">Mulai</button>
      </div>
    </div>

    <!-- S2: INSTRUKSI -->
    <div id="s2" class="screen">
      <h2 style="color: #38bdf8;">Instruksi Permainan</h2>
      <p style="margin: 16px 0; max-width: 600px; text-align: center; color: #cbd5e1;">
        Kamu bertugas sebagai <strong>Teknisi Komputer Muda SMPN 12 Semarang</strong>. Selesaikan 3 Misi berturut-turut untuk menguji pemahamanmu tentang Perangkat Input, Pemrosesan, Storage, dan Output!
      </p>
      <button class="btn" onclick="startMissions()">Mulai Misi 1</button>
    </div>

    <!-- S3: MISI 1 - IDENTIFIKASI -->
    <div id="s3" class="screen">
      <h3>Misi 1: Identifikasi Komponen & Kategori Fungsi</h3>
      <p style="margin-bottom: 8px; color: #94a3b8;">Ketuk Komponen di sebelah kiri, lalu ketuk Kategori Fungsi yang sesuai!</p>
      
      <div class="game-grid">
        <div class="drag-items">
          <div class="card" id="comp-keyboard" onclick="selectComponent('keyboard')">
            <div id="asset-slot-A2" class="asset-frame asset-contain" style="--slot-ratio: 1/1; width: 44px;" data-asset="A2" data-asset-slot="asset-slot-A2">
              <div class="asset-fallback" aria-hidden="true">
                <svg viewBox="0 0 44 44"><rect width="44" height="44" rx="6" fill="#475569"/><text x="22" y="27" fill="#38bdf8" font-size="10" font-weight="bold" text-anchor="middle">KEYBOARD</text></svg>
              </div>
              <img class="asset-image" src="./assets/images/ilustrasi-keyboard.png" alt="Keyboard Komputer" decoding="async">
            </div>
            <span>Keyboard</span>
          </div>

          <div class="card" id="comp-cpu" onclick="selectComponent('cpu')">
            <div id="asset-slot-A3" class="asset-frame asset-contain" style="--slot-ratio: 1/1; width: 44px;" data-asset="A3" data-asset-slot="asset-slot-A3">
              <div class="asset-fallback" aria-hidden="true">
                <svg viewBox="0 0 44 44"><rect width="44" height="44" rx="6" fill="#475569"/><text x="22" y="27" fill="#38bdf8" font-size="10" font-weight="bold" text-anchor="middle">CPU</text></svg>
              </div>
              <img class="asset-image" src="./assets/images/ilustrasi-cpu.png" alt="CPU Processor" decoding="async">
            </div>
            <span>Processor (CPU)</span>
          </div>

          <div class="card" id="comp-ram" onclick="selectComponent('ram')">
            <div id="asset-slot-A4" class="asset-frame asset-contain" style="--slot-ratio: 1/1; width: 44px;" data-asset="A4" data-asset-slot="asset-slot-A4">
              <div class="asset-fallback" aria-hidden="true">
                <svg viewBox="0 0 44 44"><rect width="44" height="44" rx="6" fill="#475569"/><text x="22" y="27" fill="#38bdf8" font-size="10" font-weight="bold" text-anchor="middle">RAM</text></svg>
              </div>
              <img class="asset-image" src="./assets/images/ilustrasi-ram.png" alt="RAM Memory" decoding="async">
            </div>
            <span>RAM</span>
          </div>

          <div class="card" id="comp-monitor" onclick="selectComponent('monitor')">
            <div id="asset-slot-A5" class="asset-frame asset-contain" style="--slot-ratio: 1/1; width: 44px;" data-asset="A5" data-asset-slot="asset-slot-A5">
              <div class="asset-fallback" aria-hidden="true">
                <svg viewBox="0 0 44 44"><rect width="44" height="44" rx="6" fill="#475569"/><text x="22" y="27" fill="#38bdf8" font-size="10" font-weight="bold" text-anchor="middle">MONITOR</text></svg>
              </div>
              <img class="asset-image" src="./assets/images/ilustrasi-monitor.png" alt="Monitor LED" decoding="async">
            </div>
            <span>Monitor</span>
          </div>
        </div>

        <div class="drop-targets">
          <div class="card" onclick="matchCategory('Input')">Kategori: Perangkat Input</div>
          <div class="card" onclick="matchCategory('Proses')">Kategori: Perangkat Pemrosesan</div>
          <div class="card" onclick="matchCategory('Storage')">Kategori: Perangkat Penyimpanan</div>
          <div class="card" onclick="matchCategory('Output')">Kategori: Perangkat Output</div>
        </div>
      </div>

      <div class="feedback-box" id="feedback-m1">Pilih salah satu komponen di sebelah kiri terlebih dahulu.</div>
      <button class="btn" id="btn-m1-next" style="display:none; margin-top: 12px;" onclick="goToScreen('s4')">Lanjut Misi 2</button>
    </div>

    <!-- S4: MISI 2 - ALUR KERJA -->
    <div id="s4" class="screen">
      <h3>Misi 2: Analisis Alur Pemrosesan Data</h3>
      <p style="margin: 8px 0; color: #94a3b8;">Pilihlah urutan alur pemrosesan data komputer yang paling tepat!</p>
      
      <div class="options-list">
        <button class="card" onclick="answerM2(false)">Proses &rarr; Input &rarr; Output &rarr; Storage</button>
        <button class="card" onclick="answerM2(true)">Input &rarr; Pemrosesan (CPU) &rarr; Storage / Output</button>
        <button class="card" onclick="answerM2(false)">Output &rarr; Storage &rarr; Pemrosesan &rarr; Input</button>
      </div>

      <div class="feedback-box" id="feedback-m2">Pilih salah satu alur di atas untuk memverifikasi.</div>
      <button class="btn" id="btn-m2-next" style="display:none; margin-top: 12px;" onclick="goToScreen('s5')">Lanjut Misi 3</button>
    </div>

    <!-- S5: MISI 3 - TROUBLESHOOTING -->
    <div id="s5" class="screen">
      <h3>Misi 3: Troubleshooting Komputer Lab</h3>
      <div id="asset-slot-A6" class="asset-frame asset-contain" style="--slot-ratio: 16 / 9; max-width: 320px; margin: 12px 0;" data-asset="A6" data-asset-slot="asset-slot-A6">
        <div class="asset-fallback" aria-hidden="true">
          <svg viewBox="0 0 320 180">
            <rect width="320" height="180" fill="#334155" rx="8"/>
            <text x="160" y="95" fill="#ef4444" font-size="14" font-weight="bold" text-anchor="middle">Komputer Lab Lambat!</text>
          </svg>
        </div>
        <img class="asset-image" src="./assets/images/trouble-komputer-lambat.png" alt="Troubleshooting Komputer Lambat" decoding="async">
      </div>
      <p style="max-width: 580px; text-align: center; color: #cbd5e1;">
        Studi Kasus: Komputer Lab 02 SMPN 12 Semarang lambat saat menjalankan beberapa aplikasi pembelajaran sekaligus. Komponen mana yang perlu ditingkatkan (upgrade)?
      </p>

      <div class="options-list">
        <button class="card" onclick="answerM3(true)">Tambah Kapasitas RAM (Memory)</button>
        <button class="card" onclick="answerM3(false)">Ganti Layar Monitor Lebih Besar</button>
        <button class="card" onclick="answerM3(false)">Ganti Keyboard & Mouse Baru</button>
      </div>

      <div class="feedback-box" id="feedback-m3">Tentukan keputusan perbaikan hardware.</div>
      <button class="btn" id="btn-m3-next" style="display:none; margin-top: 12px;" onclick="finishGame()">Lihat Hasil Akhir</button>
    </div>

    <!-- S6: HASIL & REFLEKSI -->
    <div id="s6" class="screen">
      <h2 style="color: #38bdf8;">Misi Berhasil Diselesaikan!</h2>
      <p id="final-score" style="font-size: 26px; font-weight: bold; color: #22c55e; margin: 12px 0;">Skor Akhir: 100</p>
      <p style="max-width: 500px; text-align: center; color: #cbd5e1;">Selamat! Kamu telah resmi mendapatkan lencana Teknisi Komputer Muda SMPN 12 Semarang.</p>
      
      <div style="margin-top: 24px; display: flex; gap: 12px;">
        <button class="btn" onclick="restartGame()">Main Lagi</button>
        <button class="btn btn-secondary" onclick="goToScreen('s1')">Ke Beranda</button>
      </div>
    </div>

  </div>

  <!-- MODAL TUJUAN PEMBELAJARAN -->
  <div class="modal" id="tp-modal" onclick="closeModalOnOutside(event)">
    <div class="modal-content">
      <h3 style="color: #38bdf8; margin-bottom: 12px;">Tujuan Pembelajaran</h3>
      <ul style="margin-left: 20px; line-height: 1.8; color: #cbd5e1;">
        <li><strong>TP1:</strong> Mengidentifikasi jenis-jenis perangkat keras komputer (Input, Proses, Storage, Output).</li>
        <li><strong>TP2:</strong> Menganalisis alur pemrosesan data pada komputer.</li>
        <li><strong>TP3:</strong> Menentukan pemecahan masalah (troubleshooting) spesifikasi perangkat keras sederhana.</li>
      </ul>
      <button class="btn" style="margin-top: 20px; width: 100%;" onclick="closeModal()">Tutup</button>
    </div>
  </div>

  <script>
    // State Management
    let score = 0;
    let currentSelectedComp = null;
    let matchedCount = 0;

    const compData = {
      'keyboard': 'Input',
      'cpu': 'Proses',
      'ram': 'Storage',
      'monitor': 'Output'
    };

    function updateHeader() {
      document.getElementById('score-display').innerText = 'Skor: ' + score;
    }

    function goToScreen(screenId) {
      document.querySelectorAll('.screen').forEach(s => s.classList.remove('active'));
      document.getElementById(screenId).classList.add('active');
    }

    function openModal() {
      document.getElementById('tp-modal').classList.add('active');
    }

    function closeModal() {
      document.getElementById('tp-modal').classList.remove('active');
    }

    function closeModalOnOutside(e) {
      if (e.target.id === 'tp-modal') {
        closeModal();
      }
    }

    document.addEventListener('keydown', (e) => {
      if (e.key === 'Escape') closeModal();
    });

    function startMissions() {
      document.getElementById('mission-progress').innerText = 'Misi: 1/3';
      goToScreen('s3');
    }

    function selectComponent(comp) {
      currentSelectedComp = comp;
      document.querySelectorAll('.drag-items .card').forEach(c => c.classList.remove('selected'));
      document.getElementById('comp-' + comp).classList.add('selected');
      document.getElementById('feedback-m1').innerText = 'Komponen dipilih: ' + comp.toUpperCase() + '. Sekarang ketuk kategori fungsi yang sesuai!';
    }

    function matchCategory(cat) {
      if (!currentSelectedComp) {
        document.getElementById('feedback-m1').innerText = 'Pilih salah satu komponen di sebelah kiri terlebih dahulu!';
        return;
      }
      
      if (compData[currentSelectedComp] === cat) {
        score += 25;
        matchedCount++;
        document.getElementById('feedback-m1').innerText = 'Tepat! ' + currentSelectedComp.toUpperCase() + ' tergolong Perangkat ' + cat + '.';
        document.getElementById('comp-' + currentSelectedComp).classList.add('correct');
        document.getElementById('comp-' + currentSelectedComp).onclick = null;
        currentSelectedComp = null;
        updateHeader();

        if (matchedCount >= 4) {
          document.getElementById('feedback-m1').innerText = 'Luar biasa! Semua 4 komponen berhasil terpasang dengan benar!';
          document.getElementById('btn-m1-next').style.display = 'inline-flex';
        }
      } else {
        document.getElementById('feedback-m1').innerText = 'Kurang tepat. Coba periksa kembali alur kerja dari ' + currentSelectedComp.toUpperCase() + '!';
      }
    }

    function answerM2(isCorrect) {
      if (isCorrect) {
        score += 35;
        updateHeader();
        document.getElementById('feedback-m2').innerText = 'Tepat! Alur kerja komputer diawali dari Input, diproses oleh CPU, lalu disajikan ke Output atau disimpan ke Storage.';
        document.getElementById('btn-m2-next').style.display = 'inline-flex';
        document.getElementById('mission-progress').innerText = 'Misi: 2/3';
      } else {
        document.getElementById('feedback-m2').innerText = 'Alur belum tepat! Ingat: Data harus dimasukkan (Input) terlebih dahulu sebelum dapat diproses CPU.';
      }
    }

    function answerM3(isCorrect) {
      if (isCorrect) {
        score += 40;
        updateHeader();
        document.getElementById('feedback-m3').innerText = 'Tepat sekali! Menambah kapasitas RAM memungkinkan komputer menyimpan data aplikasi aktif lebih banyak saat multitasking.';
        document.getElementById('btn-m3-next').style.display = 'inline-flex';
        document.getElementById('mission-progress').innerText = 'Misi: 3/3';
      } else {
        document.getElementById('feedback-m3').innerText = 'Kurang tepat. Mengganti peranti input/output tidak akan menyelesaikan masalah kecepatan komputasi sistem.';
      }
    }

    function finishGame() {
      goToScreen('s6');
      document.getElementById('final-score').innerText = 'Skor Akhir: ' + score;
    }

    function restartGame() {
      score = 0;
      matchedCount = 0;
      currentSelectedComp = null;
      updateHeader();
      document.getElementById('mission-progress').innerText = 'Misi: 0/3';
      document.querySelectorAll('.drag-items .card').forEach(c => {
        c.classList.remove('correct', 'selected');
      });
      document.getElementById('comp-keyboard').onclick = () => selectComponent('keyboard');
      document.getElementById('comp-cpu').onclick = () => selectComponent('cpu');
      document.getElementById('comp-ram').onclick = () => selectComponent('ram');
      document.getElementById('comp-monitor').onclick = () => selectComponent('monitor');
      document.getElementById('btn-m1-next').style.display = 'none';
      document.getElementById('btn-m2-next').style.display = 'none';
      document.getElementById('btn-m3-next').style.display = 'none';
      goToScreen('s1');
    }

    // Auto-load PNG Contracts with SVG Fallback Handler
    document.querySelectorAll(".asset-frame > img.asset-image").forEach((img) => {
      const frame = img.closest(".asset-frame");

      const showImage = () => {
        frame.classList.add("is-loaded");
        frame.classList.remove("is-missing");
      };

      const showFallback = () => {
        frame.classList.remove("is-loaded");
        frame.classList.add("is-missing");
      };

      img.addEventListener("load", showImage);
      img.addEventListener("error", showFallback);

      if (img.complete) {
        if (img.naturalWidth > 0) {
          showImage();
        } else {
          showFallback();
        }
      }
    });
  </script>
</body>
</html>
