import React, { useState, useEffect } from 'react';

export default function App() {
  const [hasNfcSupport, setHasNfcSupport] = useState(null);
  const [isScanning, setIsScanning] = useState(false);
  const [cardUid, setCardUid] = useState(null);
  const [readError, setReadError] = useState(null);
  const [messages, setMessages] = useState([]);
  const [scanHistory, setScanHistory] = useState([]);
  
  // Yeni State: Aktif sekme kontrolü ('scan' veya 'history')
  const [activeTab, setActiveTab] = useState('scan');
  
  const scanController = React.useRef(null);

  useEffect(() => {
    if ('NDEFReader' in window) {
      setHasNfcSupport(true);
    } else {
      setHasNfcSupport(false);
    }
    
    return () => {
      if (scanController.current) {
        scanController.current.abort();
      }
    };
  }, []);

  const handleTabChange = (tab) => {
    // Sayfa değiştiğinde tarama açıksa kapat (Güvenlik)
    if (isScanning) {
      cancelNfcScan();
    }
    setActiveTab(tab);
  };

  const startNfcScan = async () => {
    try {
      setReadError(null);
      setCardUid(null);
      setMessages([]);
      setIsScanning(true);

      scanController.current = new AbortController();
      const ndef = new window.NDEFReader();
      
      await ndef.scan({ signal: scanController.current.signal });

      ndef.onreadingerror = (event) => {
        console.warn("NFC Reading Error:", event);
        setReadError("Kart okunamadı. Lütfen telefonu sabit tutarak tekrar yaklaştırın.");
      };

      ndef.onreading = (event) => {
        const { serialNumber, message } = event;
        
        const currentUid = serialNumber ? serialNumber.toUpperCase() : "UID Okunamadı";
        setCardUid(currentUid);
        
        let decodedMessages = [];
        if (message && message.records && message.records.length > 0) {
            const decoder = new TextDecoder();
            for (const record of message.records) {
                if (record.recordType === 'text') {
                     decodedMessages.push(`Metin: ${decoder.decode(record.data)}`);
                } else if (record.recordType === 'url') {
                     decodedMessages.push(`Link: ${decoder.decode(record.data)}`);
                } else {
                    decodedMessages.push(`Kayıt Tipi: ${record.recordType}`);
                }
            }
            setMessages(decodedMessages);
        } else {
            decodedMessages = ["Kartın içinde metin/url verisi bulunamadı."];
            setMessages(decodedMessages);
        }
        
        const newRecord = {
          id: Date.now(),
          uid: currentUid,
          timestamp: new Date().toLocaleString('tr-TR'),
          data: decodedMessages.join(' | ')
        };
        
        // Okunan kartı geçmişe ekle
        setScanHistory(prevHistory => [newRecord, ...prevHistory]);
        cancelNfcScan();
      };
      
    } catch (error) {
       console.error("NFC Scan Exception:", error);
       setIsScanning(false);
       
       if (error.name === 'NotAllowedError') {
           setReadError("NFC izni reddedildi. Lütfen tarayıcı ayarlarından izin verin.");
       } else if (error.name === 'NotSupportedError') {
           setReadError("Web NFC bu cihazda desteklenmiyor.");
           setHasNfcSupport(false);
       } else {
           setReadError(`Hata: ${error.message}`);
       }
    }
  };

  const cancelNfcScan = () => {
    if (scanController.current) {
      scanController.current.abort();
    }
    setIsScanning(false);
  };

  const downloadHistory = () => {
    if (scanHistory.length === 0) return;
    
    // Güvenli CSV oluşturma (Client-side)
    const headers = "Tarih & Saat,Kart UID,İçerik Verisi\n";
    
    const csvContent = scanHistory.map(record => {
      // Çift tırnakları escape et (Güvenlik)
      const safeTime = record.timestamp.replace(/"/g, '""');
      const safeUid = record.uid.replace(/"/g, '""');
      const safeData = record.data.replace(/"/g, '""');
      return `"${safeTime}","${safeUid}","${safeData}"`;
    }).join("\n");
    
    const blob = new Blob(["\uFEFF" + headers + csvContent], { type: 'text/csv;charset=utf-8;' }); // \uFEFF for Excel UTF-8 BOM
    const url = URL.createObjectURL(blob);
    const link = document.createElement("a");
    link.setAttribute("href", url);
    link.setAttribute("download", `NFC_Kayitlari_${new Date().getTime()}.csv`);
    document.body.appendChild(link);
    link.click();
    document.body.removeChild(link);
    
    // Güvenlik için oluşturulan Blob URL'sini hafızadan temizle
    setTimeout(() => URL.revokeObjectURL(url), 100);
  };

  return (
    <div className="min-h-screen bg-gray-50 flex flex-col font-sans max-w-md mx-auto w-full shadow-2xl relative pb-16">
      
      {/* Üst Başlık */}
      <header className="bg-blue-600 text-white p-5 shadow-md text-center shrink-0 rounded-b-2xl z-10 relative">
        <h1 className="text-xl font-bold mb-1">Mifare Kart Okuyucu</h1>
        <p className="text-blue-200 text-xs">Yüksek Güvenlikli Mobil Terminal</p>
      </header>

      {/* Ana İçerik Alanı (Sekmelere göre değişir) */}
      <main className="flex-grow p-5 flex flex-col overflow-y-auto">
        
        {/* Hata Uyarıları (Her iki sekmede de görülebilir) */}
        {hasNfcSupport === false && (
          <div className="bg-red-100 border border-red-400 text-red-700 px-4 py-3 rounded-lg mb-4 text-center text-sm shadow-sm">
            <strong className="font-bold block mb-1">NFC Desteklenmiyor!</strong>
            Bu cihaz veya tarayıcı Web NFC'yi desteklemiyor. Lütfen bir Android cihazda Chrome kullanın.
          </div>
        )}

        {/* ================= SEÇENEK 1: TARAYICI EKRANI ================= */}
        {activeTab === 'scan' && (
          <div className="flex flex-col items-center justify-center flex-grow fade-in h-full">
            
            {readError && (
                <div className="bg-yellow-100 border border-yellow-400 text-yellow-800 px-4 py-3 rounded-lg mb-6 w-full text-center text-sm shadow-sm animate-pulse">
                  {readError}
                </div>
            )}

            {/* Kart Verisi Gösterimi */}
            <div className="bg-white rounded-2xl shadow-lg w-full overflow-hidden border border-gray-100 mt-auto mb-auto">
               <div className="p-6 text-center">
                   
                   {/* NFC İkonu Animasyonu */}
                   <div className="flex justify-center mb-6">
                      <div className={`p-4 rounded-full ${isScanning ? 'bg-blue-100 animate-pulse' : 'bg-gray-100'}`}>
                        <svg className={`w-12 h-12 ${isScanning ? 'text-blue-600' : 'text-gray-400'}`} fill="none" stroke="currentColor" viewBox="0 0 24 24">
                          <path strokeLinecap="round" strokeLinejoin="round" strokeWidth={1.5} d="M8.111 16.404a5.5 5.5 0 017.778 0M12 20h.01m-7.08-7.071c3.904-3.905 10.236-3.905 14.14 0M1.394 9.393c5.857-5.857 15.355-5.857 21.213 0"></path>
                        </svg>
                      </div>
                   </div>

                   <div className="text-xs font-semibold text-gray-400 mb-2 uppercase tracking-widest">Kart ID (UID)</div>
                   <div className="bg-gray-50 border border-gray-200 rounded-xl py-4 px-2 mb-4">
                       <span className={`text-2xl font-mono font-bold tracking-widest ${cardUid ? 'text-gray-900' : 'text-gray-300'}`}>
                           {cardUid ? cardUid : 'Bekleniyor...'}
                       </span>
                   </div>

                   {messages.length > 0 && (
                       <div className="text-left mt-4 border-t border-gray-100 pt-4">
                           <div className="text-xs font-semibold text-gray-400 mb-2 uppercase text-center">Kart İçi Veri</div>
                           <div className="bg-blue-50 rounded-lg p-3 text-xs text-blue-900 border border-blue-100">
                               {messages.map((msg, index) => (
                                   <div key={index} className="mb-1 last:mb-0 break-words">{msg}</div>
                               ))}
                           </div>
                       </div>
                   )}
               </div>
            </div>

            {/* Tarama Butonu */}
            <div className="w-full mt-6">
              {isScanning ? (
                <button 
                  onClick={cancelNfcScan}
                  className="w-full bg-red-500 hover:bg-red-600 text-white font-bold py-4 rounded-xl shadow-lg transition duration-200"
                >
                  Taramayı İptal Et
                </button>
              ) : (
                <button 
                  onClick={startNfcScan}
                  disabled={hasNfcSupport === false}
                  className={`w-full font-bold py-4 rounded-xl shadow-lg transition duration-200 ${
                      hasNfcSupport === false 
                      ? 'bg-gray-300 text-gray-500 cursor-not-allowed' 
                      : 'bg-blue-600 hover:bg-blue-700 text-white'
                  }`}
                >
                  Kart Okutmaya Başla
                </button>
              )}
            </div>
          </div>
        )}

        {/* ================= SEÇENEK 2: GEÇMİŞ / KAYITLAR EKRANI ================= */}
        {activeTab === 'history' && (
          <div className="flex flex-col h-full fade-in">
            <div className="flex justify-between items-center mb-4">
               <div>
                 <h2 className="text-lg font-bold text-gray-800">Okuma Geçmişi</h2>
                 <p className="text-xs text-gray-500">Toplam Kayıt: {scanHistory.length}</p>
               </div>
               <button 
                  onClick={downloadHistory}
                  disabled={scanHistory.length === 0}
                  className={`text-xs font-bold py-2 px-4 rounded-lg shadow flex items-center transition duration-200 ${
                    scanHistory.length === 0 ? 'bg-gray-200 text-gray-400 cursor-not-allowed' : 'bg-green-600 hover:bg-green-700 text-white'
                  }`}
               >
                  <svg className="w-4 h-4 mr-1" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path strokeLinecap="round" strokeLinejoin="round" strokeWidth={2} d="M4 16v1a3 3 0 003 3h10a3 3 0 003-3v-1m-4-4l-4 4m0 0l-4-4m4 4V4"></path></svg>
                  CSV İndir
               </button>
            </div>

            {scanHistory.length === 0 ? (
              <div className="flex-grow flex flex-col items-center justify-center text-gray-400 mt-10">
                <svg className="w-16 h-16 mb-4 text-gray-200" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path strokeLinecap="round" strokeLinejoin="round" strokeWidth={1} d="M9 12h6m-6 4h6m2 5H7a2 2 0 01-2-2V5a2 2 0 012-2h5.586a1 1 0 01.707.293l5.414 5.414a1 1 0 01.293.707V19a2 2 0 01-2 2z"></path></svg>
                <p>Henüz kart okutulmadı.</p>
              </div>
            ) : (
              <div className="bg-white rounded-xl shadow-sm border border-gray-200 overflow-hidden">
                <ul className="divide-y divide-gray-100 max-h-[60vh] overflow-y-auto">
                  {scanHistory.map((record) => (
                    <li key={record.id} className="p-4 hover:bg-blue-50 transition duration-150">
                       <div className="flex justify-between items-start mb-1">
                          <span className="font-mono font-bold text-gray-800 text-sm">{record.uid}</span>
                          <span className="text-[10px] font-medium text-gray-500 bg-gray-100 px-2 py-1 rounded">{record.timestamp}</span>
                       </div>
                       <div className="text-xs text-gray-400 mt-1">{record.data}</div>
                    </li>
                  ))}
                </ul>
              </div>
            )}
          </div>
        )}
      </main>

      {/* Alt Menü (Bottom Navigation Bar) */}
      <nav className="fixed bottom-0 w-full max-w-md bg-white border-t border-gray-200 flex shadow-[0_-4px_15px_-5px_rgba(0,0,0,0.1)] z-20">
        <button 
          onClick={() => handleTabChange('scan')}
          className={`flex-1 flex flex-col items-center justify-center py-3 text-xs font-medium transition-colors ${
            activeTab === 'scan' ? 'text-blue-600' : 'text-gray-400 hover:text-gray-600'
          }`}
        >
          <svg className="w-6 h-6 mb-1" fill="none" stroke="currentColor" viewBox="0 0 24 24">
            <path strokeLinecap="round" strokeLinejoin="round" strokeWidth={activeTab==='scan'? 2:1.5} d="M12 4v1m6 11h2m-6 0h-2v4m0-11v3m0 0h.01M12 12h4.01M16 20h4M4 12h4m12 0h.01M5 8h2a1 1 0 001-1V5a1 1 0 00-1-1H5a1 1 0 00-1 1v2a1 1 0 001 1zm14 0h2a1 1 0 001-1V5a1 1 0 00-1-1h-2a1 1 0 00-1 1v2a1 1 0 001 1zM5 20h2a1 1 0 001-1v-2a1 1 0 00-1-1H5a1 1 0 00-1 1v2a1 1 0 001 1z"></path>
          </svg>
          Tarayıcı
        </button>
        
        <button 
          onClick={() => handleTabChange('history')}
          className={`flex-1 flex flex-col items-center justify-center py-3 text-xs font-medium transition-colors ${
            activeTab === 'history' ? 'text-blue-600' : 'text-gray-400 hover:text-gray-600'
          }`}
        >
          <div className="relative">
            <svg className="w-6 h-6 mb-1" fill="none" stroke="currentColor" viewBox="0 0 24 24">
              <path strokeLinecap="round" strokeLinejoin="round" strokeWidth={activeTab==='history'? 2:1.5} d="M9 5H7a2 2 0 00-2 2v12a2 2 0 002 2h10a2 2 0 002-2V7a2 2 0 00-2-2h-2M9 5a2 2 0 002 2h2a2 2 0 002-2M9 5a2 2 0 012-2h2a2 2 0 012 2m-3 7h3m-3 4h3m-6-4h.01M9 16h.01"></path>
            </svg>
            {/* Bildirim Baloncuğu */}
            {scanHistory.length > 0 && (
              <span className="absolute -top-1 -right-2 bg-red-500 text-white text-[10px] font-bold px-1.5 py-0.5 rounded-full">
                {scanHistory.length}
              </span>
            )}
          </div>
          Kayıtlar
        </button>
      </nav>

    </div>
  );
}
