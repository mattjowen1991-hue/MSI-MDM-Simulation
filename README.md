<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>RDS Troubleshooting Case Study</title>
    <link href="https://fonts.googleapis.com/css2?family=JetBrains+Mono:wght@400;500;600&family=Plus+Jakarta+Sans:wght@400;500;600;700&display=swap" rel="stylesheet">
    <style>
        :root {
            --bg-dark: #0f1419;
            --bg-panel: #1a1f26;
            --bg-card: #232a33;
            --accent-blue: #58a6ff;
            --accent-green: #3fb950;
            --accent-orange: #d29922;
            --accent-red: #f85149;
            --accent-purple: #a371f7;
            --text-primary: #e6edf3;
            --text-secondary: #8b949e;
            --text-muted: #6e7681;
            --border-color: #30363d;
            --hubstaff-green: #36b37e;
        }

        * {
            margin: 0;
            padding: 0;
            box-sizing: border-box;
        }

        body {
            font-family: 'Plus Jakarta Sans', sans-serif;
            background: var(--bg-dark);
            color: var(--text-primary);
            min-height: 100vh;
            line-height: 1.6;
        }

        .container {
            max-width: 1400px;
            margin: 0 auto;
            padding: 2rem;
        }

        header {
            text-align: center;
            margin-bottom: 2rem;
            padding-bottom: 2rem;
            border-bottom: 1px solid var(--border-color);
        }

        .badge {
            display: inline-block;
            padding: 0.25rem 0.75rem;
            background: rgba(248, 81, 73, 0.15);
            color: var(--accent-red);
            border-radius: 20px;
            font-size: 0.75rem;
            font-weight: 600;
            text-transform: uppercase;
            margin-bottom: 1rem;
        }

        h1 {
            font-size: 2rem;
            font-weight: 700;
            margin-bottom: 0.5rem;
        }

        .subtitle {
            color: var(--text-secondary);
            font-size: 1rem;
        }

        /* Progress Steps */
        .progress-container {
            display: flex;
            justify-content: center;
            gap: 0;
            margin-bottom: 2rem;
            flex-wrap: wrap;
        }

        .progress-step {
            display: flex;
            align-items: center;
            gap: 0.5rem;
            padding: 0.75rem 1rem;
            background: var(--bg-panel);
            border: 1px solid var(--border-color);
            color: var(--text-muted);
            font-size: 0.85rem;
            cursor: pointer;
            transition: all 0.2s ease;
        }

        .progress-step:first-child {
            border-radius: 8px 0 0 8px;
        }

        .progress-step:last-child {
            border-radius: 0 8px 8px 0;
        }

        .progress-step.active {
            background: var(--accent-blue);
            border-color: var(--accent-blue);
            color: white;
        }

        .progress-step.complete {
            background: rgba(63, 185, 80, 0.15);
            border-color: var(--accent-green);
            color: var(--accent-green);
        }

        .step-number {
            width: 24px;
            height: 24px;
            border-radius: 50%;
            background: rgba(255, 255, 255, 0.1);
            display: flex;
            align-items: center;
            justify-content: center;
            font-weight: 600;
            font-size: 0.8rem;
        }

        .progress-step.active .step-number {
            background: rgba(255, 255, 255, 0.2);
        }

        .progress-step.complete .step-number {
            background: var(--accent-green);
            color: white;
        }

        /* Main Content */
        .main-content {
            display: grid;
            grid-template-columns: 1fr 400px;
            gap: 1.5rem;
        }

        @media (max-width: 1100px) {
            .main-content {
                grid-template-columns: 1fr;
            }
        }

        .card {
            background: var(--bg-panel);
            border: 1px solid var(--border-color);
            border-radius: 12px;
            overflow: hidden;
        }

        .card-header {
            padding: 1rem 1.25rem;
            border-bottom: 1px solid var(--border-color);
            display: flex;
            align-items: center;
            justify-content: space-between;
        }

        .card-header h3 {
            font-size: 0.95rem;
            font-weight: 600;
            display: flex;
            align-items: center;
            gap: 0.5rem;
        }

        .card-body {
            padding: 1.25rem;
        }

        /* Ticket Info */
        .ticket-info {
            display: grid;
            grid-template-columns: repeat(2, 1fr);
            gap: 1rem;
            margin-bottom: 1.5rem;
        }

        .info-item {
            background: var(--bg-card);
            padding: 0.75rem 1rem;
            border-radius: 8px;
        }

        .info-label {
            font-size: 0.75rem;
            color: var(--text-muted);
            text-transform: uppercase;
            margin-bottom: 0.25rem;
        }

        .info-value {
            font-weight: 500;
        }

        .info-value.highlight {
            color: var(--accent-orange);
        }

        /* Log Viewer */
        .log-viewer {
            background: #0d1117;
            border-radius: 8px;
            border: 1px solid var(--border-color);
            overflow: hidden;
            margin-bottom: 1rem;
        }

        .log-header {
            background: #161b22;
            padding: 0.5rem 1rem;
            display: flex;
            align-items: center;
            gap: 0.5rem;
            font-size: 0.8rem;
            color: var(--text-muted);
            border-bottom: 1px solid var(--border-color);
        }

        .log-content {
            padding: 1rem;
            font-family: 'JetBrains Mono', monospace;
            font-size: 0.8rem;
            max-height: 300px;
            overflow-y: auto;
        }

        .log-line {
            margin-bottom: 0.25rem;
            display: flex;
            gap: 0.5rem;
        }

        .log-time {
            color: var(--text-muted);
            flex-shrink: 0;
        }

        .log-level {
            flex-shrink: 0;
            padding: 0 0.25rem;
            border-radius: 3px;
            font-size: 0.7rem;
            font-weight: 600;
        }

        .log-level.debug { background: rgba(88, 166, 255, 0.15); color: var(--accent-blue); }
        .log-level.info { background: rgba(63, 185, 80, 0.15); color: var(--accent-green); }
        .log-level.warn { background: rgba(210, 153, 34, 0.15); color: var(--accent-orange); }
        .log-level.error { background: rgba(248, 81, 73, 0.15); color: var(--accent-red); }
        .log-level.audit { background: rgba(163, 113, 247, 0.15); color: var(--accent-purple); }

        .log-message {
            color: var(--text-secondary);
        }

        .log-highlight {
            background: rgba(248, 81, 73, 0.2);
            color: var(--accent-red);
            padding: 0 0.25rem;
            border-radius: 3px;
        }

        /* Questions */
        .question-section {
            margin-top: 1.5rem;
        }

        .question-text {
            font-size: 1rem;
            font-weight: 600;
            margin-bottom: 1rem;
            color: var(--accent-blue);
        }

        .options {
            display: flex;
            flex-direction: column;
            gap: 0.75rem;
        }

        .option {
            padding: 1rem;
            background: var(--bg-card);
            border: 2px solid var(--border-color);
            border-radius: 8px;
            cursor: pointer;
            transition: all 0.2s ease;
            display: flex;
            align-items: flex-start;
            gap: 0.75rem;
        }

        .option:hover {
            border-color: var(--accent-blue);
            background: rgba(88, 166, 255, 0.05);
        }

        .option.selected {
            border-color: var(--accent-blue);
            background: rgba(88, 166, 255, 0.1);
        }

        .option.correct {
            border-color: var(--accent-green);
            background: rgba(63, 185, 80, 0.1);
        }

        .option.incorrect {
            border-color: var(--accent-red);
            background: rgba(248, 81, 73, 0.1);
        }

        .option-marker {
            width: 24px;
            height: 24px;
            border-radius: 50%;
            border: 2px solid var(--border-color);
            display: flex;
            align-items: center;
            justify-content: center;
            flex-shrink: 0;
            font-size: 0.8rem;
            font-weight: 600;
        }

        .option.selected .option-marker {
            border-color: var(--accent-blue);
            background: var(--accent-blue);
            color: white;
        }

        .option.correct .option-marker {
            border-color: var(--accent-green);
            background: var(--accent-green);
            color: white;
        }

        .option.incorrect .option-marker {
            border-color: var(--accent-red);
            background: var(--accent-red);
            color: white;
        }

        .option-text {
            flex: 1;
        }

        .feedback {
            margin-top: 1rem;
            padding: 1rem;
            border-radius: 8px;
            display: none;
        }

        .feedback.show {
            display: block;
        }

        .feedback.correct {
            background: rgba(63, 185, 80, 0.1);
            border: 1px solid var(--accent-green);
        }

        .feedback.incorrect {
            background: rgba(248, 81, 73, 0.1);
            border: 1px solid var(--accent-red);
        }

        .feedback h4 {
            font-size: 0.9rem;
            margin-bottom: 0.5rem;
        }

        .feedback p {
            font-size: 0.85rem;
            color: var(--text-secondary);
        }

        /* Sidebar */
        .sidebar {
            display: flex;
            flex-direction: column;
            gap: 1.5rem;
        }

        .environment-diagram {
            padding: 1rem;
        }

        .server-box {
            background: var(--bg-card);
            border: 2px solid var(--border-color);
            border-radius: 12px;
            padding: 1rem;
            margin-bottom: 1rem;
        }

        .server-header {
            display: flex;
            align-items: center;
            gap: 0.5rem;
            margin-bottom: 1rem;
            padding-bottom: 0.75rem;
            border-bottom: 1px solid var(--border-color);
        }

        .server-icon {
            font-size: 1.5rem;
        }

        .server-name {
            font-weight: 600;
        }

        .server-type {
            font-size: 0.75rem;
            color: var(--text-muted);
        }

        .user-sessions {
            display: flex;
            flex-direction: column;
            gap: 0.5rem;
        }

        .user-session {
            display: flex;
            align-items: center;
            gap: 0.75rem;
            padding: 0.5rem 0.75rem;
            background: var(--bg-panel);
            border-radius: 6px;
            font-size: 0.85rem;
        }

        .session-status {
            width: 8px;
            height: 8px;
            border-radius: 50%;
        }

        .session-status.tracking {
            background: var(--accent-green);
            box-shadow: 0 0 8px var(--accent-green);
        }

        .session-status.not-tracking {
            background: var(--accent-red);
        }

        .session-status.idle {
            background: var(--accent-orange);
        }

        .session-user {
            flex: 1;
        }

        .session-badge {
            font-size: 0.7rem;
            padding: 0.15rem 0.5rem;
            border-radius: 10px;
            background: rgba(88, 166, 255, 0.15);
            color: var(--accent-blue);
        }

        .session-badge.problem {
            background: rgba(248, 81, 73, 0.15);
            color: var(--accent-red);
        }

        /* Key Findings */
        .findings-list {
            display: flex;
            flex-direction: column;
            gap: 0.75rem;
        }

        .finding-item {
            display: flex;
            align-items: flex-start;
            gap: 0.75rem;
            padding: 0.75rem;
            background: var(--bg-card);
            border-radius: 8px;
            font-size: 0.85rem;
        }

        .finding-icon {
            font-size: 1rem;
            flex-shrink: 0;
        }

        /* Buttons */
        .btn {
            padding: 0.75rem 1.5rem;
            border: none;
            border-radius: 8px;
            font-family: inherit;
            font-size: 0.9rem;
            font-weight: 500;
            cursor: pointer;
            transition: all 0.2s ease;
            display: inline-flex;
            align-items: center;
            gap: 0.5rem;
        }

        .btn-primary {
            background: var(--accent-blue);
            color: white;
        }

        .btn-primary:hover {
            background: #4393e6;
        }

        .btn-primary:disabled {
            opacity: 0.5;
            cursor: not-allowed;
        }

        .btn-secondary {
            background: var(--bg-card);
            color: var(--text-primary);
            border: 1px solid var(--border-color);
        }

        .btn-secondary:hover {
            background: var(--bg-panel);
        }

        .btn-group {
            display: flex;
            gap: 0.75rem;
            margin-top: 1.5rem;
        }

        /* Timeline */
        .timeline {
            position: relative;
            padding-left: 1.5rem;
        }

        .timeline::before {
            content: '';
            position: absolute;
            left: 0;
            top: 0;
            bottom: 0;
            width: 2px;
            background: var(--border-color);
        }

        .timeline-item {
            position: relative;
            padding-bottom: 1.5rem;
        }

        .timeline-item::before {
            content: '';
            position: absolute;
            left: -1.5rem;
            top: 0.25rem;
            width: 10px;
            height: 10px;
            border-radius: 50%;
            background: var(--border-color);
            transform: translateX(-4px);
        }

        .timeline-item.highlight::before {
            background: var(--accent-red);
            box-shadow: 0 0 8px var(--accent-red);
        }

        .timeline-date {
            font-size: 0.75rem;
            color: var(--text-muted);
            margin-bottom: 0.25rem;
        }

        .timeline-content {
            font-size: 0.9rem;
        }

        /* Summary Panel */
        .summary-panel {
            display: none;
        }

        .summary-panel.show {
            display: block;
        }

        .summary-stat {
            display: flex;
            justify-content: space-between;
            padding: 0.75rem 0;
            border-bottom: 1px solid var(--border-color);
        }

        .summary-stat:last-child {
            border-bottom: none;
        }

        .stat-label {
            color: var(--text-secondary);
        }

        .stat-value {
            font-weight: 600;
        }

        .stat-value.good {
            color: var(--accent-green);
        }

        /* Animations */
        @keyframes fadeIn {
            from { opacity: 0; transform: translateY(10px); }
            to { opacity: 1; transform: translateY(0); }
        }

        .fade-in {
            animation: fadeIn 0.3s ease;
        }

        /* Scrollbar */
        ::-webkit-scrollbar {
            width: 8px;
            height: 8px;
        }

        ::-webkit-scrollbar-track {
            background: var(--bg-dark);
        }

        ::-webkit-scrollbar-thumb {
            background: var(--border-color);
            border-radius: 4px;
        }

        ::-webkit-scrollbar-thumb:hover {
            background: var(--text-muted);
        }

        /* Hidden panels */
        .step-panel {
            display: none;
        }

        .step-panel.active {
            display: block;
        }

        .knowledge-callout {
            background: rgba(88, 166, 255, 0.1);
            border: 1px solid rgba(88, 166, 255, 0.3);
            border-radius: 8px;
            padding: 1rem;
            margin-top: 1rem;
        }

        .knowledge-callout h4 {
            color: var(--accent-blue);
            font-size: 0.9rem;
            margin-bottom: 0.5rem;
            display: flex;
            align-items: center;
            gap: 0.5rem;
        }

        .knowledge-callout p {
            font-size: 0.85rem;
            color: var(--text-secondary);
        }

        .knowledge-callout code {
            background: var(--bg-dark);
            padding: 0.15rem 0.4rem;
            border-radius: 4px;
            font-family: 'JetBrains Mono', monospace;
            font-size: 0.8rem;
        }
    </style>
