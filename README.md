```html
<!DOCTYPE html>
<html lang="tr">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
  <meta name="theme-color" content="#ffffff">
  <title>SmartScan Terminal | NFC Kart Okuyucu</title>

  <style>
    /* Temel Ayarlar ve Değişkenler (Dışarıdan font veya CSS yüklenmez) */
    :root {
      --brand-green: #82E600;
      --brand-green-hover: #73CC00;
      --brand-navy: #16222F;
      --bg-light: #F9FAFB;
      --border-color: #E5E7EB;
      --text-main: #111827;
      --text-muted: #6B7280;
      --white: #ffffff;
      --danger: #EF4444;
      --warning-bg: #FFFBEB;
      --warning-border: #FDE68A;
      --warning-text: #92400E;
    }

    * {
      box-sizing: border-box;
      -webkit-tap-highlight-color: transparent;
    }

    body, html {
      margin: 0;
      padding: 0;
      background-color: var(--bg-light);
      color: var(--text-main);
      /* Sistem fontları kullanılır, dışarıdan font indirilmez */
      font-family: system-ui, -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, Helvetica, Arial, sans-serif;
      -webkit-user-select: none;
      user-select: none;
      height: 100vh;
      overflow: hidden;
    }

    /* Düzen ve Yapı Sınıfları */
    .app-container {
      display: flex;
      flex-direction: column;
      max-width: 450px;
      margin: 0 auto;
      height: 100vh;
      background-color: var(--bg-light);
      box-shadow: 0 0 20px rgba(0,0,0,0.05);
      position: relative;
      border-left: 1px solid var(--border-color);
      border-right: 1px solid var(--border-color);
    }

    header {
      background-color: var(--white);
      border-bottom: 1px solid #D1FAE5;
      padding: 16px;
      display: flex;
      justify-content: space-between;
      align-items: center;
      position: sticky;
      top: 0;
      z-index: 30;
      box-shadow: 0 1px 2px rgba(0,0,0,0.05);
    }

    main {
      flex-grow: 1;
      padding: 16px;
      overflow-y: auto;
      padding-bottom: 80px; /* Alt menü için boşluk */
      display: flex;
      flex-direction: column;
      gap: 16px;
    }

    /* Tipografi ve Renkler */
    .font-bold { font-weight: 700; }
    .font-mono { font-family: ui-monospace, SFMono-Regular, Menlo, Monaco, Consolas, monospace; }
    .text-xs { font-size: 0.75rem; }
    .text-sm { font-size: 0.875rem; }
    .text-lg { font-size: 1.125rem; }
    .text-2xl { font-size: 1.5rem; }
    .text-center { text-align: center; }
    .text-muted { color: var(--text-muted); }
    .text-brand { color: var(--brand-navy); }
    
    /* UI Bileşenleri */
    .card {
      background-color: var(--white);
      border-radius: 24px;
      padding: 24px;
      box-shadow: 0 4px 6px -1px rgba(0, 0, 0, 0.1);
      border: 1px solid var(--border-color);
      transition: all 0.3s ease;
    }

    .btn {
      width: 100%;
      padding: 16px;
      border-radius: 16px;
      border: none;
      font-weight: 800;
      font-size: 1rem;
      cursor: pointer;
      text-transform: uppercase;
      letter-spacing: 0.05em;
      transition: background-color 0.2s, transform 0.1s;
      display: flex;
      justify-content: center;
      align-items: center;
      gap: 8px;
    }
    
    .btn:active { transform: scale(0.98); }
    
    .btn-primary {
      background-color: var(--brand-green);
      color: var(--brand-navy);
      box-shadow: 0 4px 14px rgba(130, 230, 0, 0.3);
    }
    .btn-primary:hover { background-color: var(--brand-green-hover); }
    .btn-primary:disabled {
      background-color: #E5E7EB;
      color: #9CA3AF;
      box-shadow: none;
      cursor: not-allowed;
    }

    .btn-danger {
      background-color: var(--danger);
      color: white;
      box-shadow: 0 4px 14px rgba(239, 68, 68, 0.3);
    }

    .btn-outline {
      background-color: var(--white);
      border: 1px solid var(--border-color);
      color: var(--text-main);
    }

    .uid-display {
      background-color: var(--brand-navy);
      color: var(--brand-green);
      padding: 20px 12px;
      border-radius: 16px;
      font-size: 1.5rem;
      font-weight: 800;
      letter-spacing: 0.1em;
      word-break: break-all;
      margin: 16px 0;
      box-shadow: inset 0 2px 4px rgba(0,0,0,0.5);
    }

    /* Alt Menü (Bottom Nav) */
    .bottom-nav {
      position: absolute;
      bottom: 0;
      left: 0;
      right: 0;
      background-color: var(--white);
      border-top: 1px solid var(--border-color);
      display: flex;
      z-index: 30;
    }

    .nav-item {
      flex: 1;
      display: flex;
      flex-direction: column;
      align-items: center;
      padding: 12px 0;
      color: var(--text-muted);
      background: none;
      border: none;
      cursor: pointer;
      font-size: 0.7rem;
      font-weight: 700;
      transition: color 0.2s;
    }
    .nav-item svg { width: 24px; height: 24px; margin-bottom: 4px; }
    .nav-item.active { color: var(--brand-green-hover); }

    /* Listeler ve Kayıtlar */
    .history-list {
      list-style: none;
      padding: 0;
      margin: 0;
      background: var(--white);
      border-radius: 16px;
      overflow: hidden;
      border: 1px solid var(--border-color);
    }
    .history-item {
      padding: 14px;
      border-bottom: 1px solid var(--border-color);
    }
    .history-item:last-child { border-bottom: none; }
    .history-item-header { display: flex; justify-content: space-between; align-items: flex-start; }
    .history-uid { color: var(--brand-navy); font-weight: bold; }
    .history-time { font-size: 0.65rem; background: var(--bg-light); padding: 2px 6px; border-radius: 10px; }
    
    /* Uyarı Mesajları */
    .alert {
      padding: 14px;
      border-radius: 16px;
      font-size: 0.8rem;
      margin-bottom: 10px;
    }
    .alert-warning { background-color: var(--warning-bg); border: 1px solid var(--warning-border); color: var(--warning-text); }
    .alert-danger { background-color: #FEF2F2; border: 1px solid #F87171; color: #991B1B; }

    /* Modallar */
    .modal-overlay {
      position: absolute;
      top: 0; left: 0; right: 0; bottom: 0;
      background-color: rgba(0,0,0,0.5);
      backdrop-filter: blur(4px);
      display: none;
      justify-content: center;
      align-items: center;
      z-index: 50;
      padding: 16px;
    }
    .modal-content {
      background: var(--white);
      padding: 24px;
      border-radius: 24px;
      width: 100%;
      max-width: 320px;
    }
    .input-field {
      width: 100%;
      padding: 12px;
      border: 1px solid var(--border-color);
      border-radius: 12px;
      font-family: monospace;
      font-size: 1rem;
      margin: 12px 0;
      text-transform: uppercase;
      outline: none;
    }
    .input-field:focus { border-color: var(--brand-green); }

    /* Animasyonlar */
    @keyframes greenPulse {
      0% { box-shadow: 0 0 0 0 rgba(130, 230, 0, 0.6); transform: scale(0.97); }
      70% { box-shadow: 0 0 0 22px rgba(130, 230, 0, 0); transform: scale(1.03); }
      100% { box-shadow: 0 0 0 0 rgba(130, 230, 0, 0); transform: scale(0.97); }
    }
    .green-pulse { animation: greenPulse 1.8s infinite ease-in-out; }
    
    @keyframes ping {
      75%, 100% { transform: scale(2); opacity: 0; }
    }
    .animate-ping { animation: ping 1.5s cubic-bezier(0, 0, 0.2, 1) infinite; }
    
    .fade-in { animation: fadeIn 0.3s ease-out forwards; }
    @keyframes fadeIn {
      from { opacity: 0; transform: translateY(5px); }
      to { opacity: 1; transform: translateY(0); }
    }

    .hidden { display: none !important; }
    .flex { display: flex; }
    .items-center { align-items: center; }
    .justify-between { justify-content: space-between; }
    .space-x-2 > * + * { margin-left: 8px; }
  </style>
</head>
<body>
  <div class="app-container">
    
    <!-- Üst Başlık -->
    <header>
      <div class="flex items-center space-x-2">
        <div style="background-color: var(--brand-navy); color: var(--brand-green); padding: 8px; border-radius: 12px;">
          <svg width="24" height="24" fill="none" stroke="currentColor" viewBox="0 0 24 24">
            <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M12 4v1m6 11h2m-6 0h-2v4m0-11v3m0 0h.01M12 12h4.01M16 20h4M4 12h4m12 0h.01M5 8h2a1 1 0 001-1V5a1 1 0 00-1-1H5a1 1 0 00-1 1v2a1 1 0 001 1zm14 0h2a1 1 0 001-1V5a1 1 0 00-1-1h-2a1 1 0 00-1 1v2a1 1 0 001 1zM5 20h2a1 1 0 001-1v-2a1 1 0 00-1-1H5a1 1 0 00-1 1v2a1 1 0 001 1z" />
          </svg>
        </div>
        <div>
          <h1 style="margin:0; font-size: 1.1rem; font-weight: 800; color: var(--brand-navy);">SmartScan Terminal</h1>
          <p style="margin:0; font-size: 0.65rem; color: var(--brand-green-hover); font-weight: 700;">Mifare & NFC Kart Okuyucu</p>
        </div>
      </div>
      <div id="statusIndicator" style="display: flex; align-items: center; gap: 6px; background: #F3F4F6; padding: 4px 10px; border-radius: 20px; font-size: 0.65rem; font-weight: bold;">
        <div id="statusDot" style="width: 8px; height: 8px; border-radius: 50%; background-color: #9CA3AF;"></div>
        <span id="statusText" style="color: #6B7280;">HAZIR</span>
      </div>
    </header>

    <!-- Ana İçerik -->
    <main id="mainContent">
      <!-- Uyarı Alanları -->
      <div id="iosWarning" class="alert alert-warning hidden">
        <strong> iOS Kısıtlaması Uyarı:</strong><br>Apple, Web NFC'yi engellemektedir. Cihazınızda test için aşağıdaki "Manuel UID Gir" butonunu kullanabilirsiniz.
      </div>
      <div id="nfcWarning" class="alert alert-warning hidden">
        <strong>⚠️ Web NFC Donanım Uyarısı:</strong><br>Bu teknoloji Android cihazlarda Chrome tarayıcısı ve HTTPS bağlantısı gerektirir.
      </div>
      <div id="deniedWarning" class="alert alert-danger hidden">
        <strong>🚫 Chrome NFC İzni Engellenmiş:</strong><br>Adres çubuğundaki Kilit İkonuna basıp NFC'ye İzin Ver yapınız.
      </div>

      <!-- Tarama Sekmesi (Varsayılan) -->
      <div id="tabScan" class="fade-in" style="display: flex; flex-direction: column; gap: 16px; flex-grow: 1;">
        
        <!-- Okuma Kartı -->
        <div class="card" id="scanCard" style="margin: auto 0; text-align: center;">
          <div style="display: flex; justify-content: center; margin: 16px 0;">
            <div id="scanIconContainer" style="padding: 24px; border-radius: 50%; background-color: #F3F4F6; color: #9CA3AF;">
              <svg width="48" height="48" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                <path stroke-linecap="round" stroke-linejoin="round" stroke-width="1.5" d="M8.111 16.404a5.5 5.5 0 017.778 0M12 20h.01m-7.08-7.071c3.904-3.905 10.236-3.905 14.14 0M1.394 9.393c5.857-5.857 15.355-5.857 21.213 0" />
              </svg>
            </div>
          </div>
          
          <div class="flex justify-between items-center" style="margin-bottom: 8px;">
            <span style="font-size: 0.7rem; font-weight: bold; color: var(--text-muted); text-transform: uppercase;">Fiziksel Kart ID (UID)</span>
            <span id="lastReadBadge" class="hidden" style="font-size: 0.6rem; font-family: monospace; font-weight: bold; background: #ECFDF5; color: #047857; padding: 2px 8px; border-radius: 12px; border: 1px solid #A7F3D0;"></span>
          </div>

          <div class="uid-display" id="uidText">BEKLEMEDE</div>

          <div id="messageContainer" class="hidden" style="margin-top: 16px; padding-top: 16px; border-top: 1px solid var(--border-color); text-align: left;">
            <div style="font-size: 0.65rem; font-weight: bold; color: var(--text-muted); text-transform: uppercase; text-align: center; margin-bottom: 8px;">Çip İçerik Verisi</div>
            <div id="messageContent" style="background-color: #F8FAFC; padding: 12px; border-radius: 12px; font-family: monospace; font-size: 0.75rem; color: var(--brand-navy); word-break: break-all;">
            </div>
          </div>
        </div>

        <!-- Kontrol Butonları -->
        <div style="display: flex; flex-direction: column; gap: 12px;">
          <div id="activeScanBanner" class="hidden" style="background-color: #ECFDF5; border: 1px solid #A7F3D0; color: #065F46; font-size: 0.75rem; padding: 14px; border-radius: 16px; font-weight: bold; text-align: center; display: flex; align-items: center; justify-content: center; gap: 8px;">
            <div class="animate-ping" style="width: 10px; height: 10px; border-radius: 50%; background-color: #059669;"></div>
            TARAMA AKTİF (Kartı Kameraya Yaklaştırın)
          </div>

          <button id="btnStartScan" class="btn btn-primary">Sürekli Taramayı Başlat</button>
          <button id="btnStopScan" class="btn btn-danger hidden">Taramayı Durdur</button>
          <button id="btnOpenManual" class="btn btn-outline" style="font-size: 0.8rem; padding: 12px;">
            <svg width="16" height="16" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M11 5H6a2 2 0 00-2 2v11a2 2 0 002 2h11a2 2 0 002-2v-5m-1.414-9.414a2 2 0 112.828 2.828L11.828 15H9v-2.828l8.586-8.586z"></path></svg>
            Manuel UID Gir
          </button>
        </div>

        <!-- İpuçları -->
        <button id="btnTroubleshoot" style="background: white; border: 1px solid var(--border-color); border-radius: 16px; padding: 12px 16px; text-align: left; font-size: 0.75rem; font-weight: bold; color: var(--text-muted); display: flex; justify-content: space-between; cursor: pointer;">
          <span>💡 Kart okuma ipuçları</span>
          <span id="troubleshootIcon" style="color: var(--brand-green-hover);">▼</span>
        </button>
        <div id="troubleshootContent" class="hidden" style="background: white; border: 1px solid var(--border-color); border-radius: 16px; padding: 16px; font-size: 0.75rem; color: var(--text-main); margin-top: -8px;">
          <div style="margin-bottom: 6px;">• <strong>Kılıf:</strong> Kalın plastik veya mıknatıslı kılıfları çıkarınız.</div>
          <div style="margin-bottom: 6px;">• <strong>Konum:</strong> Kartı telefonun arkasına 1-2 saniye sabitleyin.</div>
          <div>• <strong>Frekans:</strong> 13.56 MHz Mifare çipleri desteklenir.</div>
        </div>
      </div>

      <!-- Geçmiş Sekmesi -->
      <div id="tabHistory" class="fade-in hidden" style="display: flex; flex-direction: column; gap: 12px; flex-grow: 1;">
        <div class="flex justify-between items-center">
          <div>
            <h2 style="margin: 0; font-size: 1rem; font-weight: bold;">Okuma Geçmişi</h2>
            <p id="historyCount" style="margin: 2px 0 0 0; font-size: 0.7rem; color: var(--text-muted);">Toplam Kayıt: 0</p>
          </div>
          <button id="btnDownload" class="btn btn-primary" style="width: auto; padding: 8px 16px; font-size: 0.75rem; border-radius: 12px;" disabled>
            <svg width="16" height="16" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M4 16v1a3 3 0 003 3h10a3 3 0 003-3v-1m-4-4l-4 4m0 0l-4-4m4 4V4"></path></svg>
            İndir
          </button>
        </div>
        
        <div id="emptyHistory" style="flex-grow: 1; display: flex; flex-direction: column; align-items: center; justify-content: center; color: #D1D5DB; padding: 40px 0;">
          <svg width="64" height="64" fill="none" stroke="currentColor" viewBox="0 0 24 24" style="margin-bottom: 12px;">
            <path stroke-linecap="round" stroke-linejoin="round" stroke-width="1" d="M9 12h6m-6 4h6m2 5H7a2 2 0 01-2-2V5a2 2 0 012-2h5.586a1 1 0 01.707.293l5.414 5.414a1 1 0 01.293.707V19a2 2 0 01-2 2z"></path>
          </svg>
          <span style="font-size: 0.8rem;">Henüz kayıt yok.</span>
        </div>

        <ul id="historyList" class="history-list hidden">
          <!-- Javascript ile doldurulacak -->
        </ul>
      </div>
    </main>

    <!-- Manuel Giriş Modalı -->
    <div id="manualModal" class="modal-overlay">
      <div class="modal-content fade-in">
        <h3 style="margin: 0 0 16px 0; display: flex; align-items: center; font-size: 1rem; color: var(--brand-navy);">
          <span style="width:10px; height:10px; border-radius:50%; background-color:var(--brand-green); margin-right:8px;"></span>
          Manuel Kart UID Gir
        </h3>
        <form id="manualForm">
          <label style="font-size: 0.75rem; font-weight: bold; color: var(--text-muted);">Kart UID (Örn: 04:A2:8B):</label>
          <input type="text" id="manualInput" class="input-field" placeholder="XX:XX:XX:XX" autocomplete="off" required>
          <div style="display: flex; gap: 8px; margin-top: 16px;">
            <button type="button" id="btnCancelManual" class="btn btn-outline" style="padding: 12px; font-size: 0.8rem; border-radius: 12px;">İptal</button>
            <button type="submit" class="btn btn-primary" style="padding: 12px; font-size: 0.8rem; border-radius: 12px;">Kaydet</button>
          </div>
        </form>
      </div>
    </div>

    <!-- Alt Navigasyon -->
    <nav class="bottom-nav">
      <button id="navScan" class="nav-item active">
        <svg fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2.2" d="M12 4v1m6 11h2m-6 0h-2v4m0-11v3m0 0h.01M12 12h4.01M16 20h4M4 12h4m12 0h.01M5 8h2a1 1 0 001-1V5a1 1 0 00-1-1H5a1 1 0 00-1 1v2a1 1 0 001 1zm14 0h2a1 1 0 001-1V5a1 1 0 00-1-1h-2a1 1 0 00-1 1v2a1 1 0 001 1zM5 20h2a1 1 0 001-1v-2a1 1 0 00-1-1H5a1 1 0 00-1 1v2a1 1 0 001 1z"></path></svg>
        <span>Tarayıcı</span>
      </button>
      <button id="navHistory" class="nav-item" style="position: relative;">
        <svg fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="1.5" d="M9 5H7a2 2 0 00-2 2v12a2 2 0 002 2h10a2 2 0 002-2V7a2 2 0 00-2-2h-2M9 5a2 2 0 022 2h2a2 2 0 022-2M9 5a2 2 0 012-2h2a2 2 0 012 2m-3 7h3m-3 4h3m-6-4h.01M9 16h.01"></path></svg>
        <span>Kayıtlar</span>
        <span id="navHistoryBadge" class="hidden" style="position: absolute; top: 6px; right: 25px; background: var(--brand-green); color: var(--brand-navy); font-size: 0.55rem; padding: 2px 5px; border-radius: 10px; font-weight: 900;">0</span>
      </button>
    </nav>

  </div>

  <script>
    // --- DURUM YÖNETİMİ (STATE) ---
    const state = {
      isScanning: false,
      hasNfcSupport: null,
      history: [],
      scanController: null,
      audioUnlocked: false
    };

    // --- DOM ELEMENTLERİ ---
    const els = {
      // Uyarılar
      iosWarning: document.getElementById('iosWarning'),
      nfcWarning: document.getElementById('nfcWarning'),
      deniedWarning: document.getElementById('deniedWarning'),
      // Status
      statusIndicator: document.getElementById('statusIndicator'),
      statusDot: document.getElementById('statusDot'),
      statusText: document.getElementById('statusText'),
      // Tarama Sekmesi
      tabScan: document.getElementById('tabScan'),
      scanCard: document.getElementById('scanCard'),
      scanIconContainer: document.getElementById('scanIconContainer'),
      uidText: document.getElementById('uidText'),
      lastReadBadge: document.getElementById('lastReadBadge'),
      messageContainer: document.getElementById('messageContainer'),
      messageContent: document.getElementById('messageContent'),
      // Butonlar
      activeScanBanner: document.getElementById('activeScanBanner'),
      btnStartScan: document.getElementById('btnStartScan'),
      btnStopScan: document.getElementById('btnStopScan'),
      btnOpenManual: document.getElementById('btnOpenManual'),
      btnTroubleshoot: document.getElementById('btnTroubleshoot'),
      troubleshootContent: document.getElementById('troubleshootContent'),
      troubleshootIcon: document.getElementById('troubleshootIcon'),
      // Geçmiş Sekmesi
      tabHistory: document.getElementById('tabHistory'),
      historyCount: document.getElementById('historyCount'),
      btnDownload: document.getElementById('btnDownload'),
      emptyHistory: document.getElementById('emptyHistory'),
      historyList: document.getElementById('historyList'),
      // Navigasyon
      navScan: document.getElementById('navScan'),
      navHistory: document.getElementById('navHistory'),
      navHistoryBadge: document.getElementById('navHistoryBadge'),
      // Modal
      manualModal: document.getElementById('manualModal'),
      manualForm: document.getElementById('manualForm'),
      manualInput: document.getElementById('manualInput'),
      btnCancelManual: document.getElementById('btnCancelManual')
    };

    // --- BAŞLANGIÇ KONTROLLERİ ---
    function init() {
      // Cihaz ve Tarayıcı Kontrolü
      const isIOS = /iPad|iPhone|iPod/.test(navigator.userAgent) && !window.MSStream;
      if (isIOS) els.iosWarning.classList.remove('hidden');

      if ('NDEFReader' in window) {
        state.hasNfcSupport = true;
        // İzin kontrolü
        if (navigator.permissions && navigator.permissions.query) {
          navigator.permissions.query({ name: 'nfc' }).then(status => {
            if(status.state === 'denied') els.deniedWarning.classList.remove('hidden');
            status.onchange = () => {
              if(status.state === 'denied') els.deniedWarning.classList.remove('hidden');
              else els.deniedWarning.classList.add('hidden');
            };
          }).catch(e => console.log("İzin sorgusu desteklenmiyor."));
        }
      } else {
        state.hasNfcSupport = false;
        if (!isIOS) els.nfcWarning.classList.remove('hidden');
        els.btnStartScan.disabled = true;
        els.btnStartScan.innerText = "NFC Desteklenmiyor";
      }

      // Audio kilit açma (Kullanıcı etkileşimi şarttır)
      const unlockAudio = () => {
        if (state.audioUnlocked) return;
        try {
          const ctx = new (window.AudioContext || window.webkitAudioContext)();
          ctx.resume();
          state.audioUnlocked = true;
        } catch (e) {}
        window.removeEventListener('touchstart', unlockAudio);
        window.removeEventListener('click', unlockAudio);
      };
      window.addEventListener('touchstart', unlockAudio);
      window.addEventListener('click', unlockAudio);
    }

    // --- SES VE TİTREŞİM ---
    function playBeep() {
      try {
        const audioCtx = new (window.AudioContext || window.webkitAudioContext)();
        if (audioCtx.state === 'suspended') audioCtx.resume();
        const osc = audioCtx.createOscillator();
        const gain = audioCtx.createGain();
        osc.type = 'sine';
        osc.frequency.setValueAtTime(2400, audioCtx.currentTime); // Standart okuyucu frekansı
        gain.gain.setValueAtTime(0.7, audioCtx.currentTime);
        gain.gain.setValueAtTime(0.7, audioCtx.currentTime + 0.08);
        gain.gain.exponentialRampToValueAtTime(0.001, audioCtx.currentTime + 0.1);
        osc.connect(gain);
        gain.connect(audioCtx.destination);
        osc.start(audioCtx.currentTime);
        osc.stop(audioCtx.currentTime + 0.1);
      } catch (e) {}
    }

    function vibrate() {
      try { if (navigator.vibrate) navigator.vibrate([100, 40, 100]); } catch (e) {}
    }

    // --- NFC TARAMA MANTIĞI ---
    async function startScan() {
      try {
        state.isScanning = true;
        updateUIForScanning();
        
        if (state.scanController) state.scanController.abort();
        state.scanController = new AbortController();
        
        const ndef = new window.NDEFReader();
        
        ndef.addEventListener("readingerror", () => {
          alert("Kart okuma hatası oluştu. Lütfen kartı tekrar dokundurun.");
          try { if (navigator.vibrate) navigator.vibrate([150, 80, 150]); } catch (e) {}
        });
        
        ndef.addEventListener("reading", (event) => {
          const { serialNumber, message } = event;
          let realUid = serialNumber ? serialNumber.toUpperCase() : (event.id ? event.id.toUpperCase() : "UID-OKUNAMADI");
          
          vibrate();
          playBeep();
          flashSuccessUI();
          
          let decodedMessages = [];
          if (message && message.records && message.records.length > 0) {
            const decoder = new TextDecoder();
            for (const record of message.records) {
              if (record.recordType === 'text') decodedMessages.push(`Metin: ${decoder.decode(record.data)}`);
              else if (record.recordType === 'url') decodedMessages.push(`Link: ${decoder.decode(record.data)}`);
              else decodedMessages.push(`Kayıt Tipi: ${record.recordType}`);
            }
          } else {
            decodedMessages = ["Fiziksel Çip UID Okundu."];
          }
          
          processSuccessfulRead(realUid, decodedMessages);
        });
        
        await ndef.scan({ signal: state.scanController.signal });
        els.deniedWarning.classList.add('hidden');
      } catch (error) {
        state.isScanning = false;
        updateUIForScanning();
        if (error.name === 'NotAllowedError') {
          els.deniedWarning.classList.remove('hidden');
        } else if (error.name === 'NotSupportedError') {
          alert("Web NFC bu cihazda/tarayıcıda desteklenmiyor.");
        } else {
          alert(`NFC Başlatılamadı: ${error.message}`);
        }
      }
    }

    function stopScan() {
      if (state.scanController) state.scanController.abort();
      state.isScanning = false;
      updateUIForScanning();
    }

    function processSuccessfulRead(uid, messages) {
      const now = new Date();
      const timeStr = now.toLocaleTimeString('tr-TR');
      const dateStr = now.toLocaleString('tr-TR');
      const dataStr = messages.join(' | ');

      // Arayüzü Güncelle
      els.uidText.innerText = uid;
      els.uidText.style.color = "var(--brand-green)";
      els.lastReadBadge.innerText = `Son Okuma: ${timeStr}`;
      els.lastReadBadge.classList.remove('hidden');
      
      els.messageContent.innerHTML = messages.map(m => `<div>${m}</div>`).join('');
      els.messageContainer.classList.remove('hidden');

      // Geçmişe Ekle
      const record = { id: Date.now(), uid, timestamp: dateStr, data: dataStr };
      state.history.unshift(record);
      updateHistoryUI();
    }

    // --- ARAYÜZ (UI) GÜNCELLEMELERİ ---
    function updateUIForScanning() {
      if (state.isScanning) {
        // Üst Header Status
        els.statusDot.style.backgroundColor = 'var(--brand-green)';
        els.statusDot.classList.add('animate-ping');
        els.statusText.style.color = 'var(--brand-navy)';
        els.statusText.innerText = 'TARAMA AKTİF';
        
        // Kart Status
        els.scanIconContainer.style.backgroundColor = '#ECFDF5'; // Açık yeşil
        els.scanIconContainer.style.color = '#059669';
        els.scanIconContainer.style.border = '1px solid #A7F3D0';
        els.scanIconContainer.classList.add('green-pulse');
        if(els.uidText.innerText === 'BEKLEMEDE') els.uidText.innerText = 'KARTI DOKUNDURUN...';
        
        // Butonlar
        els.activeScanBanner.classList.remove('hidden');
        els.btnStartScan.classList.add('hidden');
        els.btnOpenManual.classList.add('hidden');
        els.btnStopScan.classList.remove('hidden');
      } else {
        // Üst Header Status
        els.statusDot.style.backgroundColor = '#9CA3AF';
        els.statusDot.classList.remove('animate-ping');
        els.statusText.style.color = '#6B7280';
        els.statusText.innerText = 'HAZIR';
        
        // Kart Status
        els.scanIconContainer.style.backgroundColor = '#F3F4F6';
        els.scanIconContainer.style.color = '#9CA3AF';
        els.scanIconContainer.style.border = 'none';
        els.scanIconContainer.classList.remove('green-pulse');
        if(els.uidText.innerText === 'KARTI DOKUNDURUN...') els.uidText.innerText = 'BEKLEMEDE';
        
        // Butonlar
        els.activeScanBanner.classList.add('hidden');
        els.btnStartScan.classList.remove('hidden');
        els.btnOpenManual.classList.remove('hidden');
        els.btnStopScan.classList.add('hidden');
      }
    }

    function flashSuccessUI() {
      els.scanCard.style.borderColor = 'var(--brand-green)';
      els.scanCard.style.boxShadow = '0 0 0 4px #D1FAE5';
      setTimeout(() => {
        els.scanCard.style.borderColor = 'var(--border-color)';
        els.scanCard.style.boxShadow = '0 4px 6px -1px rgba(0, 0, 0, 0.1)';
      }, 600);
    }

    function updateHistoryUI() {
      els.historyCount.innerText = `Toplam Kayıt: ${state.history.length}`;
      
      if (state.history.length > 0) {
        els.emptyHistory.classList.add('hidden');
        els.historyList.classList.remove('hidden');
        els.btnDownload.disabled = false;
        
        // Alt Navigasyon Badge
        els.navHistoryBadge.innerText = state.history.length;
        els.navHistoryBadge.classList.remove('hidden');

        // Listeyi yeniden oluştur
        els.historyList.innerHTML = state.history.map(rec => `
          <li class="history-item">
            <div class="history-item-header">
              <span class="history-uid font-mono">${rec.uid}</span>
              <span class="history-time">${rec.timestamp}</span>
            </div>
            <p style="margin: 4px 0 0 0; font-size: 0.75rem; color: var(--text-muted);">${rec.data}</p>
          </li>
        `).join('');
      } else {
        els.emptyHistory.classList.remove('hidden');
        els.historyList.classList.add('hidden');
        els.btnDownload.disabled = true;
        els.navHistoryBadge.classList.add('hidden');
      }
    }

    // --- SEKME GEÇİŞLERİ ---
    function switchTab(tab) {
      if (state.isScanning) stopScan();
      
      if (tab === 'scan') {
        els.tabScan.classList.remove('hidden');
        els.tabHistory.classList.add('hidden');
        
        els.navScan.classList.add('active');
        els.navScan.querySelector('svg').setAttribute('stroke-width', '2.2');
        els.navHistory.classList.remove('active');
        els.navHistory.querySelector('svg').setAttribute('stroke-width', '1.5');
      } else {
        els.tabScan.classList.add('hidden');
        els.tabHistory.classList.remove('hidden');
        
        els.navHistory.classList.add('active');
        els.navHistory.querySelector('svg').setAttribute('stroke-width', '2.2');
        els.navScan.classList.remove('active');
        els.navScan.querySelector('svg').setAttribute('stroke-width', '1.5');
      }
    }

    // --- EVENT LISTENER'LAR ---
    els.btnStartScan.addEventListener('click', startScan);
    els.btnStopScan.addEventListener('click', stopScan);
    
    // Alt Menü
    els.navScan.addEventListener('click', () => switchTab('scan'));
    els.navHistory.addEventListener('click', () => switchTab('history'));

    // Manuel Giriş Modalı
    els.btnOpenManual.addEventListener('click', () => {
      els.manualModal.style.display = 'flex';
      els.manualInput.focus();
    });
    els.btnCancelManual.addEventListener('click', () => {
      els.manualModal.style.display = 'none';
      els.manualInput.value = '';
    });
    els.manualForm.addEventListener('submit', (e) => {
      e.preventDefault();
      const val = els.manualInput.value.trim();
      if (!val) return;
      
      vibrate();
      playBeep();
      processSuccessfulRead(val.toUpperCase(), ["Manuel Girilen UID Kaydı"]);
      
      els.manualModal.style.display = 'none';
      els.manualInput.value = '';
    });

    // İpuçları Akordiyonu
    els.btnTroubleshoot.addEventListener('click', () => {
      const isHidden = els.troubleshootContent.classList.contains('hidden');
      if (isHidden) {
        els.troubleshootContent.classList.remove('hidden');
        els.troubleshootIcon.innerText = '▲';
      } else {
        els.troubleshootContent.classList.add('hidden');
        els.troubleshootIcon.innerText = '▼';
      }
    });

    // CSV İndirme
    els.btnDownload.addEventListener('click', () => {
      if (state.history.length === 0) return;
      const headers = "Tarih & Saat,Kart UID,İçerik Verisi\n";
      const csvContent = state.history.map(r => `"${r.timestamp.replace(/"/g, '""')}","${r.uid.replace(/"/g, '""')}","${r.data.replace(/"/g, '""')}"`).join("\n");
      
      const blob = new Blob(["\uFEFF" + headers + csvContent], { type: 'text/csv;charset=utf-8;' });
      const url = URL.createObjectURL(blob);
      const link = document.createElement("a");
      link.setAttribute("href", url);
      link.setAttribute("download", `NFC_Kayitlari_${new Date().getTime()}.csv`);
      document.body.appendChild(link);
      link.click();
      document.body.removeChild(link);
      setTimeout(() => URL.revokeObjectURL(url), 100);
    });

    // Uygulamayı Başlat
    document.addEventListener('DOMContentLoaded', init);

  </script>
</body>
</html>
```
