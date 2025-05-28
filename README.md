<!DOCTYPE html>
<html lang="de">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Cold Call Analytics Dashboard</title>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/Chart.js/4.4.0/chart.min.js"></script>
    <style>
        * {
            margin: 0;
            padding: 0;
            box-sizing: border-box;
        }

        body {
            font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, Oxygen, Ubuntu, Cantarell, sans-serif;
            background: linear-gradient(135deg, #0f172a 0%, #1e293b 50%, #334155 100%);
            min-height: 100vh;
            color: #f8fafc;
            padding: 20px;
        }

        .dashboard-container {
            max-width: 1400px;
            margin: 0 auto;
        }

        .header {
            text-align: center;
            margin-bottom: 2rem;
            padding: 2rem;
            background: rgba(15, 23, 42, 0.8);
            backdrop-filter: blur(20px);
            border-radius: 20px;
            border: 1px solid rgba(59, 130, 246, 0.2);
            box-shadow: 0 10px 30px rgba(0, 0, 0, 0.3);
        }

        .header h1 {
            font-size: 3rem;
            font-weight: 900;
            background: linear-gradient(135deg, #3b82f6, #8b5cf6);
            -webkit-background-clip: text;
            -webkit-text-fill-color: transparent;
            background-clip: text;
            margin-bottom: 0.5rem;
            filter: drop-shadow(0 0 10px rgba(59, 130, 246, 0.5));
        }

        .test-section {
            position: fixed;
            top: 20px;
            right: 20px;
            z-index: 9999;
            background: rgba(15, 23, 42, 0.9);
            padding: 1rem;
            border-radius: 12px;
            border: 1px solid #3b82f6;
        }

        .test-btn {
            background: #10b981;
            color: white;
            border: none;
            padding: 8px 16px;
            border-radius: 8px;
            cursor: pointer;
            margin-right: 8px;
            font-weight: 600;
        }

        .test-btn:hover {
            background: #059669;
        }

        .test-btn.success {
            background: #f59e0b;
        }

        .test-btn.success:hover {
            background: #d97706;
        }

        .stats-grid {
            display: grid;
            grid-template-columns: repeat(auto-fit, minmax(280px, 1fr));
            gap: 2rem;
            margin: 2rem 0;
        }

        .stat-card {
            background: rgba(15, 23, 42, 0.8);
            backdrop-filter: blur(20px);
            border-radius: 20px;
            padding: 2rem;
            border: 1px solid rgba(59, 130, 246, 0.2);
            box-shadow: 0 10px 30px rgba(0, 0, 0, 0.3);
        }

        .stat-number {
            font-size: 2.5rem;
            font-weight: 800;
            color: #3b82f6;
            margin: 1rem 0;
        }

        .stat-label {
            color: #94a3b8;
            font-size: 1rem;
        }
    </style>
</head>
<body>
    <!-- Test Buttons -->
    <div class="test-section">
        <button class="test-btn" onclick="testCall()">📞 Test Call</button>
        <button class="test-btn success" onclick="testSuccess()">✅ Success</button>
    </div>

    <div class="dashboard-container">
        <div class="header">
            <h1>Cold Call Dashboard</h1>
            <p>Momentum Labs Analytics</p>
        </div>

        <div class="stats-grid">
            <div class="stat-card">
                <div class="stat-number" id="totalCalls">0</div>
                <div class="stat-label">Anrufe getätigt</div>
            </div>
            
            <div class="stat-card">
                <div class="stat-number" id="totalAnswered">0</div>
                <div class="stat-label">Anrufe angenommen</div>
            </div>
            
            <div class="stat-card">
                <div class="stat-number" id="totalBooked">0</div>
                <div class="stat-label">Termine gebucht</div>
            </div>
            
            <div class="stat-card">
                <div class="stat-number" id="totalCost">$0.00</div>
                <div class="stat-label">Gesamtkosten</div>
            </div>
        </div>

        <div style="text-align: center; margin: 2rem 0; padding: 2rem; background: rgba(15, 23, 42, 0.8); border-radius: 20px;">
            <h3 style="color: #10b981; margin-bottom: 1rem;">✅ Dashboard ist bereit!</h3>
            <p style="color: #94a3b8;">Verwenden Sie die Test-Buttons oben rechts oder verbinden Sie N8N</p>
        </div>
    </div>

    <script>
        let dashboardData = {
            totalCalls: 0,
            totalAnswered: 0,
            totalBooked: 0,
            totalCost: 0
        };

        function updateDisplay() {
            document.getElementById('totalCalls').textContent = dashboardData.totalCalls;
            document.getElementById('totalAnswered').textContent = dashboardData.totalAnswered;
            document.getElementById('totalBooked').textContent = dashboardData.totalBooked;
            document.getElementById('totalCost').textContent = '$' + dashboardData.totalCost.toFixed(2);
        }

        function testCall() {
            dashboardData.totalCalls++;
            dashboardData.totalAnswered++;
            dashboardData.totalCost += 0.05;
            updateDisplay();
            alert('📞 Test-Anruf hinzugefügt!');
        }

        function testSuccess() {
            dashboardData.totalCalls++;
            dashboardData.totalAnswered++;
            dashboardData.totalBooked++;
            dashboardData.totalCost += 0.08;
            updateDisplay();
            alert('✅ Erfolgreicher Anruf hinzugefügt!');
        }

        // N8N Integration
        window.addCallData = function(data) {
            console.log('Received data:', data);
            
            dashboardData.totalCalls++;
            
            if (data.message && data.message.durationSeconds > 0) {
                dashboardData.totalAnswered++;
            }
            
            if (data.message && data.message.analysis && data.message.analysis.successEvaluation === 'true') {
                dashboardData.totalBooked++;
            }
            
            if (data.message && data.message.cost) {
                dashboardData.totalCost += data.message.cost;
            }
            
            updateDisplay();
        };

        console.log('Dashboard ready! URL:', window.location.href);
    </script>
</body>
</html>
