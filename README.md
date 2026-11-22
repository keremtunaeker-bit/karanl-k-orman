<!DOCTYPE html>
<html lang="tr">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>KARANLIK ORMAN - 3D Zombi Hayatta Kalma FPS</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/three.js/r128/three.min.js"></script>
    <link href="https://fonts.googleapis.com/css2?family=Cinzel:wght@700;900&family=Inter:wght@400;600;700;900&display=swap" rel="stylesheet">
    <style>
        body {
            margin: 0;
            overflow: hidden;
            background-color: #05080a;
            font-family: 'Inter', sans-serif;
            user-select: none;
            -webkit-user-select: none;
        }
        .horror-title {
            font-family: 'Cinzel', serif;
            text-shadow: 0 0 25px rgba(220, 38, 38, 0.75), 0 4px 12px rgba(0, 0, 0, 0.95);
        }
        canvas {
            display: block;
            width: 100%;
            height: 100%;
        }
        /* Özel kaydırma çubuğu (Market & Ayarlar) */
        ::-webkit-scrollbar {
            width: 6px;
        }
        ::-webkit-scrollbar-track {
            background: rgba(15, 23, 42, 0.6);
            border-radius: 9999px;
        }
        ::-webkit-scrollbar-thumb {
            background: rgba(239, 68, 68, 0.6);
            border-radius: 9999px;
        }
        @keyframes pulseGlow {
            0%, 100% { box-shadow: 0 0 18px rgba(220, 38, 38, 0.45); }
            50% { box-shadow: 0 0 32px rgba(220, 38, 38, 0.85); }
        }
        .btn-horror {
            animation: pulseGlow 3s infinite;
        }
    </style>
