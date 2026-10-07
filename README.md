<!DOCTYPE html>
<html lang="zh-TW">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>PET 打藥劑量計算器</title>
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
            padding: 14px 16px;
            display: flex;
            flex-direction: column;
            gap: 10px;
        }
        .header {
            display: flex;
            justify-content: space-between;
            align-items: center;
            border-bottom: 2px solid #edf2f7;
            padding-bottom: 6px;
        }
        .header h2 {
            font-size: 18px;
            color: #1a365d;
            font-weight: 700;
        }
        .sound-btn {
            background-color: #3182ce;
            color: white;
            font-size: 12px;
            padding: 4px 10px;
            border-radius: 12px;
            border: none;
            cursor: pointer;
            font-weight: bold;
        }
        
        /* 警報提示區塊 */
        .alert-box {
            padding: 8px 10px;
            border-radius: 6px;
            text-align: center;
            font-size: 15px;
            font-weight: bold;
            display: none;
        }
        .alert-too-much {
            background-color: #fff5f5;
            border: 2px solid #fc8181;
            color: #c53030;
        }
        .alert-too-less {
            background-color: #fffaf0;
            border: 2px solid #f6ad55;
            color: #dd6b20;
        }

        /* 雙欄卡片佈局 */
        .grid-2col {
            display: grid;
            grid-template-columns: 1fr 1fr;
            gap: 10px;
        }

        .input-group {
            display: flex;
            flex-direction: column;
            align-items: center;
        }
        .input-group label {
            font-size: 14px;
            font-weight: 700;
            margin-bottom: 4px;
            color: #2b6cb0;
        }
        .input-group label.yellow-label {
            color: #d69e2e;
        }
        .input-group input {
            width: 100%;
            height: 48px;
            text-align: center;
            font-size: 28px;
            font-weight: bold;
            border: 2px solid #cbd5e0;
            border-radius: 8px;
            background-color: #f8fafc;
            color: #1a202c;
            outline: none;
        }
        .input-group input:focus {
            border-color: #3182ce;
            background-color: #ffffff;
            box-shadow: 0 0 0 3px rgba(49, 130, 206, 0.25);
        }

        /* 結果顯示卡片區塊 */
        .res-card {
            border-radius: 8px;
            padding: 10px 8px;
            text-align: center;
            display: flex;
            flex-direction: column;
            justify-content: center;
            align-items: center;
        }
        .res-card .title {
            font-size: 13px;
            font-weight: 700;
            margin-bottom: 2px;
        }
        .res-card .value {
            font-size: 32px;
            font-weight: 900;
            line-height: 1.1;
        }
        .res-card .value-large {
            font-size: 36px;
        }
        
        .card-orange { background-color: #feebc8; color: #7b341e; }
        .card-green  { background-color: #c6f6d5; color: #22543d; }
        .card-range  { background-color: #fff5f5; color: #9b2c2c; border: 1.5px dashed #feb2b2; }

        .range-container {
            display: flex;
            justify-content: space-around;
            width: 100%;
            margin-top: 2px;
        }
        .range-sub {
            display: flex;
            flex-direction: column;
            align-items: center;
        }
        .range-sub span.sub-title {
            font-size: 12px;
            font-weight: 700;
            color: #742a2a;
        }
        .range-sub span.sub-val {
            font-size: 26px;
            font-weight: 900;
            color: #2b6cb0;
        }

        /* 按鈕區域 */
        .btn-group {
            display: flex;
            justify-content: center;
            margin-top: 4px;
        }
        .btn-reset {
            width: 100%;
            height: 44px;
            border: none;
            border-radius: 8px;
            font-size: 16px;
            font-weight: 700;
            color: white;
            background-color: #718096;
            cursor: pointer;
        }
        .btn-reset:active {
            opacity: 0.85;
        }
    </style>
</head>
<body onclick="unlockAudio()">

<div class="calculator-card">
    <div class="header">
        <h2>PET 打藥劑量計算器</h2>
        <button class="sound-btn" id="soundBtn" onclick="testSound(event)">🔊 啟用警示音</button>
    </div>

    <div id="alertBox" class="alert-box"></div>

    <!-- 第一排：體重 (輸入) + 預打藥量 (計算結果) -->
    <div class="grid-2col">
        <div class="input-group">
            <label>體重 (kg)</label>
            <input type="number" id="weight" step="any" placeholder="0" oninput="calculate()">
        </div>
        <div class="res-card card-orange">
            <div class="title">預打藥量 (體重×0.14)</div>
            <div class="value" id="predose">-</div>
        </div>
    </div>

    <!-- 第二排：打藥前 (輸入) + 打藥後 (輸入) -->
    <div class="grid-2col">
        <div class="input-group">
            <label class="yellow-label">打藥前</label>
            <input type="number" id="beforeDose" step="any" placeholder="0" oninput="calculate()">
        </div>
        <div class="input-group">
            <label class="yellow-label">打藥後</label>
            <input type="number" id="afterDose" step="any" placeholder="0" oninput="calculate()">
        </div>
    </div>

    <!-- 第三排：實打藥量 (計算結果) -->
    <div class="res-card card-green">
        <div class="title">實打藥量 (打藥前 - 打藥後)</div>
        <div class="value value-large" id="actualDose">-</div>
    </div>

    <!-- 第四排：安全劑量範圍 -->
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

    <!-- 第五排：按鈕 -->
    <div class="btn-group">
        <button class="btn-reset" onclick="clearAll()">🔄 清除換下一位</button>
    </div>
</div>

<script>
    let audioCtx = null;
    let lastAlertState = 'NORMAL';

    // 解鎖手機瀏覽器的 AudioContext 聲音權限
    function unlockAudio() {
        if (!audioCtx) {
            audioCtx = new (window.AudioContext || window.webkitAudioContext)();
        }
        if (audioCtx.state === 'suspended') {
            audioCtx.resume();
        }
    }

    // 點擊右上角按鈕測試音效並解鎖權限
    function testSound(e) {
        if (e) e.stopPropagation();
        unlockAudio();
        playBeep(true);
        const btn = document.getElementById('soundBtn');
        btn.innerText = '✅ 音效已開啟';
        btn.style.backgroundColor = '#38a169';
    }

    // 發出警示蜂鳴聲
    function playBeep(isTooMuch) {
        unlockAudio();
        if (!audioCtx) return;

        try {
            const now = audioCtx.currentTime;
            const osc = audioCtx.createOscillator();
            const gain = audioCtx.createGain();

            // 使用 square 方波音效，聲音最響亮且有警示感
            osc.type = 'square';
            
            if (isTooMuch) {
                // 超出上限：高音快速雙響 (嗶！嗶！)
                osc.frequency.setValueAtTime(1000, now);
                osc.frequency.setValueAtTime(1200, now + 0.15);
                gain.gain.setValueAtTime(0.3, now);
                gain.gain.exponentialRampToValueAtTime(0.01, now + 0.35);
                
                osc.connect(gain);
                gain.connect(audioCtx.destination);
                osc.start(now);
                osc.stop(now + 0.35);
            } else {
                // 低於下限：低音警告聲 (嗶—)
                osc.frequency.setValueAtTime(450, now);
                osc.frequency.setValueAtTime(350, now + 0.15);
                gain.gain.setValueAtTime(0.3, now);
                gain.gain.exponentialRampToValueAtTime(0.01, now + 0.35);

                osc.connect(gain);
                gain.connect(audioCtx.destination);
                osc.start(now);
                osc.stop(now + 0.35);
            }
        } catch (e) {
            console.log("播放音效失敗：", e);
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

            if (!isNaN(actualDose)) {
                if (actualDose > p20) {
                    alertBox.className = 'alert-box alert-too-much';
                    alertBox.innerText = '⚠️ 警告：實打藥量【太多】，超出 +20% 上限！';
                    alertBox.style.display = 'block';
                    if (lastAlertState !== 'TOO_MUCH') {
                        playBeep(true);
                        lastAlertState = 'TOO_MUCH';
                    }
                } else if (actualDose < m10) {
                    alertBox.className = 'alert-box alert-too-less';
                    alertBox.innerText = '⚠️ 注意：實打藥量【太少】，低於 -10% 下限！';
                    alertBox.style.display = 'block';
                    if (lastAlertState !== 'TOO_LESS') {
                        playBeep(false);
                        lastAlertState = 'TOO_LESS';
                    }
                } else {
                    alertBox.style.display = 'none';
                    lastAlertState = 'NORMAL';
                }
            } else {
                alertBox.style.display = 'none';
                lastAlertState = 'NORMAL';
            }
        } else {
            minus10Elem.innerText = '-';
            plus20Elem.innerText = '-';
            alertBox.style.display = 'none';
            lastAlertState = 'NORMAL';
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
        lastAlertState = 'NORMAL';
    }
</script>

</body>
</html>
