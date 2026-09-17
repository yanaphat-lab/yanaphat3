<!DOCTYPE html>
<html lang="th">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>ระบบเช็คชื่อเข้าแถวหน้าเสาธงดิจิทัล (Real-time Sync)</title>
    <style>
        :root {
            --bg-main: #0f172a;        
            --card-bg: #1e293b;        
            --primary: #3b82f6;        
            --primary-hover: #2563eb;  
            --text-main: #f8fafc;      
            --text-sub: #94a3b8;       
            --success: #22c55e;        
            --danger: #ef4444;         
            --border-color: #334155;   
        }

        body {
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
            background-color: var(--bg-main);
            margin: 0;
            padding: 16px;
            color: var(--text-main);
        }

        .container {
            max-width: 600px;
            margin: 0 auto;
            display: flex;
            flex-direction: column;
            gap: 16px;
        }

        .card {
            background: var(--card-bg);
            padding: 20px;
            border-radius: 12px;
            box-shadow: 0 4px 12px rgba(0, 0, 0, 0.3);
            border: 1px solid var(--border-color);
            position: relative;
        }

        .top-bar {
            display: flex;
            justify-content: space-between;
            align-items: center;
            margin-bottom: 12px;
        }

        .user-status-badge {
            font-size: 0.8rem;
            padding: 4px 10px;
            border-radius: 20px;
            background: #334155;
            color: var(--text-sub);
        }

        .user-status-badge.admin {
            background: #1e3a8a;
            color: #93c5fd;
            border: 1px solid #3b82f6;
        }

        .sync-indicator {
            font-size: 0.75rem;
            color: var(--text-sub);
            display: flex;
            align-items: center;
            gap: 6px;
        }

        .sync-dot {
            width: 8px;
            height: 8px;
            background-color: #22c55e;
            border-radius: 50%;
            display: inline-block;
            box-shadow: 0 0 8px #22c55e;
        }

        .sync-dot.syncing {
            background-color: #eab308;
            box-shadow: 0 0 8px #eab308;
            animation: pulse 1s infinite alternate;
        }

        @keyframes pulse {
            from { opacity: 0.4; }
            to { opacity: 1; }
        }

        h2 { 
            margin-top: 0; 
            color: #60a5fa; 
            font-size: 1.25rem; 
            display: flex;
            align-items: center;
            gap: 8px;
        }

        /* Login Modal Overlay */
        .modal-overlay {
            position: fixed;
            top: 0; left: 0; width: 100%; height: 100%;
            background: rgba(15, 23, 42, 0.85);
            backdrop-filter: blur(5px);
            display: flex;
            align-items: center;
            justify-content: center;
            z-index: 999;
        }

        .login-card {
            background: var(--card-bg);
            padding: 24px;
            border-radius: 12px;
            width: 90%;
            max-width: 380px;
            border: 1px solid var(--border-color);
            box-shadow: 0 10px 25px rgba(0,0,0,0.5);
            text-align: center;
        }

        .password-input-group {
            position: relative;
            display: flex;
            align-items: center;
        }

        .password-input-group input {
            padding-right: 45px;
        }

        .toggle-password-btn {
            position: absolute;
            right: 8px;
            background: transparent;
            border: none;
            color: var(--text-sub);
            cursor: pointer;
            font-size: 1.1rem;
            padding: 4px 8px;
            margin: 0;
            width: auto;
        }

        .datetime-box {
            text-align: center;
            background: #0f172a;
            padding: 12px;
            border-radius: 8px;
            border: 1px solid var(--border-color);
            margin-bottom: 15px;
        }

        .date-display {
            font-size: 1rem;
            color: var(--text-sub);
            margin-bottom: 4px;
        }

        .clock-display {
            font-size: 1.75rem;
            font-weight: bold;
            color: #60a5fa;
            letter-spacing: 1px;
        }

        .timeline-container {
            position: relative;
            background: linear-gradient(to right, var(--success) 0%, var(--success) 50%, var(--danger) 50%, var(--danger) 100%);
            height: 12px;
            border-radius: 6px;
            margin: 30px 0 10px 0;
            box-shadow: inset 0 2px 4px rgba(0,0,0,0.3);
        }

        .timeline-deadline {
            position: absolute;
            left: 50%;
            top: -24px;
            transform: translateX(-50%);
            font-size: 0.75rem;
            font-weight: bold;
            color: #ffffff;
            background: #0f172a;
            padding: 2px 8px;
            border-radius: 4px;
            border: 1px solid var(--border-color);
            white-space: nowrap;
        }

        .timeline-deadline::after {
            content: '';
            position: absolute;
            bottom: -16px;
            left: 50%;
            transform: translateX(-50%);
            width: 2px;
            height: 16px;
            background: #ffffff;
        }

        .timeline-pointer {
            position: absolute;
            top: -6px;
            width: 24px;
            height: 24px;
            background: var(--primary);
            border: 2px solid #ffffff;
            border-radius: 50%;
            transform: translateX(-50%);
            transition: left 0.5s ease;
            box-shadow: 0 0 10px rgba(59, 130, 246, 0.9);
            z-index: 2;
        }

        .form-row {
            display: flex;
            gap: 10px;
            flex-wrap: wrap;
        }

        .form-group {
            margin-bottom: 15px;
            flex: 1 1 calc(50% - 10px);
            min-width: 130px;
        }

        .form-group.full-width {
            flex: 1 1 100%;
        }

        label {
            display: block;
            margin-bottom: 6px;
            font-weight: 600;
            color: var(--text-main);
            font-size: 0.9rem;
        }

        input[type="text"], input[type="password"], input[type="date"] {
            width: 100%;
            padding: 12px;
            background-color: #0f172a;
            border: 1px solid var(--border-color);
            border-radius: 6px;
            color: var(--text-main);
            box-sizing: border-box;
            font-size: 1rem;
            color-scheme: dark;
        }

        input:focus {
            outline: none;
            border-color: var(--primary);
        }

        .camera-box {
            width: 100%;
            height: 220px;
            background: #0f172a;
            border: 1px solid var(--border-color);
            border-radius: 8px;
            display: flex;
            align-items: center;
            justify-content: center;
            overflow: hidden;
            position: relative;
        }

        video, canvas, img {
            width: 100%;
            height: 100%;
            object-fit: cover;
        }

        button {
            width: 100%;
            padding: 12px;
            background: var(--primary);
            color: white;
            border: none;
            border-radius: 6px;
            font-size: 1rem;
            font-weight: bold;
            cursor: pointer;
            margin-top: 8px;
            transition: background 0.2s;
        }

        button:hover { 
            background: var(--primary-hover); 
        }

        button:disabled {
            background: #475569;
            cursor: not-allowed;
            opacity: 0.7;
        }

        .camera-controls {
            display: flex;
            gap: 10px;
            margin-top: 8px;
        }

        .camera-controls button {
            margin-top: 0;
        }

        .gps-status {
            font-size: 0.85rem;
            padding: 8px 12px;
            border-radius: 6px;
            background: #0f172a;
            border: 1px solid var(--border-color);
            margin-bottom: 15px;
            display: flex;
            justify-content: space-between;
            align-items: center;
        }

        .summary-grid {
            display: grid;
            grid-template-columns: repeat(3, 1fr);
            gap: 10px;
            text-align: center;
        }

        .stat-box {
            padding: 12px;
            border-radius: 8px;
            background: #0f172a;
            border: 1px solid var(--border-color);
        }

        .stat-box.total { border-bottom: 4px solid var(--primary); }
        .stat-box.normal { border-bottom: 4px solid var(--success); }
        .stat-box.late { border-bottom: 4px solid var(--danger); }

        .stat-number { 
            font-size: 1.5rem; 
            font-weight: bold; 
            margin-top: 4px;
        }

        .history-item {
            display: flex;
            gap: 12px;
            padding: 12px 0;
            border-bottom: 1px solid var(--border-color);
            align-items: center;
        }

        .history-item:last-child {
            border-bottom: none;
        }

        .history-thumb {
            width: 50px;
            height: 50px;
            border-radius: 6px;
            object-fit: cover;
            border: 1px solid var(--border-color);
        }

        .badge {
            padding: 4px 8px;
            border-radius: 12px;
            font-size: 0.75rem;
            color: white;
            font-weight: bold;
        }
        .badge.normal { background: var(--success); }
        .badge.late { background: var(--danger); }

        .export-btn {
            background: #059669;
        }
        .export-btn:hover {
            background: #047857;
        }

        .restricted-msg {
            text-align: center;
            padding: 24px 12px;
            background: #0f172a;
            border-radius: 8px;
            border: 1px dashed var(--border-color);
            color: var(--text-sub);
            font-size: 0.9rem;
        }
    </style>
