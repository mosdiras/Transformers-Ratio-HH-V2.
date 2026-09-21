ทีมงานหม้อแปลง ผมต.หห
<html lang="th">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>เครื่องมือคำนวณและประเมินผล TTR Ratio</title>
    <style>
        * {
            box-sizing: border-box;
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
        }
        body {
            background-color: #f4f7f6;
            display: flex;
            justify-content: center;
            align-items: center;
            min-height: 100vh;
            margin: 0;
            padding: 15px;
        }
        .card {
            background: #ffffff;
            padding: 25px;
            border-radius: 12px;
            box-shadow: 0 4px 15px rgba(0,0,0,0.1);
            width: 100%;
            max-width: 500px;
        }
        h2 {
            margin-top: 0;
            color: #2c3e50;
            text-align: center;
            font-size: 1.4rem;
        }
        .form-group {
            margin-bottom: 12px;
        }
        label {
            display: block;
            margin-bottom: 5px;
            color: #444;
            font-weight: 600;
            font-size: 0.9rem;
        }
        input, select {
            width: 100%;
            padding: 10px;
            border: 1px solid #ccc;
            border-radius: 6px;
            font-size: 0.95rem;
        }
        button {
            width: 100%;
            padding: 12px;
            background-color: #007bff;
            color: white;
            border: none;
            border-radius: 6px;
            font-size: 1rem;
            font-weight: bold;
            cursor: pointer;
            margin-top: 10px;
            transition: background 0.2s;
        }
        button:hover {
            background-color: #0056b3;
        }
        .result-box {
            margin-top: 18px;
            padding: 15px;
            border-radius: 8px;
            display: none;
            border: 1px solid transparent;
        }
        /* Style แต่ละสถานะ */
        .status-excellent { background-color: #e8f5e9; color: #1b5e20; border-color: #a5d6a7; }
        .status-good { background-color: #f1f8e9; color: #33691e; border-color: #c5e1a5; }
        .status-warning { background-color: #fffde7; color: #f57f17; border-color: #fff59d; }
        .status-error { background-color: #ffebee; color: #c62828; border-color: #ef9a9a; }
        .status-critical { background-color: #b71c1c; color: #ffffff; border-color: #880e4f; }

        .result-item {
            margin-bottom: 8px;
            font-size: 0.95rem;
        }
        .status-tag {
            font-weight: bold;
            font-size: 1.1rem;
        }
    </style>
</head>
<body>

<div class="card">
    <h2>คำนวณ & ประเมินผล TTR Ratio</h2>
    
    <!-- ปุ่มเลือก Tap -->
    <div class="form-group">
        <label>เลือกตำแหน่ง Tap หม้อแปลง (H.V. Side):</label>
        <select id="tapSelect" onchange="updateHvVoltage()">
            <option value="23122">Tap 1 (23,122 V)</option>
            <option value="22550">Tap 2 (22,550 V)</option>
            <option value="22000" selected>Tap 3 (22,000 V - Nominal)</option>
            <option value="21450">Tap 4 (21,450 V)</option>
            <option value="20878">Tap 5 (20,878 V)</option>
            <option value="custom">กำหนดแรงดันเอง (Custom)</option>
        </select>
    </div>

    <div class="form-group">
        <label>แรงดันไฟฝั่งแรงสูง H.V. (Volts):</label>
        <input type="number" id="hvVolt" value="22000" step="any">
    </div>

    <div class="form-group">
        <label>แรงดันไฟฝั่งแรงต่ำ L.V. (Volts):</label>
        <input type="number" id="lvVolt" value="400" step="any">
    </div>

    <div class="form-group">
        <label>รูปแบบการต่อสายขดลวดแรงต่ำ (L.V. Connection):</label>
        <select id="lvConnection">
            <option value="star">Star / Dyn11 (คำนวณ V_LV / √3)</option>
            <option value="delta">Delta (คำนวณ V_LV ตรงๆ)</option>
        </select>
    </div>

    <hr style="border: 0; border-top: 1px solid #eee; margin: 15px 0;">

    <div class="form-group">
        <label>ค่า Ratio ที่วัดได้จริงจากเครื่อง (Measured Ratio):</label>
        <input type="number" id="measuredRatio" placeholder="เช่น 95.677" step="any">
    </div>

    <button onclick="calculateTTR()">วิเคราะห์และประเมินผล</button>

    <!-- กล่องแสดงผลลัพธ์ -->
    <div id="result" class="result-box">
        <div class="result-item">อัตราส่วนมาตรฐาน (Calculated Ratio): <strong id="resCalculated">-</strong></div>
        <div class="result-item">เปอร์เซ็นต์ความคลาดเคลื่อน (% Error): <strong id="resError">-</strong></div>
        <div class="result-item" style="margin-top: 10px;">
            สถานะ: <br><span id="resStatus" class="status-tag">-</span>
        </div>
        <div class="result-item" id="resAdvice" style="margin-top: 8px; font-size: 0.85rem;"></div>
    </div>
</div>

<script>
// ฟังก์ชันอัปเดตแรงดัน HV อัตโนมัติเมื่อเปลี่ยน Tap
function updateHvVoltage() {
    const tapSelect = document.getElementById('tapSelect');
    const hvVoltInput = document.getElementById('hvVolt');
    
    if (tapSelect.value !== 'custom') {
        hvVoltInput.value = tapSelect.value;
    }
}

function calculateTTR() {
    const hvVolt = parseFloat(document.getElementById('hvVolt').value);
    const lvVolt = parseFloat(document.getElementById('lvVolt').value);
    const lvConnection = document.getElementById('lvConnection').value;
    const measuredRatio = parseFloat(document.getElementById('measuredRatio').value);

    if (isNaN(hvVolt) || isNaN(lvVolt) || isNaN(measuredRatio)) {
        alert('กรุณากรอกข้อมูลตัวเลขให้ครบถ้วน');
        return;
    }

    // คำนวณ Phase Voltage ฝั่ง LV
    let lvPhaseVolt = lvVolt;
    if (lvConnection === 'star') {
        lvPhaseVolt = lvVolt / Math.sqrt(3);
    }

    // คำนวณ Calculated Ratio ตามทฤษฎี
    const calculatedRatio = hvVolt / lvPhaseVolt;

    // คำนวณ % Error
    const percentError = Math.abs((measuredRatio - calculatedRatio) / calculatedRatio) * 100;

    // แสดงผลตัวเลข
    document.getElementById('resCalculated').innerText = calculatedRatio.toFixed(3);
    document.getElementById('resError').innerText = percentError.toFixed(3) + ' %';

    const resultBox = document.getElementById('result');
    const statusSpan = document.getElementById('resStatus');
    const adviceDiv = document.getElementById('resAdvice');

    // ล้าง Class สไตล์เดิม
    resultBox.className = 'result-box';

    // ประเมินผลตามเกณฑ์คัดกรอง 🟢🟡🔴
    if (percentError <= 0.10) {
        resultBox.classList.add('status-excellent');
        statusSpan.innerText = '🟢 ดีมาก (≤ ±0.10%)';
        adviceDiv.innerText = 'ค่าสอดคล้องกับทฤษฎีสูงมาก ขดลวดและ Tap อยู่ในสภาพสมบูรณ์ที่สุด';
    } else if (percentError <= 0.30) {
        resultBox.classList.add('status-good');
        statusSpan.innerText = '🟢 ปกติ (> ±0.10 ถึง ±0.30%)';
        adviceDiv.innerText = 'ผ่านเกณฑ์มาตรฐาน สภาพปกติพร้อมใช้งาน';
    } else if (percentError <= 0.50) {
        resultBox.classList.add('status-warning');
        statusSpan.innerText = '🟡 ควรตรวจสอบ/ติดตาม (> ±0.30 ถึง ±0.50%)';
        adviceDiv.innerText = 'อยู่ในเกณฑ์ยอมรับได้ตามมาตรฐานสากล (ไม่เกิน 0.5%) แต่เริ่มมี Deviation ควรจดบันทึกเพื่อติดตามผลในการทดสอบครั้งถัดไป';
    } else if (percentError <= 3.00) {
        resultBox.classList.add('status-error');
        statusSpan.innerText = '🔴 ผิดเกณฑ์ ควรหาสาเหตุ (> ±0.50%)';
        adviceDiv.innerText = 'เกินเกณฑ์มาตรฐานสากล! แนะนำให้ตรวจสอบตำแหน่ง Tap Changer หน้างาน, คีบสายทดสอบใหม่ หรือเช็กจุดสัมผัสแน่นหรือไม่';
    } else {
        resultBox.classList.add('status-critical');
        statusSpan.innerText = '🔴🔴 ผิดปกติอย่างชัดเจน (> ±3.00%)';
        adviceDiv.innerText = 'วิกฤต! ค่าเบี่ยงเบนสูงมาก สันนิษฐานว่าตั้งค่า Tap ไม่ตรง, คีบสายสลับขั้ว/สลับเฟส หรือขดลวดหม้อแปลงชำรุด (Turn Short Circuit)';
    }

    resultBox.style.display = 'block';
}
</script>

</body>
</html>
# Transformers-Ratio-HH-V2.
