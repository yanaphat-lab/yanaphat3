<!DOCTYPE html>
<html lang="th">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>ระบบเช็คชื่อนักเรียน</title>
    <style>
        body {
            font-family: 'Sarabun', sans-serif;
            background-color: #f4f6f9;
            margin: 0;
            padding: 20px;
        }
        .container {
            max-width: 600px;
            margin: 0 auto;
            background: #ffffff;
            padding: 25px;
            border-radius: 12px;
            box-shadow: 0 4px 10px rgba(0,0,0,0.1);
        }
        h2 { text-align: center; color: #333; }
        .form-group { margin-bottom: 15px; }
        label { display: block; margin-bottom: 5px; font-weight: bold; }
        input, select {
            width: 100%;
            padding: 10px;
            border: 1px solid #ccc;
            border-radius: 6px;
            box-sizing: border-box;
        }
        button {
            width: 100%;
            padding: 12px;
            background-color: #28a745;
            color: white;
            border: none;
            border-radius: 6px;
            font-size: 16px;
            cursor: pointer;
            margin-top: 10px;
        }
        button:hover { background-color: #218838; }
        .data-list { margin-top: 30px; }
        table {
            width: 100%;
            border-collapse: collapse;
            margin-top: 10px;
        }
        th, td {
            border: 1px solid #ddd;
            padding: 8px;
            text-align: left;
            font-size: 14px;
        }
        th { background-color: #007bff; color: white; }
        .loading { text-align: center; color: #666; font-style: italic; }
    </style>
</head>
<body>

<div class="container">
    <h2>บันทึกข้อมูลการเช็คชื่อ</h2>
    <form id="checkinForm">
        <div class="form-group">
            <label>รหัสนักเรียน:</label>
            <input type="text" id="studentId" required>
        </div>
        <div class="form-group">
            <label>ชื่อ-นามสกุล:</label>
            <input type="text" id="name" required>
        </div>
        <div class="form-group">
            <label>ชื่อเล่น:</label>
            <input type="text" id="nickname">
        </div>
        <div class="form-group">
            <label>ชั้นเรียน / ห้อง:</label>
            <input type="text" id="gradeRoom" placeholder="เช่น ม.3/1" required>
        </div>
        <div class="form-group">
            <label>สถานะ:</label>
            <select id="status">
                <option value="มา">มา</option>
                <option value="สาย">สาย</option>
                <option value="ลา">ลา</option>
                <option value="ขาด">ขาด</option>
            </select>
        </div>
        <button type="submit" id="submitBtn">บันทึกข้อมูล</button>
    </form>

    <div class="data-list">
        <h3>ประวัติการเช็คชื่อ (ซิงค์เรียลไทม์)</h3>
        <p id="statusMsg" class="loading">กำลังโหลดข้อมูลจาก Google Sheets...</p>
        <table id="recordsTable" style="display:none;">
            <thead>
                <tr>
                    <th>วันที่/เวลา</th>
                    <th>รหัส</th>
                    <th>ชื่อ</th>
                    <th>ห้อง</th>
                    <th>สถานะ</th>
                </tr>
            </thead>
            <tbody id="tableBody"></tbody>
        </table>
    </div>
</div>

<script>
    // ⚠️ นำ Web App URL ที่ได้จาก Google Apps Script มาวางแทนที่บรรทัดนี้
    const GOOGLE_SHEET_URL = "YOUR_GOOGLE_APPS_SCRIPT_WEB_APP_URL";

    // ดึงข้อมูลเมื่อโหลดหน้าเว็บ
    window.onload = function() {
        fetchRecords();
    };

    // ฟังก์ชันดึงข้อมูลจาก Google Sheets
    function fetchRecords() {
        if (GOOGLE_SHEET_URL === "YOUR_GOOGLE_APPS_SCRIPT_WEB_APP_URL") {
            document.getElementById('statusMsg').innerText = "กรุณาใส่ Web App URL ในโค้ด HTML บรรทัดที่ 103 ก่อนนะครับ";
            return;
        }

        fetch(`${GOOGLE_SHEET_URL}?action=getRecords`)
            .then(res => res.json())
            .then(data => {
                const tbody = document.getElementById('tableBody');
                tbody.innerHTML = '';
                
                if (data.length === 0) {
                    document.getElementById('statusMsg').innerText = "ยังไม่มีข้อมูลในระบบ";
                    return;
                }

                data.forEach(item => {
                    const tr = document.createElement('tr');
                    tr.innerHTML = `
                        <td>${item.checkDate || ''} ${item.time || ''}</td>
                        <td>${item.studentId || ''}</td>
                        <td>${item.name || ''}</td>
                        <td>${item.grade || ''}/${item.room || ''}</td>
                        <td>${item.status || ''}</td>
                    `;
                    tbody.appendChild(tr);
                });

                document.getElementById('statusMsg').style.display = 'none';
                document.getElementById('recordsTable').style.display = 'table';
            })
            .catch(err => {
                document.getElementById('statusMsg').innerText = "เกิดข้อผิดพลาดในการดึงข้อมูล";
                console.error(err);
            });
    }

    // ฟังก์ชันบันทึกข้อมูลส่งไปยัง Google Sheets
    document.getElementById('checkinForm').addEventListener('submit', function(e) {
        e.preventDefault();
        
        const submitBtn = document.getElementById('submitBtn');
        submitBtn.disabled = true;
        submitBtn.innerText = "กำลังบันทึก...";

        const now = new Date();
        const dateStr = now.toLocaleDateString('th-TH');
        const timeStr = now.toLocaleTimeString('th-TH');

        const gradeRoomVal = document.getElementById('gradeRoom').value.split('/');

        const payload = {
            checkDate: dateStr,
            studentId: document.getElementById('studentId').value,
            name: document.getElementById('name').value,
            nickname: document.getElementById('nickname').value,
            grade: gradeRoomVal[0] || '',
            room: gradeRoomVal[1] || '',
            time: timeStr,
            status: document.getElementById('status').value,
            gps: '',
            photo: ''
        };

        fetch(GOOGLE_SHEET_URL, {
            method: 'POST',
            body: JSON.stringify(payload)
        })
        .then(res => res.json())
        .then(res => {
            alert("บันทึกข้อมูลเรียบร้อยแล้ว!");
            document.getElementById('checkinForm').reset();
            submitBtn.disabled = false;
            submitBtn.innerText = "บันทึกข้อมูล";
            fetchRecords(); // โหลดรายการใหม่ทันที
        })
        .catch(err => {
            alert("เกิดข้อผิดพลาดในการบันทึก");
            console.error(err);
            submitBtn.disabled = false;
            submitBtn.innerText = "บันทึกข้อมูล";
        });
    });
</script>

</body>
</html>