</head>
<body>

<!-- Login Modal -->
<div class="modal-overlay" id="loginModal">
    <div class="login-card">
        <h2 style="justify-content: center;">🔒 เข้าสู่ระบบสำหรับครูเวร</h2>
        <p style="font-size: 0.85rem; color: var(--text-sub); margin-bottom: 20px;">
            กรุณากรอกชื่อและรหัสผ่านเพื่อเข้าสู่ระบบจัดการ
        </p>
        
        <div style="margin-bottom: 12px; text-align: left;">
            <label for="teacherNameInput">ชื่อ-นามสกุลครูเวร</label>
            <input type="text" id="teacherNameInput" placeholder="เช่น ครูสมชาย ใจดี">
        </div>

        <div style="margin-bottom: 15px; text-align: left;">
            <label for="teacherPassword">รหัสผ่านครูเวร</label>
            <div class="password-input-group">
                <input type="password" id="teacherPassword" placeholder="ระบุรหัสผ่าน">
                <button type="button" class="toggle-password-btn" id="togglePasswordBtn" title="แสดง/ซ่อนรหัสผ่าน">👁️</button>
            </div>
        </div>

        <button type="button" id="loginBtn">เข้าสู่ระบบ</button>
        <button type="button" id="guestBtn" style="background: transparent; border: 1px solid var(--border-color); margin-top: 8px; color: var(--text-sub);">
            ใช้งานโหมดทั่วไป (นักเรียนเช็คชื่อ)
        </button>
    </div>
