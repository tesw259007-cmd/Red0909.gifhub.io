<!DOCTYPE html>
<html lang="zh-TW">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>個人財務管理系統 - 網站版</title>
    <!-- 引入 Chart.js 圖表庫 -->
    <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
    <style>
        :root {
            --primary: #2563eb;
            --primary-hover: #1d4ed8;
            --income: #10b981;
            --expense: #ef4444;
            --bg: #f8fafc;
            --card: #ffffff;
            --border: #e2e8f0;
            --text: #0f172a;
            --text-muted: #64748b;
        }

        * { box-sizing: border-box; margin: 0; padding: 0; font-family: system-ui, -apple-system, sans-serif; }
        body { background-color: var(--bg); color: var(--text); padding: 20px; }

        .dashboard-container {
            max-width: 1200px;
            margin: 0 auto;
            display: flex;
            flex-direction: column;
            gap: 20px;
        }

        /* 頂部數據概覽卡片 */
        .summary-grid {
            display: grid;
            grid-template-columns: repeat(auto-fit, minmax(250px, 1fr));
            gap: 20px;
        }

        .summary-card {
            background: var(--card);
            padding: 20px;
            border-radius: 12px;
            border: 1px solid var(--border);
            box-shadow: 0 1px 3px rgba(0,0,0,0.05);
        }

        .card-title { font-size: 0.875rem; color: var(--text-muted); font-weight: 500; }
        .card-value { font-size: 1.875rem; font-weight: 700; margin-top: 8px; }
        .card-value.income { color: var(--income); }
        .card-value.expense { color: var(--expense); }

        /* 主區域雙欄設計 (適用網頁版) */
        .main-grid {
            display: grid;
            grid-template-columns: 380px 1fr;
            gap: 20px;
        }

        @media (max-width: 900px) {
            .main-grid { grid-template-columns: 1fr; }
        }

        .panel {
            background: var(--card);
            padding: 24px;
            border-radius: 12px;
            border: 1px solid var(--border);
            box-shadow: 0 1px 3px rgba(0,0,0,0.05);
        }

        .panel-title {
            font-size: 1.125rem;
            font-weight: 600;
            margin-bottom: 20px;
            padding-bottom: 10px;
            border-bottom: 2px solid var(--bg);
        }

        /* 表單區域 */
        .form-tabs {
            display: flex;
            background: var(--bg);
            padding: 4px;
            border-radius: 8px;
            margin-bottom: 20px;
        }

        .tab-btn {
            flex: 1;
            padding: 8px;
            border: none;
            background: transparent;
            font-weight: 600;
            color: var(--text-muted);
            cursor: pointer;
            border-radius: 6px;
            transition: all 0.2s;
        }

        .tab-btn.active {
            background: var(--card);
            color: var(--primary);
            box-shadow: 0 1px 2px rgba(0,0,0,0.05);
        }

        .form-group { margin-bottom: 16px; }
        .form-group label { display: block; font-size: 0.875rem; font-weight: 500; margin-bottom: 6px; }
        .form-control {
            width: 100%;
            padding: 10px 14px;
            border: 1px solid var(--border);
            border-radius: 8px;
            font-size: 1rem;
            outline: none;
        }
        .form-control:focus { border-color: var(--primary); }

        .btn-submit {
            width: 100%;
            background: var(--primary);
            color: white;
            border: none;
            padding: 12px;
            border-radius: 8px;
            font-size: 1rem;
            font-weight: 600;
            cursor: pointer;
            transition: background 0.2s;
        }
        .btn-submit:hover { background: var(--primary-hover); }

        /* 右側明細表格與統計 */
        .analytics-grid {
            display: grid;
            grid-template-columns: 1fr 300px;
            gap: 20px;
            margin-top: 20px;
        }

        @media (max-width: 1100px) {
            .analytics-grid { grid-template-columns: 1fr; }
        }

        .table-container { overflow-x: auto; }
        table { width: 100%; border-collapse: collapse; text-align: left; }
        th, td { padding: 12px; border-bottom: 1px solid var(--border); font-size: 0.875rem; }
        th { color: var(--text-muted); font-weight: 600; background: var(--bg); }

        .badge {
            display: inline-block;
            padding: 4px 8px;
            border-radius: 4px;
            font-size: 0.75rem;
            font-weight: 600;
        }
        .badge.expense { background: #fef2f2; color: var(--expense); }
        .badge.income { background: #ecfdf5; color: var(--income); }

        .chart-box {
            display: flex;
            justify-content: center;
            align-items: center;
            height: 250px;
        }
    </style>
</head>
<body>

    <div class="dashboard-container">
        <!-- 頂部卡片列 -->
        <div class="summary-grid">
            <div class="summary-card">
                <div class="card-title">總資產 (現金)</div>
                <div class="card-value" id="val-wallet">NT$ 39,000</div>
            </div>
            <div class="summary-card">
                <div class="card-title">本月總收入</div>
                <div class="card-value income" id="val-income">NT$ 0</div>
            </div>
            <div class="summary-card">
                <div class="card-title">本月總支出</div>
                <div class="card-value expense" id="val-expense">NT$ 0</div>
            </div>
        </div>

        <!-- 主操作區 -->
        <div class="main-grid">
            <!-- 左側：輸入表單 -->
            <div class="panel">
                <div class="panel-title">新增交易記錄</div>
                
                <div class="form-tabs">
                    <button class="tab-btn active" onclick="setType('expense', this)">支出</button>
                    <button class="tab-btn" onclick="setType('income', this)">收入</button>
                </div>

                <form id="record-form" onsubmit="handleFormSubmit(event)">
                    <div class="form-group">
                        <label>日期時間</label>
                        <input type="datetime-local" id="field-date" class="form-control" required>
                    </div>
                    <div class="form-group">
                        <label>金額 (NT$)</label>
                        <input type="number" id="field-amount" class="form-control" placeholder="0" required min="1">
                    </div>
                    <div class="form-group">
                        <label>描述</label>
                        <input type="text" id="field-desc" class="form-control" placeholder="例：午餐、薪水" required>
                    </div>
                    <div class="form-group">
                        <label>類別</label>
                        <select id="field-category" class="form-control"></select>
                    </div>
                    <button type="submit" class="btn-submit">新增紀錄</button>
                </form>
            </div>

            <!-- 右側：資料表與圓餅圖分析 -->
            <div class="panel">
                <div class="panel-title">財務分析與明細</div>
                
                <div class="analytics-grid">
                    <div class="table-container">
                        <table>
                            <thead>
                                <tr>
                                    <th>日期</th>
                                    <th>類別</th>
                                    <th>描述</th>
                                    <th>金額</th>
                                    <th>操作</th>
                                </tr>
                            </thead>
                            <tbody id="table-body">
                                <!-- 動態更新 -->
                            </tbody>
                        </table>
                    </div>

                    <div class="chart-box">
                        <canvas id="expenseChart"></canvas>
                    </div>
                </div>
            </div>
        </div>
    </div>

    <script>
        let currentType = 'expense';
        let balance = JSON.parse(localStorage.getItem('wp_wallet')) || 39000;
        let records = JSON.parse(localStorage.getItem('wp_records')) || [];

        const categories = {
            expense: ['餐飲', '健康', '服裝', '交通', '健身', '娛樂', '禮物', '家具', '寵物', '旅遊', '其他'],
            income: ['工資', '獎金', '津貼', '股利', '投資', '抽獎', '其他']
        };

        window.onload = function() {
            // 初始化時間為當前時間
            const now = new Date();
            now.setMinutes(now.getMinutes() - now.getTimezoneOffset());
            document.getElementById('field-date').value = now.toISOString().slice(0, 16);
            
            updateCategoryOptions();
            updateUI();
            initChart();
        };

        function setType(type, btn) {
            currentType = type;
            document.querySelectorAll('.tab-btn').forEach(b => b.classList.remove('active'));
            btn.classList.add('active');
            updateCategoryOptions();
        }

        function updateCategoryOptions() {
            const select = document.getElementById('field-category');
            select.innerHTML = categories[currentType].map(c => `<option value="${c}">${c}</option>`).join('');
        }

        function handleFormSubmit(e) {
            e.preventDefault();
            const amt = parseFloat(document.getElementById('field-amount').value);
            const desc = document.getElementById('field-desc').value;
            const date = document.getElementById('field-date').value;
            const cat = document.getElementById('field-category').value;

            if (currentType === 'expense') {
                balance -= amt;
            } else {
                balance += amt;
            }

            const record = { id: Date.now(), type: currentType, amount: amt, desc: desc, date: date, category: cat };
            records.unshift(record);

            localStorage.setItem('wp_wallet', balance);
            localStorage.setItem('wp_records', JSON.stringify(records));

            document.getElementById('field-amount').value = '';
            document.getElementById('field-desc').value = '';

            updateUI();
            updateChart();
        }

        function deleteRecord(id) {
            const index = records.findIndex(r => r.id === id);
            if (index !== -1) {
                const r = records[index];
                if (r.type === 'expense') balance += r.amount;
                else balance -= r.amount;

                records.splice(index, 1);
                localStorage.setItem('wp_wallet', balance);
                localStorage.setItem('wp_records', JSON.stringify(records));
                updateUI();
                updateChart();
            }
        }

        function updateUI() {
            document.getElementById('val-wallet').innerText = `NT$ ${balance.toLocaleString()}`;

            let monthInc = 0, monthExp = 0;
            const currentMonth = new Date().toISOString().substring(0, 7);

            const tbody = document.getElementById('table-body');
            tbody.innerHTML = '';

            records.forEach(r => {
                if (r.date.substring(0, 7) === currentMonth) {
                    if (r.type === 'income') monthInc += r.amount;
                    if (r.type === 'expense') monthExp += r.amount;
                }

                const badgeClass = r.type === 'expense' ? 'expense' : 'income';
                const typeSign = r.type === 'expense' ? '-' : '+';

                tbody.innerHTML += `
                    <tr>
                        <td>${r.date.replace('T', ' ')}</td>
                        <td><span class="badge ${badgeClass}">${r.category}</span></td>
                        <td>${r.desc}</td>
                        <td style="font-weight:600;">${typeSign} $${r.amount.toLocaleString()}</td>
                        <td><button onclick="deleteRecord(${r.id})" style="color:red; border:none; background:none; cursor:pointer;">刪除</button></td>
                    </tr>
                `;
            });

            document.getElementById('val-income').innerText = `NT$ ${monthInc.toLocaleString()}`;
            document.getElementById('val-expense').innerText = `NT$ ${monthExp.toLocaleString()}`;
        }

        let myChart;
        function initChart() {
            const ctx = document.getElementById('expenseChart').getContext('2d');
            myChart = new Chart(ctx, {
                type: 'doughnut',
                data: { labels: [], datasets: [{ data: [], backgroundColor: ['#ef4444', '#3b82f6', '#f59e0b', '#10b981', '#8b5cf6', '#ec4899'] }] },
                options: { responsive: true, maintainAspectRatio: false, plugins: { legend: { position: 'bottom' } } }
            });
            updateChart();
        }

        function updateChart() {
            let stats = {};
            records.filter(r => r.type === 'expense').forEach(r => {
                stats[r.category] = (stats[r.category] || 0) + r.amount;
            });

            myChart.data.labels = Object.keys(stats);
            myChart.data.datasets[0].data = Object.values(stats);
            myChart.update();
        }
    </script>
</body>
</html>
