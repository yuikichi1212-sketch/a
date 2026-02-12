<!DOCTYPE html>
<html lang="ja">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Liquid Glass Calculator</title>
    <style>
        /* --- Liquid Glass Theme Settings --- */
        :root {
            --glass-bg: rgba(255, 255, 255, 0.25);
            --glass-border: rgba(255, 255, 255, 0.6);
            --glass-shadow: 0 8px 32px 0 rgba(31, 38, 135, 0.15);
            --text-main: #4a5568;
            --text-dim: #a0aec0;
            --accent-color: #667eea; /* ソフトな青紫 */
        }

        body {
            font-family: 'Helvetica Neue', Arial, sans-serif;
            margin: 0;
            height: 100vh;
            display: flex;
            justify-content: center;
            align-items: center;
            /* 背景グラデーション（ガラス効果を際立たせるため） */
            background: linear-gradient(135deg, #e0c3fc 0%, #8ec5fc 100%);
            overflow: hidden;
        }

        /* 浮遊する背景装飾（リキッド感） */
        .blob {
            position: absolute;
            background: rgba(255, 255, 255, 0.4);
            border-radius: 50%;
            filter: blur(40px);
            z-index: -1;
        }
        .blob-1 { width: 300px; height: 300px; top: -50px; left: -50px; background: #ff9a9e; }
        .blob-2 { width: 350px; height: 350px; bottom: -80px; right: -80px; background: #a18cd1; }

        /* --- Calculator Container --- */
        .calculator {
            width: 340px;
            background: var(--glass-bg);
            backdrop-filter: blur(16px); /* すりガラス効果 */
            -webkit-backdrop-filter: blur(16px);
            border: 1px solid var(--glass-border);
            border-radius: 24px;
            box-shadow: var(--glass-shadow);
            padding: 20px;
            display: flex;
            flex-direction: column;
            gap: 15px;
        }

        /* --- Display Area --- */
        .display-area {
            text-align: right;
            padding: 10px 5px;
            margin-bottom: 5px;
        }
        #input-line {
            font-size: 14px;
            color: var(--text-dim);
            min-height: 1.2em;
            letter-spacing: 1px;
        }
        #result-line {
            font-size: 36px;
            font-weight: 300;
            color: var(--text-main);
            margin-top: 5px;
            overflow-x: auto;
            white-space: nowrap;
        }

        /* --- Mode Switcher (Glass Style) --- */
        .mode-switch {
            display: flex;
            background: rgba(255, 255, 255, 0.3);
            border-radius: 12px;
            padding: 4px;
        }
        .mode-switch label {
            flex: 1;
            text-align: center;
            font-size: 11px;
            color: var(--text-main);
            padding: 6px 0;
            cursor: pointer;
            border-radius: 8px;
            transition: 0.3s;
        }
        .mode-switch input { display: none; }
        .mode-switch input:checked + span {
            background: #ffffff;
            box-shadow: 0 2px 5px rgba(0,0,0,0.05);
            font-weight: bold;
            color: var(--accent-color);
            display: block;
            height: 100%;
            width: 100%;
            border-radius: 8px;
            display: flex;
            align-items: center;
            justify-content: center;
        }

        /* --- Keypad --- */
        .keypad {
            display: grid;
            grid-template-columns: repeat(4, 1fr);
            gap: 12px;
        }
        button {
            height: 55px;
            border: 1px solid rgba(255,255,255,0.4);
            background: rgba(255, 255, 255, 0.25);
            border-radius: 16px;
            font-size: 18px;
            color: var(--text-main);
            cursor: pointer;
            transition: all 0.2s ease;
            backdrop-filter: blur(4px);
        }
        button:hover {
            background: rgba(255, 255, 255, 0.6);
            transform: translateY(-2px);
        }
        button:active { transform: scale(0.95); }
        
        .btn-op { color: var(--accent-color); font-weight: bold; background: rgba(255, 255, 255, 0.4); }
        .btn-eq { 
            grid-column: span 2; 
            background: var(--accent-color); 
            color: white; 
            border: none;
            box-shadow: 0 4px 15px rgba(102, 126, 234, 0.4);
        }
        .btn-eq:hover { background: #5a67d8; color: white; }
        .btn-ac { color: #e53e3e; background: rgba(255, 220, 220, 0.3); }

        /* --- History --- */
        .history-container {
            margin-top: 10px;
            border-top: 1px solid rgba(255,255,255,0.3);
            padding-top: 10px;
        }
        .history-label {
            font-size: 11px;
            color: var(--text-dim);
            margin-bottom: 5px;
            padding-left: 5px;
        }
        #history-list {
            max-height: 100px;
            overflow-y: auto;
            display: flex;
            flex-direction: column;
            gap: 6px;
        }
        /* スクロールバーのカスタマイズ */
        #history-list::-webkit-scrollbar { width: 4px; }
        #history-list::-webkit-scrollbar-thumb { background: rgba(0,0,0,0.1); border-radius: 4px; }

        .history-item {
            font-size: 13px;
            color: var(--text-main);
            background: rgba(255,255,255,0.3);
            padding: 8px 12px;
            border-radius: 8px;
            cursor: pointer;
            display: flex;
            justify-content: space-between;
            transition: 0.2s;
        }
        .history-item:hover {
            background: rgba(255,255,255,0.6);
        }
        .history-arrow { opacity: 0.3; font-size: 10px; }
    </style>
</head>
<body>

<div class="blob blob-1"></div>
<div class="blob blob-2"></div>

<div class="calculator">
    
    <div class="mode-switch">
        <label>
            <input type="radio" name="calcMode" value="auto" checked>
            <span>Auto (無限時)</span>
        </label>
        <label>
            <input type="radio" name="calcMode" value="force">
            <span>強制分数</span>
        </label>
        <label>
            <input type="radio" name="calcMode" value="decimal">
            <span>小数のみ</span>
        </label>
    </div>

    <div class="display-area">
        <div id="input-line"></div>
        <div id="result-line">0</div>
    </div>

    <div class="keypad">
        <button class="btn-ac" onclick="clearCalc()">AC</button>
        <button class="btn-op" onclick="backspace()">DEL</button>
        <button class="btn-op" onclick="append('/')">分数(/)</button>
        <button class="btn-op" onclick="append('*')">×</button>

        <button onclick="append('7')">7</button>
        <button onclick="append('8')">8</button>
        <button onclick="append('9')">9</button>
        <button class="btn-op" onclick="append('/')">÷</button>

        <button onclick="append('4')">4</button>
        <button onclick="append('5')">5</button>
        <button onclick="append('6')">6</button>
        <button class="btn-op" onclick="append('-')">-</button>

        <button onclick="append('1')">1</button>
        <button onclick="append('2')">2</button>
        <button onclick="append('3')">3</button>
        <button class="btn-op" onclick="append('+')">+</button>

        <button onclick="append('0')">0</button>
        <button onclick="append('.')">.</button>
        <button class="btn-eq" onclick="calculate()">=</button>
    </div>

    <div class="history-container">
        <div class="history-label">履歴 (タップで復元)</div>
        <div id="history-list">
            </div>
    </div>
</div>

<script>
    let currentInput = "";
    const inputLine = document.getElementById('input-line');
    const resultLine = document.getElementById('result-line');
    const historyList = document.getElementById('history-list');

    // 初期化処理
    window.onload = () => {
        loadHistory();
    };

    function append(val) {
        currentInput += val;
        updateDisplay();
    }

    function clearCalc() {
        currentInput = "";
        resultLine.innerText = "0";
        updateDisplay();
    }

    function backspace() {
        currentInput = currentInput.slice(0, -1);
        updateDisplay();
    }

    function updateDisplay() {
        // 見やすく整形（*を×に見た目だけ変える等はここで行えますが今回はシンプルに）
        inputLine.innerText = currentInput;
    }

    // 計算実行
    function calculate() {
        try {
            if (!currentInput) return;

            // 計算実行（evalは簡易的なため使用。本来はライブラリ推奨）
            const rawResult = eval(currentInput); 
            
            // モードに応じた変換
            const mode = document.querySelector('input[name="calcMode"]:checked').value;
            let finalOutput = rawResult.toString();

            if (mode === 'decimal') {
                finalOutput = rawResult.toString();
            } else {
                // 無限小数判定（簡易的：文字列表現が長い場合）
                const strVal = rawResult.toString();
                const isLongDecimal = strVal.includes('.') && strVal.length > 10;
                
                if (mode === 'force') {
                    finalOutput = toFraction(rawResult);
                } else if (mode === 'auto' && isLongDecimal) {
                    finalOutput = toFraction(rawResult);
                }
            }

            resultLine.innerText = finalOutput;
            addToHistory(currentInput, finalOutput);
            currentInput = rawResult.toString(); // 次の計算のために結果を入力へ

        } catch (e) {
            resultLine.innerText = "Error";
        }
    }

    // 小数 -> 分数変換ロジック
    function toFraction(val) {
        if (Number.isInteger(val)) return val.toString();
        
        const tolerance = 1.0E-9; // 許容誤差
        let h1 = 1, h2 = 0, k1 = 0, k2 = 1;
        let b = val;
        
        do {
            let a = Math.floor(b);
            let aux = h1; h1 = a * h1 + h2; h2 = aux;
            aux = k1; k1 = a * k1 + k2; k2 = aux;
            b = 1 / (b - a);
        } while (Math.abs(val - h1 / k1) > val * tolerance);

        return `${h1}/${k1}`;
    }

    // --- 履歴機能 ---

    function addToHistory(formula, result) {
        const item = { formula, result, id: Date.now() };
        
        // UIに追加
        prependHistoryItem(item);

        // LocalStorageに保存
        let history = JSON.parse(localStorage.getItem('glass_calc_history') || "[]");
        history.unshift(item);
        if (history.length > 20) history.pop(); // 最大20件
        localStorage.setItem('glass_calc_history', JSON.stringify(history));
    }

    function loadHistory() {
        let history = JSON.parse(localStorage.getItem('glass_calc_history') || "[]");
        history.forEach(item => prependHistoryItem(item, false)); // false = 後ろに追加しない（読み込み順序維持のため工夫が必要だが今回は簡易的に逆順ループせずunshift同様の処理を行う）
        // 読み込み時は逆順になっているので、一旦クリアして正しい順序で描画するのがベター
        historyList.innerHTML = ''; 
        history.forEach(item => {
            const el = createHistoryElement(item);
            historyList.appendChild(el);
        });
    }

    function prependHistoryItem(item) {
        const el = createHistoryElement(item);
        historyList.prepend(el);
    }

    function createHistoryElement(item) {
        const div = document.createElement('div');
        div.className = 'history-item';
        div.innerHTML = `
            <span>${item.formula} = <strong>${item.result}</strong></span>
            <span class="history-arrow">↺</span>
        `;
        // タップで復元
        div.onclick = () => {
            currentInput = item.formula; // 式を戻す（計算結果を戻したい場合は item.result に変更）
            updateDisplay();
            resultLine.innerText = item.result;
        };
        return div;
    }

</script>
</body>
</html>
