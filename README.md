[index (2).html](https://github.com/user-attachments/files/33131338/index.2.html)[Uploa<!DOCTYPE html>
<html lang="zh-TW">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>PET 打藥劑量計算器 (手機直式版)</title>
    <!-- Tailwind CSS CDN -->
    <script src="https://cdn.tailwindcss.com"></script>
    <!-- SheetJS (xlsx) CDN for Excel Export -->
    <script src="https://cdnjs.cloudflare.com/ajax/libs/xlsx/0.18.5/xlsx.full.min.js"></script>
    <style>
        /* custom scrollbar and mobile-friendly tap highlights */
        * {
            -webkit-tap-highlight-color: transparent;
        }
        body {
            font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, "Helvetica Neue", Arial, sans-serif;
            background-color: #f1f5f9;
        }
    </style>
</head>
<body class="min-h-screen pb-12 pt-4 px-3 sm:px-6 flex flex-col justify-start items-center">

    <div class="w-full max-w-md bg-white rounded-2xl shadow-xl overflow-hidden border border-slate-200">
        
        <!-- Header Section -->
        <div class="bg-slate-800 text-white py-5 px-4 text-center relative shadow-sm">
            <h1 class="text-2xl font-bold tracking-wide">PET 打藥劑量計算器</h1>
            <p class="text-xs text-slate-300 mt-1">行動裝置 / 直式快速計算版</p>
            <div id="recordCounter" class="mt-3 inline-block bg-slate-700/80 text-amber-300 text-xs font-semibold px-3 py-1 rounded-full border border-slate-600">
                目前累積記錄筆數：0 筆
            </div>
        </div>

        <!-- Alert Notification Box -->
        <div id="alertBox" class="hidden m-4 p-4 rounded-xl text-center text-sm font-bold border-2 transition-all duration-300 shadow-sm animate-pulse"></div>

        <!-- Main Form Cards Container -->
        <div class="p-4 space-y-4">

            <!-- 1. 體重 (kg) 卡片 -->
            <div class="bg-blue-50/80 border-2 border-blue-200 rounded-xl p-4 shadow-sm">
                <div class="flex justify-between items-center mb-2">
                    <label for="weight" class="text-blue-900 font-bold text-base flex items-center gap-1.5">
                        <span class="inline-block w-2.5 h-2.5 rounded-full bg-blue-500"></span>
                        體重 (kg)
                    </label>
                </div>
                <input type="number" step="any" id="weight" placeholder="請輸入體重" oninput="calculate()"
                    class="w-full h-14 text-center text-2xl font-bold bg-white text-slate-800 rounded-lg border-2 border-blue-300 focus:border-blue-500 focus:ring-2 focus:ring-blue-200 outline-none transition-all shadow-inner">
            </div>

            <!-- 2. 預打藥量 卡片 (自動計算) -->
            <div class="bg-amber-50/80 border-2 border-amber-200 rounded-xl p-4 shadow-sm">
                <div class="flex justify-between items-baseline mb-1">
                    <span class="text-amber-900 font-bold text-base flex items-center gap-1.5">
                        <span class="inline-block w-2.5 h-2.5 rounded-full bg-amber-500"></span>
                        預打藥量
                    </span>
                    <span class="text-xs font-medium text-amber-700 bg-amber-100/80 px-2 py-0.5 rounded">體重 × 0.14</span>
                </div>
                <div class="h-14 bg-white rounded-lg border-2 border-amber-200 flex items-center justify-center shadow-inner">
                    <span id="predose" class="text-2xl font-extrabold text-amber-900">-</span>
                </div>
            </div>

            <!-- 3. 打藥前與打藥後 卡片 (並排或直式) -->
            <div class="bg-yellow-50/80 border-2 border-yellow-200 rounded-xl p-4 shadow-sm space-y-3">
                <div class="text-yellow-900 font-bold text-base flex items-center gap-1.5 border-b border-yellow-200/60 pb-2">
                    <span class="inline-block w-2.5 h-2.5 rounded-full bg-yellow-500"></span>
                    打藥數值紀錄
                </div>
                
                <div class="grid grid-cols-2 gap-3">
                    <div>
                        <label for="beforeDose" class="block text-xs font-semibold text-yellow-800 mb-1">打藥前</label>
                        <input type="number" step="any" id="beforeDose" placeholder="打藥前" oninput="calculate()"
                            class="w-full h-12 text-center text-xl font-bold bg-white text-slate-800 rounded-lg border-2 border-yellow-300 focus:border-yellow-500 focus:ring-2 focus:ring-yellow-200 outline-none transition-all shadow-inner">
                    </div>
                    <div>
                        <label for="afterDose" class="block text-xs font-semibold text-yellow-800 mb-1">打藥後</label>
                        <input type="number" step="any" id="afterDose" placeholder="打藥後" oninput="calculate()"
                            class="w-full h-12 text-center text-xl font-bold bg-white text-slate-800 rounded-lg border-2 border-yellow-300 focus:border-yellow-500 focus:ring-2 focus:ring-yellow-200 outline-none transition-all shadow-inner">
                    </div>
                </div>
            </div>

            <!-- 4. 實打藥量 卡片 (自動計算) -->
            <div class="bg-emerald-50/80 border-2 border-emerald-300 rounded-xl p-4 shadow-sm">
                <div class="flex justify-between items-baseline mb-1">
                    <span class="text-emerald-900 font-bold text-base flex items-center gap-1.5">
                        <span class="inline-block w-2.5 h-2.5 rounded-full bg-emerald-500"></span>
                        實打藥量
                    </span>
                    <span class="text-xs font-medium text-emerald-700 bg-emerald-100 px-2 py-0.5 rounded">打藥前 − 打藥後</span>
                </div>
                <div class="h-14 bg-white rounded-lg border-2 border-emerald-300 flex items-center justify-center shadow-inner">
                    <span id="actualDose" class="text-3xl font-black text-emerald-700">-</span>
                </div>
            </div>

            <!-- 5. 參考範圍 (-10% / +20%) 卡片 -->
            <div class="bg-red-50/60 border-2 border-red-200 rounded-xl p-4 shadow-sm">
                <div class="text-red-900 font-bold text-base mb-2 flex items-center gap-1.5">
                    <span class="inline-block w-2.5 h-2.5 rounded-full bg-red-400"></span>
                    建議實打範圍 (基準 -10% ~ +20%)
                </div>
                <div class="grid grid-cols-2 gap-3">
                    <div class="bg-white p-3 rounded-lg border border-red-200 text-center shadow-inner">
                        <div class="text-xs text-red-600 font-medium mb-1">-10% 下限</div>
                        <div id="minus10" class="text-xl font-bold text-slate-700">-</div>
                    </div>
                    <div class="bg-white p-3 rounded-lg border border-red-200 text-center shadow-inner">
                        <div class="text-xs text-red-600 font-medium mb-1">+20% 上限</div>
                        <div id="plus20" class="text-xl font-bold text-slate-700">-</div>
                    </div>
                </div>
            </div>

            <!-- 操作按鈕區塊 -->
            <div class="pt-2 space-y-3">
                <button onclick="exportToExcel()" 
                    class="w-full py-4 bg-emerald-600 hover:bg-emerald-700 active:bg-emerald-800 text-white font-bold text-base rounded-xl shadow-lg transition duration-150 flex items-center justify-center gap-2 touch-manipulation">
                    <svg class="w-5 h-5" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M12 10v6m0 0l-3-3m3 3l3-3m2 8H7a2 2 0 01-2-2V5a2 2 0 012-2h5.586a1 1 0 01.707.293l5.414 5.414a1 1 0 01.293.707V19a2 2 0 01-2 2z"></path></svg>
                    📥 下載完整多筆 Excel 總表
                </button>
                <button onclick="clearAll()" 
                    class="w-full py-3.5 bg-slate-500 hover:bg-slate-600 active:bg-slate-700 text-white font-bold text-base rounded-xl shadow transition duration-150 flex items-center justify-center gap-2 touch-manipulation">
                    <svg class="w-5 h-5" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M4 4v5h.582m15.356 2A8.001 8.001 0 004.582 9m0 0H9m11 11v-5h-.581m0 0a8.003 8.003 0 01-15.357-2m15.357 2H15"></path></svg>
                    清除重置（換下一位）
                </button>
            </div>

        </div>
    </div>

    <!-- JavaScript 邏輯部分 -->
    <script>
        // 儲存累積紀錄的陣列與當前紀錄指標
        let multiRecords = [];
        let currentRecordId = null;

        // 警報聲音提示 (Web Audio API)
        function playBeep(isTooMuch) {
            try {
                const AudioContext = window.AudioContext || window.webkitAudioContext;
                if (!AudioContext) return;
                const audioCtx = new AudioContext();
                const oscillator = audioCtx.createOscillator();
                const gainNode = audioCtx.createGain();
                
                oscillator.type = 'sine';
                if (isTooMuch) {
                    // 太高：高頻警告音
                    oscillator.frequency.setValueAtTime(800, audioCtx.currentTime);
                    oscillator.frequency.setValueAtTime(1000, audioCtx.currentTime + 0.15);
                } else {
                    // 太低：低頻提醒音
                    oscillator.frequency.setValueAtTime(500, audioCtx.currentTime);
                    oscillator.frequency.setValueAtTime(400, audioCtx.currentTime + 0.15);
                }
                
                gainNode.gain.setValueAtTime(0.3, audioCtx.currentTime);
                oscillator.connect(gainNode);
                gainNode.connect(audioCtx.destination);
                
                oscillator.start();
                oscillator.stop(audioCtx.currentTime + 0.4);
            } catch (e) {
                console.log("音效播放失敗或瀏覽器受限:", e);
            }
        }

        // 核心劑量計算 logic
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

            // 計算預打藥量 (體重 × 0.14)
            if (!isNaN(weight) && weight > 0) {
                predose = weight * 0.14;
                predoseElem.innerText = predose.toFixed(2);
            } else {
                predoseElem.innerText = '-';
            }

            // 計算實打藥量 (打藥前 − 打藥後)
            if (!isNaN(beforeDose) && !isNaN(afterDose)) {
                actualDose = beforeDose - afterDose;
                actualDoseElem.innerText = actualDose.toFixed(2);
            } else {
                actualDoseElem.innerText = '-';
            }

            // 計算上下限範圍 (-10% / +20%)
            if (!isNaN(predose)) {
                const m10 = predose * 0.9;
                const p20 = predose * 1.2;
                
                minus10Elem.innerText = m10.toFixed(2);
                plus20Elem.innerText = p20.toFixed(2);

                // 自動背景更新/記錄
                if (!isNaN(weight) && !isNaN(beforeDose) && !isNaN(afterDose) && !isNaN(actualDose)) {
                    const now = new Date().toLocaleString('zh-TW', { hour12: false });
                    const recordData = {
                        "記錄時間": now,
                        "體重 (kg)": weight,
                        "預打藥量": parseFloat(predose.toFixed(2)),
                        "打藥前": beforeDose,
                        "打藥後": afterDose,
                        "實打藥量": parseFloat(actualDose.toFixed(2)),
                        "-10% 範圍": parseFloat(m10.toFixed(2)),
                        "+20% 範圍": parseFloat(p20.toFixed(2))
                    };

                    if (currentRecordId === null) {
                        multiRecords.push(recordData);
                        currentRecordId = multiRecords.length - 1;
                    } else {
                        multiRecords[currentRecordId] = recordData;
                    }

                    document.getElementById('recordCounter').innerText = `目前累積記錄筆數：${multiRecords.length} 筆`;
                }

                // 警報條件判斷
                if (!isNaN(actualDose)) {
                    if (actualDose > p20) {
                        alertBox.className = 'm-4 p-4 rounded-xl text-center text-sm font-bold border-2 transition-all duration-300 shadow-sm bg-red-100 border-red-400 text-red-700 block';
                        alertBox.innerText = '⚠️ 警告：實打藥量【太多】，已超出 +20% 上限範圍！';
                        playBeep(true);
                    } else if (actualDose < m10) {
                        alertBox.className = 'm-4 p-4 rounded-xl text-center text-sm font-bold border-2 transition-all duration-300 shadow-sm bg-amber-100 border-amber-400 text-amber-800 block';
                        alertBox.innerText = '⚠️ 注意：實打藥量【太少】，低於 -10% 下限範圍！';
                        playBeep(false);
                    } else {
                        alertBox.className = 'hidden';
                    }
                } else {
                    alertBox.className = 'hidden';
                }
            } else {
                minus10Elem.innerText = '-';
                plus20Elem.innerText = '-';
                alertBox.className = 'hidden';
            }
        }

        // 清除當前輸入（換下一位患者）
        function clearAll() {
            document.getElementById('weight').value = '';
            document.getElementById('beforeDose').value = '';
            document.getElementById('afterDose').value = '';

            document.getElementById('predose').innerText = '-';
            document.getElementById('actualDose').innerText = '-';
            document.getElementById('minus10').innerText = '-';
            document.getElementById('plus20').innerText = '-';

            document.getElementById('alertBox').className = 'hidden';

            // 重置當前筆數 id，下次輸入將新增為新紀錄
            currentRecordId = null;
        }

        // 匯出 Excel 檔案
        function exportToExcel() {
            if (multiRecords.length === 0) {
                alert('目前尚無累積的計算紀錄可供下載！請先輸入體重與打藥數值。');
                return;
            }

            // 使用 SheetJS 產生工作表與活頁簿
            const worksheet = XLSX.utils.json_to_sheet(multiRecords);
            const workbook = XLSX.utils.book_new();
            XLSX.utils.book_append_sheet(workbook, worksheet, "PET打藥總表");

            // 檔名加上目前日期時間
            const now = new Date();
            const year = now.getFullYear();
            const month = String(now.getMonth() + 1).padStart(2, '0');
            const day = String(now.getDate()).padStart(2, '0');
            const hour = String(now.getHours()).padStart(2, '0');
            const minute = String(now.getMinutes()).padStart(2, '0');
            
            const fileName = `PET打藥劑量總表_${year}${month}${day}_${hour}${minute}.xlsx`;

            // 觸發下載
            XLSX.writeFile(workbook, fileName);
        }
    </script>
</body>
</html>ding index (2).html…]()
