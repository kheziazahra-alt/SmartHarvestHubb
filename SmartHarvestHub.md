<!DOCTYPE html>
<html lang="id">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Smart Harvest Hub</title>
<link rel="preconnect" href="https://fonts.googleapis.com">
<link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
<link href="https://fonts.googleapis.com/css2?family=Plus+Jakarta+Sans:wght@400;500;700;800&display=swap" rel="stylesheet">
<script src="https://unpkg.com/mqtt/dist/mqtt.min.js"></script>
<style>
  :root { --tua:#14452F; --daun:#3E8E5A; --padi:#E3B341; --cabai:#C8402E; --tanah:#EEF3EA; --teks:#17261D; --redup:#4A5E50; --garis:#D6E0D2; }
  * { box-sizing: border-box; }
  body {
    margin: 0; font-family: "Plus Jakarta Sans", system-ui, sans-serif; color: var(--teks);
    background-color: #CFE0C8;
    background-image:
      url("data:image/svg+xml,%3Csvg xmlns='http://www.w3.org/2000/svg' width='120' height='120' viewBox='0 0 120 120'%3E%3Cg fill='%233E8E5A' fill-opacity='.14'%3E%3Cpath d='M20 40c0-14 10-24 24-24 0 14-10 24-24 24z'/%3E%3Cpath d='M82 104c0-12 8-20 20-20 0 12-8 20-20 20z'/%3E%3C/g%3E%3Ccircle cx='90' cy='28' r='5' fill='%23E3B341' fill-opacity='.28'/%3E%3Ccircle cx='30' cy='92' r='4' fill='%23C8402E' fill-opacity='.2'/%3E%3C/svg%3E"),
      radial-gradient(circle at 15% 10%, rgba(227,179,65,.35), transparent 40%),
      radial-gradient(circle at 90% 85%, rgba(62,142,90,.35), transparent 45%),
      linear-gradient(160deg, #D9EBCF, #B9D6B0);
    background-attachment: fixed;
  }
  .dunia { min-height: 100vh; display: grid; grid-template-columns: 1fr; justify-items: center; }
  .intro { display: none; }
  .app { width: 100%; max-width: 420px; min-height: 100vh; background: var(--tanah); display: flex; flex-direction: column; }
  @media (min-width: 900px) {
    .dunia { grid-template-columns: 1fr 420px; gap: 64px; align-items: center; max-width: 1000px; margin: 0 auto; padding: 32px; }
    .intro { display: block; }
    .app { min-height: 860px; max-height: 860px; border-radius: 32px; overflow: hidden; box-shadow: 0 24px 60px rgba(20,69,47,.3); }
    main { overflow-y: auto; }
  }
  .intro h2 { font-size: 40px; line-height: 1.1; margin: 20px 0 12px; color: var(--tua); }
  .intro p { font-size: 17px; line-height: 1.6; color: var(--redup); max-width: 440px; }
  .intro ul { padding: 0; list-style: none; margin: 24px 0 0; }
  .intro li { padding: 10px 0 10px 18px; border-left: 3px solid var(--padi); margin-bottom: 8px; font-weight: 500; }

  header { background-color: var(--tua); background-image: repeating-linear-gradient(135deg, rgba(255,255,255,.06) 0 2px, transparent 2px 14px); color: #fff; padding: 10px 12px 9px; display: flex; gap: 8px; align-items: center; }
  header svg { width: 32px; height: 32px; flex: none; }
  header h1 { margin: 0; font-size: 15px; font-weight: 800; }
  header p { margin: 1px 0 0; font-size: 10.5px; color: #CFE5D6; }
  main { flex: 1; padding: 8px; }
  .page { display: none; }
  .page.aktif { display: block; }

  .card { background: #fff; border: 1px solid var(--garis); border-radius: 12px; padding: 8px 10px; margin-bottom: 7px; }
  .card.warn { background: #FFF3D6; border-color: #EBCB7A; }
  .card.bahaya { background: #FCE1DC; border-color: #E8A79D; }
  .tegas { border-left: 4px solid var(--padi); }
  h3 { margin: 0 0 3px; font-size: 12px; font-weight: 700; }
  p { margin: 2px 0; font-size: 11px; line-height: 1.4; color: var(--redup); }
  .baris { display: flex; justify-content: space-between; align-items: center; padding: 4px 0; border-bottom: 1px solid #E8EFE5; font-size: 11.5px; }
  .baris:last-child { border-bottom: 0; }
  .baris b { font-weight: 700; color: var(--teks); }
  .lencana { font-size: 9.5px; font-weight: 700; padding: 1px 7px; border-radius: 99px; background: #DDEFE2; color: var(--tua); white-space: nowrap; }
  .lencana.w { background: var(--padi); color: #3A2A00; }
  .lencana.m { background: #F5C9C2; color: #6B1A10; }
  .angka { display: flex; gap: 7px; }
  .angka .card { flex: 1; }
  .angka strong { display: block; font-size: 20px; font-weight: 800; color: var(--tua); line-height: 1.1; }
  .bar { height: 8px; background: #E1EADD; border-radius: 99px; overflow: hidden; position: relative; margin: 8px 0 3px; }
  .bar i { display: block; height: 100%; background: var(--daun); border-radius: 99px; transition: width .25s, background .25s; }
  .bar.gas i.tinggi { background: var(--cabai); }
  .bar em { position: absolute; top: -2px; bottom: -2px; width: 2px; background: var(--teks); left: 44%; }
  input[type=range] { width: 100%; accent-color: var(--daun); }
  .aksi { width: 100%; min-height: 40px; border: 0; border-radius: 10px; background: var(--tua); color: #fff; font: inherit; font-weight: 700; font-size: 13px; cursor: pointer; margin-top: 8px; }
  .aksi:hover { background: #0F3624; }
  .aksi.abu { background: #E1EADD; color: var(--tua); }

  /* Daftar sayur (ringkas) */
  .judulkartu { display: flex; justify-content: space-between; align-items: center; margin-bottom: 4px; }
  .judulkartu h3 { margin: 0; }
  .tambah { width: 26px; height: 26px; border-radius: 50%; border: 0; background: var(--tua); color: #fff; font-size: 18px; line-height: 1; cursor: pointer; display: grid; place-items: center; box-shadow: 0 2px 0 var(--padi); }
  .tambah:hover { background: #0F3624; }
  .sayur { padding: 5px 0; border-bottom: 1px solid #E8EFE5; }
  .sayur:last-child { border-bottom: 0; }
  .sayur .atas { display: flex; justify-content: space-between; align-items: center; gap: 6px; }
  .sayur .nama { font-weight: 700; font-size: 12px; }
  .sayur .info { font-size: 10.5px; color: var(--redup); margin-top: 0; }
  .sayur .info b { color: var(--teks); }
  .sayur .titik { color: var(--padi); }
  .sayur .bar { margin: 4px 0 0; height: 5px; }
  .sayur .bar i.rendah { background: var(--padi); }
  .sayur .bar i.kritis { background: var(--cabai); }
  .hapus { border: 0; background: none; color: #9AAB9E; font-size: 16px; cursor: pointer; padding: 0 2px; line-height: 1; }
  .hapus:hover { color: var(--cabai); }
  .kosong { font-size: 12px; color: var(--redup); padding: 8px 0; }

  /* Form tambah */
  #formTambah { display: none; background: #F4F9F1; border: 1px dashed var(--daun); border-radius: 12px; padding: 10px; margin: 6px 0 8px; }
  #formTambah.buka { display: block; }
  #formTambah label { display: block; font-size: 11.5px; font-weight: 700; color: var(--tua); margin: 6px 0 2px; }
  #formTambah input { width: 100%; font: inherit; font-size: 13px; padding: 8px 10px; border: 1px solid var(--garis); border-radius: 8px; background: #fff; }
  #formTambah .dua { display: flex; gap: 8px; }
  #formTambah .dua > div { flex: 1; }
  #formTambah .tombol { display: flex; gap: 8px; }
  #formTambah .tombol .aksi { margin-top: 10px; }
  #pesanForm { color: var(--cabai); font-size: 11.5px; font-weight: 700; margin: 6px 0 0; min-height: 0; }

  nav { display: flex; background: #fff; border-top: 1px solid var(--garis); }
  nav button { flex: 1; min-height: 44px; border: 0; background: none; font: inherit; font-size: 9.5px; font-weight: 500; color: var(--redup); cursor: pointer; display: flex; flex-direction: column; align-items: center; justify-content: center; gap: 1px; }
  nav svg { width: 18px; height: 18px; fill: none; stroke: currentColor; stroke-width: 1.8; stroke-linecap: round; stroke-linejoin: round; }
  nav button.aktif { color: var(--tua); font-weight: 800; box-shadow: inset 0 3px 0 var(--padi); }
  button:focus-visible, input:focus-visible { outline: 3px solid var(--padi); outline-offset: 2px; }
  .catatan { font-size: 11px; text-align: center; margin: 4px 0 0; }

  /* Harga: tap nama sayur untuk lihat harganya */
  .chips { display: flex; gap: 6px; overflow-x: auto; padding: 2px 0 6px; scrollbar-width: none; }
  .chips::-webkit-scrollbar { display: none; }
  .chip { flex: none; border: 1px solid var(--garis); background: #F4F9F1; color: var(--redup); font: inherit; font-size: 11px; font-weight: 700; padding: 5px 11px; border-radius: 99px; cursor: pointer; }
  .chip.aktif { background: var(--tua); border-color: var(--tua); color: #fff; box-shadow: 0 2px 0 var(--padi); }
  .hargaBesar { display: flex; justify-content: space-between; align-items: baseline; padding: 6px 2px 0; }
  .hargaBesar span { font-size: 11px; color: var(--redup); }
  .hargaBesar strong { font-size: 22px; font-weight: 800; color: var(--tua); }
</style>
</head>
<body>

<svg width="0" height="0" style="position:absolute" aria-hidden="true">
  <symbol id="emblem" viewBox="0 0 64 64">
    <polygon points="32,3 57,17.5 57,46.5 32,61 7,46.5 7,17.5" fill="#1D5A3B" stroke="#E3B341" stroke-width="2.5" stroke-linejoin="round"/>
    <path d="M32 50V32" stroke="#fff" stroke-width="3" stroke-linecap="round" fill="none"/>
    <path d="M32 40C22 40 18 34 18 28C26 28 32 32 32 40Z" fill="#7CC38F"/>
    <path d="M32 33C32 25 38 21 46 21C46 29 40 33 32 33Z" fill="#fff"/>
    <path d="M23 15Q32 9 41 15M27 19Q32 16 37 19" stroke="#E3B341" stroke-width="2.2" stroke-linecap="round" fill="none"/>
  </symbol>
</svg>

<div class="dunia">
  <section class="intro">
    <svg width="88" height="88" aria-hidden="true"><use href="#emblem"/></svg>
    <h2>Panen terpantau,<br>harga tetap terjaga.</h2>
    <p>Smart Harvest Hub memantau lahan dan gudang lewat drone, sensor IoT, dan CCTV cerdas, lalu memberi tahu petani kapan hasil panen harus dikirim ke pasar.</p>
    <ul>
      <li>Peringatan cuaca dan kelembapan lahan</li>
      <li>Prediksi umur simpan hasil panen di gudang</li>
      <li>Rekomendasi kirim ke pasar atau UMKM olahan</li>
    </ul>
  </section>

  <div class="app">
    <header>
      <svg aria-hidden="true"><use href="#emblem"/></svg>
      <div>
        <h1>Smart Harvest Hub</h1>
        <p id="statusKoneksi">Lahan 01 dan Gudang 01 • Menghubungkan MQTT...</p>
      </div>
    </header>
    <main>
      <section class="page aktif" id="beranda">
        <div class="card warn tegas" id="kartuAlertBeranda">
          <span class="lencana w">Peringatan cuaca BMKG</span>
          <h3 style="margin-top:7px" id="judulAlertBeranda">Hujan lebat diprakirakan besok sore</h3>
          <p id="isiAlertBeranda">Disarankan menunda panen cabai di Lahan 01.</p>
        </div>
        <div class="angka">
          <div class="card"><p>Suhu lahan</p><strong id="berandaSuhu">29&deg;C</strong></div>
          <div class="card"><p>Kelembapan tanah</p><strong id="berandaTanah">62%</strong></div>
        </div>
        <div class="card">
          <div class="judulkartu">
            <h3>Stok gudang</h3>
            <button class="tambah" id="bukaForm" aria-label="Tambah sayur" title="Tambah sayur">+</button>
          </div>
          <div id="formTambah">
            <label for="fNama">Nama sayur</label>
            <input id="fNama" type="text" placeholder="contoh: Tomat" maxlength="24">
            <div class="dua">
              <div><label for="fSisa">Sisa umur (hari)</label><input id="fSisa" type="number" min="0" inputmode="numeric" placeholder="3"></div>
              <div><label for="fTotal">Total umur (hari)</label><input id="fTotal" type="number" min="1" inputmode="numeric" placeholder="7"></div>
            </div>
            <label for="fHarga">Harga pasar (Rp/kg)</label>
            <input id="fHarga" type="number" min="0" inputmode="numeric" placeholder="12000">
            <p id="pesanForm"></p>
            <div class="tombol">
              <button class="aksi" id="simpanSayur">Simpan</button>
              <button class="aksi abu" id="batalSayur" type="button">Batal</button>
            </div>
          </div>
          <div id="daftarGudang"></div>
        </div>
        <div class="card">
          <h3>Harga pasar induk hari ini</h3>
          <div class="chips" id="chipHarga" role="tablist"></div>
          <div class="hargaBesar" id="hargaTerpilih" aria-live="polite"></div>
        </div>
      </section>
      <section class="page" id="lahan">
        <div class="card">
          <h3>Sensor lapangan</h3>
          <div class="baris"><span>Suhu udara</span><b id="lahanSuhu">29&deg;C</b></div>
          <div class="baris"><span>Kelembapan udara</span><b id="lahanLembap">78%</b></div>
          <div class="baris"><span>Kelembapan tanah</span><b id="lahanTanah">62%</b></div>
          <div class="bar"><i id="barTanah" style="width:62%"></i></div>
          <div class="baris"><span>Hujan</span><b id="lahanHujan">Tidak</b></div>
        </div>
        <div class="card">
          <h3>Drone</h3>
          <div class="baris"><span>Terbang terakhir</span><b>06.30</b></div>
          <div class="baris"><span>Kesehatan tanaman</span><span class="lencana">Baik</span></div>
        </div>
        <div class="card warn tegas" id="kartuAlertLahan">
          <h3>Peringatan kelembapan</h3>
          <p id="isiAlertLahan">Tanah mendekati jenuh jika hujan turun besok.</p>
        </div>
      </section>
      <section class="page" id="gudang">
        <div class="card">
          <h3>Kondisi penyimpanan</h3>
          <div class="baris"><span>Suhu</span><b id="gudangSuhu">10&deg;C</b></div>
          <div class="baris"><span>Kelembapan</span><b id="gudangLembap">88%</b></div>
          <div class="baris"><span>Pendingin</span><span class="lencana" id="statusPendingin">Aktif</span></div>
        </div>
        <div class="card">
          <h3>Coba alarm gas</h3>
          <p>Geser sampai melewati garis hitam (batas 1800) atau pantau nilai real-time.</p>
          <div class="bar gas"><i id="isiGas" style="width:15%"></i><em></em></div>
          <input type="range" id="gas" min="0" max="4095" value="600" aria-label="Nilai gas">
          <div class="baris" style="border:0"><span>Nilai gas</span><b id="nilaiGas">600</b></div>
        </div>
        <div class="card" id="kartuAlarm">
          <h3 id="judulAlarm">Kondisi aman</h3>
          <p id="isiAlarm">LED dan buzzer mati.</p>
        </div>
        <div class="card">
          <h3>Prediksi CCTV cerdas</h3>
          <div class="baris"><span>Sisa umur simpan (cabai merah)</span><b id="gudangSisaHari">3 hari</b></div>
          <div class="baris"><span>Area mulai layu</span><b id="gudangLayu">12%</b></div>
        </div>
      </section>
      <section class="page" id="pasar">
        <div class="card">
          <h3>Harga komoditas</h3>
          <div id="daftarHargaPasar"></div>
        </div>
        <div class="card tegas">
          <h3>Rekomendasi kirim</h3>
          <p>Stok lahan 200 kg, stok gudang 120 kg.</p>
          <p><b>Kirim 120 kg cabai ke Pasar Induk</b></p>
          <button class="aksi" id="setujui">Setujui pengiriman</button>
          <p id="hasilSetuju" style="display:none;color:var(--tua);font-weight:700">Pengiriman disetujui (simulasi).</p>
        </div>
        <div class="card">
          <h3>Alternatif</h3>
          <p>Salurkan sisa stok ke UMKM olahan, misalnya saus dan keripik, agar harga tidak jatuh.</p>
        </div>
      </section>
      <p class="catatan" id="catatanBawah">Menunggu data real-time dari Wokwi MQTT...</p>
    </main>
    <nav aria-label="Menu utama">
      <button class="aktif" data-ke="beranda"><svg viewBox="0 0 24 24"><path d="M3 11l9-8 9 8v9H3z"/></svg>Beranda</button>
      <button data-ke="lahan"><svg viewBox="0 0 24 24"><path d="M5 19c0-8 5-14 14-14 0 9-6 14-14 14zM5 19l7-7"/></svg>Lahan</button>
      <button data-ke="gudang"><svg viewBox="0 0 24 24"><path d="M3 9l9-5 9 5v11H3zM8 20v-6h8v6"/></svg>Gudang</button>
      <button data-ke="pasar"><svg viewBox="0 0 24 24"><path d="M4 9l1-5h14l1 5a3 3 0 01-6 0 3 3 0 01-6 0 3 3 0 01-4 0zM5 12v8h14v-8"/></svg>Pasar</button>
    </nav>
  </div>
</div>

<script>
  // ---------- Navigasi tab ----------
  var tombol = document.querySelectorAll("nav button");
  var halaman = document.querySelectorAll(".page");
  tombol.forEach(function (t) {
    t.addEventListener("click", function () {
      tombol.forEach(function (x) { x.classList.remove("aktif"); });
      halaman.forEach(function (h) { h.classList.remove("aktif"); });
      t.classList.add("aktif");
      document.getElementById(t.dataset.ke).classList.add("aktif");
    });
  });

  // ---------- Data sayur (bisa ditambah lewat tombol +) ----------
  var KUNCI = "smartharvest-sayur";
  var sayur = [
    { id: "cabai",    nama: "Cabai merah", sisa: 3, total: 7, harga: 48000 },
    { id: "bayam",    nama: "Bayam",       sisa: 4, total: 5, harga: 10000 },
    { id: "sawi",     nama: "Sawi",        sisa: 6, total: 9, harga: 14000 },
    { id: "kangkung", nama: "Kangkung",    sisa: 2, total: 6, harga: 15000 }
  ];

  try {
    var tersimpan = JSON.parse(localStorage.getItem(KUNCI));
    if (Array.isArray(tersimpan) && tersimpan.length) sayur = tersimpan;
  } catch (e) {}

  function simpan() {
    try { localStorage.setItem(KUNCI, JSON.stringify(sayur)); } catch (e) {}
  }

  function esc(s) {
    return String(s).replace(/[&<>"']/g, function (c) {
      return { "&": "&amp;", "<": "&lt;", ">": "&gt;", '"': "&quot;", "'": "&#39;" }[c];
    });
  }

  function rupiah(n) { return "Rp " + Number(n).toLocaleString("id-ID"); }

  function status(s) {
    var r = s.total > 0 ? s.sisa / s.total : 0;
    if (s.sisa <= 0 || r <= 0.3) return { teks: "Kirim hari ini", kelas: "m", bar: "kritis" };
    if (r <= 0.5) return { teks: "Segera kirim", kelas: "w", bar: "rendah" };
    return { teks: "Aman", kelas: "", bar: "" };
  }

  var hargaDipilih = null;

  function gambarHarga() {
    var chip = document.getElementById("chipHarga");
    var box = document.getElementById("hargaTerpilih");
    if (!sayur.length) { chip.innerHTML = ""; box.innerHTML = '<span>Belum ada sayur.</span>'; return; }
    if (!sayur.some(function (s) { return s.id === hargaDipilih; })) hargaDipilih = sayur[0].id;
    chip.innerHTML = sayur.map(function (s) {
      return '<button class="chip' + (s.id === hargaDipilih ? ' aktif' : '') + '" role="tab" data-harga="' + esc(s.id) + '">' + esc(s.nama) + '</button>';
    }).join("");
    var d = sayur.filter(function (s) { return s.id === hargaDipilih; })[0];
    box.innerHTML = '<span>' + esc(d.nama) + ' per kg</span><strong>' + rupiah(d.harga) + '</strong>';
  }

  document.getElementById("chipHarga").addEventListener("click", function (e) {
    var id = e.target.getAttribute("data-harga");
    if (!id) return;
    hargaDipilih = id;
    gambarHarga();
  });

  function gambar() {
    gambarHarga();
    var g = document.getElementById("daftarGudang");
    var hp = document.getElementById("daftarHargaPasar");

    if (!sayur.length) {
      g.innerHTML = '<p class="kosong">Belum ada sayur. Tekan tombol + untuk menambah.</p>';
      hp.innerHTML = "";
      return;
    }

    g.innerHTML = sayur.map(function (s, i) {
      var st = status(s);
      var persen = Math.min(100, Math.max(0, Math.round(s.sisa / s.total * 100)));
      var no = (i + 1 < 10 ? "0" : "") + (i + 1);
      return '<div class="sayur">' +
        '<div class="atas"><span class="nama">Gudang ' + no + ': ' + esc(s.nama) + '</span>' +
        '<span><span class="lencana ' + st.kelas + '">' + st.teks + '</span> ' +
        '<button class="hapus" data-hapus="' + esc(s.id) + '" aria-label="Hapus ' + esc(s.nama) + '">&times;</button></span></div>' +
        '<div class="info">Sisa umur simpan ' + s.sisa + ' dari ' + s.total + ' hari</div>' +
        '<div class="bar"><i class="' + st.bar + '" style="width:' + persen + '%"></i></div></div>';
    }).join("");

    var baris = sayur.map(function (s) {
      return '<div class="baris"><span>' + esc(s.nama) + '</span><b>' + rupiah(s.harga) + '/kg</b></div>';
    }).join("");
    hp.innerHTML = baris;
  }

  document.getElementById("daftarGudang").addEventListener("click", function (e) {
    var id = e.target.getAttribute("data-hapus");
    if (!id) return;
    sayur = sayur.filter(function (s) { return s.id !== id; });
    simpan(); gambar();
  });

  // ---------- Form tambah sayur ----------
  var form = document.getElementById("formTambah");
  var pesan = document.getElementById("pesanForm");

  function tutupForm() {
    form.classList.remove("buka");
    ["fNama", "fSisa", "fTotal", "fHarga"].forEach(function (i) { document.getElementById(i).value = ""; });
    pesan.textContent = "";
  }

  document.getElementById("bukaForm").addEventListener("click", function () {
    form.classList.toggle("buka");
    if (form.classList.contains("buka")) document.getElementById("fNama").focus();
  });
  document.getElementById("batalSayur").addEventListener("click", tutupForm);

  document.getElementById("simpanSayur").addEventListener("click", function () {
    var nama = document.getElementById("fNama").value.trim();
    var sisa = parseInt(document.getElementById("fSisa").value, 10);
    var total = parseInt(document.getElementById("fTotal").value, 10);
    var harga = parseInt(document.getElementById("fHarga").value, 10);

    if (!nama) { pesan.textContent = "Isi nama sayur dulu."; return; }
    if (isNaN(total) || total < 1) { pesan.textContent = "Isi total umur simpan (minimal 1 hari)."; return; }
    if (isNaN(sisa) || sisa < 0) { pesan.textContent = "Isi sisa umur simpan (0 atau lebih)."; return; }
    if (sisa > total) { pesan.textContent = "Sisa umur tidak boleh lebih besar dari total."; return; }
    if (isNaN(harga) || harga < 0) { harga = 0; }

    sayur.push({ id: "s" + Date.now(), nama: nama, sisa: sisa, total: total, harga: harga });
    simpan(); gambar(); tutupForm();
  });

  gambar();

  // ---------- Tampilan alarm gas ----------
  function updateTampilanGas(n, dariSensor) {
    n = Math.min(4095, Math.max(0, Number(n) || 0));
    var isi = document.getElementById("isiGas");
    var kartu = document.getElementById("kartuAlarm");
    var slider = document.getElementById("gas");

    if (slider) slider.value = n;
    document.getElementById("nilaiGas").textContent = n;
    isi.style.width = (n / 4095 * 100) + "%";

    if (n > 1800) {
      isi.className = "tinggi";
      kartu.className = "card bahaya";
      document.getElementById("judulAlarm").textContent = "Risiko pembusukan";
      document.getElementById("isiAlarm").textContent = "Gas tinggi. LED dan buzzer menyala, segera keluarkan stok.";
    } else {
      isi.className = "";
      kartu.className = "card";
      document.getElementById("judulAlarm").textContent = "Kondisi aman";
      document.getElementById("isiAlarm").textContent = dariSensor
        ? "Gas normal. LED dan buzzer mati."
        : "LED dan buzzer mati.";
    }
  }

  var gas = document.getElementById("gas");
  gas.addEventListener("input", function () {
    updateTampilanGas(gas.value, false);
  });

  document.getElementById("setujui").addEventListener("click", function () {
    document.getElementById("hasilSetuju").style.display = "block";
  });

  // ---------- Koneksi MQTT WebSockets ke HiveMQ ----------
  const MQTT_PREFIX = "smartharvest-tim1";
  const WS_URL = "wss://broker.hivemq.com:8884/mqtt";
  const statusEl = document.getElementById("statusKoneksi");
  const catatanEl = document.getElementById("catatanBawah");

  const client = mqtt.connect(WS_URL, {
    clientId: "smartharvest-web-" + Math.random().toString(16).slice(2, 8),
    clean: true,
    reconnectPeriod: 3000
  });

  client.on("connect", function () {
    statusEl.textContent = "Lahan 01 dan Gudang 01 • Terhubung MQTT";
    client.subscribe([
      MQTT_PREFIX + "/lahan/data",
      MQTT_PREFIX + "/lahan/alert",
      MQTT_PREFIX + "/gudang/data",
      MQTT_PREFIX + "/gudang/alert"
    ]);
  });

  client.on("offline", function () {
    statusEl.textContent = "Lahan 01 dan Gudang 01 • Koneksi terputus";
  });

  client.on("error", function () {
    statusEl.textContent = "Lahan 01 dan Gudang 01 • Gangguan koneksi";
  });

  client.on("message", function (topic, message) {
    var data;
    try {
      data = JSON.parse(message.toString());
    } catch (e) {
      return;
    }

    catatanEl.textContent = "Data real-time aktif (" + new Date().toLocaleTimeString("id-ID") + ")";

    // 1. Data Lahan
    if (topic === MQTT_PREFIX + "/lahan/data") {
      var suhu = Number(data.suhu);
      var lembap = Number(data.kelembapan);
      var tanah = Number(data.tanah);
      var hujan = Boolean(data.hujan);

      if (!Number.isNaN(suhu)) {
        document.getElementById("berandaSuhu").innerHTML = suhu.toFixed(0) + "&deg;C";
        document.getElementById("lahanSuhu").innerHTML = suhu.toFixed(0) + "&deg;C";
      }
      if (!Number.isNaN(lembap)) {
        document.getElementById("lahanLembap").textContent = lembap.toFixed(0) + "%";
      }
      if (!Number.isNaN(tanah)) {
        var tanahBulat = Math.min(100, Math.max(0, Math.round(tanah)));
        document.getElementById("berandaTanah").textContent = tanahBulat + "%";
        document.getElementById("lahanTanah").textContent = tanahBulat + "%";
        document.getElementById("barTanah").style.width = tanahBulat + "%";
      }
      document.getElementById("lahanHujan").textContent = hujan ? "Ya" : "Tidak";
    }

    // 2. Alert Lahan
    if (topic === MQTT_PREFIX + "/lahan/alert" && data.pesan) {
      document.getElementById("isiAlertLahan").textContent = data.pesan;
      document.getElementById("judulAlertBeranda").textContent = "Peringatan Lahan 01";
      document.getElementById("isiAlertBeranda").textContent = data.pesan;
    }

    // 3. Data Gudang
    if (topic === MQTT_PREFIX + "/gudang/data") {
      var suhuG = Number(data.suhu);
      var lembapG = Number(data.kelembapan);
      var pendinginAktif = Boolean(data.pendingin);
      var sisaHari = Number(data.sisa_hari);

      if (!Number.isNaN(suhuG)) {
        document.getElementById("gudangSuhu").innerHTML = suhuG.toFixed(0) + "&deg;C";
      }
      if (!Number.isNaN(lembapG)) {
        document.getElementById("gudangLembap").textContent = lembapG.toFixed(0) + "%";
      }

      var badgePendingin = document.getElementById("statusPendingin");
      badgePendingin.textContent = pendinginAktif ? "Aktif" : "Mati";
      badgePendingin.className = pendinginAktif ? "lencana" : "lencana w";

      if (data.gas !== undefined) {
        updateTampilanGas(data.gas, true);
      }

      // Data sensor sisa_hari berlaku untuk Gudang 01 (cabai merah)
      if (!Number.isNaN(sisaHari) && sisaHari >= 0) {
        document.getElementById("gudangSisaHari").textContent = sisaHari + " hari";
        var cabai = sayur.filter(function (s) { return s.id === "cabai"; })[0];
        if (cabai) {
          cabai.sisa = Math.min(sisaHari, cabai.total);
          gambar();
        }
      }
    }

    // 4. Alert Gudang
    if (topic === MQTT_PREFIX + "/gudang/alert" && data.pesan) {
      var kartu = document.getElementById("kartuAlarm");
      kartu.className = "card bahaya";
      document.getElementById("judulAlarm").textContent = "Peringatan Gudang";
      document.getElementById("isiAlarm").textContent = data.pesan;
    }
  });
</script>
</body>
</html>
