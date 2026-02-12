<!DOCTYPE html>
<html lang="ja">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Pure White Glass Calc</title>
    <style>
        :root {
            /* 白と薄いグレーのみの構成 */
            --bg-gradient: linear-gradient(135deg, #f5f7fa 0%, #c3cfe2 100%);
            --glass-white: rgba(255, 255, 255, 0.7);
            --glass-border: rgba(255, 255, 255, 0.8);
            --text-main: #2d3436;
            --text-sub: #636e72;
            --accent: #0984e3; /* 落ち着いた青 */
        }

        body {
            font-family: -apple-system, sans-serif;
            margin: 0;
            height: 100vh;
            display: flex;
            justify-content: center;
            align-items: center;
            background: var(--bg-gradient);
        }

        /* 電卓本体 */
        .calculator {
            width: 320px;
            background: var(--glass-white);
            backdrop-filter: blur(20px);
            -webkit-backdrop-filter: blur(20px);
            border: 1px solid var(--glass-border);
            border-radius: 30px;
            box-shadow: 0 20px 50px rgba(0,0,0,0.05);
            padding: 25px;
        }

        /* モード切替（よりシンプルに） */
        .mode-switch {
            display: flex;
            background: rgba(0,0,0,0.03);
            border-radius: 12px;
            padding: 2px;
            margin-bottom: 20px;
        }
        .mode-switch label {
            flex: 1;
            text-align: center;
            font-size: 10px;
            color: var(--text-sub);
            padding: 6px 0;
            cursor: pointer;
            border-radius: 10px;
        }
        .mode-switch input { display: none; }
        .mode-switch input:checked + span {
            background: #fff;
            color: var(--text-main);
            display: block;
            box-shadow: 0 2px 6px rgba(0,0,0,0.05);
            border-radius: 10px;
        }

        /* ディスプレイ */
        .display {
            text-align: right;
            margin-bottom: 25px;
            padding: 0 10px;
        }
        #formula { font-size: 14px; color: var(--text-sub); min-height: 1.5em; margin-bottom: 5px; }
        #result { font-size: 38px; font-weight: 200; color: var(--text-main); }

        /* ボタン */
        .keypad {
            display: grid;
            grid-template-columns: repeat(4, 1fr);
            gap: 12px;
        }
        button {
            height: 60px;
            border: none;
            background: rgba(255, 255, 255, 0.5);
            border-radius: 20px;
            font-size: 20px;
            color: var(--text-main);
            cursor: pointer;
            transition: all 0.2s;
            border: 1px solid rgba(255,255,255,0.5);
        }
        button:hover { background: #fff; transform: translateY(-2px); box-shadow: 0 5px 15px rgba(0,0,0,0.05); }
        button:active { transform: scale(0.95); }

        .btn-op { color: var(--accent); font-weight: bold; }
        .btn-ac { color: #ff7675; }
        .btn-eq { grid-column: span 2; background: #fff; border: 1px solid var(--accent); color: var(--accent); }

        /* 履歴（シンプル版） */
        .history {
            margin-top: 20px;
            border-top: 1px solid rgba(0,0,0,0.05);
            padding-top: 15px;
            max-height: 80px;
            overflow-y: auto;
        }
        .hist-item {
            font-size: 12px;
            color: var(--text-sub);
            padding: 5px;
            cursor: pointer;
            text-align: right;
            border-radius: 8px;
        }
        .hist-item:hover { background: rgba(255,255,255,0.8); color: var(--text-main); }
    </style>
</head>
<body>

<div class="calculator">
    <div class="mode-switch">
        <label><input type="radio" name="m" value="auto" checked><span>自動</span></label>
        <label><input type="radio" name="m" value="force"><span>強制分数</span></label>
        <label><input type="radio" name="m" value="decimal"><span>小数</span></label>
    </div>

    <div class="display">
        <div id="formula"></div>
        <div id="result">0</div>
    </div>

    <div class="keypad">
        <button class="btn-ac" onclick="clearAll()">C</button>
        <button onclick="back()">←</button>
        <button class="btn-op" onclick="add('/')">/</button>
        <button class="btn-op" onclick="add('*')">×</button>

        <button onclick="add('7')">7</button>
        <button onclick="add('8')">8</button>
        <button onclick="add('9')">9</button>
        <button class="btn-op" onclick="add('/')">÷</button>

        <button onclick="add('4')">4</button>
        <button onclick="add('5')">5</button>
        <button onclick="add('6')">6</button>
        <button class="btn-op" onclick="add('-')">-</button>

        <button onclick="add('1')">1</button>
        <button onclick="add('2')">2</button>
        <button onclick="add('3')">3</button>
        <button class="btn-op" onclick="add('+')">+</button>

        <button onclick="add('0')">0</button>
        <button onclick="add('.')">.</button>
        <button class="btn-eq" onclick="solve()">=</button>
    </div>

    <div class="history" id="hist-list"></div>
</div>

<script>
    let exp = "";
    const gcd = (a, b) => b ? gcd(b, a % b) : Math.abs(a);

    function add(v) { exp += v; update(); }
    function clearAll() { exp = ""; document.getElementById('result').innerText = "0"; update(); }
    function back() { exp = exp.slice(0, -1); update(); }
    function update() { document.getElementById('formula').innerText = exp; }

    function solve() {
        try {
            const res = eval(exp);
            const mode = document.querySelector('input[name="m"]:checked').value;
            let out = res.toString();

            if (mode !== 'decimal') {
                const isInf = out.length > 10;
                if (mode === 'force' || (mode === 'auto' && isInf)) out = toFrac(res);
            }

            document.getElementById('result').innerText = out;
            save(exp, out);
            exp = res.toString();
        } catch(e) { document.getElementById('result').innerText = "Error"; }
    }

    function toFrac(n) {
        if (Number.isInteger(n)) return n;
        let d = 1000000, num = Math.round(n * d), common = gcd(num, d);
        return `${num/common}/${d/common}`;
    }

    function save(f, r) {
        const div = document.createElement('div');
        div.className = "hist-item";
        div.innerText = `${f} = ${r}`;
        div.onclick = () => { exp = f; update(); document.getElementById('result').innerText = r; };
        const list = document.getElementById('hist-list');
        list.prepend(div);
    }
</script>

</body>
</html>
