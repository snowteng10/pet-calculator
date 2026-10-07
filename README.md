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

        /* 雙欄卡片佈局 */
        .grid-2col {
            display: grid;
            grid-template-columns: 1fr 1fr;
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
            color: #d69e2e;
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
<body onclick="initAudio()">

<div class="calculator-card">
    <div class="header">
        <h2>PET 打藥劑量計算器</h2>
        <div class="counter-badge" id="recordCounter">紀錄：0 筆</div>
    </div>

    <div id="alertBox" class="alert-box"></div>

    <!-- 第一排：體重 (輸入) + 預打藥量 (計算結果) -->
    <div class="grid-2col">
        <div class="input-group">
            <label style="color:#2b6cb0;">體重 (kg)</label>
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
            <label>打藥前</label>
            <input type="number" id="beforeDose" step="any" placeholder="0" oninput="calculate()">
        </div>
        <div class="input-group">
            <label>打藥後</label>
            <input type="number" id="afterDose" step="any" placeholder="0" oninput="calculate()">
        </div>
    </div>

    <!-- 第三排：實打藥量 (計算結果) -->
    <div class="res-card card-green">
        <div class="title">實打藥量 (打藥前 - 打藥後)</div>
        <div class="value" id="actualDose">-</div>
    </div>

    <!-- 第四排：安全劑量範圍 -->
    <div class="res-card card-range">
        <div class="title">安全劑量建議範圍</div>
        <div class="range-container">
            <div class="range-sub">
                <span class="sub-title">-1
