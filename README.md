<html lang="zh-CN">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=yes">
    <title>融资方案配比工具</title>
    <style>
        * { box-sizing: border-box; margin: 0; padding: 0; }
        body { font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, sans-serif; background: #f5f7fa; color: #333; padding: 15px; max-width: 750px; margin: 0 auto; }
        h2 { text-align: center; margin-bottom: 10px; font-size: 1.3em; color: #1a3b5d; }
        .card { background: #fff; border-radius: 12px; padding: 15px; margin-bottom: 15px; box-shadow: 0 2px 12px rgba(0,0,0,0.06); }
        .input-group { margin-bottom: 10px; }
        .input-group label { display: block; font-size: 0.9em; font-weight: 600; margin-bottom: 4px; color: #555; }
        .input-group input { width: 100%; padding: 8px; border: 1px solid #ddd; border-radius: 8px; font-size: 1em; }
        .flex-row { display: flex; gap: 10px; }
        .flex-row > .input-group { flex: 1; }
        .btn { display: block; width: 100%; background: #007aff; color: white; border: none; padding: 10px; border-radius: 8px; font-size: 1em; font-weight: bold; margin-top: 10px; cursor: pointer; }
        .btn:active { opacity: 0.8; }
        .btn-scan { background: #34c759; }
        .note { font-size: 0.8em; color: #888; margin-top: 4px; line-height: 1.4; }
        table { width: 100%; border-collapse: collapse; font-size: 0.8em; margin-top: 10px; }
        th { background: #4f81bd; color: white; padding: 6px; text-align: center; }
        td { padding: 6px; text-align: center; border-bottom: 1px solid #eee; }
        .highlight { background: #e6f2ff; font-weight: bold; }
        .result-badge { text-align: center; font-size: 1em; margin: 10px 0; padding: 8px; border-radius: 8px; }
        .ok { background: #d4edda; color: #155724; }
        .warn { background: #fff3cd; color: #856404; }
        /* 散点图容器及 tooltip */
        .chart-container { position: relative; margin-top: 10px; }
        #scatterTooltip {
            position: absolute;
            background: rgba(0,0,0,0.8);
            color: #fff;
            padding: 4px 8px;
            border-radius: 4px;
            font-size: 12px;
            pointer-events: none;
            display: none;
            white-space: nowrap;
        }
    </style>
</head>
<body>
    <h2>📊 融资方案配比 · 交互工具</h2>

    <!-- 单点计算 -->
    <div class="card">
        <h3>🔢 单点计算</h3>
        <div class="flex-row">
            <div class="input-group"><label>总融资额 T（万元）</label><input type="number" id="T" value="5000" step="100"></div>
            <div class="input-group"><label>新股投前估值（万元）</label><input type="number" id="preMoney" value="45000" step="1000"></div>
        </div>
        <div class="flex-row">
            <div class="input-group"><label>老股转让估值 V_old（万元）</label><input type="number" id="Vold" value="40000" step="1000"></div>
            <div class="input-group"><label>A出让比例（例8%填0.08）</label><input type="number" id="aShare" value="0.08" step="0.01"></div>
        </div>
        <div class="flex-row">
            <div class="input-group"><label>现有总股本（万股）</label><input type="number" id="totalShares" value="10000" step="100"></div>
            <div class="input-group"><label>创始人投前持股（%）</label><input type="number" id="founderPre" value="32.20" step="0.1"></div>
        </div>
        <div class="input-group"><label>创始人最低可接受投后持股（%）</label><input type="number" id="founderMin" value="27.26" step="0.1"></div>
        <button class="btn" onclick="calculate()">计算</button>
        <div id="resultStatus" class="result-badge"></div>
        <table id="resultTable"><thead><tr><th>指标</th><th>数值</th></tr></thead><tbody></tbody></table>
        <div id="founderWarning" style="margin-top:10px; display:none;"></div>
    </div>

    <!-- 一键扫描 -->
    <div class="card">
        <h3>🔍 一键扫描可行方案</h3>
        <div class="note" style="margin-bottom:8px;">自动遍历总融资额、老股估值、出让比例，找出满足“综合价4-4.5”且“创始人持股≥底线”的所有组合。</div>
        <div class="flex-row">
            <div class="input-group"><label>T 融资额列表（逗号分隔，万元）</label><input type="text" id="Tlist" value="4000,5000,10000"></div>
            <div class="input-group"><label>老股估值步长（万元）</label><input type="number" id="Vstep" value="1000" step="500"></div>
        </div>
        <div class="flex-row">
            <div class="input-group"><label>出让比例下限（%）</label><input type="number" id="aMin" value="8" step="0.5"></div>
            <div class="input-group"><label>出让比例上限（%）</label><input type="number" id="aMax" value="12" step="0.5"></div>
            <div class="input-group"><label>出让步长（%）</label><input type="number" id="aStep" value="0.5" step="0.1"></div>
        </div>
        <button class="btn btn-scan" onclick="scanAll()">开始扫描</button>
        <div id="scanInfo" class="note" style="margin-top:8px;"></div>
        <div style="overflow-x:auto;"><table id="scanTable" style="display:none;"><thead><tr><th>T(万)</th><th>老股估值</th><th>A%</th><th>综合成本</th><th>投资人股比</th><th>公司入账</th><th>A套现</th><th>创始人投后</th></tr></thead><tbody></tbody></table></div>
    </div>

    <!-- 散点图（纯 Canvas，增强标签与悬停） -->
    <div class="card">
        <h3>📈 可行域散点图</h3>
        <div class="flex-row">
            <div class="input-group"><label>观察 T 值（万元）</label><input type="number" id="chartT" value="5000" step="100" onchange="drawChart()"></div>
            <div class="input-group"><label>创始人最低投后（%）</label><input type="number" id="chartFounderMin" value="27.26" step="0.1" onchange="drawChart()"></div>
        </div>
        <div class="flex-row">
            <div class="input-group"><label>出让比例下限（%）</label><input type="number" id="chartAMin" value="8" step="0.5" onchange="drawChart()"></div>
            <div class="input-group"><label>出让比例上限（%）</label><input type="number" id="chartAMax" value="12" step="0.5" onchange="drawChart()"></div>
        </div>
        <div class="chart-container">
            <canvas id="scatterCanvas" width="600" height="350"></canvas>
            <div id="scatterTooltip"></div>
        </div>
        <div class="note">● 绿色 = 满足所有约束  ● 灰色 = 不满足  —— 红色虚线：综合成本 4.0 / 4.5</div>
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
    const blended = T / invShares;
    const postShares = shares + newShares;
    const invPct = invShares / postShares;
    const founderPostPct = (shares * (founderPre/100)) / postShares * 100;
    return { oldAmount, newAmount, oldShares, newShares, invShares, blended, postShares, invPct, founderPostPct };
}

// ========== 单次计算 ==========
function calculate() {
    const T = +document.getElementById('T').value;
    const pre = +document.getElementById('preMoney').value;
    const Vold = +document.getElementById('Vold').value;
    const a = +document.getElementById('aShare').value;
    const shares = +document.getElementById('totalShares').value;
    const founderPre = +document.getElementById('founderPre').value;
    const founderMin = +document.getElementById('founderMin').value;
    if (!T || !pre || !Vold || !a || !shares || !founderPre) return alert("请填完所有参数");
    const res = calcOne(T, pre, Vold, a, shares, founderPre);
    const status = (res.blended>=4 && res.blended<=4.5) ? "✅ 综合价可行" : (res.blended<4?"⚠️ 综合价太低":"⚠️ 综合价太高");
    document.getElementById('resultStatus').innerText = status;
    document.getElementById('resultStatus').className = 'result-badge ' + (status.includes('✅')?'ok':'warn');
    let html = `<tr><td>老股每股价格</td><td>${(Vold/shares).toFixed(2)}元</td></tr>
    <tr><td>新股每股价格</td><td>${(pre/shares).toFixed(2)}元</td></tr>
    <tr><td>老股成交金额</td><td>${res.oldAmount.toFixed(0)}万</td></tr>
    <tr><td>新股融资金额</td><td>${res.newAmount.toFixed(0)}万</td></tr>
    <tr class="highlight"><td>综合每股成本</td><td>${res.blended.toFixed(2)}元</td></tr>
    <tr><td>投资人总股数</td><td>${res.invShares.toFixed(2)}万股</td></tr>
    <tr class="highlight"><td>投资人投后股比</td><td>${(res.invPct*100).toFixed(2)}%</td></tr>
    <tr><td>公司入账</td><td>${res.newAmount.toFixed(0)}万</td></tr>
    <tr><td>A套现</td><td>${res.oldAmount.toFixed(0)}万</td></tr>
    <tr style="background:#f0f7ff;"><td>创始人投后持股</td><td><strong>${res.founderPostPct.toFixed(2)}%</strong></td></tr>`;
    document.querySelector('#resultTable tbody').innerHTML = html;
    const warnDiv = document.getElementById('founderWarning');
    if (res.founderPostPct < founderMin) {
        warnDiv.style.display='block'; warnDiv.style.background='#f8d7da'; warnDiv.style.color='#721c24'; warnDiv.style.padding='8px'; warnDiv.style.borderRadius='8px';
        warnDiv.innerText = `⚠️ 创始人投后 ${res.founderPostPct.toFixed(2)}% 低于最低要求 ${founderMin}%`;
    } else warnDiv.style.display='none';
}

// ========== 一键扫描 ==========
function scanAll() {
    const Tlist = document.getElementById('Tlist').value.split(',').map(Number).filter(n=>!isNaN(n));
    const Vstep = +document.getElementById('Vstep').value;
    const aMin = +document.getElementById('aMin').value/100;
    const aMax = +document.getElementById('aMax').value/100;
    const aStep = +document.getElementById('aStep').value/100;
    const pre = +document.getElementById('preMoney').value;
    const shares = +document.getElementById('totalShares').value;
    const founderPre = +document.getElementById('founderPre').value;
    const founderMin = +document.getElementById('founderMin').value;
    if (!Tlist.length) return alert("请输入T列表");
    const results = [];
    for (let T of Tlist) {
        for (let Vold=25000; Vold<=45000; Vold+=Vstep) {
            for (let a=aMin; a<=aMax; a+=aStep) {
                const res = calcOne(T, pre, Vold, a, shares, founderPre);
                if (res.blended<4 || res.blended>4.5) continue;
                if (res.founderPostPct < founderMin) continue;
                if (res.newAmount <= 0) continue;
                results.push({T, Vold, a, ...res});
            }
        }
    }
    results.sort((a,b)=>b.newAmount-a.newAmount);
    const tbody = document.querySelector('#scanTable tbody');
    tbody.innerHTML = results.length ? results.map(r=>`<tr><td>${r.T}</td><td>${r.Vold}</td><td>${(r.a*100).toFixed(1)}%</td><td>${r.blended.toFixed(2)}</td><td>${(r.invPct*100).toFixed(2)}%</td><td>${r.newAmount.toFixed(0)}</td><td>${r.oldAmount.toFixed(0)}</td><td>${r.founderPostPct.toFixed(2)}%</td></tr>`).join('') : '<tr><td colspan="8">无满足所有条件的方案</td></tr>';
    document.getElementById('scanTable').style.display='table';
    document.getElementById('scanInfo').innerText = `✅ 找到 ${results.length} 个可行方案（按公司进账降序）`;
}

// ========== 纯 Canvas 散点图（带轴标签和悬停） ==========
function drawChart() {
    const canvas = document.getElementById('scatterCanvas');
    const ctx = canvas.getContext('2d');
    const W = canvas.width, H = canvas.height;
    ctx.clearRect(0,0,W,H);

    const T = +document.getElementById('chartT').value;
    const pre = +document.getElementById('preMoney').value;
    const founderPre = +document.getElementById('founderPre').value;
    const founderMin = +document.getElementById('chartFounderMin').value;
    const aMin = +document.getElementById('chartAMin').value/100;
    const aMax = +document.getElementById('chartAMax').value/100;
    const shares = 10000;
    if (!T || !pre || !founderPre) return;

    const Vmin = 24000, Vmax = 46000;
    const yMin = 2.5, yMax = 6.5;
    const padLeft = 50, padRight = 20, padTop = 20, padBottom = 40;
    const plotW = W - padLeft - padRight;
    const plotH = H - padTop - padBottom;

    // 绘制背景网格
    ctx.strokeStyle = '#eee'; ctx.lineWidth = 0.5;
    for (let v=25000; v<=45000; v+=5000) {
        const x = padLeft + (v-Vmin)/(Vmax-Vmin)*plotW;
        ctx.beginPath(); ctx.moveTo(x, padTop); ctx.lineTo(x, H-padBottom); ctx.stroke();
    }
    for (let y=3; y<=6; y+=0.5) {
        const yPos = H - padBottom - ((y-yMin)/(yMax-yMin))*plotH;
        ctx.beginPath(); ctx.moveTo(padLeft, yPos); ctx.lineTo(W-padRight, yPos); ctx.stroke();
    }

    // 存储所有点的信息用于悬停检测
    const points = [];

    // 绘制散点
    for (let Vold=25000; Vold<=45000; Vold+=1000) {
        for (let a=aMin; a<=aMax; a+=0.005) {
            const res = calcOne(T, pre, Vold, a, shares, founderPre);
            const feasible = (res.blended>=4 && res.blended<=4.5 && res.founderPostPct>=founderMin);
            const x = padLeft + (Vold-Vmin)/(Vmax-Vmin)*plotW;
            const y = H - padBottom - ((res.blended-yMin)/(yMax-yMin))*plotH;
            ctx.beginPath();
            const radius = feasible ? 4 : 3;
            ctx.arc(x, y, radius, 0, Math.PI*2);
            ctx.fillStyle = feasible ? 'rgba(75,192,192,0.9)' : 'rgba(200,200,200,0.6)';
            ctx.fill();
            // 存储点信息
            points.push({ x, y, Vold, a, blended: res.blended, feasible });
        }
    }

    // 红色虚线 4.0 / 4.5
    ctx.strokeStyle = 'red'; ctx.lineWidth = 1.5; ctx.setLineDash([5,3]);
    [4.0,4.5].forEach(val=>{
        const y = H - padBottom - ((val-yMin)/(yMax-yMin))*plotH;
        ctx.beginPath(); ctx.moveTo(padLeft, y); ctx.lineTo(W-padRight, y); ctx.stroke();
    });
    ctx.setLineDash([]);

    // 坐标轴
    ctx.strokeStyle = '#333'; ctx.lineWidth = 1;
    ctx.beginPath();
    ctx.moveTo(padLeft, padTop); ctx.lineTo(padLeft, H-padBottom); ctx.lineTo(W-padRight, H-padBottom);
    ctx.stroke();

    // X轴刻度标签
    ctx.fillStyle = '#333'; ctx.font = '10px sans-serif'; ctx.textAlign = 'center';
    for (let v=25000; v<=45000; v+=5000) {
        const x = padLeft + (v-Vmin)/(Vmax-Vmin)*plotW;
        ctx.fillText((v/10000).toFixed(1)+'亿', x, H-padBottom+15);
    }
    ctx.fillText('老股估值 V_old（万元）', W/2, H-5);

    // Y轴刻度标签
    ctx.textAlign = 'right';
    for (let y=3; y<=6; y+=0.5) {
        const yPos = H - padBottom - ((y-yMin)/(yMax-yMin))*plotH;
        ctx.fillText(y.toFixed(1), padLeft-8, yPos+3);
    }
    ctx.save();
    ctx.translate(12, H/2);
    ctx.rotate(-Math.PI/2);
    ctx.textAlign = 'center';
    ctx.fillText('综合每股成本（元）', 0, 0);
    ctx.restore();

    // 悬停事件处理
    const tooltip = document.getElementById('scatterTooltip');
    canvas.onmousemove = function(e) {
        const rect = canvas.getBoundingClientRect();
        const scaleX = canvas.width / rect.width;   // canvas 物理像素比
        const scaleY = canvas.height / rect.height;
        const mouseX = (e.clientX - rect.left) * scaleX;
        const mouseY = (e.clientY - rect.top) * scaleY;

        // 寻找最近的点（距离阈值 8px）
        let minDist = 8;
        let closest = null;
        for (let p of points) {
            const dx = mouseX - p.x;
            const dy = mouseY - p.y;
            const dist = Math.sqrt(dx*dx+dy*dy);
            if (dist < minDist) {
                minDist = dist;
                closest = p;
            }
        }
        if (closest) {
            tooltip.style.display = 'block';
            tooltip.style.left = (closest.x / scaleX + 10) + 'px';
            tooltip.style.top = (closest.y / scaleY - 20) + 'px';
            tooltip.innerHTML = `V_old:${closest.Vold}万, a:${(closest.a*100).toFixed(1)}%<br>综合成本:${closest.blended.toFixed(2)}元`;
        } else {
            tooltip.style.display = 'none';
        }
    };
    canvas.onmouseleave = function() {
        tooltip.style.display = 'none';
    };
}
window.onload = function(){ calculate(); drawChart(); };
</script>
</body>
</html>
