<!DOCTYPE html>
<html lang="ar" dir="rtl">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>SALAHELDIN DASHBOARD</title>
    <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
    <style>
        :root {
            --bg: #0d1117;
            --card: #161b22;
            --blue: #58a6ff;
            --green: #2ea043;
            --red: #da3633;
            --text: #c9d1d9;
            --border: #30363d;
        }

        body {
            margin: 0;
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
            background-color: var(--bg);
            color: var(--text);
            padding-bottom: 50px;
        }

        .header {
            background-color: var(--card);
            padding: 20px;
            text-align: center;
            border-bottom: 2px solid var(--blue);
            box-shadow: 0 4px 10px rgba(0,0,0,0.3);
        }

        .header h1 {
            margin: 0;
            font-size: 22px;
            color: var(--blue);
            letter-spacing: 1px;
        }

        .container {
            padding: 15px;
            max-width: 600px;
            margin: auto;
        }

        /* لوحة الإحصائيات */
        .dashboard-grid {
            display: grid;
            grid-template-columns: 1fr 1fr;
            gap: 12px;
            margin-bottom: 20px;
        }

        .stat-card {
            background: var(--card);
            padding: 15px;
            border-radius: 12px;
            border: 1px solid var(--border);
            text-align: center;
        }

        .stat-card span {
            display: block;
            font-size: 11px;
            color: #8b949e;
            margin-bottom: 5px;
            text-transform: uppercase;
        }

        .stat-card b {
            font-size: 18px;
        }

        /* الرسم البياني */
        .chart-container {
            background: var(--card);
            padding: 15px;
            border-radius: 12px;
            border: 1px solid var(--border);
            margin-bottom: 20px;
        }

        /* نموذج الإدخال */
        .form-card {
            background: var(--card);
            padding: 20px;
            border-radius: 12px;
            border: 1px solid var(--border);
            margin-bottom: 20px;
        }

        .form-card h3 {
            margin-top: 0;
            font-size: 16px;
            border-right: 4px solid var(--blue);
            padding-right: 10px;
        }

        input, select, textarea {
            width: 100%;
            padding: 12px;
            margin: 8px 0;
            border-radius: 6px;
            border: 1px solid var(--border);
            background: #0d1117;
            color: white;
            box-sizing: border-box;
            font-size: 14px;
        }

        button {
            width: 100%;
            padding: 14px;
            background-color: var(--blue);
            color: #0d1117;
            border: none;
            border-radius: 6px;
            font-weight: bold;
            font-size: 16px;
            cursor: pointer;
            margin-top: 10px;
        }

        /* الجدول */
        .table-container {
            overflow-x: auto;
        }

        table {
            width: 100%;
            border-collapse: collapse;
            background: var(--card);
            border-radius: 12px;
            font-size: 13px;
        }

        th, td {
            padding: 12px;
            text-align: center;
            border-bottom: 1px solid var(--border);
        }

        th {
            background: #21262d;
            color: #8b949e;
        }

        .profit { color: var(--green); font-weight: bold; }
        .loss { color: var(--red); font-weight: bold; }

        .note-cell {
            font-size: 11px;
            color: #8b949e;
            max-width: 100px;
            overflow: hidden;
            text-overflow: ellipsis;
            white-space: nowrap;
        }

        .delete-btn {
            background: none;
            color: var(--red);
            border: 1px solid var(--red);
            padding: 5px 10px;
            font-size: 11px;
            width: auto;
            margin-top: 20px;
        }
    </style>
</head>
<body>

<div class="header">
    <h1>SALAHELDIN DASHBOARD 🚀</h1>
</div>

<div class="container">
    <div class="dashboard-grid">
        <div class="stat-card"><span>عدد الصفقات</span><b id="count">0</b></div>
        <div class="stat-card"><span>نسبة النجاح</span><b id="winRate" style="color:var(--green)">0%</b></div>
        <div class="stat-card"><span>صافي الربح</span><b id="netPnl">$0.00</b></div>
        <div class="stat-card"><span>عامل الربح (PF)</span><b id="pf">0.00</b></div>
    </div>

    <div class="chart-container">
        <canvas id="equityChart"></canvas>
    </div>

    <div class="form-card">
        <h3>📝 تدوين صفقة جديدة</h3>
        <input type="text" id="asset" placeholder="اسم الأصل (مثلاً: Gold, BTC, Tesla)">
        <select id="type">
            <option value="buy">شراء (Long)</option>
            <option value="sell">بيع (Short)</option>
        </select>
        <div style="display: flex; gap: 10px;">
            <input type="number" id="entry" placeholder="سعر الدخول">
            <input type="number" id="exit" placeholder="سعر الخروج">
        </div>
        <input type="number" id="risk" placeholder="المخاطرة الفعلية بالدولار ($)">
        <textarea id="notes" placeholder="ملاحظاتك: لماذا دخلت هذه الصفقة؟"></textarea>
        <button onclick="saveTrade()">حفظ وتحليل البيانات</button>
    </div>

    <div class="table-container">
        <table>
            <thead>
                <tr>
                    <th>الأصل</th>
                    <th>R:R</th>
                    <th>الربح/الخسارة</th>
                    <th>ملاحظات</th>
                </tr>
            </thead>
            <tbody id="tradeTableBody"></tbody>
        </table>
    </div>

    <button class="delete-btn" onclick="clearData()">🗑️ مسح جميع البيانات</button>
