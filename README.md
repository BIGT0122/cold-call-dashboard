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

        .header p {
            font-size: 1.2rem;
            color: #94a3b8;
        }

        .status-bar {
            display: flex;
            align-items: center;
            justify-content: space-between;
            background: rgba(15, 23, 42, 0.8);
            backdrop-filter: blur(20px);
            padding: 1rem 2rem;
            border-radius: 16px;
            box-shadow: 0 10px 25px rgba(0, 0, 0, 0.2);
            margin-bottom: 2rem;
            border: 1px solid rgba(59, 130, 246, 0.2);
        }

        .status-indicator {
            display: flex;
            align-items: center;
            gap: 12px;
        }

        .status-dot {
            width: 12px;
            height: 12px;
            border-radius: 50%;
            background: #10b981;
            box-shadow: 0 0 0 4px rgba(16, 185, 129, 0.2);
            animation: pulse 2s infinite;
        }

        @keyframes pulse {
            0% { box-shadow: 0 0 0 4px rgba(16, 185, 129, 0.2); }
            50% { box-shadow: 0 0 0 8px rgba(16, 185, 129, 0.1); }
            100% { box-shadow: 0 0 0 4px rgba(16, 185, 129, 0.2); }
        }

        .time-selector {
            display: flex;
            background: rgba(15, 23, 42, 0.6);
            border-radius: 15px;
            padding: 8px;
            backdrop-filter: blur(10px);
            margin: 0 auto 2rem;
            width: fit-content;
            border: 1px solid rgba(59, 130, 246, 0.2);
        }

        .time-btn {
            background: transparent;
            border: none;
            padding: 12px 24px;
            border-radius: 12px;
            font-size: 14px;
            font-weight: 600;
            color: #94a3b8;
            cursor: pointer;
            transition: all 0.3s ease;
        }

        .time-btn.active {
            background: linear-gradient(135deg, #3b82f6, #8b5cf6);
            color: white;
            box-shadow: 0 4px 15px rgba(59, 130, 246, 0.4);
        }

        .stats-grid {
            display: grid;
            grid-template-columns: repeat(auto-fit, minmax(280px, 1fr));
            gap: 2rem;
            margin-bottom: 3rem;
        }

        .stat-card {
            background: rgba(15, 23, 42, 0.8);
            backdrop-filter: blur(20px);
            border-radius: 20px;
            padding: 2rem;
            border: 1px solid rgba(59, 130, 246, 0.2);
            box-shadow: 0 10px 30px rgba(0, 0, 0, 0.3);
            transition: all 0.4s ease;
            position: relative;
            overflow: hidden;
        }

        .stat-card::before {
            content: '';
            position: absolute;
            top: 0;
            left: 0;
            right: 0;
            height: 4px;
            background: var(--card-gradient);
        }

        .stat-card:hover {
            transform: translateY(-5px);
            box-shadow: 0 20px 40px rgba(0, 0, 0, 0.4);
            border-color: rgba(59, 130, 246, 0.4);
        }

        .stat-header {
            display: flex;
            align-items: center;
            justify-content: space-between;
            margin-bottom: 1.5rem;
        }

        .stat-icon {
            width: 50px;
            height: 50px;
            border-radius: 12px;
            display: flex;
            align-items: center;
            justify-content: center;
            font-size: 1.4rem;
            background: var(--icon-gradient);
            color: white;
            box-shadow: 0 8px 20px rgba(59, 130, 246, 0.3);
        }

        .stat-number {
            font-size: 2.8rem;
            font-weight: 800;
            color: #f8fafc;
            margin-bottom: 0.5rem;
        }

        .stat-label {
            color: #94a3b8;
            font-size: 0.95rem;
            font-weight: 500;
            margin-bottom: 1rem;
        }

        .stat-change {
            display: flex;
            align-items: center;
            gap: 6px;
            font-size: 14px;
            font-weight: 600;
            padding: 6px 12px;
            border-radius: 20px;
            background: rgba(16, 185, 129, 0.2);
            color: #10b981;
            width: fit-content;
        }

        /* Card specific gradients */
        .card-calls { --card-gradient: linear-gradient(135deg, #3b82f6, #1e40af); --icon-gradient: linear-gradient(135deg, #3b82f6, #1e40af); }
        .card-answered { --card-gradient: linear-gradient(135deg, #10b981, #059669); --icon-gradient: linear-gradient(135deg, #10b981, #059669); }
        .card-booked { --card-gradient: linear-gradient(135deg, #f59e0b, #d97706); --icon-gradient: linear-gradient(135deg, #f59e0b, #d97706); }
        .card-duration { --card-gradient: linear-gradient(135deg, #8b5cf6, #7c3aed); --icon-gradient: linear-gradient(135deg, #8b5cf6, #7c3aed); }
        .card-cost { --card-gradient: linear-gradient(135deg, #ef4444, #dc2626); --icon-gradient: linear-gradient(135deg, #ef4444, #dc2626); }
        .card-roi { --card-gradient: linear-gradient(135deg, #06b6d4, #0891b2); --icon-gradient: linear-gradient(135deg, #06b6d4, #0891b2); }

        .main-content {
            display: grid;
            grid-template-columns: 2fr 1fr;
            gap: 2rem;
            margin-bottom: 3rem;
        }

        .chart-card {
            background: rgba(15, 23, 42, 0.8);
            backdrop-filter: blur(20px);
            border-radius: 20px;
            padding: 2rem;
            border: 1px solid rgba(59, 130, 246, 0.2);
            box-shadow: 0 10px 30px rgba(0, 0, 0, 0.3);
        }

        .card-title {
            font-size: 1.4rem;
            font-weight: 700;
            margin-bottom: 1.5rem;
            color: #f8fafc;
            position: relative;
        }

        .card-title::after {
            content: '';
            position: absolute;
            bottom: -8px;
            left: 0;
            width: 60px;
            height: 3px;
            background: linear-gradient(135deg, #3b82f6, #8b5cf6);
            border-radius: 2px;
        }

        .insights-container {
            display: flex;
            flex-direction: column;
            gap: 1.5rem;
        }

        .insight-section {
            background: rgba(30, 41, 59, 0.6);
            border-radius: 16px;
            padding: 1.5rem;
            border: 1px solid rgba(59, 130, 246, 0.1);
        }

        .insight-title {
            font-size: 1.1rem;
            font-weight: 600;
            margin-bottom: 1rem;
            color: #f8fafc;
        }

        .cost-breakdown {
            display: grid;
            grid-template-columns: 1fr 1fr;
            gap: 1rem;
        }

        .cost-item {
            text-align: center;
            padding: 1rem;
            background: rgba(59, 130, 246, 0.1);
            border-radius: 12px;
            transition: all 0.3s ease;
        }

        .cost-item:hover {
            background: rgba(59, 130, 246, 0.2);
            transform: translateY(-2px);
        }

        .cost-value {
            font-size: 1.2rem;
            font-weight: 700;
            color: #3b82f6;
            margin-bottom: 0.5rem;
        }

        .cost-label {
            font-size: 0.8rem;
            color: #94a3b8;
            text-transform: uppercase;
            letter-spacing: 0.5px;
        }

        .performance-grid {
            display: grid;
            grid-template-columns: repeat(auto-fit, minmax(350px, 1fr));
            gap: 2rem;
            margin-bottom: 3rem;
        }

        .time-performance {
            display: flex;
            flex-direction: column;
            gap: 12px;
        }

        .time-slot {
            display: flex;
            align-items: center;
            justify-content: space-between;
            padding: 12px 16px;
            background: rgba(30, 41, 59, 0.6);
            border-radius: 12px;
            transition: all 0.3s ease;
            cursor: pointer;
            border: 1px solid rgba(59, 130, 246, 0.1);
        }

        .time-slot:hover {
            background: rgba(59, 130, 246, 0.2);
            transform: translateX(5px);
        }

        .time-label {
            font-weight: 500;
            color: #f8fafc;
        }

        .success-rate {
            display: flex;
            align-items: center;
            gap: 8px;
        }

        .rate-bar {
            width: 60px;
            height: 6px;
            background: rgba(100, 116, 139, 0.3);
            border-radius: 3px;
            overflow: hidden;
        }

        .rate-fill {
            height: 100%;
            background: linear-gradient(90deg, #10b981, #059669);
            border-radius: 3px;
            transition: width 0.8s ease;
        }

        .recent-calls {
            margin-top: 1.5rem;
        }

        .call-item {
            display: flex;
            justify-content: space-between;
            align-items: center;
            padding: 1rem;
            border-bottom: 1px solid rgba(100, 116, 139, 0.2);
            transition: background 0.3s ease;
        }

        .call-item:hover {
            background: rgba(59, 130, 246, 0.05);
        }

        .call-item:last-child {
            border-bottom: none;
        }

        .call-info h4 {
            font-weight: 600;
            color: #f8fafc;
            margin-bottom: 0.2rem;
        }

        .call-info p {
            font-size: 0.9rem;
            color: #94a3b8;
        }

        .call-status {
            padding: 0.3rem 0.8rem;
            border-radius: 12px;
            font-size: 0.8rem;
            font-weight: 500;
        }

        .status-success { background: rgba(16, 185, 129, 0.2); color: #10b981; }
        .status-failed { background: rgba(239, 68, 68, 0.2); color: #ef4444; }
        .status-pending { background: rgba(245, 158, 11, 0.2); color: #f59e0b; }

        .api-section {
            background: rgba(15, 23, 42, 0.8);
            backdrop-filter: blur(20px);
            border-radius: 20px;
            padding: 2rem;
            border: 1px solid rgba(59, 130, 246, 0.2);
            box-shadow: 0 10px 30px rgba(0, 0, 0, 0.3);
            margin-bottom: 2rem;
        }

        .api-endpoint {
            background: rgba(30, 41, 59, 0.6);
            border-radius: 12px;
            padding: 1.5rem;
            margin: 1rem 0;
            font-family: 'SF Mono', Monaco, 'Cascadia Code', 'Roboto Mono', Consolas, monospace;
            font-size: 13px;
            line-height: 1.6;
            position: relative;
            border-left: 4px solid #3b82f6;
        }

        .method {
            display: inline-block;
            padding: 4px 8px;
            border-radius: 6px;
            font-size: 11px;
            font-weight: 700;
            margin-right: 12px;
            text-transform: uppercase;
            letter-spacing: 0.5px;
            background: linear-gradient(135deg, #10b981, #059669);
            color: white;
        }

        .demo-section {
            text-align: center;
            padding: 2rem;
            background: rgba(15, 23, 42, 0.8);
            backdrop-filter: blur(20px);
            border-radius: 20px;
            border: 1px solid rgba(59, 130, 246, 0.2);
            box-shadow: 0 10px 30px rgba(0, 0, 0, 0.3);
            margin-bottom: 2rem;
        }

        .demo-btn {
            background: linear-gradient(135deg, #f59e0b, #d97706);
            color: white;
            padding: 1rem 2rem;
            border: none;
            border-radius: 12px;
            font-size: 1rem;
            font-weight: 600;
            cursor: pointer;
            transition: all 0.3s ease;
            text-decoration: none;
            display: inline-block;
            margin: 0.5rem;
        }

        .demo-btn:hover {
            transform: translateY(-2px);
            box-shadow: 0 8px 25px rgba(245, 158, 11, 0.4);
        }

        @media (max-width: 1200px) {
            .main-content {
                grid-template-columns: 1fr;
            }
            .performance-grid {
                grid-template-columns: 1fr;
            }
        }

        @media (max-width: 768px) {
            body {
                padding: 10px;
            }
            .header h1 {
                font-size: 2rem;
            }
            .stats-grid {
                grid-template-columns: 1fr;
            }
        }

        .loading {
            display: inline-block;
            width: 20px;
            height: 20px;
            border: 3px solid rgba(59, 130, 246, 0.3);
            border-radius: 50%;
            border-top-color: #3b82f6;
            animation: spin 1s ease-in-out infinite;
        }

        @keyframes spin {
            to { transform: rotate(360deg); }
        }
    </style>
</head>
<body>
    <div class="dashboard-container">
        <!-- Header -->
        <div class="header">
            <h1>Cold Call Analytics</h1>
            <p>Momentum Labs Dashboard</p>
        </div>

        <!-- Status Bar -->
        <div class="status-bar">
            <div class="status-indicator">
                <div class="status-dot"></div>
                <span style="font-weight: 600;">Live Dashboard</span>
                <span style="background: rgba(16, 185, 129, 0.2); color: #10b981; padding: 6px 12px; border-radius: 8px; font-size: 12px; font-weight: 600; margin-left: 12px;">🔗 N8N Connected</span>
            </div>
            <div style="color: #94a3b8;">
                Letztes Update: <span id="lastUpdate">Lädt...</span>
            </div>
        </div>

        <!-- Time Selector -->
        <div class="time-selector">
            <button class="time-btn active" onclick="switchPeriod('today')">Heute</button>
            <button class="time-btn" onclick="switchPeriod('week')">Diese Woche</button>
            <button class="time-btn" onclick="switchPeriod('month')">Dieser Monat</button>
            <button class="time-btn" onclick="switchPeriod('all')">Gesamt</button>
        </div>

        <!-- Stats Grid -->
        <div class="stats-grid">
            <div class="stat-card card-calls">
                <div class="stat-header">
                    <div class="stat-icon">📞</div>
                </div>
                <div class="stat-number" id="totalCalls">0</div>
                <div class="stat-label">Anrufe getätigt</div>
                <div class="stat-change">
                    <span>↗</span>
                    <span id="callsChange">+0%</span>
                </div>
            </div>
            
            <div class="stat-card card-answered">
                <div class="stat-header">
                    <div class="stat-icon">✅</div>
                </div>
                <div class="stat-number" id="totalAnswered">0</div>
                <div class="stat-label">Anrufe angenommen</div>
                <div class="stat-change">
                    <span>↗</span>
                    <span id="answeredChange">+0%</span>
                </div>
            </div>
            
            <div class="stat-card card-booked">
                <div class="stat-header">
                    <div class="stat-icon">📅</div>
                </div>
                <div class="stat-number" id="totalBooked">0</div>
                <div class="stat-label">Termine gebucht</div>
                <div class="stat-change">
                    <span>↗</span>
                    <span id="bookedChange">+0%</span>
                </div>
            </div>
            
            <div class="stat-card card-duration">
                <div class="stat-header">
                    <div class="stat-icon">⏱️</div>
                </div>
                <div class="stat-number" id="avgDuration">0:00</div>
                <div class="stat-label">Ø Gesprächsdauer</div>
                <div class="stat-change">
                    <span>↗</span>
                    <span id="durationChange">+0s</span>
                </div>
            </div>
            
            <div class="stat-card card-cost">
                <div class="stat-header">
                    <div class="stat-icon">💰</div>
                </div>
                <div class="stat-number" id="totalCost">$0.00</div>
                <div class="stat-label">Gesamtkosten</div>
                <div class="stat-change">
                    <span>↗</span>
                    <span id="costChange">+$0.00</span>
                </div>
            </div>
            
            <div class="stat-card card-roi">
                <div class="stat-header">
                    <div class="stat-icon">📈</div>
                </div>
                <div class="stat-number" id="conversionRate">0%</div>
                <div class="stat-label">Conversion Rate</div>
                <div class="stat-change">
                    <span>↗</span>
                    <span id="roiChange">+0%</span>
                </div>
            </div>
        </div>

        <!-- Main Content -->
        <div class="main-content">
            <div class="chart-card">
                <div class="card-title">Performance Trends</div>
                <canvas id="trendsChart" width="400" height="200"></canvas>
            </div>
            
            <div class="insights-container">
                <div class="insight-section">
                    <div class="insight-title">🎯 AI Insights</div>
                    <div id="aiInsights">
                        <p style="color: #94a3b8; font-size: 14px; line-height: 1.6;">Warte auf Anrufdaten...</p>
                    </div>
                </div>

                <div class="insight-section">
                    <div class="insight-title">📊 Kostenaufschlüsselung</div>
                    <div class="cost-breakdown">
                        <div class="cost-item">
                            <div class="cost-value" id="costSTT">$0.00</div>
                            <div class="cost-label">STT</div>
                        </div>
                        <div class="cost-item">
                            <div class="cost-value" id="costLLM">$0.00</div>
                            <div class="cost-label">LLM</div>
                        </div>
                        <div class="cost-item">
                            <div class="cost-value" id="costTTS">$0.00</div>
                            <div class="cost-label">TTS</div>
                        </div>
                        <div class="cost-item">
                            <div class="cost-value" id="costVAPI">$0.00</div>
                            <div class="cost-label">VAPI</div>
                        </div>
                    </div>
                </div>
            </div>
        </div>

        <!-- Performance Grid -->
        <div class="performance-grid">
            <div class="chart-card">
                <div class="card-title">⏰ Optimale Anrufzeiten</div>
                <div class="time-performance" id="timePerformance">
                    <!-- Wird dynamisch gefüllt -->
                </div>
            </div>

            <div class="chart-card">
                <div class="card-title">📞 Letzte Anrufe</div>
                <div class="recent-calls" id="recentCalls">
                    <div class="call-item">
                        <div class="call-info">
                            <h4>Warte auf erste Anrufe...</h4>
                            <p>Anrufdaten werden hier angezeigt</p>
                        </div>
                        <div class="call-status status-pending">Pending</div>
                    </div>
                </div>
            </div>
        </div>

        <!-- N8N Integration Section -->
        <div class="api-section">
            <div class="card-title">🔗 N8N Webhook Integration</div>
            <p style="color: #94a3b8; margin-bottom: 1.5rem;">Fügen Sie diesen Code zu Ihrem N8N-Workflow hinzu:</p>
            
            <div class="api-endpoint">
                <span class="method">POST</span>
                <strong>JavaScript Code für N8N Function Node:</strong><br><br>
<pre style="color: #94a3b8; line-height: 1.6;">// Dashboard Webhook URL (anpassen!)
const dashboardUrl = 'https://ihre-domain.com/dashboard.html';

// Webhook-Daten an Dashboard senden
const webhookData = {
  message: $input.all()[0].json.body.message
};

// PostMessage an Dashboard senden (wenn gleiche Domain)
if (typeof window !== 'undefined') {
  window.postMessage({
    type: 'n8n-webhook',
    data: webhookData
  }, '*');
}

return $input.all();</pre>
            </div>

            <div style="background: rgba(16, 185, 129, 0.1); padding: 1rem; border-radius: 8px; margin-top: 1rem;">
                <strong style="color: #10b981;">✅ Setup-Anleitung:</strong><br>
                <p style="color: #94a3b8; font-size: 0.9rem; margin-top: 0.5rem;">
                    1. HTML-Datei auf Ihren Server laden<br>
                    2. Function-Node nach "Parse Call Result" hinzufügen<br>
                    3. Dashboard-URL in N8N eintragen<br>
                    4. Testen mit den Demo-Buttons oben
                </p>
            </div>
        </div>
    </div>

    <script>
        // Dashboard State Management
        let dashboardData = {
            today: { calls: 0, answered: 0, booked: 0, totalCost: 0, avgDuration: 0, costs: { stt: 0, llm: 0, tts: 0, vapi: 0 } },
            week: { calls: 0, answered: 0, booked: 0, totalCost: 0, avgDuration: 0, costs: { stt: 0, llm: 0, tts: 0, vapi: 0 } },
            month: { calls: 0, answered: 0, booked: 0, totalCost: 0, avgDuration: 0, costs: { stt: 0, llm: 0, tts: 0, vapi: 0 } },
            all: { calls: 0, answered: 0, booked: 0, totalCost: 0, avgDuration: 0, costs: { stt: 0, llm: 0, tts: 0, vapi: 0 } }
        };

        let callHistory = [];
        let timePerformance = {};
        let currentPeriod = 'today';
        let trendsChart = null;

        // Initialize Dashboard
        function initDashboard() {
            updateTimestamp();
            setupChart();
            setupTimePerformance();
            loadStoredData();
            updateDisplay();
            
            // Auto-refresh timestamp
            setInterval(updateTimestamp, 30000);
        }

        // N8N Webhook Data Processor
        function processWebhookData(data) {
            try {
                console.log('Processing webhook data:', data);

                if (!data.message) {
                    console.log('No message in webhook data');
                    return;
                }
                
                const callData = {
                    callId: data.message.call?.id || generateId(),
                    startedAt: data.message.startedAt,
                    endedAt: data.message.endedAt,
                    endedReason: data.message.endedReason,
                    durationSeconds: data.message.durationSeconds || 0,
                    cost: data.message.cost || 0,
                    costBreakdown: data.message.costBreakdown || {},
                    successEvaluation: data.message.analysis?.successEvaluation === 'true',
                    summary: data.message.analysis?.summary || '',
                    leadData: data.message.call?.metadata || {}
                };

                // Add to call history
                callHistory.push(callData);
                
                // Update aggregated data
                updateAggregatedData(callData);
                
                // Update display
                updateDisplay();
                updateChart();
                
                // Store data
                storeData();
                
                // Generate insights
                generateInsights();
                
                console.log('Webhook data processed successfully:', callData);
                
            } catch (error) {
                console.error('Error processing webhook data:', error);
            }
        }

        // Update Aggregated Data
        function updateAggregatedData(callData) {
            const callDate = new Date(callData.startedAt);
            const now = new Date();
            
            // Calculate time periods
            const isToday = callDate.toDateString() === now.toDateString();
            const isThisWeek = getWeekStart(callDate).getTime() === getWeekStart(now).getTime();
            const isThisMonth = callDate.getMonth() === now.getMonth() && callDate.getFullYear() === now.getFullYear();
            
            // Update all-time data
            updatePeriodData('all', callData);
            
            // Update period-specific data
            if (isToday) updatePeriodData('today', callData);
            if (isThisWeek) updatePeriodData('week', callData);
            if (isThisMonth) updatePeriodData('month', callData);
            
            // Update time performance
            updateTimePerformanceData(callData);
        }

        function updatePeriodData(period, callData) {
            const data = dashboardData[period];
            
            data.calls++;
            data.totalCost += callData.cost;
            
            // Check if call was answered
            if (callData.durationSeconds > 0) {
                data.answered++;
                
                // Update average duration
                const totalDuration = (data.avgDuration * (data.answered - 1)) + callData.durationSeconds;
                data.avgDuration = totalDuration / data.answered;
            }
            
            // Check if appointment was booked
            if (callData.successEvaluation) {
                data.booked++;
            }
            
            // Update cost breakdown
            if (callData.costBreakdown) {
                data.costs.stt += callData.costBreakdown.stt || 0;
                data.costs.llm += callData.costBreakdown.llm || 0;
                data.costs.tts += callData.costBreakdown.tts || 0;
                data.costs.vapi += callData.costBreakdown.vapi || 0;
            }
        }

        function updateTimePerformanceData(callData) {
            const callTime = new Date(callData.startedAt);
            const hour = callTime.getHours();
            const timeSlot = getTimeSlot(hour);
            
            if (!timePerformance[timeSlot]) {
                timePerformance[timeSlot] = { total: 0, successful: 0 };
            }
            
            timePerformance[timeSlot].total++;
            if (callData.successEvaluation) {
                timePerformance[timeSlot].successful++;
            }
        }

        // Helper Functions
        function getWeekStart(date) {
            const d = new Date(date);
            const day = d.getDay();
            const diff = d.getDate() - day + (day === 0 ? -6 : 1);
            return new Date(d.setDate(diff));
        }

        function getTimeSlot(hour) {
            if (hour >= 8 && hour < 12) return 'Vormittag (8-12h)';
            if (hour >= 12 && hour < 16) return 'Mittag (12-16h)';
            if (hour >= 16 && hour < 20) return 'Nachmittag (16-20h)';
            return 'Abend (20-8h)';
        }

        function generateId() {
            return 'call_' + Date.now() + '_' + Math.random().toString(36).substr(2, 9);
        }

        // Update Display
        function updateDisplay() {
            const data = dashboardData[currentPeriod];
            
            // Update main stats
            animateNumber('totalCalls', data.calls);
            animateNumber('totalAnswered', data.answered);
            animateNumber('totalBooked', data.booked);
            
            // Update duration
            document.getElementById('avgDuration').textContent = formatDuration(data.avgDuration);
            
            // Update costs
            document.getElementById('totalCost').textContent = ' + data.totalCost.toFixed(3);
            
            // Update cost breakdown
            document.getElementById('costSTT').textContent = ' + data.costs.stt.toFixed(3);
            document.getElementById('costLLM').textContent = ' + data.costs.llm.toFixed(3);
            document.getElementById('costTTS').textContent = ' + data.costs.tts.toFixed(3);
            document.getElementById('costVAPI').textContent = ' + data.costs.vapi.toFixed(3);
            
            // Calculate conversion rate
            const conversionRate = data.answered > 0 ? (data.booked / data.answered * 100) : 0;
            document.getElementById('conversionRate').textContent = conversionRate.toFixed(1) + '%';
            
            // Update change indicators
            const answerRate = data.calls > 0 ? (data.answered / data.calls * 100) : 0;
            const bookingRate = data.answered > 0 ? (data.booked / data.answered * 100) : 0;
            
            document.getElementById('callsChange').textContent = '+' + data.calls;
            document.getElementById('answeredChange').textContent = answerRate.toFixed(1) + '%';
            document.getElementById('bookedChange').textContent = bookingRate.toFixed(1) + '%';
            document.getElementById('roiChange').textContent = '+' + conversionRate.toFixed(0) + '%';
            
            // Update time performance
            updateTimePerformanceDisplay();
            
            // Update recent calls
            updateRecentCalls();
        }

        function updateTimePerformanceDisplay() {
            const container = document.getElementById('timePerformance');
            container.innerHTML = '';
            
            const timeSlots = ['Vormittag (8-12h)', 'Mittag (12-16h)', 'Nachmittag (16-20h)', 'Abend (20-8h)'];
            
            timeSlots.forEach(slot => {
                const performance = timePerformance[slot] || { total: 0, successful: 0 };
                const successRate = performance.total > 0 ? (performance.successful / performance.total * 100) : 0;
                
                const slotElement = document.createElement('div');
                slotElement.className = 'time-slot';
                slotElement.innerHTML = `
                    <span class="time-label">${slot}</span>
                    <div class="success-rate">
                        <div class="rate-bar">
                            <div class="rate-fill" style="width: ${successRate}%"></div>
                        </div>
                        <span style="font-size: 14px; font-weight: 600; color: #10b981;">${successRate.toFixed(1)}%</span>
                    </div>
                `;
                container.appendChild(slotElement);
            });
        }

        function updateRecentCalls() {
            const container = document.getElementById('recentCalls');
            
            if (callHistory.length === 0) {
                container.innerHTML = `
                    <div class="call-item">
                        <div class="call-info">
                            <h4>Warte auf erste Anrufe...</h4>
                            <p>Anrufdaten werden hier angezeigt</p>
                        </div>
                        <div class="call-status status-pending">Pending</div>
                    </div>
                `;
                return;
            }

            const recentCalls = callHistory.slice(-5).reverse();
            container.innerHTML = recentCalls.map(call => {
                const duration = formatDuration(call.durationSeconds);
                const status = call.successEvaluation ? 'Termin gebucht' : 
                             call.durationSeconds > 0 ? 'Gespräch geführt' : 'Nicht erreicht';
                const statusClass = call.successEvaluation ? 'status-success' : 
                                  call.durationSeconds > 0 ? 'status-pending' : 'status-failed';
                const timeAgo = getTimeAgo(call.startedAt);
                
                return `
                    <div class="call-item">
                        <div class="call-info">
                            <h4>${call.leadData.leadName || 'Unbekannt'}</h4>
                            <p>${call.leadData.company || 'Keine Firma'} • ${duration} • ${timeAgo}</p>
                        </div>
                        <div class="call-status ${statusClass}">${status}</div>
                    </div>
                `;
            }).join('');
        }

        function getTimeAgo(timestamp) {
            const now = new Date();
            const callTime = new Date(timestamp);
            const diffMs = now - callTime;
            const diffMins = Math.floor(diffMs / 60000);
            
            if (diffMins < 1) return 'Gerade eben';
            if (diffMins < 60) return `Vor ${diffMins} Min`;
            
            const diffHours = Math.floor(diffMins / 60);
            if (diffHours < 24) return `Vor ${diffHours} Std`;
            
            const diffDays = Math.floor(diffHours / 24);
            return `Vor ${diffDays} Tag${diffDays > 1 ? 'en' : ''}`;
        }

        // Chart Setup
        function setupChart() {
            const ctx = document.getElementById('trendsChart').getContext('2d');
            
            trendsChart = new Chart(ctx, {
                type: 'line',
                data: {
                    labels: [],
                    datasets: [{
                        label: 'Anrufe',
                        data: [],
                        borderColor: '#3b82f6',
                        backgroundColor: 'rgba(59, 130, 246, 0.1)',
                        tension: 0.4,
                        fill: true
                    }, {
                        label: 'Termine gebucht',
                        data: [],
                        borderColor: '#10b981',
                        backgroundColor: 'rgba(16, 185, 129, 0.1)',
                        tension: 0.4,
                        fill: true
                    }]
                },
                options: {
                    responsive: true,
                    plugins: {
                        legend: {
                            position: 'top',
                            labels: { color: '#f8fafc' }
                        }
                    },
                    scales: {
                        y: {
                            beginAtZero: true,
                            ticks: { color: '#94a3b8' },
                            grid: { color: 'rgba(148, 163, 184, 0.1)' }
                        },
                        x: {
                            ticks: { color: '#94a3b8' },
                            grid: { color: 'rgba(148, 163, 184, 0.1)' }
                        }
                    }
                }
            });
        }

        function updateChart() {
            if (!trendsChart) return;
            
            // Generate trend data based on current period
            const trendData = generateTrendData();
            
            trendsChart.data.labels = trendData.labels;
            trendsChart.data.datasets[0].data = trendData.calls;
            trendsChart.data.datasets[1].data = trendData.bookings;
            trendsChart.update();
        }

        function generateTrendData() {
            const periods = currentPeriod === 'today' ? 24 : 
                          currentPeriod === 'week' ? 7 : 
                          currentPeriod === 'month' ? 30 : 12;
            
            const labels = [];
            const calls = [];
            const bookings = [];

            // Generate data based on actual call history
            for (let i = periods - 1; i >= 0; i--) {
                if (currentPeriod === 'today') {
                    const hour = 23 - i;
                    labels.push(`${hour}:00`);
                    
                    const hourCalls = callHistory.filter(call => {
                        const callHour = new Date(call.startedAt).getHours();
                        return callHour === hour;
                    });
                    
                    calls.push(hourCalls.length);
                    bookings.push(hourCalls.filter(call => call.successEvaluation).length);
                } else {
                    const date = new Date();
                    date.setDate(date.getDate() - i);
                    
                    if (currentPeriod === 'week') {
                        labels.push(date.toLocaleDateString('de-DE', { weekday: 'short' }));
                    } else {
                        labels.push(date.getDate().toString());
                    }
                    
                    const dayCalls = callHistory.filter(call => {
                        const callDate = new Date(call.startedAt);
                        return callDate.toDateString() === date.toDateString();
                    });
                    
                    calls.push(dayCalls.length);
                    bookings.push(dayCalls.filter(call => call.successEvaluation).length);
                }
            }
            
            return { labels, calls, bookings };
        }

        // Insights Generation
        function generateInsights() {
            const data = dashboardData[currentPeriod];
            const insights = [];
            
            if (data.calls === 0) {
                insights.push("📊 Warte auf erste Anrufdaten...");
            } else {
                const conversionRate = data.answered > 0 ? (data.booked / data.answered * 100) : 0;
                const answerRate = data.calls > 0 ? (data.answered / data.calls * 100) : 0;
                
                if (conversionRate > 25) {
                    insights.push(`🎯 Überdurchschnittliche Conversion-Rate von ${conversionRate.toFixed(1)}%`);
                } else if (conversionRate < 15 && data.answered > 0) {
                    insights.push(`⚠️ Conversion-Rate bei ${conversionRate.toFixed(1)}% - Optimierungspotential vorhanden`);
                }
                
                if (answerRate > 50) {
                    insights.push(`📞 Gute Erreichbarkeit mit ${answerRate.toFixed(1)}% Annahmequote`);
                } else if (answerRate < 30 && data.calls > 0) {
                    insights.push(`📵 Niedrige Annahmequote von ${answerRate.toFixed(1)}% - Anrufzeiten optimieren`);
                }
                
                const costPerBooking = data.booked > 0 ? (data.totalCost / data.booked) : 0;
                if (costPerBooking > 0) {
                    insights.push(`💰 Kosten pro Terminbuchung: ${costPerBooking.toFixed(2)}`);
                }
                
                if (data.avgDuration > 120) {
                    insights.push(`⏱️ Längere Gespräche führen zu mehr Erfolg (Ø ${formatDuration(data.avgDuration)})`);
                }
                
                // Best time analysis
                const bestTimeSlot = getBestTimeSlot();
                if (bestTimeSlot) {
                    insights.push(`⏰ Beste Anrufzeit: ${bestTimeSlot}`);
                }
            }
            
            const container = document.getElementById('aiInsights');
            if (insights.length > 0) {
                container.innerHTML = insights.map(insight => 
                    `<p style="color: #f8fafc; font-size: 14px; line-height: 1.6; margin-bottom: 8px;">${insight}</p>`
                ).join('');
            } else {
                container.innerHTML = '<p style="color: #94a3b8; font-size: 14px; line-height: 1.6;">Sammle Daten für detaillierte Insights...</p>';
            }
        }

        function getBestTimeSlot() {
            let bestSlot = '';
            let bestRate = 0;
            
            Object.entries(timePerformance).forEach(([slot, data]) => {
                const rate = data.total > 0 ? (data.successful / data.total) : 0;
                if (rate > bestRate && data.total >= 2) {
                    bestRate = rate;
                    bestSlot = slot;
                }
            });
            
            return bestSlot;
        }

        // Utility Functions
        function animateNumber(elementId, targetValue) {
            const element = document.getElementById(elementId);
            const currentValue = parseInt(element.textContent) || 0;
            const increment = Math.ceil((targetValue - currentValue) / 15);
            
            if (currentValue < targetValue) {
                element.textContent = Math.min(currentValue + increment, targetValue);
                setTimeout(() => animateNumber(elementId, targetValue), 50);
            }
        }

        function formatDuration(seconds) {
            const minutes = Math.floor(seconds / 60);
            const remainingSeconds = Math.floor(seconds % 60);
            return `${minutes}:${remainingSeconds.toString().padStart(2, '0')}`;
        }

        function updateTimestamp() {
            document.getElementById('lastUpdate').textContent = new Date().toLocaleTimeString('de-DE');
        }

        function switchPeriod(period) {
            currentPeriod = period;
            
            // Update button states
            document.querySelectorAll('.time-btn').forEach(btn => {
                btn.classList.remove('active');
            });
            
            // Find and activate the clicked button
            event.target.classList.add('active');
            
            updateDisplay();
            updateChart();
        }

        function setupTimePerformance() {
            updateTimePerformanceDisplay();
        }

        // Data Persistence
        function storeData() {
            try {
                localStorage.setItem('coldCallDashboard', JSON.stringify({
                    dashboardData,
                    callHistory,
                    timePerformance
                }));
            } catch (error) {
                console.error('Error storing data:', error);
            }
        }

        function loadStoredData() {
            try {
                const stored = localStorage.getItem('coldCallDashboard');
                if (stored) {
                    const data = JSON.parse(stored);
                    dashboardData = data.dashboardData || dashboardData;
                    callHistory = data.callHistory || [];
                    timePerformance = data.timePerformance || {};
                }
            } catch (error) {
                console.error('Error loading stored data:', error);
            }
        }

        // N8N Integration - Public API
        window.addCallData = function(webhookData) {
            console.log('Received webhook data:', webhookData);
            processWebhookData(webhookData);
        };

        // PostMessage listener for N8N integration
        window.addEventListener('message', function(event) {
            if (event.data.type === 'n8n-webhook') {
                console.log('Received N8N webhook via postMessage:', event.data.data);
                processWebhookData(event.data.data);
            }
        });

        // Initialize Dashboard
        document.addEventListener('DOMContentLoaded', function() {
            initDashboard();
            console.log('Dashboard initialized and ready for N8N webhook data!');
        });
    </script>
</body>
</html>
