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
            padding: 12px;
        }
        .calculator-container {
            background: #ffffff;
            padding: 24px 20px;
            border-radius: 16px;
            box-shadow: 0 10px 30px rgba(0, 0, 0, 0.15);
            width: 100%;
            max-width: 500px;
        }
        h2 {
            text-align: center;
            color: #1a365d;
            margin-bottom: 6px;
            font-size: 24px;
        }
        .record-counter {
            text-align: center;
            color: #4a5568;
            font-size: 15px;
            margin-bottom: 16px;
            font-weight: bold;
        }
        .alert-box {
            padding: 12px;
            border-radius: 10px;
            text-align: center;
            font-size: 16px;
            font-weight: bold;
            margin-bottom: 16px;
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

        /* 雙欄卡片直式佈局 */
        .form-grid {
            display: flex;
            flex-direction: column;
            gap: 12px;
            margin-bottom: 20px;
        }
        .grid-row {
            display: grid;
            grid-template-columns: 1fr 1fr;
            gap: 10px;
        }

        .card {
            border: 2px solid #cbd5e0;
            border-radius: 10px;
            padding: 10px;
            text-align: center;
            display: flex;
            flex-direction: column;
            justify-content: center;
            align-items: center;
        }
        .card label, .card .card-title {
            font-size: 14px;
            font-weight: bold;
            margin-bottom: 6px;
            display: block;
        }

        /* 上層顏色設定 */
        .card-blue { background-color: #bee3f8; color: #1a365d; border-color: #90cdf4; }
        .card-orange { background-color: #feebc8; color: #7b341e; border-color: #fbd38d; }
        .card-yellow { background-color: #fefcbf; color: #744210; border-color: #faf089; }
        .card-green { background-color: #c6f6d5; color: #22543d; border-color: #9ae6b4; }
        .card-range { background-color: #fed7d7; color: #9b2c2c; border-color: #feb2b2; }

        input {
            width: 100%;
            padding: 8px;
            box-sizing: border-box;
            border: 2px solid #a0aec0;
            border-radius: 8px;
            text-align: center;
            font-size: 20px;
            font-weight: bold;
            color: #2d3748;
            background-color: #ffffff;
            outline: none;
        }
        input:focus {
            border-color: #3182ce;
            box-shadow: 0 0 6px rgba(49, 130, 206, 0.4);
        }

        .val-text {
            font-size: 22px;
            font-weight: bold;
        }

        .range-box {
            display: flex;
            justify-content: space-around;
            width: 100%;
            margin-top: 4px;
        }
        .range-item {
            display: flex;
            flex-direction: column;
        }
        .range-item span {
            font-size: 12px;
            font-weight: normal;
        }
        .range-item strong {
            font-size: 20px;
            color: #2b6cb0;
        }

        .btn-container {
            display: flex;
            flex-direction: column;
            gap: 10px;
        }
        button {
            background-color: #3182ce;
            color: white;
            border: none;
            padding: 12px;
            font-size: 16px;
            border-radius: 10px;
            cursor: pointer;
            font-weight: bold;
            transition: background 0.2s;
        }
        button.btn-success { background-color: #38a169; }
        button.btn-secondary { background-color: #718096; }
        button:active { opacity: 0.9; }
    </style>
</head>
<body onclick="initAudio()">

<div class="calculator-container">
    <h2>PET 打藥劑量計算器</h2>
    <div class="record-counter" id="recordCounter">目前累積記錄筆數：0 筆</div>
    
    <div id="alertBox" class="alert-box"></div>

    <div class="form-grid">
        <!-- 第一排：體重 + 預打藥量 -->
        <div class="grid-row">
            <div class="card card-blue">
                <label for="weight">體重 (kg)</label>
                <input type="number" id="weight" step="any" placeholder="輸入體重" oninput="calculate()">
            </div>
            <div class="card card-orange">
                <span class="card-title">預打藥量<br><small style="font-size:11px; font-weight:normal;">(體重 × 0.14)</small></span>
                <span class="val-text" id="predose">-</span>
            </div>
