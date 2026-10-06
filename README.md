<!DOCTYPE html>
<html lang="zh-TW">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>PET 打藥劑量計算器</title>
    <!-- 引入 SheetJS 庫以支援匯出 Excel -->
    <script src="https://cdnjs.cloudflare.com/ajax/libs/xlsx/0.18.5/xlsx.full.min.js"></script>
    <style>
        * {
            box-sizing: border-box;
            margin: 0;
            padding: 0;
            font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, "Helvetica Neue", Arial, sans-serif;
        }
        body {
            background-color: #f0f4f8;
            min-height: 100vh;
            display: flex;
            justify-content: center;
            align-items: center;
            padding: 8px;
        }
        .calculator-card {
            background: #ffffff;
            border-radius: 12px;
            box-shadow: 0 4px 15px rgba(0, 0, 0, 0.1);
            width: 100%;
            max-width: 480px;
            padding: 12px 14px;
            display: flex;
            flex-direction: column;
            gap: 8px;
        }
        .header {
            display: flex;
            justify-content: space-between;
            align-items: center;
            border-bottom: 2px solid #edf2f7;
            padding-bottom: 6px;
        }
        .header h2 {
            font-size: 17px;
            color: #1a365d;
            font-weight: 700;
        }
        .counter-badge {
            background-color: #e2e8f0;
            color: #4a5568;
            font-size: 11px;
            padding: 2px 8px;
            border-radius: 10px;
            font-weight: 600;
        }
        
        /* 警報提示區塊 */
        .alert-box {
            padding: 6px 10px;
            border-radius: 6px;
            text-align: center;
            font-size: 13px;
            font-weight: bold;
            display: none;
        }
        .alert-too-much {
            background-color: #fff5f5;
            border: 1.5px solid #fc8181;
            color: #c53030;
        }
        .alert-too-less {
            background-color: #fffaf0;
            border: 1.5px solid #f6ad55;
            color: #dd6b20;
        }

        /* 表單區域 - 輸入格組合 */
        .input-grid {
            display: grid;
            grid-template-columns: 1fr 1fr 1fr;
            gap: 8px;
        }
        .input-group {
            display: flex;
            flex-direction: column;
            align-items: center;
        }
        .input-group label {
            font-size: 12px;
            font-weight: 700;
            margin-bottom: 3px;
            color: #2d3748;
        }
        .input-group input {
            width: 100%;
            height: 38px;
            text-align: center;
            font-size: 16px;
            font-weight: bold;
            border: 1.5px solid #cbd5e0;
            border-radius: 6px;
            background-color: #f8fafc;
            color: #1a202c;
            outline: none;
        }
        .input-group input:focus {
            border-color: #3182ce;
            background-color: #ffffff;
            box-shadow: 0 0 0 2px rgba(49, 130, 206, 0.2);
        }

        /* 結果顯示卡片區塊 */
        .results-grid {
            display: grid;
            grid-template-columns: 1fr 1fr;
            gap: 8px;
        }
        .res-card {
            border-radius: 8px;
            padding: 6px 8px;
            text-align: center;
            display: flex;
            flex-direction: column;
            justify-content: center;
        }
        .res-card .title {
            font-size: 11px;
            font-weight: 600;
            margin-bottom: 2px;
        }
        .res-card .value {
            font-size: 18px;
            font-weight: 800;
        }
        
        .card-orange { background-color: #feebc8; color: #7b341e; }
        .card-green  { background-color: #c6f6d5; color: #22543d; }
        .card-range  { background-color: #fff5f5; color: #9b2c2c; border: 1px dashed #feb2b2; }

        .range-container {
            display: flex;
            justify-content: space-around;
            margin-top: 2px;
        }
        .range-sub {
            display: flex;
            flex-direction: column;
        }
        .range-sub span.sub-title {
            font-size: 10px;
            color: #742a2a;
        }
        .range-sub span.sub-val {
            font-size: 15px;
            font-weight: 700;
            color: #2b6cb0;
        }

        /* 按鈕區域 */
        .btn-group {
            display: grid;
            grid-template-columns: 1fr 1fr;
            gap: 8px;
            margin-top: 2px;
        }
        button {
            height: 38px;
            border: none;
            border-radius: 6px;
            font-size: 13px;
            font-weight: 700;
            color: white;
            cursor: pointer;
            transition: active 0.1s;
        }
        button:active {
            opacity: 0.85;
        }
        .btn-excel { background-color: #38a169; }
        .btn-reset { background-color: #718096; }
    </style>
</head>
<body>

<div class="calculator-card">
    <div class="header">
        <h2>PET 打藥劑量計算器</h2>
        <div class="counter-badge" id="recordCounter">紀錄：0 筆</div>
    </div>

    <div id="alertBox" class="alert-box"></div>

    <div class="input-grid">
        <div class="input-group">
            <label style="color:#2b6cb0;">體重 (kg)</label>
            <input type="number" id="weight" step="any" placeholder="0" oninput="calculate()">
        </div>
        <div class="input-group">
            <label style="color:#d69e2e;">打藥前</label>
            <input type="number" id="beforeDose" step="any" placeholder="0" oninput="calculate()">
        </div>
        <div class="input-group">
            <label style="color:#d69e2e;">打藥後</label>
            <input type="number" id="afterDose" step="any" placeholder="0" oninput="calculate()">
        </div>
    </div>

    <div class="results-grid">
        <div class="res-card card-orange">
            <div class="title">預打藥量 (體重×0.14)</div>
            <div class="value" id="predose">-</div>
        </div>
        <div class="res-card card-green">
            <div class="title">實打藥量 (前 - 後)</div>
            <div class="value" id="actualDose">-</div>
        </div>
    </div>

    <div class="res-card card-range">
        <div class="title">安全劑量建議範圍</div>
        <div class="range-container">
            <div class="range-sub">
                <span class="sub-title">-10% 下限</span>
                <span class="sub-val" id="minus10">-</span>
            </div>
            <div class="range-sub">
                <span class="sub-title">+20% 上限</span>
                <span class="sub-val" id="plus20">-</span>
            </div>
        </div>
    </div>

    <div class="btn-group">
        <button class="btn-excel" onclick="exportToExcel()">📥 下載 Excel 總表</button>
        <button class="btn-reset" onclick="clearAll()">🔄 清除換下一位</button>
    </div>
</div>

<script>
    let multiRecords = [];
    let currentRecordId = null;

    function playBeep(isTooMuch) {
        try {
            const audioCtx = new (window.AudioContext || window.webkitAudioContext)();
            const oscillator = audioCtx.createOscillator();
            const gainNode = audioCtx.createGain();
            
            oscillator.type = 'sine';
            if (isTooMuch) {
                oscillator.frequency.setValueAtTime(800, audioCtx.currentTime);
                oscillator.frequency.setValueAtTime(1000, audioCtx.currentTime + 0.15);
            } else {
                oscillator.frequency.setValueAtTime(500, audioCtx.currentTime);
                oscillator.frequency.setValueAtTime(400, audioCtx.currentTime + 0.15);
            }
            
            gainNode.gain.setValueAtTime(0.3, audioCtx.currentTime);
            oscillator.connect(gainNode);
            gainNode.connect(audioCtx.destination);
            
            oscillator.start();
            oscillator.stop(audioCtx.currentTime + 0.4);
        } catch (e) {
            console.log("音效無法播放", e);
        }
    }

    function calculate() {
        const weight = parseFloat(document.getElementById('weight').value);
        const beforeDose = parseFloat(document.getElementById('beforeDose').value);
        const afterDose = parseFloat(document.getElementById('afterDose').value);

        const predoseElem = document.getElementById('predose');
        const actualDoseElem = document.getElementById('actualDose');
        const minus10Elem = document.getElementById('minus10');
        const plus20Elem = document.getElementById('plus20');
        const alertBox = document.getElementById('alertBox');

        let predose = NaN;
        let actualDose = NaN;

        if (!isNaN(weight)) {
            predose = weight * 0.14;
            predoseElem.innerText = predose.toFixed(2);
        } else {
            predoseElem.innerText = '-';
        }

        if (!isNaN(beforeDose) && !isNaN(afterDose)) {
            actualDose = beforeDose - afterDose;
            actualDoseElem.innerText = actualDose.toFixed(2);
        } else {
            actualDoseElem.innerText = '-';
        }

        if (!isNaN(predose)) {
            const m10 = predose * 0.9;
            const p20 = predose * 1.2;
            
            minus10Elem.innerText = m10.toFixed(2);
            plus20Elem.innerText = p20.toFixed(2);

            if (!isNaN(weight) && !isNaN(beforeDose) && !isNaN(afterDose) && !isNaN(actualDose)) {
                const now = new Date().toLocaleString('zh-TW');
                const recordData = {
                    "記錄時間": now,
                    "體重 (kg)": weight,
                    "預打藥量": predose.toFixed(2),
                    "打藥前": beforeDose,
                    "打藥後": afterDose,
                    "實打藥量": actualDose.toFixed(2),
                    "-10% 範圍": m10.toFixed(2),
                    "+20% 範圍": p20.toFixed(2)
                };

                if (currentRecordId === null) {
                    multiRecords.push(recordData);
                    currentRecordId = multiRecords.length - 1;
                } else {
                    multiRecords[currentRecordId] = recordData;
                }

                document.getElementById('recordCounter').innerText = `紀錄：${multiRecords.length} 筆`;
            }

            if (!isNaN(actualDose)) {
                if (actualDose > p20) {
                    alertBox.className = 'alert-box alert-too-much';
                    alertBox.innerText = '⚠️ 警告：實打藥量【太多】，超出 +20% 上限！';
                    alertBox.style.display = 'block';
                    playBeep(true);
                } else if (actualDose < m10) {
                    alertBox.className = 'alert-box alert-too-less';
                    alertBox.innerText = '⚠️ 注意：實打藥量【太少】，低於 -10% 下限！';
                    alertBox.style.display = 'block';
                    playBeep(false);
                } else {
                    alertBox.style.display = 'none';
                }
            } else {
                alertBox.style.display = 'none';
            }
        } else {
            minus10Elem.innerText = '-';
            plus20Elem.innerText = '-';
            alertBox.style.display = 'none';
        }
    }

    function clearAll() {
        document.getElementById('weight').value = '';
        document.getElementById('beforeDose').value = '';
        document.getElementById('afterDose').value = '';

        document.getElementById('predose').innerText = '-';
        document.getElementById('actualDose').innerText = '-';
        document.getElementById('minus10').innerText = '-';
        document.getElementById('plus20').innerText = '-';

        document.getElementById('alertBox').style.display = 'none';
        currentRecordId = null;
    }

    function exportToExcel() {
        if (multiRecords.length === 0) {
            alert('目前尚無累積的計算紀錄可供下載！');
            return;
        }

        const worksheet = XLSX.utils.json_to_sheet(multiRecords);
        const workbook = XLSX.utils.book_new();
        XLSX.utils.book_append_sheet(workbook, worksheet, "PET打藥總表");

        const now = new Date();
        const timestamp = `${now.getFullYear()}${String(now.getMonth()+1).padStart(2,'0')}${String(now.getDate()).padStart(2,'0')}_${String(now.getHours()).padStart(2,'0')}${String(now.getMinutes()).padStart(2,'0')}`;
        XLSX.writeFile(workbook, `PET打藥紀錄_${timestamp}.xlsx`);
    }
</script>

</body>
</html>
