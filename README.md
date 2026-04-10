<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Cognos Exploration | Age Group vs Package Type Analysis</title>
    <script src="https://cdn.jsdelivr.net/npm/chart.js@4.4.0/dist/chart.umd.min.js"></script>
    <style>
        * {
            margin: 0;
            padding: 0;
            box-sizing: border-box;
        }

        body {
            font-family: 'Segoe UI', 'Arial', sans-serif;
            background: linear-gradient(135deg, #0f172a 0%, #1e293b 100%);
            padding: 30px 20px;
        }

        .exploration-container {
            max-width: 1400px;
            margin: 0 auto;
            background: white;
            border-radius: 24px;
            box-shadow: 0 25px 50px -12px rgba(0,0,0,0.25);
            overflow: hidden;
        }

        /* Header */
        .exploration-header {
            background: linear-gradient(135deg, #0a2b3e 0%, #1a4a6f 100%);
            color: white;
            padding: 30px 40px;
            text-align: center;
        }

        .exploration-header h1 {
            font-size: 28px;
            margin-bottom: 8px;
        }

        .exploration-header p {
            opacity: 0.85;
            font-size: 14px;
        }

        .dataset-badge {
            display: inline-block;
            background: rgba(255,255,255,0.2);
            padding: 5px 15px;
            border-radius: 20px;
            font-size: 12px;
            margin-top: 12px;
        }

        /* Filter Section - Season Toggle */
        .filter-section {
            background: #f8fafc;
            padding: 20px 40px;
            border-bottom: 1px solid #e2e8f0;
            display: flex;
            align-items: center;
            justify-content: center;
            gap: 30px;
            flex-wrap: wrap;
        }

        .season-toggle {
            display: flex;
            gap: 15px;
            background: white;
            padding: 8px 20px;
            border-radius: 40px;
            border: 1px solid #cbd5e1;
        }

        .season-toggle label {
            display: flex;
            align-items: center;
            gap: 8px;
            cursor: pointer;
            font-size: 14px;
            font-weight: 600;
            padding: 8px 20px;
            border-radius: 30px;
            transition: all 0.3s;
        }

        .season-toggle input[type="radio"] {
            display: none;
        }

        .season-toggle label:has(input:checked) {
            background: #2563eb;
            color: white;
        }

        .season-toggle label:has(input:not(:checked)) {
            color: #475569;
        }

        .insight-badge {
            background: #eef2ff;
            padding: 8px 20px;
            border-radius: 30px;
            font-size: 13px;
            font-weight: 600;
            color: #2563eb;
        }

        /* KPI Row */
        .kpi-row {
            display: grid;
            grid-template-columns: repeat(3, 1fr);
            gap: 20px;
            padding: 30px 40px;
            background: #f8fafc;
            border-bottom: 1px solid #e2e8f0;
        }

        .kpi-card {
            background: white;
            border-radius: 16px;
            padding: 20px;
            text-align: center;
            border: 1px solid #e2e8f0;
        }

        .kpi-value {
            font-size: 32px;
            font-weight: 800;
            color: #1e293b;
        }

        .kpi-label {
            font-size: 12px;
            color: #64748b;
            margin-top: 5px;
        }

        /* Charts Layout */
        .charts-row {
            display: grid;
            grid-template-columns: 1fr 1fr;
            gap: 30px;
            padding: 30px 40px;
            border-bottom: 1px solid #e2e8f0;
        }

        .chart-card {
            background: white;
            border: 1px solid #e2e8f0;
            border-radius: 20px;
            padding: 20px;
        }

        .chart-title {
            font-size: 16px;
            font-weight: 700;
            color: #1e293b;
            margin-bottom: 16px;
            border-left: 4px solid #2563eb;
            padding-left: 12px;
        }

        canvas {
            max-height: 300px;
            width: 100%;
        }

        /* Data Table */
        .table-container {
            padding: 0 40px 30px 40px;
        }

        .data-table {
            width: 100%;
            border-collapse: collapse;
            margin: 20px 0;
        }

        .data-table th, .data-table td {
            padding: 12px;
            text-align: left;
            border-bottom: 1px solid #e2e8f0;
        }

        .data-table th {
            background: #f8fafc;
            font-weight: 700;
            color: #1e293b;
        }

        .highlight-row {
            background: #fefce8;
            font-weight: 700;
        }

        /* Annotation Box */
        .annotation-box {
            background: #eff6ff;
            border-left: 4px solid #2563eb;
            padding: 20px 30px;
            margin: 0 40px 30px 40px;
            border-radius: 12px;
        }

        .annotation-title {
            font-weight: 700;
            font-size: 16px;
            margin-bottom: 10px;
            color: #1e293b;
        }

        .conclusion-box {
            background: #dcfce7;
            border-left: 4px solid #10b981;
            margin: 0 40px 30px 40px;
            padding: 20px 30px;
            border-radius: 12px;
        }

        /* Footer */
        .exploration-footer {
            background: #f1f5f9;
            padding: 15px 40px;
            text-align: center;
            font-size: 10px;
            color: #64748b;
            border-top: 1px solid #e2e8f0;
        }

        @media (max-width: 900px) {
            .charts-row { grid-template-columns: 1fr; }
            .kpi-row { grid-template-columns: 1fr; }
        }

        @media print {
            body { background: white; padding: 0; }
            .filter-section { break-inside: avoid; }
            .kpi-card { break-inside: avoid; }
            .chart-card { break-inside: avoid; }
        }
    </style>
</head>
<body>
<div class="exploration-container">
    <!-- HEADER -->
    <div class="exploration-header">
        <h1>🔍 Cognos Analytics Exploration</h1>
        <p>Customer Age Group vs Package Type Analysis | SkyWings Booking Dataset</p>
        <div class="dataset-badge">📊 Dataset: 8,452 bookings | Period: Apr 2025 - Mar 2026</div>
    </div>

    <!-- SEASON FILTER TOGGLE (Exploration Prompt) -->
    <div class="filter-section">
        <span class="insight-badge">🎛️ Exploration Filter</span>
        <div class="season-toggle">
            <label>
                <input type="radio" name="season" value="peak" checked> 
                <span>🔴 PEAK SEASON (Dec-Jan, Jun-Jul)</span>
            </label>
            <label>
                <input type="radio" name="season" value="offpeak"> 
                <span>🟡 OFF-PEAK SEASON (Feb-May, Aug-Nov)</span>
            </label>
        </div>
        <span class="insight-badge">Click to compare seasons</span>
    </div>

    <!-- KPI ROW -->
    <div class="kpi-row" id="kpiRow">
        <!-- Dynamic KPI -->
    </div>

    <!-- CHARTS ROW -->
    <div class="charts-row">
        <div class="chart-card">
            <div class="chart-title">📊 Revenue by Age Group & Package Type</div>
            <canvas id="stackedBarChart"></canvas>
        </div>
        <div class="chart-card">
            <div class="chart-title">📈 Revenue by Age Group (Line Chart)</div>
            <canvas id="lineChart"></canvas>
        </div>
    </div>

    <!-- DATA TABLE -->
    <div class="table-container">
        <div class="chart-title">📋 Detailed Revenue Breakdown by Age Group</div>
        <table class="data-table" id="dataTable">
            <thead>
                <tr>
                    <th>Age Group</th>
                    <th>Flight (₹ Cr)</th>
                    <th>Hotel (₹ Cr)</th>
                    <th>Combo (₹ Cr)</th>
                    <th>Total Revenue (₹ Cr)</th>
                    <th>Primary Package</th>
                </tr>
            </thead>
            <tbody id="tableBody">
            </tbody>
        </table>
    </div>

    <!-- ANNOTATION / INSIGHT BOX -->
    <div class="annotation-box" id="annotationBox">
        <div class="annotation-title">📌 EXPLORATION INSIGHT</div>
        <div id="insightText">Loading...</div>
    </div>

    <!-- CONCLUSION BOX -->
    <div class="conclusion-box" id="conclusionBox">
        <div class="annotation-title">✅ FINAL ANSWER</div>
        <div id="conclusionText">Loading...</div>
    </div>

    <!-- FOOTER -->
    <div class="exploration-footer">
        <span>Cognos Analytics Exploration | Age Group vs Package Type | Page 1 of 1</span>
    </div>
</div>

<script>
    // ============================================
    // DATA FOR PEAK SEASON
    // ============================================
    const peakData = {
        ageGroups: ['18-25', '26-35', '36-45', '46-55', '55+'],
        flight: [3.2, 2.8, 2.5, 2.2, 1.8],
        hotel: [0.8, 1.6, 2.0, 2.3, 2.4],
        combo: [0.8, 11.8, 8.3, 6.0, 4.0],
        totals: [4.8, 16.2, 12.8, 10.5, 8.2],
        primary: ['Flight', 'Combo', 'Combo', 'Combo', 'Combo'],
        primaryPct: [65, 72, 68, 65, 58]
    };

    // ============================================
    // DATA FOR OFF-PEAK SEASON
    // ============================================
    const offpeakData = {
        ageGroups: ['18-25', '26-35', '36-45', '46-55', '55+'],
        flight: [2.0, 1.8, 1.5, 1.2, 1.0],
        hotel: [0.5, 1.0, 1.5, 1.8, 2.0],
        combo: [0.4, 4.0, 5.2, 6.8, 8.4],
        totals: [2.9, 6.8, 8.2, 9.8, 11.4],
        primary: ['Flight', 'Combo', 'Combo', 'Combo', 'Combo'],
        primaryPct: [68, 58, 64, 70, 74]
    };

    let currentSeason = 'peak';
    let stackedChart, lineChart;

    // Format currency
    function formatCr(amount) {
        return '₹' + amount.toFixed(1) + ' Cr';
    }

    // Update KPIs
    function updateKPIs(data, seasonName) {
        const totalRevenue = data.totals.reduce((a,b) => a+b, 0);
        const topGroup = data.ageGroups[data.totals.indexOf(Math.max(...data.totals))];
        const topRevenue = Math.max(...data.totals);
        const topPackage = data.primary[data.totals.indexOf(Math.max(...data.totals))];
        const topPct = data.primaryPct[data.totals.indexOf(Math.max(...data.totals))];

        document.getElementById('kpiRow').innerHTML = `
            <div class="kpi-card">
                <div class="kpi-value">${formatCr(totalRevenue)}</div>
                <div class="kpi-label">Total Revenue (${seasonName})</div>
            </div>
            <div class="kpi-card">
                <div class="kpi-value">${topGroup}</div>
                <div class="kpi-label">🏆 Highest Revenue Age Group</div>
            </div>
            <div class="kpi-card">
                <div class="kpi-value">${topPackage} (${topPct}%)</div>
                <div class="kpi-label">Primary Package Type</div>
            </div>
        `;
    }

    // Update Stacked Bar Chart
    function updateStackedChart(data) {
        if (stackedChart) stackedChart.destroy();
        const ctx = document.getElementById('stackedBarChart').getContext('2d');
        stackedChart = new Chart(ctx, {
            type: 'bar',
            data: {
                labels: data.ageGroups,
                datasets: [
                    { label: 'Flight Only', data: data.flight, backgroundColor: '#3b82f6', borderRadius: 4 },
                    { label: 'Hotel Only', data: data.hotel, backgroundColor: '#10b981', borderRadius: 4 },
                    { label: 'Combo Package', data: data.combo, backgroundColor: '#f59e0b', borderRadius: 4 }
                ]
            },
            options: {
                responsive: true,
                maintainAspectRatio: true,
                plugins: {
                    tooltip: { callbacks: { label: (ctx) => `${ctx.dataset.label}: ₹${ctx.raw} Cr` } },
                    legend: { position: 'top' }
                },
                scales: {
                    x: { stacked: true, title: { display: true, text: 'Age Group' } },
                    y: { stacked: true, title: { display: true, text: 'Revenue (₹ Cr)' }, beginAtZero: true }
                }
            }
        });
    }

    // Update Line Chart
    function updateLineChart(data, seasonName) {
        if (lineChart) lineChart.destroy();
        const ctx = document.getElementById('lineChart').getContext('2d');
        lineChart = new Chart(ctx, {
            type: 'line',
            data: {
                labels: data.ageGroups,
                datasets: [{
                    label: `Revenue by Age Group (${seasonName})`,
                    data: data.totals,
                    borderColor: '#ef4444',
                    backgroundColor: 'rgba(239, 68, 68, 0.1)',
                    borderWidth: 3,
                    fill: true,
                    pointRadius: 6,
                    pointBackgroundColor: '#dc2626'
                }]
            },
            options: {
                responsive: true,
                maintainAspectRatio: true,
                plugins: {
                    tooltip: { callbacks: { label: (ctx) => `Revenue: ₹${ctx.raw} Cr` } }
                },
                scales: {
                    y: { beginAtZero: true, title: { display: true, text: 'Revenue (₹ Cr)' } }
                }
            }
        });
    }

    // Update Data Table
    function updateTable(data) {
        const tbody = document.getElementById('tableBody');
        let rows = '';
        let highestRevenue = Math.max(...data.totals);
        
        for (let i = 0; i < data.ageGroups.length; i++) {
            const isHighest = data.totals[i] === highestRevenue;
            rows += `
                <tr ${isHighest ? 'class="highlight-row"' : ''}>
                    <td><strong>${data.ageGroups[i]}</strong></td>
                    <td>${formatCr(data.flight[i])}</td>
                    <td>${formatCr(data.hotel[i])}</td>
                    <td>${formatCr(data.combo[i])}</td>
                    <td><strong>${formatCr(data.totals[i])}</strong></td>
                    <td>${data.primary[i]} (${data.primaryPct[i]}%)</td>
                </tr>
            `;
        }
        
        // Add total row
        const totalFlight = data.flight.reduce((a,b) => a+b, 0);
        const totalHotel = data.hotel.reduce((a,b) => a+b, 0);
        const totalCombo = data.combo.reduce((a,b) => a+b, 0);
        const grandTotal = data.totals.reduce((a,b) => a+b, 0);
        
        rows += `
            <tr style="background:#f1f5f9; font-weight:700;">
                <td><strong>TOTAL</strong></td>
                <td>${formatCr(totalFlight)}</td>
                <td>${formatCr(totalHotel)}</td>
                <td>${formatCr(totalCombo)}</td>
                <td>${formatCr(grandTotal)}</td>
                <td>-</td>
            </tr>
        `;
        
        tbody.innerHTML = rows;
    }

    // Update Annotation Box
    function updateAnnotation(data, seasonName) {
        const topGroup = data.ageGroups[data.totals.indexOf(Math.max(...data.totals))];
        const topRevenue = Math.max(...data.totals);
        const topPackage = data.primary[data.totals.indexOf(Math.max(...data.totals))];
        const topPct = data.primaryPct[data.totals.indexOf(Math.max(...data.totals))];
        const secondGroup = data.ageGroups[data.totals.indexOf([...data.totals].sort((a,b)=>b-a)[1])];
        
        document.getElementById('insightText').innerHTML = `
            <strong>During ${seasonName}:</strong> Age group <strong>${topGroup}</strong> generates the highest revenue with <strong>${formatCr(topRevenue)}</strong>, 
            primarily through <strong>${topPackage} packages (${topPct}%)</strong>. 
            The second highest contributor is age group <strong>${secondGroup}</strong>. 
            Combo packages dominate across all age groups, contributing ${((data.combo.reduce((a,b)=>a+b,0)/data.totals.reduce((a,b)=>a+b,0))*100).toFixed(0)}% of total revenue.
        `;
    }

    // Update Conclusion Box
    function updateConclusion(peakData, offpeakData) {
        const peakTop = peakData.ageGroups[peakData.totals.indexOf(Math.max(...peakData.totals))];
        const peakRevenue = Math.max(...peakData.totals);
        const offpeakTop = offpeakData.ageGroups[offpeakData.totals.indexOf(Math.max(...offpeakData.totals))];
        const offpeakRevenue = Math.max(...offpeakData.totals);
        
        document.getElementById('conclusionText').innerHTML = `
            <table style="width:100%; border-collapse:collapse;">
                <tr style="background:#166534; color:white;">
                    <th style="padding:10px;">Season</th>
                    <th style="padding:10px;">Highest Revenue Age Group</th>
                    <th style="padding:10px;">Total Revenue</th>
                    <th style="padding:10px;">Primary Package</th>
                </tr>
                <tr style="background:#dcfce7;">
                    <td style="padding:10px;"><strong>🔴 PEAK SEASON</strong><br>(Dec-Jan, Jun-Jul)</td>
                    <td style="padding:10px; font-size:20px; font-weight:800;">${peakTop}</td>
                    <td style="padding:10px;">${formatCr(peakRevenue)}</td>
                    <td style="padding:10px;">Combo (72%)</td>
                </tr>
                <tr style="background:#fef3c7;">
                    <td style="padding:10px;"><strong>🟡 OFF-PEAK SEASON</strong><br>(Feb-May, Aug-Nov)</td>
                    <td style="padding:10px; font-size:20px; font-weight:800;">${offpeakTop}</td>
                    <td style="padding:10px;">${formatCr(offpeakRevenue)}</td>
                    <td style="padding:10px;">Combo (74%)</td>
                </tr>
            </table>
            <p style="margin-top:15px;"><strong>💡 KEY FINDING:</strong> Age group <strong>${peakTop}</strong> generates highest revenue during Peak Season, while age group <strong>${offpeakTop}</strong> leads during Off-Peak Season — a clear generational shift in travel behavior.</p>
        `;
    }

    // Refresh entire dashboard based on selected season
    function refreshDashboard() {
        const data = currentSeason === 'peak' ? peakData : offpeakData;
        const seasonName = currentSeason === 'peak' ? 'PEAK SEASON' : 'OFF-PEAK SEASON';
        
        updateKPIs(data, seasonName);
        updateStackedChart(data);
        updateLineChart(data, seasonName);
        updateTable(data);
        updateAnnotation(data, seasonName);
        updateConclusion(peakData, offpeakData);
    }

    // Season toggle event listener
    const radioButtons = document.querySelectorAll('input[name="season"]');
    radioButtons.forEach(radio => {
        radio.addEventListener('change', (e) => {
            if (e.target.checked) {
                currentSeason = e.target.value;
                refreshDashboard();
            }
        });
    });

    // Initialize dashboard
    refreshDashboard();
</script>
</body>
</html>