</div>

<div class="container">
    
    <!-- Top Bar Status -->
    <div class="top-bar">
        <span class="user-status-badge" id="userStatusBadge">👤 โหมดทั่วไป (นักเรียน)</span>
        <div style="display: flex; align-items: center; gap: 10px;">
            <div class="sync-indicator">
                <span class="sync-dot" id="syncDot"></span>
                <span id="syncText">ซิงค์ข้อมูลแล้ว</span>
            </div>
            <button type="button" id="authToggleBtn" style="width: auto; padding: 4px 12px; margin: 0; font-size: 0.8rem; background: #334155;">
                🔒 ล็อกอินครู
            </button>
        </div>
    </div>

    <!-- Header & Live Timeline -->
    <div class="card">
        <h2>⏱️ ไทม์ไลน์เวลาปัจจุบัน</h2>
        
        <div class="datetime-box">
            <div class="date-display" id="dateDisplay">วัน-... ที่ - ... พ.ศ. ....</div>
            <div class="clock-display" id="clockDisplay">00:00:00 น.</div>
        </div>
        
        <div class="timeline-container">
            <div class="timeline-deadline">08:00 น. (เส้นแบ่งสาย)</div>
            <div class="timeline-pointer" id="timePointer"></div>
        </div>
        <div style="display: flex; justify-content: space-between; font-size: 0.75rem; color: var(--text-sub);">
            <span style="color: var(--success); font-weight: bold;">🟢 06:00 น. (ปกติ)</span>
            <span style="color: var(--danger); font-weight: bold;">🔴 10:00 น. (สาย)</span>
        </div>
    </div>

    <!-- Form Check-in -->
    <div class="card">
        <h2>📝 บันทึกเข้าแถว</h2>
        
        <div class="gps-status">
            <span>📍 พิกัดปัจจุบัน: <strong id="gpsText" style="color: #60a5fa;">กำลังระบุพิกัด...</strong></span>
            <button type="button" onclick="getLocation()" style="width: auto; padding: 4px 8px; margin: 0; font-size: 0.75rem; background: #475569;">รีเฟรช GPS</button>
        </div>

        <form id="checkinForm">
            <div class="form-row">
                <div class="form-group" style="flex: 1;">
                    <label for="checkDate">วันที่/เดือน/ปี ที่เช็ค</label>
                    <input type="date" id="checkDate" required>
                </div>
                <div class="form-group" style="flex: 1;">
                    <label for="studentId">รหัสนักเรียน</label>
                    <input type="text" id="studentId" placeholder="เช่น 12345" required>
                </div>
            </div>

            <div class="form-row">
                <div class="form-group" style="flex: 1.2;">
                    <label for="fullname">ชื่อ-นามสกุล</label>
                    <input type="text" id="fullname" placeholder="ระบุชื่อและนามสกุล" required>
                </div>
                <div class="form-group" style="flex: 0.8;">
                    <label for="nickname">ชื่อเล่น</label>
                    <input type="text" id="nickname" placeholder="เช่น ต้น" required>
                </div>
            </div>

            <div class="form-row">
                <div class="form-group" style="flex: 1;">
                    <label for="grade">ชั้น</label>
                    <input type="text" id="grade" placeholder="เช่น ม.1 หรือ ป.6" required>
                </div>
                <div class="form-group" style="flex: 1;">
                    <label for="room">ห้อง</label>
                    <input type="text" id="room" placeholder="เช่น 1" required>
                </div>
            </div>

            <div class="form-group full-width">
                <label>ถ่ายรูปตนเอง</label>
                <div class="camera-box">
                    <video id="webcam" autoplay playsinline></video>
                    <canvas id="canvas" style="display:none;"></canvas>
                    <img id="photoPreview" style="display:none;" alt="รูปถ่าย">
                </div>
                <div class="camera-controls">
                    <button type="button" id="snapBtn" style="flex: 2; background:#475569;">📸 ถ่ายรูป</button>
                    <button type="button" id="switchCamBtn" style="flex: 1; background:#334155;">🔄 สลับกล้อง</button>
                </div>
            </div>

            <button type="submit" id="submitBtn">ส่งข้อมูลเช็คชื่อ</button>
        </form>
    </div>

    <!-- Dashboard Summary -->
    <div class="card">
        <h2>📊 สรุปจำนวนการมา</h2>
        <div class="summary-grid">
            <div class="stat-box total">
                <div style="color: var(--text-sub);">ทั้งหมด</div>
                <div class="stat-number" id="countTotal" style="color: var(--primary);">0</div>
            </div>
            <div class="stat-box normal">
                <div style="color: var(--text-sub);">ปกติ</div>
                <div class="stat-number" id="countNormal" style="color: var(--success);">0</div>
            </div>
            <div class="stat-box late">
                <div style="color: var(--text-sub);">สาย</div>
                <div class="stat-number" id="countLate" style="color: var(--danger);">0</div>
            </div>
        </div>
    </div>

    <!-- History Log (Teacher Only View) -->
    <div class="card">
        <div style="display: flex; justify-content: space-between; align-items: center; margin-bottom: 12px;">
            <h2 style="margin: 0;">📜 ประวัติการเช็คชื่อ</h2>
            <button type="button" class="export-btn" id="exportCsvBtn" style="width: auto; padding: 6px 12px; margin: 0; font-size: 0.85rem; display: none;">🟢 ส่งออก CSV / Excel</button>
        </div>
        
        <!-- ข้อความแจ้งเตือนกรณีนักเรียนใช้งาน -->
        <div id="restrictedNotice" class="restricted-msg">
            🔒 เฉพาะ <strong>"ครูเวร"</strong> เท่านั้นที่สามารถดูรายการและประวัติการเช็คชื่อได้<br>
            <span style="font-size: 0.75rem; color: #64748b; display: inline-block; margin-top: 6px;">(กดปุ่ม "ล็อกอินครู" มุมขวาบนเพื่อเข้าสู่ระบบ)</span>
        </div>

        <!-- รายการประวัติ (แสดงเฉพาะเมื่อล็อกอิน) -->
        <div id="historyList" style="display: none;">
            <!-- ข้อมูลประวัติจะมาแสดงตรงนี้ -->
        </div>
    </div>