</div>

<script>
    let trades = JSON.parse(localStorage.getItem('salaheldin_trades')) || [];
    let myChart;

    function saveTrade() {
        const asset = document.getElementById('asset').value;
        const type = document.getElementById('type').value;
        const entry = parseFloat(document.getElementById('entry').value);
        const exit = parseFloat(document.getElementById('exit').value);
        const risk = parseFloat(document.getElementById('risk').value);
        const notes = document.getElementById('notes').value;

        if (!asset || !entry || !exit || !risk) {
            alert("أكمل جميع البيانات الأساسية يا صلاح الدين");
            return;
        }

        // حساب الربح والخسارة بناءً على الاتجاه
        let pnl = type === 'buy' ? (exit - entry) : (entry - exit);
        // حساب الربح الفعلي بناءً على نسبة المخاطرة
        let finalPnl = (pnl / entry) * (entry / risk) * risk; 
        // تبسيط الحساب للمستخدم ليكون منطقياً بالدولار
        let rr = (finalPnl / risk).toFixed(2);

        const newTrade = { asset, type, finalPnl, rr, notes: notes || "لا توجد ملاحظات" };
        trades.push(newTrade);
        localStorage.setItem('salaheldin_trades', JSON.stringify(trades));
        
        // إعادة ضبط النموذج
        document.getElementById('asset').value = '';
        document.getElementById('entry').value = '';
        document.getElementById('exit').value = '';
        document.getElementById('risk').value = '';
        document.getElementById('notes').value = '';

        updateUI();
    }

    function updateUI() {
        const tableBody = document.getElementById('tradeTableBody');
        tableBody.innerHTML = '';
        
        let totalPnl = 0;
        let wins = 0;
        let grossWin = 0;
        let grossLoss = 0;
        let equityData = [0];

        // عرض الصفقات من الأحدث للأقدم
        [...trades].reverse().forEach(t => {
            const row = `<tr>
                <td><b>${t.asset}</b></td>
                <td>${t.rr}</td>
                <td class="${t.finalPnl >= 0 ? 'profit' : 'loss'}">${t.finalPnl.toFixed(2)}$</td>
                <td class="note-cell">${t.notes}</td>
            </tr>`;
            tableBody.innerHTML += row;
        });

        // حساب الإحصائيات
        trades.forEach(t => {
            totalPnl += t.finalPnl;
            equityData.push(totalPnl);
            if (t.finalPnl > 0) {
                wins++;
                grossWin += t.finalPnl;
            } else {
                grossLoss += Math.abs(t.finalPnl);
            }
        });

        document.getElementById('count').innerText = trades.length;
        document.getElementById('winRate').innerText = trades.length ? ((wins / trades.length) * 100).toFixed(1) + "%" : "0%";
        document.getElementById('netPnl').innerText = "$" + totalPnl.toFixed(2);
        document.getElementById('pf').innerText = grossLoss > 0 ? (grossWin / grossLoss).toFixed(2) : grossWin.toFixed(1);

        renderChart(equityData);
    }

    function renderChart(data) {
        const ctx = document.getElementById('equityChart').getContext('2d');
        if (myChart) myChart.destroy();
        
        myChart = new Chart(ctx, {
            type: 'line',
            data: {
                labels: data.map((_, i) => i),
                datasets: [{
                    label: 'نمو المحفظة',
                    data: data,
                    borderColor: '#58a6ff',
                    backgroundColor: 'rgba(88, 166, 255, 0.1)',
                    fill: true,
                    tension: 0.3
                }]
            },
            options: {
                responsive: true,
                plugins: { legend: { display: false } },
                scales: {
                    x: { display: false },
                    y: { grid: { color: '#30363d' } }
                }
            }
        });
    }

    function clearData() {
        if (confirm("هل أنت متأكد من حذف جميع بياناتك يا صلاح الدين؟")) {
            localStorage.removeItem('salaheldin_trades');
            trades = [];
            updateUI();
        }
    }

    // التشغيل عند الفتح
    updateUI();
</script>

</body>
</html>
SALAHELDIN TRADING JOURNAL
موقع ويب لتسجيل وتحليل صفقات التداول يدويًا مع إحصائيات ورسم بياني، مجاني ومناسب للجوال.
جميع البيانات محفوظة محليًا في المتصفح.

المميزات
إدخال يدوي كامل للصفقات
إحصائيات تلقائية: عدد الصفقات، Win Rate، صافي الأرباح، Profit Factor
رسم بياني لمنحنى رأس المال (Equity Curve)
سجل صفقات وملاحظات لكل صفقة
متوافق مع الجوال والكمبيوتر
مجاني بالكاملSALAHELDIN TRADING JOURNAL
موقع ويب لتسجيل وتحليل صفقات التداول يدويًا مع إحصائيات ورسم بياني، مجاني ومناسب للجوال.
جميع البيانات محفوظة محليًا في المتصفح.

المميزات
إدخال يدوي كامل للصفقات
إحصائيات تلقائية: عدد الصفقات، Win Rate، صافي الأرباح، Profit Factor
رسم بياني لمنحنى رأس المال (Equity Curve)
سجل صفقات وملاحظات لكل صفقة
متوافق مع الجوال والكمبيوتر
مجاني بالكامل