</head>
<body>
    <div class="container">
        <header>
            <div class="badge">High Priority Escalation</div>
            <h1>🖥️ RDS Silent App Troubleshooting</h1>
            <p class="subtitle">Interactive Case Study: User Not Tracking on Multi-User RDS Server</p>
        </header>

        <div class="progress-container">
            <div class="progress-step active" onclick="goToStep(1)">
                <span class="step-number">1</span>
                <span>Ticket Review</span>
            </div>
            <div class="progress-step" onclick="goToStep(2)">
                <span class="step-number">2</span>
                <span>Log Analysis</span>
            </div>
            <div class="progress-step" onclick="goToStep(3)">
                <span class="step-number">3</span>
                <span>Root Cause</span>
            </div>
            <div class="progress-step" onclick="goToStep(4)">
                <span class="step-number">4</span>
                <span>Resolution</span>
            </div>
            <div class="progress-step" onclick="goToStep(5)">
                <span class="step-number">5</span>
                <span>Summary</span>
            </div>
        </div>

        <div class="main-content">
            <div class="content-area">
                <!-- Step 1: Ticket Review -->
                <div class="step-panel active" id="step-1">
                    <div class="card">
                        <div class="card-header">
                            <h3>📋 Ticket Details</h3>
                            <span style="color: var(--accent-orange); font-size: 0.85rem;">MRR: $2,564.59</span>
                        </div>
                        <div class="card-body">
                            <div class="ticket-info">
                                <div class="info-item">
                                    <div class="info-label">Affected User</div>
                                    <div class="info-value">Alex Thompson</div>
                                </div>
                                <div class="info-item">
                                    <div class="info-label">Comparison User</div>
                                    <div class="info-value">Sarah Martinez (working)</div>
                                </div>
                                <div class="info-item">
                                    <div class="info-label">Environment</div>
                                    <div class="info-value highlight">RDS Server (~170 seats)</div>
                                </div>
                                <div class="info-item">
                                    <div class="info-label">Last Activity</div>
                                    <div class="info-value highlight">Dec 18, 2025</div>
                                </div>
                            </div>

                            <div style="background: var(--bg-card); padding: 1rem; border-radius: 8px; margin-bottom: 1.5rem;">
                                <h4 style="font-size: 0.9rem; margin-bottom: 0.75rem;">Problem Description:</h4>
                                <p style="color: var(--text-secondary); font-size: 0.9rem;">
                                    One specific user (Alex Thompson) is not recording any activity via the Silent App, 
                                    despite being logged in and actively working. Other users on the same RDS server 
                                    are recording activity normally. The user was previously disabled, then re-enabled. 
                                    After re-enablement, activity was recorded only once (Dec 18), and nothing since.
                                </p>
                            </div>

                            <h4 style="font-size: 0.9rem; margin-bottom: 0.75rem;">Event Timeline:</h4>
                            <div class="timeline">
                                <div class="timeline-item">
                                    <div class="timeline-date">Dec 9, 2025</div>
                                    <div class="timeline-content">Silent App installed on HQ-RDS01</div>
                                </div>
                                <div class="timeline-item">
                                    <div class="timeline-date">Dec 15, 2025</div>
                                    <div class="timeline-content">Alex Thompson's account disabled by admin</div>
                                </div>
                                <div class="timeline-item">
                                    <div class="timeline-date">Dec 18, 2025</div>
                                    <div class="timeline-content">Account re-enabled, brief tracking recorded</div>
                                </div>
                                <div class="timeline-item highlight">
                                    <div class="timeline-date">Dec 18, 2025 - 19:06</div>
                                    <div class="timeline-content" style="color: var(--accent-red);">User session logged off - app shutdown</div>
                                </div>
                                <div class="timeline-item">
                                    <div class="timeline-date">Dec 19, 2025 - Present</div>
                                    <div class="timeline-content">No activity recorded despite daily logins</div>
                                </div>
                            </div>

                            <div class="question-section">
                                <div class="question-text">What's your first diagnostic step?</div>
                                <div class="options" id="q1-options">
                                    <div class="option" onclick="selectOption(1, 0, this)">
                                        <div class="option-marker">A</div>
                                        <div class="option-text">Reinstall the Silent App on the server</div>
                                    </div>
                                    <div class="option" onclick="selectOption(1, 1, this)">
                                        <div class="option-marker">B</div>
                                        <div class="option-text">Request and analyze client logs from the user's session</div>
                                    </div>
                                    <div class="option" onclick="selectOption(1, 2, this)">
                                        <div class="option-marker">C</div>
                                        <div class="option-text">Ask the user to restart their computer</div>
                                    </div>
                                    <div class="option" onclick="selectOption(1, 3, this)">
                                        <div class="option-marker">D</div>
                                        <div class="option-text">Escalate immediately to engineering</div>
                                    </div>
                                </div>
                                <div class="feedback" id="q1-feedback"></div>
                            </div>

                            <div class="btn-group">
                                <button class="btn btn-primary" id="next-1" onclick="nextStep(1)" disabled>Continue to Log Analysis →</button>
                            </div>
                        </div>
                    </div>
                </div>

                <!-- Step 2: Log Analysis -->
                <div class="step-panel" id="step-2">
                    <div class="card">
                        <div class="card-header">
                            <h3>📄 Log Analysis</h3>
                        </div>
                        <div class="card-body">
                            <p style="color: var(--text-secondary); margin-bottom: 1rem;">
                                You've received the client logs from Alex's session. Here are the key entries from <code>hubstaff.log</code>:
                            </p>

                            <div class="log-viewer">
                                <div class="log-header">
                                    <span>📁</span>
                                    <span>hubstaff.log (Dec 18, 2025)</span>
                                </div>
                                <div class="log-content">
                                    <div class="log-line">
                                        <span class="log-time">14:00:00</span>
                                        <span class="log-level debug">DEBUG</span>
                                        <span class="log-message">Session.cpp: Rolling Activity Start: 2025-12-18 19:00:00Z</span>
                                    </div>
                                    <div class="log-line">
                                        <span class="log-time">14:00:00</span>
                                        <span class="log-level debug">DEBUG</span>
                                        <span class="log-message">Backend.cpp: Applied activity. Total: 03:19:12</span>
                                    </div>
                                    <div class="log-line">
                                        <span class="log-time">18:32:17</span>
                                        <span class="log-level debug">DEBUG</span>
                                        <span class="log-message">NetworkAccess.cpp: Response: 200 Type: application/json</span>
                                    </div>
                                    <div class="log-line">
                                        <span class="log-time">19:05:29</span>
                                        <span class="log-level debug">DEBUG</span>
                                        <span class="log-message">Backend_System.cpp: Heart beat : 2892</span>
                                    </div>
                                    <div class="log-line">
                                        <span class="log-time">19:06:09</span>
                                        <span class="log-level info">INFO</span>
                                        <span class="log-message log-highlight">FLApplication_win32.cpp: Received QueryEndSession 0: LOGOFF</span>
                                    </div>
                                    <div class="log-line">
                                        <span class="log-time">19:06:09</span>
                                        <span class="log-level info">INFO</span>
                                        <span class="log-message log-highlight">FLApplication_win32.cpp: Received EndSession 1: LOGOFF</span>
                                    </div>
                                    <div class="log-line">
                                        <span class="log-time">19:06:09</span>
                                        <span class="log-level info">INFO</span>
                                        <span class="log-message">FLApplication_win32.cpp: Proceeding with shutdown</span>
                                    </div>
                                    <div class="log-line">
                                        <span class="log-time">19:06:09</span>
                                        <span class="log-level audit">AUDIT</span>
                                        <span class="log-message log-highlight">Backend_System.cpp: (SHUTDOWN) Shutdown client</span>
                                    </div>
                                    <div class="log-line">
                                        <span class="log-time">19:06:09</span>
                                        <span class="log-level info">INFO</span>
                                        <span class="log-message">FLApplication_win32.cpp: Shutdown complete</span>
                                    </div>
                                    <div class="log-line" style="margin-top: 1rem; padding-top: 0.5rem; border-top: 1px dashed var(--border-color);">
                                        <span class="log-time" style="color: var(--accent-red);">—</span>
                                        <span class="log-level error">NOTE</span>
                                        <span class="log-message" style="color: var(--accent-red);">No entries after Dec 18, 2025</span>
                                    </div>
                                </div>
                            </div>

                            <div class="knowledge-callout">
                                <h4>💡 Key Observation</h4>
                                <p>The logs show the app received a <code>LOGOFF</code> signal and shut down cleanly on Dec 18 at 19:06:09. 
                                After this point, there are no additional log entries, meaning the app never started again for this user's session.</p>
                            </div>

                            <div class="question-section">
                                <div class="question-text">What does the absence of logs after Dec 18 indicate?</div>
                                <div class="options" id="q2-options">
                                    <div class="option" onclick="selectOption(2, 0, this)">
                                        <div class="option-marker">A</div>
                                        <div class="option-text">The Silent App was uninstalled from the server</div>
                                    </div>
                                    <div class="option" onclick="selectOption(2, 1, this)">
                                        <div class="option-marker">B</div>
                                        <div class="option-text">The app is crashing on startup</div>
                                    </div>
                                    <div class="option" onclick="selectOption(2, 2, this)">
                                        <div class="option-marker">C</div>
                                        <div class="option-text">The app isn't starting for this user's session after the logoff</div>
                                    </div>
                                    <div class="option" onclick="selectOption(2, 3, this)">
                                        <div class="option-marker">D</div>
                                        <div class="option-text">The logs are being written to a different location</div>
                                    </div>
                                </div>
                                <div class="feedback" id="q2-feedback"></div>
                            </div>

                            <div class="btn-group">
                                <button class="btn btn-secondary" onclick="goToStep(1)">← Back</button>
                                <button class="btn btn-primary" id="next-2" onclick="nextStep(2)" disabled>Continue to Root Cause →</button>
                            </div>
                        </div>
                    </div>
                </div>

                <!-- Step 3: Root Cause -->
                <div class="step-panel" id="step-3">
                    <div class="card">
                        <div class="card-header">
                            <h3>🔍 Root Cause Analysis</h3>
                        </div>
                        <div class="card-body">
                            <p style="color: var(--text-secondary); margin-bottom: 1rem;">
                                You request the service logs from <code>C:\ProgramData\Netsoft-Logs\Corporate</code> to understand why the app isn't starting:
                            </p>

                            <div class="log-viewer">
                                <div class="log-header">
                                    <span>📁</span>
                                    <span>hubstaff-service logs (bootstrap sequence)</span>
                                </div>
                                <div class="log-content">
                                    <div class="log-line">
                                        <span class="log-time">08:00:31</span>
                                        <span class="log-level info">INFO</span>
                                        <span class="log-message">hs_service::bootstrap: Starting user session detection</span>
                                    </div>
                                    <div class="log-line">
                                        <span class="log-time">08:00:31</span>
                                        <span class="log-level debug">DEBUG</span>
                                        <span class="log-message log-highlight">Users to process: CORP\Administrator, CORP\jsmith, CORP\smartinez, CORP\bwilson, CORP\emendez</span>
                                    </div>
                                    <div class="log-line">
                                        <span class="log-time">08:00:31</span>
                                        <span class="log-level info">INFO</span>
                                        <span class="log-message">Matched user session: CORP\smartinez → Sarah Martinez</span>
                                    </div>
                                    <div class="log-line">
                                        <span class="log-time">08:00:31</span>
                                        <span class="log-level info">INFO</span>
                                        <span class="log-message">App instance running for CORP\smartinez - skipping new instance</span>
                                    </div>
                                    <div class="log-line" style="color: var(--accent-orange);">
                                        <span class="log-time">08:00:31</span>
                                        <span class="log-level warn">WARN</span>
                                        <span class="log-message">No match found for tracked user: Alex Thompson (expected: CORP\athompson)</span>
                                    </div>
                                </div>
                            </div>

                            <div style="background: rgba(210, 153, 34, 0.1); border: 1px solid var(--accent-orange); padding: 1rem; border-radius: 8px; margin: 1rem 0;">
                                <h4 style="color: var(--accent-orange); font-size: 0.9rem; margin-bottom: 0.5rem;">⚠️ Critical Finding from Dev Team</h4>
                                <p style="font-size: 0.85rem; color: var(--text-secondary);">
                                    "The Silent App doesn't fully support concurrent multi-user RDS sessions. If an app instance is already running 
                                    for one user, new instances won't start for other users. This is a known limitation with a task to improve it."
                                </p>
                            </div>

                            <div class="question-section">
                                <div class="question-text">Based on this information, what's the root cause?</div>
                                <div class="options" id="q3-options">
                                    <div class="option" onclick="selectOption(3, 0, this)">
                                        <div class="option-marker">A</div>
                                        <div class="option-text">The user's account credentials are incorrect</div>
                                    </div>
                                    <div class="option" onclick="selectOption(3, 1, this)">
                                        <div class="option-marker">B</div>
                                        <div class="option-text">Another user's app instance is preventing Alex's from starting (multi-user limitation)</div>
                                    </div>
                                    <div class="option" onclick="selectOption(3, 2, this)">
                                        <div class="option-marker">C</div>
                                        <div class="option-text">The server's firewall is blocking the connection</div>
                                    </div>
                                    <div class="option" onclick="selectOption(3, 3, this)">
                                        <div class="option-marker">D</div>
                                        <div class="option-text">Alex needs to manually start the app each day</div>
                                    </div>
                                </div>
                                <div class="feedback" id="q3-feedback"></div>
                            </div>

                            <div class="btn-group">
                                <button class="btn btn-secondary" onclick="goToStep(2)">← Back</button>
                                <button class="btn btn-primary" id="next-3" onclick="nextStep(3)" disabled>Continue to Resolution →</button>
                            </div>
                        </div>
                    </div>
                </div>

                <!-- Step 4: Resolution -->
                <div class="step-panel" id="step-4">
                    <div class="card">
                        <div class="card-header">
                            <h3>✅ Resolution Strategy</h3>
                        </div>
                        <div class="card-body">
                            <p style="color: var(--text-secondary); margin-bottom: 1.5rem;">
                                Now that you understand the root cause, you need to determine the best resolution path.
                            </p>

                            <h4 style="font-size: 0.95rem; margin-bottom: 1rem;">Available Options:</h4>
                            
                            <div style="display: flex; flex-direction: column; gap: 1rem; margin-bottom: 1.5rem;">
                                <div style="background: var(--bg-card); padding: 1rem; border-radius: 8px; border-left: 3px solid var(--accent-green);">
                                    <h5 style="font-size: 0.9rem; color: var(--accent-green); margin-bottom: 0.5rem;">Short-term Workaround</h5>
                                    <p style="font-size: 0.85rem; color: var(--text-secondary);">
                                        Have Sarah Martinez (the working user) fully log out of the RDS server, then have Alex log in. 
                                        The app should start for Alex's session. Coordinate timing to minimize disruption.
                                    </p>
                                </div>
                                
                                <div style="background: var(--bg-card); padding: 1rem; border-radius: 8px; border-left: 3px solid var(--accent-blue);">
                                    <h5 style="font-size: 0.9rem; color: var(--accent-blue); margin-bottom: 0.5rem;">Medium-term Solution</h5>
                                    <p style="font-size: 0.85rem; color: var(--text-secondary);">
                                        Work with the customer to implement session management policies - ensure tracked users 
                                        fully log out (not just disconnect) at end of day so sessions restart fresh.
                                    </p>
                                </div>
                                
                                <div style="background: var(--bg-card); padding: 1rem; border-radius: 8px; border-left: 3px solid var(--accent-purple);">
                                    <h5 style="font-size: 0.9rem; color: var(--accent-purple); margin-bottom: 0.5rem;">Long-term Fix</h5>
                                    <p style="font-size: 0.85rem; color: var(--text-secondary);">
                                        Engineering has a task to improve multi-user RDS support. This would allow concurrent 
                                        app instances for different user sessions. No ETA currently.
                                    </p>
                                </div>
                            </div>

                            <div class="question-section">
                                <div class="question-text">What should you tell the customer about the resolution?</div>
                                <div class="options" id="q4-options">
                                    <div class="option" onclick="selectOption(4, 0, this)">
                                        <div class="option-marker">A</div>
                                        <div class="option-text">"This is a bug and we can't help until engineering fixes it."</div>
                                    </div>
                                    <div class="option" onclick="selectOption(4, 1, this)">
                                        <div class="option-marker">B</div>
                                        <div class="option-text">"Multi-user RDS has limitations. For now, have other tracked users log out fully before Alex logs in. We're working on improving this."</div>
                                    </div>
                                    <div class="option" onclick="selectOption(4, 2, this)">
                                        <div class="option-marker">C</div>
                                        <div class="option-text">"Just reinstall the app and it will work."</div>
                                    </div>
                                    <div class="option" onclick="selectOption(4, 3, this)">
                                        <div class="option-marker">D</div>
                                        <div class="option-text">"Alex should use a different computer instead of RDS."</div>
                                    </div>
                                </div>
                                <div class="feedback" id="q4-feedback"></div>
                            </div>

                            <div class="btn-group">
                                <button class="btn btn-secondary" onclick="goToStep(3)">← Back</button>
                                <button class="btn btn-primary" id="next-4" onclick="nextStep(4)" disabled>View Summary →</button>
                            </div>
                        </div>
                    </div>
                </div>

                <!-- Step 5: Summary -->
                <div class="step-panel" id="step-5">
                    <div class="card">
                        <div class="card-header">
                            <h3>📊 Case Summary</h3>
                        </div>
                        <div class="card-body">
                            <div class="summary-stat">
                                <span class="stat-label">Questions Answered</span>
                                <span class="stat-value" id="total-answered">0/4</span>
                            </div>
                            <div class="summary-stat">
                                <span class="stat-label">Correct Answers</span>
                                <span class="stat-value good" id="total-correct">0/4</span>
                            </div>

                            <div style="margin-top: 1.5rem;">
                                <h4 style="font-size: 1rem; margin-bottom: 1rem;">Key Takeaways:</h4>
                                <div class="findings-list">
                                    <div class="finding-item">
                                        <span class="finding-icon">📋</span>
                                        <div>
                                            <strong>Always start with logs</strong> - They tell the story of what actually happened vs. what the user reports.
                                        </div>
                                    </div>
                                    <div class="finding-item">
                                        <span class="finding-icon">🔍</span>
                                        <div>
                                            <strong>Look for the last action</strong> - The LOGOFF signal at 19:06:09 was the critical moment. No logs after = app never restarted.
                                        </div>
                                    </div>
                                    <div class="finding-item">
                                        <span class="finding-icon">🖥️</span>
                                        <div>
                                            <strong>RDS environments are complex</strong> - Multi-user sessions can have unexpected interactions. Not all apps support concurrent instances.
                                        </div>
                                    </div>
                                    <div class="finding-item">
                                        <span class="finding-icon">💬</span>
                                        <div>
                                            <strong>Communicate transparently</strong> - Acknowledge limitations, provide workarounds, and set expectations for future improvements.
                                        </div>
                                    </div>
                                </div>
                            </div>

                            <div class="knowledge-callout" style="margin-top: 1.5rem;">
                                <h4>🎯 Support Engineer Skills Demonstrated</h4>
                                <p>This case required: log analysis, understanding RDS architecture, collaboration with engineering, 
                                and crafting a customer response that provides both immediate solutions and long-term expectations.</p>
                            </div>

                            <div class="btn-group">
                                <button class="btn btn-secondary" onclick="goToStep(1)">← Start Over</button>
                                <button class="btn btn-primary" onclick="window.print()">🖨️ Print Summary</button>
                            </div>
                        </div>
                    </div>
                </div>
            </div>

            <div class="sidebar">
                <div class="card">
                    <div class="card-header">
                        <h3>🖥️ Environment</h3>
                    </div>
                    <div class="card-body environment-diagram">
                        <div class="server-box">
                            <div class="server-header">
                                <span class="server-icon">🖥️</span>
                                <div>
                                    <div class="server-name">HQ-RDS01</div>
                                    <div class="server-type">Windows Server 2019 • RDS Host</div>
                                </div>
                            </div>
                            <div class="user-sessions">
                                <div class="user-session">
                                    <span class="session-status not-tracking"></span>
                                    <span class="session-user">athompson</span>
                                    <span class="session-badge problem">Not Tracking</span>
                                </div>
                                <div class="user-session">
                                    <span class="session-status tracking"></span>
                                    <span class="session-user">smartinez</span>
                                    <span class="session-badge">Tracking</span>
                                </div>
                                <div class="user-session">
                                    <span class="session-status idle"></span>
                                    <span class="session-user">jsmith</span>
                                    <span class="session-badge" style="background: rgba(210, 153, 34, 0.15); color: var(--accent-orange);">Not Tracked</span>
                                </div>
                                <div class="user-session">
                                    <span class="session-status idle"></span>
                                    <span class="session-user">+5 others</span>
                                </div>
                            </div>
                        </div>
                        <p style="font-size: 0.8rem; color: var(--text-muted); text-align: center;">
                            Multiple simultaneous RDP sessions active
                        </p>
                    </div>
                </div>

                <div class="card">
                    <div class="card-header">
                        <h3>🔑 Key Facts</h3>
                    </div>
                    <div class="card-body">
                        <div class="findings-list">
                            <div class="finding-item">
                                <span class="finding-icon">📅</span>
                                <div>Last activity: Dec 18, 2025</div>
                            </div>
                            <div class="finding-item">
                                <span class="finding-icon">👥</span>
                                <div>~170 seats, 8-10+ concurrent users</div>
                            </div>
                            <div class="finding-item">
                                <span class="finding-icon">✅</span>
                                <div>Sarah Martinez tracking normally</div>
                            </div>
                            <div class="finding-item">
                                <span class="finding-icon">📱</span>
                                <div>Silent App v1.7.9</div>
                            </div>
                        </div>
                    </div>
                </div>

                <div class="card">
                    <div class="card-header">
                        <h3>📚 Quick Reference</h3>
                    </div>
                    <div class="card-body" style="font-size: 0.85rem;">
                        <p style="color: var(--text-secondary); margin-bottom: 0.75rem;"><strong>Log Locations:</strong></p>
                        <ul style="color: var(--text-muted); padding-left: 1rem; margin-bottom: 1rem;">
                            <li>Client logs: User's AppData</li>
                            <li>Service logs: <code style="font-size: 0.75rem;">C:\ProgramData\Netsoft-Logs\Corporate</code></li>
                        </ul>
                        <p style="color: var(--text-secondary); margin-bottom: 0.75rem;"><strong>Key Log Signals:</strong></p>
                        <ul style="color: var(--text-muted); padding-left: 1rem;">
                            <li><code>LOGOFF</code> = Session ended</li>
                            <li><code>SHUTDOWN</code> = App closed</li>
                            <li>No entries = App not running</li>
                        </ul>
                    </div>
                </div>
            </div>
        </div>
    </div>

    <script>
        let currentStep = 1;
        let answers = {};
        const correctAnswers = { 1: 1, 2: 2, 3: 1, 4: 1 };
        
        const feedback = {
            1: {
                correct: {
                    title: "✅ Correct!",
                    text: "Log analysis is always the first step. The logs tell you exactly what happened - when the app started, ran, and stopped. They're your primary diagnostic tool before making any changes."
                },
                incorrect: {
                    title: "❌ Not quite",
                    text: "Before taking any action like reinstalling or restarting, you need to understand what's actually happening. Request and analyze the client logs first - they'll show you exactly when and why the app stopped working."
                }
            },
            2: {
                correct: {
                    title: "✅ Correct!",
                    text: "The clean LOGOFF → SHUTDOWN sequence with no subsequent entries means the app closed properly but never started again. If it was crashing, you'd see startup attempts. If uninstalled, there'd be no prior logs. The app simply isn't being launched for this user's session."
                },
                incorrect: {
                    title: "❌ Not quite",
                    text: "The logs show a clean shutdown sequence. No entries after that point indicates the app never started again for this user - not that it crashed or was uninstalled. The service is still running (checking for updates), but it's not launching the app for this particular session."
                }
            },
            3: {
                correct: {
                    title: "✅ Correct!",
                    text: "The Silent App has a known limitation with concurrent multi-user RDS sessions. When Sarah's app instance is running, the service won't start a new instance for Alex. This is why 'it works for others but not this one user' - the first active tracked user's session 'wins'."
                },
                incorrect: {
                    title: "❌ Not quite",
                    text: "The service logs reveal the real issue: in multi-user RDS environments, only one app instance runs at a time. Sarah Martinez's session already has an active instance, which prevents Alex's from starting. This is a product limitation, not a configuration or credential issue."
                }
            },
            4: {
                correct: {
                    title: "✅ Excellent communication!",
                    text: "This response acknowledges the limitation honestly, provides an actionable workaround (coordinating logouts), and sets expectations for future improvements. It's helpful, transparent, and maintains trust while being realistic about current capabilities."
                },
                incorrect: {
                    title: "❌ Not the best approach",
                    text: "Good customer communication acknowledges limitations while providing practical workarounds. The best response explains the multi-user limitation, offers the session coordination workaround, and mentions ongoing improvement work - keeping the customer informed and supported."
                }
            }
        };

        function selectOption(question, optionIndex, element) {
            // Clear previous selection
            const options = document.querySelectorAll(`#q${question}-options .option`);
            options.forEach(opt => opt.classList.remove('selected', 'correct', 'incorrect'));
            
            // Mark selection
            element.classList.add('selected');
            answers[question] = optionIndex;
            
            // Check answer
            const isCorrect = optionIndex === correctAnswers[question];
            
            // Show feedback
            const feedbackEl = document.getElementById(`q${question}-feedback`);
            feedbackEl.className = `feedback show ${isCorrect ? 'correct' : 'incorrect'}`;
            feedbackEl.innerHTML = `
                <h4>${feedback[question][isCorrect ? 'correct' : 'incorrect'].title}</h4>
                <p>${feedback[question][isCorrect ? 'correct' : 'incorrect'].text}</p>
            `;
            
            // Mark option as correct/incorrect
            element.classList.remove('selected');
            element.classList.add(isCorrect ? 'correct' : 'incorrect');
            
            // If incorrect, show correct answer
            if (!isCorrect) {
                options[correctAnswers[question]].classList.add('correct');
            }
            
            // Enable next button
            document.getElementById(`next-${question}`).disabled = false;
        }

        function nextStep(fromStep) {
            goToStep(fromStep + 1);
        }

        function goToStep(step) {
            // Update progress
            document.querySelectorAll('.progress-step').forEach((el, i) => {
                el.classList.remove('active', 'complete');
                if (i + 1 < step) el.classList.add('complete');
                if (i + 1 === step) el.classList.add('active');
            });
            
            // Show correct panel
            document.querySelectorAll('.step-panel').forEach(panel => {
                panel.classList.remove('active');
            });
            document.getElementById(`step-${step}`).classList.add('active');
            
            currentStep = step;
            
            // Update summary if on last step
            if (step === 5) {
                updateSummary();
            }
        }

        function updateSummary() {
            const answered = Object.keys(answers).length;
            const correct = Object.keys(answers).filter(q => answers[q] === correctAnswers[q]).length;
            
            document.getElementById('total-answered').textContent = `${answered}/4`;
            document.getElementById('total-correct').textContent = `${correct}/4`;
        }
    </script>
</body>
</html>
