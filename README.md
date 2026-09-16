<!DOCTYPE html>
<html lang="id">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no, viewport-fit=cover">
    <title>ABSENSI KELAS</title>
    
    <!-- Meta Tags for PWA -->
    <meta name="theme-color" content="#1e40af">
    <meta name="apple-mobile-web-app-capable" content="yes">
    <meta name="apple-mobile-web-app-status-bar-style" content="black-translucent">
    
    <!-- Libraries -->
    <script src="https://cdn.tailwindcss.com"></script>
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.4.0/css/all.min.css">
    <script src="https://unpkg.com/dexie@3.2.4/dist/dexie.js"></script>
    <script src="https://unpkg.com/html5-qrcode"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/qrcodejs/1.0.0/qrcode.min.js"></script>
    <script src="https://cdn.jsdelivr.net/npm/sweetalert2@11"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/xlsx/0.18.5/xlsx.full.min.js"></script>

    <style>
        @import url('https://fonts.googleapis.com/css2?family=Inter:wght@300;400;500;600;700;800&display=swap');
        
        :root {
            --primary: #1e40af; 
            --bg-color: #f1f5f9; 
        }

        body {
            font-family: 'Inter', sans-serif;
            background-color: var(--bg-color);
            -webkit-tap-highlight-color: transparent;
            margin: 0; padding: 0;
            overflow: hidden; /* Prevent body scroll, handle inside app-container */
        }

        /* Responsive Container: Max width for tablet feel, full width for mobile */
        #app-container {
            width: 100%;
            max-width: 800px; 
            margin: 0 auto;
            background-color: #ffffff;
            height: 100vh;
            height: 100dvh;
            position: relative;
            display: flex;
            flex-direction: column;
            box-shadow: 0 0 20px rgba(0,0,0,0.1);
        }

        .screen {
            display: none;
            flex-direction: column;
            height: 100%;
            width: 100%;
            background-color: #f8fafc;
            position: absolute;
            top: 0; left: 0;
            z-index: 10;
        }
        .screen.active {
            display: flex;
            animation: fadeIn 0.3s ease-out;
        }

        @keyframes fadeIn {
            from { opacity: 0; transform: translateY(10px); }
            to { opacity: 1; transform: translateY(0); }
        }

        .scrollable { overflow-y: auto; overflow-x: hidden; }
        .scrollable::-webkit-scrollbar { width: 6px; }
        .scrollable::-webkit-scrollbar-thumb { background-color: #cbd5e1; border-radius: 6px; }

        .brand-gradient { background: linear-gradient(135deg, #1e3a8a 0%, #3b82f6 100%); }
        
        /* Nav & Scanner Styles */
        .nav-item { transition: color 0.2s, transform 0.2s; }
        .nav-item.active { color: var(--primary); font-weight: 700; }
        .nav-item.active i { transform: scale(1.15); }

        #reader { width: 100%; max-width: 500px; margin: 0 auto; background: #000; overflow: hidden; position: relative; border-radius: 12px; }
        #reader video { object-fit: cover !important; width: 100% !important; border-radius: 12px; }
        
        .swal2-container { z-index: 99999 !important; }

        /* Loader */
        .loader-spinner { border: 3px solid #f3f3f3; border-top: 3px solid #3b82f6; border-radius: 50%; width: 20px; height: 20px; animation: spin 1s linear infinite; display: inline-block; }
        @keyframes spin { 0% { transform: rotate(0deg); } 100% { transform: rotate(360deg); } }
    </style>
</head>
<body>

<div id="app-container">

    <!-- ==============================================
         1. WELCOME SCREEN
    =================================================== -->
    <div id="screen-welcome" class="screen active brand-gradient justify-center items-center text-white relative">
        <div class="text-center px-6 animate-[fadeIn_0.5s_ease-out]">
            <div class="w-24 h-24 bg-white rounded-3xl flex items-center justify-center mx-auto mb-6 shadow-2xl shadow-blue-900/50">
                <i class="fa-solid fa-qrcode text-5xl text-blue-800"></i>
            </div>
            <h1 class="text-4xl font-black tracking-tight mb-2">ABSENSI KELAS</h1>
            <p class="text-blue-100 font-medium text-sm mb-8 max-w-xs mx-auto">Kelola kehadiran guru dan peserta didik dengan mudah dan cepat melalui QR Code.</p>
            
            <button onclick="showScreen('screen-login')" class="bg-white text-blue-800 font-black px-10 py-4 rounded-xl shadow-lg hover:bg-gray-50 active:scale-95 transition-all w-full max-w-xs">
                MULAI SEKARANG <i class="fa-solid fa-arrow-right ml-2"></i>
            </button>
        </div>
        <div class="absolute bottom-8 w-full text-center opacity-80">
            <p class="text-[10px] uppercase tracking-widest font-bold">Diciptakan oleh</p>
            <p class="text-sm font-semibold mt-1">Dhona Chindy Ferdiana, M.Pd, Gr</p>
        </div>
    </div>


    <!-- ==============================================
         2. LOGIN SCREEN
    =================================================== -->
    <div id="screen-login" class="screen bg-white">
        <div class="flex-1 flex flex-col justify-center items-center p-6 md:p-12 relative scrollable">
            <div class="text-center mb-8 w-full max-w-sm">
                <div class="inline-block p-4 rounded-3xl bg-blue-50 text-blue-800 mb-4 shadow-sm border border-blue-100">
                    <i class="fa-solid fa-users-gear text-4xl"></i>
                </div>
                <h1 class="text-3xl font-black text-gray-800 tracking-tight">LOGIN SISTEM</h1>
                <p class="text-gray-500 text-xs mt-2 font-medium">Gunakan akun yang telah diberikan Admin.</p>
            </div>
            
            <form id="form-login" onsubmit="event.preventDefault(); handleLogin();" class="space-y-4 w-full max-w-sm">
                <div>
                    <label class="block text-xs font-black text-gray-700 mb-1 uppercase tracking-wide">Username</label>
                    <div class="relative">
                        <i class="fa-solid fa-user absolute left-4 top-3.5 text-gray-400"></i>
                        <input type="text" id="login-username" class="w-full pl-11 pr-4 py-3 bg-gray-50 rounded-xl border border-gray-200 text-sm font-bold focus:bg-white focus:border-blue-500 outline-none transition" placeholder="Masukkan Username" required autocapitalize="none">
                    </div>
                </div>
                <div>
                    <label class="block text-xs font-black text-gray-700 mb-1 uppercase tracking-wide">Password</label>
                    <div class="relative">
                        <i class="fa-solid fa-lock absolute left-4 top-3.5 text-gray-400"></i>
                        <input type="password" id="login-password" class="w-full pl-11 pr-12 py-3 bg-gray-50 rounded-xl border border-gray-200 text-sm font-bold tracking-wider focus:bg-white focus:border-blue-500 outline-none transition" placeholder="••••••••" required>
                        <button type="button" onclick="togglePassword('login-password', 'login-eye')" class="absolute right-4 top-3.5 text-gray-400 hover:text-gray-700"><i class="fa-solid fa-eye" id="login-eye"></i></button>
                    </div>
                </div>
                <button type="submit" class="w-full bg-blue-800 hover:bg-blue-900 active:scale-[0.98] text-white font-black tracking-wide py-3.5 px-4 rounded-xl shadow-lg transition flex justify-center items-center mt-6">
                    MASUK <i class="fa-solid fa-arrow-right-to-bracket ml-2"></i>
                </button>
            </form>
            
            <div class="mt-8 text-center text-[10px] text-gray-400 font-bold uppercase tracking-widest max-w-sm p-4 bg-gray-50 rounded-xl border border-gray-100">
                <i class="fa-solid fa-circle-info text-blue-500 mb-1 text-lg"></i><br>
                Akun Guru hanya dapat dibuat oleh Administrator melalui sistem pusat.
            </div>
        </div>
    </div>


    <!-- ==============================================
         3. ADMIN DASHBOARD
    =================================================== -->
    <div id="screen-admin" class="screen">
        <header class="brand-gradient text-white pt-8 pb-4 px-5 md:px-8 shadow-md flex justify-between items-center shrink-0 z-20">
            <div>
                <p class="text-[10px] uppercase tracking-wider font-bold text-blue-200 mb-0.5">Administrator</p>
                <h1 class="text-xl md:text-2xl font-black leading-tight">Manajemen Guru</h1>
            </div>
            <button onclick="handleLogout()" class="w-10 h-10 rounded-full bg-white/10 hover:bg-white/20 flex items-center justify-center border border-white/20 transition">
                <i class="fa-solid fa-power-off text-white"></i>
            </button>
        </header>

        <main class="flex-1 scrollable bg-gray-50 p-4 md:p-6 space-y-5">
            <!-- Stats -->
            <div class="grid grid-cols-2 gap-4">
                <div class="bg-white p-4 rounded-2xl shadow-sm border border-gray-200 flex items-center gap-3">
                    <div class="w-10 h-10 rounded-full bg-blue-100 text-blue-600 flex items-center justify-center text-lg"><i class="fa-solid fa-chalkboard-user"></i></div>
                    <div><p class="text-[10px] text-gray-500 font-black uppercase">Total Guru</p><h3 class="text-2xl font-black text-gray-800" id="adm-stat-total">0</h3></div>
                </div>
                <div class="bg-white p-4 rounded-2xl shadow-sm border border-gray-200 flex items-center gap-3">
                    <div class="w-10 h-10 rounded-full bg-green-100 text-green-600 flex items-center justify-center text-lg"><i class="fa-solid fa-user-check"></i></div>
                    <div><p class="text-[10px] text-gray-500 font-black uppercase">Guru Aktif</p><h3 class="text-2xl font-black text-gray-800" id="adm-stat-aktif">0</h3></div>
                </div>
            </div>

            <!-- Actions -->
            <div class="bg-white rounded-3xl shadow-sm border border-gray-200 overflow-hidden">
                <div class="p-4 border-b border-gray-100 bg-gray-50 flex justify-between items-center">
                    <h3 class="font-black text-gray-800 text-sm uppercase">Import Excel Guru</h3>
                </div>
                <div class="p-4 flex flex-col sm:flex-row gap-3">
                    <button onclick="downloadTemplateGuru()" class="flex-1 bg-white text-gray-700 text-xs font-black py-3 rounded-xl border border-gray-300 shadow-sm flex items-center justify-center hover:bg-gray-50">
                        <i class="fa-solid fa-download mr-2 text-blue-600 text-lg"></i> TEMPLATE
                    </button>
                    <button onclick="document.getElementById('file-import-guru').click()" class="flex-1 bg-green-600 hover:bg-green-700 text-white text-xs font-black py-3 rounded-xl shadow-md transition flex items-center justify-center">
                        <i class="fa-solid fa-file-excel mr-2 text-lg"></i> UPLOAD EXCEL
                    </button>
                    <input type="file" id="file-import-guru" accept=".xlsx, .xls" class="hidden" onchange="handleImportGuru(event)">
                </div>
            </div>

            <!-- List Guru -->
            <div class="bg-white rounded-3xl shadow-sm border border-gray-200 flex flex-col min-h-[300px]">
                <div class="p-4 border-b border-gray-100 bg-gray-50 flex justify-between items-center gap-3">
                    <h3 class="font-black text-gray-800 text-sm uppercase shrink-0">Daftar Akun Guru</h3>
                    <button onclick="openModal('modal-add-guru')" class="bg-blue-800 text-white text-[10px] font-bold px-3 py-1.5 rounded-lg shadow-sm"><i class="fa-solid fa-plus mr-1"></i> Manual</button>
                </div>
                <div class="p-3 border-b border-gray-100">
                    <div class="relative w-full">
                        <i class="fa-solid fa-search absolute left-3 top-2.5 text-gray-400 text-sm"></i>
                        <input type="text" id="adm-search-guru" onkeyup="filterAdminGuru()" placeholder="Cari nama guru / username..." class="w-full pl-9 pr-3 py-2 rounded-lg border border-gray-200 text-xs focus:border-blue-500 outline-none">
                    </div>
                </div>
                <div class="p-0 overflow-y-auto max-h-[50vh]">
                    <ul id="adm-list-guru" class="divide-y divide-gray-100"></ul>
                </div>
            </div>
        </main>
    </div>


    <!-- ==============================================
         4. GURU DASHBOARD & MAIN NAVIGATION
    =================================================== -->
    <div id="screen-guru" class="screen">
        <header class="bg-white pt-8 pb-3 px-5 md:px-6 shadow-sm flex justify-between items-center shrink-0 z-20 border-b border-gray-200 relative">
            <div class="flex items-center space-x-3">
                <div class="w-10 h-10 rounded-full brand-gradient text-white flex items-center justify-center font-black shadow-md text-lg">
                    <i class="fa-solid fa-chalkboard-user"></i>
                </div>
                <div>
                    <h1 class="text-sm font-black text-gray-800 leading-tight truncate max-w-[200px]" id="guru-header-name">Nama Guru</h1>
                    <p class="text-[10px] text-gray-500 font-bold uppercase tracking-wider">Dashboard Guru</p>
                </div>
            </div>
            <button onclick="handleLogout()" class="text-red-500 bg-red-50 w-9 h-9 rounded-full flex items-center justify-center border border-red-100 transition" title="Logout">
                <i class="fa-solid fa-right-from-bracket"></i>
            </button>
        </header>

        <main class="flex-1 scrollable bg-gray-50 pb-[80px] md:pb-[90px] relative overflow-x-hidden">
            
            <!-- TAB: BERANDA -->
            <div id="guru-view-beranda" class="guru-active-view p-4 md:p-6 space-y-5">
                <div class="bg-gradient-to-br from-blue-800 to-blue-600 rounded-3xl p-6 shadow-lg shadow-blue-200 text-white relative overflow-hidden">
                    <div class="absolute -right-4 -bottom-4 opacity-10"><i class="fa-solid fa-school text-9xl"></i></div>
                    <div class="relative z-10">
                        <p class="text-xs text-blue-200 font-bold mb-1 tracking-widest uppercase" id="guru-date-today">Tanggal</p>
                        <h2 class="text-2xl font-black mb-1">ABSENSI KELAS</h2>
                        <p class="text-xs text-blue-100 mb-5">Sistem Manajemen Kehadiran Digital</p>
                        
                        <div class="grid grid-cols-2 gap-3 mt-4">
                            <div class="bg-white/10 p-3 rounded-2xl backdrop-blur-sm border border-white/20">
                                <p class="text-[10px] font-bold text-blue-200 uppercase">Total Kelas</p>
                                <h3 class="text-2xl font-black" id="guru-stat-kelas">0</h3>
                            </div>
                            <div class="bg-white/10 p-3 rounded-2xl backdrop-blur-sm border border-white/20">
                                <p class="text-[10px] font-bold text-blue-200 uppercase">Total Siswa</p>
                                <h3 class="text-2xl font-black" id="guru-stat-siswa">0</h3>
                            </div>
                        </div>
                    </div>
                </div>

                <div class="bg-white rounded-3xl p-5 shadow-sm border border-gray-200">
                    <h3 class="font-black text-gray-800 text-sm uppercase mb-3 flex items-center"><i class="fa-solid fa-bolt text-yellow-500 mr-2"></i> Jalan Pintas</h3>
                    <div class="grid grid-cols-2 gap-3">
                        <button onclick="guruSwitchView('siswa')" class="bg-blue-50 text-blue-700 p-3 rounded-xl font-bold text-xs border border-blue-100 flex flex-col items-center justify-center h-20 shadow-sm"><i class="fa-solid fa-users text-xl mb-1"></i> Data Siswa</button>
                        <button onclick="guruSwitchView('rekap')" class="bg-green-50 text-green-700 p-3 rounded-xl font-bold text-xs border border-green-100 flex flex-col items-center justify-center h-20 shadow-sm"><i class="fa-solid fa-file-excel text-xl mb-1"></i> Rekap / Excel</button>
                    </div>
                </div>
            </div>

            <!-- TAB: SISWA (Pusat Manajemen Kelas & Siswa via Excel) -->
            <div id="guru-view-siswa" class="guru-active-view hidden p-4 md:p-6 space-y-4 flex-col min-h-full">
                <div>
                    <h2 class="text-xl font-black text-gray-800 tracking-tight">Manajemen Siswa & Kelas</h2>
                    <p class="text-[10px] text-gray-500 font-bold uppercase mt-1">Upload Excel otomatis membuat Kelas & QR</p>
                </div>

                <!-- Panel Upload Excel -->
                <div class="bg-white rounded-2xl p-4 shadow-sm border border-blue-200 bg-blue-50/50">
                    <div class="flex flex-col sm:flex-row gap-2">
                        <button onclick="downloadTemplateSiswa()" class="flex-1 text-[10px] bg-white border border-gray-300 text-gray-700 font-bold px-3 py-2.5 rounded-xl shadow-sm flex items-center justify-center"><i class="fa-solid fa-download mr-1.5 text-blue-600 text-sm"></i> 1. Template Excel</button>
                        <button onclick="document.getElementById('file-import-siswa').click()" class="flex-1 text-[10px] bg-green-600 text-white font-bold px-3 py-2.5 rounded-xl shadow-sm flex items-center justify-center"><i class="fa-solid fa-file-excel mr-1.5 text-sm"></i> 2. Upload Excel Siswa</button>
                        <input type="file" id="file-import-siswa" accept=".xlsx, .xls" class="hidden" onchange="handleImportSiswa(event)">
                    </div>
                </div>

                <!-- Daftar Kelas & Siswa -->
                <div class="bg-white rounded-3xl shadow-sm border border-gray-200 flex flex-col flex-1">
                    <div class="p-4 border-b border-gray-100 bg-gray-50 flex flex-col sm:flex-row justify-between items-start sm:items-center gap-3 shrink-0 rounded-t-3xl">
                        <select id="filter-siswa-kelas" class="w-full sm:w-auto p-2 bg-white border border-gray-300 rounded-lg text-xs font-bold outline-none shadow-sm" onchange="loadGuruSiswa()">
                            <option value="">Semua Kelas</option>
                        </select>
                        <div class="flex gap-2 w-full sm:w-auto justify-end">
                            <button onclick="printQRMassal()" class="text-[10px] bg-gray-800 text-white font-bold px-3 py-2 rounded-lg shadow-sm flex items-center"><i class="fa-solid fa-print mr-1"></i> Cetak QR Kelas</button>
                            <button onclick="openModalAddSiswa()" class="text-[10px] bg-blue-800 text-white font-bold px-3 py-2 rounded-lg shadow-sm flex items-center"><i class="fa-solid fa-plus mr-1"></i> Manual</button>
                        </div>
                    </div>
                    <div class="p-3 border-b border-gray-100 shrink-0">
                        <div class="relative w-full">
                            <i class="fa-solid fa-search absolute left-3 top-2.5 text-gray-400 text-sm"></i>
                            <input type="text" id="search-siswa" onkeyup="filterGuruStudents()" placeholder="Cari nama atau NIS siswa..." class="w-full pl-9 pr-3 py-2 rounded-lg border border-gray-200 text-xs focus:border-blue-500 outline-none bg-gray-50">
                        </div>
                    </div>
                    
                    <ul id="guru-list-siswa" class="overflow-y-auto flex-1 divide-y divide-gray-100 min-h-[300px]">
                        <div class="text-center text-gray-400 text-xs font-bold py-16">Memuat data...</div>
                    </ul>
                </div>
            </div>


            <!-- TAB: SCANNER (Inti Aplikasi) -->
            <div id="guru-view-scan" class="guru-active-view hidden p-4 md:p-6 flex-col min-h-full">
                
                <!-- State 1: Setup Sesi (Pilih Kelas dll) -->
                <div id="scan-setup-panel" class="bg-white rounded-3xl shadow-sm border border-gray-200 p-5 md:p-8 space-y-5 animate-[fadeIn_0.3s_ease]">
                    <div class="text-center mb-6">
                        <div class="w-16 h-16 bg-blue-50 text-blue-600 rounded-full flex items-center justify-center text-3xl mx-auto mb-3"><i class="fa-solid fa-camera"></i></div>
                        <h2 class="text-xl font-black text-gray-800">Mulai Sesi Absensi</h2>
                        <p class="text-xs text-gray-500 font-medium mt-1">Pilih kelas yang akan diabsen hari ini.</p>
                    </div>

                    <div>
                        <label class="block text-xs font-black text-gray-700 mb-2 uppercase tracking-wide">Pilih Kelas</label>
                        <select id="scan-sel-kelas" class="w-full p-4 bg-gray-50 border border-gray-200 rounded-2xl text-sm font-bold outline-none focus:border-blue-500 shadow-inner">
                            <option value="">-- Memuat Kelas --</option>
                        </select>
                    </div>
                    <div>
                        <label class="block text-xs font-black text-gray-700 mb-2 uppercase tracking-wide">Batas Hadir (Jam Terlambat)</label>
                        <input type="time" id="scan-time-late" class="w-full p-4 bg-gray-50 border border-gray-200 rounded-2xl text-sm font-bold outline-none focus:border-blue-500 shadow-inner">
                    </div>
                    
                    <button onclick="startScannerSession()" class="w-full bg-blue-800 text-white font-black py-4 rounded-2xl shadow-lg hover:bg-blue-900 active:scale-95 transition flex justify-center items-center mt-4">
                        BUKA KAMERA SCANNER <i class="fa-solid fa-qrcode ml-2 text-lg"></i>
                    </button>
                </div>

                <!-- State 2: Live Scanner & Daftar (Hidden secara default) -->
                <div id="scan-active-panel" class="hidden flex-col h-[calc(100vh-140px)] animate-[fadeIn_0.3s_ease]">
                    
                    <!-- Header Info -->
                    <div class="bg-gray-900 text-white p-4 rounded-t-2xl flex justify-between items-center shrink-0 shadow-lg">
                        <div>
                            <div class="flex items-center text-red-500 text-[10px] font-black uppercase tracking-widest mb-1"><div class="w-2 h-2 rounded-full bg-red-500 animate-pulse mr-1.5 shadow-[0_0_8px_rgba(239,68,68,1)]"></div> Live Record</div>
                            <h3 id="live-class-name" class="font-black text-lg leading-tight">Kelas</h3>
                            <p id="live-time-info" class="text-[10px] text-gray-400 font-medium">Batas: --:--</p>
                        </div>
                        <button onclick="confirmCloseSession()" class="bg-red-600 text-white px-4 py-2.5 rounded-xl font-black text-xs shadow-md">TUTUP SESI</button>
                    </div>

                    <!-- Camera Area -->
                    <div class="bg-black relative flex flex-col justify-center items-center shrink-0 overflow-hidden" style="min-height: 250px; max-height: 40vh;">
                        <div id="reader" class="w-full h-full object-cover"></div>
                        
                        <!-- Overlay -->
                        <div class="absolute inset-0 pointer-events-none flex flex-col items-center justify-center z-10">
                            <div class="w-48 h-48 md:w-64 md:h-64 relative">
                                <div class="absolute w-8 h-8 border-t-4 border-l-4 border-white top-0 left-0 rounded-tl-lg"></div>
                                <div class="absolute w-8 h-8 border-t-4 border-r-4 border-white top-0 right-0 rounded-tr-lg"></div>
                                <div class="absolute w-8 h-8 border-b-4 border-l-4 border-white bottom-0 left-0 rounded-bl-lg"></div>
                                <div class="absolute w-8 h-8 border-b-4 border-r-4 border-white bottom-0 right-0 rounded-br-lg"></div>
                                <div class="w-full h-0.5 bg-red-500/80 absolute shadow-[0_0_10px_rgba(239,68,68,1)] top-1/2 transform -translate-y-1/2 animate-[scanline_2s_linear_infinite]"></div>
                            </div>
                        </div>

                        <!-- Camera Controls -->
                        <div class="absolute bottom-3 right-3 z-20 flex gap-2">
                            <button onclick="switchCamera()" class="w-10 h-10 bg-black/60 backdrop-blur text-white rounded-full flex items-center justify-center border border-white/20 shadow-lg text-sm" title="Ganti Kamera">
                                <i class="fa-solid fa-camera-rotate"></i>
                            </button>
                        </div>
                    </div>

                    <!-- Stats & Live List -->
                    <div class="bg-white flex-1 rounded-b-2xl shadow-lg flex flex-col overflow-hidden border-x border-b border-gray-200">
                        
                        <!-- Grid Stats -->
                        <div class="grid grid-cols-4 gap-1 p-3 shrink-0 bg-gray-50 border-b border-gray-200">
                            <div class="bg-white p-2 rounded-lg text-center border border-gray-200 shadow-sm"><div id="ls-total" class="font-black text-sm text-gray-800">0</div><div class="text-[8px] font-bold text-gray-500 uppercase">Total</div></div>
                            <div class="bg-green-50 p-2 rounded-lg text-center border border-green-200 shadow-sm"><div id="ls-hadir" class="font-black text-sm text-green-700">0</div><div class="text-[8px] font-bold text-green-700 uppercase">Hadir</div></div>
                            <div class="bg-orange-50 p-2 rounded-lg text-center border border-orange-200 shadow-sm"><div id="ls-telat" class="font-black text-sm text-orange-700">0</div><div class="text-[8px] font-bold text-orange-700 uppercase">Telat</div></div>
                            <div class="bg-red-50 p-2 rounded-lg text-center border border-red-200 shadow-sm"><div id="ls-belum" class="font-black text-sm text-red-700">0</div><div class="text-[8px] font-bold text-red-700 uppercase">Belum</div></div>
                        </div>

                        <div class="flex justify-between items-center px-4 py-2 shrink-0 bg-white border-b border-gray-100">
                            <h4 class="font-black text-[11px] text-gray-800 uppercase tracking-wide">Daftar Absensi Hari Ini</h4>
                            <button onclick="openManualAbsen()" class="text-[10px] font-bold bg-blue-100 text-blue-800 px-3 py-1.5 rounded-lg shadow-sm border border-blue-200"><i class="fa-solid fa-pen-to-square mr-1"></i> Absen Manual</button>
                        </div>

                        <!-- Scrollable List -->
                        <div class="flex-1 overflow-y-auto bg-white p-2">
                            <ul id="live-scan-list" class="space-y-1.5">
                                <!-- Terisi dinamis -->
                            </ul>
                        </div>
                    </div>
                </div>

                <!-- Toast Overlay untuk Scanner -->
                <div id="scan-toast" class="absolute top-[30%] left-1/2 transform -translate-x-1/2 -translate-y-1/2 px-6 py-4 rounded-3xl shadow-[0_10px_40px_rgba(0,0,0,0.4)] font-black flex flex-col items-center text-center transition-all duration-200 opacity-0 scale-90 pointer-events-none z-[100] min-w-[280px] border-4 bg-white">
                    <i id="scan-toast-icon" class="fa-solid fa-check-circle text-5xl mb-2"></i>
                    <span id="scan-toast-title" class="text-sm uppercase tracking-widest text-gray-500 mb-1">Status</span>
                    <span id="scan-toast-desc" class="text-xl font-black leading-tight text-gray-800">Nama Siswa</span>
                </div>
            </div>

            <!-- TAB: REKAP ABSENSI -->
            <div id="guru-view-rekap" class="guru-active-view hidden p-4 md:p-6 space-y-5">
                <div>
                    <h2 class="text-xl font-black text-gray-800 tracking-tight">Rekap & Export Excel</h2>
                    <p class="text-[10px] text-gray-500 font-bold uppercase mt-1">Unduh laporan absensi harian/bulanan</p>
                </div>

                <div class="bg-white rounded-3xl p-5 md:p-6 shadow-sm border border-gray-200">
                    <div class="space-y-4">
                        <div>
                            <label class="block text-xs font-black text-gray-700 mb-1.5 uppercase">Pilih Kelas</label>
                            <select id="export-kelas" class="w-full p-3.5 bg-gray-50 border border-gray-200 rounded-xl text-sm font-bold outline-none focus:border-green-500">
                                <option value="">Semua Kelas</option>
                            </select>
                        </div>
                        <div>
                            <label class="block text-xs font-black text-gray-700 mb-1.5 uppercase">Pilih Tanggal Sesi</label>
                            <input type="date" id="export-tanggal" class="w-full p-3.5 bg-gray-50 border border-gray-200 rounded-xl text-sm font-bold outline-none focus:border-green-500">
                            <p class="text-[9px] text-gray-400 font-bold mt-1">*Kosongkan tanggal untuk export seluruh data kelas tersebut.</p>
                        </div>
                        
                        <button onclick="exportExcelRekap()" class="w-full bg-green-600 text-white font-black tracking-wide py-4 rounded-xl shadow-lg hover:bg-green-700 active:scale-95 transition flex justify-center items-center mt-2">
                            <i class="fa-solid fa-file-excel mr-2 text-lg"></i> DOWNLOAD LAPORAN EXCEL
                        </button>
                    </div>
                </div>

                <!-- Riwayat Sesi Lokal Terakhir -->
                <div class="bg-white rounded-3xl shadow-sm border border-gray-200 overflow-hidden mt-4">
                    <div class="p-4 bg-gray-50 border-b border-gray-100 flex items-center">
                        <i class="fa-solid fa-clock-rotate-left text-gray-400 mr-2"></i>
                        <h3 class="text-[11px] font-black text-gray-800 uppercase tracking-widest">Riwayat Sesi Terakhir</h3>
                    </div>
                    <div id="guru-list-sesi" class="divide-y divide-gray-100 max-h-[40vh] overflow-y-auto"></div>
                </div>
            </div>

        </main>

        <!-- Bottom Navigation for Guru (Fixed for mobile, styling adapted for tablet) -->
        <nav class="bg-white border-t border-gray-200 flex justify-around items-center h-[70px] shrink-0 pb-safe shadow-[0_-5px_20px_rgba(0,0,0,0.05)] z-30 absolute bottom-0 w-full px-2">
            <button onclick="guruSwitchView('beranda')" class="nav-item active flex flex-col items-center justify-center w-full h-full text-gray-400 hover:text-blue-600" data-target="beranda">
                <i class="fa-solid fa-house text-[22px] mb-1"></i><span class="text-[9px] uppercase tracking-wider font-bold">Beranda</span>
            </button>
            <button onclick="guruSwitchView('siswa')" class="nav-item flex flex-col items-center justify-center w-full h-full text-gray-400 hover:text-blue-600" data-target="siswa">
                <i class="fa-solid fa-users text-[22px] mb-1"></i><span class="text-[9px] uppercase tracking-wider font-bold">Data</span>
            </button>
            
            <!-- Scan Center FAB Button -->
            <div class="relative w-full h-full flex justify-center">
                <button onclick="guruSwitchView('scan')" class="scan-fab absolute -top-6 w-[60px] h-[60px] rounded-full text-white flex flex-col items-center justify-center border-[5px] border-white z-40 bg-gradient-to-r from-blue-700 to-blue-500 shadow-lg shadow-blue-300 transition-transform active:scale-95" title="Mulai Absensi">
                    <i class="fa-solid fa-camera text-2xl"></i>
                </button>
                <span class="absolute bottom-1.5 text-[9px] font-black text-blue-800 uppercase tracking-widest">Scan</span>
            </div>

            <button onclick="guruSwitchView('rekap')" class="nav-item flex flex-col items-center justify-center w-full h-full text-gray-400 hover:text-blue-600" data-target="rekap">
                <i class="fa-solid fa-file-excel text-[22px] mb-1"></i><span class="text-[9px] uppercase tracking-wider font-bold">Rekap</span>
            </button>
        </nav>
    </div>


    <!-- ==============================================
         MODALS (Tambah Data Manual dll)
    =================================================== -->
    
    <!-- Modal: Tambah Guru Manual (Admin) -->
    <div id="modal-add-guru" class="fixed inset-0 bg-black/60 z-[100] hidden flex-col justify-end md:justify-center md:items-center backdrop-blur-sm p-4">
        <div class="bg-white rounded-3xl w-full max-w-sm flex flex-col shadow-2xl transform transition-transform scale-95 opacity-0" id="modal-add-guru-content">
            <div class="p-5 border-b border-gray-100 flex justify-between items-center bg-gray-50 rounded-t-3xl">
                <h3 class="font-black text-gray-800 text-sm uppercase tracking-wide">Tambah Akun Guru</h3>
                <button onclick="closeModal('modal-add-guru')" class="w-8 h-8 bg-gray-200 text-gray-600 rounded-full hover:bg-gray-300"><i class="fa-solid fa-xmark"></i></button>
            </div>
            <div class="p-6">
                <form onsubmit="event.preventDefault(); saveGuruManual();" class="space-y-4">
                    <div><label class="block text-xs font-black text-gray-700 mb-1 uppercase">Nama Lengkap</label><input type="text" id="inp-guru-nama" class="w-full p-3 bg-gray-50 border border-gray-200 rounded-xl text-sm font-bold focus:border-blue-500 outline-none" required></div>
                    <div><label class="block text-xs font-black text-gray-700 mb-1 uppercase">Username Login</label><input type="text" id="inp-guru-uname" class="w-full p-3 bg-gray-50 border border-gray-200 rounded-xl text-sm font-bold focus:border-blue-500 outline-none" required autocapitalize="none"></div>
                    <div><label class="block text-xs font-black text-gray-700 mb-1 uppercase">Password</label><input type="password" id="inp-guru-pass" class="w-full p-3 bg-gray-50 border border-gray-200 rounded-xl text-sm font-bold tracking-wider focus:border-blue-500 outline-none" required></div>
                    <button type="submit" class="w-full bg-blue-800 text-white font-black py-3.5 rounded-xl mt-2 shadow-md hover:bg-blue-900 transition">SIMPAN GURU</button>
                </form>
            </div>
        </div>
    </div>

    <!-- Modal: Tambah Siswa Manual (Guru) -->
    <div id="modal-add-siswa" class="fixed inset-0 bg-black/60 z-[100] hidden flex-col justify-end md:justify-center md:items-center backdrop-blur-sm p-4">
        <div class="bg-white rounded-3xl w-full max-w-sm flex flex-col shadow-2xl transform transition-transform scale-95 opacity-0" id="modal-add-siswa-content">
            <div class="p-5 border-b border-gray-100 flex justify-between items-center bg-gray-50 rounded-t-3xl">
                <h3 class="font-black text-gray-800 text-sm uppercase tracking-wide">Tambah Siswa Manual</h3>
                <button onclick="closeModal('modal-add-siswa')" class="w-8 h-8 bg-gray-200 text-gray-600 rounded-full hover:bg-gray-300"><i class="fa-solid fa-xmark"></i></button>
            </div>
            <div class="p-6 overflow-y-auto max-h-[70vh]">
                <form onsubmit="event.preventDefault(); saveSiswaManual();" class="space-y-4">
                    <div>
                        <label class="block text-xs font-black text-gray-700 mb-1 uppercase">Pilih / Ketik Kelas</label>
                        <!-- Menggunakan datalist agar guru bisa milih kelas existing atau ketik baru -->
                        <input type="text" id="inp-sw-kelas" list="datalist-kelas" class="w-full p-3 bg-gray-50 border border-gray-200 rounded-xl text-sm font-bold focus:border-blue-500 outline-none uppercase" required placeholder="Contoh: VII A">
                        <datalist id="datalist-kelas"></datalist>
                    </div>
                    <div><label class="block text-xs font-black text-gray-700 mb-1 uppercase">NIS / NISN</label><input type="text" id="inp-sw-nis" class="w-full p-3 bg-gray-50 border border-gray-200 rounded-xl text-sm font-bold focus:border-blue-500 outline-none" required></div>
                    <div><label class="block text-xs font-black text-gray-700 mb-1 uppercase">Nama Lengkap</label><input type="text" id="inp-sw-nama" class="w-full p-3 bg-gray-50 border border-gray-200 rounded-xl text-sm font-bold focus:border-blue-500 outline-none capitalize" required></div>
                    <div>
                        <label class="block text-xs font-black text-gray-700 mb-1 uppercase">Jenis Kelamin</label>
                        <select id="inp-sw-jk" class="w-full p-3 bg-gray-50 border border-gray-200 rounded-xl text-sm font-bold focus:border-blue-500 outline-none">
                            <option value="L">Laki-Laki (L)</option><option value="P">Perempuan (P)</option>
                        </select>
                    </div>
                    <button type="submit" class="w-full bg-blue-800 text-white font-black py-3.5 rounded-xl mt-4 shadow-md hover:bg-blue-900 transition flex items-center justify-center">
                        SIMPAN & BUAT QR <i class="fa-solid fa-qrcode ml-2"></i>
                    </button>
                </form>
            </div>
        </div>
    </div>

    <!-- Area Tersembunyi untuk Generate Gambar QR Code (Dipakai untuk Print) -->
    <div id="qr-render-area" class="hidden"></div>
    <style> @keyframes scanline { 0% { top: 0%; } 50% { top: 100%; } 100% { top: 0%; } } </style>
</div> <!-- End App Container -->


<!-- ==============================================
     JAVASCRIPT LOGIC
=================================================== -->
<script>
    // 1. Inisialisasi Database (Dexie.js)
    const db = new Dexie("AbsensiKelas_PWA_DB");
    db.version(1).stores({
        users: 'id, role, username, password, name, status', // role: 'ADMIN' | 'GURU'
        classes: '++id, teacherId, name', 
        students: '++id, classId, teacherId, nis, name, gender, qrToken', // qrToken = ID internal QR
        sessions: '++id, classId, teacherId, date, timeStart, timeEnd, status', // status: 'OPEN' | 'CLOSED'
        attendance: '++id, sessionId, studentId, classId, teacherId, date, status, timestamp, method, [sessionId+studentId]' // Constraint untuk anti-duplikat
    });

    // Variabel Global
    let currentUser = null;
    let activeSession = null;
    let html5QrCode = null;
    let isProcessingScan = false;
    let currentCameraId = null;
    let availableCameras = [];
    let cameraIndex = 0;
    
    // Cache Lokal untuk UI Guru
    let guruCachedClasses = [];
    let guruCachedStudents = [];
    let sessionStudents = [];

    // Audio API untuk Beep
    const audioCtx = new (window.AudioContext || window.webkitAudioContext)();
    function playBeep(type) {
        try {
            if(audioCtx.state === 'suspended') audioCtx.resume();
            const osc = audioCtx.createOscillator(); const gain = audioCtx.createGain();
            osc.connect(gain); gain.connect(audioCtx.destination);
            if (type === 'success') { 
                osc.type = 'sine'; osc.frequency.setValueAtTime(1000, audioCtx.currentTime); 
                gain.gain.setValueAtTime(0.1, audioCtx.currentTime); osc.start(); osc.stop(audioCtx.currentTime + 0.1); 
            } else { 
                osc.type = 'sawtooth'; osc.frequency.setValueAtTime(200, audioCtx.currentTime); 
                osc.frequency.exponentialRampToValueAtTime(100, audioCtx.currentTime + 0.3); 
                gain.gain.setValueAtTime(0.2, audioCtx.currentTime); osc.start(); osc.stop(audioCtx.currentTime + 0.3); 
            }
        } catch(e){}
        if(navigator.vibrate) navigator.vibrate(type==='success'?[100]:[50,50,50]);
    }

    // Helper: Hashing Sederhana (Untuk demo PWA offline)
    async function hashString(str) {
        const buffer = await crypto.subtle.digest('SHA-256', new TextEncoder().encode(str));
        return Array.from(new Uint8Array(buffer)).map(b => b.toString(16).padStart(2, '0')).join('');
    }

    // UI Helpers
    function togglePassword(inputId, iconId) {
        const inp = document.getElementById(inputId); const ico = document.getElementById(iconId);
        if(inp.type === 'password') { inp.type = 'text'; ico.classList.replace('fa-eye', 'fa-eye-slash'); }
        else { inp.type = 'password'; ico.classList.replace('fa-eye-slash', 'fa-eye'); }
    }
    
    function showScreen(id) {
        document.querySelectorAll('.screen').forEach(el => el.classList.remove('active'));
        document.getElementById(id).classList.add('active');
    }
    
    function openModal(id) {
        const m = document.getElementById(id); const c = document.getElementById(`${id}-content`);
        m.classList.remove('hidden'); m.classList.add('flex');
        setTimeout(() => { c.classList.remove('scale-95', 'opacity-0'); c.classList.add('scale-100', 'opacity-100'); }, 10);
    }
    
    function closeModal(id) {
        const m = document.getElementById(id); const c = document.getElementById(`${id}-content`);
        c.classList.remove('scale-100', 'opacity-100'); c.classList.add('scale-95', 'opacity-0');
        setTimeout(() => { m.classList.add('hidden'); m.classList.remove('flex'); }, 200);
    }

    const generateToken = (nis) => `AK-STU-${Date.now().toString(36).toUpperCase()}-${Math.random().toString(36).substr(2,4).toUpperCase()}`;
    const formatTimeShort = (ts) => new Date(ts).toLocaleTimeString('id-ID', {hour:'2-digit', minute:'2-digit'});
    const formatDate = (ts) => new Date(ts).toLocaleDateString('id-ID', {day:'2-digit', month:'short', year:'numeric'});

    // 2. Boot & Autentikasi
    window.onload = async () => {
        history.pushState(null, document.title, location.href); 
        
        // Buat Admin Master jika DB kosong
        const adminCount = await db.users.where('role').equals('ADMIN').count();
        if (adminCount === 0) {
            const hashed = await hashString('123456'); // Default Admin Password
            await db.users.put({ id: 'admin_master', username: 'admin', password: hashed, name: 'Administrator', role: 'ADMIN', status: 'ACTIVE' });
        }
        
        // Cek Login Session lokal
        const uid = localStorage.getItem('ak_uid_pwa');
        if (uid) {
            const user = await db.users.get(uid);
            if (user && user.status === 'ACTIVE') {
                currentUser = user; 
                routeUser();
                return;
            }
        }
        // Jika tidak ada session, tetap di Welcome Screen (default aktif di HTML)
    };

    async function handleLogin() {
        const uname = document.getElementById('login-username').value.trim().toLowerCase();
        const pass = document.getElementById('login-password').value;
        if (!uname || !pass) return Swal.fire('Oops', 'Isi username dan password.', 'warning');
        
        Swal.fire({title: 'Memverifikasi...', allowOutsideClick: false, didOpen: () => Swal.showLoading()});
        const hashed = await hashString(pass);
        const users = await db.users.where('username').equals(uname).toArray();
        const user = users.find(u => u.password === hashed);
        
        Swal.close();
        if (user) {
            if(user.status !== 'ACTIVE') return Swal.fire('Ditolak', 'Akun dinonaktifkan Admin.', 'error');
            currentUser = user;
            localStorage.setItem('ak_uid_pwa', user.id);
            routeUser();
        } else { Swal.fire('Gagal', 'Username atau password salah!', 'error'); }
    }

    function handleLogout() {
        Swal.fire({title: 'Keluar dari Sistem?', icon: 'warning', showCancelButton: true, confirmButtonText: 'Ya, Logout'}).then(async (r) => {
            if(r.isConfirmed) {
                // Pastikan kamera mati jika di menu scan
                if(html5QrCode) { try { await html5QrCode.stop(); html5QrCode.clear(); } catch(e){} }
                localStorage.removeItem('ak_uid_pwa'); currentUser = null;
                document.getElementById('form-login').reset();
                showScreen('screen-login');
            }
        });
    }

    function routeUser() {
        if(currentUser.role === 'ADMIN') initAdminDash();
        else initGuruDash();
    }

    // 3. Modul ADMIN (Manajemen Akun Guru via Excel)
    async function initAdminDash() {
        showScreen('screen-admin');
        const gurus = await db.users.where('role').equals('GURU').toArray();
        document.getElementById('adm-stat-total').innerText = gurus.length;
        document.getElementById('adm-stat-aktif').innerText = gurus.filter(g=>g.status==='ACTIVE').length;
        renderAdminGuruList(gurus);
    }

    function renderAdminGuruList(list) {
        const ul = document.getElementById('adm-list-guru');
        if(list.length === 0) { ul.innerHTML = '<div class="p-10 text-center text-xs text-gray-400 font-bold">Belum ada akun guru. Gunakan Upload Excel.</div>'; return; }
        
        list.sort((a,b) => a.name.localeCompare(b.name));
        ul.innerHTML = list.map((g,i) => `
            <li class="p-4 border-b border-gray-100 flex flex-col md:flex-row justify-between items-start md:items-center hover:bg-blue-50 transition gap-3 ${g.status!=='ACTIVE'?'opacity-60 bg-gray-50':''}">
                <div class="truncate">
                    <h4 class="font-black text-sm text-gray-800 truncate">${i+1}. ${g.name}</h4>
                    <p class="text-[10px] text-gray-500 font-mono font-bold mt-1">@${g.username} | <span class="${g.status==='ACTIVE'?'text-green-600':'text-red-500'}">${g.status}</span></p>
                </div>
                <div class="flex gap-2 w-full md:w-auto shrink-0 justify-end">
                    <button onclick="resetPassGuru('${g.id}')" class="text-[10px] bg-yellow-100 text-yellow-800 font-bold px-3 py-1.5 rounded-lg border border-yellow-200"><i class="fa-solid fa-key mr-1"></i> Reset Pass</button>
                    <button onclick="toggleStatusGuru('${g.id}', '${g.status}')" class="text-[10px] font-black px-3 py-1.5 rounded-lg border ${g.status==='ACTIVE'?'bg-red-50 text-red-600 border-red-200':'bg-green-50 text-green-600 border-green-200'}">
                        ${g.status==='ACTIVE'?'Nonaktifkan':'Aktifkan'}
                    </button>
                </div>
            </li>
        `).join('');
    }

    async function filterAdminGuru() {
        const term = document.getElementById('adm-search-guru').value.toLowerCase();
        const gurus = await db.users.where('role').equals('GURU').toArray();
        renderAdminGuruList(gurus.filter(g => g.name.toLowerCase().includes(term) || g.username.toLowerCase().includes(term)));
    }

    function downloadTemplateGuru() {
        const ws = XLSX.utils.aoa_to_sheet([ ["nama_guru", "username", "password", "status"], ["Fauzan Ahmad", "fauzan", "guru123", "AKTIF"] ]);
        const wb = XLSX.utils.book_new(); XLSX.utils.book_append_sheet(wb, ws, "Data Guru");
        XLSX.writeFile(wb, "Template_Import_Guru.xlsx");
    }

    async function handleImportGuru(event) {
        const file = event.target.files[0]; if(!file) return;
        Swal.fire({title: 'Memproses Excel...', allowOutsideClick:false, didOpen:()=>{Swal.showLoading()}});
        
        const reader = new FileReader();
        reader.onload = async (e) => {
            try {
                const workbook = XLSX.read(new Uint8Array(e.target.result), {type: 'array'});
                const json = XLSX.utils.sheet_to_json(workbook.Sheets[workbook.SheetNames[0]], {header: 1});
                const headers = json[0].map(h => String(h).toLowerCase().trim().replace(/\s/g,'_'));
                const idxNama = headers.findIndex(h => h.includes('nama'));
                const idxUname = headers.findIndex(h => h.includes('user'));
                const idxPass = headers.findIndex(h => h.includes('pass'));
                const idxStat = headers.findIndex(h => h.includes('stat'));

                if (idxNama === -1 || idxUname === -1 || idxPass === -1) throw new Error("Format kolom salah. Download template terlebih dahulu.");

                let success=0, dup=0, err=0; let logs=[];

                await db.transaction('rw', db.users, async () => {
                    for(let i=1; i<json.length; i++) {
                        const row = json[i]; if(!row || row.length===0) continue;
                        const name = String(row[idxNama]||'').trim();
                        const uname = String(row[idxUname]||'').trim().toLowerCase().replace(/\s/g,'');
                        const pass = String(row[idxPass]||'').trim();
                        const stat = (String(row[idxStat]||'AKTIF').toUpperCase() === 'NONAKTIF') ? 'INACTIVE' : 'ACTIVE';

                        if(!name || !uname || !pass) { err++; continue; }
                        
                        const exist = await db.users.where('username').equals(uname).count();
                        if(exist > 0) { dup++; logs.push(`Username '${uname}' sudah ada.`); } 
                        else {
                            const hashed = await hashString(pass);
                            await db.users.put({ id: 'G_'+Date.now()+'_'+i, role: 'GURU', username: uname, password: hashed, name: name, status: stat });
                            success++;
                        }
                    }
                });
                event.target.value = ''; initAdminDash();
                let html = `<b>Berhasil:</b> ${success} Guru<br><b>Duplikat (Dilewati):</b> ${dup}<br><b>Error:</b> ${err}`;
                Swal.fire({title: 'Hasil Import', html: html, icon: success>0?'success':'warning'});
            } catch(err) { event.target.value = ''; Swal.fire('Gagal', err.message, 'error'); }
        }; reader.readAsArrayBuffer(file);
    }

    async function saveGuruManual() {
        const name = document.getElementById('inp-guru-nama').value.trim();
        const uname = document.getElementById('inp-guru-uname').value.trim().toLowerCase().replace(/\s/g,'');
        const pass = document.getElementById('inp-guru-pass').value;

        if(pass.length < 6) return Swal.fire('Error', 'Password minimal 6 karakter', 'error');
        const exist = await db.users.where('username').equals(uname).count();
        if(exist > 0) return Swal.fire('Gagal', 'Username sudah terpakai.', 'error');

        const hashed = await hashString(pass);
        await db.users.put({ id: 'G_'+Date.now(), role: 'GURU', username: uname, password: hashed, name: name, status: 'ACTIVE' });
        
        closeModal('modal-add-guru'); document.getElementById('inp-guru-nama').value=''; document.getElementById('inp-guru-uname').value=''; document.getElementById('inp-guru-pass').value='';
        Swal.fire('Sukses', 'Akun Guru berhasil dibuat.', 'success'); initAdminDash();
    }
    
    async function toggleStatusGuru(id, current) { await db.users.update(id, {status: current==='ACTIVE'?'INACTIVE':'ACTIVE'}); initAdminDash(); }
    function resetPassGuru(id) {
        Swal.fire({title: 'Reset Password?', text: "Password akan diubah menjadi '123456'", icon: 'warning', showCancelButton: true}).then(async r=>{
            if(r.isConfirmed){ const h = await hashString('123456'); await db.users.update(id, {password: h}); Swal.fire('Berhasil', 'Password direset.', 'success'); }
        });
    }

    // 4. Modul GURU (Inti Sistem Kelas & Siswa)
    function guruSwitchView(viewId) {
        document.querySelectorAll('.nav-item').forEach(el => el.classList.toggle('active', el.dataset.target === viewId));
        document.querySelectorAll('.guru-active-view').forEach(el => el.classList.add('hidden'));
        document.getElementById(`guru-view-${viewId}`).classList.remove('hidden');

        // Pastikan kamera mati jika keluar dari tab scan
        if(viewId !== 'scan' && html5QrCode) { 
            try { html5QrCode.stop().then(() => { html5QrCode.clear(); document.getElementById('scan-active-panel').classList.add('hidden'); document.getElementById('scan-setup-panel').classList.remove('hidden'); activeSession=null; }); } 
            catch(e){} 
        }

        if(viewId === 'beranda') initGuruDash();
        if(viewId === 'siswa') loadGuruClassesFilter(); // Memuat dropdown kelas dan list siswa
        if(viewId === 'scan') loadGuruScanSetup();
        if(viewId === 'rekap') loadGuruRekap();
    }

    async function initGuruDash() {
        showScreen('screen-guru');
        document.getElementById('guru-header-name').innerText = currentUser.name;
        document.getElementById('guru-date-today').innerText = formatDate(Date.now());
        
        // Load UI Datalist manual input
        guruCachedClasses = await db.classes.where('teacherId').equals(currentUser.id).toArray();
        document.getElementById('datalist-kelas').innerHTML = guruCachedClasses.map(c=>`<option value="${c.name}">`).join('');

        document.getElementById('guru-stat-kelas').innerText = guruCachedClasses.length;
        document.getElementById('guru-stat-siswa').innerText = await db.students.where('teacherId').equals(currentUser.id).count();
    }

    // A. Manajemen Siswa & Kelas (Auto-Generate Class dari Excel)
    async function loadGuruClassesFilter() {
        guruCachedClasses = await db.classes.where('teacherId').equals(currentUser.id).toArray();
        guruCachedClasses.sort((a,b)=>a.name.localeCompare(b.name));
        
        const sel = document.getElementById('filter-siswa-kelas');
        sel.innerHTML = '<option value="">Semua Kelas</option>' + guruCachedClasses.map(c=>`<option value="${c.id}">${c.name}</option>`).join('');
        loadGuruSiswa();
    }

    async function loadGuruSiswa() {
        const cId = document.getElementById('filter-siswa-kelas').value;
        if(cId) guruCachedStudents = await db.students.where({teacherId: currentUser.id, classId: parseInt(cId)}).toArray();
        else guruCachedStudents = await db.students.where('teacherId').equals(currentUser.id).toArray();
        
        document.getElementById('search-siswa').value = '';
        renderGuruStudentsList(guruCachedStudents);
    }

    function renderGuruStudentsList(list) {
        const ul = document.getElementById('guru-list-siswa');
        if(list.length === 0) { 
            ul.innerHTML = '<div class="text-center text-gray-400 text-xs font-bold py-16">Belum ada siswa. Silakan Import Excel.</div>'; 
            return; 
        }
        
        list.sort((a,b) => a.name.localeCompare(b.name));
        ul.innerHTML = list.map((s,i) => {
            const cObj = guruCachedClasses.find(c=>c.id===s.classId);
            const cName = cObj ? cObj.name : '?';
            return `
            <li class="p-4 flex flex-col sm:flex-row justify-between items-start sm:items-center hover:bg-blue-50 transition gap-3">
                <div class="truncate">
                    <div class="flex items-center gap-2 mb-1">
                        <span class="text-[10px] font-black bg-blue-100 text-blue-800 px-2 py-0.5 rounded uppercase">${cName}</span>
                        <span class="text-[10px] text-gray-500 font-bold font-mono">NIS: ${s.nis}</span>
                    </div>
                    <h4 class="font-black text-sm text-gray-800 truncate">${s.name}</h4>
                </div>
                <div class="flex space-x-2 shrink-0 w-full sm:w-auto justify-end mt-2 sm:mt-0">
                    <button onclick="printSingleQR('${s.qrToken}', '${s.name}', '${s.nis}', '${cName}')" class="text-[10px] bg-blue-50 text-blue-600 font-bold px-3 py-2 rounded-lg border border-blue-100"><i class="fa-solid fa-qrcode mr-1"></i> Lihat QR</button>
                    <button onclick="delGuruSiswa(${s.id})" class="text-[10px] bg-red-50 text-red-500 font-bold px-3 py-2 rounded-lg border border-red-100"><i class="fa-solid fa-trash-can mr-1"></i> Hapus</button>
                </div>
            </li>
        `}).join('');
    }

    function filterGuruStudents() {
        const term = document.getElementById('search-siswa').value.toLowerCase();
        const f = guruCachedStudents.filter(s => s.name.toLowerCase().includes(term) || s.nis.toLowerCase().includes(term));
        renderGuruStudentsList(f);
    }

    function downloadTemplateSiswa() {
        const ws = XLSX.utils.aoa_to_sheet([
            ["nis", "nama_siswa", "kelas", "jenis_kelamin"], 
            ["1001", "Ahmad Rizky", "VII A", "L"], 
            ["1002", "Budi Santoso", "VII A", "L"],
            ["1003", "Citra Ayu", "VII B", "P"]
        ]);
        const wb = XLSX.utils.book_new(); XLSX.utils.book_append_sheet(wb, ws, "Data Siswa");
        XLSX.writeFile(wb, "Template_Data_Siswa.xlsx");
    }

    async function handleImportSiswa(event) {
        const file = event.target.files[0]; if(!file) return;
        Swal.fire({title: 'Membaca File...', html:'Sistem mendeteksi kelas dan membuat QR Otomatis.', allowOutsideClick:false, didOpen:()=>{Swal.showLoading()}});
        
        const reader = new FileReader();
        reader.onload = async (e) => {
            try {
                const workbook = XLSX.read(new Uint8Array(e.target.result), {type: 'array'});
                const json = XLSX.utils.sheet_to_json(workbook.Sheets[workbook.SheetNames[0]], {header: 1});
                
                const headers = json[0].map(h => String(h).toLowerCase().trim().replace(/\s/g,'_'));
                const idxNis = headers.findIndex(h => h.includes('nis'));
                const idxNama = headers.findIndex(h => h.includes('nama'));
                const idxKelas = headers.findIndex(h => h.includes('kelas'));
                const idxJk = headers.findIndex(h => h.includes('jenis'));

                if(idxNis===-1 || idxNama===-1 || idxKelas===-1) throw new Error("Format kolom Excel salah. Wajib ada NIS, NAMA, dan KELAS.");

                let success=0, dup=0;
                let classSet = new Set(); 

                // 1. Temukan kelas unik dari Excel
                for(let i=1; i<json.length; i++) {
                    const row = json[i]; if(!row || row.length===0) continue;
                    const k = String(row[idxKelas]||'').trim().toUpperCase();
                    if(k) classSet.add(k);
                }

                await db.transaction('rw', db.classes, db.students, async () => {
                    // 2. Buat mapping kelas (Buat baru jika belum ada)
                    let classMap = {}; 
                    for (const clsName of classSet) {
                        let existCls = await db.classes.where({teacherId: currentUser.id, name: clsName}).first();
                        if (existCls) { classMap[clsName] = existCls.id; } 
                        else {
                            const newId = await db.classes.add({ teacherId: currentUser.id, name: clsName });
                            classMap[clsName] = newId;
                        }
                    }

                    // 3. Masukkan Siswa & Generate QR Otomatis
                    for(let i=1; i<json.length; i++) {
                        const row = json[i]; if(!row || row.length===0) continue;
                        
                        const nis = String(row[idxNis]||'').trim();
                        const nama = String(row[idxNama]||'').trim().toUpperCase();
                        const kelasName = String(row[idxKelas]||'').trim().toUpperCase();
                        const jk = (String(row[idxJk]||'L').trim().toUpperCase().charAt(0) === 'P') ? 'P' : 'L';

                        if(!nis || !nama || !kelasName) continue;

                        const existStudent = await db.students.where({teacherId: currentUser.id, nis: nis}).first();
                        if(existStudent) { dup++; } 
                        else {
                            const token = generateToken(nis);
                            await db.students.add({ 
                                teacherId: currentUser.id, classId: classMap[kelasName], 
                                nis: nis, name: nama, gender: jk, qrToken: token 
                            });
                            success++;
                        }
                    }
                });

                event.target.value = ''; 
                loadGuruClassesFilter(); 
                
                let html = `<b>Berhasil Diimpor:</b> ${success} Siswa<br><b>Kelas Dibuat Otomatis:</b> ${classSet.size} Kelas<br><b>QR Code Otomatis:</b> ${success}<br><b>Duplikat (Dilewati):</b> ${dup}`;
                Swal.fire({title: 'Import Selesai', html: html, icon: 'success'});
                
            } catch(error) { event.target.value = ''; Swal.fire('Error Import', error.message, 'error'); }
        }; 
        reader.readAsArrayBuffer(file);
    }

    function openModalAddSiswa() { openModal('modal-add-siswa'); }
    async function saveSiswaManual() {
        const clsName = document.getElementById('inp-sw-kelas').value.trim().toUpperCase();
        const nis = document.getElementById('inp-sw-nis').value.trim();
        const name = document.getElementById('inp-sw-nama').value.trim().toUpperCase();
        const jk = document.getElementById('inp-sw-jk').value;
        if(!clsName || !nis || !name) return;

        const exist = await db.students.where({teacherId: currentUser.id, nis: nis}).first();
        if(exist) return Swal.fire('Gagal', `NIS ${nis} sudah terdaftar.`, 'error');

        // Otomatis tangani kelas
        let clsId;
        let existCls = await db.classes.where({teacherId: currentUser.id, name: clsName}).first();
        if(existCls) clsId = existCls.id; else clsId = await db.classes.add({ teacherId: currentUser.id, name: clsName });

        const token = generateToken(nis);
        await db.students.add({ teacherId: currentUser.id, classId: clsId, nis: nis, name: name, gender: jk, qrToken: token });
        
        closeModal('modal-add-siswa'); document.getElementById('inp-sw-nis').value=''; document.getElementById('inp-sw-nama').value=''; 
        loadGuruClassesFilter(); Swal.fire('Sukses', 'Siswa disimpan & QR Code otomatis dibuat.', 'success');
    }

    async function delGuruSiswa(id) {
        Swal.fire({title: 'Hapus Permanen?', text: 'Data siswa tidak dapat dikembalikan.', icon: 'warning', showCancelButton: true}).then(async r=>{
            if(r.isConfirmed) { await db.students.delete(id); loadGuruSiswa(); }
        });
    }

    function printQRMassal() {
        const cId = parseInt(document.getElementById('filter-siswa-kelas').value);
        if(!cId) return Swal.fire('Pilih Kelas', 'Pilih kelas di dropdown untuk mencetak semua QR kelas tersebut.', 'info');
        executePrintQRClass(cId);
    }

    async function executePrintQRClass(cId) {
        const cls = await db.classes.get(cId);
        const students = await db.students.where({teacherId: currentUser.id, classId: cId}).toArray();
        if(students.length===0) return;
        
        Swal.fire({title: 'Membuat Dokumen Cetak...', allowOutsideClick:false, didOpen:()=>{Swal.showLoading()}});
        
        const renderArea = document.getElementById('qr-render-area'); renderArea.innerHTML = '';
        let printWin = window.open('', '_blank');
        
        let html = `<html><head><title>Cetak QR - ${cls.name}</title><style>body{font-family:sans-serif; margin:0; padding:10mm;} .grid{display:grid; grid-template-columns:repeat(auto-fit, minmax(160px, 1fr)); gap:10px;} .card{border:1px solid #000; padding:10px; text-align:center; border-radius:8px; page-break-inside:avoid;} .h{font-size:10px; font-weight:bold; border-bottom:1px solid #000; padding-bottom:3px; margin-bottom:5px;} .qr{margin:5px auto; width:100px; height:100px;} .qr img{width:100%; height:100%;} .n{font-size:12px; font-weight:bold; margin-top:3px;} .m{font-size:9px;} @media print{ @page{margin:5mm;} button{display:none;} }</style></head><body><button onclick="window.print()" style="padding:10px; background:#1e40af; color:#fff; font-weight:bold; width:100%; border:none; margin-bottom:10px;">PRINT</button><div class="grid">`;

        for(let s of students) {
            const t = document.createElement('div'); renderArea.appendChild(t);
            const getUri = () => new Promise(res => {
                new QRCode(t, {text: s.qrToken, width: 128, height: 128, correctLevel: QRCode.CorrectLevel.M});
                setTimeout(()=>{ const i=t.querySelector('img'), c=t.querySelector('canvas'); if(i&&i.src.length>50) res(i.src); else if(c) res(c.toDataURL("image/png")); else res(''); }, 40);
            });
            const uri = await getUri();
            html += `<div class="card"><div class="h">ABSENSI KELAS</div><div class="qr"><img src="${uri}" /></div><div class="n">${s.name}</div><div class="m">NIS: ${s.nis}<br>Kelas: ${cls.name}</div></div>`;
        }
        
        html += `</div></body></html>`;
        printWin.document.write(html); printWin.document.close(); Swal.close();
        renderArea.innerHTML = '';
    }

    function printSingleQR(token, name, nis, clsName) {
        let printWin = window.open('', '_blank'); const t = document.createElement('div');
        new QRCode(t, {text: token, width: 256, height: 256});
        setTimeout(()=>{ 
            const i=t.querySelector('img');
            let html = `<html><head><style>body{text-align:center; font-family:sans-serif;} .card{border:2px solid #000; padding:20px; display:inline-block; border-radius:10px; margin-top:20px;} img{width:200px; height:200px; margin:10px auto;} @media print{button{display:none;}}</style></head><body><button onclick="window.print()" style="padding:10px; background:blue; color:white;">PRINT</button><br><div class="card"><h3>ABSENSI KELAS</h3><img src="${i.src}" /><h2>${name}</h2><p>NIS: ${nis} | Kelas: ${clsName}</p></div></body></html>`;
            printWin.document.write(html); printWin.document.close();
        }, 50);
    }


    // ==============================================================
    // 5. CORE ATTENDANCE ENGINE (Scanner Real, Validasi, List)
    // ==============================================================
    
    async function loadGuruScanSetup() {
        guruCachedClasses = await db.classes.where('teacherId').equals(currentUser.id).toArray();
        const sel = document.getElementById('scan-sel-kelas');
        sel.innerHTML = '<option value="">-- Pilih Kelas --</option>' + guruCachedClasses.map(c=>`<option value="${c.id}">${c.name}</option>`).join('');
        
        const now = new Date(); now.setMinutes(now.getMinutes() + 15);
        document.getElementById('scan-time-late').value = now.toTimeString().slice(0,5);
    }

    async function startScannerSession() {
        const cId = parseInt(document.getElementById('scan-sel-kelas').value);
        const lateStr = document.getElementById('scan-time-late').value;
        if(!cId || !lateStr) return Swal.fire('Oops', 'Pilih Kelas dan isi Batas Jam Hadir.', 'warning');
        
        const cls = await db.classes.get(cId);
        const students = await db.students.where({teacherId: currentUser.id, classId: cId}).toArray();
        if(students.length === 0) return Swal.fire('Kosong', 'Belum ada siswa di kelas ini.', 'error');

        // Buat DB Session Baru
        const [hh, mm] = lateStr.split(':'); const dObj = new Date(); dObj.setHours(hh, mm, 0, 0);
        const now = Date.now();
        const sId = await db.sessions.add({ classId: cId, teacherId: currentUser.id, date: now, timeStart: now, timeEnd: null, status: 'OPEN' });
        
        activeSession = await db.sessions.get(sId);
        activeSession._cls = cls;
        activeSession._students = students;
        activeSession._lateLimit = dObj.getTime();
        
        // Transisi UI Scanner
        document.getElementById('scan-setup-panel').classList.add('hidden');
        document.getElementById('scan-active-panel').classList.remove('hidden');
        document.getElementById('scan-active-panel').classList.add('flex');
        
        document.getElementById('live-class-name').innerText = `ABSENSI ${cls.name}`;
        document.getElementById('live-time-info').innerText = `Batas: ${lateStr}`;
        document.getElementById('live-scan-list').innerHTML = '';
        
        updateLiveStats();
        initCamera(); // Memulai kamera sesungguhnya
    }

    async function initCamera() {
        try {
            availableCameras = await Html5Qrcode.getCameras();
            if(availableCameras && availableCameras.length > 0) {
                // Utamakan kamera belakang
                let backCamIndex = availableCameras.findIndex(c => c.label.toLowerCase().includes('back') || c.label.toLowerCase().includes('environment'));
                cameraIndex = backCamIndex !== -1 ? backCamIndex : 0;
                startRealScanning();
            } else {
                throw new Error("Tidak ada kamera terdeteksi di perangkat.");
            }
        } catch(err) {
            Swal.fire('Kamera Error', 'Izin kamera ditolak atau tidak tersedia. Gunakan tombol Absen Manual.', 'error');
            console.error(err);
        }
    }

    function startRealScanning() {
        if(html5QrCode) { try { html5QrCode.stop(); } catch(e){} html5QrCode.clear(); }
        html5QrCode = new Html5Qrcode("reader");
        
        const camId = availableCameras[cameraIndex].id;
        // Konfigurasi agar responsif & stabil membaca QR kecil/jauh
        html5QrCode.start(
            camId, 
            { fps: 10, qrbox: { width: 250, height: 250 }, aspectRatio: 1.0, disableFlip: false }, 
            onScanSuccess, 
            undefined // Ignore failures to keep console clean
        ).catch(err => { console.error("Scanner Start Error", err); });
    }

    function switchCamera() {
        if(availableCameras.length < 2) return; // Swal.fire('Info', 'Hanya 1 kamera.', 'info'); //Silent ignore for better UX
        cameraIndex = (cameraIndex + 1) % availableCameras.length;
        if(html5QrCode) {
            try { html5QrCode.stop().then(() => startRealScanning()); } 
            catch(e){ startRealScanning(); }
        }
    }

    // THE VALIDATION ENGINE (Dipanggil saat QR terdeteksi)
    async function onScanSuccess(decodedText) {
        if(isProcessingScan || !activeSession) return;
        isProcessingScan = true; // Lock mechanism

        try {
            const token = decodedText.trim();
            // 1. Cari Siswa berdasarkan QR
            const student = await db.students.where('qrToken').equals(token).first();

            if(!student) {
                showScannerToast('QR TIDAK DIKENAL', 'Bukan QR Sistem Ini', 'error'); 
                playBeep('error');
            } 
            // 2. Validasi Salah Kelas
            else if (student.classId !== activeSession.classId || student.teacherId !== currentUser.id) {
                const wrongClass = await db.classes.get(student.classId);
                showScannerToast('SALAH KELAS', `Siswa ${wrongClass?wrongClass.name:'kelas lain'}`, 'error'); 
                playBeep('error');
            } 
            else {
                // 3. Cek Duplikat Hari Ini (berdasarkan date YYYY-MM-DD dan studentId)
                const todayStr = new Date().toISOString().split('T')[0];
                const exist = await db.attendance.where({studentId: student.id, classId: student.classId, date: todayStr}).first();

                if(exist) {
                    showScannerToast('SUDAH ABSEN', student.name, 'warning'); 
                    playBeep('error');
                } else {
                    // 4. Kalkulasi Status (Hadir/Telat)
                    const now = Date.now();
                    const stat = now > activeSession._lateLimit ? 'TERLAMBAT' : 'HADIR';
                    
                    // Simpan
                    await db.attendance.add({ sessionId: activeSession.id, studentId: student.id, classId: student.classId, teacherId: currentUser.id, date: todayStr, status: stat, timestamp: now, method: 'QR_SCAN' });
                    
                    // Feedback Sukses
                    showScannerToast(stat, student.name, 'success'); 
                    playBeep('success');
                    updateLiveStats();
                }
            }
        } catch(e) { console.error(e); }
        
        // Cooldown sebelum membaca QR berikutnya
        setTimeout(() => { isProcessingScan = false; }, 1200); 
    }

    // UI Feedback Cepat di atas Kamera
    function showScannerToast(title, desc, type) {
        const toast = document.getElementById('scan-toast');
        const icon = document.getElementById('scan-toast-icon');
        const titEl = document.getElementById('scan-toast-title');
        const desEl = document.getElementById('scan-toast-desc');

        titEl.innerText = title; desEl.innerText = desc;
        
        toast.className = `absolute top-[30%] left-1/2 transform -translate-x-1/2 -translate-y-1/2 px-6 py-4 rounded-3xl shadow-[0_10px_40px_rgba(0,0,0,0.5)] font-black flex flex-col items-center text-center transition-all duration-200 z-[100] min-w-[280px] border-4 scale-100 opacity-100 bg-white`;
        
        if(type==='success'){ toast.classList.add('border-green-500'); icon.className='fa-solid fa-check-circle text-5xl mb-2 text-green-500'; titEl.className='text-sm uppercase tracking-widest text-green-600 mb-1'; }
        else if(type==='warning'){ toast.classList.add('border-yellow-500'); icon.className='fa-solid fa-triangle-exclamation text-5xl mb-2 text-yellow-500'; titEl.className='text-sm uppercase tracking-widest text-yellow-600 mb-1'; }
        else { toast.classList.add('border-red-500'); icon.className='fa-solid fa-xmark text-5xl mb-2 text-red-500'; titEl.className='text-sm uppercase tracking-widest text-red-600 mb-1'; }

        // Hilangkan otomatis
        setTimeout(() => { toast.classList.replace('scale-100', 'scale-90'); toast.classList.replace('opacity-100', 'opacity-0'); }, 1500);
    }

    // Live Statistik & Daftar yang merender seluruh siswa di kelas saat sesi aktif
    async function updateLiveStats() {
        if(!activeSession) return;
        
        const todayStr = new Date().toISOString().split('T')[0];
        const att = await db.attendance.where({classId: activeSession.classId, date: todayStr}).toArray();
        const total = activeSession._students.length;
        
        let h=0, t=0, izsk=0;
        const presentMap = {}; 
        att.forEach(a => { 
            presentMap[a.studentId] = a; 
            if(a.status==='HADIR')h++; else if(a.status==='TERLAMBAT')t++; else if(a.status==='IZIN'||a.status==='SAKIT')izsk++; 
        });
        
        const belum = total - att.length;
        
        document.getElementById('ls-total').innerText = total; document.getElementById('ls-hadir').innerText = h; 
        document.getElementById('ls-telat').innerText = t; document.getElementById('ls-belum').innerText = belum;

        // Render List Real-time
        const listUl = document.getElementById('live-scan-list');
        
        // Urutkan: Yang baru absen di atas (jika ada absen), yang belum di bawah
        let combinedList = activeSession._students.map(s => {
            const a = presentMap[s.id];
            return { student: s, att: a, ts: a ? a.timestamp : 0 };
        });
        
        // Sort by timestamp descending (yang baru absen di atas), jika belum absen (ts=0) akan di bawah
        combinedList.sort((a,b) => b.ts - a.ts);
        
        let html = '';
        combinedList.forEach(item => {
            const s = item.student; const a = item.att;
            if(a) {
                let c = 'text-green-600 bg-green-50'; if(a.status==='TERLAMBAT') c='text-orange-600 bg-orange-50'; else if(['IZIN','SAKIT'].includes(a.status)) c='text-blue-600 bg-blue-50'; else if(a.status==='ALPA') c='text-red-600 bg-red-50';
                html += `<li class="flex justify-between items-center text-xs p-2 rounded-xl mb-1 border border-gray-100 ${c} animate-[fadeIn_0.3s_ease]"><div class="truncate"><span class="font-black text-gray-800">${s.name}</span></div><div class="flex items-center gap-2 shrink-0"><span class="font-black tracking-wider uppercase">${a.status}</span><span class="text-[9px] text-gray-500 font-mono">${formatTimeShort(a.timestamp)}</span></div></li>`;
            } else {
                html += `<li class="flex justify-between items-center text-xs p-2 rounded-xl mb-1 border border-gray-100 bg-gray-50 opacity-70"><div class="truncate"><span class="font-bold text-gray-600">${s.name}</span></div><div class="shrink-0"><span class="text-[9px] font-black bg-gray-200 text-gray-500 px-2 py-0.5 rounded uppercase">Belum</span></div></li>`;
            }
        });
        listUl.innerHTML = html;
    }

    // Modal Absen Manual untuk backup
    async function openManualAbsen() {
        if(!activeSession) return;
        const todayStr = new Date().toISOString().split('T')[0];
        const att = await db.attendance.where({classId: activeSession.classId, date: todayStr}).toArray();
        const pIds = new Set(att.map(a=>a.studentId));
        const absent = activeSession._students.filter(s => !pIds.has(s.id)); 
        
        if(absent.length === 0) return Swal.fire('Selesai', 'Semua siswa di kelas ini sudah melakukan absensi hari ini.', 'info');

        let html = `<select id="man-stu" class="w-full p-3 border border-gray-200 rounded-xl mb-3 text-sm font-bold bg-gray-50 outline-none">` + absent.map(s => `<option value="${s.id}">${s.name}</option>`).join('') + `</select>
        <select id="man-stat" class="w-full p-3 border border-gray-200 rounded-xl text-sm font-bold bg-gray-50 outline-none"><option value="HADIR">HADIR</option><option value="TERLAMBAT">TERLAMBAT</option><option value="IZIN">IZIN</option><option value="SAKIT">SAKIT</option><option value="ALPA">ALPA</option></select>`;
        
        Swal.fire({ title: 'Absen Manual', html: html, showCancelButton: true, confirmButtonText: 'Simpan', confirmButtonColor: '#1e40af' })
        .then(async (r) => {
            if(r.isConfirmed) {
                const sId = parseInt(document.getElementById('man-stu').value);
                const stat = document.getElementById('man-stat').value;
                const s = absent.find(x => x.id === sId); const now = Date.now();
                
                await db.attendance.add({ sessionId: activeSession.id, studentId: s.id, classId: s.classId, teacherId: currentUser.id, date: todayStr, status: stat, timestamp: now, method: 'MANUAL' });
                updateLiveStats();
            }
        });
    }

    async function confirmCloseSession() {
        Swal.fire({ title: 'Tutup Kamera & Sesi?', text:'Anda akan kembali ke menu pengaturan absensi.', icon:'warning', showCancelButton: true, confirmButtonColor: '#dc2626' }).then(r => { 
            if(r.isConfirmed) executeCloseSession(); 
        });
    }

    async function executeCloseSession() {
        if(html5QrCode) { try { await html5QrCode.stop(); html5QrCode.clear(); } catch(e){} }
        if(activeSession) {
            await db.sessions.update(activeSession.id, { timeEnd: Date.now(), status: 'CLOSED' });
            Swal.fire('Selesai', 'Kamera dimatikan. Data absensi tersimpan.', 'success'); 
            activeSession = null;
        }
        document.getElementById('scan-active-panel').classList.add('hidden'); document.getElementById('scan-active-panel').classList.remove('flex');
        document.getElementById('scan-setup-panel').classList.remove('hidden');
    }


    // ==============================================================
    // 6. MODULE REKAP & EXPORT (SheetJS Export Asli)
    // ==============================================================
    async function loadGuruRekap() {
        guruCachedClasses = await db.classes.where('teacherId').equals(currentUser.id).toArray();
        document.getElementById('export-kelas').innerHTML = '<option value="">Semua Kelas</option>' + guruCachedClasses.map(c => `<option value="${c.id}">${c.name}</option>`).join('');

        const sess = await db.sessions.where('teacherId').equals(currentUser.id).and(s => s.status === 'CLOSED').reverse().limit(20).toArray();
        const list = document.getElementById('guru-list-sesi');
        if(sess.length === 0) { list.innerHTML = '<div class="p-8 text-center text-xs text-gray-400 font-bold">Belum ada riwayat sesi tertutup.</div>'; return; }
        
        let html = '';
        for(let s of sess) {
            const cls = guruCachedClasses.find(x => x.id === s.classId);
            const attCount = await db.attendance.where('sessionId').equals(s.id).count();
            html += `
            <div class="p-4 bg-white hover:bg-blue-50 transition border-b border-gray-100 flex justify-between items-center">
                <div>
                    <span class="text-[10px] font-black bg-blue-100 text-blue-800 px-2 py-0.5 rounded uppercase">${cls?cls.name:'?'}</span>
                    <p class="text-xs font-black text-gray-800 mt-1">${formatDate(s.date)}</p>
                </div>
                <div class="text-right">
                    <p class="text-[10px] text-gray-500 font-mono font-bold">${formatTimeShort(s.timeStart)}</p>
                    <p class="text-[9px] font-black text-green-600 mt-1 uppercase">${attCount} Tercatat</p>
                </div>
            </div>`;
        }
        list.innerHTML = html;
    }

    async function exportExcelRekap() {
        const cId = parseInt(document.getElementById('export-kelas').value);
        const tgl = document.getElementById('export-tanggal').value; // format YYYY-MM-DD
        Swal.fire({title: 'Membuat Laporan Excel...', allowOutsideClick:false, didOpen:()=>{Swal.showLoading()}});

        try {
            // Filter
            let query = db.attendance.where('teacherId').equals(currentUser.id);
            if(cId && tgl) query = db.attendance.where({classId: cId, date: tgl});
            else if(cId) query = db.attendance.where('classId').equals(cId);
            // Index 'date' tidak ada langsung di level atas, gunakan filter manual jika hanya tgl
            
            let allAtt = await query.toArray();
            if(!cId && tgl) allAtt = allAtt.filter(a => a.date === tgl);
            
            if(allAtt.length === 0) return Swal.fire('Kosong', 'Tidak ada data absensi untuk kriteria ini.', 'info');
            
            const allStudents = await db.students.where('teacherId').equals(currentUser.id).toArray();
            const allClasses = await db.classes.where('teacherId').equals(currentUser.id).toArray();
            
            const wsData = [
                ["LAPORAN ABSENSI KELAS (SYSTEM EXPORT)"],
                ["Guru:", currentUser.name], ["Filter Tanggal:", tgl || "Semua Waktu"], [],
                ["TANGGAL", "KELAS", "NIS", "NAMA SISWA", "STATUS ABSEN", "WAKTU SCAN", "METODE"]
            ];
            
            allAtt.sort((a,b) => a.timestamp - b.timestamp);

            for(let a of allAtt) {
                const stu = allStudents.find(x => x.id === a.studentId);
                const cls = allClasses.find(x => x.id === a.classId);
                if(stu && cls) {
                    wsData.push([a.date, cls.name, stu.nis, stu.name, a.status, formatTimeShort(a.timestamp), a.method]);
                }
            }
            
            const ws = XLSX.utils.aoa_to_sheet(wsData); 
            ws['!cols'] = [{wch:12}, {wch:12}, {wch:12}, {wch:30}, {wch:15}, {wch:15}, {wch:12}];
            const wb = XLSX.utils.book_new(); 
            XLSX.utils.book_append_sheet(wb, ws, "Rekap Absensi");
            
            let fName = `Laporan_Absen_${cId?'SatuKelas':'SemuaKelas'}_${tgl||'All'}.xlsx`;
            XLSX.writeFile(wb, fName);
            Swal.close();
        } catch(e) { console.error(e); Swal.fire('Error', 'Gagal membuat file Excel.', 'error'); }
    }

</script>
</body>
</html>
