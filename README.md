<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Multi-User 9-Stage Job Workflow Management</title>
    <style>
        body {
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
            background-color: #f4f6f9;
            margin: 0;
            padding: 20px;
            color: #333;
        }
        .header {
            text-align: center;
            margin-bottom: 20px;
            position: relative;
        }
        .header h1 { margin: 0; color: #2c3e50; }
        .logout-area {
            position: absolute;
            right: 20px;
            top: 10px;
            font-size: 14px;
        }
        .logout-btn {
            background: #e74c3c;
            color: white;
            border: none;
            padding: 5px 10px;
            border-radius: 4px;
            cursor: pointer;
        }
        .login-container {
            max-width: 400px;
            margin: 100px auto;
            background: white;
            padding: 30px;
            border-radius: 8px;
            box-shadow: 0 4px 10px rgba(0,0,0,0.1);
        }
        .container {
            display: flex;
            gap: 20px;
            max-width: 1400px;
            margin: 0 auto;
        }
        .form-section {
            background: white;
            padding: 20px;
            border-radius: 8px;
            box-shadow: 0 4px 6px rgba(0,0,0,0.05);
            width: 300px;
            height: fit-content;
        }
        .board-section {
            flex: 1;
            background: white;
            padding: 20px;
            border-radius: 8px;
            box-shadow: 0 4px 6px rgba(0,0,0,0.05);
        }
        h2 {
            margin-top: 0;
            color: #34495e;
            font-size: 18px;
            border-bottom: 2px solid #ecf0f1;
            padding-bottom: 10px;
        }
        label {
            display: block;
            margin: 15px 0 5px;
            font-weight: 600;
            font-size: 14px;
        }
        input, textarea, select {
            width: 100%;
            padding: 10px;
            border: 1px solid #ccc;
            border-radius: 4px;
            box-sizing: border-box;
            font-size: 14px;
            background: white;
        }
        textarea { height: 80px; resize: vertical; }
        button {
            background: #3498db;
            color: white;
            border: none;
            padding: 12px;
            width: 100%;
            border-radius: 4px;
            cursor: pointer;
            font-weight: bold;
            font-size: 14px;
            margin-top: 15px;
        }
        button:hover { background: #2980b9; }
        table { width: 100%; border-collapse: collapse; margin-top: 10px; }
        th, td { padding: 12px 15px; text-align: left; border-bottom: 1px solid #e2e8f0; vertical-align: middle; }
        th { background-color: #f8fafc; color: #64748b; }
        .badge {
            display: inline-block;
            padding: 6px 12px;
            border-radius: 20px;
            font-size: 12px;
            font-weight: bold;
            background-color: #e0f2fe;
            color: #0369a1;
        }
        .action-btn { background: #10b981; padding: 6px 12px; width: auto; font-size: 12px; margin-top: 0; }
        .action-btn:hover { background: #059669; }
        .delete-btn { background: #ef4444; width: auto; padding: 6px 12px; margin-top: 0; margin-left: 5px; }
        .delete-btn:hover { background: #dc2626; }
        .stage-select { width: 150px; padding: 4px; font-size: 12px; display: inline-block; margin-right: 5px; }
        .empty-msg { text-align: center; color: #94a3b8; font-style: italic; }
        .hidden { display: none !important; }
        .user-banner { background: #e8f4fd; padding: 10px; border-radius: 5px; margin-bottom: 15px; font-weight: bold; color: #2b6cb0;}
    </style>
</head>
<body>

    <div id="loginPage" class="login-container">
        <h2 style="text-align: center;">Workflow System Login</h2>
        <p style="text-align: center; color: #7f8c8d; font-size: 13px;">Enter your Stage ID or Admin details</p>
        <label for="username">Username</label>
        <input type="text" id="username" placeholder="e.g. admin, design, print">
        
        <label for="password">Password</label>
        <input type="password" id="password" placeholder="••••••••">
        
        <button onclick="handleLogin()">Login</button>
    </div>

    <div id="appPage" class="hidden">
        <div class="header">
            <h1>Production Workflow Tracker</h1>
            <p>9-Stage Local Management System</p>
            <div class="logout-area">
                <span id="userDisplay"></span>
                <button class="logout-btn" onclick="handleLogout()">Logout</button>
            </div>
        </div>

        <div class="container">
            <div class="form-section" id="adminFormSection">
                <h2>Log New Job Order</h2>
                <label for="clientName">Client Name</label>
                <input type="text" id="clientName" placeholder="Customer Name">

                <label for="jobDesc">Job Description</label>
                <textarea id="jobDesc" placeholder="Details of the print order..."></textarea>

                <label for="initialStage">Start Job Directly at Stage:</label>
                <select id="initialStage">
                    </select>

                <button onclick="addJob()">Add to Pipeline</button>
            </div>

            <div class="board-section">
                <div class="user-banner" id="userBanner"></div>
                <h2>Job Dashboard</h2>
                <table>
                    <thead>
                        <tr>
                            <th style="width: 8%;">ID</th>
                            <th style="width: 20%;">Client</th>
                            <th style="width: 30%;">Description</th>
                            <th style="width: 20%;">Current Stage</th>
                            <th style="width: 22%;">Actions</th>
                        </tr>
                    </thead>
                    <tbody id="jobTableBody">
                        </tbody>
                </table>
            </div>
        </div>
    </div>

    <script>
        const STAGES = [
            "1. Order Received",
            "2. Design Process",
            "3. Printing",
            "4. Lamination/Finishing",
            "5. Quality Check",
            "6. Packing",
            "7. Billing/Invoice",
            "8. Payment Received",
            "9. Dispatched"
        ];

        const ACCOUNTS = {
            "admin": { password: "admin123", role: "admin", stageIndex: -1, label: "Administrator" },
            "order": { password: "123", role: "worker", stageIndex: 0, label: "Stage 1: Order Dept" },
            "design": { password: "123", role: "worker", stageIndex: 1, label: "Stage 2: Design Dept" },
            "print": { password: "123", role: "worker", stageIndex: 2, label: "Stage 3: Printing Dept" },
            "lamination": { password: "123", role: "worker", stageIndex: 3, label: "Stage 4: Lamination Dept" },
            "qc": { password: "123", role: "worker", stageIndex: 4, label: "Stage 5: Quality Check" },
            "packing": { password: "123", role: "worker", stageIndex: 5, label: "Stage 6: Packing Dept" },
            "billing": { password: "123", role: "worker", stageIndex: 6, label: "Stage 7: Billing Dept" },
            "payment": { password: "123", role: "worker", stageIndex: 7, label: "Stage 8: Accounts Dept" },
            "dispatch": { password: "123", role: "worker", stageIndex: 8, label: "Stage 9: Dispatch Dept" }
        };

        let currentUser = null;
        let jobs = JSON.parse(localStorage.getItem('secure_workflow_jobs_v2')) || [];

        // Initialize drop-down options for creation form
        const stageDropdown = document.getElementById('initialStage');
        STAGES.forEach((stage, idx) => {
            let opt = document.createElement('option');
            opt.value = idx;
            opt.innerText = stage;
            stageDropdown.appendChild(opt);
        });

        function saveToStorage() {
            localStorage.setItem('secure_workflow_jobs_v2', JSON.stringify(jobs));
        }

        function handleLogin() {
            const userIn = document.getElementById('username').value.trim().toLowerCase();
            const passIn = document.getElementById('password').value;

            if (ACCOUNTS[userIn] && ACCOUNTS[userIn].password === passIn) {
                currentUser = ACCOUNTS[userIn];
                currentUser.username = userIn;
                
                document.getElementById('loginPage').classList.add('hidden');
                document.getElementById('appPage').classList.remove('hidden');
                
                if (currentUser.role === 'admin') {
                    document.getElementById('adminFormSection').classList.remove('hidden');
                    document.getElementById('userBanner').innerText = "Logged in as Master Admin. You can jump jobs directly to any stage using the options below.";
                } else {
                    document.getElementById('adminFormSection').classList.add('hidden');
                    document.getElementById('userBanner').innerText = `Logged in as: ${currentUser.label}. Processing tasks for your department only.`;
                }
                
                document.getElementById('userDisplay').innerText = `👤 ${currentUser.label} | `;
                renderJobs();
            } else {
                alert("Invalid Username or Password.");
            }
        }

        function handleLogout() {
            currentUser = null;
            document.getElementById('username').value = '';
            document.getElementById('password').value = '';
            document.getElementById('appPage').classList.add('hidden');
            document.getElementById('loginPage').classList.remove('hidden');
        }

        function renderJobs() {
            const tbody = document.getElementById('jobTableBody');
            tbody.innerHTML = '';

            let visibleJobs = jobs;
            if (currentUser.role !== 'admin') {
                visibleJobs = jobs.filter(j => j.stageIndex === currentUser.stageIndex);
            }

            if (visibleJobs.length === 0) {
                tbody.innerHTML = `<tr><td colspan="5" class="empty-msg">No active tasks currently waiting.</td></tr>`;
                return;
            }

            visibleJobs.forEach(job => {
                const tr = document.createElement('tr');
                let actionCellHtml = '';

                if (currentUser.role === 'admin') {
                    // Admin Control: Direct Drop-down to move job anywhere + delete button
                    let dropdownHtml = `<select class="stage-select" onchange="changeStageDirectly('${job.id}', this.value)">`;
                    STAGES.forEach((stg, sIdx) => {
                        const selected = (job.stageIndex === sIdx) ? 'selected' : '';
                        dropdownHtml += `<option value="${sIdx}" ${selected}>${stg}</option>`;
                    });
                    dropdownHtml += `</select>`;

                    actionCellHtml = `
                        ${dropdownHtml}
                        <button class="delete-btn" onclick="deleteJob('${job.id}')">❌</button>
                    `;
                } else {
                    // Worker Control: Single sequential "Next" push button
                    const actionButtonLabel = (job.stageIndex === 8) ? 'Finish Job' : 'Pass Next ➡️';
                    actionCellHtml = `<button class="action-btn" onclick="advanceJob('${job.id}')">${actionButtonLabel}</button>`;
                }

                tr.innerHTML = `
                    <td><strong>#${job.id}</strong></td>
                    <td>${escapeHtml(job.client)}</td>
                    <td>${escapeHtml(job.desc)}</td>
                    <td><span class="badge">${STAGES[job.stageIndex]}</span></td>
                    <td>${actionCellHtml}</td>
                `;
                tbody.appendChild(tr);
            });
        }

        function addJob() {
            const clientInput = document.getElementById('clientName');
            const descInput = document.getElementById('jobDesc');
            const stageInput = document.getElementById('initialStage');
            
            const client = clientInput.value.trim();
            const desc = descInput.value.trim();
            const chosenStage = parseInt(stageInput.value);

            if (!client) { alert('Please enter a Client Name.'); return; }

            const newJob = {
                id: Date.now().toString().slice(-4),
                client: client,
                desc: desc || "No description details provided.",
                stageIndex: chosenStage
            };

            jobs.push(newJob);
            saveToStorage();
            renderJobs();

            clientInput.value = '';
            descInput.value = '';
            stageInput.value = 0; // reset to stage 1
        }

        function changeStageDirectly(id, targetStageIndex) {
            const job = jobs.find(j => j.id == id);
            if (job) {
                job.stageIndex = parseInt(targetStageIndex);
                saveToStorage();
                renderJobs();
            }
        }

        function advanceJob(id) {
            const job = jobs.find(j => j.id == id);
            if (job) {
                if (job.stageIndex < 8) {
                    job.stageIndex++;
                } else {
                    alert(`Job #${job.id} has cleared the workflow and is marked as Complete.`);
                    jobs = jobs.filter(j => j.id != id);
                }
                saveToStorage();
                renderJobs();
            }
        }

        function deleteJob(id) {
            if (!confirm("Delete this job permanently?")) return;
            jobs = jobs.filter(j => j.id != id);
            saveToStorage();
            renderJobs();
        }

        function escapeHtml(str) {
            return str.replace(/&/g, "&amp;").replace(/</g, "&lt;").replace(/>/g, "&gt;").replace(/"/g, "&quot;").replace(/'/g, "&#039;");
        }
    </script>
</body>
</html>
