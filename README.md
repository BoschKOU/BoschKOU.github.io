<!DOCTYPE html>
<html lang="zh-CN">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=yes">
    <title>融资方案配比 · 投前估值版</title>
    <script src="https://cdn.jsdelivr.net/npm/chart.js@4.4.0/dist/chart.umd.min.js"></script>
    <style>
        * { box-sizing: border-box; margin: 0; padding: 0; }
        body { font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, Helvetica, Arial, sans-serif; background: #f5f7fa; color: #333; padding: 15px; max-width: 750px; margin: 0 auto; }
        h2 { text-align: center; margin-bottom: 10px; font-size: 1.3em; color: #1a3b5d; }
        .card { background: #fff; border-radius: 12px; padding: 15px; margin-bottom: 15px; box-shadow: 0 2px 12px rgba(0,0,0,0.06); }
        .input-group { margin-bottom: 10px; }
        .input-group label { display: block; font-size: 0.9em; font-weight: 600; margin-bottom: 4px; color: #555; }
        .input-group input, .input-group select { width: 100%; padding: 8px; border: 1px solid #ddd; border-radius: 8px; font-size: 1em; }
        .flex-row { display: flex; gap: 10px; }
        .flex-row > .input-group { flex: 1; }
        .btn { display: block; width: 100%; background: #007aff; color: white; border: none; padding: 10px; border-radius: 8px; font-size: 1em; font-weight: bold; margin-top: 10px; cursor: pointer; }
        .btn:active { opacity: 0.8; }
        .btn-scan { background: #34c759; }
        .note { font-size: 0.8em; color: #888; margin-top: 4px; line-height: 1.4; }
        canvas { max-width: 100%; height: auto; margin-top: 10px; }
        table { width: 100%; border-collapse: collapse; font-size: 0.8em; margin-top: 10px; }
        th { background: #4f81bd; color: white; padding: 6px; text-align: center; }
        td { padding: 6px; text-align: center; border-bottom: 1px solid #eee; }
        .highlight { background: #e6f2ff; font-weight: bold; }
        .result-badge { text-align: center; font-size: 1em; margin: 10px 0; padding: 8px; border-radius: 8px; }
        .ok { background: #d4edda; color: #155724; }
        .warn { background: #fff3cd; color: #856404; }
        hr { margin: 15px 0; border: 0.5px solid #ddd; }
    </style>
</head>
<body>
    <h2>📊 融资方案配比 · 聚焦投前估值</h2>

    <!-- ========== 1. 单次计算卡片 ========== -->
    <div class="card">
        <h3>🔢 单点计算</h3>
        <div class="flex-row">
            <div class="input-group">
                <label>总融资额 T（万元）</label>
                <input type="number" id="T" value="5000" step="100">
            </div>
            <div class="input-group">
                <label>新股投前估值（万元）</label>
                <input type="number" id="preMoney" value="45000" step="1000">
            </div>
        </div>
        <div class="flex-row">
            <div class="input-group">
                <label>老股转让估值 V_old（万元）</label>
                <input type="number" id="Vold" value="40000" step="1000">
            </div>
            <div class="input-group">
                <label>A出让比例（例8%填0.08）</label>
                <input type="number" id="aShare" value="0.08" step="0.01">
            </div>
        </div>
        <div class="flex-row">
            <div class="input-group">
                <label>现有总股本（万股，建议固定10000）</label>
                <input type="number" id="totalShares" value="10000" step="100">
            </div>
            <div class="input-group">
                <label>创始人投前持股（%）</label>
                <input type="number" id="founderPre" value="32.20" step="0.1">
            </div>
        </div>
        <div class="input-group">
            <label>创始人最低可接受投后持股（%）</label>
            <input type="number" id="founderMin" value="27.26" step="0.1">
        </div>
        <button class="btn" onclick="calculate()">计算</button>
        <div id="resultStatus" class="result-badge"></div>
        <table id="resultTable"><thead><tr><th>指标</th><th>数值</th></tr></thead><tbody></tbody></table>
        <div id="founderWarning" style="margin-top:10px; display:none;"></div>
    </div>

    <!-- ========== 2. 一键扫描全部可行方案 ========== -->
    <div class="card">
        <h3>🔍 一键扫描可行方案（按投前估值4~4.5亿筛选）</h3>
        <div class="note" style="margin-bottom:10px;">
            💡 <strong>通俗解释：</strong> 计算机自动尝试各种可能的总融资额、老股估值、出让比例组合，<br>
            找出所有满足“综合投前估值在4~4.5亿”且“创始人持股≥底线”的方案。<br>
            下面的“步长”就是网格的精细度：步长越小，检查越密，结果越完整，但计算时间稍长。
        </div>
        <div class="flex-row">
            <div class="input-group">
                <label>T 融资额列表（逗号分隔，万元）</label>
                <input type="text" id="Tlist" value="4000,5000,10000">
            </div>
            <div class="input-group">
                <label>老股估值步长（万元，推荐500或1000）</label>
                <input type="number" id="Vstep" value="1000" step="500">
            </div>
        </div>
        <div class="flex-row">
            <div class="input-group">
                <label>出让比例下限（%）</label>
                <input type="number" id="aMin" value="8" step="0.5">
            </div>
            <div class="input-group">
                <label>出让比例上限（%）</label>
                <input type="number" id="aMax" value="12" step="0.5">
            </div>
            <div class="input-group">
                <label>出让步长（%，推荐0.5）</label>
                <input type="number" id="aStep" value="0.5" step="0.1">
            </div>
        </div>
        <button class="btn btn-scan" onclick="scanAll()">开始扫描可行方案</button>
        <div id="scanInfo" class="note" style="margin-top:8px;"></div>
        <div style="overflow-x:auto;">
            <table id="scanTable" style="display:none;">
                <thead>
                    <tr>
                        <th>T(万)</th><th>老股估值(万)</th><th>A出让%</th>
                        <th>综合每股成本(元)</th>
                        <th>投后估值(万)</th>
                        <th>投前估值(万)</th>
                        <th>投资人股比</th>
                        <th>公司入账(万)</th>
                        <th>A套现(万)</th>
                        <th>创始人投后</th>
                    </tr>
                </thead>
                <tbody></tbody>
            </table>
        </div>
    </div>

    <!-- ========== 3. 散点图（独立控制） ========== -->
    <div class="card">
        <h3>📈 可行域散点图（投前估值视角）</h3>
        <div class="note" style="margin-bottom:10px;">
            选择你想观察的总融资额 T，图表显示不同老股估值和出让比例下的综合投前估值（万元）。<br>
            绿色点 = 满足投前估值4~4.5亿且创始人持股≥底线。
        </div>
        <div class="flex-row">
            <div class="input-group">
                <label>观察的 T 值（万元）</label>
                <select id="chartT" onchange="drawChart()">
                    <option value="4000">4000</option>
                    <option value="5000" selected>5000</option>
                    <option value="10000">10000</option>
                </select>
            </div>
            <div class="input-group">
                <label>创始人最低投后（%）</label>
                <input type="number" id="chartFounderMin" value="27.26" step="0.1" onchange="drawChart()">
            </div>
        </div>
        <div class="flex-row">
            <div class="input-group">
                <label>出让比例下限（%）</label>
                <input type="number" id="chartAMin" value="8" step="0.5" onchange="drawChart()">
            </div>
            <div class="input-group">
                <label>出让比例上限（%）</label>
                <input type="number" id="chartAMax" value="12" step="0.5" onchange="drawChart()">
            </div>
        </div>
        <canvas id="feasibleChart" width="400" height="300"></canvas>
        <div class="note">红色虚线：投前估值 4.0亿 和 4.5亿。绿点：通过所有约束。</div>
    </div>

    <script>
        // ========== 通用计算函数 ==========
        function calcOne(T, pre, Vold, a, shares, founderPre) {
            const oldAmount = a * Vold;
            const newAmount = T - oldAmount;
            const newPrice = pre / shares;
            const oldShares = a * shares;
            const newShares = newAmount / newPrice;
            const invShares = oldShares + newShares;
            const blended = T / invShares; // 综合每股成本（元/股）
            const postShares = shares + newShares;
            const invPct = invShares / postShares;
            const founderPreShares = shares * (founderPre / 100);
            const founderPostPct = (founderPreShares / postShares) * 100;
            // 计算投后估值（万元）= 综合每股成本 * 总股本（万股）
            const postValuation = blended * shares;
            // 投前估值（万元）= 投后估值 - 总融资额
            const preValuation = postValuation - T;
            return {
                T, pre, Vold, a, shares, founderPre,
                oldAmount, newAmount, oldShares, newShares,
                invShares, blended, postShares, invPct, founderPostPct,
                postValuation, preValuation
            };
        }

        // 判断状态（基于投前估值是否在4~4.5亿）
        function judgeByPre(preVal) {
            if (preVal >= 40000 && preVal <= 45000) return "✅ 投前估值可行";
            if (preVal < 40000) return "⚠️ 投前估值偏低（＜4亿）";
            return "⚠️ 投前估值偏高（＞4.5亿）";
        }

        // ========== 单次计算 ==========
        function calculate() {
            const T = parseFloat(document.getElementById('T').value);
            const pre = parseFloat(document.getElementById('preMoney').value);
            const Vold = parseFloat(document.getElementById('Vold').value);
            const a = parseFloat(document.getElementById('aShare').value);
            const shares = parseFloat(document.getElementById('totalShares').value);
            const founderPre = parseFloat(document.getElementById('founderPre').value);
            const founderMin = parseFloat(document.getElementById('founderMin').value);

            if (isNaN(T) || isNaN(pre) || isNaN(Vold) || isNaN(a) || isNaN(shares) || isNaN(founderPre) || shares <= 0) {
                alert("请填写所有参数");
                return;
            }

            const res = calcOne(T, pre, Vold, a, shares, founderPre);
            const status = judgeByPre(res.preValuation);
            const statusDiv = document.getElementById('resultStatus');
            statusDiv.innerText = status;
            statusDiv.className = 'result-badge ' + (status.includes('可行') ? 'ok' : 'warn');

            const tbody = document.querySelector('#resultTable tbody');
            tbody.innerHTML = `
                <tr><td>老股每股价格（元）</td><td>${(Vold/shares).toFixed(2)}</td></tr>
                <tr><td>新股每股价格（元）</td><td>${(pre/shares).toFixed(2)}</td></tr>
                <tr><td>老股成交金额（万元）</td><td>${res.oldAmount.toFixed(0)}</td></tr>
                <tr><td>新股融资金额（万元）</td><td>${res.newAmount.toFixed(0)}</td></tr>
                <tr class="highlight"><td>综合每股成本（元）</td><td>${res.blended.toFixed(2)}</td></tr>
                <tr class="highlight"><td>综合投后估值（万元）</td><td>${res.postValuation.toFixed(0)}</td></tr>
                <tr class="highlight"><td>综合投前估值（万元）</td><td>${res.preValuation.toFixed(0)}</td></tr>
                <tr><td>投资人总获股数（万股）</td><td>${res.invShares.toFixed(2)}</td></tr>
                <tr class="highlight"><td>投资人投后股比</td><td>${(res.invPct*100).toFixed(2)}%</td></tr>
                <tr><td>公司入账资金（万元）</td><td>${res.newAmount.toFixed(0)}</td></tr>
                <tr><td>老股东A套现金额（万元）</td><td>${res.oldAmount.toFixed(0)}</td></tr>
                <tr style="background:#f0f7ff;"><td>创始人投后持股</td><td><strong>${res.founderPostPct.toFixed(2)}%</strong></td></tr>
            `;

            // 创始人最低线警告
            const warningDiv = document.getElementById('founderWarning');
            if (res.founderPostPct < founderMin) {
                warningDiv.style.display = 'block';
                warningDiv.style.background = '#f8d7da';
                warningDiv.style.color = '#721c24';
                warningDiv.style.padding = '8px';
                warningDiv.style.borderRadius = '8px';
                warningDiv.innerText = `⚠️ 创始人投后 ${res.founderPostPct.toFixed(2)}% 低于最低要求 ${founderMin}%`;
            } else {
                warningDiv.style.display = 'none';
            }
        }

        // ========== 一键扫描全部可行方案 ==========
        function scanAll() {
            const TlistStr = document.getElementById('Tlist').value;
            const Tarr = TlistStr.split(',').map(Number).filter(n => !isNaN(n));
            const Vstep = parseFloat(document.getElementById('Vstep').value);
            const aMin = parseFloat(document.getElementById('aMin').value) / 100;
            const aMax = parseFloat(document.getElementById('aMax').value) / 100;
            const aStep = parseFloat(document.getElementById('aStep').value) / 100;
            const pre = parseFloat(document.getElementById('preMoney').value);
            const shares = parseFloat(document.getElementById('totalShares').value);
            const founderPre = parseFloat(document.getElementById('founderPre').value);
            const founderMin = parseFloat(document.getElementById('founderMin').value);

            if (Tarr.length === 0 || isNaN(Vstep) || isNaN(aMin) || isNaN(aMax) || isNaN(aStep)) {
                alert("请正确填写扫描参数");
                return;
            }

            const results = [];
            for (let T of Tarr) {
                for (let Vold = 25000; Vold <= 45000; Vold += Vstep) {
                    for (let a = aMin; a <= aMax; a += aStep) {
                        const res = calcOne(T, pre, Vold, a, shares, founderPre);
                        // 核心判定：综合投前估值在4~4.5亿（即40000~45000万元）
                        if (res.preValuation < 40000 || res.preValuation > 45000) continue;
                        if (res.founderPostPct < founderMin) continue;
                        if (res.newAmount <= 0) continue;
                        results.push(res);
                    }
                }
            }

            // 按投前估值排序（最接近4.25亿的排前面）
            results.sort((a,b) => Math.abs(a.preValuation - 42500) - Math.abs(b.preValuation - 42500));

            const tbody = document.querySelector('#scanTable tbody');
            tbody.innerHTML = '';
            if (results.length === 0) {
                tbody.innerHTML = '<tr><td colspan="10">没有找到满足所有条件的方案，请放宽约束（如降低创始人最低持股、扩大出让比例范围等）。</td></tr>';
            } else {
                for (let r of results) {
                    tbody.innerHTML += `<tr>
                        <td>${r.T}</td>
                        <td>${r.Vold}</td>
                        <td>${(r.a*100).toFixed(1)}%</td>
                        <td>${r.blended.toFixed(2)}</td>
                        <td>${r.postValuation.toFixed(0)}</td>
                        <td>${r.preValuation.toFixed(0)}</td>
                        <td>${(r.invPct*100).toFixed(2)}%</td>
                        <td>${r.newAmount.toFixed(0)}</td>
                        <td>${r.oldAmount.toFixed(0)}</td>
                        <td>${r.founderPostPct.toFixed(2)}%</td>
                    </tr>`;
                }
            }
            document.getElementById('scanTable').style.display = 'table';
            document.getElementById('scanInfo').innerText = `✅ 找到 ${results.length} 个可行方案，按投前估值最接近4.25亿排序。`;
        }

        // ========== 散点图（投前估值视角） ==========
        let chartInstance = null;
        function drawChart() {
            const T = parseFloat(document.getElementById('chartT').value);
            const pre = parseFloat(document.getElementById('preMoney').value);
            const founderPre = parseFloat(document.getElementById('founderPre').value);
            const founderMin = parseFloat(document.getElementById('chartFounderMin').value);
            const aMin = parseFloat(document.getElementById('chartAMin').value) / 100;
            const aMax = parseFloat(document.getElementById('chartAMax').value) / 100;
            const shares = parseFloat(document.getElementById('totalShares').value); // 同步读取

            const VoldList = [];
            for (let v = 25000; v <= 45000; v += 1000) VoldList.push(v);
            const aList = [];
            for (let a = aMin; a <= aMax; a += 0.005) aList.push(a);

            const datasets = [];
            for (let a of aList) {
                const feasiblePoints = [];
                const unfeasiblePoints = [];
                for (let Vold of VoldList) {
                    const res = calcOne(T, pre, Vold, a, shares, founderPre);
                    const feasible = (res.preValuation >= 40000 && res.preValuation <= 45000 && res.founderPostPct >= founderMin);
                    if (feasible) {
                        feasiblePoints.push({x: Vold, y: res.preValuation});
                    } else {
                        unfeasiblePoints.push({x: Vold, y: res.preValuation});
                    }
                }
                if (feasiblePoints.length > 0) {
                    datasets.push({
                        label: `a=${(a*100).toFixed(1)}% 可行`,
                        data: feasiblePoints,
                        backgroundColor: 'rgba(75,192,192,0.9)',
                        pointRadius: 5,
                        showLine: false,
                        type: 'scatter'
                    });
                }
                if (unfeasiblePoints.length > 0) {
                    datasets.push({
                        label: `a=${(a*100).toFixed(1)}% 不可行`,
                        data: unfeasiblePoints,
                        backgroundColor: 'rgba(200,200,200,0.5)',
                        pointRadius: 3,
                        showLine: false,
                        type: 'scatter'
                    });
                }
            }

            const ctx = document.getElementById('feasibleChart').getContext('2d');
            if (chartInstance) chartInstance.destroy();
            chartInstance = new Chart(ctx, {
                type: 'scatter',
                data: { datasets },
                options: {
                    plugins: {
                        tooltip: {
                            callbacks: {
                                label: (ctx) => `老股估值:${ctx.raw.x}万, 投前估值:${ctx.raw.y.toFixed(0)}万`
                            }
                        },
                        legend: { display: false }
                    },
                    scales: {
                        x: {
                            title: { display: true, text: '老股估值 V_old（万元）' },
                            min: 24000, max: 46000
                        },
                        y: {
                            title: { display: true, text: '综合投前估值（万元）' },
                            min: 30000, max: 55000
                        }
                    }
                },
                plugins: [{
                    id: 'refLines',
                    afterDraw: (chart) => {
                        const { ctx, scales: { x, y } } = chart;
                        ctx.save();
                        ctx.setLineDash([5,5]);
                        ctx.strokeStyle = 'red';
                        ctx.lineWidth = 1.5;
                        [40000, 45000].forEach(val => {
                            ctx.beginPath();
                            ctx.moveTo(x.left, y.getPixelForValue(val));
                            ctx.lineTo(x.right, y.getPixelForValue(val));
                            ctx.stroke();
                        });
                        ctx.restore();
                    }
                }]
            });
        }

        // 页面加载时初始化
        window.onload = function() {
            calculate();
            drawChart();
        };
    </script>
</body>
</html>
