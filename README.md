import React, { useState, useEffect, useRef } from ‘react’;
import * as Chart from ‘chart.js/auto’;
import { Clock, Users, TrendingUp, Calendar, AlertCircle, CheckCircle, Upload, Download } from ‘lucide-react’;
import Papa from ‘papaparse’;

const TimeAttendanceDashboard = () => {
const initialData = [
{ user: “adrian”, loginTime: “2025-10-20 08:09:00”, logoutTime: “2025-10-20 17:02:00”, workTime: “08:53” },
{ user: “adrian”, loginTime: “2025-10-21 08:30:00”, logoutTime: “2025-10-21 17:04:00”, workTime: “08:34” },
{ user: “adrian”, loginTime: “2025-10-22 08:31:00”, logoutTime: “2025-10-22 17:08:00”, workTime: “08:37” },
{ user: “adrian”, loginTime: “2025-10-23 08:07:00”, logoutTime: “2025-10-23 16:09:00”, workTime: “08:02” },
{ user: “ajimenez”, loginTime: “2025-10-20 08:38:00”, logoutTime: “2025-10-20 16:56:00”, workTime: “08:16” },
{ user: “ajimenez”, loginTime: “2025-10-21 08:43:00”, logoutTime: “2025-10-21 16:46:00”, workTime: “08:03” },
{ user: “ajimenez”, loginTime: “2025-10-22 08:54:00”, logoutTime: “2025-10-22 17:05:00”, workTime: “08:11” },
{ user: “ajimenez”, loginTime: “2025-10-23 08:38:00”, logoutTime: “2025-10-23 16:57:00”, workTime: “08:19” },
{ user: “Andrea”, loginTime: “2025-10-20 08:38:00”, logoutTime: “2025-10-20 16:55:00”, workTime: “08:17” },
{ user: “Andrea”, loginTime: “2025-10-21 08:37:00”, logoutTime: “2025-10-21 19:28:00”, workTime: “10:51” },
{ user: “Arturo”, loginTime: “2025-10-22 08:03:00”, logoutTime: “2025-10-22 17:23:00”, workTime: “09:20” },
{ user: “Arturo”, loginTime: “2025-10-23 08:41:00”, logoutTime: “2025-10-23 17:25:00”, workTime: “08:44” },
{ user: “brenda”, loginTime: “2025-10-20 08:43:00”, logoutTime: “2025-10-20 17:07:00”, workTime: “08:24” },
{ user: “brenda”, loginTime: “2025-10-21 08:37:00”, logoutTime: “2025-10-21 17:16:00”, workTime: “08:39” },
{ user: “brenda”, loginTime: “2025-10-22 08:41:00”, logoutTime: “2025-10-22 17:25:00”, workTime: “08:44” },
{ user: “brenda”, loginTime: “2025-10-23 08:38:00”, logoutTime: “2025-10-23 17:19:00”, workTime: “08:41” },
{ user: “carolina”, loginTime: “2025-10-20 09:23:00”, logoutTime: “2025-10-20 16:59:00”, workTime: “07:36” },
{ user: “carolina”, loginTime: “2025-10-21 08:40:00”, logoutTime: “2025-10-21 17:03:00”, workTime: “08:23” },
{ user: “carolina”, loginTime: “2025-10-22 08:47:00”, logoutTime: “2025-10-22 17:03:00”, workTime: “08:16” },
{ user: “carolina”, loginTime: “2025-10-23 08:27:00”, logoutTime: “2025-10-23 17:03:00”, workTime: “08:36” },
];

const [rawData, setRawData] = useState(initialData);
const [selectedUser, setSelectedUser] = useState(‘all’);
const [selectedDate, setSelectedDate] = useState(‘all’);
const fileInputRef = useRef(null);

const chartRefs = {
userHours: useRef(null),
punctuality: useRef(null),
dailyTrend: useRef(null),
avgHours: useRef(null),
loginPattern: useRef(null)
};

const chartInstances = useRef({});

const handleFileUpload = (event) => {
const file = event.target.files[0];
if (file) {
Papa.parse(file, {
header: true,
skipEmptyLines: true,
complete: (results) => {
const normalizedData = results.data.map(row => ({
user: row.User || row.user || ‘’,
loginTime: row[‘Login Time’] || row.loginTime || row[‘login time’] || ‘’,
logoutTime: row[‘Logout Time’] || row.logoutTime || row[‘logout time’] || ‘’,
workTime: row[‘Work Time’] || row.workTime || row[‘work time’] || ‘’
})).filter(row => row.user && row.loginTime);

```
      if (normalizedData.length > 0) {
        setRawData(normalizedData);
        alert(`Successfully loaded ${normalizedData.length} records!`);
      } else {
        alert('No valid data found in CSV. Please check the format.');
      }
    },
    error: (error) => {
      alert('Error parsing CSV file: ' + error.message);
    }
  });
}
```

};

const downloadHTML = () => {
const htmlContent = `<!DOCTYPE html>

<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Time & Attendance Dashboard</title>
    <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/PapaParse/5.3.2/papaparse.min.js"></script>
    <style>
        * { margin: 0; padding: 0; box-sizing: border-box; }
        body { font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, sans-serif; background: linear-gradient(to bottom right, #f9fafb, #f3f4f6); padding: 24px; }
        .container { max-width: 1280px; margin: 0 auto; }
        .header { margin-bottom: 24px; }
        .header h1 { font-size: 30px; font-weight: bold; color: #111827; margin-bottom: 8px; }
        .header p { color: #6b7280; }
        .controls { background: white; border-radius: 12px; box-shadow: 0 1px 3px rgba(0,0,0,0.1); padding: 20px; margin-bottom: 24px; display: flex; gap: 16px; flex-wrap: wrap; }
        .control-group { flex: 1; min-width: 200px; }
        .control-group label { display: block; font-size: 14px; font-weight: 600; color: #374151; margin-bottom: 8px; }
        .control-group select, .control-group input { width: 100%; padding: 10px 12px; border: 1px solid #d1d5db; border-radius: 8px; font-size: 14px; }
        .btn { padding: 10px 16px; border: none; border-radius: 8px; font-size: 14px; font-weight: 600; cursor: pointer; display: inline-flex; align-items: center; gap: 8px; }
        .btn-primary { background: #3b82f6; color: white; }
        .btn-primary:hover { background: #2563eb; }
        .btn-secondary { background: #10b981; color: white; }
        .btn-secondary:hover { background: #059669; }
        .kpi-grid { display: grid; grid-template-columns: repeat(auto-fit, minmax(250px, 1fr)); gap: 24px; margin-bottom: 24px; }
        .kpi-card { background: white; border-radius: 12px; box-shadow: 0 1px 3px rgba(0,0,0,0.1); padding: 24px; border-left: 4px solid; }
        .kpi-card.blue { border-left-color: #3b82f6; }
        .kpi-card.green { border-left-color: #10b981; }
        .kpi-card.purple { border-left-color: #8b5cf6; }
        .kpi-card.orange { border-left-color: #f59e0b; }
        .kpi-card.emerald { border-left-color: #10b981; }
        .kpi-card.red { border-left-color: #ef4444; }
        .kpi-label { font-size: 14px; color: #6b7280; font-weight: 500; }
        .kpi-value { font-size: 36px; font-weight: bold; color: #111827; margin-top: 8px; }
        .kpi-subtitle { font-size: 12px; color: #9ca3af; margin-top: 4px; }
        .chart-grid { display: grid; grid-template-columns: repeat(auto-fit, minmax(500px, 1fr)); gap: 24px; margin-bottom: 24px; }
        .chart-card { background: white; border-radius: 12px; box-shadow: 0 1px 3px rgba(0,0,0,0.1); padding: 24px; border: 1px solid #e5e7eb; }
        .chart-title { font-size: 18px; font-weight: bold; color: #111827; margin-bottom: 16px; }
        .chart-container { height: 320px; position: relative; }
        .table-card { background: white; border-radius: 12px; box-shadow: 0 1px 3px rgba(0,0,0,0.1); padding: 24px; border: 1px solid #e5e7eb; overflow-x: auto; }
        table { width: 100%; border-collapse: collapse; }
        thead { background: #f9fafb; }
        th { padding: 12px 16px; text-align: left; font-size: 12px; font-weight: 700; color: #6b7280; text-transform: uppercase; border-bottom: 2px solid #e5e7eb; }
        td { padding: 12px 16px; font-size: 14px; color: #374151; border-bottom: 1px solid #e5e7eb; }
        tbody tr:hover { background: #f9fafb; }
        .badge { display: inline-block; padding: 4px 10px; border-radius: 12px; font-size: 12px; font-weight: 600; }
        .badge.green { background: #d1fae5; color: #065f46; }
        .badge.yellow { background: #fef3c7; color: #92400e; }
        .badge.red { background: #fee2e2; color: #991b1b; }
        .badge.gray { background: #f3f4f6; color: #374151; }
        .badge.orange { background: #fed7aa; color: #9a3412; }
    </style>
</head>
<body>
    <div class="container">
        <div class="header">
            <h1>⏰ Time & Attendance Dashboard</h1>
            <p>Comprehensive analysis of employee login and logout records</p>
        </div>

```
    <div class="controls">
        <div class="control-group">
            <label>Upload CSV File</label>
            <input type="file" id="csvFile" accept=".csv" />
        </div>
        <div class="control-group">
            <label>Filter by Employee</label>
            <select id="userFilter">
                <option value="all">All Employees</option>
            </select>
        </div>
        <div class="control-group">
            <label>Filter by Date</label>
            <select id="dateFilter">
                <option value="all">All Dates</option>
            </select>
        </div>
    </div>

    <div class="kpi-grid">
        <div class="kpi-card blue">
            <div class="kpi-label">Total Hours Worked</div>
            <div class="kpi-value" id="kpi-total">0.0</div>
            <div class="kpi-subtitle">Across all employees</div>
        </div>
        <div class="kpi-card green">
            <div class="kpi-label">Average Hours/Day</div>
            <div class="kpi-value" id="kpi-avg">0.00</div>
            <div class="kpi-subtitle">Per employee per day</div>
        </div>
        <div class="kpi-card purple">
            <div class="kpi-label">Active Employees</div>
            <div class="kpi-value" id="kpi-users">0</div>
            <div class="kpi-subtitle">Unique users tracked</div>
        </div>
        <div class="kpi-card orange">
            <div class="kpi-label">Average Login Time</div>
            <div class="kpi-value" id="kpi-login">0:00</div>
            <div class="kpi-subtitle">Mean arrival time</div>
        </div>
        <div class="kpi-card emerald">
            <div class="kpi-label">Punctuality Rate</div>
            <div class="kpi-value" id="kpi-punctuality">0%</div>
            <div class="kpi-subtitle">On-time arrivals ≤8:30</div>
        </div>
        <div class="kpi-card red">
            <div class="kpi-label">Overtime Instances</div>
            <div class="kpi-value" id="kpi-overtime">0</div>
            <div class="kpi-subtitle">Days with >9 hours</div>
        </div>
    </div>

    <div class="chart-grid">
        <div class="chart-card">
            <div class="chart-title">Total Hours by Employee</div>
            <div class="chart-container"><canvas id="chartUserHours"></canvas></div>
        </div>
        <div class="chart-card">
            <div class="chart-title">Punctuality Analysis</div>
            <div class="chart-container"><canvas id="chartPunctuality"></canvas></div>
        </div>
    </div>

    <div class="chart-grid">
        <div class="chart-card">
            <div class="chart-title">Daily Trend - Average Hours</div>
            <div class="chart-container"><canvas id="chartDailyTrend"></canvas></div>
        </div>
        <div class="chart-card">
            <div class="chart-title">Login Time Distribution</div>
            <div class="chart-container"><canvas id="chartLoginPattern"></canvas></div>
        </div>
    </div>

    <div class="table-card">
        <div class="chart-title">Detailed Employee Report</div>
        <table id="employeeTable">
            <thead>
                <tr>
                    <th>Employee</th>
                    <th>Total Hours</th>
                    <th>Days Worked</th>
                    <th>Avg Hrs/Day</th>
                    <th>Avg Login</th>
                    <th>Punctuality</th>
                    <th>Overtime</th>
                </tr>
            </thead>
            <tbody></tbody>
        </table>
    </div>
</div>

<script>
    let rawData = ${JSON.stringify(rawData)};
    let charts = {};

    function processData(data, userFilter, dateFilter) {
        return data.filter(d => d.workTime && d.workTime !== '-').map(d => {
            const [hours, minutes] = d.workTime.split(':').map(Number);
            const workHours = hours + (minutes / 60);
            const date = d.loginTime.split(' ')[0];
            const [loginHour, loginMinute] = d.loginTime.split(' ')[1].split(':').map(Number);
            const loginDecimal = loginHour + (loginMinute / 60);
            
            return {
                user: d.user.toLowerCase(),
                loginTime: d.loginTime,
                logoutTime: d.logoutTime,
                workTime: d.workTime,
                workHours: workHours,
                date: date,
                loginHour: loginDecimal,
                isPunctual: loginDecimal <= 8.5,
                isOvertime: workHours > 9
            };
        }).filter(d => {
            const userMatch = userFilter === 'all' || d.user === userFilter;
            const dateMatch = dateFilter === 'all' || d.date === dateFilter;
            return userMatch && dateMatch;
        });
    }

    function updateDashboard() {
        const userFilter = document.getElementById('userFilter').value;
        const dateFilter = document.getElementById('dateFilter').value;
        const filteredData = processData(rawData, userFilter, dateFilter);

        const kpis = {
            totalHours: filteredData.reduce((sum, d) => sum + d.workHours, 0).toFixed(1),
            avgHours: filteredData.length > 0 ? (filteredData.reduce((sum, d) => sum + d.workHours, 0) / filteredData.length).toFixed(2) : '0.00',
            uniqueUsers: new Set(filteredData.map(d => d.user)).size,
            avgLoginTime: filteredData.length > 0 ? (() => {
                const avg = filteredData.reduce((sum, d) => sum + d.loginHour, 0) / filteredData.length;
                return Math.floor(avg) + ':' + String(Math.round((avg % 1) * 60)).padStart(2, '0');
            })() : '0:00',
            punctualityRate: filteredData.length > 0 ? ((filteredData.filter(d => d.isPunctual).length / filteredData.length) * 100).toFixed(0) : '0',
            overtimeCount: filteredData.filter(d => d.isOvertime).length
        };

        document.getElementById('kpi-total').textContent = kpis.totalHours;
        document.getElementById('kpi-avg').textContent = kpis.avgHours;
        document.getElementById('kpi-users').textContent = kpis.uniqueUsers;
        document.getElementById('kpi-login').textContent = kpis.avgLoginTime;
        document.getElementById('kpi-punctuality').textContent = kpis.punctualityRate + '%';
        document.getElementById('kpi-overtime').textContent = kpis.overtimeCount;

        const userSummary = Object.values(
            filteredData.reduce((acc, d) => {
                if (!acc[d.user]) {
                    acc[d.user] = { user: d.user, totalHours: 0, days: 0, totalLogin: 0, punctualDays: 0, overtimeDays: 0 };
                }
                acc[d.user].totalHours += d.workHours;
                acc[d.user].days += 1;
                acc[d.user].totalLogin += d.loginHour;
                if (d.isPunctual) acc[d.user].punctualDays += 1;
                if (d.isOvertime) acc[d.user].overtimeDays += 1;
                return acc;
            }, {})
        ).map(s => ({
            user: s.user,
            totalHours: s.totalHours.toFixed(2),
            days: s.days,
            avgHours: (s.totalHours / s.days).toFixed(2),
            avgLogin: Math.floor(s.totalLogin / s.days) + ':' + String(Math.round(((s.totalLogin / s.days) % 1) * 60)).padStart(2, '0'),
            punctualityRate: ((s.punctualDays / s.days) * 100).toFixed(0),
            overtimeDays: s.overtimeDays
        })).sort((a, b) => parseFloat(b.totalHours) - parseFloat(a.totalHours));

        updateTable(userSummary);
        updateCharts(filteredData, userSummary);
    }

    function updateTable(userSummary) {
        const tbody = document.querySelector('#employeeTable tbody');
        tbody.innerHTML = userSummary.map(user => {
            const punctualityClass = parseInt(user.punctualityRate) >= 75 ? 'green' : parseInt(user.punctualityRate) >= 50 ? 'yellow' : 'red';
            const overtimeClass = user.overtimeDays === 0 ? 'gray' : user.overtimeDays <= 2 ? 'orange' : 'red';
            return \`<tr>
                <td style="font-weight:600">\${user.user}</td>
                <td>\${user.totalHours}</td>
                <td>\${user.days}</td>
                <td>\${user.avgHours}</td>
                <td>\${user.avgLogin}</td>
                <td><span class="badge \${punctualityClass}">\${user.punctualityRate}%</span></td>
                <td><span class="badge \${overtimeClass}">\${user.overtimeDays} days</span></td>
            </tr>\`;
        }).join('');
    }

    function updateCharts(filteredData, userSummary) {
        Object.values(charts).forEach(chart => chart && chart.destroy());

        charts.userHours = new Chart(document.getElementById('chartUserHours'), {
            type: 'bar',
            data: {
                labels: userSummary.map(u => u.user),
                datasets: [{ label: 'Total Hours', data: userSummary.map(u => parseFloat(u.totalHours)), backgroundColor: '#3b82f6', borderRadius: 6 }]
            },
            options: { responsive: true, maintainAspectRatio: false, plugins: { legend: { display: false } } }
        });

        const onTime = filteredData.filter(d => d.loginHour <= 8.5).length;
        const late = filteredData.filter(d => d.loginHour > 8.5).length;
        charts.punctuality = new Chart(document.getElementById('chartPunctuality'), {
            type: 'doughnut',
            data: {
                labels: ['On Time (≤8:30)', 'Late (>8:30)'],
                datasets: [{ data: [onTime, late], backgroundColor: ['#10b981', '#ef4444'] }]
            },
            options: { responsive: true, maintainAspectRatio: false }
        });

        const dailyData = Object.values(
            filteredData.reduce((acc, d) => {
                if (!acc[d.date]) acc[d.date] = { date: d.date, totalHours: 0, count: 0 };
                acc[d.date].totalHours += d.workHours;
                acc[d.date].count += 1;
                return acc;
            }, {})
        ).map(d => ({ date: d.date.substring(5), avg: d.totalHours / d.count })).sort((a, b) => a.date.localeCompare(b.date));

        charts.dailyTrend = new Chart(document.getElementById('chartDailyTrend'), {
            type: 'line',
            data: {
                labels: dailyData.map(d => d.date),
                datasets: [{ label: 'Average Hours', data: dailyData.map(d => d.avg), borderColor: '#8b5cf6', backgroundColor: 'rgba(139, 92, 246, 0.1)', tension: 0.4, fill: true }]
            },
            options: { responsive: true, maintainAspectRatio: false }
        });

        const timeSlots = { 'Before 7:00': 0, '7:00-8:00': 0, '8:00-8:30': 0, '8:30-9:00': 0, 'After 9:00': 0 };
        filteredData.forEach(d => {
            if (d.loginHour < 7) timeSlots['Before 7:00']++;
            else if (d.loginHour < 8) timeSlots['7:00-8:00']++;
            else if (d.loginHour < 8.5) timeSlots['8:00-8:30']++;
            else if (d.loginHour < 9) timeSlots['8:30-9:00']++;
            else timeSlots['After 9:00']++;
        });

        charts.loginPattern = new Chart(document.getElementById('chartLoginPattern'), {
            type: 'bar',
            data: {
                labels: Object.keys(timeSlots),
                datasets: [{ label: 'Login Count', data: Object.values(timeSlots), backgroundColor: ['#06b6d4', '#10b981', '#3b82f6', '#f59e0b', '#ef4444'], borderRadius: 6 }]
            },
            options: { responsive: true, maintainAspectRatio: false, plugins: { legend: { display: false } } }
        });
    }

    function updateFilters() {
        const users = ['all', ...new Set(rawData.map(d => d.user.toLowerCase()))].sort();
        const dates = ['all', ...new Set(rawData.map(d => d.loginTime.split(' ')[0]))].sort();

        const userSelect = document.getElementById('userFilter');
        userSelect.innerHTML = users.map(u => \`<option value="\${u}">\${u === 'all' ? 'All Employees' : u}</option>\`).join('');

        const dateSelect = document.getElementById('dateFilter');
        dateSelect.innerHTML = dates.map(d => \`<option value="\${d}">\${d === 'all' ? 'All Dates' : d}</option>\`).join('');
    }

    document.getElementById('csvFile').addEventListener('change', (e) => {
        const file = e.target.files[0];
        if (file) {
            Papa.parse(file, {
                header: true,
                skipEmptyLines: true,
                complete: (results) => {
                    const normalizedData = results.data.map(row => ({
                        user: row.User || row.user || '',
                        loginTime: row['Login Time'] || row.loginTime || row['login time'] || '',
                        logoutTime: row['Logout Time'] || row.logoutTime || row['logout time'] || '',
                        workTime: row['Work Time'] || row.workTime || row['work time'] || ''
                    })).filter(row => row.user && row.loginTime);
                    
                    if (normalizedData.length > 0) {
                        rawData = normalizedData;
                        updateFilters();
                        updateDashboard();
                        alert('Successfully loaded ' + normalizedData.length + ' records!');
                    }
                }
            });
        }
    });

    document.getElementById('userFilter').addEventListener('change', updateDashboard);
    document.getElementById('dateFilter').addEventListener('change', updateDashboard);

    updateFilters();
    updateDashboard();
</script>
```

</body>
</html>`;

```
const blob = new Blob([htmlContent], { type: 'text/html' });
const url = URL.createObjectURL(blob);
const a = document.createElement('a');
a.href = url;
a.download = 'time-attendance-dashboard.html';
document.body.appendChild(a);
a.click();
document.body.removeChild(a);
URL.revokeObjectURL(url);
```

};

const processedData = rawData.filter(d => d.workTime && d.workTime !== ‘-’).map(d => {
const [hours, minutes] = d.workTime.split(’:’).map(Number);
const workHours = hours + (minutes / 60);
const date = d.loginTime.split(’ ‘)[0];
const [loginHour, loginMinute] = d.loginTime.split(’ ‘)[1].split(’:’).map(Number);
const loginDecimal = loginHour + (loginMinute / 60);

```
return {
  user: d.user.toLowerCase(),
  loginTime: d.loginTime,
  logoutTime: d.logoutTime,
  workTime: d.workTime,
  workHours: workHours,
  date: date,
  loginHour: loginDecimal,
  isPunctual: loginDecimal <= 8.5,
  isOvertime: workHours > 9
};
```

});

const filteredData = processedData.filter(d => {
const userMatch = selectedUser === ‘all’ || d.user === selectedUser;
const dateMatch = selectedDate === ‘all’ || d.date === selectedDate;
return userMatch && dateMatch;
});

const users = [‘all’, …new Set(processedData.map(d => d.user))].sort();
const dates = [‘all’, …new Set(processedData.map(d => d.date))].sort();

const kpis = {
totalHours: filteredData.reduce((sum, d) => sum + d.workHours, 0).toFixed(1),
avgHours: filteredData.length > 0 ? (filteredData.reduce((sum, d) => sum + d.workHours, 0) / filteredData.length).toFixed(2) : ‘0.00’,
uniqueUsers: new Set(filteredData.map(d => d.user)).size,
avgLoginTime: filteredData.length > 0 ? (() => {
const avg = filteredData.reduce((sum, d) => sum + d.loginHour, 0) / filteredData.length;
return `${Math.floor(avg)}