</div>

<script>
    // ⚠️ นำ Web App URL ที่ได้จาก Google Apps Script มาใส่ในช่องนี้
    const GOOGLE_SHEET_URL = "YOUR_GOOGLE_APPS_SCRIPT_WEB_APP_URL"; 
    const TEACHER_PASS = "111"; 

    let isLoggedInTeacher = false;
    let currentTeacherName = "";
    let capturedImageData = null;
    let records = [];
    let currentGpsCoords = "ไม่ทราบพิกัด";
    let isFetching = false;
    
    let currentStream = null;
    let currentFacingMode = 'user';

    const loginModal = document.getElementById('loginModal');
    const teacherNameInput = document.getElementById('teacherNameInput');
    const teacherPassword = document.getElementById('teacherPassword');
    const togglePasswordBtn = document.getElementById('togglePasswordBtn');
    const loginBtn = document.getElementById('loginBtn');
    const guestBtn = document.getElementById('guestBtn');
    const authToggleBtn = document.getElementById('authToggleBtn');
    const userStatusBadge = document.getElementById('userStatusBadge');
    const exportCsvBtn = document.getElementById('exportCsvBtn');
    const restrictedNotice = document.getElementById('restrictedNotice');
    const historyList = document.getElementById('historyList');
    const submitBtn = document.getElementById('submitBtn');

    const syncDot = document.getElementById('syncDot');
    const syncText = document.getElementById('syncText');

    const video = document.getElementById('webcam');
    const canvas = document.getElementById('canvas');
    const photoPreview = document.getElementById('photoPreview');
    const snapBtn = document.getElementById('snapBtn');
    const switchCamBtn = document.getElementById('switchCamBtn');
    const timePointer = document.getElementById('timePointer');

    const thaiDays = ['อาทิตย์', 'จันทร์', 'อังคาร', 'พุธ', 'พฤหัสบดี', 'ศุกร์', 'เสาร์'];
    const thaiMonths = [
        'มกราคม', 'กุมภาพันธ์', 'มีนาคม', 'เมษายน', 'พฤษภาคม', 'มิถุนายน',
        'กรกฎาคม', 'สิงหาคม', 'กันยายน', 'ตุลาคม', 'พฤศจิกายน', 'ธันวาคม'
    ];

    document.getElementById('checkDate').valueAsDate = new Date();

    // Toggle Password Visibility
    togglePasswordBtn.addEventListener('click', () => {
        const type = teacherPassword.getAttribute('type') === 'password' ? 'text' : 'password';
        teacherPassword.setAttribute('type', type);
        togglePasswordBtn.innerText = type === 'password' ? '👁️' : '🙈';
    });

    // LocalStorage Fallback สำหรับบันทึกข้อมูลในเครื่องเมื่อไม่มีสัญญาณเน็ต
    function loadLocalRecords() {
        const saved = localStorage.getItem('attendance_records');
        if (saved) {
            try {
                records = JSON.parse(saved);
            } catch (e) {
                records = [];
            }
        }
    }

    function saveLocalRecords() {
        localStorage.setItem('attendance_records', JSON.stringify(records));
    }

    // Real-time Data Polling Function (ดึงข้อมูลล่าสุดอัตโนมัติ)
    async function fetchLatestRecords() {
        if (isFetching) return;
        
        if (GOOGLE_SHEET_URL && GOOGLE_SHEET_URL !== "YOUR_GOOGLE_APPS_SCRIPT_WEB_APP_URL") {
            try {
                isFetching = true;
                syncDot.classList.add('syncing');
                syncText.innerText = 'กำลังซิงค์...';

                const res = await fetch(GOOGLE_SHEET_URL + "?action=getRecords");
                if (res.ok) {
                    const data = await res.json();
                    if (Array.isArray(data)) {
                        records = data;
                        saveLocalRecords();
                    }
                }
                syncDot.classList.remove('syncing');
                syncText.innerText = 'ซิงค์เรียบร้อย';
            } catch (err) {
                console.warn('Realtime fetch failed, falling back to local records:', err);
                syncDot.classList.remove('syncing');
                syncText.innerText = 'โหมดออฟไลน์';
            } finally {
                isFetching = false;
            }
        } else {
            loadLocalRecords();
            syncText.innerText = 'บันทึกในเครื่อง';
        }

        updateDashboard();
        if (isLoggedInTeacher) {
            renderHistory();
        }
    }

    // ตั้งค่า Polling ดึงข้อมูลใหม่ทุกๆ 5 วินาที เพื่ออัปเดตแบบ Real-time
    setInterval(fetchLatestRecords, 5000);
    loadLocalRecords();
    fetchLatestRecords();

    // System Auth UI Controller
    function updateAuthUI() {
        if (isLoggedInTeacher) {
            const displayName = currentTeacherName ? `ครู${currentTeacherName}` : 'ครูเวร';
            userStatusBadge.innerText = `👨‍🏫 ${displayName} (สิทธิ์จัดการ)`;
            userStatusBadge.classList.add('admin');
            authToggleBtn.innerText = "🚪 ออกจากระบบ";
            authToggleBtn.style.background = "#ef4444";
            
            exportCsvBtn.style.display = "block";
            restrictedNotice.style.display = "none";
            historyList.style.display = "block";
            renderHistory();
        } else {
            userStatusBadge.innerText = "👤 โหมดทั่วไป (นักเรียน)";
            userStatusBadge.classList.remove('admin');
            authToggleBtn.innerText = "🔒 ล็อกอินครู";
            authToggleBtn.style.background = "#334155";
            
            exportCsvBtn.style.display = "none";
            restrictedNotice.style.display = "block";
            historyList.style.display = "none";
        }
    }

    loginBtn.addEventListener('click', () => {
        const inputName = teacherNameInput.value.trim();
        
        if (!inputName) {
            alert('กรุณากรอกชื่อ-นามสกุลครูเวร');
            return;
        }

        if (teacherPassword.value === TEACHER_PASS) {
            isLoggedInTeacher = true;
            currentTeacherName = inputName;
            loginModal.style.display = 'none';
            teacherPassword.value = '';
            updateAuthUI();
            alert(`เข้าสู่ระบบเรียบร้อยแล้ว\nสวัสดีครับ/ค่ะ ${currentTeacherName}`);
        } else {
            alert('รหัสผ่านไม่ถูกต้อง');
        }
    });

    guestBtn.addEventListener('click', () => {
        isLoggedInTeacher = false;
        currentTeacherName = "";
        loginModal.style.display = 'none';
        updateAuthUI();
    });

    authToggleBtn.addEventListener('click', () => {
        if (isLoggedInTeacher) {
            isLoggedInTeacher = false;
            currentTeacherName = "";
            updateAuthUI();
            alert('ออกจากระบบเรียบร้อยแล้ว');
        } else {
            loginModal.style.display = 'flex';
        }
    });

    function formatThaiDate(dateString) {
        if (!dateString) return "-";
        const parts = dateString.split('-');
        if (parts.length !== 3) return dateString;
        const yearBE = parseInt(parts[0], 10) + 543;
        return `${parts[2]}/${parts[1]}/${yearBE}`;
    }

    // 1. Clock & Timeline Logic
    function updateClock() {
        const now = new Date();
        const dayName = thaiDays[now.getDay()];
        const dayDate = now.getDate();
        const monthName = thaiMonths[now.getMonth()];
        const yearBE = now.getFullYear() + 543;
        
        document.getElementById('dateDisplay').innerText = `วัน${dayName}ที่ ${dayDate} ${monthName} พ.ศ. ${yearBE}`;

        const hours = now.getHours();
        const minutes = now.getMinutes();
        const seconds = now.getSeconds();
        
        document.getElementById('clockDisplay').innerText = 
            `${String(hours).padStart(2, '0')}:${String(minutes).padStart(2, '0')}:${String(seconds).padStart(2, '0')} น.`;

        const startHour = 6;
        const endHour = 10;
        const currentTotalMinutes = (hours * 60) + minutes;
        const startMinutes = startHour * 60;
        const endMinutes = endHour * 60;

        let percentage = ((currentTotalMinutes - startMinutes) / (endMinutes - startMinutes)) * 100;
        if (percentage < 0) percentage = 0;
        if (percentage > 100) percentage = 100;

        timePointer.style.left = `${percentage}%`;
    }
    setInterval(updateClock, 1000);
    updateClock();

    // 2. Fetch GPS Location
    function getLocation() {
        const gpsText = document.getElementById('gpsText');
        gpsText.innerText = 'กำลังระบุพิกัด...';

        if (navigator.geolocation) {
            navigator.geolocation.getCurrentPosition(
                (position) => {
                    const lat = position.coords.latitude.toFixed(5);
                    const lng = position.coords.longitude.toFixed(5);
                    currentGpsCoords = `${lat}, ${lng}`;
                    gpsText.innerText = currentGpsCoords;
                },
                (error) => {
                    currentGpsCoords = "ปิดการใช้งาน GPS";
                    gpsText.innerText = currentGpsCoords;
                },
                { enableHighAccuracy: true }
            );
        } else {
            currentGpsCoords = "ไม่รองรับ GPS";
            gpsText.innerText = currentGpsCoords;
        }
    }
    getLocation();

    // 3. Camera Setup
    async function initCamera(facingMode) {
        if (currentStream) {
            currentStream.getTracks().forEach(track => track.stop());
        }

        try {
            const stream = await navigator.mediaDevices.getUserMedia({ 
                video: { facingMode: facingMode }, 
                audio: false 
            });
            currentStream = stream;
            video.srcObject = stream;
        } catch (err) {
            alert('ไม่สามารถเปิดกล้องได้ โปรดอนุญาตสิทธิ์การเข้าถึงกล้อง');
        }
    }

    initCamera(currentFacingMode);

    switchCamBtn.addEventListener('click', () => {
        currentFacingMode = (currentFacingMode === 'user') ? 'environment' : 'user';
        if (video.style.display === 'none') {
            photoPreview.style.display = 'none';
            video.style.display = 'block';
            capturedImageData = null;
            snapBtn.innerText = '📸 ถ่ายรูป';
        }
        initCamera(currentFacingMode);
    });

    snapBtn.addEventListener('click', () => {
        if (video.style.display !== 'none') {
            canvas.width = video.videoWidth;
            canvas.height = video.videoHeight;
            canvas.getContext('2d').drawImage(video, 0, 0);
            capturedImageData = canvas.toDataURL('image/png');
            
            photoPreview.src = capturedImageData;
            photoPreview.style.display = 'block';
            video.style.display = 'none';
            snapBtn.innerText = '🔄 ถ่ายใหม่';
        } else {
            photoPreview.style.display = 'none';
            video.style.display = 'block';
            capturedImageData = null;
            snapBtn.innerText = '📸 ถ่ายรูป';
        }
    });

    // 4. Submit Form
    document.getElementById('checkinForm').addEventListener('submit', async (e) => {
        e.preventDefault();

        const checkDateRaw = document.getElementById('checkDate').value;
        const checkDateFormatted = formatThaiDate(checkDateRaw);
        const studentId = document.getElementById('studentId').value.trim();
        const nickname = document.getElementById('nickname').value.trim();
        const name = document.getElementById('fullname').value.trim();
        const grade = document.getElementById('grade').value.trim();
        const room = document.getElementById('room').value.trim();
        
        if (!capturedImageData) {
            alert('กรุณากดถ่ายรูปก่อนส่งข้อมูล');
            return;
        }

        submitBtn.disabled = true;
        submitBtn.innerText = '⏳ กำลังบันทึกข้อมูล...';

        const now = new Date();
        const timeStr = `${String(now.getHours()).padStart(2, '0')}:${String(now.getMinutes()).padStart(2, '0')}:${String(now.getSeconds()).padStart(2, '0')}`;
        
        const isLate = (now.getHours() > 8) || (now.getHours() === 8 && (now.getMinutes() > 0 || now.getSeconds() > 0));
        const status = isLate ? 'สาย' : 'ปกติ';

        const record = {
            checkDate: checkDateFormatted,
            studentId: studentId,
            nickname: nickname,
            name: name,
            grade: grade,
            room: room,
            time: timeStr,
            status: status,
            gps: currentGpsCoords,
            photo: capturedImageData
        };

        // เพิ่มข้อมูลลงในเครื่องก่อนเพื่อให้แสดงผลทันที
        records.unshift(record);
        saveLocalRecords();
        updateDashboard();
        
        if (isLoggedInTeacher) {
            renderHistory();
        }

        // ส่งข้อมูลไปยัง Google Sheet / Web API Cloud
        if (GOOGLE_SHEET_URL && GOOGLE_SHEET_URL !== "YOUR_GOOGLE_APPS_SCRIPT_WEB_APP_URL") {
            await sendToGoogleSheet(record);
        }

        document.getElementById('studentId').value = '';
        document.getElementById('nickname').value = '';
        document.getElementById('fullname').value = '';
        
        submitBtn.disabled = false;
        submitBtn.innerText = 'ส่งข้อมูลเช็คชื่อ';

        snapBtn.click();
        
        alert(`เช็คชื่อสำเร็จ!\nวันที่: ${checkDateFormatted}\nนักเรียน: ${name} (${nickname})\nชั้น ${grade} ห้อง ${room}\nเวลา: ${timeStr} น. [${status}]`);
    });

    async function sendToGoogleSheet(recordData) {
        try {
            syncDot.classList.add('syncing');
            syncText.innerText = 'กำลังส่งข้อมูล...';
            
            await fetch(GOOGLE_SHEET_URL, {
                method: 'POST',
                headers: { 'Content-Type': 'text/plain;charset=utf-8' },
                body: JSON.stringify(recordData)
            });

            syncDot.classList.remove('syncing');
            syncText.innerText = 'ซิงค์เรียบร้อย';
            fetchLatestRecords();
        } catch (err) {
            console.error('Error sending to Google Sheet:', err);
            syncDot.classList.remove('syncing');
            syncText.innerText = 'บันทึกในเครื่อง';
        }
    }

    function updateDashboard() {
        const total = records.length;
        const normal = records.filter(r => r.status === 'ปกติ').length;
        const late = records.filter(r => r.status === 'สาย').length;

        document.getElementById('countTotal').innerText = total;
        document.getElementById('countNormal').innerText = normal;
        document.getElementById('countLate').innerText = late;
    }

    function renderHistory() {
        historyList.innerHTML = '';

        if (records.length === 0) {
            historyList.innerHTML = '<div style="text-align:center; color: var(--text-sub); padding: 20px;">ยังไม่มีประวัติการเช็คชื่อในวันนี้</div>';
            return;
        }

        records.forEach((rec) => {
            const item = document.createElement('div');
            item.className = 'history-item';

            const badgeClass = rec.status === 'สาย' ? 'late' : 'normal';
            const imgSrc = rec.photo || 'https://via.placeholder.com/50';

            item.innerHTML = `
                <img src="${imgSrc}" class="history-thumb" alt="รูปถ่ายนักเรียน">
                <div style="flex: 1;">
                    <div style="font-weight: bold; font-size: 0.95rem;">${rec.name} (${rec.nickname})</div>
                    <div style="font-size: 0.8rem; color: var(--text-sub);">
                        รหัส: ${rec.studentId} | ชั้น ${rec.grade}/${rec.room}
                    </div>
                    <div style="font-size: 0.75rem; color: var(--text-sub);">
                        📅 ${rec.checkDate} | ⏰ ${rec.time} น. | 📍 ${rec.gps}
                    </div>
                </div>
                <div>
                    <span class="badge ${badgeClass}">${rec.status}</span>
                </div>
            `;
            historyList.appendChild(item);
        });
    }

    // Export CSV Functions
    exportCsvBtn.addEventListener('click', () => {
        if (records.length === 0) {
            alert('ไม่มีข้อมูลสำหรับส่งออก');
            return;
        }

        let csvContent = "\uFEFF"; // BOM สำหรับภาษาไทยใน Excel
        csvContent += "วันที่,รหัสนักเรียน,ชื่อ-นามสกุล,ชื่อเล่น,ชั้น,ห้อง,เวลาเข้าแถว,สถานะ,พิกัด GPS\n";

        records.forEach(r => {
            const row = [
                `"${r.checkDate || ''}"`,
                `"${r.studentId || ''}"`,
                `"${r.name || ''}"`,
                `"${r.nickname || ''}"`,
                `"${r.grade || ''}"`,
                `"${r.room || ''}"`,
                `"${r.time || ''}"`,
                `"${r.status || ''}"`,
                `"${r.gps || ''}"`
            ];
            csvContent += row.join(",") + "\n";
        });

        const blob = new Blob([csvContent], { type: 'text/csv;charset=utf-8;' });
        const url = URL.createObjectURL(blob);
        const link = document.createElement("a");
        link.setAttribute("href", url);
        link.setAttribute("download", `รายงานการเข้าแถว_${new Date().toISOString().slice(0,10)}.csv`);
        document.body.appendChild(link);
        link.click();
        document.body.removeChild(link);
    });
</script>
</body>
</html>
