<!DOCTYPE html>
<html lang="ja">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>普通で便利な分数電卓</title>
    <style>
        body { font-family: sans-serif; background-color: #e0e7ff; display: flex; justify-content: center; align-items: center; min-height: 100vh; margin: 0; }
        .calc-container { background: #ffffff; width: 360px; border-radius: 15px; box-shadow: 0 8px 20px rgba(0,0,0,0.15); padding: 20px; }
        
        /* モード選択 */
        .mode-selector { display: flex; font-size: 11px; background: #eee; border-radius: 8px; margin-bottom: 10px; padding: 3px; }
        .mode-selector label { flex: 1; text-align: center; padding: 5px; cursor: pointer; border-radius: 6px; }
        .mode-selector input { display: none; }
        .mode-selector input:checked + span { background: #4f46e5; color: white; border-radius: 5px; display: block; padding: 2px; }

        .display { background: #f8fafc; border: 2px solid #e2e8f0; border-radius: 10px; padding: 15px; text-align: right; margin-bottom: 15px; }
        #formula { font-size: 14px; color: #64748b; min-height: 1.5em; overflow-x: auto; }
        #result { font-size: 28px; font-weight: bold; color: #1e293b; overflow-x: auto; }

        .btn-grid { display: grid; grid-template-columns: repeat(4, 1fr); gap: 10px; }
        button { height: 55px; border: none; border-radius: 10px; font-size: 18px; cursor: pointer; background: #f1f5f9; transition: 0.1s; }
        button:hover { background: #e2e8f0; }
        button.op { background: #e0e7ff; color: #4f46e5; font-weight: bold; }
        button.eq { background: #4f46e5; color: white; grid-column: span 2; }
        button.ac { background: #fecaca; color: #dc2626; }

        .history-box { margin-top: 20px; border-top: 1px solid #e2e8f0; padding-top: 10px; }
        .history-title { font-size: 12px; font-weight: bold; color: #64748b; margin-bottom: 5px; }
        #history-list { max-height: 120px; overflow-y: auto; font-size: 13px; color: #475569; }
        .history-item { padding: 4px 0; border-bottom: 1px dashed #eee; }
    </style>
</head>
<body>

<div class="calc-container">
    <div class="mode-selector">
        <label><input type="radio" name="mode" value="auto" checked><span>自動(無限時分数)</span></label>
        <label><input type="radio" name="mode" value="force"><span>強制分数</span></label>
        <label><input type="radio" name="mode" value="decimal"><span>絶対小数</span></label>
    </div>

    <div class="display">
        <div id="formula"></div>
        <div id="result">0</div>
    </div>

    <div class="btn-grid">
        <button class="ac" onclick="clearAll()">AC</button>
        <button class="op" onclick="backspace()">DEL</button>
        <button class="op" onclick="input('/')">分数(/)</button>
        <button class="op" onclick="input('*')">×</button>

        <button onclick="input('7')">7</button>
        <button onclick="input('8')">8</button>
        <button onclick="input('9')">9</button>
        <button class="op" onclick="input('/')">÷</button>

        <button onclick="input('4')">4</button>
        <button onclick="input('5')">5</button>
        <button onclick="input('6')">6</button>
        <button class="op" onclick="input('-')">-</button>

        <button onclick="input('1')">1</button>
        <button onclick="input('2')">2</button>
        <button onclick="input('3')">3</button>
        <button class="op" onclick="input('+')">+</button>

        <button onclick="input('0')">0</button>
        <button onclick="input('.')">.</button>
        <button class="eq" onclick="solve()">=</button>
    </div>

    <div class="history-box">
        <div class="history-title">履歴 (デバイス保存)</div>
        <div id="history-list"></div>
    </div>
</div>

<script>
    let currentFormula = "";
    const gcd = (a, b) => b ? gcd(b, a % b) : Math.abs(a);

    // 起動時に履歴読み込み
    window.onload = () => {
        const saved = JSON.parse(localStorage.getItem('calc_history') || "[]");
        saved.forEach(item => addHistoryUI(item));
    };

    function input(v) {
        currentFormula += v;
        updateView();
    }

    function clearAll() {
        currentFormula = "";
        document.getElementById('result').innerText = "0";
        updateView();
    }

    function backspace() {
        currentFormula = currentFormula.slice(0, -1);
        updateView();
    }

    function updateView() {
        document.getElementById('formula').innerText = currentFormula;
    }

    function solve() {
        try {
            const rawResult = eval(currentFormula);
            const mode = document.querySelector('input[name="mode"]:checked').value;
            let finalDisplay = "";

            if (mode === 'decimal') {
                finalDisplay = rawResult.toString();
            } else {
                const isInfinite = rawResult.toString().length > 12; // 簡易的な無限・長小数判定
                if (mode === 'force' || (mode === 'auto' && isInfinite)) {
                    finalDisplay = toFraction(rawResult);
                } else {
                    finalDisplay = rawResult.toString();
                }
            }

            const record = `${currentFormula} = ${finalDisplay}`;
            document.getElementById('result').innerText = finalDisplay;
            saveHistory(record);
            currentFormula = rawResult.toString();
        } catch (e) {
            document.getElementById('result').innerText = "Error";
        }
    }

    function toFraction(dec) {
        if (Number.isInteger(dec)) return dec.toString();
        const precision = 1000000;
        let num = Math.round(dec * precision);
        let den = precision;
        let common = gcd(num, den);
        return `${num / common}/${den / common}`;
    }

    function saveHistory(item) {
        addHistoryUI(item);
        const saved = JSON.parse(localStorage.getItem('calc_history') || "[]");
        saved.unshift(item);
        localStorage.setItem('calc_history', JSON.stringify(saved.slice(0, 20)));
    }

    function addHistoryUI(text) {
        const list = document.getElementById('history-list');
        const div = document.createElement('div');
        div.className = "history-item";
        div.innerText = text;
        list.prepend(div);
    }
</script>
</body>
</html>