</head>
<body class="text-white antialiased">

    <!-- 3D WebGL Sahne Konteyneri -->
    <div id="game-container" class="fixed inset-0 z-0"></div>

    <!-- Kırmızı Hasar Flaş Efekti -->
    <div id="damage-flash" class="fixed inset-0 z-10 pointer-events-none opacity-0 transition-opacity duration-150"
         style="background: radial-gradient(circle, transparent 45%, rgba(220, 38, 38, 0.72) 100%);"></div>

    <!-- AWP Keskin Nişancı Dürbün Arayüzü (Sağ Tık ADS) -->
    <div id="sniper-scope" class="hidden fixed inset-0 z-10 pointer-events-none flex items-center justify-center bg-black/45">
        <div class="relative w-[82vmin] h-[82vmin] rounded-full border-[6px] border-black shadow-[0_0_0_9999px_rgba(0,0,0,0.92)] flex items-center justify-center overflow-hidden">
            <div class="absolute inset-x-0 h-[1.5px] bg-red-500/80"></div>
            <div class="absolute inset-y-0 w-[1.5px] bg-red-500/80"></div>
            <div class="w-3 h-3 rounded-full bg-red-500 shadow-[0_0_10px_#ef4444]"></div>
            <div class="absolute bottom-8 text-[11px] font-mono tracking-widest text-red-400/90">AWP 8X OPTİK</div>
        </div>
    </div>

    <!-- Nişangah (Crosshair) ve Vuruş İmleci (Hitmarker) -->
    <div id="crosshair-container" class="hidden fixed inset-0 z-10 pointer-events-none flex items-center justify-center">
        <div id="crosshair" class="relative w-6 h-6 flex items-center justify-center transition-transform duration-75">
            <div class="w-1 h-1 rounded-full bg-emerald-400 shadow-[0_0_6px_#34d399]"></div>
            <div class="absolute top-0 w-[2px] h-1.5 bg-white/80 rounded-full"></div>
            <div class="absolute bottom-0 w-[2px] h-1.5 bg-white/80 rounded-full"></div>
            <div class="absolute left-0 h-[2px] w-1.5 bg-white/80 rounded-full"></div>
            <div class="absolute right-0 h-[2px] w-1.5 bg-white/80 rounded-full"></div>

            <!-- Hitmarker Çarpısı -->
            <div id="hitmarker" class="opacity-0 transition-opacity duration-75 absolute inset-0 flex items-center justify-center">
                <div class="w-5 h-0.5 bg-white rotate-45 absolute shadow-[0_0_6px_#fff]"></div>
                <div class="w-5 h-0.5 bg-white -rotate-45 absolute shadow-[0_0_6px_#fff]"></div>
            </div>
        </div>
    </div>

    <!-- Bildirim Mesajı (Toast) -->
    <div id="toast-msg" class="fixed top-24 left-1/2 -translate-x-1/2 z-30 pointer-events-none opacity-0 transition-all duration-300 bg-black/85 border border-emerald-500/50 text-emerald-300 px-5 py-2 rounded-full text-sm font-bold tracking-wide shadow-xl">
        Bildirim
    </div>

    <!-- Kombo Göstergesi -->
    <div id="combo-display" class="fixed top-28 right-6 z-20 pointer-events-none opacity-0 transition-all duration-200 transform scale-90 text-right">
        <div id="combo-count" class="text-3xl sm:text-4xl font-black italic text-amber-400 drop-shadow-[0_0_12px_rgba(245,158,11,0.9)]">KOMBO x2</div>
        <div class="text-[11px] font-bold tracking-widest uppercase text-amber-200">Seri Katliam!</div>
    </div>

    <!-- OYUN İÇİ HUD -->
    <div id="hud" class="hidden fixed inset-0 z-10 pointer-events-none flex flex-col justify-between p-4 sm:p-6">
        <!-- Üst Bilgi Alanı -->
        <div class="flex flex-wrap justify-between items-start gap-3">
            <!-- Sol Üst: Mini Radar + Can, Bakiye, Dalga ve Leş -->
            <div class="flex items-start gap-3">
                <!-- Taktiksel Mini Radar -->
                <div class="relative bg-black/80 backdrop-blur-md p-1.5 rounded-full border-2 border-emerald-500/40 shadow-[0_0_20px_rgba(16,185,129,0.25)] shrink-0">
                    <canvas id="minimap-canvas" class="w-24 h-24 sm:w-28 sm:h-28 rounded-full block"></canvas>
                    <div class="absolute bottom-1 left-1/2 -translate-x-1/2 text-[9px] font-bold tracking-widest text-emerald-400/90 bg-black/75 px-1.5 rounded">RADAR</div>
                </div>

                <!-- Can Barı & İstatistik Kutusu -->
                <div class="bg-black/75 backdrop-blur-md p-3.5 sm:p-4 rounded-2xl border border-white/10 min-w-[205px] sm:min-w-[240px] shadow-2xl">
                    <div class="flex justify-between items-center mb-1.5">
                        <span class="text-xs font-bold uppercase tracking-wider text-red-400 flex items-center gap-1.5">
                            <span class="w-2.5 h-2.5 rounded-full bg-red-500 animate-pulse"></span> CAN
                        </span>
                        <span id="hud-hp-text" class="text-sm font-extrabold text-white">20 / 20</span>
                    </div>
                    <div class="w-full h-3 bg-gray-800 rounded-full overflow-hidden border border-white/10">
                        <div id="hud-hp-bar" class="h-full bg-gradient-to-r from-red-600 to-rose-500 transition-all duration-200" style="width: 100%;"></div>
                    </div>

                    <div class="flex items-center justify-between mt-3 pt-2.5 border-t border-white/10 relative">
                        <div>
                            <div class="text-[11px] text-gray-400 uppercase font-semibold">Bakiye</div>
                            <div id="hud-money" class="text-lg font-black text-emerald-400">0 TL</div>
                            <div id="money-gain" class="absolute -bottom-5 left-0 text-xs font-black text-emerald-300 opacity-0 transition-all duration-300">+1 TL</div>
                        </div>
                        <div class="text-center">
                            <div class="text-[11px] text-gray-400 uppercase font-semibold">Dalga</div>
                            <div id="hud-wave" class="text-lg font-black text-amber-400">1</div>
                        </div>
                        <div class="text-right">
                            <div class="text-[11px] text-gray-400 uppercase font-semibold">Leş</div>
                            <div id="hud-kills" class="text-lg font-black text-white">0</div>
                        </div>
                    </div>
                </div>
            </div>

            <!-- Sağ Üst: Aktif Silah, Sonsuz Mermi, El Feneri ve El Bombası -->
            <div class="bg-black/75 backdrop-blur-md p-3.5 sm:p-4 rounded-2xl border border-white/10 text-right min-w-[195px] sm:min-w-[230px] shadow-2xl">
                <div class="text-[11px] text-gray-400 uppercase font-semibold">Aktif Silah</div>
                <div id="hud-weapon-name" class="text-sm sm:text-base font-extrabold text-white truncate">Taktik Tabanca</div>
                <div class="text-xs font-bold text-cyan-400 mt-0.5">∞ SONSUZ MERMİ</div>

                <div class="flex justify-end gap-2 mt-2.5 pt-2 border-t border-white/10 text-xs font-bold">
                    <span id="hud-flashlight" class="px-2 py-1 rounded-lg bg-amber-500/20 text-amber-300 border border-amber-500/30">[T] Fener: AÇIK</span>
                    <span id="hud-grenades" class="px-2 py-1 rounded-lg bg-orange-500/20 text-orange-300 border border-orange-500/30">[G] Bomba: 1</span>
                </div>
            </div>
        </div>

        <!-- Alt Kısa Tuş Rehberi -->
        <div class="flex justify-center">
            <div class="bg-black/65 backdrop-blur-md px-4 py-2 rounded-full border border-white/10 text-[11px] sm:text-xs text-gray-300 flex flex-wrap justify-center gap-x-4 gap-y-1">
                <span><strong class="text-white">WASD</strong> Hareket</span>
                <span><strong class="text-white">Shift</strong> Koş</span>
                <span><strong class="text-white">Space</strong> Zıpla</span>
                <span><strong class="text-emerald-400">Sol Tık</strong> Ateş</span>
                <span><strong class="text-cyan-400">Sağ Tık</strong> Nişan/Dürbün</span>
                <span><strong class="text-amber-400">[F]</strong> Market</span>
                <span><strong class="text-orange-400">[G]</strong> Bomba</span>
                <span><strong class="text-gray-200">[ESC]</strong> Duraklat</span>
            </div>
        </div>
    </div>

    <!-- İMLEÇ KİLİDİ UYARISI (Oyun esnasında fare kilidi çözülürse) -->
    <div id="lock-overlay" class="hidden fixed inset-0 z-20 bg-black/65 backdrop-blur-sm flex items-center justify-center cursor-pointer">
        <div class="text-center p-6 bg-gray-900/90 border border-red-500/40 rounded-2xl max-w-sm mx-4 shadow-2xl">
            <div class="text-xl font-black text-white mb-2">OYUNA DEVAM ET</div>
            <p class="text-sm text-gray-300 mb-4">Nişan almak ve ateş etmek için ekrana tıklayın.</p>
            <div class="inline-block px-5 py-2.5 bg-red-600 hover:bg-red-500 rounded-xl font-bold text-sm transition">TIKLA VE DEVAM ET</div>
        </div>
    </div>

    <!-- 1. ANA LOBİ MENÜSÜ ("KARANLIK ORMAN") -->
    <div id="main-menu" class="fixed inset-0 z-30 flex items-center justify-center bg-gradient-to-b from-black/65 via-black/45 to-black/85 p-4">
        <div class="max-w-lg w-full bg-gray-950/80 backdrop-blur-xl border border-red-900/50 rounded-3xl p-7 sm:p-9 shadow-[0_0_60px_rgba(0,0,0,0.95)] text-center">
            <div class="inline-block px-3.5 py-1 rounded-full bg-red-950/80 border border-red-600/40 text-red-400 text-xs font-bold tracking-widest uppercase mb-3">
                3D Hayatta Kalma FPS
            </div>
            <h1 class="horror-title text-4xl sm:text-5xl font-black tracking-wider text-red-500 mb-2">
                KARANLIK ORMAN
            </h1>
            <p class="text-sm text-gray-400 mb-6">
                Sislerin arasından gelen zombi sürülerine karşı hayatta kal, her leşten <span class="text-emerald-400 font-bold">1 TL</span> kazan ve cephaneliğini güçlendir.
            </p>

            <!-- Kalıcı Rekor ve Bakiye Kartı -->
            <div class="grid grid-cols-3 gap-2.5 bg-black/60 border border-white/10 rounded-2xl p-3.5 mb-6">
                <div>
                    <div class="text-[11px] text-gray-400 uppercase font-semibold">Bakiye</div>
                    <div id="lobby-money" class="text-lg font-black text-emerald-400">0 TL</div>
                </div>
                <div class="border-x border-white/10">
                    <div class="text-[11px] text-gray-400 uppercase font-semibold">En Yüksek Dalga</div>
                    <div id="lobby-best-wave" class="text-lg font-black text-amber-400">1</div>
                </div>
                <div>
                    <div class="text-[11px] text-gray-400 uppercase font-semibold">En Çok Leş</div>
                    <div id="lobby-best-kills" class="text-lg font-black text-red-400">0</div>
                </div>
            </div>

            <!-- Ana Lobi Butonları -->
            <div class="space-y-3">
                <button id="btn-start-game" class="btn-horror w-full py-4 rounded-2xl bg-gradient-to-r from-red-700 to-red-600 hover:from-red-600 hover:to-red-500 text-white font-black text-lg tracking-wider transition transform active:scale-98 shadow-lg cursor-pointer">
                    OYUNA BAŞLA
                </button>
                <button id="btn-open-market" class="w-full py-3.5 rounded-2xl bg-emerald-950/80 hover:bg-emerald-900/90 border border-emerald-500/40 text-emerald-300 font-extrabold text-base tracking-wide transition transform active:scale-98 cursor-pointer">
                    SİLAH MARKETİ
                </button>
                <button id="btn-open-settings" class="w-full py-3.5 rounded-2xl bg-gray-900/90 hover:bg-gray-800 border border-white/15 text-gray-200 font-bold text-base tracking-wide transition transform active:scale-98 cursor-pointer">
                    AYARLAR
                </button>
            </div>
        </div>
    </div>

    <!-- 2. DURAKLATMA MENÜSÜ (ESC) -->
    <div id="pause-menu" class="hidden fixed inset-0 z-30 flex items-center justify-center bg-black/75 backdrop-blur-md p-4">
        <div class="max-w-md w-full bg-gray-950/95 border border-white/15 rounded-3xl p-7 text-center shadow-2xl">
            <h2 class="text-3xl font-black text-white tracking-wide mb-2">OYUN DURAKLATILDI</h2>
            <p class="text-sm text-gray-400 mb-6">Karanlık ormanda tehlike her an devam ediyor.</p>
            <div class="space-y-3">
                <button id="btn-resume-game" class="w-full py-3.5 rounded-2xl bg-red-600 hover:bg-red-500 text-white font-black text-base transition cursor-pointer">
                    DEVAM ET
                </button>
                <button id="btn-pause-market" class="w-full py-3.5 rounded-2xl bg-emerald-900/80 hover:bg-emerald-800 border border-emerald-500/40 text-emerald-300 font-extrabold text-base transition cursor-pointer">
                    SİLAH MARKETİ (F)
                </button>
                <button id="btn-pause-settings" class="w-full py-3.5 rounded-2xl bg-gray-800 hover:bg-gray-700 text-gray-200 font-bold text-base transition cursor-pointer">
                    AYARLAR
                </button>
                <button id="btn-pause-lobby" class="w-full py-3.5 rounded-2xl bg-gray-900 hover:bg-gray-800 border border-white/10 text-gray-400 font-bold text-base transition cursor-pointer">
                    LOBİYE DÖN
                </button>
            </div>
        </div>
    </div>

    <!-- 3. SİLAH MARKETİ MENÜSÜ (Lobi ve Oyun İçi 'F' Tuşu) -->
    <div id="market-menu" class="hidden fixed inset-0 z-40 flex items-center justify-center bg-black/80 backdrop-blur-md p-4">
        <div class="max-w-4xl w-full bg-gray-950/95 border border-emerald-500/30 rounded-3xl p-5 sm:p-7 shadow-2xl flex flex-col max-h-[90vh]">
            <div class="flex flex-wrap justify-between items-center gap-3 pb-4 border-b border-white/10">
                <div>
                    <h2 class="text-2xl sm:text-3xl font-black text-emerald-400 tracking-wide">SİLAH MARKETİ</h2>
                    <p class="text-xs sm:text-sm text-gray-400">Her zombi 1 TL kazandırır. Tüm silahlar sonsuz mermilidir.</p>
                </div>
                <div class="flex items-center gap-3">
                    <div class="bg-emerald-950/90 border border-emerald-500/40 px-4 py-2 rounded-2xl">
                        <span class="text-xs text-emerald-300 block">Mevcut Bakiye</span>
                        <span id="market-money" class="text-xl font-black text-emerald-400">0 TL</span>
                    </div>
                    <button id="btn-close-market" class="px-4 py-3 rounded-2xl bg-gray-800 hover:bg-gray-700 text-white font-bold text-sm transition cursor-pointer">
                        KAPAT ✕
                    </button>
                </div>
            </div>

            <!-- Silah Listesi Grid -->
            <div id="weapon-list" class="grid grid-cols-1 md:grid-cols-2 gap-3.5 overflow-y-auto py-4 pr-1">
                <!-- JavaScript ile dinamik doldurulacak -->
            </div>
        </div>
    </div>

    <!-- 4. AYARLAR MENÜSÜ -->
    <div id="settings-menu" class="hidden fixed inset-0 z-40 flex items-center justify-center bg-black/80 backdrop-blur-md p-4">
        <div class="max-w-md w-full bg-gray-950/95 border border-white/15 rounded-3xl p-6 sm:p-8 shadow-2xl">
            <h2 class="text-2xl sm:text-3xl font-black text-white mb-5">AYARLAR</h2>

            <div class="space-y-5">
                <!-- Fare Hassasiyeti -->
                <div>
                    <div class="flex justify-between text-sm font-bold mb-1.5">
                        <span class="text-gray-300">Fare Hassasiyeti</span>
                        <span id="sens-val" class="text-red-400">1.0x</span>
                    </div>
                    <input id="input-sens" type="range" min="0.3" max="2.5" step="0.1" value="1.0"
                           class="w-full accent-red-500 bg-gray-800 h-2 rounded-lg cursor-pointer">
                </div>

                <!-- Ses Seviyesi -->
                <div>
                    <div class="flex justify-between text-sm font-bold mb-1.5">
                        <span class="text-gray-300">Ses Efektleri Seviyesi</span>
                        <span id="vol-val" class="text-emerald-400">70%</span>
                    </div>
                    <input id="input-vol" type="range" min="0" max="100" step="5" value="70"
                           class="w-full accent-emerald-500 bg-gray-800 h-2 rounded-lg cursor-pointer">
                </div>

                <!-- Sis / Grafik Kalitesi -->
                <div>
                    <div class="text-sm font-bold text-gray-300 mb-2">Sis & Atmosfer Kalitesi</div>
                    <div class="grid grid-cols-2 gap-2">
                        <button id="btn-gfx-high" class="py-2.5 rounded-xl bg-red-600 text-white font-bold text-xs border border-red-400 cursor-pointer">
                            YÜKSEK (Gölgeler + Ateşböceği)
                        </button>
                        <button id="btn-gfx-low" class="py-2.5 rounded-xl bg-gray-800 text-gray-300 font-bold text-xs border border-white/10 cursor-pointer">
                            PERFORMANS (Hızlı FPS)
                        </button>
                    </div>
                </div>

                <!-- Düz Fare Kontrolü Bilgilendirmesi -->
                <div class="p-3.5 rounded-2xl bg-emerald-950/40 border border-emerald-500/30 text-xs text-emerald-200 leading-relaxed">
                    <strong>✓ Standart Düz Fare Kontrolü Aktif:</strong> Fareyi yukarı ittiğinizde yukarı, aşağı çektiğinizde aşağı bakarsınız (Ters eksen kapalıdır).
                </div>
            </div>

            <button id="btn-close-settings" class="mt-6 w-full py-3.5 rounded-2xl bg-white text-gray-950 hover:bg-gray-200 font-black text-base transition cursor-pointer">
                KAYDET VE DÖN
            </button>
        </div>
    </div>

    <!-- 5. ÖLÜM EKRANI ("ÖLDÜNÜZ") -->
    <div id="death-menu" class="hidden fixed inset-0 z-40 flex items-center justify-center bg-red-950/85 backdrop-blur-md p-4">
        <div class="max-w-md w-full bg-gray-950/95 border-2 border-red-600/60 rounded-3xl p-7 sm:p-8 text-center shadow-[0_0_60px_rgba(220,38,38,0.65)]">
            <h2 class="horror-title text-5xl font-black text-red-500 tracking-wider mb-2">ÖLDÜNÜZ</h2>
            <p class="text-sm text-gray-400 mb-6">Karanlık orman bir kurban daha aldı...</p>

            <!-- Koşu İstatistikleri -->
            <div class="grid grid-cols-3 gap-2 bg-black/70 border border-white/10 rounded-2xl p-4 mb-6">
                <div>
                    <div class="text-[11px] text-gray-400 uppercase font-semibold">Ulaşılan Dalga</div>
                    <div id="death-wave" class="text-xl font-black text-amber-400">1</div>
                </div>
                <div class="border-x border-white/10">
                    <div class="text-[11px] text-gray-400 uppercase font-semibold">Öldürülen Zombi</div>
                    <div id="death-kills" class="text-xl font-black text-white">0</div>
                </div>
                <div>
                    <div class="text-[11px] text-gray-400 uppercase font-semibold">Toplam Bakiye</div>
                    <div id="death-money" class="text-xl font-black text-emerald-400">0 TL</div>
                </div>
            </div>

            <!-- Ölüm Ekranı Butonları -->
            <div class="space-y-3">
                <button id="btn-restart-game" class="w-full py-4 rounded-2xl bg-gradient-to-r from-red-700 to-red-600 hover:from-red-600 hover:to-red-500 text-white font-black text-base tracking-wider transition cursor-pointer">
                    YENİDEN BAŞLA
                </button>
                <button id="btn-death-lobby" class="w-full py-3.5 rounded-2xl bg-emerald-950/90 hover:bg-emerald-900 border border-emerald-500/40 text-emerald-300 font-extrabold text-base tracking-wide transition cursor-pointer">
                    LOBİYE DÖN
                </button>
                <button id="btn-death-settings" class="w-full py-3.5 rounded-2xl bg-gray-900 hover:bg-gray-800 border border-white/15 text-gray-200 font-bold text-base tracking-wide transition cursor-pointer">
                    AYARLAR
                </button>
            </div>
        </div>
    </div>

    <script>
        /**
         * PROSEDÜREL SES MOTORU (WEB AUDIO API)
         * Harici ses dosyası olmadan gerçekçi silah, zombi, patlama ve orman ambiyans sesleri üretir.
         */
        class SoundEngine {
            constructor() {
                this.ctx = null;
                this.volume = 0.7;
            }

            init() {
                if (!this.ctx) {
                    const AudioCtx = window.AudioContext || window.webkitAudioContext;
                    this.ctx = new AudioCtx();
                }
                if (this.ctx.state === 'suspended') {
                    this.ctx.resume();
                }
            }

            setVolume(val) {
                this.volume = Math.max(0, Math.min(1, val));
            }

            playShoot(weaponId) {
                if (!this.ctx || this.volume <= 0) return;
                const now = this.ctx.currentTime;

                const osc = this.ctx.createOscillator();
                const gain = this.ctx.createGain();

                if (weaponId === 'plasma') {
                    osc.type = 'sawtooth';
                    osc.frequency.setValueAtTime(920, now);
                    osc.frequency.exponentialRampToValueAtTime(140, now + 0.16);
                    gain.gain.setValueAtTime(this.volume * 0.35, now);
                    gain.gain.exponentialRampToValueAtTime(0.001, now + 0.17);
                } else if (weaponId === 'sniper') {
                    osc.type = 'triangle';
                    osc.frequency.setValueAtTime(240, now);
                    osc.frequency.exponentialRampToValueAtTime(35, now + 0.42);
                    gain.gain.setValueAtTime(this.volume * 0.7, now);
                    gain.gain.exponentialRampToValueAtTime(0.001, now + 0.44);
                } else if (weaponId === 'shotgun') {
                    osc.type = 'square';
                    osc.frequency.setValueAtTime(180, now);
                    osc.frequency.exponentialRampToValueAtTime(40, now + 0.28);
                    gain.gain.setValueAtTime(this.volume * 0.6, now);
                    gain.gain.exponentialRampToValueAtTime(0.001, now + 0.3);
                } else if (weaponId === 'minigun') {
                    osc.type = 'sawtooth';
                    osc.frequency.setValueAtTime(210, now);
                    osc.frequency.exponentialRampToValueAtTime(55, now + 0.09);
                    gain.gain.setValueAtTime(this.volume * 0.35, now);
                    gain.gain.exponentialRampToValueAtTime(0.001, now + 0.1);
                } else {
                    osc.type = 'sawtooth';
                    osc.frequency.setValueAtTime(310, now);
                    osc.frequency.exponentialRampToValueAtTime(55, now + 0.14);
                    gain.gain.setValueAtTime(this.volume * 0.45, now);
                    gain.gain.exponentialRampToValueAtTime(0.001, now + 0.15);
                }

                osc.connect(gain);
                gain.connect(this.ctx.destination);
                osc.start(now);
                osc.stop(now + 0.45);

                // Barut patlama gürültüsü (White Noise Burst)
                const bufferSize = this.ctx.sampleRate * 0.12;
                const buffer = this.ctx.createBuffer(1, bufferSize, this.ctx.sampleRate);
                const data = buffer.getChannelData(0);
                for (let i = 0; i < bufferSize; i++) {
                    data[i] = (Math.random() * 2 - 1) * Math.exp(-i / (bufferSize * 0.35));
                }
                const noise = this.ctx.createBufferSource();
                noise.buffer = buffer;
                const nGain = this.ctx.createGain();
                nGain.gain.setValueAtTime(this.volume * 0.4, now);
                nGain.gain.exponentialRampToValueAtTime(0.001, now + 0.12);
                noise.connect(nGain);
                nGain.connect(this.ctx.destination);
                noise.start(now);
            }

            playHeadshot() {
                if (!this.ctx || this.volume <= 0) return;
                const now = this.ctx.currentTime;
                const osc = this.ctx.createOscillator();
                const gain = this.ctx.createGain();
                osc.type = 'sine';
                osc.frequency.setValueAtTime(1250, now);
                osc.frequency.exponentialRampToValueAtTime(1850, now + 0.09);
                gain.gain.setValueAtTime(this.volume * 0.45, now);
                gain.gain.exponentialRampToValueAtTime(0.001, now + 0.11);
                osc.connect(gain);
                gain.connect(this.ctx.destination);
                osc.start(now);
                osc.stop(now + 0.12);
            }

            playZombieDie() {
                if (!this.ctx || this.volume <= 0) return;
                const now = this.ctx.currentTime;
                const osc = this.ctx.createOscillator();
                const gain = this.ctx.createGain();
                osc.type = 'sawtooth';
                osc.frequency.setValueAtTime(135, now);
                osc.frequency.exponentialRampToValueAtTime(42, now + 0.35);
                gain.gain.setValueAtTime(this.volume * 0.35, now);
                gain.gain.exponentialRampToValueAtTime(0.001, now + 0.36);
                osc.connect(gain);
                gain.connect(this.ctx.destination);
                osc.start(now);
                osc.stop(now + 0.37);
            }

            playPlayerHurt() {
                if (!this.ctx || this.volume <= 0) return;
                const now = this.ctx.currentTime;
                const osc = this.ctx.createOscillator();
                const gain = this.ctx.createGain();
                osc.type = 'square';
                osc.frequency.setValueAtTime(110, now);
                osc.frequency.linearRampToValueAtTime(60, now + 0.2);
                gain.gain.setValueAtTime(this.volume * 0.5, now);
                gain.gain.exponentialRampToValueAtTime(0.001, now + 0.22);
                osc.connect(gain);
                gain.connect(this.ctx.destination);
                osc.start(now);
                osc.stop(now + 0.23);
            }

            playExplosion() {
                if (!this.ctx || this.volume <= 0) return;
                const now = this.ctx.currentTime;
                const bufferSize = this.ctx.sampleRate * 0.65;
                const buffer = this.ctx.createBuffer(1, bufferSize, this.ctx.sampleRate);
                const data = buffer.getChannelData(0);
                for (let i = 0; i < bufferSize; i++) {
                    data[i] = (Math.random() * 2 - 1) * Math.exp(-i / (bufferSize * 0.28));
                }
                const noise = this.ctx.createBufferSource();
                noise.buffer = buffer;

                const filter = this.ctx.createBiquadFilter();
                filter.type = 'lowpass';
                filter.frequency.setValueAtTime(420, now);
                filter.frequency.exponentialRampToValueAtTime(65, now + 0.6);

                const gain = this.ctx.createGain();
                gain.gain.setValueAtTime(this.volume * 0.9, now);
                gain.gain.exponentialRampToValueAtTime(0.001, now + 0.65);

                noise.connect(filter);
                filter.connect(gain);
                gain.connect(this.ctx.destination);
                noise.start(now);
            }

            playPickup() {
                if (!this.ctx || this.volume <= 0) return;
                const now = this.ctx.currentTime;
                const osc = this.ctx.createOscillator();
                const gain = this.ctx.createGain();
                osc.type = 'sine';
                osc.frequency.setValueAtTime(520, now);
                osc.frequency.setValueAtTime(780, now + 0.08);
                gain.gain.setValueAtTime(this.volume * 0.35, now);
                gain.gain.exponentialRampToValueAtTime(0.001, now + 0.2);
                osc.connect(gain);
                gain.connect(this.ctx.destination);
                osc.start(now);
                osc.stop(now + 0.21);
            }

            playClick() {
                if (!this.ctx || this.volume <= 0) return;
                const now = this.ctx.currentTime;
                const osc = this.ctx.createOscillator();
                const gain = this.ctx.createGain();
                osc.type = 'sine';
                osc.frequency.setValueAtTime(620, now);
                gain.gain.setValueAtTime(this.volume * 0.2, now);
                gain.gain.exponentialRampToValueAtTime(0.001, now + 0.05);
                osc.connect(gain);
                gain.connect(this.ctx.destination);
                osc.start(now);
                osc.stop(now + 0.06);
            }

            playAmbienceHoot() {
                if (!this.ctx || this.volume <= 0) return;
                const now = this.ctx.currentTime;
                const osc = this.ctx.createOscillator();
                const gain = this.ctx.createGain();
                osc.type = 'sine';
                osc.frequency.setValueAtTime(380, now);
                osc.frequency.exponentialRampToValueAtTime(300, now + 0.35);
                osc.frequency.setValueAtTime(335, now + 0.48);
                osc.frequency.exponentialRampToValueAtTime(255, now + 1.0);
                gain.gain.setValueAtTime(0.001, now);
                gain.gain.linearRampToValueAtTime(this.volume * 0.12, now + 0.12);
                gain.gain.exponentialRampToValueAtTime(0.001, now + 1.05);
                osc.connect(gain);
                gain.connect(this.ctx.destination);
                osc.start(now);
                osc.stop(now + 1.1);
            }
        }

        const sfx = new SoundEngine();

        /**
         * SİLAH KATALOĞU (7 DETAYLI 3D SİLAH)
         */
        const WEAPONS = [
            {
                id: 'pistol',
                name: 'Taktik Tabanca',
                price: 0,
                damage: 1, // Normal zombiyi 3 vücut veya 2 kafa vuruşunda öldürür
                fireRate: 260,
                spread: 0.012,
                pellets: 1,
                recoil: 0.06,
                color: 0x3b82f6,
                desc: 'Başlangıç silahı. Susturuculu ve kırmızı noktalı. Zombileri 3 vuruşta indirir.'
            },
            {
                id: 'shotgun',
                name: 'M870 Pompalı Tüfek',
                price: 10,
                damage: 1,
                fireRate: 780,
                spread: 0.075,
                pellets: 5,
                recoil: 0.16,
                color: 0xf97316,
                desc: 'Tek atışta 5 ağır saçma fırlatır. Yakın mesafede zombileri paramparça eder.'
            },
            {
                id: 'smg',
                name: 'MP5 Hafif Makineli',
                price: 20,
                damage: 1,
                fireRate: 115,
                spread: 0.025,
                pellets: 1,
                recoil: 0.045,
                color: 0x10b981,
                desc: 'Holografik nişangahlı seri taramalı tüfek. Zombi sürülerini hızla eritir.'
            },
            {
                id: 'ak47',
                name: 'AK-47 Taarruz Tüfeği',
                price: 35,
                damage: 1.5, // 2 vuruşta öldürür
                fireRate: 145,
                spread: 0.019,
                pellets: 1,
                recoil: 0.075,
                color: 0xef4444,
                desc: 'Yüksek kalibre mermileriyle zombileri 2 vuruşta indirir. Efsanevi ateş gücü.'
            },
            {
                id: 'sniper',
                name: 'AWP Keskin Nişancı',
                price: 55,
                damage: 4, // Tek vuruşta öldürür
                fireRate: 920,
                spread: 0.001,
                pellets: 1,
                recoil: 0.22,
                color: 0xa855f7,
                desc: 'Sağ tık ile 8x optik dürbün açar. Zırh delici mermisi zombileri tek atışta yok eder.'
            },
            {
                id: 'plasma',
                name: 'X-99 Plazma Tüfeği',
                price: 80,
                damage: 3,
                fireRate: 100,
                spread: 0.014,
                pellets: 1,
                recoil: 0.05,
                color: 0x06b6d4,
                desc: 'Parlayan plazma bobinleriyle yüksek enerjili ışınlar ateşler.'
            },
            {
                id: 'minigun',
                name: 'M134 Minigun',
                price: 120,
                damage: 1.5,
                fireRate: 65,
                spread: 0.032,
                pellets: 1,
                recoil: 0.04,
                color: 0xfacc15,
                desc: '6 döner namlulu ağır makineli! Dakikada binlerce mermiyle ormanı dümdüz eder.'
            }
        ];

        /**
         * OYUN DURUMU VE KALICI REKORLAR
         */
        const state = {
            isPlaying: false,
            isPaused: false,
            isDead: false,
            isMarketOpen: false,
            previousMenu: 'main',
            hp: 20,
            maxHp: 20,
            money: parseInt(localStorage.getItem('karanlik_orman_money') || '0', 10),
            kills: 0,
            wave: 1,
            grenades: 1,
            combo: 0,
            comboTimer: 0,
            bestWave: parseInt(localStorage.getItem('karanlik_orman_best_wave') || '1', 10),
            bestKills: parseInt(localStorage.getItem('karanlik_orman_best_kills') || '0', 10),
            ownedWeapons: JSON.parse(localStorage.getItem('karanlik_orman_weapons') || '["pistol"]'),
            currentWeaponId: localStorage.getItem('karanlik_orman_equipped') || 'pistol',
            sensitivity: 1.0,
            highGraphics: true,
            lastShotTime: 0,
            isMouseDown: false,
            isAiming: false,
            flashlightOn: true
        };

        function saveProgress() {
            localStorage.setItem('karanlik_orman_money', state.money.toString());
            localStorage.setItem('karanlik_orman_best_wave', state.bestWave.toString());
            localStorage.setItem('karanlik_orman_best_kills', state.bestKills.toString());
            localStorage.setItem('karanlik_orman_weapons', JSON.stringify(state.ownedWeapons));
            localStorage.setItem('karanlik_orman_equipped', state.currentWeaponId);
        }

        // Three.js Bileşenleri
        let scene, camera, renderer;
        let gunGroup, muzzleFlash, muzzleLight, flashlight, fireflies, starsMesh, minigunBarrels;
        let gunRecoil = 0;
        let walkBobTime = 0;

        // Oyuncu Fizik & Düz Fare Bakış Kontrolü (Ters DEĞİL)
        const player = {
            height: 1.75,
            velocity: new THREE.Vector3(),
            onGround: true,
            walkSpeed: 7.4,
            sprintSpeed: 11.4,
            jumpForce: 8.6,
            yaw: 0,
            pitch: 0
        };

        const keys = { w: false, a: false, s: false, d: false, space: false, shift: false };
        const zombies = [];
        const dyingZombies = [];
        const particles = [];
        const tracers = [];
        const pickups = [];
        const activeGrenades = [];
        const barrels = [];
        const fogLayers = [];
        const obstacles = []; // Ağaç gövdeleri ve taş duvar çarpışma daireleri

        let lastTime = performance.now();
        let spawnTimer = 0;
        let ambienceTimer = 9;
        let lobbyCamAngle = 0;

        /**
         * PROSEDÜREL CANVAS DOKULARI (Harici resim URL'si yok - sıfır hata garantisi)
         */
        function createGroundTexture() {
            const canvas = document.createElement('canvas');
            canvas.width = 512;
            canvas.height = 512;
            const ctx = canvas.getContext('2d');

            // Koyu toz-toprak tabanı
            ctx.fillStyle = '#1b1713';
            ctx.fillRect(0, 0, 512, 512);

            // Toprak, çakıl ve toz taneleri
            for (let i = 0; i < 6500; i++) {
                const x = Math.random() * 512;
                const y = Math.random() * 512;
                const r = Math.random() * 2.5 + 0.5;
                const shade = Math.floor(20 + Math.random() * 28);
                ctx.fillStyle = `rgb(${shade + 8}, ${shade + 3}, ${shade - 2})`;
                ctx.fillRect(x, y, r, r);
            }

            // Kurumuş yaprak ve ot lekeleri
            for (let i = 0; i < 180; i++) {
                ctx.fillStyle = Math.random() > 0.5 ? 'rgba(34, 46, 26, 0.35)' : 'rgba(55, 42, 28, 0.35)';
                ctx.beginPath();
                ctx.ellipse(Math.random() * 512, Math.random() * 512, 5, 2, Math.random() * Math.PI, 0, Math.PI * 2);
                ctx.fill();
            }

            // Yerde kan sıçramaları
            for (let i = 0; i < 24; i++) {
                const bx = Math.random() * 512;
                const by = Math.random() * 512;
                ctx.fillStyle = 'rgba(110, 10, 10, 0.55)';
                ctx.beginPath();
                ctx.arc(bx, by, 4 + Math.random() * 12, 0, Math.PI * 2);
                ctx.fill();
                for (let d = 0; d < 5; d++) {
                    ctx.beginPath();
                    ctx.arc(bx + (Math.random() - 0.5) * 28, by + (Math.random() - 0.5) * 28, 1.5 + Math.random() * 3, 0, Math.PI * 2);
                    ctx.fill();
                }
            }

            const tex = new THREE.CanvasTexture(canvas);
            tex.wrapS = THREE.RepeatWrapping;
            tex.wrapT = THREE.RepeatWrapping;
            tex.repeat.set(28, 28);
            return tex;
        }

        function createBarkTexture() {
            const canvas = document.createElement('canvas');
            canvas.width = 256;
            canvas.height = 256;
            const ctx = canvas.getContext('2d');
            ctx.fillStyle = '#241b15';
            ctx.fillRect(0, 0, 256, 256);

            for (let i = 0; i < 140; i++) {
                ctx.fillStyle = Math.random() > 0.5 ? '#17110d' : '#33261d';
                ctx.fillRect(Math.random() * 256, 0, 2 + Math.random() * 4, 256);
            }
            const tex = new THREE.CanvasTexture(canvas);
            tex.wrapS = THREE.RepeatWrapping;
            tex.wrapT = THREE.RepeatWrapping;
            return tex;
        }

        function createStoneTexture() {
            const canvas = document.createElement('canvas');
            canvas.width = 256;
            canvas.height = 256;
            const ctx = canvas.getContext('2d');
            ctx.fillStyle = '#33383d';
            ctx.fillRect(0, 0, 256, 256);

            for (let i = 0; i < 900; i++) {
                const v = 35 + Math.floor(Math.random() * 45);
                ctx.fillStyle = `rgb(${v}, ${v + 2}, ${v + 5})`;
                ctx.fillRect(Math.random() * 256, Math.random() * 256, 6, 6);
            }
            ctx.strokeStyle = '#1e2226';
            ctx.lineWidth = 3;
            for (let y = 0; y < 256; y += 64) {
                ctx.beginPath();
                ctx.moveTo(0, y);
                ctx.lineTo(256, y);
                ctx.stroke();
            }
            const tex = new THREE.CanvasTexture(canvas);
            tex.wrapS = THREE.RepeatWrapping;
            tex.wrapT = THREE.RepeatWrapping;
            return tex;
        }

        /**
         * 3D DÜNYA KURULUMU (KARANLIK ORMAN)
         */
        function init3D() {
            const container = document.getElementById('game-container');
            scene = new THREE.Scene();
            scene.background = new THREE.Color(0x070b10);
            scene.fog = new THREE.FogExp2(0x0b1219, 0.036);

            camera = new THREE.PerspectiveCamera(72, window.innerWidth / window.innerHeight, 0.1, 180);
            camera.rotation.order = 'YXZ'; // Standart FPS Euler sırası (Ters fare kontrolü yok)
            camera.position.set(0, player.height, 0);
            scene.add(camera);

            renderer = new THREE.WebGLRenderer({ antialias: true, powerPreference: 'high-performance' });
            renderer.setSize(window.innerWidth, window.innerHeight);
            renderer.setPixelRatio(Math.min(window.devicePixelRatio, 2));
            renderer.shadowMap.enabled = true;
            renderer.shadowMap.type = THREE.PCFSoftShadowMap;
            container.appendChild(renderer.domElement);

            // Ay Işığı ve Ortam Aydınlatması
            const hemiLight = new THREE.HemisphereLight(0x3b4d63, 0x1a1510, 0.72);
            scene.add(hemiLight);

            const moonDirLight = new THREE.DirectionalLight(0x8ab4f8, 0.85);
            moonDirLight.position.set(40, 65, -50);
            moonDirLight.castShadow = true;
            moonDirLight.shadow.mapSize.width = 1024;
            moonDirLight.shadow.mapSize.height = 1024;
            moonDirLight.shadow.camera.near = 5;
            moonDirLight.shadow.camera.far = 150;
            const d = 48;
            moonDirLight.shadow.camera.left = -d;
            moonDirLight.shadow.camera.right = d;
            moonDirLight.shadow.camera.top = d;
            moonDirLight.shadow.camera.bottom = -d;
            scene.add(moonDirLight);

            // Taktiksel El Feneri (Kameraya bağlı SpotLight)
            flashlight = new THREE.SpotLight(0xfff4d6, 2.8, 38, Math.PI / 5.5, 0.45, 1.2);
            flashlight.position.set(0.18, -0.1, -0.2);
            flashlight.target.position.set(0, -0.2, -12);
            camera.add(flashlight);
            camera.add(flashlight.target);

            // Tozlu ve Kanlı Zemin
            const groundGeo = new THREE.PlaneGeometry(160, 160);
            const groundMat = new THREE.MeshStandardMaterial({
                map: createGroundTexture(),
                roughness: 0.92,
                metalness: 0.05
            });
            const ground = new THREE.Mesh(groundGeo, groundMat);
            ground.rotation.x = -Math.PI / 2;
            ground.receiveShadow = true;
            scene.add(ground);

            // Dolunay ve Yıldızlı Gece Gökyüzü
            const moonMesh = new THREE.Mesh(
                new THREE.SphereGeometry(5.5, 24, 24),
                new THREE.MeshBasicMaterial({ color: 0xe2e8f0, fog: false })
            );
            moonMesh.position.set(55, 68, -75);
            scene.add(moonMesh);

            createStarfield();
            createDriftingFogLayers();
            createFireflies();
            createBloodPuddles();
            createRuinedForest();
            createRuinedWalls();
            createExplosiveBarrels();

            // Eldivenli 3D Silah Modelini Yükle
            buildWeaponModel(state.currentWeaponId);

            window.addEventListener('resize', onWindowResize);
        }

        function createStarfield() {
            const count = 420;
            const positions = new Float32Array(count * 3);
            for (let i = 0; i < count; i++) {
                const theta = Math.random() * Math.PI * 2;
                const phi = Math.random() * (Math.PI * 0.44);
                const r = 135;
                positions[i * 3] = r * Math.sin(phi) * Math.cos(theta);
                positions[i * 3 + 1] = 25 + r * Math.cos(phi);
                positions[i * 3 + 2] = r * Math.sin(phi) * Math.sin(theta);
            }
            const geo = new THREE.BufferGeometry();
            geo.setAttribute('position', new THREE.BufferAttribute(positions, 3));
            starsMesh = new THREE.Points(geo, new THREE.PointsMaterial({
                color: 0xf8fafc,
                size: 0.45,
                transparent: true,
                opacity: 0.75,
                fog: false
            }));
            scene.add(starsMesh);
        }

        function createDriftingFogLayers() {
            const fogMat = new THREE.MeshBasicMaterial({
                color: 0x1e293b,
                transparent: true,
                opacity: 0.13,
                depthWrite: false,
                side: THREE.DoubleSide
            });
            for (let i = 0; i < 14; i++) {
                const plane = new THREE.Mesh(new THREE.PlaneGeometry(26, 5), fogMat);
                plane.position.set((Math.random() - 0.5) * 95, 1.8 + Math.random() * 2, (Math.random() - 0.5) * 95);
                plane.rotation.y = Math.random() * Math.PI;
                scene.add(plane);
                fogLayers.push({
                    mesh: plane,
                    speedX: (Math.random() - 0.5) * 0.85,
                    speedZ: (Math.random() - 0.5) * 0.85
                });
            }
        }

        function createFireflies() {
            const count = 130;
            const positions = new Float32Array(count * 3);
            for (let i = 0; i < count; i++) {
                positions[i * 3] = (Math.random() - 0.5) * 90;
                positions[i * 3 + 1] = 0.6 + Math.random() * 4.5;
                positions[i * 3 + 2] = (Math.random() - 0.5) * 90;
            }
            const geo = new THREE.BufferGeometry();
            geo.setAttribute('position', new THREE.BufferAttribute(positions, 3));
            fireflies = new THREE.Points(geo, new THREE.PointsMaterial({
                color: 0xa3e635,
                size: 0.16,
                transparent: true,
                opacity: 0.8
            }));
            scene.add(fireflies);
        }

        function createBloodPuddles() {
            const mat = new THREE.MeshBasicMaterial({ color: 0x5c0808, transparent: true, opacity: 0.75 });
            for (let i = 0; i < 32; i++) {
                const puddle = new THREE.Mesh(new THREE.CircleGeometry(0.45 + Math.random() * 0.85, 14), mat);
                puddle.rotation.x = -Math.PI / 2;
                puddle.position.set((Math.random() - 0.5) * 75, 0.02, (Math.random() - 0.5) * 75);
                scene.add(puddle);
            }
        }

        function createRuinedForest() {
            const barkMat = new THREE.MeshStandardMaterial({ map: createBarkTexture(), roughness: 0.95 });

            // Yıkık dökük kuru ağaçlar
            for (let i = 0; i < 65; i++) {
                const angle = Math.random() * Math.PI * 2;
                const dist = 8 + Math.random() * 56;
                const x = Math.cos(angle) * dist;
                const z = Math.sin(angle) * dist;

                const treeGroup = new THREE.Group();
                const height = 4.5 + Math.random() * 6.5;
                const radius = 0.28 + Math.random() * 0.25;

                const trunk = new THREE.Mesh(new THREE.CylinderGeometry(radius * 0.55, radius, height, 8), barkMat);
                trunk.position.y = height / 2;
                trunk.castShadow = true;
                trunk.receiveShadow = true;
                treeGroup.add(trunk);

                // Kırık dallar
                const branchCount = 2 + Math.floor(Math.random() * 3);
                for (let b = 0; b < branchCount; b++) {
                    const bLen = 1.2 + Math.random() * 1.8;
                    const branch = new THREE.Mesh(new THREE.CylinderGeometry(0.05, 0.12, bLen, 6), barkMat);
                    branch.position.y = height * (0.55 + Math.random() * 0.35);
                    branch.rotation.z = (Math.random() > 0.5 ? 1 : -1) * (0.5 + Math.random() * 0.6);
                    branch.rotation.y = Math.random() * Math.PI * 2;
                    treeGroup.add(branch);
                }

                treeGroup.position.set(x, 0, z);
                treeGroup.rotation.z = (Math.random() - 0.5) * 0.14;
                scene.add(treeGroup);
                obstacles.push({ x, z, r: radius + 0.35 });
            }

            // Yere devrilmiş kütükler
            for (let i = 0; i < 18; i++) {
                const angle = Math.random() * Math.PI * 2;
                const dist = 10 + Math.random() * 45;
                const x = Math.cos(angle) * dist;
                const z = Math.sin(angle) * dist;
                const logLen = 3.2 + Math.random() * 2.5;
                const log = new THREE.Mesh(new THREE.CylinderGeometry(0.24, 0.3, logLen, 8), barkMat);
                log.position.set(x, 0.24, z);
                log.rotation.z = Math.PI / 2;
                log.rotation.y = Math.random() * Math.PI;
                log.castShadow = true;
                log.receiveShadow = true;
                scene.add(log);
            }
        }

        function createRuinedWalls() {
            const stoneMat = new THREE.MeshStandardMaterial({ map: createStoneTexture(), roughness: 0.88 });
            const wallSpecs = [
                { x: -14, z: -16, ry: 0.4, w: 4.5, h: 1.8 },
                { x: 16, z: -14, ry: -0.5, w: 5.0, h: 2.1 },
                { x: -18, z: 15, ry: 1.1, w: 4.2, h: 1.7 },
                { x: 17, z: 18, ry: -0.9, w: 4.8, h: 1.9 }
            ];
            wallSpecs.forEach(w => {
                const wall = new THREE.Mesh(new THREE.BoxGeometry(w.w, w.h, 0.75), stoneMat);
                wall.position.set(w.x, w.h / 2, w.z);
                wall.rotation.y = w.ry;
                wall.castShadow = true;
                wall.receiveShadow = true;
                scene.add(wall);
                obstacles.push({ x: w.x, z: w.z, r: w.w * 0.45 });
            });
        }

        function createExplosiveBarrels() {
            barrels.forEach(b => scene.remove(b.group));
            barrels.length = 0;

            const barrelMat = new THREE.MeshStandardMaterial({ color: 0xdc2626, roughness: 0.35, metalness: 0.65 });
            const stripeMat = new THREE.MeshBasicMaterial({ color: 0xfacc15 });
            const capMat = new THREE.MeshStandardMaterial({ color: 0x1f2937, roughness: 0.5, metalness: 0.8 });

            const positions = [
                { x: -9, z: -11 }, { x: 13, z: -15 }, { x: -16, z: 9 },
                { x: 11, z: 13 }, { x: 0, z: -24 }, { x: 24, z: 2 },
                { x: -25, z: -4 }, { x: 4, z: 25 }
            ];

            positions.forEach(pos => {
                const group = new THREE.Group();
                const body = new THREE.Mesh(new THREE.CylinderGeometry(0.42, 0.42, 1.18, 16), barrelMat);
                body.position.y = 0.59;
                body.castShadow = true;
                body.receiveShadow = true;
                group.add(body);

                const stripe1 = new THREE.Mesh(new THREE.CylinderGeometry(0.43, 0.43, 0.14, 16), stripeMat);
                stripe1.position.y = 0.85;
                const stripe2 = new THREE.Mesh(new THREE.CylinderGeometry(0.43, 0.43, 0.14, 16), stripeMat);
                stripe2.position.y = 0.35;
                const rim = new THREE.Mesh(new THREE.CylinderGeometry(0.44, 0.44, 0.06, 16), capMat);
                rim.position.y = 1.18;

                const warnLight = new THREE.PointLight(0xef4444, 0.65, 3.5);
                warnLight.position.y = 1.3;

                group.add(stripe1, stripe2, rim, warnLight);
                group.position.set(pos.x, 0, pos.z);
                scene.add(group);

                const barrelObj = { group, bodyMesh: body, x: pos.x, z: pos.z, active: true };
                body.userData.barrel = barrelObj;
                barrels.push(barrelObj);
            });
        }

        function explodeBarrel(barrel) {
            if (!barrel.active) return;
            barrel.active = false;
            const pos = barrel.group.position.clone();
            scene.remove(barrel.group);

            const idx = barrels.indexOf(barrel);
            if (idx !== -1) barrels.splice(idx, 1);

            sfx.playExplosion();
            spawnBloodParticles(pos.clone().add(new THREE.Vector3(0, 0.6, 0)), 28);

            const blast = new THREE.Mesh(
                new THREE.SphereGeometry(0.6, 16, 16),
                new THREE.MeshBasicMaterial({ color: 0xf97316, transparent: true, opacity: 0.95 })
            );
            blast.position.copy(pos).add(new THREE.Vector3(0, 0.5, 0));
            scene.add(blast);

            let scale = 1;
            const interval = setInterval(() => {
                scale += 1.6;
                blast.scale.setScalar(scale);
                blast.material.opacity -= 0.15;
                if (blast.material.opacity <= 0) {
                    clearInterval(interval);
                    scene.remove(blast);
                }
            }, 28);

            for (let i = zombies.length - 1; i >= 0; i--) {
                const z = zombies[i];
                if (z.group.position.distanceTo(pos) <= 9.0) {
                    damageZombie(z, 8, z.group.position.clone().add(new THREE.Vector3(0, 1, 0)), false);
                }
            }
            showToast('PATLAYICI VARİL İNFİLAK ETTİ!');
        }

        /**
         * DETAYLI 3D SİLAH MODELLERİ + TAKTİK ELDİVENLİ ELLER + KIRMIZI LAZER
         */
        function buildWeaponModel(weaponId) {
            if (gunGroup) camera.remove(gunGroup);
            minigunBarrels = null;

            gunGroup = new THREE.Group();
            const wData = WEAPONS.find(w => w.id === weaponId) || WEAPONS[0];

            const darkMetal = new THREE.MeshStandardMaterial({ color: 0x161920, roughness: 0.28, metalness: 0.88 });
            const gunMetal = new THREE.MeshStandardMaterial({ color: 0x334155, roughness: 0.32, metalness: 0.85 });
            const chromeMat = new THREE.MeshStandardMaterial({ color: 0x94a3b8, roughness: 0.18, metalness: 0.95 });
            const woodMat = new THREE.MeshStandardMaterial({ color: 0x5c3419, roughness: 0.72, metalness: 0.1 });
            const glowMat = new THREE.MeshBasicMaterial({ color: wData.color });
            const holoGlassMat = new THREE.MeshBasicMaterial({ color: 0x38bdf8, transparent: true, opacity: 0.35 });
            const gloveMat = new THREE.MeshStandardMaterial({ color: 0x111827, roughness: 0.85, metalness: 0.1 });
            const sleeveMat = new THREE.MeshStandardMaterial({ color: 0x1f2937, roughness: 0.9 });

            // Taktik Kırmızı Lazer Işını
            const laserUnit = new THREE.Mesh(new THREE.BoxGeometry(0.022, 0.022, 0.07), darkMetal);
            laserUnit.position.set(0.042, 0.015, -0.22);
            const laserBeam = new THREE.Mesh(
                new THREE.CylinderGeometry(0.0018, 0.0018, 16, 6),
                new THREE.MeshBasicMaterial({ color: 0xef4444, transparent: true, opacity: 0.4 })
            );
            laserBeam.rotation.x = Math.PI / 2;
            laserBeam.position.set(0.042, 0.015, -8.25);
            gunGroup.add(laserUnit, laserBeam);

            // Eldivenli Sağ ve Sol Eller
            const rightHand = new THREE.Group();
            const rPalm = new THREE.Mesh(new THREE.BoxGeometry(0.058, 0.068, 0.085), gloveMat);
            const rArm = new THREE.Mesh(new THREE.CylinderGeometry(0.036, 0.044, 0.34, 10), sleeveMat);
            rArm.rotation.x = Math.PI / 2.35;
            rArm.position.set(0.01, -0.08, 0.16);
            rightHand.add(rPalm, rArm);
            rightHand.position.set(0.005, -0.068, -0.02);
            gunGroup.add(rightHand);

            const leftHand = new THREE.Group();
            const lPalm = new THREE.Mesh(new THREE.BoxGeometry(0.056, 0.062, 0.08), gloveMat);
            const lArm = new THREE.Mesh(new THREE.CylinderGeometry(0.034, 0.042, 0.36, 10), sleeveMat);
            lArm.rotation.set(Math.PI / 2.2, 0, -0.35);
            lArm.position.set(-0.08, -0.09, 0.16);
            leftHand.add(lPalm, lArm);

            let barrelTipZ = -0.35;

            if (weaponId === 'pistol') {
                leftHand.position.set(-0.025, -0.075, -0.04);
                const slide = new THREE.Mesh(new THREE.BoxGeometry(0.056, 0.062, 0.29), gunMetal);
                slide.position.set(0, 0.02, -0.08);
                const suppressor = new THREE.Mesh(new THREE.CylinderGeometry(0.016, 0.016, 0.15, 12), darkMetal);
                suppressor.rotation.x = Math.PI / 2;
                suppressor.position.set(0, 0.024, -0.29);
                const grip = new THREE.Mesh(new THREE.BoxGeometry(0.05, 0.14, 0.068), darkMetal);
                grip.position.set(0, -0.06, 0.02);
                grip.rotation.x = 0.24;
                const redDot = new THREE.Mesh(new THREE.BoxGeometry(0.028, 0.022, 0.036), glowMat);
                redDot.position.set(0, 0.06, -0.05);
                const underRail = new THREE.Mesh(new THREE.BoxGeometry(0.04, 0.016, 0.11), darkMetal);
                underRail.position.set(0, -0.015, -0.15);
                gunGroup.add(slide, suppressor, grip, redDot, underRail);
                barrelTipZ = -0.37;
            } else if (weaponId === 'shotgun') {
                leftHand.position.set(-0.01, -0.035, -0.34);
                const body = new THREE.Mesh(new THREE.BoxGeometry(0.068, 0.078, 0.34), darkMetal);
                body.position.set(0, 0.01, -0.06);
                const barrel = new THREE.Mesh(new THREE.CylinderGeometry(0.018, 0.018, 0.62, 14), chromeMat);
                barrel.rotation.x = Math.PI / 2;
                barrel.position.set(0, 0.032, -0.32);
                const magTube = new THREE.Mesh(new THREE.CylinderGeometry(0.015, 0.015, 0.52, 12), gunMetal);
                magTube.rotation.x = Math.PI / 2;
                magTube.position.set(0, 0.002, -0.28);
                const pump = new THREE.Mesh(new THREE.CylinderGeometry(0.029, 0.029, 0.21, 14), woodMat);
                pump.rotation.x = Math.PI / 2;
                pump.position.set(0, -0.002, -0.34);
                const stock = new THREE.Mesh(new THREE.BoxGeometry(0.056, 0.11, 0.26), woodMat);
                stock.position.set(0, -0.03, 0.19);
                stock.rotation.x = 0.15;
                gunGroup.add(body, barrel, magTube, pump, stock);
                barrelTipZ = -0.64;
            } else if (weaponId === 'smg') {
                leftHand.position.set(-0.01, -0.03, -0.24);
                const body = new THREE.Mesh(new THREE.BoxGeometry(0.062, 0.082, 0.36), darkMetal);
                body.position.set(0, 0.01, -0.1);
                const suppressor = new THREE.Mesh(new THREE.CylinderGeometry(0.023, 0.023, 0.22, 16), gunMetal);
                suppressor.rotation.x = Math.PI / 2;
                suppressor.position.set(0, 0.02, -0.37);
                const mag = new THREE.Mesh(new THREE.BoxGeometry(0.032, 0.21, 0.048), chromeMat);
                mag.position.set(0, -0.11, -0.15);
                mag.rotation.x = -0.18;
                const foregrip = new THREE.Mesh(new THREE.BoxGeometry(0.028, 0.08, 0.032), darkMetal);
                foregrip.position.set(0, -0.05, -0.24);
                const holoFrame = new THREE.Mesh(new THREE.BoxGeometry(0.044, 0.038, 0.058), darkMetal);
                holoFrame.position.set(0, 0.068, -0.08);
                const holoGlass = new THREE.Mesh(new THREE.PlaneGeometry(0.034, 0.026), holoGlassMat);
                holoGlass.position.set(0, 0.068, -0.08);
                const holoDot = new THREE.Mesh(new THREE.SphereGeometry(0.004, 8, 8), glowMat);
                holoDot.position.set(0, 0.068, -0.08);
                gunGroup.add(body, suppressor, mag, foregrip, holoFrame, holoGlass, holoDot);
                barrelTipZ = -0.49;
            } else if (weaponId === 'ak47') {
                leftHand.position.set(-0.01, -0.03, -0.28);
                const receiver = new THREE.Mesh(new THREE.BoxGeometry(0.064, 0.082, 0.42), darkMetal);
                receiver.position.set(0, 0.01, -0.1);
                const woodGuard = new THREE.Mesh(new THREE.BoxGeometry(0.056, 0.062, 0.21), woodMat);
                woodGuard.position.set(0, 0.012, -0.32);
                const barrel = new THREE.Mesh(new THREE.CylinderGeometry(0.013, 0.013, 0.34, 12), chromeMat);
                barrel.rotation.x = Math.PI / 2;
                barrel.position.set(0, 0.024, -0.52);
                const muzzleBrake = new THREE.Mesh(new THREE.CylinderGeometry(0.018, 0.016, 0.05, 12), darkMetal);
                muzzleBrake.rotation.x = Math.PI / 2;
                muzzleBrake.position.set(0, 0.024, -0.69);
                const bananaMag = new THREE.Mesh(new THREE.BoxGeometry(0.038, 0.23, 0.068), woodMat);
                bananaMag.position.set(0, -0.11, -0.14);
                bananaMag.rotation.x = -0.32;
                const stock = new THREE.Mesh(new THREE.BoxGeometry(0.052, 0.1, 0.24), woodMat);
                stock.position.set(0, -0.02, 0.2);
                gunGroup.add(receiver, woodGuard, barrel, muzzleBrake, bananaMag, stock);
                barrelTipZ = -0.71;
            } else if (weaponId === 'sniper') {
                leftHand.position.set(-0.01, -0.025, -0.32);
                const chassis = new THREE.Mesh(
                    new THREE.BoxGeometry(0.064, 0.082, 0.56),
                    new THREE.MeshStandardMaterial({ color: 0x3f4e3c, roughness: 0.55, metalness: 0.3 })
                );
                chassis.position.set(0, 0.0, -0.12);
                const barrel = new THREE.Mesh(new THREE.CylinderGeometry(0.016, 0.02, 0.68, 14), chromeMat);
                barrel.rotation.x = Math.PI / 2;
                barrel.position.set(0, 0.026, -0.58);
                const bipod1 = new THREE.Mesh(new THREE.CylinderGeometry(0.006, 0.006, 0.16, 8), darkMetal);
                bipod1.position.set(-0.035, -0.06, -0.45);
                bipod1.rotation.z = 0.25;
                const bipod2 = new THREE.Mesh(new THREE.CylinderGeometry(0.006, 0.006, 0.16, 8), darkMetal);
                bipod2.position.set(0.035, -0.06, -0.45);
                bipod2.rotation.z = -0.25;
                const scopeTube = new THREE.Mesh(new THREE.CylinderGeometry(0.027, 0.027, 0.34, 16), darkMetal);
                scopeTube.rotation.x = Math.PI / 2;
                scopeTube.position.set(0, 0.084, -0.12);
                const scopeLens = new THREE.Mesh(new THREE.CircleGeometry(0.024, 16), glowMat);
                scopeLens.position.set(0, 0.084, -0.292);
                scopeLens.rotation.y = Math.PI;
                gunGroup.add(chassis, barrel, bipod1, bipod2, scopeTube, scopeLens);
                barrelTipZ = -0.94;
            } else if (weaponId === 'plasma') {
                leftHand.position.set(-0.01, -0.035, -0.26);
                const coreBody = new THREE.Mesh(new THREE.BoxGeometry(0.088, 0.098, 0.48), chromeMat);
                coreBody.position.set(0, 0.01, -0.14);
                const coil1 = new THREE.Mesh(new THREE.TorusGeometry(0.056, 0.009, 10, 22), glowMat);
                coil1.position.set(0, 0.01, -0.26);
                const coil2 = new THREE.Mesh(new THREE.TorusGeometry(0.05, 0.009, 10, 22), glowMat);
                coil2.position.set(0, 0.01, -0.39);
                const plasmaCore = new THREE.Mesh(new THREE.CylinderGeometry(0.024, 0.024, 0.46, 14), glowMat);
                plasmaCore.rotation.x = Math.PI / 2;
                plasmaCore.position.set(0, 0.01, -0.22);
                gunGroup.add(coreBody, coil1, coil2, plasmaCore);
                barrelTipZ = -0.46;
            } else if (weaponId === 'minigun') {
                leftHand.position.set(-0.02, 0.04, -0.24);
                const motorHousing = new THREE.Mesh(new THREE.CylinderGeometry(0.068, 0.068, 0.34, 16), darkMetal);
                motorHousing.rotation.x = Math.PI / 2;
                motorHousing.position.set(0, -0.01, -0.1);
                const topHandle = new THREE.Mesh(new THREE.BoxGeometry(0.09, 0.02, 0.18), chromeMat);
                topHandle.position.set(0, 0.075, -0.16);
                const ammoDrum = new THREE.Mesh(new THREE.CylinderGeometry(0.075, 0.075, 0.16, 16), gunMetal);
                ammoDrum.position.set(0.08, -0.07, -0.12);

                minigunBarrels = new THREE.Group();
                minigunBarrels.position.set(0, -0.01, -0.45);
                for (let b = 0; b < 6; b++) {
                    const ang = (b / 6) * Math.PI * 2;
                    const singleBarrel = new THREE.Mesh(new THREE.CylinderGeometry(0.011, 0.011, 0.44, 10), chromeMat);
                    singleBarrel.rotation.x = Math.PI / 2;
                    singleBarrel.position.set(Math.cos(ang) * 0.036, Math.sin(ang) * 0.036, 0);
                    minigunBarrels.add(singleBarrel);
                }
                const clamp = new THREE.Mesh(new THREE.TorusGeometry(0.044, 0.008, 8, 18), glowMat);
                clamp.position.set(0, 0, -0.16);
                minigunBarrels.add(clamp);

                gunGroup.add(motorHousing, topHandle, ammoDrum, minigunBarrels);
                barrelTipZ = -0.68;
            }

            gunGroup.add(leftHand);

            // Namlu Alevi & Işığı
            muzzleFlash = new THREE.Mesh(
                new THREE.SphereGeometry(0.072, 10, 10),
                new THREE.MeshBasicMaterial({
                    color: weaponId === 'plasma' ? 0x06b6d4 : 0xfbbf24,
                    transparent: true,
                    opacity: 0
                })
            );
            muzzleFlash.position.set(0, 0.022, barrelTipZ);

            muzzleLight = new THREE.PointLight(weaponId === 'plasma' ? 0x06b6d4 : 0xffaa00, 0, 7);
            muzzleLight.position.set(0, 0.04, barrelTipZ - 0.1);

            gunGroup.add(muzzleFlash, muzzleLight);
            gunGroup.position.set(0.28, -0.24, -0.45);
            camera.add(gunGroup);
        }

        /**
         * ANİMASYONLU 3D ZOMBİ OLUŞTURMA (Yeşil Vücut + Parlayan Beyaz Gözler)
         */
        function spawnZombie() {
            const group = new THREE.Group();

            // Dalga ilerledikçe özel zombi türleri (Koşucu & Dev)
            let type = 'normal';
            const roll = Math.random();
            if (state.wave >= 2 && roll < 0.28) type = 'runner';
            if (state.wave >= 3 && roll > 0.82) type = 'tank';

            const scale = type === 'tank' ? 1.45 : (type === 'runner' ? 0.94 : 1.0);
            const maxHp = type === 'tank' ? 9 : (type === 'runner' ? 2 : 3); // Başlangıç tabancasıyla normal zombi 3 vuruşta ölür
            const speed = type === 'runner' ? (4.4 + state.wave * 0.18) : (type === 'tank' ? 1.85 : (2.6 + state.wave * 0.14));

            const skinColor = type === 'runner' ? 0x3f7d20 : (type === 'tank' ? 0x1e3f18 : 0x2d6a2e);
            const shirtColor = type === 'tank' ? 0x292524 : 0x1e293b;
            const pantsColor = 0x171717;

            const skinMat = new THREE.MeshStandardMaterial({ color: skinColor, roughness: 0.75 });
            const shirtMat = new THREE.MeshStandardMaterial({ color: shirtColor, roughness: 0.9 });
            const pantsMat = new THREE.MeshStandardMaterial({ color: pantsColor, roughness: 0.9 });
            const whiteEyeMat = new THREE.MeshBasicMaterial({ color: 0xffffff });

            // Bacaklar
            const leftLeg = new THREE.Mesh(new THREE.BoxGeometry(0.18, 0.72, 0.18), pantsMat);
            leftLeg.position.set(-0.13, 0.36, 0);
            leftLeg.castShadow = true;
            const rightLeg = new THREE.Mesh(new THREE.BoxGeometry(0.18, 0.72, 0.18), pantsMat);
            rightLeg.position.set(0.13, 0.36, 0);
            rightLeg.castShadow = true;

            // Gövde
            const torso = new THREE.Mesh(new THREE.BoxGeometry(0.52, 0.75, 0.28), shirtMat);
            torso.position.set(0, 1.08, 0);
            torso.castShadow = true;

            // Kafa & Parlayan Beyaz Gözler
            const head = new THREE.Mesh(new THREE.BoxGeometry(0.38, 0.38, 0.38), skinMat);
            head.position.set(0, 1.66, 0);
            head.castShadow = true;

            const leftEye = new THREE.Mesh(new THREE.SphereGeometry(0.042, 8, 8), whiteEyeMat);
            leftEye.position.set(-0.09, 1.69, 0.195);
            const rightEye = new THREE.Mesh(new THREE.SphereGeometry(0.042, 8, 8), whiteEyeMat);
            rightEye.position.set(0.09, 1.69, 0.195);

            // İleri Uzanan Zombi Kolları
            const leftArm = new THREE.Mesh(new THREE.BoxGeometry(0.15, 0.15, 0.65), skinMat);
            leftArm.position.set(-0.35, 1.32, 0.26);
            leftArm.castShadow = true;
            const rightArm = new THREE.Mesh(new THREE.BoxGeometry(0.15, 0.15, 0.65), skinMat);
            rightArm.position.set(0.35, 1.32, 0.26);
            rightArm.castShadow = true;

            group.add(leftLeg, rightLeg, torso, head, leftEye, rightEye, leftArm, rightArm);
            group.scale.setScalar(scale);

            // Oyuncunun etrafında sislerin içinden doğma pozisyonu
            const angle = Math.random() * Math.PI * 2;
            const dist = 26 + Math.random() * 18;
            group.position.set(
                Math.max(-68, Math.min(68, camera.position.x + Math.cos(angle) * dist)),
                0,
                Math.max(-68, Math.min(68, camera.position.z + Math.sin(angle) * dist))
            );
            scene.add(group);

            // Raycast Vuruş Kutuları (Gövde ve Kafa ayrı -> Kafadan vuruş 2x hasar)
            const hitBox = new THREE.Mesh(
                new THREE.BoxGeometry(0.75 * scale, 1.38 * scale, 0.65 * scale),
                new THREE.MeshBasicMaterial({ visible: false })
            );
            hitBox.position.copy(group.position).add(new THREE.Vector3(0, 0.72 * scale, 0));
            scene.add(hitBox);

            const headBox = new THREE.Mesh(
                new THREE.BoxGeometry(0.5 * scale, 0.48 * scale, 0.5 * scale),
                new THREE.MeshBasicMaterial({ visible: false })
            );
            headBox.position.copy(group.position).add(new THREE.Vector3(0, 1.66 * scale, 0));
            scene.add(headBox);

            const zombieObj = {
                group,
                hitBox,
                headBox,
                leftLeg,
                rightLeg,
                leftArm,
                rightArm,
                hp: maxHp,
                maxHp,
                speed,
                scale,
                type,
                walkPhase: Math.random() * Math.PI * 2,
                attackCooldown: 0
            };

            hitBox.userData.zombie = zombieObj;
            hitBox.userData.isHead = false;
            headBox.userData.zombie = zombieObj;
            headBox.userData.isHead = true;

            zombies.push(zombieObj);
        }

        /**
         * ATEŞ ETME, HASAR VE İKMAL SİSTEMİ
         */
        function shootWeapon() {
            if (!state.isPlaying || state.isPaused || state.isDead || state.isMarketOpen) return;

            const wData = WEAPONS.find(w => w.id === state.currentWeaponId) || WEAPONS[0];
            const now = performance.now();
            if (now - state.lastShotTime < wData.fireRate) return;
            state.lastShotTime = now;

            sfx.playShoot(wData.id);
            gunRecoil = wData.recoil;
            if (muzzleFlash) muzzleFlash.material.opacity = 0.95;
            if (muzzleLight) muzzleLight.intensity = 3.5;

            const spreadMult = state.isAiming ? 0.35 : 1.0;
            const raycaster = new THREE.Raycaster();
            const allTargets = [
                ...zombies.map(z => z.hitBox),
                ...zombies.map(z => z.headBox),
                ...barrels.filter(b => b.active).map(b => b.bodyMesh)
            ];

            for (let p = 0; p < wData.pellets; p++) {
                const spreadX = (Math.random() - 0.5) * wData.spread * spreadMult;
                const spreadY = (Math.random() - 0.5) * wData.spread * spreadMult;
                raycaster.setFromCamera(new THREE.Vector2(spreadX, spreadY), camera);

                const intersects = raycaster.intersectObjects(allTargets, false);
                let endPoint = raycaster.ray.origin.clone().addScaledVector(raycaster.ray.direction, 48);

                if (intersects.length > 0) {
                    const hit = intersects[0];
                    endPoint = hit.point;

                    if (hit.object.userData.barrel) {
                        explodeBarrel(hit.object.userData.barrel);
                    } else {
                        const hitZombie = hit.object.userData.zombie;
                        const isHeadshot = Boolean(hit.object.userData.isHead);
                        if (hitZombie && hitZombie.hp > 0) {
                            const finalDamage = isHeadshot ? wData.damage * 2 : wData.damage;
                            damageZombie(hitZombie, finalDamage, hit.point, isHeadshot);
                            showHitmarker(isHeadshot);
                        }
                    }
                }

                const muzzleWorldPos = new THREE.Vector3();
                muzzleFlash.getWorldPosition(muzzleWorldPos);
                spawnTracer(muzzleWorldPos, endPoint, wData.color);
            }
        }

        function damageZombie(zombie, amount, hitPoint, isHead) {
            zombie.hp -= amount;
            spawnBloodParticles(hitPoint, isHead ? 14 : 7);

            // Kısa süreli kırmızı vuruş parlaması
            zombie.group.children.forEach(part => {
                if (part.material && part.material.emissive) {
                    part.material.emissive.setHex(0x7f1d1d);
                    setTimeout(() => {
                        if (part.material && part.material.emissive) part.material.emissive.setHex(0x000000);
                    }, 90);
                }
            });

            if (zombie.hp <= 0) {
                if (isHead) sfx.playHeadshot();
                sfx.playZombieDie();
                spawnBloodParticles(hitPoint, isHead ? 28 : 18);

                // Yerde kalıcı kan birikintisi
                const puddle = new THREE.Mesh(
                    new THREE.CircleGeometry(0.55 + Math.random() * 0.4, 14),
                    new THREE.MeshBasicMaterial({ color: 0x5c0606, transparent: true, opacity: 0.78 })
                );
                puddle.rotation.x = -Math.PI / 2;
                puddle.position.set(zombie.group.position.x, 0.025, zombie.group.position.z);
                scene.add(puddle);

                // İkmal Sandığı Düşürme Şansı (%28)
                if (Math.random() < 0.28) {
                    spawnPickup(zombie.group.position.clone());
                }

                scene.remove(zombie.hitBox);
                scene.remove(zombie.headBox);
                const idx = zombies.indexOf(zombie);
                if (idx !== -1) zombies.splice(idx, 1);

                dyingZombies.push({ group: zombie.group, timer: 1.5 });

                // Kombo ve Ödül: Her zombi öldürmede 1 TL!
                state.combo += 1;
                state.comboTimer = 4.2;
                updateComboUI();

                state.kills += 1;
                state.money += 1;

                // Her 10 leşte yeni dalga
                const newWave = Math.floor(state.kills / 10) + 1;
                if (newWave > state.wave) {
                    state.wave = newWave;
                    showToast(`DALGA ${state.wave} BAŞLADI!`);
                }

                if (state.wave > state.bestWave) state.bestWave = state.wave;
                if (state.kills > state.bestKills) state.bestKills = state.kills;

                saveProgress();
                updateHUD();
                showMoneyGain('+1 TL');
            }
        }

        function spawnPickup(pos) {
            const type = Math.random() < 0.65 ? 'medkit' : 'grenade';
            const group = new THREE.Group();

            const boxMat = new THREE.MeshStandardMaterial({
                color: type === 'medkit' ? 0x10b981 : 0xf97316,
                emissive: type === 'medkit' ? 0x065f46 : 0x9a3412,
                roughness: 0.3
            });
            const crate = new THREE.Mesh(new THREE.BoxGeometry(0.45, 0.35, 0.45), boxMat);
            crate.position.y = 0.35;
            group.add(crate);

            const light = new THREE.PointLight(type === 'medkit' ? 0x10b981 : 0xf97316, 1.2, 4);
            light.position.y = 0.6;
            group.add(light);

            group.position.set(pos.x, 0, pos.z);
            scene.add(group);
            pickups.push({ group, type, life: 25 });
        }

        function throwGrenade() {
            if (!state.isPlaying || state.isPaused || state.isDead || state.isMarketOpen) return;
            if (state.grenades <= 0) {
                showToast('El bomban kalmadı!');
                return;
            }
            state.grenades -= 1;
            updateHUD();

            const mesh = new THREE.Mesh(
                new THREE.SphereGeometry(0.14, 12, 12),
                new THREE.MeshStandardMaterial({ color: 0x3f6212, metalness: 0.6, roughness: 0.4 })
            );
            mesh.position.copy(camera.position).add(new THREE.Vector3(0, -0.1, 0));
            scene.add(mesh);

            const dir = new THREE.Vector3();
            camera.getWorldDirection(dir);
            const vel = dir.multiplyScalar(15).add(new THREE.Vector3(0, 5.5, 0));

            activeGrenades.push({ mesh, vel, timer: 1.8 });
        }

        function detonateGrenade(g) {
            scene.remove(g.mesh);
            sfx.playExplosion();
            spawnBloodParticles(g.mesh.position, 26);

            const blast = new THREE.Mesh(
                new THREE.SphereGeometry(0.6, 16, 16),
                new THREE.MeshBasicMaterial({ color: 0xfb923c, transparent: true, opacity: 0.9 })
            );
            blast.position.copy(g.mesh.position);
            scene.add(blast);

            let scale = 1;
            const iv = setInterval(() => {
                scale += 1.5;
                blast.scale.setScalar(scale);
                blast.material.opacity -= 0.15;
                if (blast.material.opacity <= 0) {
                    clearInterval(iv);
                    scene.remove(blast);
                }
            }, 28);

            for (let i = zombies.length - 1; i >= 0; i--) {
                const z = zombies[i];
                if (z.group.position.distanceTo(g.mesh.position) <= 8.5) {
                    damageZombie(z, 7, z.group.position.clone().add(new THREE.Vector3(0, 1, 0)), false);
                }
            }
        }

        function spawnBloodParticles(origin, count) {
            const mat = new THREE.MeshBasicMaterial({ color: 0xb91c1c });
            const geo = new THREE.BoxGeometry(0.07, 0.07, 0.07);
            for (let i = 0; i < count; i++) {
                const m = new THREE.Mesh(geo, mat);
                m.position.copy(origin);
                scene.add(m);
                particles.push({
                    mesh: m,
                    vel: new THREE.Vector3((Math.random() - 0.5) * 6, 1.5 + Math.random() * 4, (Math.random() - 0.5) * 6),
                    life: 0.45 + Math.random() * 0.35
                });
            }
        }

        function spawnTracer(start, end, colorHex) {
            const geo = new THREE.BufferGeometry().setFromPoints([start, end]);
            const mat = new THREE.LineBasicMaterial({ color: colorHex, transparent: true, opacity: 0.85 });
            const line = new THREE.Line(geo, mat);
            scene.add(line);
            tracers.push({ line, life: 0.07 });
        }

        /**
         * HUD, MİNİ RADAR VE ARAYÜZ GÜNCELLEMELERİ
         */
        function updateHUD() {
            document.getElementById('hud-hp-text').textContent = `${Math.max(0, state.hp)} / ${state.maxHp}`;
            const hpPct = Math.max(0, (state.hp / state.maxHp) * 100);
            document.getElementById('hud-hp-bar').style.width = `${hpPct}%`;

            document.getElementById('hud-money').textContent = `${state.money} TL`;
            document.getElementById('hud-wave').textContent = state.wave;
            document.getElementById('hud-kills').textContent = state.kills;
            document.getElementById('hud-grenades').textContent = `[G] Bomba: ${state.grenades}`;
            document.getElementById('hud-flashlight').textContent = `[T] Fener: ${state.flashlightOn ? 'AÇIK' : 'KAPALI'}`;

            const wData = WEAPONS.find(w => w.id === state.currentWeaponId) || WEAPONS[0];
            document.getElementById('hud-weapon-name').textContent = wData.name;

            // Lobi ve Market bakiye senkronizasyonu
            document.getElementById('lobby-money').textContent = `${state.money} TL`;
            document.getElementById('lobby-best-wave').textContent = state.bestWave;
            document.getElementById('lobby-best-kills').textContent = state.bestKills;
            document.getElementById('market-money').textContent = `${state.money} TL`;
        }

        function showHitmarker(isHeadshot) {
            const hm = document.getElementById('hitmarker');
            hm.querySelectorAll('div').forEach(bar => {
                bar.className = isHeadshot
                    ? 'w-6 h-0.5 bg-red-500 rotate-45 absolute shadow-[0_0_8px_#ef4444]'
                    : 'w-5 h-0.5 bg-white rotate-45 absolute shadow-[0_0_6px_#fff]';
            });
            hm.children[1].classList.replace('rotate-45', '-rotate-45');
            hm.style.opacity = '1';
            clearTimeout(hm.hideTimer);
            hm.hideTimer = setTimeout(() => { hm.style.opacity = '0'; }, 110);
        }

        function showMoneyGain(text) {
            const el = document.getElementById('money-gain');
            el.textContent = text;
            el.style.opacity = '1';
            el.style.transform = 'translateY(-8px)';
            clearTimeout(el.hideTimer);
            el.hideTimer = setTimeout(() => {
                el.style.opacity = '0';
                el.style.transform = 'translateY(0)';
            }, 650);
        }

        function updateComboUI() {
            const box = document.getElementById('combo-display');
            const countEl = document.getElementById('combo-count');
            if (state.combo >= 2) {
                countEl.textContent = `KOMBO x${state.combo}`;
                box.style.opacity = '1';
                box.style.transform = 'scale(1.06)';
                setTimeout(() => { box.style.transform = 'scale(1)'; }, 120);
            } else {
                box.style.opacity = '0';
            }
        }

        function showToast(msg) {
            const el = document.getElementById('toast-msg');
            el.textContent = msg;
            el.style.opacity = '1';
            clearTimeout(el.timer);
            el.timer = setTimeout(() => { el.style.opacity = '0'; }, 2000);
        }

        function drawMinimap() {
            const canvas = document.getElementById('minimap-canvas');
            if (!canvas) return;
            const ctx = canvas.getContext('2d');
            if (canvas.width !== 120 || canvas.height !== 120) {
                canvas.width = 120;
                canvas.height = 120;
            }

            const cx = 60, cy = 60, range = 34;
            ctx.clearRect(0, 0, 120, 120);

            ctx.fillStyle = 'rgba(6, 20, 14, 0.88)';
            ctx.beginPath();
            ctx.arc(cx, cy, 58, 0, Math.PI * 2);
            ctx.fill();

            ctx.strokeStyle = 'rgba(16, 185, 129, 0.25)';
            ctx.lineWidth = 1;
            ctx.beginPath();
            ctx.arc(cx, cy, 28, 0, Math.PI * 2);
            ctx.stroke();

            const sinY = Math.sin(player.yaw);
            const cosY = Math.cos(player.yaw);

            const toRadar = (wx, wz) => {
                const dx = wx - camera.position.x;
                const dz = wz - camera.position.z;
                const rx = dx * cosY + dz * sinY;
                const rz = -dx * sinY + dz * cosY;
                return {
                    x: cx + (rx / range) * 54,
                    y: cy + (rz / range) * 54,
                    dist: Math.hypot(dx, dz)
                };
            };

            // Patlayıcı Variller (Sarı)
            barrels.forEach(b => {
                if (!b.active) return;
                const p = toRadar(b.x, b.z);
                if (p.dist < range) {
                    ctx.fillStyle = '#facc15';
                    ctx.beginPath();
                    ctx.arc(p.x, p.y, 2.8, 0, Math.PI * 2);
                    ctx.fill();
                }
            });

            // İkmal Sandıkları (Yeşil)
            pickups.forEach(pk => {
                const p = toRadar(pk.group.position.x, pk.group.position.z);
                if (p.dist < range) {
                    ctx.fillStyle = '#10b981';
                    ctx.beginPath();
                    ctx.arc(p.x, p.y, 3.2, 0, Math.PI * 2);
                    ctx.fill();
                }
            });

            // Zombiler (Kırmızı)
            zombies.forEach(z => {
                const p = toRadar(z.group.position.x, z.group.position.z);
                if (p.dist < range) {
                    ctx.fillStyle = z.type === 'tank' ? '#f43f5e' : '#ef4444';
                    ctx.beginPath();
                    ctx.arc(p.x, p.y, z.type === 'tank' ? 4 : 2.8, 0, Math.PI * 2);
                    ctx.fill();
                }
            });

            // Merkezde Oyuncu İleri Yön Üçgeni
            ctx.fillStyle = '#ffffff';
            ctx.beginPath();
            ctx.moveTo(cx, cy - 5);
            ctx.lineTo(cx - 4, cy + 4);
            ctx.lineTo(cx + 4, cy + 4);
            ctx.closePath();
            ctx.fill();
        }

        /**
         * SİLAH MARKETİ RENDER & SATIN ALMA
         */
        function renderWeaponMarket() {
            const list = document.getElementById('weapon-list');
            list.innerHTML = '';
            document.getElementById('market-money').textContent = `${state.money} TL`;

            WEAPONS.forEach(w => {
                const owned = state.ownedWeapons.includes(w.id);
                const equipped = state.currentWeaponId === w.id;
                const canAfford = state.money >= w.price;

                const card = document.createElement('div');
                card.className = `p-4 rounded-2xl border transition flex flex-col justify-between ${
                    equipped
                        ? 'bg-emerald-950/50 border-emerald-400/70 shadow-[0_0_20px_rgba(16,185,129,0.2)]'
                        : 'bg-gray-900/90 border-white/10 hover:border-white/25'
                }`;

                let btnHtml = '';
                if (equipped) {
                    btnHtml = `<button disabled class="w-full py-2.5 rounded-xl bg-emerald-600/40 text-emerald-200 font-black text-xs tracking-wider border border-emerald-400/40">KUŞANILDI ✓</button>`;
                } else if (owned) {
                    btnHtml = `<button data-action="equip" data-id="${w.id}" class="w-full py-2.5 rounded-xl bg-blue-600 hover:bg-blue-500 text-white font-black text-xs tracking-wider transition cursor-pointer">KUŞAN</button>`;
                } else {
                    btnHtml = `<button data-action="buy" data-id="${w.id}" ${canAfford ? '' : 'disabled'} class="w-full py-2.5 rounded-xl ${
                        canAfford
                            ? 'bg-emerald-600 hover:bg-emerald-500 text-white cursor-pointer'
                            : 'bg-gray-800 text-gray-500 cursor-not-allowed'
                    } font-black text-xs tracking-wider transition">SATIN AL (${w.price} TL)</button>`;
                }

                card.innerHTML = `
                    <div>
                        <div class="flex justify-between items-start gap-2 mb-1.5">
                            <h3 class="text-base font-extrabold text-white">${w.name}</h3>
                            <span class="px-2.5 py-0.5 rounded-full text-xs font-black ${
                                w.price === 0 ? 'bg-blue-500/20 text-blue-300' : 'bg-emerald-500/20 text-emerald-300'
                            }">${w.price === 0 ? 'ÜCRETSİZ' : w.price + ' TL'}</span>
                        </div>
                        <p class="text-xs text-gray-400 mb-3 leading-relaxed">${w.desc}</p>
                        <div class="flex gap-3 text-[11px] font-bold text-gray-300 mb-3">
                            <span>Hasar: <strong class="text-red-400">${w.damage * w.pellets}</strong></span>
                            <span>Hız: <strong class="text-amber-400">${Math.round(1000 / w.fireRate)}/sn</strong></span>
                            <span>Mermi: <strong class="text-cyan-400">∞</strong></span>
                        </div>
                    </div>
                    ${btnHtml}
                `;
                list.appendChild(card);
            });

            list.querySelectorAll('button[data-action]').forEach(btn => {
                btn.addEventListener('click', () => {
                    sfx.playClick();
                    const id = btn.getAttribute('data-id');
                    const action = btn.getAttribute('data-action');
                    const weapon = WEAPONS.find(item => item.id === id);
                    if (!weapon) return;

                    if (action === 'buy' && state.money >= weapon.price && !state.ownedWeapons.includes(id)) {
                        state.money -= weapon.price;
                        state.ownedWeapons.push(id);
                        state.currentWeaponId = id;
                        buildWeaponModel(id);
                        saveProgress();
                        updateHUD();
                        renderWeaponMarket();
                        showToast(`${weapon.name} satın alındı!`);
                    } else if (action === 'equip') {
                        state.currentWeaponId = id;
                        buildWeaponModel(id);
                        saveProgress();
                        updateHUD();
                        renderWeaponMarket();
                        showToast(`${weapon.name} kuşanıldı!`);
                    }
                });
            });
        }

        /**
         * OYUN DÖNGÜSÜ VE FİZİK GÜNCELLEMELERİ
         */
        function updateGame(dt) {
            // Lobi ekranındayken karanlık ormanda sinematik kamera gezintisi
            if (!state.isPlaying) {
                lobbyCamAngle += dt * 0.14;
                camera.position.set(Math.cos(lobbyCamAngle) * 18, 4.2, Math.sin(lobbyCamAngle) * 18);
                camera.lookAt(0, 1.5, 0);
                if (fireflies) fireflies.rotation.y += dt * 0.03;
                return;
            }

            if (state.isPaused || state.isDead || state.isMarketOpen) return;

            // Oyuncu Hareketi (WASD + Shift + Space)
            const forward = new THREE.Vector3(-Math.sin(player.yaw), 0, -Math.cos(player.yaw));
            const right = new THREE.Vector3(Math.cos(player.yaw), 0, -Math.sin(player.yaw));
            const moveDir = new THREE.Vector3();

            if (keys.w) moveDir.add(forward);
            if (keys.s) moveDir.sub(forward);
            if (keys.d) moveDir.add(right);
            if (keys.a) moveDir.sub(right);

            const isMoving = moveDir.lengthSq() > 0;
            if (isMoving) moveDir.normalize();

            const speed = keys.shift ? player.sprintSpeed : player.walkSpeed;
            let nextX = camera.position.x + moveDir.x * speed * dt;
            let nextZ = camera.position.z + moveDir.z * speed * dt;

            // Harita sınırı & Engel çarpışması
            nextX = Math.max(-72, Math.min(72, nextX));
            nextZ = Math.max(-72, Math.min(72, nextZ));

            for (const obs of obstacles) {
                const dx = nextX - obs.x;
                const dz = nextZ - obs.z;
                const dist = Math.hypot(dx, dz);
                const minDist = obs.r + 0.45;
                if (dist < minDist && dist > 0.001) {
                    nextX = obs.x + (dx / dist) * minDist;
                    nextZ = obs.z + (dz / dist) * minDist;
                }
            }

            camera.position.x = nextX;
            camera.position.z = nextZ;

            // Yerçekimi ve Zıplama
            if (keys.space && player.onGround) {
                player.velocity.y = player.jumpForce;
                player.onGround = false;
            }
            player.velocity.y -= 22 * dt;
            camera.position.y += player.velocity.y * dt;

            if (camera.position.y <= player.height) {
                camera.position.y = player.height;
                player.velocity.y = 0;
                player.onGround = true;
            }

            // Silah ADS (Sağ Tık Yakınlaştırma), Sallanma ve Geri Tepme Animasyonu
            const targetFov = state.isAiming ? (state.currentWeaponId === 'sniper' ? 24 : 52) : 72;
            camera.fov += (targetFov - camera.fov) * Math.min(1, dt * 14);
            camera.updateProjectionMatrix();

            const sniperScope = document.getElementById('sniper-scope');
            if (state.isAiming && state.currentWeaponId === 'sniper') {
                sniperScope.classList.remove('hidden');
                if (gunGroup) gunGroup.visible = false;
            } else {
                sniperScope.classList.add('hidden');
                if (gunGroup) gunGroup.visible = true;
            }

            if (gunGroup) {
                if (isMoving && player.onGround) {
                    walkBobTime += dt * (keys.shift ? 13 : 9);
                } else {
                    walkBobTime += dt * 1.8;
                }
                const bobX = Math.cos(walkBobTime * 0.5) * (isMoving ? 0.012 : 0.002);
                const bobY = Math.abs(Math.sin(walkBobTime)) * (isMoving ? 0.014 : 0.002);

                const baseGunX = state.isAiming ? 0.0 : 0.28;
                const baseGunY = state.isAiming ? -0.17 : -0.24;
                const baseGunZ = state.isAiming ? -0.36 : -0.45;

                gunRecoil = Math.max(0, gunRecoil - dt * 0.65);
                gunGroup.position.x += (baseGunX + bobX - gunGroup.position.x) * Math.min(1, dt * 16);
                gunGroup.position.y += (baseGunY - bobY - gunGroup.position.y) * Math.min(1, dt * 16);
                gunGroup.position.z = baseGunZ + gunRecoil;
                gunGroup.rotation.x = gunRecoil * 1.4;
            }

            if (muzzleFlash && muzzleFlash.material.opacity > 0) {
                muzzleFlash.material.opacity = Math.max(0, muzzleFlash.material.opacity - dt * 18);
            }
            if (muzzleLight && muzzleLight.intensity > 0) {
                muzzleLight.intensity = Math.max(0, muzzleLight.intensity - dt * 45);
            }

            // Basılı tutarak seri ateş etme
            if (state.isMouseDown) {
                shootWeapon();
                if (minigunBarrels) minigunBarrels.rotation.z += dt * 26;
            }

            // Kombo sayacı süresi
            if (state.combo > 0) {
                state.comboTimer -= dt;
                if (state.comboTimer <= 0) {
                    state.combo = 0;
                    updateComboUI();
                }
            }

            // Orman ambiyans sesi
            ambienceTimer -= dt;
            if (ambienceTimer <= 0) {
                sfx.playAmbienceHoot();
                ambienceTimer = 14 + Math.random() * 12;
            }

            // Ateşböcekleri ve sis tabakaları
            if (fireflies) fireflies.rotation.y += dt * 0.03;
            fogLayers.forEach(fl => {
                fl.mesh.position.x += fl.speedX * dt;
                fl.mesh.position.z += fl.speedZ * dt;
                if (fl.mesh.position.x > 50) fl.mesh.position.x = -50;
                if (fl.mesh.position.x < -50) fl.mesh.position.x = 50;
                if (fl.mesh.position.z > 50) fl.mesh.position.z = -50;
                if (fl.mesh.position.z < -50) fl.mesh.position.z = 50;
            });

            // Zombi Doğma Sistemi
            const maxZombies = Math.min(28, 7 + state.wave * 3);
            spawnTimer -= dt;
            if (spawnTimer <= 0 && zombies.length < maxZombies) {
                spawnZombie();
                spawnTimer = Math.max(0.55, 2.1 - state.wave * 0.14);
            }

            // Zombi Yapay Zekası, Animasyonu ve Saldırı
            for (let i = zombies.length - 1; i >= 0; i--) {
                const z = zombies[i];
                const toPlayer = new THREE.Vector3(
                    camera.position.x - z.group.position.x,
                    0,
                    camera.position.z - z.group.position.z
                );
                const dist = toPlayer.length();

                if (dist > 0.01) {
                    toPlayer.normalize();
                    z.group.rotation.y = Math.atan2(toPlayer.x, toPlayer.z);

                    if (dist > 1.35 * z.scale) {
                        z.group.position.x += toPlayer.x * z.speed * dt;
                        z.group.position.z += toPlayer.z * z.speed * dt;
                    }
                }

                // Yürüme ve Kol Sallanma Animasyonu
                z.walkPhase += dt * z.speed * 3.2;
                z.leftLeg.rotation.x = Math.sin(z.walkPhase) * 0.65;
                z.rightLeg.rotation.x = -Math.sin(z.walkPhase) * 0.65;
                z.leftArm.rotation.x = Math.sin(z.walkPhase) * 0.22 - 0.12;
                z.rightArm.rotation.x = -Math.sin(z.walkPhase) * 0.22 - 0.12;

                // Vuruş kutularını güncelle
                z.hitBox.position.set(z.group.position.x, 0.72 * z.scale, z.group.position.z);
                z.headBox.position.set(z.group.position.x, 1.66 * z.scale, z.group.position.z);

                // Oyuncuya yaklaşınca 1 can götürür (20 can üzerinden)
                z.attackCooldown = Math.max(0, z.attackCooldown - dt);
                if (dist <= 1.55 * z.scale && z.attackCooldown <= 0) {
                    z.attackCooldown = 1.05;
                    damagePlayer(1);
                }
            }

            // Ölen zombilerin geriye devrilmesi
            for (let i = dyingZombies.length - 1; i >= 0; i--) {
                const dz = dyingZombies[i];
                dz.timer -= dt;
                if (dz.group.rotation.x > -Math.PI / 2) {
                    dz.group.rotation.x = Math.max(-Math.PI / 2, dz.group.rotation.x - dt * 4.5);
                }
                if (dz.timer <= 0) {
                    scene.remove(dz.group);
                    dyingZombies.splice(i, 1);
                }
            }

            // İkmal Sandıkları Toplama
            for (let i = pickups.length - 1; i >= 0; i--) {
                const pk = pickups[i];
                pk.life -= dt;
                pk.group.rotation.y += dt * 2.2;
                pk.group.position.y = Math.sin(performance.now() * 0.004) * 0.1;

                const d = Math.hypot(camera.position.x - pk.group.position.x, camera.position.z - pk.group.position.z);
                if (d < 1.6) {
                    sfx.playPickup();
                    if (pk.type === 'medkit') {
                        state.hp = Math.min(state.maxHp, state.hp + 5);
                        showToast('+5 CAN PAKETİ ALINDI!');
                    } else {
                        state.grenades += 1;
                        showToast('+1 EL BOMBASI ALINDI!');
                    }
                    updateHUD();
                    scene.remove(pk.group);
                    pickups.splice(i, 1);
                    continue;
                }

                if (pk.life <= 0) {
                    scene.remove(pk.group);
                    pickups.splice(i, 1);
                }
            }

            // Atılan El Bombaları
            for (let i = activeGrenades.length - 1; i >= 0; i--) {
                const g = activeGrenades[i];
                g.timer -= dt;
                g.vel.y -= 18 * dt;
                g.mesh.position.addScaledVector(g.vel, dt);
                if (g.mesh.position.y <= 0.15) {
                    g.mesh.position.y = 0.15;
                    g.vel.y *= -0.45;
                    g.vel.x *= 0.7;
                    g.vel.z *= 0.7;
                }
                if (g.timer <= 0) {
                    detonateGrenade(g);
                    activeGrenades.splice(i, 1);
                }
            }

            // Kan Parçacıkları & Mermi İzleri
            for (let i = particles.length - 1; i >= 0; i--) {
                const p = particles[i];
                p.life -= dt;
                p.vel.y -= 14 * dt;
                p.mesh.position.addScaledVector(p.vel, dt);
                if (p.life <= 0) {
                    scene.remove(p.mesh);
                    particles.splice(i, 1);
                }
            }
            for (let i = tracers.length - 1; i >= 0; i--) {
                const t = tracers[i];
                t.life -= dt;
                t.line.material.opacity = Math.max(0, t.life / 0.07);
                if (t.life <= 0) {
                    scene.remove(t.line);
                    tracers.splice(i, 1);
                }
            }

            drawMinimap();
        }

        function damagePlayer(amount) {
            if (state.isDead) return;
            state.hp = Math.max(0, state.hp - amount);
            sfx.playPlayerHurt();
            updateHUD();

            const flash = document.getElementById('damage-flash');
            flash.style.opacity = '1';
            setTimeout(() => { flash.style.opacity = '0'; }, 180);

            if (state.hp <= 0) {
                triggerGameOver();
            }
        }

        function triggerGameOver() {
            state.isDead = true;
            state.isMouseDown = false;
            state.isAiming = false;
            saveProgress();

            if (document.pointerLockElement) {
                document.exitPointerLock();
            }

            document.getElementById('hud').classList.add('hidden');
            document.getElementById('crosshair-container').classList.add('hidden');
            document.getElementById('sniper-scope').classList.add('hidden');
            document.getElementById('lock-overlay').classList.add('hidden');

            document.getElementById('death-wave').textContent = state.wave;
            document.getElementById('death-kills').textContent = state.kills;
            document.getElementById('death-money').textContent = `${state.money} TL`;
            document.getElementById('death-menu').classList.remove('hidden');
        }

        function resetMatchState() {
            zombies.forEach(z => {
                scene.remove(z.group);
                scene.remove(z.hitBox);
                scene.remove(z.headBox);
            });
            zombies.length = 0;

            dyingZombies.forEach(dz => scene.remove(dz.group));
            dyingZombies.length = 0;

            pickups.forEach(pk => scene.remove(pk.group));
            pickups.length = 0;

            activeGrenades.forEach(g => scene.remove(g.mesh));
            activeGrenades.length = 0;

            createExplosiveBarrels();

            state.hp = state.maxHp;
            state.kills = 0;
            state.wave = 1;
            state.grenades = 1;
            state.combo = 0;
            state.comboTimer = 0;
            state.isDead = false;
            state.isPaused = false;
            state.isMarketOpen = false;
            state.isAiming = false;
            state.isMouseDown = false;
            updateComboUI();

            camera.position.set(0, player.height, 0);
            player.yaw = 0;
            player.pitch = 0;
            camera.rotation.set(0, 0, 0);

            // Başlangıçta 5 zombi ormanda belirsizce dolaşmaya başlar
            for (let i = 0; i < 5; i++) spawnZombie();
            updateHUD();
        }

        function startGame() {
            sfx.init();
            sfx.playClick();
            resetMatchState();
            state.isPlaying = true;

            document.getElementById('main-menu').classList.add('hidden');
            document.getElementById('death-menu').classList.add('hidden');
            document.getElementById('pause-menu').classList.add('hidden');
            document.getElementById('market-menu').classList.add('hidden');
            document.getElementById('settings-menu').classList.add('hidden');

            document.getElementById('hud').classList.remove('hidden');
            document.getElementById('crosshair-container').classList.remove('hidden');

            requestPointerLockSafe();
        }

        function returnToLobby() {
            sfx.playClick();
            state.isPlaying = false;
            state.isPaused = false;
            state.isDead = false;
            state.isMarketOpen = false;
            state.isMouseDown = false;
            state.isAiming = false;

            if (document.pointerLockElement) {
                document.exitPointerLock();
            }

            document.getElementById('hud').classList.add('hidden');
            document.getElementById('crosshair-container').classList.add('hidden');
            document.getElementById('sniper-scope').classList.add('hidden');
            document.getElementById('lock-overlay').classList.add('hidden');
            document.getElementById('pause-menu').classList.add('hidden');
            document.getElementById('death-menu').classList.add('hidden');
            document.getElementById('market-menu').classList.add('hidden');
            document.getElementById('settings-menu').classList.add('hidden');

            updateHUD();
            document.getElementById('main-menu').classList.remove('hidden');
        }

        function requestPointerLockSafe() {
            try {
                const el = renderer.domElement;
                if (el && el.requestPointerLock) {
                    el.requestPointerLock();
                }
            } catch (e) {
                // Sandbox iframe kısıtlamalarına karşı güvenli yakalama
            }
        }

        function toggleMarket(open) {
            sfx.playClick();
            state.isMarketOpen = open;
            const marketEl = document.getElementById('market-menu');
            if (open) {
                state.isMouseDown = false;
                state.isAiming = false;
                if (document.pointerLockElement) document.exitPointerLock();
                renderWeaponMarket();
                marketEl.classList.remove('hidden');
                document.getElementById('lock-overlay').classList.add('hidden');
            } else {
                marketEl.classList.add('hidden');
                if (state.isPlaying && !state.isPaused && !state.isDead) {
                    requestPointerLockSafe();
                }
            }
        }

        function openSettings(fromMenu) {
            sfx.playClick();
            state.previousMenu = fromMenu;
            document.getElementById('main-menu').classList.add('hidden');
            document.getElementById('pause-menu').classList.add('hidden');
            document.getElementById('death-menu').classList.add('hidden');
            document.getElementById('settings-menu').classList.remove('hidden');
        }

        function closeSettings() {
            sfx.playClick();
            document.getElementById('settings-menu').classList.add('hidden');
            if (state.previousMenu === 'main') {
                document.getElementById('main-menu').classList.remove('hidden');
            } else if (state.previousMenu === 'pause') {
                document.getElementById('pause-menu').classList.remove('hidden');
            } else if (state.previousMenu === 'death') {
                document.getElementById('death-menu').classList.remove('hidden');
            }
        }

        function onWindowResize() {
            if (!camera || !renderer) return;
            camera.aspect = window.innerWidth / window.innerHeight;
            camera.updateProjectionMatrix();
            renderer.setSize(window.innerWidth, window.innerHeight);
        }

        function animateLoop() {
            requestAnimationFrame(animateLoop);
            const now = performance.now();
            const dt = Math.min(0.08, (now - lastTime) / 1000);
            lastTime = now;

            updateGame(dt);
            if (renderer && scene && camera) {
                renderer.render(scene, camera);
            }
        }

        /**
         * OLAY DİNLEYİCİLERİ (KLAVYE, FARE VE MENÜ BUTONLARI)
         */
        function setupEventListeners() {
            // Standart Düz Fare Bakışı (Ters Eksen YOK!)
            document.addEventListener('mousemove', (e) => {
                if (!state.isPlaying || state.isPaused || state.isDead || state.isMarketOpen) return;
                if (document.pointerLockElement !== renderer.domElement) return;

                const sensFactor = 0.0022 * state.sensitivity * (state.isAiming ? 0.55 : 1.0);
                player.yaw -= e.movementX * sensFactor;
                player.pitch -= e.movementY * sensFactor; // Yukarı fare = yukarı bakış (düz eksen)
                player.pitch = Math.max(-Math.PI / 2.2, Math.min(Math.PI / 2.2, player.pitch));

                camera.rotation.y = player.yaw;
                camera.rotation.x = player.pitch;
            });

            // Fare Tıklamaları (Sol Tık Ateş, Sağ Tık Dürbün/Nişan)
            document.addEventListener('mousedown', (e) => {
                if (!state.isPlaying || state.isPaused || state.isDead || state.isMarketOpen) return;
                if (document.pointerLockElement !== renderer.domElement) {
                    requestPointerLockSafe();
                    return;
                }
                if (e.button === 0) {
                    state.isMouseDown = true;
                    shootWeapon();
                } else if (e.button === 2) {
                    state.isAiming = true;
                }
            });

            document.addEventListener('mouseup', (e) => {
                if (e.button === 0) state.isMouseDown = false;
                if (e.button === 2) state.isAiming = false;
            });

            document.addEventListener('contextmenu', (e) => e.preventDefault());

            // Pointer Lock Durum Takibi
            document.addEventListener('pointerlockchange', () => {
                const locked = document.pointerLockElement === renderer.domElement;
                const lockOverlay = document.getElementById('lock-overlay');
                if (state.isPlaying && !state.isPaused && !state.isDead && !state.isMarketOpen) {
                    if (locked) {
                        lockOverlay.classList.add('hidden');
                    } else {
                        state.isMouseDown = false;
                        state.isAiming = false;
                        lockOverlay.classList.remove('hidden');
                    }
                }
            });

            document.getElementById('lock-overlay').addEventListener('click', () => {
                sfx.init();
                requestPointerLockSafe();
            });

            // Klavye Kontrolleri
            document.addEventListener('keydown', (e) => {
                const code = e.code;
                if (code === 'KeyW') keys.w = true;
                if (code === 'KeyA') keys.a = true;
                if (code === 'KeyS') keys.s = true;
                if (code === 'KeyD') keys.d = true;
                if (code === 'Space') keys.space = true;
                if (code === 'ShiftLeft' || code === 'ShiftRight') keys.shift = true;

                // F Tuşu ile Silah Marketi Aç/Kapat
                if (code === 'KeyF' && state.isPlaying && !state.isDead && !state.isPaused) {
                    toggleMarket(!state.isMarketOpen);
                }

                // T Tuşu ile El Feneri
                if (code === 'KeyT' && state.isPlaying && !state.isDead && !state.isPaused) {
                    sfx.playClick();
                    state.flashlightOn = !state.flashlightOn;
                    if (flashlight) flashlight.visible = state.flashlightOn;
                    updateHUD();
                }

                // G Tuşu ile El Bombası
                if (code === 'KeyG' && state.isPlaying && !state.isDead && !state.isPaused) {
                    throwGrenade();
                }

                // P veya ESC ile Duraklatma Menüsü
                if ((code === 'KeyP' || code === 'Escape') && state.isPlaying && !state.isDead) {
                    if (state.isMarketOpen) {
                        toggleMarket(false);
                    } else if (!state.isPaused) {
                        state.isPaused = true;
                        state.isMouseDown = false;
                        if (document.pointerLockElement) document.exitPointerLock();
                        document.getElementById('lock-overlay').classList.add('hidden');
                        document.getElementById('pause-menu').classList.remove('hidden');
                    }
                }
            });

            document.addEventListener('keyup', (e) => {
                const code = e.code;
                if (code === 'KeyW') keys.w = false;
                if (code === 'KeyA') keys.a = false;
                if (code === 'KeyS') keys.s = false;
                if (code === 'KeyD') keys.d = false;
                if (code === 'Space') keys.space = false;
                if (code === 'ShiftLeft' || code === 'ShiftRight') keys.shift = false;
            });

            // Lobi Butonları
            document.getElementById('btn-start-game').addEventListener('click', startGame);
            document.getElementById('btn-open-market').addEventListener('click', () => {
                sfx.init();
                toggleMarket(true);
            });
            document.getElementById('btn-open-settings').addEventListener('click', () => {
                sfx.init();
                openSettings('main');
            });

            // Duraklatma Butonları
            document.getElementById('btn-resume-game').addEventListener('click', () => {
                sfx.playClick();
                state.isPaused = false;
                document.getElementById('pause-menu').classList.add('hidden');
                requestPointerLockSafe();
            });
            document.getElementById('btn-pause-market').addEventListener('click', () => {
                document.getElementById('pause-menu').classList.add('hidden');
                state.isPaused = false;
                toggleMarket(true);
            });
            document.getElementById('btn-pause-settings').addEventListener('click', () => openSettings('pause'));
            document.getElementById('btn-pause-lobby').addEventListener('click', returnToLobby);

            // Market Kapatma
            document.getElementById('btn-close-market').addEventListener('click', () => toggleMarket(false));

            // Ölüm Ekranı Butonları ("ÖLDÜNÜZ" -> Yeniden Başla, Lobiye Dön, Ayarlar)
            document.getElementById('btn-restart-game').addEventListener('click', startGame);
            document.getElementById('btn-death-lobby').addEventListener('click', returnToLobby);
            document.getElementById('btn-death-settings').addEventListener('click', () => openSettings('death'));

            // Ayarlar Kontrolleri
            document.getElementById('input-sens').addEventListener('input', (e) => {
                state.sensitivity = parseFloat(e.target.value);
                document.getElementById('sens-val').textContent = `${state.sensitivity.toFixed(1)}x`;
            });
            document.getElementById('input-vol').addEventListener('input', (e) => {
                const v = parseInt(e.target.value, 10);
                sfx.setVolume(v / 100);
                document.getElementById('vol-val').textContent = `${v}%`;
            });
            document.getElementById('btn-gfx-high').addEventListener('click', () => {
                sfx.playClick();
                state.highGraphics = true;
                renderer.shadowMap.enabled = true;
                if (fireflies) fireflies.visible = true;
                document.getElementById('btn-gfx-high').className = 'py-2.5 rounded-xl bg-red-600 text-white font-bold text-xs border border-red-400 cursor-pointer';
                document.getElementById('btn-gfx-low').className = 'py-2.5 rounded-xl bg-gray-800 text-gray-300 font-bold text-xs border border-white/10 cursor-pointer';
            });
            document.getElementById('btn-gfx-low').addEventListener('click', () => {
                sfx.playClick();
                state.highGraphics = false;
                renderer.shadowMap.enabled = false;
                if (fireflies) fireflies.visible = false;
                document.getElementById('btn-gfx-low').className = 'py-2.5 rounded-xl bg-red-600 text-white font-bold text-xs border border-red-400 cursor-pointer';
                document.getElementById('btn-gfx-high').className = 'py-2.5 rounded-xl bg-gray-800 text-gray-300 font-bold text-xs border border-white/10 cursor-pointer';
            });
            document.getElementById('btn-close-settings').addEventListener('click', closeSettings);
        }

        // Pencere yüklendiğinde 3D sahneyi ve animasyon döngüsünü başlat
        window.onload = function () {
            init3D();
            setupEventListeners();
            updateHUD();
            animateLoop();
        };
    </script>
</body>
</html>
