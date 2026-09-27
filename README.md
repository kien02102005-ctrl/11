<!DOCTYPE html>
<html lang="vi">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>WORKSPACE - Khoa Hóa Lý (Realtime KPI Master Sync)</title>
    <!-- Chart.js -->
    <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
    <!-- FontAwesome Icon -->
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.4.0/css/all.min.css">
    
    <!-- Firebase SDK (Modular v9/v10 Compat) -->
    <script src="https://www.gstatic.com/firebasejs/10.8.0/firebase-app-compat.js"></script>
    <script src="https://www.gstatic.com/firebasejs/10.8.0/firebase-database-compat.js"></script>

    <!-- Thư viện docx & FileSaver -->
    <script src="https://unpkg.com/docx@8.5.0/build/index.umd.js"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/FileSaver.js/2.0.5/FileSaver.min.js"></script>

    <style>
        :root {
            --primary: #1e40af;
            --primary-hover: #1d4ed8;
            --sidebar-bg: #0f172a;
            --bg-light: #f1f5f9;
        }

        * {
            box-sizing: border-box;
            margin: 0;
            padding: 0;
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
        }

        html, body {
            width: 100%;
            height: 100%;
            margin: 0;
            padding: 0;
            background-color: var(--bg-light);
            overflow-x: auto;
            overflow-y: auto;
        }

        /* --- 1. GIAO DIỆN ĐĂNG NHẬP --- */
        #login-screen {
            position: fixed;
            top: 0; left: 0; width: 100vw; height: 100vh;
            background: linear-gradient(135deg, #1e3a8a, #3b82f6);
            display: flex;
            justify-content: center;
            align-items: center;
            z-index: 9999;
        }

        .login-card {
            background: #fff;
            padding: 40px 30px;
            border-radius: 12px;
            box-shadow: 0 10px 25px rgba(0,0,0,0.2);
            width: 380px;
            max-width: 90%;
            text-align: center;
        }

        .login-card .icon {
            font-size: 48px;
            color: #2563eb;
            margin-bottom: 15px;
        }

        .login-card h2 {
            color: #1e293b;
            margin-bottom: 5px;
            text-transform: uppercase;
        }

        .login-card p {
            color: #64748b;
            font-size: 13px;
            margin-bottom: 25px;
        }

        .form-group {
            text-align: left;
            margin-bottom: 18px;
        }

        .form-group label {
            display: block;
            font-size: 13px;
            font-weight: 600;
            color: #334155;
            margin-bottom: 5px;
        }

        .form-group input, .form-group select {
            width: 100%;
            padding: 10px 12px;
            border: 1px solid #cbd5e1;
            border-radius: 6px;
            outline: none;
            font-size: 14px;
        }

        .form-group input:focus, .form-group select:focus {
            border-color: #2563eb;
        }

        .btn-login {
            width: 100%;
            padding: 12px;
            background-color: #2563eb;
            color: #fff;
            border: none;
            border-radius: 6px;
            font-weight: bold;
            font-size: 15px;
            cursor: pointer;
            transition: 0.2s;
        }

        .btn-login:hover {
            background-color: #1d4ed8;
        }

        /* --- 2. GIAO DIỆN CHÍNH --- */
        #app-screen {
            display: flex;
            width: 100%;
            min-width: 1200px;
            min-height: 100vh;
        }

        .sidebar {
            width: 250px;
            background-color: var(--sidebar-bg);
            color: #fff;
            display: flex;
            flex-direction: column;
            flex-shrink: 0;
        }

        .sidebar-brand {
            padding: 20px;
            font-size: 20px;
            font-weight: bold;
            display: flex;
            align-items: center;
            gap: 10px;
            border-bottom: 1px solid #334155;
        }

        .user-profile {
            padding: 15px 20px;
            border-bottom: 1px solid #334155;
        }

        .user-profile .name {
            font-weight: 600;
            font-size: 15px;
        }

        .user-profile .badge {
            display: inline-block;
            background-color: #ef4444;
            color: #fff;
            font-size: 10px;
            padding: 2px 8px;
            border-radius: 10px;
            margin-top: 4px;
        }

        .nav-list {
            list-style: none;
            padding: 15px 0;
        }

        .nav-item {
            padding: 12px 20px;
            display: flex;
            align-items: center;
            gap: 12px;
            cursor: pointer;
            color: #94a3b8;
            transition: 0.2s;
            font-size: 14px;
        }

        .nav-item:hover, .nav-item.active {
            background-color: #2563eb;
            color: #fff;
        }

        .main-content {
            flex: 1;
            display: flex;
            flex-direction: column;
            min-height: 100vh;
            overflow-x: auto;
            overflow-y: auto;
        }

        .top-bar {
            background-color: #fff;
            padding: 15px 25px;
            display: flex;
            justify-content: space-between;
            align-items: center;
            border-bottom: 1px solid #e2e8f0;
            width: 100%;
        }

        .top-bar h2 {
            font-size: 18px;
            color: #1e293b;
        }

        .top-actions {
            display: flex;
            gap: 10px;
            align-items: center;
        }

        .admin-select-box {
            background: #f8fafc;
            border: 1px solid #cbd5e1;
            padding: 6px 12px;
            border-radius: 6px;
            font-size: 13px;
            display: none;
            align-items: center;
            gap: 8px;
        }

        .admin-select-box select {
            padding: 4px 8px;
            border-radius: 4px;
            border: 1px solid #cbd5e1;
            outline: none;
            font-weight: bold;
            color: #1e40af;
        }

        .btn-action {
            padding: 8px 14px;
            border-radius: 6px;
            border: 1px solid #cbd5e1;
            background: #fff;
            cursor: pointer;
            font-size: 13px;
            display: flex;
            align-items: center;
            gap: 6px;
        }

        .btn-action.btn-danger {
            background-color: #ef4444;
            color: white;
            border: none;
        }

        .tab-content {
            flex: 1;
            padding: 25px;
            display: none;
            width: 100%;
        }

        .tab-content.active {
            display: block;
        }

        .dashboard-grid {
            display: grid;
            grid-template-columns: repeat(auto-fit, minmax(400px, 1fr));
            gap: 20px;
            padding: 10px 0;
            width: 100%;
        }

        .card {
            background: #ffffff;
            border-radius: 12px;
            padding: 24px;
            box-shadow: 0 4px 12px rgba(0, 0, 0, 0.05);
            border: 1px solid #f1f5f9;
            margin-bottom: 20px;
            width: 100%;
        }

        .card h3 {
            font-size: 18px;
            font-weight: 700;
            color: #334155;
            margin-bottom: 20px;
        }

        .chart-container {
            position: relative;
            height: 350px;
            width: 100%;
        }

        .task-input-bar {
            background: #fff;
            padding: 15px;
            border-radius: 8px;
            box-shadow: 0 1px 3px rgba(0,0,0,0.1);
            margin-bottom: 20px;
            display: flex;
            gap: 12px;
            flex-wrap: wrap;
            align-items: center;
            width: 100%;
        }

        .task-input-bar input[type="text"],
        .task-input-bar select,
        .task-input-bar input[type="date"] {
            padding: 8px 12px;
            border: 1px solid #cbd5e1;
            border-radius: 6px;
            outline: none;
            font-size: 14px;
        }

        .task-input-bar input[type="text"] { flex: 2; min-width: 200px; }
        .task-input-bar select { flex: 1; min-width: 120px; }
        .task-input-bar input[type="date"] { flex: 1; min-width: 140px; cursor: pointer; }

        .table-container {
            background: #fff;
            border-radius: 8px;
            overflow-x: auto;
            box-shadow: 0 1px 3px rgba(0,0,0,0.1);
            width: 100%;
        }

        table {
            width: 100%;
            border-collapse: collapse;
            text-align: left;
            font-size: 14px;
        }

        th, td {
            padding: 12px 16px;
            border-bottom: 1px solid #e2e8f0;
        }

        th {
            background-color: #f8fafc;
            color: #475569;
        }

        .inline-date-picker {
            border: 1px solid #cbd5e1;
            border-radius: 4px;
            padding: 4px 6px;
            font-size: 13px;
            outline: none;
            cursor: pointer;
            background: #fff;
        }

        tr.row-overdue { background-color: #fef2f2 !important; }
        tr.row-overdue td { color: #991b1b; }

        .badge-overdue {
            background-color: #ef4444;
            color: #ffffff;
            font-size: 11px;
            padding: 3px 8px;
            border-radius: 4px;
            font-weight: bold;
            display: inline-flex;
            align-items: center;
            gap: 4px;
            margin-left: 6px;
        }

        .status-tag {
            padding: 4px 8px;
            border-radius: 4px;
            font-size: 12px;
            font-weight: 500;
        }

        .status-doing { background: #dbeafe; color: #1d4ed8; }
        .status-todo { background: #f3e8ff; color: #6b21a8; }
        .status-done { background: #dcfce7; color: #15803d; }

        .kanban-board {
            display: grid;
            grid-template-columns: repeat(3, 1fr);
            gap: 20px;
            width: 100%;
            min-width: 800px;
        }

        .kanban-col {
            background: #e2e8f0;
            border-radius: 8px;
            padding: 15px;
            min-height: 500px;
        }

        .kanban-col-header {
            font-weight: bold;
            margin-bottom: 15px;
            display: flex;
            justify-content: space-between;
            align-items: center;
        }

        .kanban-card {
            background: #fff;
            padding: 15px;
            border-radius: 6px;
            margin-bottom: 10px;
            box-shadow: 0 1px 2px rgba(0,0,0,0.1);
            cursor: grab;
            border-left: 4px solid transparent;
        }

        .kanban-card.card-overdue {
            background-color: #fef2f2;
            border-left: 4px solid #ef4444;
        }

        .kanban-card:active { cursor: grabbing; }
        .kanban-card .id { color: #2563eb; font-size: 12px; font-weight: bold; }
        .kanban-card .title { font-size: 14px; font-weight: 600; margin: 5px 0 10px; }
        .kanban-card .meta { font-size: 12px; color: #64748b; display: flex; justify-content: space-between; align-items: center; }

        .kpi-section-title {
            background: #1e40af;
            color: #ffffff;
            padding: 10px 16px;
            font-size: 15px;
            font-weight: bold;
            border-radius: 6px 6px 0 0;
            display: flex;
            justify-content: space-between;
            align-items: center;
        }

        .row-sub-header {
            background-color: #f1f5f9;
            font-weight: bold;
            color: #1e293b;
        }

        .kpi-table input[type="number"] {
            width: 75px;
            padding: 4px 6px;
            border: 1px solid #cbd5e1;
            border-radius: 4px;
            text-align: center;
            font-weight: 600;
        }

        .btn-sm {
            padding: 4px 8px;
            font-size: 11px;
            border-radius: 4px;
            border: none;
            cursor: pointer;
            margin-right: 2px;
        }
        .btn-edit { background-color: #f59e0b; color: white; }
        .btn-delete { background-color: #ef4444; color: white; }
        .btn-attach { background-color: #3b82f6; color: white; }
        .btn-download { background-color: #10b981; color: white; }
        .btn-view { background-color: #6366f1; color: white; }

        .file-tag {
            display: inline-flex;
            align-items: center;
            gap: 4px;
            background: #e2e8f0;
            padding: 2px 6px;
            border-radius: 4px;
            font-size: 11px;
            margin: 2px 0;
            color: #1e293b;
        }

        .file-tag a {
            color: #2563eb;
            text-decoration: none;
            font-weight: 600;
        }

        .file-tag a:hover {
            text-decoration: underline;
        }

        .kpi-target-bar {
            background: #e0f2fe;
            border: 1px solid #bae6fd;
            padding: 12px 20px;
            border-radius: 8px;
            margin-bottom: 20px;
            display: flex;
            align-items: center;
            gap: 15px;
            flex-wrap: wrap;
            width: 100%;
        }

        .kpi-target-bar select {
            padding: 6px 12px;
            border-radius: 6px;
            border: 1px solid #0284c7;
            font-weight: bold;
            color: #0369a1;
            outline: none;
        }

        .kpi-month-selector-bar {
            background: #ffffff;
            border: 1px solid #cbd5e1;
            padding: 12px 20px;
            border-radius: 8px;
            margin-bottom: 20px;
            display: flex;
            align-items: center;
            gap: 15px;
            box-shadow: 0 1px 3px rgba(0,0,0,0.05);
            width: 100%;
        }

        .kpi-month-selector-bar select {
            padding: 6px 12px;
            border-radius: 6px;
            border: 1px solid #2563eb;
            font-weight: bold;
            color: #1e40af;
            outline: none;
            background: #f0f9ff;
        }

        .kpi-badge-type {
            display: inline-block;
            padding: 4px 10px;
            border-radius: 20px;
            font-size: 12px;
            font-weight: bold;
            margin-left: 10px;
        }
        .type-leader { background-color: #fef3c7; color: #b45309; border: 1px solid #fde68a; }
        .type-staff { background-color: #e0e7ff; color: #3730a3; border: 1px solid #c7d2fe; }
        .type-cleaner { background-color: #dcfce7; color: #15803d; border: 1px solid #bbf7d0; }

        .kpi-total-box {
            display: flex;
            justify-content: flex-end;
            gap: 30px;
            margin-top: 20px;
            font-size: 16px;
            font-weight: bold;
            background: #f8fafc;
            padding: 15px 20px;
            border-radius: 8px;
            border: 1px solid #e2e8f0;
            width: 100%;
        }

        .sync-status {
            font-size: 12px;
            padding: 4px 8px;
            border-radius: 4px;
            background: #dcfce7;
            color: #15803d;
            display: flex;
            align-items: center;
            gap: 5px;
        }

        /* --- STYLES CHO BẢNG XẾP LOẠI KPI --- */
        .kpi-ranking-box {
            margin-top: 30px;
            background: #ffffff;
            border-radius: 8px;
            border: 1px solid #cbd5e1;
            overflow: hidden;
        }

        .kpi-ranking-title {
            background-color: #1e3a8a;
            color: #ffffff;
            font-weight: bold;
            text-align: center;
            padding: 10px;
            font-size: 15px;
            text-transform: uppercase;
        }

        .ranking-result-badge {
            display: inline-block;
            padding: 6px 16px;
            border-radius: 20px;
            font-weight: bold;
            font-size: 14px;
        }
        .rank-excel { background-color: #dcfce7; color: #15803d; border: 1px solid #86efac; }
        .rank-good { background-color: #dbeafe; color: #1e40af; border: 1px solid #93c5fd; }
        .rank-fair { background-color: #fef3c7; color: #b45309; border: 1px solid #fde68a; }
        .rank-poor { background-color: #fee2e2; color: #b91c1c; border: 1px solid #fca5a5; }

        /* --- STYLES CHO PHẦN BOX CHAT BỔ SUNG --- */
        .chat-layout {
            display: flex;
            gap: 20px;
            height: 550px;
        }

        .chat-user-list {
            width: 260px;
            background: #f8fafc;
            border: 1px solid #cbd5e1;
            border-radius: 8px;
            overflow-y: auto;
            display: flex;
            flex-direction: column;
        }

        .chat-user-item {
            padding: 12px 15px;
            border-bottom: 1px solid #e2e8f0;
            cursor: pointer;
            display: flex;
            align-items: center;
            justify-content: space-between;
            transition: 0.2s;
        }

        .chat-user-item:hover, .chat-user-item.active {
            background-color: #dbeafe;
            color: #1e40af;
            font-weight: bold;
        }

        .chat-unread-badge {
            background-color: #ef4444;
            color: white;
            font-size: 11px;
            font-weight: bold;
            min-width: 20px;
            height: 20px;
            border-radius: 50%;
            display: flex;
            align-items: center;
            justify-content: center;
            padding: 0 4px;
            line-height: 1;
        }

        .chat-container {
            flex: 1;
            display: flex;
            flex-direction: column;
            gap: 15px;
            background: #ffffff;
            border-radius: 8px;
            padding: 20px;
            border: 1px solid #cbd5e1;
            height: 100%;
        }

        .chat-input-area {
            background: #f8fafc;
            border: 1px solid #e2e8f0;
            padding: 12px;
            border-radius: 8px;
            display: flex;
            flex-direction: column;
            gap: 10px;
        }

        .chat-input-area textarea {
            width: 100%;
            height: 70px;
            padding: 10px;
            border: 1px solid #cbd5e1;
            border-radius: 6px;
            outline: none;
            resize: none;
            font-size: 14px;
        }

        .chat-file-preview {
            font-size: 12px;
            color: #2563eb;
            display: flex;
            align-items: center;
            gap: 6px;
        }

        .chat-history {
            flex: 1;
            display: flex;
            flex-direction: column;
            gap: 12px;
            overflow-y: auto;
            padding-right: 5px;
        }

        .chat-msg-item {
            max-width: 75%;
            padding: 10px 14px;
            border-radius: 8px;
            position: relative;
            word-wrap: break-word;
        }

        .chat-msg-sent {
            align-self: flex-end;
            background: #dbeafe;
            border: 1px solid #93c5fd;
        }

        .chat-msg-received {
            align-self: flex-start;
            background: #f1f5f9;
            border: 1px solid #cbd5e1;
        }

        .chat-msg-header {
            display: flex;
            justify-content: space-between;
            gap: 10px;
            font-size: 11px;
            color: #64748b;
            margin-bottom: 4px;
        }

        .chat-msg-sender {
            font-weight: bold;
            color: #1e40af;
        }

        .chat-msg-body {
            font-size: 14px;
            color: #1e293b;
            white-space: pre-wrap;
        }

        /* Modal Xem File Trực Tiếp */
        .modal-overlay {
            position: fixed;
            top: 0; left: 0; width: 100vw; height: 100vh;
            background: rgba(0,0,0,0.6);
            display: none;
            justify-content: center;
            align-items: center;
            z-index: 10000;
        }
        .modal-content {
            background: #fff;
            width: 85%;
            height: 88%;
            border-radius: 8px;
            display: flex;
            flex-direction: column;
            overflow: hidden;
            box-shadow: 0 5px 20px rgba(0,0,0,0.3);
        }
        .modal-header {
            padding: 12px 20px;
            background: #1e293b;
            color: white;
            display: flex;
            justify-content: space-between;
            align-items: center;
        }
        .modal-body {
            flex: 1;
            background: #f8fafc;
            position: relative;
        }
        .modal-body iframe {
            width: 100%;
            height: 100%;
            border: none;
        }

        /* --- STYLES BỔ SUNG CHO MỤC NHIỆM VỤ KPI --- */
        .kpi-task-input-area {
            width: 100%;
            border: 1px solid #cbd5e1;
            border-radius: 4px;
            padding: 8px;
            font-size: 13px;
            outline: none;
            resize: vertical;
            min-height: 60px;
            background: #fff;
        }
        .kpi-task-input-area:focus {
            border-color: #2563eb;
        }

        .admin-edit-input {
            width: 100%;
            padding: 4px 8px;
            border: 1px solid #3b82f6;
            border-radius: 4px;
            font-size: 13px;
        }
    </style>
</head>
<body>

    <!-- MODAL XEM FILE TRỰC TIẾP -->
    <div id="file-viewer-modal" class="modal-overlay">
        <div class="modal-content">
            <div class="modal-header">
                <span id="modal-file-title" style="font-weight: bold; font-size: 15px;"><i class="fa-solid fa-file-eye"></i> Xem trực tiếp tài liệu</span>
                <button onclick="closeFileViewer()" style="background: none; border: none; color: white; font-size: 20px; cursor: pointer;"><i class="fa-solid fa-xmark"></i></button>
            </div>
            <div class="modal-body" id="modal-file-body">
                <!-- iFrame hoặc hình ảnh preview sẽ được chèn động vào đây -->
            </div>
        </div>
    </div>

    <!-- 1. MÀN HÌNH ĐĂNG NHẬP -->
    <div id="login-screen">
        <div class="login-card">
            <div class="icon"><i class="fa-solid fa-flask"></i></div>
            <h2>KHOA HÓA LÝ</h2>
            <p id="form-sub-title">Đăng nhập hệ thống quản trị công việc (Realtime)</p>
            
            <div style="display: flex; gap: 10px; margin-bottom: 20px; border-bottom: 2px solid #e2e8f0; padding-bottom: 10px;">
                <button type="button" id="tab-login-btn" onclick="toggleAuthTab('login')" style="flex: 1; padding: 8px; border: none; background: none; font-weight: bold; color: #2563eb; border-bottom: 2px solid #2563eb; cursor: pointer;">ĐĂNG NHẬP</button>
                <button type="button" id="tab-register-btn" onclick="toggleAuthTab('register')" style="flex: 1; padding: 8px; border: none; background: none; font-weight: bold; color: #64748b; cursor: pointer;">ĐĂNG KÝ</button>
            </div>

            <form id="login-form" onsubmit="handleLogin(event)">
                <div class="form-group">
                    <label>Tên đăng nhập</label>
                    <input type="text" id="login-username" placeholder="Nhập tên đăng nhập..." required>
                </div>
                <div class="form-group">
                    <label>Mật khẩu</label>
                    <input type="password" id="login-password" placeholder="Nhập mật khẩu..." required>
                </div>
                <button type="submit" class="btn-login">ĐĂNG NHẬP</button>
            </form>

            <form id="register-form" onsubmit="handleRegister(event)" style="display: none;">
                <div class="form-group">
                    <label>Gmail</label>
                    <input type="email" id="reg-email" placeholder="example@gmail.com" required>
                </div>
                <div class="form-group">
                    <label>Tên đăng nhập</label>
                    <input type="text" id="reg-username" placeholder="Tạo tên đăng nhập..." required>
                </div>
                <div class="form-group">
                    <label>Mật khẩu</label>
                    <input type="password" id="reg-password" placeholder="Nhập mật khẩu..." required>
                </div>
                <div class="form-group">
                    <label>Xác nhận mật khẩu</label>
                    <input type="password" id="reg-confirm-password" placeholder="Nhập lại mật khẩu..." required>
                </div>
                <div class="form-group">
                    <label>Loại Bảng KPI áp dụng</label>
                    <select id="reg-kpi-type">
                        <option value="staff">Bảng KPI Dành cho Cán bộ / Nhân viên</option>
                        <option value="leader">Bảng KPI Dành cho Lãnh đạo</option>
                        <option value="cleaner">Bảng KPI Dành cho Lao Công</option>
                    </select>
                </div>
                <button type="submit" class="btn-login" style="background-color: #10b981;">ĐĂNG KÝ TÀI KHOẢN</button>
            </form>
        </div>
    </div>

    <!-- 2. MÀN HÌNH CHÍNH WEB APP -->
    <div id="app-screen" style="display: none;">
        <!-- Sidebar -->
        <div class="sidebar">
            <div class="sidebar-brand">
                <i class="fa-solid fa-shapes"></i> WORKSPACE
            </div>
            <div class="user-profile">
                <div class="name" id="user-display-name">Cán bộ</div>
                <span class="badge" id="user-role-badge">User</span>
            </div>
            <ul class="nav-list">
                <li class="nav-item active" onclick="switchTab('tong-quan', this)">
                    <i class="fa-solid fa-chart-pie"></i> Tổng quan
                </li>
                <li class="nav-item" onclick="switchTab('danh-sach', this)">
                    <i class="fa-solid fa-list-check"></i> Danh sách
                </li>
                <li class="nav-item" onclick="switchTab('kanban', this)">
                    <i class="fa-solid fa-table-columns"></i> Tiến độ công việc
                </li>
                <li class="nav-item" onclick="switchTab('lich-cong-tac', this)">
                    <i class="fa-solid fa-calendar-days"></i> Lịch công tác
                </li>
                <li class="nav-item" onclick="switchTab('nhiem-vu-kpi', this)">
                    <i class="fa-solid fa-list-no-check"></i> Nhiệm vụ KPI
                </li>
                <li class="nav-item" onclick="switchTab('kpi', this)">
                    <i class="fa-solid fa-award"></i> Đánh giá KPI
                </li>
                <li class="nav-item" id="nav-admin-users" style="display: none;" onclick="switchTab('admin-users', this)">
                    <i class="fa-solid fa-users-gear"></i> Quản lý Users
                </li>
                <li class="nav-item" onclick="switchTab('box-chat', this)">
                    <i class="fa-solid fa-comments"></i> Box chat
                </li>
            </ul>
        </div>

        <!-- Main Content -->
        <div class="main-content">
            <!-- Top Bar -->
            <div class="top-bar">
                <h2 id="page-title">Dashboard Thống Kê</h2>
                <div class="top-actions">
                    <div class="sync-status"><i class="fa-solid fa-arrows-rotate fa-spin"></i> Đồng bộ Realtime</div>
                    <div class="admin-select-box" id="admin-user-selector">
                        <span><i class="fa-solid fa-user-pen"></i> Xem data công việc của:</span>
                        <select id="select-target-user" onchange="changeTargetUser(this.value)"></select>
                    </div>
                    <button class="btn-action btn-danger" onclick="logout()"><i class="fa-solid fa-power-off"></i> Đăng xuất</button>
                </div>
            </div>

            <!-- Tab 1: Tổng quan -->
            <div id="tab-tong-quan" class="tab-content active">
                <div id="admin-master-overview-banner" style="display:none; background: #eff6ff; border: 1px solid #bfdbfe; padding: 12px 20px; border-radius: 8px; margin-bottom: 20px; font-weight: bold; color: #1e40af;">
                    <i class="fa-solid fa-circle-info"></i> Chế độ Admin: Đang tổng hợp dữ liệu toàn bộ tài khoản thường trong hệ thống.
                </div>
                <div class="dashboard-grid">
                    <div class="card">
                        <h3>Tỷ lệ Trạng thái</h3>
                        <div class="chart-container">
                            <canvas id="statusChart"></canvas>
                        </div>
                    </div>
                    <div class="card">
                        <h3>Mức độ Ưu tiên</h3>
                        <div class="chart-container">
                            <canvas id="priorityChart"></canvas>
                        </div>
                    </div>
                </div>
            </div>

            <!-- Tab 2: Danh sách -->
            <div id="tab-danh-sach" class="tab-content">
                <h3 style="font-size: 16px; color: #334155; margin-bottom: 12px;">Quản lý Công Việc Độc Lập</h3>
                
                <div class="task-input-bar">
                    <input type="text" id="newTaskName" placeholder="Nhập tên công việc mới..." />
                    <select id="newTaskPriority">
                        <option value="Bình thường">Bình thường</option>
                        <option value="Cao">Cao</option>
                        <option value="Thấp">Thấp</option>
                    </select>
                    <input type="date" id="newTaskDueDate" />
                    <button class="btn-login" style="width: auto; padding: 8px 18px;" onclick="addNewTask()"><i class="fa-solid fa-plus"></i> Thêm công việc</button>
                </div>

                <div class="table-container">
                    <table>
                        <thead>
                            <tr>
                                <th>Mã</th>
                                <th>Tên công việc</th>
                                <th>Người nhận</th>
                                <th>Mức ưu tiên</th>
                                <th>Trạng thái</th>
                                <th>Hạn chót</th>
                                <th>File đính kèm</th>
                                <th>Thao tác</th>
                            </tr>
                        </thead>
                        <tbody id="task-table-body"></tbody>
                    </table>
                </div>
            </div>

            <!-- Tab 3: Kanban / Tiến độ công việc -->
            <div id="tab-kanban" class="tab-content">
                <div class="kanban-board">
                    <div class="kanban-col" id="col-todo" ondragover="allowDrop(event)" ondrop="drop(event, 'Chưa làm')">
                        <div class="kanban-col-header">🌙 Chưa làm <span id="count-todo">0</span></div>
                        <div class="kanban-cards" id="cards-todo"></div>
                    </div>
                    <div class="kanban-col" id="col-doing" ondragover="allowDrop(event)" ondrop="drop(event, 'Đang làm')">
                        <div class="kanban-col-header">⌛ Đang làm <span id="count-doing">0</span></div>
                        <div class="kanban-cards" id="cards-doing"></div>
                    </div>
                    <div class="kanban-col" id="col-done" ondragover="allowDrop(event)" ondrop="drop(event, 'Hoàn thành')">
                        <div class="kanban-col-header">✔️ Hoàn thành <span id="count-done">0</span></div>
                        <div class="kanban-cards" id="cards-done"></div>
                    </div>
                </div>
            </div>

            <!-- Tab 4: Lịch công tác -->
            <div id="tab-lich-cong-tac" class="tab-content">
                <div class="card">
                    <div style="display: flex; justify-content: space-between; align-items: center; margin-bottom: 15px;">
                        <div>
                            <h3 style="margin-bottom: 5px;"><i class="fa-solid fa-file-invoice" style="color: #2563eb;"></i> LỊCH CÔNG TÁC VÀ TÀI LIỆU KHOA HÓA LÝ</h3>
                            <p style="color: #64748b; font-size: 13px;">Tất cả tài khoản cán bộ có thể xem trực tiếp và tải về các file Lịch công tác (Word, Excel, PDF...) do Admin đăng tải.</p>
                        </div>
                        <div id="admin-schedule-upload-container" style="display: none;">
                            <input type="file" id="schedule-file-input" style="display: none;" onchange="uploadScheduleFile(this)" accept=".doc,.docx,.xls,.xlsx,.pdf,.ppt,.pptx,.txt" />
                            <button class="btn-login" style="width: auto; background-color: #10b981; padding: 10px 18px;" onclick="document.getElementById('schedule-file-input').click()">
                                <i class="fa-solid fa-cloud-arrow-up"></i> Đăng file Lịch công tác mới
                            </button>
                        </div>
                    </div>

                    <div class="table-container">
                        <table>
                            <thead>
                                <tr>
                                    <th style="width: 60px; text-align: center;">STT</th>
                                    <th>Tên Tệp / File Lịch Công Tác</th>
                                    <th style="width: 180px;">Ngày Đăng Tải</th>
                                    <th style="width: 200px; text-align: center;">Tải Về & Xem</th>
                                    <th id="th-schedule-action" style="width: 100px; text-align: center; display: none;">Thao tác</th>
                                </tr>
                            </thead>
                            <tbody id="schedule-files-table-body">
                                <tr>
                                    <td colspan="5" style="text-align: center; color: #94a3b8; padding: 20px;">Đang tải danh sách file...</td>
                                </tr>
                            </tbody>
                        </table>
                    </div>
                </div>
            </div>

            <!-- TAB MỚI: NHIỆM VỤ KPI -->
            <div id="tab-nhiem-vu-kpi" class="tab-content">
                <div class="kpi-month-selector-bar">
                    <i class="fa-regular fa-calendar-check" style="font-size: 22px; color: #2563eb;"></i>
                    <span style="font-weight: bold; color: #1e293b;">KỲ ĐÁNH GIÁ KPI:</span>
                    <select id="select-kpi-task-month" onchange="changeKPITaskMonth(this.value)"></select>

                    <span style="font-size: 13px; color: #64748b; margin-left: auto;">
                        <i class="fa-solid fa-clock-rotate-left"></i> Dữ liệu nhiệm vụ tách biệt theo tháng
                    </span>
                </div>

                <div class="kpi-target-bar" id="kpi-task-admin-target-bar" style="display: none;">
                    <i class="fa-solid fa-user-check" style="font-size: 20px; color: #0284c7;"></i>
                    <span style="font-weight: 600; color: #0369a1;">Chọn tài khoản để xem/sửa Nhiệm vụ KPI:</span>
                    <select id="select-kpi-task-target-user" onchange="changeKPITaskTargetUser(this.value)"></select>
                </div>

                <div class="card">
                    <div style="display: flex; justify-content: space-between; align-items: center; margin-bottom: 20px;">
                        <h3 style="font-size: 18px; margin: 0; display: flex; align-items: center;">
                            NHIỆM VỤ KPI CỦA: <span id="kpi-task-target-name-display" style="color: #2563eb; text-transform: uppercase; margin-left: 8px;"></span>
                        </h3>
                        <div>
                            <button id="btn-add-kpi-task-row" class="btn-login" style="width: auto; background-color: #10b981; padding: 8px 16px; display: none;" onclick="addKPITaskRow()">
                                <i class="fa-solid fa-plus"></i> Thêm Hàng Nhiệm Vụ
                            </button>
                        </div>
                    </div>

                    <div class="table-container">
                        <table>
                            <thead>
                                <tr>
                                    <th style="width: 60px; text-align: center;">STT</th>
                                    <th style="width: 45%;">Nội dung nhiệm vụ</th>
                                    <th style="width: 45%;">Minh chứng tối thiểu</th>
                                    <th id="th-kpi-task-action" style="width: 60px; text-align: center; display: none;">Xóa</th>
                                </tr>
                            </thead>
                            <tbody id="kpi-tasks-table-body">
                                <tr>
                                    <td colspan="4" style="text-align: center; color: #94a3b8; padding: 20px;">Đang tải dữ liệu nhiệm vụ KPI...</td>
                                </tr>
                            </tbody>
                        </table>
                    </div>

                    <div style="text-align: center; margin-top: 25px; display: flex; justify-content: center; gap: 15px;">
                        <button class="btn-login" style="width: auto; padding: 10px 24px; background-color: #2563eb;" onclick="saveKPITasksData()">
                            <i class="fa-solid fa-floppy-disk"></i> LƯU DỮ LIỆU NHIỆM VỤ KPI
                        </button>
                        <button class="btn-login" style="width: auto; background-color: #0284c7; padding: 10px 24px;" onclick="exportKPITaskWord()">
                            <i class="fa-solid fa-file-word"></i> XUẤT FILE WORD NHIỆM VỤ KPI
                        </button>
                    </div>
                </div>
            </div>

            <!-- Tab 5: Đánh giá KPI -->
            <div id="tab-kpi" class="tab-content">
                <div class="kpi-month-selector-bar">
                    <i class="fa-regular fa-calendar-check" style="font-size: 22px; color: #2563eb;"></i>
                    <span style="font-weight: bold; color: #1e293b;">KỲ ĐÁNH GIÁ KPI:</span>
                    <select id="select-kpi-month" onchange="changeKPIMonth(this.value)"></select>

                    <span style="font-size: 13px; color: #64748b; margin-left: auto;">
                        <i class="fa-solid fa-clock-rotate-left"></i> Dữ liệu các tháng trước được lưu trữ tự động
                    </span>
                </div>

                <div class="kpi-target-bar" id="kpi-admin-target-bar" style="display: none;">
                    <i class="fa-solid fa-user-check" style="font-size: 20px; color: #0284c7;"></i>
                    <span style="font-weight: 600; color: #0369a1;">Chọn tài khoản để chấm KPI:</span>
                    <select id="select-kpi-target-user" onchange="changeKPITargetUser(this.value)"></select>

                    <div style="margin-left: 20px; display: flex; align-items: center; gap: 8px;">
                        <span style="font-weight: 600; color: #0369a1;"><i class="fa-solid fa-arrow-right-arrow-left"></i> Chuyển Bảng KPI:</span>
                        <select id="select-kpi-type-change" onchange="adminChangeUserKPIType(this.value)">
                            <option value="staff">Bảng KPI Dành cho Cán bộ / Nhân viên</option>
                            <option value="leader">Bảng KPI Dành cho Lãnh đạo</option>
                            <option value="cleaner">Bảng KPI Dành cho Lao Công</option>
                        </select>
                    </div>
                </div>

                <div class="card">
                    <div style="display: flex; justify-content: space-between; align-items: center; margin-bottom: 20px;">
                        <h3 style="font-size: 18px; margin: 0; display: flex; align-items: center;">
                            PHIẾU ĐÁNH GIÁ KPI CỦA: <span id="kpi-target-name-display" style="color: #2563eb; text-transform: uppercase; margin-left: 8px;"></span>
                            <span id="kpi-type-badge-display" class="kpi-badge-type"></span>
                        </h3>
                        <button id="btn-add-main-section" class="btn-login" style="width: auto; background-color: #8b5cf6; padding: 8px 16px; display: none;" onclick="addMainSection()">
                            <i class="fa-solid fa-folder-plus"></i> Thêm Mục Lớn
                        </button>
                    </div>

                    <div id="kpi-sections-wrapper"></div>
                    
                    <div class="kpi-total-box">
                        <div>
                            TỔNG ĐIỂM TỰ CHẤM: 
                            <span id="kpi-total-self" style="color: #2563eb;">0</span> / 
                            <span id="kpi-total-max" style="color: #64748b;">0</span>
                        </div>
                        <div style="border-left: 2px solid #cbd5e1; padding-left: 20px;">
                            TỔNG ĐIỂM ĐÁNH GIÁ: 
                            <span id="kpi-total-admin" style="color: #059669;">0</span>
                        </div>
                    </div>

                    <!-- BẢNG XẾP LOẠI RIÊNG ĐỘC LẬP BÊN DƯỚI -->
                    <div class="kpi-ranking-box">
                        <div class="kpi-ranking-title"><i class="fa-solid fa-ranking-star"></i> BẢNG XẾP LOẠI</div>
                        <div class="table-container">
                            <table>
                                <thead>
                                    <tr>
                                        <th style="width: 60px; text-align: center;">STT</th>
                                        <th style="width: 40%;">Mức xếp loại</th>
                                        <th style="width: 40%;">Mức điểm</th>
                                        <th style="width: 20%; text-align: center;">Đạt được</th>
                                    </tr>
                                </thead>
                                <tbody>
                                    <tr>
                                        <td style="text-align: center;">1</td>
                                        <td><strong>Hoàn thành xuất sắc nhiệm vụ</strong></td>
                                        <td>Từ 90 điểm trở lên</td>
                                        <td style="text-align: center;" id="rank-check-1"></td>
                                    </tr>
                                    <tr>
                                        <td style="text-align: center;">2</td>
                                        <td><strong>Hoàn thành tốt nhiệm vụ</strong></td>
                                        <td>Từ 75 đến dưới 90 điểm</td>
                                        <td style="text-align: center;" id="rank-check-2"></td>
                                    </tr>
                                    <tr>
                                        <td style="text-align: center;">3</td>
                                        <td><strong>Hoàn thành nhiệm vụ</strong></td>
                                        <td>Từ 50 đến dưới 75 điểm</td>
                                        <td style="text-align: center;" id="rank-check-3"></td>
                                    </tr>
                                    <tr>
                                        <td style="text-align: center;">4</td>
                                        <td><strong>Không hoàn thành nhiệm vụ</strong></td>
                                        <td>Dưới 50 điểm</td>
                                        <td style="text-align: center;" id="rank-check-4"></td>
                                    </tr>
                                </tbody>
                            </table>
                        </div>
                        <div style="padding: 15px; background: #f8fafc; border-top: 1px solid #cbd5e1; display: flex; align-items: center; justify-content: space-between;">
                            <span style="font-weight: bold; color: #1e293b; font-size: 15px;"><i class="fa-solid fa-award" style="color: #f59e0b;"></i> KẾT QUẢ XẾP LOẠI TỰ ĐỘNG:</span>
                            <span id="kpi-ranking-result-display" class="ranking-result-badge rank-good">Chưa xếp loại</span>
                        </div>
                    </div>
                    
                    <div style="margin-top: 15px; text-align: right; font-size: 13px; color: #475569;">
                        <i class="fa-regular fa-clock" style="color: #0284c7;"></i> Lần cuối lưu điểm (<span id="selected-month-text" style="font-weight: bold; color: #0284c7;"></span>): <span id="kpi-last-time-saved" style="font-weight: bold; color: #1e293b;">Chưa có dữ liệu</span>
                    </div>

                    <div style="text-align: center; margin-top: 20px; display: flex; justify-content: center; gap: 15px; flex-wrap: wrap;">
                        <button id="btn-save-kpi-admin" class="btn-login" style="width: auto; padding: 10px 24px; background-color: #2563eb; display: none;" onclick="saveKPIRatingByAdmin()">
                            <i class="fa-solid fa-floppy-disk"></i> LƯU KẾT QUẢ ĐÁNH GIÁ THÁNG NÀY (ADMIN)
                        </button>
                        <button id="btn-save-kpi-user" class="btn-login" style="width: auto; padding: 10px 24px; background-color: #10b981; display: none;" onclick="saveKPIRatingByUser()">
                            <i class="fa-solid fa-user-check"></i> LƯU KẾT QUẢ TỰ CHẤM THÁNG NÀY
                        </button>
                        <button class="btn-login" onclick="exportKPIWord()" style="width: auto; background-color: #0284c7; padding: 10px 24px;">
                            <i class="fa-solid fa-file-word"></i> XUẤT FILE WORD
                        </button>
                    </div>
                </div>
            </div>

            <!-- Tab 6: Quản lý Users -->
            <div id="tab-admin-users" class="tab-content">
                <div class="card">
                    <h3>Danh Sách Tài Khoản Trong Hệ Thống (Online Sync)</h3>
                    <p style="color: #64748b; font-size: 13px; margin-bottom: 15px;">* Bạn có thể phân loại bảng đánh giá KPI cho từng tài khoản (Bao gồm cả Admin) tại cột "Loại Bảng KPI".</p>
                    <div class="table-container">
                        <table>
                            <thead>
                                <tr>
                                    <th>STT</th>
                                    <th>Tên đăng nhập</th>
                                    <th>Email</th>
                                    <th>Loại Bảng KPI</th>
                                    <th>Thao tác</th>
                                </tr>
                            </thead>
                            <tbody id="user-management-body"></tbody>
                        </table>
                    </div>
                </div>
            </div>

            <!-- Tab 7: Box Chat (HỆ THỐNG CHAT 2 CHIỀU ĐỘC LẬP) -->
            <div id="tab-box-chat" class="tab-content">
                <div class="card" style="margin-bottom: 0;">
                    <h3><i class="fa-solid fa-comments" style="color: #2563eb;"></i> BOX CHAT TRAO ĐỔI VỚI ADMIN</h3>
                    <p style="color: #64748b; font-size: 13px; margin-bottom: 15px;" id="chat-sub-title">Tất cả tin nhắn và file đính kèm được trao đổi trực tiếp và bảo mật giữa bạn và Admin.</p>
                    
                    <div class="chat-layout">
                        <!-- Cột bên trái: Danh sách User (Chỉ hiển thị với ADMIN) -->
                        <div class="chat-user-list" id="chat-user-list-sidebar" style="display: none;">
                            <div style="padding: 10px 15px; font-weight: bold; background: #e2e8f0; border-bottom: 1px solid #cbd5e1; font-size: 13px;">
                                <i class="fa-solid fa-users"></i> CUỘC TRÒ CHUYỆN
                            </div>
                            <div id="chat-user-items-wrapper"></div>
                        </div>

                        <!-- Cột bên phải: Giao diện Chat chính -->
                        <div class="chat-container">
                            <div style="font-weight: bold; color: #1e40af; border-bottom: 1px solid #e2e8f0; padding-bottom: 8px;" id="chat-header-title">
                                Trò chuyện với Admin
                            </div>

                            <!-- Khung lịch sử tin nhắn -->
                            <div class="chat-history" id="chat-history-list">
                                <div style="text-align: center; color: #94a3b8; padding: 20px;">Đang tải tin nhắn...</div>
                            </div>

                            <!-- Khung nhập tin nhắn & gửi tệp đính kèm -->
                            <div class="chat-input-area">
                                <textarea id="chat-message-input" placeholder="Nhập nội dung tin nhắn..."></textarea>
                                
                                <div style="display: flex; justify-content: space-between; align-items: center; flex-wrap: wrap; gap: 10px;">
                                    <div style="display: flex; align-items: center; gap: 10px;">
                                        <input type="file" id="chat-file-input" style="display: none;" onchange="handleChatFileSelect(this)" />
                                        <button class="btn-action" onclick="document.getElementById('chat-file-input').click()">
                                            <i class="fa-solid fa-paperclip"></i> Đính kèm tệp
                                        </button>
                                        <span id="chat-file-name-preview" class="chat-file-preview"></span>
                                    </div>
                                    <button class="btn-login" style="width: auto; padding: 8px 20px;" onclick="sendChatMessage()">
                                        <i class="fa-solid fa-paper-plane"></i> Gửi tin nhắn
                                    </button>
                                </div>
                            </div>
                        </div>
                    </div>
                </div>
            </div>

        </div>
    </div>

    <script>
        // CẤU HÌNH FIREBASE
        const firebaseConfig = {
            apiKey: "AiZaSyDvMPWqdqgTJ5SzMrhhV56EOpv95yow4VA",
            authDomain: "kien02102005-381b4.firebaseapp.com",
            projectId: "kien02102005-381b4",
            storageBucket: "kien02102005-381b4.firebasestorage.app",
            messagingSenderId: "554013353586",
            appId: "1:554013353586:web:8a64d8ba969104053404eb",
            measurementId: "G-TJE25HB0PQ",
            databaseURL: "https://kien02102005-381b4-default-rtdb.firebaseio.com"
        };

        firebase.initializeApp(firebaseConfig);
        const db = firebase.database();

        const ADMIN_USERNAME = 'linhnguyenxuan';
        const ADMIN_PASSWORD = '051214';

        let currentUser = '';          
        let targetUser = '';           
        let kpiTargetUser = '';        
        let selectedKpiMonth = ''; 
        let selectedKPITaskMonth = '';
        let kpiTaskTargetUser = '';
        let kpiTasksList = [];
        let isAdmin = false;
        let lastKpiTimestamp = 'Chưa có dữ liệu';
        let isKpiSavedForCurrentMonth = false;

        let selectedChatFile = null;
        let activeChatTargetUser = ''; 

        let registeredUsers = [];
        let tasks = [];
        let scheduleFiles = [];
        let chatMessagesMap = {};
        let allUsersTasksMap = {}; 
        let kpiDataList = [];
        let sectionMaxScores = { A: 50, B: 30, C: 20 };
        let sectionTitles = {
            A: "QUẢN LÝ & CHUYÊN MÔN",
            B: "Ý THỨC CHẤP HÀNH & KỶ LUẬT",
            C: "NĂNG LỰC ĐỔI MỚI & NHIỆM VỤ ĐỘT XUẤT"
        };

        let masterKPITemplates = {
            staff: { 
                sectionMaxScores: { A: 50, B: 30, C: 20 }, 
                sectionTitles: { A: "QUẢN LÝ & CHUYÊN MÔN", B: "Ý THỨC CHẤP HÀNH & KỶ LUẬT", C: "NĂNG LỰC ĐỔI MỚI & NHIỆM VỤ ĐỘT XUẤT" },
                kpiDataList: [] 
            },
            leader: { 
                sectionMaxScores: { A: 50, B: 30, C: 20 }, 
                sectionTitles: { A: "QUẢN LÝ & CHUYÊN MÔN", B: "Ý THỨC CHẤP HÀNH & KỶ LUẬT", C: "NĂNG LỰC ĐỔI MỚI & NHIỆM VỤ ĐỘT XUẤT" },
                kpiDataList: [] 
            },
            cleaner: { 
                sectionMaxScores: { A: 70, B: 30 }, 
                sectionTitles: { A: "CÔNG TÁC CHUYÊN MÔN & VỆ SINH", B: "Ý THỨC CHẤP HÀNH & KỶ LUẬT" },
                kpiDataList: [] 
            }
        };

        let statusChartInstance = null;
        let priorityChartInstance = null;

        /* --- HÀM XEM FILE TRỰC TIẾP --- */
        function viewFileOnline(fileName, base64Data) {
            const modal = document.getElementById('file-viewer-modal');
            const titleEl = document.getElementById('modal-file-title');
            const bodyEl = document.getElementById('modal-file-body');
            
            titleEl.innerHTML = `<i class="fa-solid fa-file-lines"></i> Đang xem file: <strong>${fileName}</strong>`;
            bodyEl.innerHTML = '';

            const ext = fileName.split('.').pop().toLowerCase();

            if (['png', 'jpg', 'jpeg', 'gif', 'svg', 'webp'].includes(ext)) {
                bodyEl.innerHTML = `<div style="display:flex; justify-content:center; align-items:center; height:100%;"><img src="${base64Data}" style="max-width:100%; max-height:100%; object-fit:contain;" /></div>`;
            } else if (ext === 'pdf') {
                bodyEl.innerHTML = `<iframe src="${base64Data}"></iframe>`;
            } else if (ext === 'txt') {
                fetch(base64Data)
                    .then(res => res.text())
                    .then(text => {
                        bodyEl.innerHTML = `<pre style="padding:20px; white-space:pre-wrap; font-family:monospace; height:100%; overflow:auto;">${text}</pre>`;
                    });
            } else if (['doc', 'docx', 'xls', 'xlsx', 'ppt', 'pptx'].includes(ext)) {
                try {
                    const arr = base64Data.split(',');
                    const mime = arr[0].match(/:(.*?);/)[1];
                    const bstr = atob(arr[1]);
                    let n = bstr.length;
                    const u8arr = new Uint8Array(n);
                    while (n--) {
                        u8arr[n] = bstr.charCodeAt(n);
                    }
                    const blob = new Blob([u8arr], { type: mime });
                    const blobUrl = URL.createObjectURL(blob);
                    
                    bodyEl.innerHTML = `
                        <div style="padding: 30px; text-align: center; font-family: sans-serif;">
                            <i class="fa-solid fa-file-word" style="font-size: 60px; color: #2563eb; margin-bottom: 20px;"></i>
                            <h3 style="margin-bottom: 10px;">Tài liệu Office (${ext.toUpperCase()})</h3>
                            <p style="color: #64748b; margin-bottom: 20px;">Trình duyệt sẵn sàng mở file bản xem trước trong thẻ nội bộ mới.</p>
                            <a href="${blobUrl}" target="_blank" class="btn-login" style="display: inline-block; width: auto; padding: 10px 20px; text-decoration: none;">
                                <i class="fa-solid fa-up-right-from-square"></i> Mở khung xem file chi tiết
                            </a>
                        </div>
                    `;
                } catch(e) {
                    bodyEl.innerHTML = `<iframe src="${base64Data}"></iframe>`;
                }
            } else {
                bodyEl.innerHTML = `<iframe src="${base64Data}"></iframe>`;
            }

            modal.style.display = 'flex';
        }

        function closeFileViewer() {
            document.getElementById('file-viewer-modal').style.display = 'none';
            document.getElementById('modal-file-body').innerHTML = '';
        }

        listenRealtimeUsers();
        listenRealtimeTemplates();
        listenAllUsersTasks(); 
        listenRealtimeSchedules();
        listenRealtimeChat();

        function isValidNumber(val) {
            if (val === null || val === undefined) return false;
            let str = String(val).replace(',', '.').trim();
            if (str === '') return false;
            let num = Number(str);
            return !isNaN(num) && isFinite(num);
        }

        function parseFloatStrict(val) {
            if (!isValidNumber(val)) return 0;
            return parseFloat(String(val).replace(',', '.').trim());
        }

        function convertToDisplayDate(isoDateStr) {
            if (!isoDateStr) return '';
            if (isoDateStr.includes('/')) return isoDateStr;
            const parts = isoDateStr.split('-');
            if (parts.length === 3) return `${parts[2]}/${parts[1]}/${parts[0]}`;
            return isoDateStr;
        }

        function convertToISODate(displayDateStr) {
            if (!displayDateStr) return '';
            if (displayDateStr.includes('-')) return displayDateStr;
            const parts = displayDateStr.split('/');
            if (parts.length === 3) return `${parts[2]}-${parts[1].padStart(2, '0')}-${parts[0].padStart(2, '0')}`;
            return displayDateStr;
        }

        const defaultStaffKPIStructure = [
            {
                id: 'sub_a1', section: 'A', code: 'I', title: 'Phẩm chất chính trị, phẩm chất đạo đức, văn hóa thực thi công vụ, nhiệm vụ và ý thức kỷ luật, kỷ cương trong thực thi công vụ, nhiệm vụ', maxScore: 10,
                items: [
                    { id: 'item_a1_1', title: 'Phẩm chất chính trị, phẩm chất đạo đức, văn hóa thực thi công vụ, nhiệm vụ', criteria: '', maxScore: 5, selfScore: 5, adminScore: 5 },
                    { id: 'item_a1_2', title: 'Ý thức kỷ luật, kỷ cương trong thực thi công vụ, nhiệm vụ', criteria: '', maxScore: 5, selfScore: 5, adminScore: 5 }
                ]
            },
            {
                id: 'sub_a2', section: 'A', code: 'II', title: 'Năng lực chuyên môn, nghiệp vụ theo yêu cầu của vị trí việc làm; khả năng đáp ứng yêu cầu thực thi nhiệm vụ được giao; tinh thần trách nhiệm trong thực thi công vụ, nhiệm vụ; thái độ phục vụ người dân, doanh nghiệp và khả năng phối hợp với đồng nghiệp', maxScore: 10,
                items: [
                    { id: 'item_a2_1', title: 'Năng lực chuyên môn, nghiệp vụ theo yêu cầu của vị trí việc làm', criteria: '', maxScore: 2.5, selfScore: 2.5, adminScore: 2.5 },
                    { id: 'item_a2_2', title: 'Khả năng đáp ứng yêu cầu thực thi nhiệm vụ được giao thường xuyên, đột xuất', criteria: '', maxScore: 2.5, selfScore: 2.5, adminScore: 2.5 },
                    { id: 'item_a2_3', title: 'Tinh thần trách nhiệm trong thực thi công vụ, nhiệm vụ', criteria: '', maxScore: 2.5, selfScore: 2.5, adminScore: 2.5 },
                    { id: 'item_a2_4', title: 'Thái độ phục vụ người dân, doanh nghiệp và khả năng phối hợp với đồng nghiệp', criteria: '', maxScore: 2.5, selfScore: 2.5, adminScore: 2.5 }
                ]
            }
        ];

        const defaultLeaderKPIStructure = [
            {
                id: 'sub_a1', section: 'A', code: 'I', title: 'Năng lực Lãnh đạo & Quản lý điều hành (Lãnh đạo)', maxScore: 30,
                items: [
                    { id: 'item_a1_1', title: 'Xây dựng kế hoạch và chỉ đạo thực hiện nhiệm vụ khoa', criteria: '100% chỉ tiêu năm đạt tiến độ', maxScore: 15, selfScore: 15, adminScore: 15 },
                    { id: 'item_a1_2', title: 'Quản lý nhân sự và phát triển đội ngũ', criteria: 'Không có cán bộ vi phạm kỷ luật', maxScore: 15, selfScore: 14, adminScore: 15 }
                ]
            }
        ];

        const defaultCleanerKPIStructure = [
            {
                id: 'sub_cleaner_a1', section: 'A', code: 'I', title: 'Công tác Vệ sinh & Môi trường làm việc', maxScore: 40,
                items: [
                    { id: 'item_cleaner_a1_1', title: 'Dọn dẹp vệ sinh khu vực được phân công (Phòng làm việc, hành lang, nhà vệ sinh)', criteria: 'Sạch sẽ, gọn gàng, đúng lịch trình', maxScore: 20, selfScore: 20, adminScore: 20 },
                    { id: 'item_cleaner_a1_2', title: 'Thu gom và phân loại rác thải đúng quy định', criteria: 'Không tồn đọng rác thải qua ngày', maxScore: 20, selfScore: 20, adminScore: 20 }
                ]
            },
            {
                id: 'sub_cleaner_a2', section: 'A', code: 'II', title: 'Bảo quản Vật tư & Thiết bị Vệ sinh', maxScore: 30,
                items: [
                    { id: 'item_cleaner_a2_1', title: 'Quản lý và sử dụng tiết kiệm dung dịch, hóa chất, dụng cụ vệ sinh', criteria: 'Không lãng phí, sử dụng đúng hướng dẫn', maxScore: 15, selfScore: 15, adminScore: 15 },
                    { id: 'item_cleaner_a2_2', title: 'Bảo quản và kiểm tra trang thiết bị làm việc', criteria: 'Bảo dưỡng tốt, báo cáo kịp thời hư hỏng', maxScore: 15, selfScore: 15, adminScore: 15 }
                ]
            },
            {
                id: 'sub_cleaner_b1', section: 'B', code: 'I', title: 'Chấp hành Kỷ luật & Thái độ làm việc', maxScore: 30,
                items: [
                    { id: 'item_cleaner_b1_1', title: 'Chấp hành thời gian làm việc và nội quy khoa/viện', criteria: 'Đúng giờ, không tự ý bỏ vị trí', maxScore: 15, selfScore: 15, adminScore: 15 },
                    { id: 'item_cleaner_b1_2', title: 'Thái độ giao tiếp, ứng xử với cán bộ và đồng nghiệp', criteria: 'Mực thước, hòa nhã, lịch sự', maxScore: 15, selfScore: 15, adminScore: 15 }
                ]
            }
        ];

        const defaultTasks = [
            { id: 'T001', name: 'Nghiên cứu tài liệu khoa học', status: 'Đang làm', date: '02/09/2026', priority: 'Cao', files: [] },
            { id: 'T002', name: 'Chuẩn bị hóa chất phòng thí nghiệm', status: 'Chưa làm', date: '30/09/2026', priority: 'Bình thường', files: [] },
            { id: 'T003', name: 'Viết báo cáo tổng kết tháng', status: 'Hoàn thành', date: '01/09/2026', priority: 'Thấp', files: [] }
        ];

        function initMonthSelector() {
            const monthSelect = document.getElementById('select-kpi-month');
            const taskMonthSelect = document.getElementById('select-kpi-task-month');
            
            if (monthSelect) monthSelect.innerHTML = '';
            if (taskMonthSelect) taskMonthSelect.innerHTML = '';

            const now = new Date();
            const currentYear = now.getFullYear();

            selectedKpiMonth = `${currentYear}-${String(now.getMonth() + 1).padStart(2, '0')}`;
            selectedKPITaskMonth = selectedKpiMonth;

            for (let y = currentYear; y >= currentYear - 1; y--) {
                for (let m = 12; m >= 1; m--) {
                    const monthKey = `${y}-${String(m).padStart(2, '0')}`;
                    const label = `Đánh giá KPI Tháng ${m}/${y}`;
                    
                    if (monthSelect) {
                        const option = document.createElement('option');
                        option.value = monthKey;
                        option.innerText = label;
                        if (monthKey === selectedKpiMonth) option.selected = true;
                        monthSelect.appendChild(option);
                    }

                    if (taskMonthSelect) {
                        const optionTask = document.createElement('option');
                        optionTask.value = monthKey;
                        optionTask.innerText = `Nhiệm vụ KPI Tháng ${m}/${y}`;
                        if (monthKey === selectedKPITaskMonth) optionTask.selected = true;
                        taskMonthSelect.appendChild(optionTask);
                    }
                }
            }
            updateSelectedMonthText();
        }

        function updateSelectedMonthText() {
            if (!selectedKpiMonth) return;
            const parts = selectedKpiMonth.split('-');
            const el = document.getElementById('selected-month-text');
            if (el) el.innerText = `Tháng ${parts[1]}/${parts[0]}`;
        }

        function changeKPIMonth(newMonth) {
            selectedKpiMonth = newMonth;
            updateSelectedMonthText();
            listenRealtimeKPI();
        }

        function changeKPITaskMonth(newMonth) {
            selectedKPITaskMonth = newMonth;
            listenRealtimeKPITasks();
        }

        function changeKPITaskTargetUser(val) {
            kpiTaskTargetUser = val;
            listenRealtimeKPITasks();
        }

        function listenRealtimeKPITasks() {
            if (!kpiTaskTargetUser || !selectedKPITaskMonth) return;

            const nameDisplay = document.getElementById('kpi-task-target-name-display');
            if (nameDisplay) nameDisplay.innerText = kpiTaskTargetUser;

            const btnAddRow = document.getElementById('btn-add-kpi-task-row');
            const thAction = document.getElementById('th-kpi-task-action');

            if (isAdmin) {
                if (btnAddRow) btnAddRow.style.display = 'inline-block';
                if (thAction) thAction.style.display = 'table-cell';
            } else {
                if (btnAddRow) btnAddRow.style.display = 'none';
                if (thAction) thAction.style.display = 'none';
            }

            db.ref(`kpiTasks/${selectedKPITaskMonth}/${kpiTaskTargetUser}`).on('value', snapshot => {
                const data = snapshot.val();
                if (data && Array.isArray(data)) {
                    kpiTasksList = data;
                } else {
                    kpiTasksList = [
                        { content: 'Thực hiện giảng dạy và quản lý sinh viên đúng tiến độ', proof: 'Lịch giảng dạy, bảng điểm danh' },
                        { content: 'Nghiên cứu khoa học và công bố bài báo chuyên ngành', proof: 'Bản thảo bài báo, xác nhận nộp bài' }
                    ];
                }
                renderKPITasksTable();
            });
        }

        function renderKPITasksTable() {
            const tbody = document.getElementById('kpi-tasks-table-body');
            if (!tbody) return;
            tbody.innerHTML = '';

            if (kpiTasksList.length === 0) {
                tbody.innerHTML = `<tr><td colspan="${isAdmin ? 4 : 3}" style="text-align:center; color:#94a3b8; padding:20px;">Chưa có dữ liệu nhiệm vụ KPI cho tháng này.</td></tr>`;
                return;
            }

            kpiTasksList.forEach((task, index) => {
                const tr = document.createElement('tr');
                tr.innerHTML = `
                    <td style="text-align: center; font-weight: bold;">${index + 1}</td>
                    <td>
                        <textarea class="kpi-task-input-area" ${!isAdmin ? 'readonly style="background:#f8fafc;"' : ''} onchange="updateKPITaskContent(${index}, this.value)" placeholder="Nhập nội dung nhiệm vụ...">${task.content || ''}</textarea>
                    </td>
                    <td>
                        <textarea class="kpi-task-input-area" onchange="updateKPITaskProof(${index}, this.value)" placeholder="Nhập minh chứng tối thiểu...">${task.proof || ''}</textarea>
                    </td>
                    ${isAdmin ? `
                        <td style="text-align: center;">
                            <button class="btn-sm btn-delete" onclick="deleteKPITaskRow(${index})"><i class="fa-solid fa-trash"></i></button>
                        </td>
                    ` : ''}
                `;
                tbody.appendChild(tr);
            });
        }

        function updateKPITaskContent(index, val) {
            if (kpiTasksList[index]) {
                kpiTasksList[index].content = val;
            }
        }

        function updateKPITaskProof(index, val) {
            if (kpiTasksList[index]) {
                kpiTasksList[index].proof = val;
            }
        }

        function addKPITaskRow() {
            if (!isAdmin) return;
            kpiTasksList.push({ content: '', proof: '' });
            renderKPITasksTable();
        }

        function deleteKPITaskRow(index) {
            if (!isAdmin) return;
            if (confirm('Bạn có chắc chắn muốn xóa hàng nhiệm vụ này?')) {
                kpiTasksList.splice(index, 1);
                renderKPITasksTable();
            }
        }

        function saveKPITasksData() {
            if (!kpiTaskTargetUser || !selectedKPITaskMonth) return;
            db.ref(`kpiTasks/${selectedKPITaskMonth}/${kpiTaskTargetUser}`).set(kpiTasksList, (err) => {
                if (!err) {
                    alert('Đã lưu thành công dữ liệu Nhiệm vụ KPI!');
                } else {
                    alert('Lỗi khi lưu dữ liệu: ' + err.message);
                }
            });
        }

        function exportKPITaskWord() {
            if (!window.docx) {
                alert("Thư viện xuất file Word chưa tải xong. Vui lòng thử lại!");
                return;
            }

            const { Document, Packer, Paragraph, TextRun, Table, TableRow, TableCell, AlignmentType, WidthType, BorderStyle } = window.docx;

            const tableRows = [
                new TableRow({
                    children: [
                        new TableCell({
                            children: [new Paragraph({ children: [new TextRun({ text: "STT", bold: true, font: "Times New Roman" })], alignment: AlignmentType.CENTER })],
                            width: { size: 10, type: WidthType.PERCENTAGE }
                        }),
                        new TableCell({
                            children: [new Paragraph({ children: [new TextRun({ text: "Nội dung nhiệm vụ", bold: true, font: "Times New Roman" })], alignment: AlignmentType.CENTER })],
                            width: { size: 45, type: WidthType.PERCENTAGE }
                        }),
                        new TableCell({
                            children: [new Paragraph({ children: [new TextRun({ text: "Minh chứng tối thiểu", bold: true, font: "Times New Roman" })], alignment: AlignmentType.CENTER })],
                            width: { size: 45, type: WidthType.PERCENTAGE }
                        })
                    ]
                })
            ];

            kpiTasksList.forEach((task, index) => {
                tableRows.push(
                    new TableRow({
                        children: [
                            new TableCell({
                                children: [new Paragraph({ children: [new TextRun({ text: String(index + 1), font: "Times New Roman" })], alignment: AlignmentType.CENTER })],
                                width: { size: 10, type: WidthType.PERCENTAGE }
                            }),
                            new TableCell({
                                children: [new Paragraph({ children: [new TextRun({ text: task.content || "", font: "Times New Roman" })] })],
                                width: { size: 45, type: WidthType.PERCENTAGE }
                            }),
                            new TableCell({
                                children: [new Paragraph({ children: [new TextRun({ text: task.proof || "", font: "Times New Roman" })] })],
                                width: { size: 45, type: WidthType.PERCENTAGE }
                            })
                        ]
                    })
                );
            });

            const monthParts = selectedKPITaskMonth.split('-');
            const monthText = `Tháng ${monthParts[1]} năm ${monthParts[0]}`;

            const doc = new Document({
                sections: [{
                    properties: {},
                    children: [
                        new Paragraph({
                            children: [new TextRun({ text: `NHIỆM VỤ KPI - ${monthText.toUpperCase()}`, bold: true, size: 28, font: "Times New Roman", color: "1E40AF" })],
                            alignment: AlignmentType.CENTER,
                            spacing: { after: 200 }
                        }),
                        new Paragraph({
                            children: [new TextRun({ text: `Họ và tên cán bộ: ${kpiTaskTargetUser.toUpperCase()}`, bold: true, size: 24, font: "Times New Roman" })],
                            alignment: AlignmentType.LEFT,
                            spacing: { after: 300 }
                        }),
                        new Table({
                            rows: tableRows,
                            width: { size: 100, type: WidthType.PERCENTAGE }
                        })
                    ]
                }]
            });

            Packer.toBlob(doc).then(blob => {
                saveAs(blob, `Nhiem_Vu_KPI_${kpiTaskTargetUser}_${selectedKPITaskMonth}.docx`);
            });
        }

        window.addEventListener('DOMContentLoaded', () => {
            initMonthSelector();
            const dateInput = document.getElementById('newTaskDueDate');
            if (dateInput) {
                dateInput.value = new Date().toISOString().split('T')[0];
            }
        });

        /* --- BOX CHAT REALTIME 2 CHIỀU --- */
        function listenRealtimeChat() {
            db.ref('chatConversations').on('value', snapshot => {
                const data = snapshot.val();
                chatMessagesMap = data || {};
                renderChatInterface();
            });
        }

        function markMessagesAsRead(userKey) {
            if (!isAdmin || !userKey) return;
            const userChatData = chatMessagesMap[userKey] || {};
            const updates = {};
            let hasUnread = false;

            Object.keys(userChatData).forEach(msgId => {
                const msg = userChatData[msgId];
                if (msg.sender !== ADMIN_USERNAME && msg.readByAdmin === false) {
                    updates[`chatConversations/${userKey}/${msgId}/readByAdmin`] = true;
                    hasUnread = true;
                }
            });

            if (hasUnread) {
                db.ref().update(updates);
            }
        }

        function renderChatInterface() {
            const sidebar = document.getElementById('chat-user-list-sidebar');
            const userWrapper = document.getElementById('chat-user-items-wrapper');
            const chatHeader = document.getElementById('chat-header-title');

            if (isAdmin) {
                sidebar.style.display = 'flex';
                userWrapper.innerHTML = '';

                const normalUsers = registeredUsers.filter(u => u.username !== ADMIN_USERNAME);
                
                if (normalUsers.length === 0) {
                    userWrapper.innerHTML = '<div style="padding:15px; font-size:12px; color:#94a3b8;">Chưa có người dùng.</div>';
                } else {
                    const userChatMeta = normalUsers.map(u => {
                        const userChatData = chatMessagesMap[u.username] || {};
                        const messages = Object.values(userChatData);
                        let unreadCount = 0;
                        let lastTimestamp = 0;

                        messages.forEach(msg => {
                            if (msg.createdTime && msg.createdTime > lastTimestamp) {
                                lastTimestamp = msg.createdTime;
                            }
                            if (msg.sender !== ADMIN_USERNAME && msg.readByAdmin === false) {
                                unreadCount++;
                            }
                        });

                        return {
                            username: u.username,
                            unreadCount: unreadCount,
                            lastTimestamp: lastTimestamp
                        };
                    });

                    userChatMeta.sort((a, b) => {
                        if (a.unreadCount > 0 && b.unreadCount === 0) return -1;
                        if (a.unreadCount === 0 && b.unreadCount > 0) return 1;
                        return b.lastTimestamp - a.lastTimestamp;
                    });

                    if (!activeChatTargetUser || !normalUsers.some(u => u.username === activeChatTargetUser)) {
                        activeChatTargetUser = userChatMeta[0] ? userChatMeta[0].username : '';
                    }

                    userChatMeta.forEach(meta => {
                        const item = document.createElement('div');
                        item.className = `chat-user-item ${meta.username === activeChatTargetUser ? 'active' : ''}`;
                        item.onclick = () => {
                            activeChatTargetUser = meta.username;
                            markMessagesAsRead(activeChatTargetUser);
                            renderChatInterface();
                        };

                        const badgeHTML = meta.unreadCount > 0 
                            ? `<span class="chat-unread-badge" title="${meta.unreadCount} tin nhắn chưa đọc">${meta.unreadCount}</span>` 
                            : '';

                        item.innerHTML = `
                            <div style="display: flex; align-items: center; gap: 8px;">
                                <i class="fa-solid fa-user"></i> 
                                <span>${meta.username}</span>
                                ${badgeHTML}
                            </div>
                            <i class="fa-solid fa-chevron-right" style="font-size:10px;"></i>
                        `;
                        userWrapper.appendChild(item);
                    });

                    markMessagesAsRead(activeChatTargetUser);
                }

                chatHeader.innerText = `Trò chuyện 2 chiều với cán bộ: ${activeChatTargetUser || 'Chưa chọn'}`;
                renderChatHistory(activeChatTargetUser);
            } else {
                sidebar.style.display = 'none';
                chatHeader.innerText = `Trò chuyện trực tiếp với Admin`;
                renderChatHistory(currentUser);
            }
        }

        function renderChatHistory(userKey) {
            const historyEl = document.getElementById('chat-history-list');
            historyEl.innerHTML = '';

            if (!userKey) {
                historyEl.innerHTML = '<div style="text-align: center; color: #94a3b8; padding: 20px;">Không tìm thấy dữ liệu trò chuyện.</div>';
                return;
            }

            const userChatData = chatMessagesMap[userKey] || {};
            const messages = Object.values(userChatData);

            if (messages.length === 0) {
                historyEl.innerHTML = '<div style="text-align: center; color: #94a3b8; padding: 20px;">Chưa có tin nhắn nào. Hãy gửi tin nhắn đầu tiên!</div>';
                return;
            }

            messages.sort((a, b) => (a.createdTime || 0) - (b.createdTime || 0));

            messages.forEach(msg => {
                const msgDiv = document.createElement('div');
                const isMyMessage = msg.sender === currentUser;
                msgDiv.className = `chat-msg-item ${isMyMessage ? 'chat-msg-sent' : 'chat-msg-received'}`;

                let fileHTML = '';
                if (msg.file) {
                    fileHTML = `
                        <div class="file-tag" style="margin-top: 6px; gap: 6px;">
                            <i class="fa-solid fa-paperclip"></i>
                            <a href="${msg.file.data}" download="${msg.file.name}" title="Tải tệp đính kèm">${msg.file.name}</a>
                            <button class="btn-sm btn-view" onclick="viewFileOnline('${msg.file.name.replace(/'/g, "\\'")}', '${msg.file.data}')"><i class="fa-solid fa-eye"></i> Xem</button>
                        </div>
                    `;
                }

                msgDiv.innerHTML = `
                    <div class="chat-msg-header">
                        <span class="chat-msg-sender">${msg.sender}</span>
                        <span>${msg.timestamp}</span>
                    </div>
                    <div class="chat-msg-body">${msg.text ? msg.text : ''}</div>
                    ${fileHTML}
                `;
                historyEl.appendChild(msgDiv);
            });

            historyEl.scrollTop = historyEl.scrollHeight;
        }

        function handleChatFileSelect(inputEl) {
            const file = inputEl.files[0];
            const previewEl = document.getElementById('chat-file-name-preview');
            if (!file) {
                selectedChatFile = null;
                previewEl.innerHTML = '';
                return;
            }

            if (file.size > 8 * 1024 * 1024) {
                alert('Dung lượng tệp vượt quá 8MB. Vui lòng chọn tệp nhỏ hơn!');
                inputEl.value = '';
                selectedChatFile = null;
                previewEl.innerHTML = '';
                return;
            }

            const reader = new FileReader();
            reader.onload = function(e) {
                selectedChatFile = {
                    name: file.name,
                    data: e.target.result
                };
                previewEl.innerHTML = `<i class="fa-solid fa-file-arrow-up"></i> Tệp: <strong>${file.name}</strong> <i class="fa-solid fa-xmark" style="cursor:pointer; color:#ef4444; margin-left:4px;" onclick="clearChatFile()"></i>`;
            };
            reader.readAsDataURL(file);
        }

        function clearChatFile() {
            selectedChatFile = null;
            document.getElementById('chat-file-input').value = '';
            document.getElementById('chat-file-name-preview').innerHTML = '';
        }

        function sendChatMessage() {
            const messageInput = document.getElementById('chat-message-input');
            const text = messageInput.value.trim();

            if (!text && !selectedChatFile) {
                alert('Vui lòng nhập nội dung tin nhắn hoặc đính kèm tệp!');
                return;
            }

            const targetConversationUser = isAdmin ? activeChatTargetUser : currentUser;
            if (!targetConversationUser) {
                alert('Vui lòng chọn người dùng để nhắn tin!');
                return;
            }

            const now = Date.now();
            const msgId = 'MSG_' + now;
            const newMsg = {
                id: msgId,
                sender: currentUser,
                timestamp: new Date().toLocaleString('vi-VN'),
                createdTime: now,
                text: text,
                file: selectedChatFile || null,
                readByAdmin: isAdmin ? true : false
            };

            db.ref(`chatConversations/${targetConversationUser}/${msgId}`).set(newMsg, (err) => {
                if (!err) {
                    messageInput.value = '';
                    clearChatFile();
                } else {
                    alert('Lỗi khi gửi tin nhắn: ' + err.message);
                }
            });
        }

        function listenRealtimeSchedules() {
            db.ref('scheduleFiles').on('value', snapshot => {
                const data = snapshot.val();
                if (data) {
                    scheduleFiles = Object.values(data);
                } else {
                    scheduleFiles = [];
                }
                renderScheduleFiles();
            });
        }

        function uploadScheduleFile(inputEl) {
            if (!isAdmin) {
                alert("Chỉ có tài khoản Admin (linhnguyenxuan) mới có quyền đăng tệp Lịch công tác!");
                return;
            }

            const file = inputEl.files[0];
            if (!file) return;

            if (file.size > 10 * 1024 * 1024) {
                alert('Dung lượng file vượt quá 10MB. Vui lòng chọn file nhỏ hơn!');
                inputEl.value = '';
                return;
            }

            const reader = new FileReader();
            reader.onload = function(e) {
                const fileId = 'SCH_' + Date.now();
                const newSchedule = {
                    id: fileId,
                    name: file.name,
                    data: e.target.result,
                    uploadedAt: new Date().toLocaleString('vi-VN'),
                    uploadedBy: currentUser
                };

                db.ref(`scheduleFiles/${fileId}`).set(newSchedule, (err) => {
                    if (!err) {
                        alert(`Đã tải lên tệp Lịch công tác "${file.name}" thành công!`);
                    } else {
                        alert('Lỗi tải file lên: ' + err.message);
                    }
                });
            };
            reader.readAsDataURL(file);
            inputEl.value = '';
        }

        function deleteScheduleFile(fileId) {
            if (!isAdmin) return;
            if (confirm("Bạn có chắc chắn muốn xóa tệp Lịch công tác này?")) {
                db.ref(`scheduleFiles/${fileId}`).remove();
            }
        }

        function renderScheduleFiles() {
            const tbody = document.getElementById('schedule-files-table-body');
            const thAction = document.getElementById('th-schedule-action');
            const uploadBtnContainer = document.getElementById('admin-schedule-upload-container');

            if (isAdmin) {
                thAction.style.display = 'table-cell';
                uploadBtnContainer.style.display = 'block';
            } else {
                thAction.style.display = 'none';
                uploadBtnContainer.style.display = 'none';
            }

            if (!tbody) return;
            tbody.innerHTML = '';

            if (scheduleFiles.length === 0) {
                tbody.innerHTML = `<tr><td colspan="${isAdmin ? 5 : 4}" style="text-align:center; color:#94a3b8; padding:20px;">Chưa có tệp Lịch công tác nào được đăng tải.</td></tr>`;
                return;
            }

            scheduleFiles.sort((a, b) => b.id.localeCompare(a.id));

            scheduleFiles.forEach((file, index) => {
                const tr = document.createElement('tr');
                const safeName = file.name.replace(/'/g, "\\'");
                tr.innerHTML = `
                    <td style="text-align: center;"><strong>${index + 1}</strong></td>
                    <td>
                        <i class="fa-solid fa-file-lines" style="color:#2563eb; margin-right:6px;"></i>
                        <strong>${file.name}</strong>
                    </td>
                    <td>${file.uploadedAt}</td>
                    <td style="text-align: center;">
                        <button class="btn-sm btn-view" onclick="viewFileOnline('${safeName}', '${file.data}')" style="padding: 6px 12px; margin-right: 4px;">
                            <i class="fa-solid fa-eye"></i> Xem trực tiếp
                        </button>
                        <a href="${file.data}" download="${file.name}" class="btn-sm btn-download" style="text-decoration:none; display:inline-block; padding: 6px 12px;">
                            <i class="fa-solid fa-download"></i> Tải về
                        </a>
                    </td>
                    ${isAdmin ? `
                        <td style="text-align: center;">
                            <button class="btn-sm btn-delete" onclick="deleteScheduleFile('${file.id}')"><i class="fa-solid fa-trash"></i> Xóa</button>
                        </td>
                    ` : ''}
                `;
                tbody.appendChild(tr);
            });
        }

        function listenRealtimeTemplates() {
            db.ref('kpiTemplates').on('value', snapshot => {
                const data = snapshot.val();
                if (data) {
                    masterKPITemplates = data;
                    ['staff', 'leader', 'cleaner'].forEach(type => {
                        if (!masterKPITemplates[type]) {
                            masterKPITemplates[type] = {
                                sectionMaxScores: type === 'cleaner' ? { A: 70, B: 30 } : { A: 50, B: 30, C: 20 },
                                sectionTitles: type === 'cleaner' ? 
                                    { A: "CÔNG TÁC CHUYÊN MÔN & VỆ SINH", B: "Ý THỨC CHẤP HÀNH & KỶ LUẬT" } : 
                                    { A: "QUẢN LÝ & CHUYÊN MÔN", B: "Ý THỨC CHẤP HÀNH & KỶ LUẬT", C: "NĂNG LỰC ĐỔI MỚI & NHIỆM VỤ ĐỘT XUẤT" },
                                kpiDataList: type === 'cleaner' ? defaultCleanerKPIStructure : type === 'leader' ? defaultLeaderKPIStructure : defaultStaffKPIStructure
                            };
                            db.ref(`kpiTemplates/${type}`).set(masterKPITemplates[type]);
                        }
                    });
                } else {
                    masterKPITemplates = {
                        staff: { 
                            sectionMaxScores: { A: 50, B: 30, C: 20 }, 
                            sectionTitles: { A: "QUẢN LÝ & CHUYÊN MÔN", B: "Ý THỨC CHẤP HÀNH & KỶ LUẬT", C: "NĂNG LỰC ĐỔI MỚI & NHIỆM VỤ ĐỘT XUẤT" },
                            kpiDataList: defaultStaffKPIStructure 
                        },
                        leader: { 
                            sectionMaxScores: { A: 50, B: 30, C: 20 }, 
                            sectionTitles: { A: "QUẢN LÝ & CHUYÊN MÔN", B: "Ý THỨC CHẤP HÀNH & KỶ LUẬT", C: "NĂNG LỰC ĐỔI MỚI & NHIỆM VỤ ĐỘT XUẤT" },
                            kpiDataList: defaultLeaderKPIStructure 
                        },
                        cleaner: { 
                            sectionMaxScores: { A: 70, B: 30 }, 
                            sectionTitles: { A: "CÔNG TÁC CHUYÊN MÔN & VỆ SINH", B: "Ý THỨC CHẤP HÀNH & KỶ LUẬT" },
                            kpiDataList: defaultCleanerKPIStructure 
                        }
                    };
                    db.ref('kpiTemplates').set(masterKPITemplates);
                }
                
                if (kpiTargetUser) {
                    loadAndMergeUserKPI(kpiTargetUser, selectedKpiMonth);
                }
            });
        }

        function listenRealtimeUsers() {
            db.ref('users').on('value', snapshot => {
                const data = snapshot.val();
                let loadedUsers = data ? Object.values(data) : [];
                
                const hasAdmin = loadedUsers.some(u => u.username === ADMIN_USERNAME);
                if (!hasAdmin) {
                    const adminObj = { email: 'admin@hoaly.edu.vn', username: ADMIN_USERNAME, password: ADMIN_PASSWORD, kpiType: 'leader' };
                    db.ref(`users/${ADMIN_USERNAME}`).set(adminObj);
                    loadedUsers.push(adminObj);
                }

                registeredUsers = loadedUsers;
                if (isAdmin) {
                    populateAdminUserSelector();
                    populateKPITargetSelector();
                    populateKPITaskTargetSelector();
                    renderUserManagementTable();
                    renderDashboard(); 
                }
                renderChatInterface();
            });
        }

        function populateKPITaskTargetSelector() {
            const sel = document.getElementById('select-kpi-task-target-user');
            if (!sel) return;
            sel.innerHTML = '';
            registeredUsers.forEach(u => {
                const opt = document.createElement('option');
                opt.value = u.username;
                opt.innerText = u.username;
                if (u.username === kpiTaskTargetUser) opt.selected = true;
                sel.appendChild(opt);
            });
        }

        function listenAllUsersTasks() {
            db.ref('tasks').on('value', snapshot => {
                const data = snapshot.val();
                if (data) {
                    allUsersTasksMap = data;
                } else {
                    allUsersTasksMap = {};
                }
                if (isAdmin) {
                    renderDashboard();
                }
            });
        }

        function listenRealtimeTasks() {
            if (!targetUser) return;
            db.ref(`tasks/${targetUser}`).on('value', snapshot => {
                const data = snapshot.val();
                if (data) {
                    tasks = Object.values(data);
                } else {
                    tasks = defaultTasks.map(t => ({ ...t, user: targetUser }));
                    saveUserData();
                }
                renderDashboard();
                renderTaskList();
                renderKanban();
            });
        }

        function listenRealtimeKPI() {
            if (!kpiTargetUser || !selectedKpiMonth) return;
            
            const targetUserInfo = registeredUsers.find(u => u.username === kpiTargetUser);
            if (targetUserInfo) {
                const sel = document.getElementById('select-kpi-type-change');
                if (sel) sel.value = targetUserInfo.kpiType || 'staff';
            }
            
            if (isAdmin) {
                document.getElementById('btn-save-kpi-admin').style.display = 'inline-block';
                document.getElementById('btn-add-main-section').style.display = 'inline-block';
                if (currentUser === kpiTargetUser) {
                    document.getElementById('btn-save-kpi-user').style.display = 'inline-block';
                } else {
                    document.getElementById('btn-save-kpi-user').style.display = 'none';
                }
            } else {
                document.getElementById('btn-save-kpi-admin').style.display = 'none';
                document.getElementById('btn-add-main-section').style.display = 'none';
                document.getElementById('btn-save-kpi-user').style.display = 'inline-block';
            }
            
            loadAndMergeUserKPI(kpiTargetUser, selectedKpiMonth);
        }

        function loadAndMergeUserKPI(username, monthKey) {
            const targetUserInfo = registeredUsers.find(u => u.username === username);
            const userKpiType = (targetUserInfo && targetUserInfo.kpiType) ? targetUserInfo.kpiType : 'staff';
            const currentTemplate = masterKPITemplates[userKpiType] || masterKPITemplates['staff'];

            db.ref(`kpi/${monthKey}/${username}`).on('value', snapshot => {
                const data = snapshot.val();
                sectionMaxScores = currentTemplate.sectionMaxScores || (userKpiType === 'cleaner' ? { A: 70, B: 30 } : { A: 50, B: 30, C: 20 });
                
                const defaultTitles = userKpiType === 'cleaner' ? 
                    { A: "CÔNG TÁC CHUYÊN MÔN & VỆ SINH", B: "Ý THỨC CHẤP HÀNH & KỶ LUẬT" } : 
                    { A: "QUẢN LÝ & CHUYÊN MÔN", B: "Ý THỨC CHẤP HÀNH & KỶ LUẬT", C: "NĂNG LỰC ĐỔI MỚI & NHIỆM VỤ ĐỘT XUẤT" };

                sectionTitles = currentTemplate.sectionTitles || defaultTitles;

                if (data && data.kpiDataList) {
                    lastKpiTimestamp = data.timestamp || 'Chưa ghi nhận';
                    isKpiSavedForCurrentMonth = true;
                    kpiDataList = data.kpiDataList;
                } else {
                    lastKpiTimestamp = 'Chưa có dữ liệu';
                    isKpiSavedForCurrentMonth = false;
                    kpiDataList = JSON.parse(JSON.stringify(currentTemplate.kpiDataList));
                }
                renderKPITable();
            });
        }

        function mergeKPIWithTemplate(templateList, userList) {
            const templateCopy = JSON.parse(JSON.stringify(templateList || []));
            const userItemMap = {};

            if (userList) {
                userList.forEach(sub => {
                    if (sub.items) {
                        sub.items.forEach(item => {
                            userItemMap[item.id] = {
                                title: item.title,
                                criteria: item.criteria,
                                maxScore: item.maxScore,
                                selfScore: item.selfScore,
                                adminScore: item.adminScore
                            };
                        });
                    }
                });
            }

            templateCopy.forEach(sub => {
                if (sub.items) {
                    sub.items.forEach(item => {
                        if (userItemMap[item.id]) {
                            item.title = userItemMap[item.id].title || item.title;
                            item.criteria = userItemMap[item.id].criteria || item.criteria;
                            item.maxScore = userItemMap[item.id].maxScore !== undefined ? userItemMap[item.id].maxScore : item.maxScore;
                            item.selfScore = userItemMap[item.id].selfScore !== undefined ? userItemMap[item.id].selfScore : item.maxScore;
                            item.adminScore = userItemMap[item.id].adminScore !== undefined ? userItemMap[item.id].adminScore : item.maxScore;
                        }
                    });
                }
            });

            return templateCopy;
        }

        function saveUserData() {
            if (!targetUser) return;
            db.ref(`tasks/${targetUser}`).set(tasks);
        }

        function saveKPIRatingStorage(timestamp = null) {
            if (!kpiTargetUser || !selectedKpiMonth) return;
            let timeSaved = timestamp || lastKpiTimestamp;
            db.ref(`kpi/${selectedKpiMonth}/${kpiTargetUser}`).set({
                timestamp: timeSaved,
                kpiDataList: kpiDataList
            }, (err) => {
                if (!err) {
                    isKpiSavedForCurrentMonth = true;
                }
            });
        }

        function isTaskOverdue(dateStr, status) {
            if (status === 'Hoàn thành' || !dateStr) return false;
            let day, month, year;
            if (dateStr.includes('-')) {
                [year, month, day] = dateStr.split('-');
            } else if (dateStr.includes('/')) {
                [day, month, year] = dateStr.split('/');
            } else {
                return false;
            }
            const dueDate = new Date(parseInt(year), parseInt(month) - 1, parseInt(day), 23, 59, 59);
            return dueDate < new Date();
        }

        function toggleAuthTab(tab) {
            const loginForm = document.getElementById('login-form');
            const registerForm = document.getElementById('register-form');
            const loginBtn = document.getElementById('tab-login-btn');
            const regBtn = document.getElementById('tab-register-btn');
            const subTitle = document.getElementById('form-sub-title');

            if (tab === 'login') {
                loginForm.style.display = 'block'; registerForm.style.display = 'none';
                loginBtn.style.color = '#2563eb'; loginBtn.style.borderBottom = '2px solid #2563eb';
                regBtn.style.color = '#64748b'; regBtn.style.borderBottom = 'none';
                subTitle.innerText = 'Đăng nhập hệ thống quản trị công việc (Realtime)';
            } else {
                loginForm.style.display = 'none'; registerForm.style.display = 'block';
                regBtn.style.color = '#10b981'; regBtn.style.borderBottom = '2px solid #10b981';
                loginBtn.style.color = '#64748b'; loginBtn.style.borderBottom = 'none';
                subTitle.innerText = 'Tạo tài khoản mới cho cán bộ';
            }
        }

        function handleRegister(e) {
            e.preventDefault();
            const email = document.getElementById('reg-email').value.trim();
            const username = document.getElementById('reg-username').value.trim();
            const password = document.getElementById('reg-password').value;
            const confirmPassword = document.getElementById('reg-confirm-password').value;
            const kpiType = document.getElementById('reg-kpi-type').value;

            if (username === ADMIN_USERNAME) { alert('Tên đăng nhập trùng với Admin!'); return; }
            if (password !== confirmPassword) { alert('Mật khẩu không khớp!'); return; }
            if (registeredUsers.some(u => u.username === username)) { alert('Tên đăng nhập đã tồn tại!'); return; }

            const newUser = { email, username, password, kpiType: kpiType };
            db.ref(`users/${username}`).set(newUser, (err) => {
                if (!err) {
                    alert('Đăng ký tài khoản thành công!');
                    document.getElementById('register-form').reset();
                    toggleAuthTab('login');
                    document.getElementById('login-username').value = username;
                } else {
                    alert('Lỗi đăng ký: ' + err.message);
                }
            });
        }

        function handleLogin(e) {
            e.preventDefault();
            const usernameInput = document.getElementById('login-username').value.trim();
            const passwordInput = document.getElementById('login-password').value;

            const validUser = registeredUsers.find(u => u.username === usernameInput && u.password === passwordInput);

            if (usernameInput === ADMIN_USERNAME && passwordInput === ADMIN_PASSWORD) {
                currentUser = ADMIN_USERNAME;
                isAdmin = true;
            } else if (validUser) {
                currentUser = validUser.username;
                isAdmin = false;
            } else {
                alert('Tên đăng nhập hoặc mật khẩu không đúng!');
                return;
            }

            targetUser = currentUser;
            kpiTargetUser = currentUser;
            kpiTaskTargetUser = currentUser;

            document.getElementById('login-screen').style.display = 'none';
            document.getElementById('app-screen').style.display = 'flex';
            document.getElementById('user-display-name').innerText = currentUser;
            document.getElementById('user-role-badge').innerText = isAdmin ? 'Admin Root' : 'Cán bộ';

            if (isAdmin) {
                document.getElementById('admin-user-selector').style.display = 'flex';
                document.getElementById('kpi-admin-target-bar').style.display = 'flex';
                document.getElementById('kpi-task-admin-target-bar').style.display = 'flex';
                document.getElementById('nav-admin-users').style.display = 'flex';
                document.getElementById('admin-master-overview-banner').style.display = 'block';
                document.getElementById('chat-sub-title').innerText = 'Chọn tài khoản bên trái để trao đổi thông tin 2 chiều và quản lý tin nhắn.';
                populateAdminUserSelector();
                populateKPITargetSelector();
                populateKPITaskTargetSelector();
                renderUserManagementTable();
            } else {
                document.getElementById('admin-user-selector').style.display = 'none';
                document.getElementById('kpi-admin-target-bar').style.display = 'none';
                document.getElementById('kpi-task-admin-target-bar').style.display = 'none';
                document.getElementById('nav-admin-users').style.display = 'none';
                document.getElementById('admin-master-overview-banner').style.display = 'none';
                document.getElementById('chat-sub-title').innerText = 'Tất cả tin nhắn và file đính kèm của bạn sẽ được gửi trực tiếp và bảo mật đến tài khoản Admin.';
            }

            listenRealtimeTasks();
            listenRealtimeKPI();
            listenRealtimeKPITasks();
            renderScheduleFiles();
            renderChatInterface();
        }

        function logout() {
            currentUser = '';
            targetUser = '';
            kpiTargetUser = '';
            kpiTaskTargetUser = '';
            isAdmin = false;
            document.getElementById('app-screen').style.display = 'none';
            document.getElementById('login-screen').style.display = 'flex';
        }

        function populateAdminUserSelector() {
            const select = document.getElementById('select-target-user');
            if (!select) return;
            select.innerHTML = '';
            registeredUsers.forEach(u => {
                const opt = document.createElement('option');
                opt.value = u.username;
                opt.innerText = u.username + (u.username === currentUser ? ' (Tôi)' : '');
                if (u.username === targetUser) opt.selected = true;
                select.appendChild(opt);
            });
        }

        function populateKPITargetSelector() {
            const select = document.getElementById('select-kpi-target-user');
            if (!select) return;
            select.innerHTML = '';
            registeredUsers.forEach(u => {
                const opt = document.createElement('option');
                opt.value = u.username;
                opt.innerText = u.username + (u.username === currentUser ? ' (Tôi)' : '');
                if (u.username === kpiTargetUser) opt.selected = true;
                select.appendChild(opt);
            });
        }

        function changeTargetUser(val) {
            targetUser = val;
            listenRealtimeTasks();
        }

        function changeKPITargetUser(val) {
            kpiTargetUser = val;
            listenRealtimeKPI();
        }

        function adminChangeUserKPIType(newType) {
            if (!isAdmin) return;
            if (confirm(`Bạn có chắc muốn chuyển loại bảng KPI của tài khoản "${kpiTargetUser}" sang "${newType}"?`)) {
                db.ref(`users/${kpiTargetUser}/kpiType`).set(newType, (err) => {
                    if (!err) {
                        alert('Đã cập nhật loại Bảng KPI thành công!');
                        listenRealtimeKPI();
                    }
                });
            }
        }

        function switchTab(tabId, element) {
            document.querySelectorAll('.tab-content').forEach(el => el.classList.remove('active'));
            document.querySelectorAll('.nav-item').forEach(el => el.classList.remove('active'));
            document.getElementById(`tab-${tabId}`).classList.add('active');
            element.classList.add('active');

            const titles = {
                'tong-quan': 'Dashboard Thống Kê',
                'danh-sach': 'Danh Sách Công Việc',
                'kanban': 'Bảng Tiến Độ Kanban',
                'lich-cong-tac': 'Lịch Công Tác Khoa Hóa Lý',
                'nhiem-vu-kpi': 'Quản Lý Nhiệm Vụ KPI',
                'kpi': 'Phiếu Đánh Giá KPI Viên Chức',
                'admin-users': 'Quản Lý Tài Khoản Hệ Thống',
                'box-chat': 'Box Chat Trao Đổi 2 Chiều'
            };
            document.getElementById('page-title').innerText = titles[tabId] || 'Workspace';

            if (tabId === 'tong-quan') renderDashboard();
            if (tabId === 'kanban') renderKanban();
            if (tabId === 'box-chat') {
                renderChatInterface();
                markMessagesAsRead(activeChatTargetUser);
            }
        }

        function renderDashboard() {
            let activeTasksList = [];

            if (isAdmin) {
                const allTasksCombined = [];
                Object.keys(allUsersTasksMap).forEach(u => {
                    const userTasks = Object.values(allUsersTasksMap[u] || {});
                    userTasks.forEach(t => allTasksCombined.push({ ...t, user: u }));
                });
                activeTasksList = allTasksCombined;
            } else {
                activeTasksList = tasks;
            }

            const statusCounts = { 'Chưa làm': 0, 'Đang làm': 0, 'Hoàn thành': 0 };
            const priorityCounts = { 'Cao': 0, 'Bình thường': 0, 'Thấp': 0 };

            activeTasksList.forEach(t => {
                if (statusCounts[t.status] !== undefined) statusCounts[t.status]++;
                if (priorityCounts[t.priority] !== undefined) priorityCounts[t.priority]++;
            });

            if (statusChartInstance) statusChartInstance.destroy();
            const ctxStatus = document.getElementById('statusChart').getContext('2d');
            statusChartInstance = new Chart(ctxStatus, {
                type: 'doughnut',
                data: {
                    labels: ['Chưa làm', 'Đang làm', 'Hoàn thành'],
                    datasets: [{
                        data: [statusCounts['Chưa làm'], statusCounts['Đang làm'], statusCounts['Hoàn thành']],
                        backgroundColor: ['#a855f7', '#3b82f6', '#22c55e']
                    }]
                },
                options: { responsive: true, maintainAspectRatio: false }
            });

            if (priorityChartInstance) priorityChartInstance.destroy();
            const ctxPriority = document.getElementById('priorityChart').getContext('2d');
            priorityChartInstance = new Chart(ctxPriority, {
                type: 'bar',
                data: {
                    labels: ['Cao', 'Bình thường', 'Thấp'],
                    datasets: [{
                        label: 'Số lượng công việc',
                        data: [priorityCounts['Cao'], priorityCounts['Bình thường'], priorityCounts['Thấp']],
                        backgroundColor: ['#ef4444', '#f59e0b', '#64748b']
                    }]
                },
                options: { responsive: true, maintainAspectRatio: false, scales: { y: { beginAtZero: true, ticks: { stepSize: 1 } } } }
            });
        }

        function renderTaskList() {
            const tbody = document.getElementById('task-table-body');
            if (!tbody) return;
            tbody.innerHTML = '';

            tasks.forEach((t) => {
                const tr = document.createElement('tr');
                const overdue = isTaskOverdue(t.date, t.status);
                if (overdue) tr.classList.add('row-overdue');

                const displayDate = convertToDisplayDate(t.date);
                const isoDate = convertToISODate(t.date);

                let statusClass = 'status-todo';
                if (t.status === 'Đang làm') statusClass = 'status-doing';
                if (t.status === 'Hoàn thành') statusClass = 'status-done';

                let filesHTML = '';
                if (t.files && t.files.length > 0) {
                    t.files.forEach((f, fIdx) => {
                        const safeFileName = f.name.replace(/'/g, "\\'");
                        filesHTML += `
                            <div class="file-tag">
                                <i class="fa-solid fa-paperclip"></i> 
                                <a href="${f.data}" download="${f.name}">${f.name}</a>
                                <button class="btn-sm btn-view" onclick="viewFileOnline('${safeFileName}', '${f.data}')"><i class="fa-solid fa-eye"></i> Xem</button>
                                <i class="fa-solid fa-xmark" style="cursor:pointer; color:#ef4444; margin-left:4px;" onclick="deleteTaskFile('${t.id}', ${fIdx})"></i>
                            </div><br>
                        `;
                    });
                } else {
                    filesHTML = '<span style="color:#94a3b8; font-size:12px;">Chưa có</span>';
                }

                tr.innerHTML = `
                    <td><strong>${t.id}</strong></td>
                    <td>${t.name}</td>
                    <td><strong>${t.user || targetUser}</strong></td>
                    <td>${t.priority}</td>
                    <td>
                        <select onchange="updateTaskStatus('${t.id}', this.value)" style="padding:4px; border-radius:4px;" class="status-tag ${statusClass}">
                            <option value="Chưa làm" ${t.status === 'Chưa làm' ? 'selected' : ''}>Chưa làm</option>
                            <option value="Đang làm" ${t.status === 'Đang làm' ? 'selected' : ''}>Đang làm</option>
                            <option value="Hoàn thành" ${t.status === 'Hoàn thành' ? 'selected' : ''}>Hoàn thành</option>
                        </select>
                    </td>
                    <td>
                        <input type="date" value="${isoDate}" class="inline-date-picker" onchange="updateTaskDueDate('${t.id}', this.value)" />
                        ${overdue ? '<span class="badge-overdue"><i class="fa-solid fa-triangle-exclamation"></i> Quá hạn</span>' : ''}
                    </td>
                    <td>
                        ${filesHTML}
                        <input type="file" id="file_input_${t.id}" style="display:none;" onchange="handleAttachFile('${t.id}', this)" />
                        <button class="btn-sm btn-attach" style="margin-top:4px;" onclick="document.getElementById('file_input_${t.id}').click()"><i class="fa-solid fa-paperclip"></i> Đính file</button>
                    </td>
                    <td>
                        <button class="btn-sm btn-delete" onclick="deleteTask('${t.id}')"><i class="fa-solid fa-trash"></i> Xóa</button>
                    </td>
                `;
                tbody.appendChild(tr);
            });
        }

        function addNewTask() {
            const name = document.getElementById('newTaskName').value.trim();
            const priority = document.getElementById('newTaskPriority').value;
            const dueDate = document.getElementById('newTaskDueDate').value;

            if (!name) { alert('Vui lòng nhập tên công việc!'); return; }

            const newId = 'T' + String(Date.now()).slice(-4);
            const newTask = {
                id: newId,
                name: name,
                status: 'Chưa làm',
                date: convertToDisplayDate(dueDate),
                priority: priority,
                user: targetUser,
                files: []
            };

            tasks.push(newTask);
            saveUserData();
            document.getElementById('newTaskName').value = '';
        }

        function updateTaskStatus(taskId, newStatus) {
            const t = tasks.find(x => x.id === taskId);
            if (t) {
                t.status = newStatus;
                saveUserData();
            }
        }

        function updateTaskDueDate(taskId, newIsoDate) {
            const t = tasks.find(x => x.id === taskId);
            if (t) {
                t.date = convertToDisplayDate(newIsoDate);
                saveUserData();
            }
        }

        function deleteTask(taskId) {
            if (confirm('Bạn có chắc chắn muốn xóa công việc này?')) {
                tasks = tasks.filter(t => t.id !== taskId);
                saveUserData();
            }
        }

        function handleAttachFile(taskId, inputEl) {
            const file = inputEl.files[0];
            if (!file) return;

            if (file.size > 5 * 1024 * 1024) {
                alert('Dung lượng tệp đính kèm vượt quá 5MB. Vui lòng chọn tệp nhỏ hơn!');
                inputEl.value = '';
                return;
            }

            const reader = new FileReader();
            reader.onload = function(e) {
                const t = tasks.find(x => x.id === taskId);
                if (t) {
                    if (!t.files) t.files = [];
                    t.files.push({ name: file.name, data: e.target.result });
                    saveUserData();
                }
            };
            reader.readAsDataURL(file);
            inputEl.value = '';
        }

        function deleteTaskFile(taskId, fileIdx) {
            const t = tasks.find(x => x.id === taskId);
            if (t && t.files && t.files[fileIdx]) {
                t.files.splice(fileIdx, 1);
                saveUserData();
            }
        }

        function renderKanban() {
            const todoCol = document.getElementById('cards-todo');
            const doingCol = document.getElementById('cards-doing');
            const doneCol = document.getElementById('cards-done');

            if (!todoCol || !doingCol || !doneCol) return;

            todoCol.innerHTML = ''; doingCol.innerHTML = ''; doneCol.innerHTML = '';

            let cTodo = 0, cDoing = 0, cDone = 0;

            tasks.forEach(t => {
                const overdue = isTaskOverdue(t.date, t.status);
                const card = document.createElement('div');
                card.className = `kanban-card ${overdue ? 'card-overdue' : ''}`;
                card.draggable = true;
                card.ondragstart = (e) => e.dataTransfer.setData('text/plain', t.id);

                card.innerHTML = `
                    <div class="id">${t.id} ${overdue ? '<span class="badge-overdue"><i class="fa-solid fa-triangle-exclamation"></i> Quá hạn</span>' : ''}</div>
                    <div class="title">${t.name}</div>
                    <div class="meta">
                        <span><i class="fa-regular fa-calendar"></i> ${convertToDisplayDate(t.date)}</span>
                        <span style="font-weight:bold; color:${t.priority === 'Cao' ? '#ef4444' : '#64748b'};">${t.priority}</span>
                    </div>
                `;

                if (t.status === 'Chưa làm') { todoCol.appendChild(card); cTodo++; }
                else if (t.status === 'Đang làm') { doingCol.appendChild(card); cDoing++; }
                else if (t.status === 'Hoàn thành') { doneCol.appendChild(card); cDone++; }
            });

            document.getElementById('count-todo').innerText = cTodo;
            document.getElementById('count-doing').innerText = cDoing;
            document.getElementById('count-done').innerText = cDone;
        }

        function allowDrop(e) { e.preventDefault(); }
        function drop(e, status) {
            e.preventDefault();
            const taskId = e.dataTransfer.getData('text/plain');
            updateTaskStatus(taskId, status);
        }

        function renderKPITable() {
            const wrapper = document.getElementById('kpi-sections-wrapper');
            const targetNameEl = document.getElementById('kpi-target-name-display');
            const badgeEl = document.getElementById('kpi-type-badge-display');
            const savedTimeEl = document.getElementById('kpi-last-time-saved');

            if (targetNameEl) targetNameEl.innerText = kpiTargetUser;
            if (savedTimeEl) savedTimeEl.innerText = lastKpiTimestamp;

            const targetUserInfo = registeredUsers.find(u => u.username === kpiTargetUser);
            const userKpiType = (targetUserInfo && targetUserInfo.kpiType) ? targetUserInfo.kpiType : 'staff';

            if (badgeEl) {
                if (userKpiType === 'leader') {
                    badgeEl.innerText = 'Loại: Lãnh đạo'; badgeEl.className = 'kpi-badge-type type-leader';
                } else if (userKpiType === 'cleaner') {
                    badgeEl.innerText = 'Loại: Lao Công'; badgeEl.className = 'kpi-badge-type type-cleaner';
                } else {
                    badgeEl.innerText = 'Loại: Cán bộ / Nhân viên'; badgeEl.className = 'kpi-badge-type type-staff';
                }
            }

            if (!wrapper) return;
            wrapper.innerHTML = '';

            let grandTotalSelf = 0;
            let grandTotalAdmin = 0;
            let grandTotalMax = 0;

            const sections = ['A', 'B', 'C'];

            sections.forEach(secKey => {
                const secItems = kpiDataList.filter(x => x.section === secKey);
                if (secItems.length === 0 && !sectionTitles[secKey]) return;

                const secContainer = document.createElement('div');
                secContainer.style.marginBottom = '25px';

                const titleText = sectionTitles[secKey] || `MỤC ${secKey}`;
                const maxSecScore = sectionMaxScores[secKey] || 0;
                grandTotalMax += maxSecScore;

                let secSelfSum = 0;
                let secAdminSum = 0;

                secItems.forEach(sub => {
                    if (sub.items) {
                        sub.items.forEach(item => {
                            secSelfSum += parseFloatStrict(item.selfScore);
                            secAdminSum += parseFloatStrict(item.adminScore);
                        });
                    }
                });

                grandTotalSelf += secSelfSum;
                grandTotalAdmin += secAdminSum;

                secContainer.innerHTML = `
                    <div class="kpi-section-title">
                        <span>${secKey}. ${titleText} (Tối đa: ${maxSecScore} điểm)</span>
                        ${isAdmin ? `
                            <button class="btn-sm btn-edit" onclick="editSectionHeader('${secKey}')"><i class="fa-solid fa-pen"></i> Sửa tiêu đề/điểm</button>
                        ` : ''}
                    </div>
                    <div class="table-container" style="border-radius: 0 0 8px 8px;">
                        <table class="kpi-table">
                            <thead>
                                <tr>
                                    <th style="width: 50px; text-align: center;">STT</th>
                                    <th>Nội Dung Đánh Giá</th>
                                    <th style="width: 25%;">Tiêu chí đánh giá / Minh chứng</th>
                                    <th style="width: 90px; text-align: center;">Điểm tối đa</th>
                                    <th style="width: 100px; text-align: center;">Điểm tự chấm</th>
                                    <th style="width: 100px; text-align: center;">Điểm đánh giá</th>
                                    ${isAdmin ? '<th style="width: 110px; text-align: center;">Thao tác</th>' : ''}
                                </tr>
                            </thead>
                            <tbody id="kpi-tbody-${secKey}"></tbody>
                        </table>
                    </div>
                `;

                wrapper.appendChild(secContainer);

                const tbody = document.getElementById(`kpi-tbody-${secKey}`);

                secItems.forEach(sub => {
                    const trSub = document.createElement('tr');
                    trSub.className = 'row-sub-header';
                    trSub.innerHTML = `
                        <td style="text-align: center;"><strong>${sub.code}</strong></td>
                        <td colspan="${isAdmin ? 6 : 5}">
                            <strong>${sub.title}</strong> (Tối đa: ${sub.maxScore} điểm)
                            ${isAdmin ? `
                                <button class="btn-sm btn-edit" style="margin-left: 10px;" onclick="editSubSection('${sub.id}')"><i class="fa-solid fa-pen"></i> Sửa</button>
                                <button class="btn-sm btn-attach" onclick="addItemToSubSection('${sub.id}')"><i class="fa-solid fa-plus"></i> Thêm Tiêu Chí</button>
                                <button class="btn-sm btn-delete" onclick="deleteSubSection('${sub.id}')"><i class="fa-solid fa-trash"></i> Xóa Mục</button>
                            ` : ''}
                        </td>
                    `;
                    tbody.appendChild(trSub);

                    if (sub.items) {
                        sub.items.forEach((item, itemIdx) => {
                            const trItem = document.createElement('tr');
                            const canEditSelf = (currentUser === kpiTargetUser);
                            const canEditAdmin = isAdmin;

                            trItem.innerHTML = `
                                <td style="text-align: center; color: #64748b;">${itemIdx + 1}</td>
                                <td>${item.title}</td>
                                <td style="font-size: 13px; color: #475569;">${item.criteria || ''}</td>
                                <td style="text-align: center; font-weight: bold;">${item.maxScore}</td>
                                <td style="text-align: center;">
                                    <input type="number" step="0.1" max="${item.maxScore}" min="0" value="${item.selfScore}" 
                                        ${!canEditSelf ? 'disabled style="background:#f1f5f9;"' : ''} 
                                        onchange="updateKPIScore('${item.id}', 'selfScore', this.value, ${item.maxScore})" />
                                </td>
                                <td style="text-align: center;">
                                    <input type="number" step="0.1" max="${item.maxScore}" min="0" value="${item.adminScore}" 
                                        ${!canEditAdmin ? 'disabled style="background:#f1f5f9;"' : ''} 
                                        onchange="updateKPIScore('${item.id}', 'adminScore', this.value, ${item.maxScore})" />
                                </td>
                                ${isAdmin ? `
                                    <td style="text-align: center;">
                                        <button class="btn-sm btn-edit" onclick="editKPIItem('${sub.id}', '${item.id}')"><i class="fa-solid fa-pen"></i></button>
                                        <button class="btn-sm btn-delete" onclick="deleteKPIItem('${sub.id}', '${item.id}')"><i class="fa-solid fa-trash"></i></button>
                                    </td>
                                ` : ''}
                            `;
                            tbody.appendChild(trItem);
                        });
                    }
                });
            });

            document.getElementById('kpi-total-self').innerText = grandTotalSelf.toFixed(1);
            document.getElementById('kpi-total-admin').innerText = grandTotalAdmin.toFixed(1);
            document.getElementById('kpi-total-max').innerText = grandTotalMax.toFixed(1);

            updateRankingTable(grandTotalAdmin);
        }

        function updateRankingTable(score) {
            document.getElementById('rank-check-1').innerText = '';
            document.getElementById('rank-check-2').innerText = '';
            document.getElementById('rank-check-3').innerText = '';
            document.getElementById('rank-check-4').innerText = '';

            const badge = document.getElementById('kpi-ranking-result-display');
            if (!badge) return;

            if (score >= 90) {
                document.getElementById('rank-check-1').innerHTML = '<i class="fa-solid fa-circle-check" style="color: #16a34a; font-size: 18px;"></i>';
                badge.innerText = 'Hoàn thành xuất sắc nhiệm vụ';
                badge.className = 'ranking-result-badge rank-excel';
            } else if (score >= 75) {
                document.getElementById('rank-check-2').innerHTML = '<i class="fa-solid fa-circle-check" style="color: #2563eb; font-size: 18px;"></i>';
                badge.innerText = 'Hoàn thành tốt nhiệm vụ';
                badge.className = 'ranking-result-badge rank-good';
            } else if (score >= 50) {
                document.getElementById('rank-check-3').innerHTML = '<i class="fa-solid fa-circle-check" style="color: #d97706; font-size: 18px;"></i>';
                badge.innerText = 'Hoàn thành nhiệm vụ';
                badge.className = 'ranking-result-badge rank-fair';
            } else {
                document.getElementById('rank-check-4').innerHTML = '<i class="fa-solid fa-circle-check" style="color: #dc2626; font-size: 18px;"></i>';
                badge.innerText = 'Không hoàn thành nhiệm vụ';
                badge.className = 'ranking-result-badge rank-poor';
            }
        }

        function updateKPIScore(itemId, field, val, maxVal) {
            let num = parseFloatStrict(val);
            if (num > maxVal) {
                alert(`Điểm nhập vào (${num}) vượt quá điểm tối đa (${maxVal})! System tự động giới hạn về điểm tối đa.`);
                num = maxVal;
            }
            if (num < 0) num = 0;

            kpiDataList.forEach(sub => {
                if (sub.items) {
                    sub.items.forEach(item => {
                        if (item.id === itemId) {
                            item[field] = num;
                        }
                    });
                }
            });

            renderKPITable();
        }

        function saveKPIRatingByAdmin() {
            if (!isAdmin) return;
            const nowStr = new Date().toLocaleString('vi-VN');
            lastKpiTimestamp = nowStr;
            saveKPIRatingStorage(nowStr);
            alert(`[ADMIN] Đã lưu kết quả đánh giá KPI cho tài khoản "${kpiTargetUser}" tháng ${selectedKpiMonth} thành công!`);
            renderKPITable();
        }

        function saveKPIRatingByUser() {
            if (currentUser !== kpiTargetUser) {
                alert("Bạn chỉ có quyền lưu điểm tự chấm cho chính tài khoản của bạn!");
                return;
            }
            const nowStr = new Date().toLocaleString('vi-VN');
            lastKpiTimestamp = nowStr;
            saveKPIRatingStorage(nowStr);
            alert(`[CÁN BỘ] Đã lưu kết quả tự chấm điểm KPI tháng ${selectedKpiMonth} thành công!`);
            renderKPITable();
        }

        function editSectionHeader(secKey) {
            if (!isAdmin) return;
            const currentTitle = sectionTitles[secKey] || '';
            const currentMaxScore = sectionMaxScores[secKey] || 0;

            const newTitle = prompt(`Nhập tên tiêu đề cho Mục ${secKey}:`, currentTitle);
            if (newTitle === null) return;

            const newMaxScoreStr = prompt(`Nhập điểm tối đa cho Mục ${secKey}:`, currentMaxScore);
            if (newMaxScoreStr === null) return;

            const newMaxScore = parseFloatStrict(newMaxScoreStr);

            sectionTitles[secKey] = newTitle.trim();
            sectionMaxScores[secKey] = newMaxScore;

            const targetUserInfo = registeredUsers.find(u => u.username === kpiTargetUser);
            const userKpiType = (targetUserInfo && targetUserInfo.kpiType) ? targetUserInfo.kpiType : 'staff';
            
            db.ref(`kpiTemplates/${userKpiType}/sectionTitles/${secKey}`).set(newTitle.trim());
            db.ref(`kpiTemplates/${userKpiType}/sectionMaxScores/${secKey}`).set(newMaxScore);

            renderKPITable();
        }

        function addMainSection() {
            if (!isAdmin) return;
            const secKey = prompt("Nhập mã mục lớn (A, B, C, D...):", "C");
            if (!secKey) return;

            const code = prompt("Nhập ký hiệu la mã (I, II, III...):", "I");
            if (!code) return;

            const title = prompt("Nhập tên mục lớn mới:", "");
            if (!title) return;

            const maxScoreStr = prompt("Nhập điểm tối đa mục:", "10");
            const maxScore = parseFloatStrict(maxScoreStr);

            const newSub = {
                id: 'sub_' + Date.now(),
                section: secKey.toUpperCase(),
                code: code,
                title: title,
                maxScore: maxScore,
                items: []
            };

            kpiDataList.push(newSub);
            
            const targetUserInfo = registeredUsers.find(u => u.username === kpiTargetUser);
            const userKpiType = (targetUserInfo && targetUserInfo.kpiType) ? targetUserInfo.kpiType : 'staff';
            db.ref(`kpiTemplates/${userKpiType}/kpiDataList`).set(kpiDataList);

            renderKPITable();
        }

        function editSubSection(subId) {
            if (!isAdmin) return;
            const sub = kpiDataList.find(x => x.id === subId);
            if (!sub) return;

            const newTitle = prompt("Sửa tên nhóm công việc:", sub.title);
            if (newTitle === null) return;

            const newScoreStr = prompt("Sửa điểm tối đa nhóm:", sub.maxScore);
            if (newScoreStr === null) return;

            sub.title = newTitle.trim();
            sub.maxScore = parseFloatStrict(newScoreStr);

            renderKPITable();
        }

        function deleteSubSection(subId) {
            if (!isAdmin) return;
            if (confirm("Bạn có chắc chắn muốn xóa toàn bộ mục này và các tiêu chí con bên trong?")) {
                kpiDataList = kpiDataList.filter(x => x.id !== subId);
                renderKPITable();
            }
        }

        function addItemToSubSection(subId) {
            if (!isAdmin) return;
            const sub = kpiDataList.find(x => x.id === subId);
            if (!sub) return;

            const title = prompt("Nhập tên tiêu chí đánh giá mới:", "");
            if (!title) return;

            const criteria = prompt("Nhập tiêu chí/minh chứng yêu cầu (nếu có):", "");
            const maxScoreStr = prompt("Nhập điểm tối đa tiêu chí:", "5");
            const maxScore = parseFloatStrict(maxScoreStr);

            if (!sub.items) sub.items = [];
            sub.items.push({
                id: 'item_' + Date.now(),
                title: title.trim(),
                criteria: criteria ? criteria.trim() : '',
                maxScore: maxScore,
                selfScore: maxScore,
                adminScore: maxScore
            });

            renderKPITable();
        }

        function editKPIItem(subId, itemId) {
            if (!isAdmin) return;
            const sub = kpiDataList.find(x => x.id === subId);
            if (!sub || !sub.items) return;

            const item = sub.items.find(x => x.id === itemId);
            if (!item) return;

            const newTitle = prompt("Sửa tên tiêu chí:", item.title);
            if (newTitle === null) return;

            const newCriteria = prompt("Sửa minh chứng/tiêu chí:", item.criteria || '');
            if (newCriteria === null) return;

            const newMaxScoreStr = prompt("Sửa điểm tối đa:", item.maxScore);
            if (newMaxScoreStr === null) return;

            item.title = newTitle.trim();
            item.criteria = newCriteria.trim();
            item.maxScore = parseFloatStrict(newMaxScoreStr);

            if (item.selfScore > item.maxScore) item.selfScore = item.maxScore;
            if (item.adminScore > item.maxScore) item.adminScore = item.maxScore;

            renderKPITable();
        }

        function deleteKPIItem(subId, itemId) {
            if (!isAdmin) return;
            const sub = kpiDataList.find(x => x.id === subId);
            if (!sub || !sub.items) return;

            if (confirm("Xóa tiêu chí đánh giá này?")) {
                sub.items = sub.items.filter(x => x.id !== itemId);
                renderKPITable();
            }
        }

        function renderUserManagementTable() {
            const tbody = document.getElementById('user-management-body');
            if (!tbody) return;
            tbody.innerHTML = '';

            registeredUsers.forEach((u, idx) => {
                const tr = document.createElement('tr');
                const isRoot = (u.username === ADMIN_USERNAME);
                const kpiType = u.kpiType || 'staff';

                tr.innerHTML = `
                    <td style="text-align:center;"><strong>${idx + 1}</strong></td>
                    <td><strong>${u.username}</strong> ${isRoot ? '<span class="badge" style="background:#2563eb;">Admin Root</span>' : ''}</td>
                    <td>${u.email || 'N/A'}</td>
                    <td>
                        <select onchange="updateUserKPITypeFromTable('${u.username}', this.value)" style="padding:4px 8px; border-radius:4px; border:1px solid #cbd5e1; font-weight:bold; color:#1e40af;">
                            <option value="staff" ${kpiType === 'staff' ? 'selected' : ''}>Bảng Cán bộ / Nhân viên</option>
                            <option value="leader" ${kpiType === 'leader' ? 'selected' : ''}>Bảng Lãnh đạo</option>
                            <option value="cleaner" ${kpiType === 'cleaner' ? 'selected' : ''}>Bảng Lao Công</option>
                        </select>
                    </td>
                    <td>
                        ${!isRoot ? `<button class="btn-sm btn-delete" onclick="deleteUserAccount('${u.username}')"><i class="fa-solid fa-user-xmark"></i> Xóa tài khoản</button>` : '<span style="color:#94a3b8; font-size:12px;">Mặc định</span>'}
                    </td>
                `;
                tbody.appendChild(tr);
            });
        }

        function updateUserKPITypeFromTable(username, newType) {
            if (!isAdmin) return;
            db.ref(`users/${username}/kpiType`).set(newType, (err) => {
                if (!err) {
                    alert(`Đã cập nhật loại Bảng KPI cho tài khoản "${username}" thành: ${newType}`);
                    if (username === kpiTargetUser) listenRealtimeKPI();
                }
            });
        }

        function deleteUserAccount(username) {
            if (!isAdmin) return;
            if (confirm(`Bạn có chắc chắn muốn xóa tài khoản "${username}" khỏi hệ thống?`)) {
                db.ref(`users/${username}`).remove();
                db.ref(`tasks/${username}`).remove();
            }
        }

        /* --- HÀM XUẤT FILE WORD KPI THEO ĐÚNG MẪU YÊU CẦU --- */
        function exportKPIWord() {
            if (!window.docx) {
                alert("Thư viện xuất file Word chưa tải xong. Vui lòng thử lại sau giây lát!");
                return;
            }

            const { Document, Packer, Paragraph, TextRun, Table, TableRow, TableCell, AlignmentType, WidthType, BorderStyle } = window.docx;

            // Xử lý lấy Tháng / Năm từ Kỳ đánh giá
            let monthStr = "";
            let yearStr = "2026";
            if (selectedKpiMonth) {
                const parts = selectedKpiMonth.split('-');
                if (parts.length === 2) {
                    monthStr = parseInt(parts[1], 10).toString();
                    yearStr = parts[0];
                }
            }

            // Xác định Chức vụ dựa trên loại bảng KPI
            const targetUserInfo = registeredUsers.find(u => u.username === kpiTargetUser);
            const userKpiType = (targetUserInfo && targetUserInfo.kpiType) ? targetUserInfo.kpiType : 'staff';
            let positionText = "Cán bộ / Nhân viên";
            if (userKpiType === 'leader') positionText = "Lãnh đạo";
            if (userKpiType === 'cleaner') positionText = "Lao Công";

            // 1. Tạo Bảng Header (Trái: Đơn vị / Khoa, Phải: Quốc hiệu)
            const headerTable = new Table({
                width: { size: 100, type: WidthType.PERCENTAGE },
                borders: {
                    top: { style: BorderStyle.NONE },
                    bottom: { style: BorderStyle.NONE },
                    left: { style: BorderStyle.NONE },
                    right: { style: BorderStyle.NONE },
                    insideHorizontal: { style: BorderStyle.NONE },
                    insideVertical: { style: BorderStyle.NONE }
                },
                rows: [
                    new TableRow({
                        children: [
                            new TableCell({
                                children: [
                                    new Paragraph({
                                        children: [new TextRun({ text: "TRUNG TÂM KSBT BẮC NINH", font: "Times New Roman", size: 20 })],
                                        alignment: AlignmentType.CENTER
                                    }),
                                    new Paragraph({
                                        children: [new TextRun({ text: "KHOA HÓA LÝ", bold: true, underline: {}, font: "Times New Roman", size: 20 })],
                                        alignment: AlignmentType.CENTER
                                    })
                                ],
                                width: { size: 45, type: WidthType.PERCENTAGE }
                            }),
                            new TableCell({
                                children: [
                                    new Paragraph({
                                        children: [new TextRun({ text: "CỘNG HÒA XÃ HỘI CHỦ NGHĨA VIỆT NAM", bold: true, font: "Times New Roman", size: 20 })],
                                        alignment: AlignmentType.CENTER
                                    }),
                                    new Paragraph({
                                        children: [new TextRun({ text: "Độc lập - Tự do - Hạnh phúc", bold: true, underline: {}, font: "Times New Roman", size: 20 })],
                                        alignment: AlignmentType.CENTER
                                    })
                                ],
                                width: { size: 55, type: WidthType.PERCENTAGE }
                            })
                        ]
                    })
                ]
            });

            // 2. Tạo Bảng KPI (Chỉ gồm 4 cột: Nội Dung, Điểm tối đa, Điểm tự chấm, Điểm đánh giá)
            const kpiTableRows = [
                new TableRow({
                    children: [
                        new TableCell({
                            children: [new Paragraph({ children: [new TextRun({ text: "STT", bold: true, font: "Times New Roman", size: 20 })], alignment: AlignmentType.CENTER })],
                            width: { size: 8, type: WidthType.PERCENTAGE }
                        }),
                        new TableCell({
                            children: [new Paragraph({ children: [new TextRun({ text: "Nội Dung Đánh Giá", bold: true, font: "Times New Roman", size: 20 })], alignment: AlignmentType.CENTER })],
                            width: { size: 52, type: WidthType.PERCENTAGE }
                        }),
                        new TableCell({
                            children: [new Paragraph({ children: [new TextRun({ text: "Điểm tối đa", bold: true, font: "Times New Roman", size: 20 })], alignment: AlignmentType.CENTER })],
                            width: { size: 13, type: WidthType.PERCENTAGE }
                        }),
                        new TableCell({
                            children: [new Paragraph({ children: [new TextRun({ text: "Điểm tự chấm", bold: true, font: "Times New Roman", size: 20 })], alignment: AlignmentType.CENTER })],
                            width: { size: 13, type: WidthType.PERCENTAGE }
                        }),
                        new TableCell({
                            children: [new Paragraph({ children: [new TextRun({ text: "Điểm đánh giá", bold: true, font: "Times New Roman", size: 20 })], alignment: AlignmentType.CENTER })],
                            width: { size: 14, type: WidthType.PERCENTAGE }
                        })
                    ]
                })
            ];

            let grandTotalSelf = 0;
            let grandTotalAdmin = 0;
            let grandTotalMax = 0;

            const sections = ['A', 'B', 'C'];
            sections.forEach(secKey => {
                const secItems = kpiDataList.filter(x => x.section === secKey);
                if (secItems.length === 0 && !sectionTitles[secKey]) return;

                const maxSecScore = sectionMaxScores[secKey] || 0;
                grandTotalMax += maxSecScore;

                let secSelfSum = 0;
                let secAdminSum = 0;

                secItems.forEach(sub => {
                    if (sub.items) {
                        sub.items.forEach(item => {
                            secSelfSum += parseFloatStrict(item.selfScore);
                            secAdminSum += parseFloatStrict(item.adminScore);
                        });
                    }
                });

                grandTotalSelf += secSelfSum;
                grandTotalAdmin += secAdminSum;

                // Dòng tiêu đề mục lớn
                kpiTableRows.push(
                    new TableRow({
                        children: [
                            new TableCell({
                                children: [new Paragraph({ children: [new TextRun({ text: secKey, bold: true, font: "Times New Roman", size: 20 })], alignment: AlignmentType.CENTER })],
                                width: { size: 8, type: WidthType.PERCENTAGE }
                            }),
                            new TableCell({
                                children: [new Paragraph({ children: [new TextRun({ text: `${sectionTitles[secKey] || 'MỤC ' + secKey}`, bold: true, font: "Times New Roman", size: 20 })] })],
                                width: { size: 52, type: WidthType.PERCENTAGE }
                            }),
                            new TableCell({
                                children: [new Paragraph({ children: [new TextRun({ text: String(maxSecScore), bold: true, font: "Times New Roman", size: 20 })], alignment: AlignmentType.CENTER })],
                                width: { size: 13, type: WidthType.PERCENTAGE }
                            }),
                            new TableCell({
                                children: [new Paragraph({ children: [new TextRun({ text: secSelfSum.toFixed(1), bold: true, font: "Times New Roman", size: 20 })], alignment: AlignmentType.CENTER })],
                                width: { size: 13, type: WidthType.PERCENTAGE }
                            }),
                            new TableCell({
                                children: [new Paragraph({ children: [new TextRun({ text: secAdminSum.toFixed(1), bold: true, font: "Times New Roman", size: 20 })], alignment: AlignmentType.CENTER })],
                                width: { size: 14, type: WidthType.PERCENTAGE }
                            })
                        ]
                    })
                );

                secItems.forEach(sub => {
                    // Dòng nhóm công việc
                    kpiTableRows.push(
                        new TableRow({
                            children: [
                                new TableCell({
                                    children: [new Paragraph({ children: [new TextRun({ text: sub.code, bold: true, font: "Times New Roman", size: 20 })], alignment: AlignmentType.CENTER })],
                                    width: { size: 8, type: WidthType.PERCENTAGE }
                                }),
                                new TableCell({
                                    children: [new Paragraph({ children: [new TextRun({ text: sub.title, bold: true, font: "Times New Roman", size: 20 })] })],
                                    columnSpan: 4,
                                    width: { size: 92, type: WidthType.PERCENTAGE }
                                })
                            ]
                        })
                    );

                    // Danh sách tiêu chí con
                    if (sub.items) {
                        sub.items.forEach((item, itemIdx) => {
                            kpiTableRows.push(
                                new TableRow({
                                    children: [
                                        new TableCell({
                                            children: [new Paragraph({ children: [new TextRun({ text: String(itemIdx + 1), font: "Times New Roman", size: 20 })], alignment: AlignmentType.CENTER })],
                                            width: { size: 8, type: WidthType.PERCENTAGE }
                                        }),
                                        new TableCell({
                                            children: [
                                                new Paragraph({ children: [new TextRun({ text: item.title, font: "Times New Roman", size: 20 })] }),
                                                item.criteria ? new Paragraph({ children: [new TextRun({ text: `(Minh chứng: ${item.criteria})`, italics: true, font: "Times New Roman", size: 18 })] }) : new Paragraph({})
                                            ],
                                            width: { size: 52, type: WidthType.PERCENTAGE }
                                        }),
                                        new TableCell({
                                            children: [new Paragraph({ children: [new TextRun({ text: String(item.maxScore), font: "Times New Roman", size: 20 })], alignment: AlignmentType.CENTER })],
                                            width: { size: 13, type: WidthType.PERCENTAGE }
                                        }),
                                        new TableCell({
                                            children: [new Paragraph({ children: [new TextRun({ text: String(item.selfScore), font: "Times New Roman", size: 20 })], alignment: AlignmentType.CENTER })],
                                            width: { size: 13, type: WidthType.PERCENTAGE }
                                        }),
                                        new TableCell({
                                            children: [new Paragraph({ children: [new TextRun({ text: String(item.adminScore), font: "Times New Roman", size: 20 })], alignment: AlignmentType.CENTER })],
                                            width: { size: 14, type: WidthType.PERCENTAGE }
                                        })
                                    ]
                                })
                            );
                        });
                    }
                });
            });

            // Hàng Tổng Cộng
            kpiTableRows.push(
                new TableRow({
                    children: [
                        new TableCell({
                            children: [new Paragraph({ children: [new TextRun({ text: "TỔNG ĐIỂM", bold: true, font: "Times New Roman", size: 20 })], alignment: AlignmentType.CENTER })],
                            columnSpan: 2,
                            width: { size: 60, type: WidthType.PERCENTAGE }
                        }),
                        new TableCell({
                            children: [new Paragraph({ children: [new TextRun({ text: grandTotalMax.toFixed(1), bold: true, font: "Times New Roman", size: 20 })], alignment: AlignmentType.CENTER })],
                            width: { size: 13, type: WidthType.PERCENTAGE }
                        }),
                        new TableCell({
                            children: [new Paragraph({ children: [new TextRun({ text: grandTotalSelf.toFixed(1), bold: true, font: "Times New Roman", size: 20 })], alignment: AlignmentType.CENTER })],
                            width: { size: 13, type: WidthType.PERCENTAGE }
                        }),
                        new TableCell({
                            children: [new Paragraph({ children: [new TextRun({ text: grandTotalAdmin.toFixed(1), bold: true, font: "Times New Roman", size: 20 })], alignment: AlignmentType.CENTER })],
                            width: { size: 14, type: WidthType.PERCENTAGE }
                        })
                    ]
                })
            );

            // 3. Tính toán Bảng xếp loại
            let rankText = "Chưa xếp loại";
            let check1 = "", check2 = "", check3 = "", check4 = "";

            if (grandTotalAdmin >= 90) { check1 = "[X]"; rankText = "Hoàn thành xuất sắc nhiệm vụ"; }
            else if (grandTotalAdmin >= 75) { check2 = "[X]"; rankText = "Hoàn thành tốt nhiệm vụ"; }
            else if (grandTotalAdmin >= 50) { check3 = "[X]"; rankText = "Hoàn thành nhiệm vụ"; }
            else { check4 = "[X]"; rankText = "Không hoàn thành nhiệm vụ"; }

            const rankingTable = new Table({
                width: { size: 100, type: WidthType.PERCENTAGE },
                rows: [
                    new TableRow({
                        children: [
                            new TableCell({
                                children: [new Paragraph({ children: [new TextRun({ text: "STT", bold: true, font: "Times New Roman", size: 20 })], alignment: AlignmentType.CENTER })],
                                width: { size: 10, type: WidthType.PERCENTAGE }
                            }),
                            new TableCell({
                                children: [new Paragraph({ children: [new TextRun({ text: "Mức xếp loại", bold: true, font: "Times New Roman", size: 20 })], alignment: AlignmentType.CENTER })],
                                width: { size: 45, type: WidthType.PERCENTAGE }
                            }),
                            new TableCell({
                                children: [new Paragraph({ children: [new TextRun({ text: "Mức điểm", bold: true, font: "Times New Roman", size: 20 })], alignment: AlignmentType.CENTER })],
                                width: { size: 30, type: WidthType.PERCENTAGE }
                            }),
                            new TableCell({
                                children: [new Paragraph({ children: [new TextRun({ text: "Đạt được", bold: true, font: "Times New Roman", size: 20 })], alignment: AlignmentType.CENTER })],
                                width: { size: 15, type: WidthType.PERCENTAGE }
                            })
                        ]
                    }),
                    new TableRow({
                        children: [
                            new TableCell({ children: [new Paragraph({ children: [new TextRun({ text: "1", font: "Times New Roman", size: 20 })], alignment: AlignmentType.CENTER })] }),
                            new TableCell({ children: [new Paragraph({ children: [new TextRun({ text: "Hoàn thành xuất sắc nhiệm vụ", bold: true, font: "Times New Roman", size: 20 })] })] }),
                            new TableCell({ children: [new Paragraph({ children: [new TextRun({ text: "Từ 90 điểm trở lên", font: "Times New Roman", size: 20 })] })] }),
                            new TableCell({ children: [new Paragraph({ children: [new TextRun({ text: check1, bold: true, font: "Times New Roman", size: 20 })], alignment: AlignmentType.CENTER })] })
                        ]
                    }),
                    new TableRow({
                        children: [
                            new TableCell({ children: [new Paragraph({ children: [new TextRun({ text: "2", font: "Times New Roman", size: 20 })], alignment: AlignmentType.CENTER })] }),
                            new TableCell({ children: [new Paragraph({ children: [new TextRun({ text: "Hoàn thành tốt nhiệm vụ", bold: true, font: "Times New Roman", size: 20 })] })] }),
                            new TableCell({ children: [new Paragraph({ children: [new TextRun({ text: "Từ 75 đến dưới 90 điểm", font: "Times New Roman", size: 20 })] })] }),
                            new TableCell({ children: [new Paragraph({ children: [new TextRun({ text: check2, bold: true, font: "Times New Roman", size: 20 })], alignment: AlignmentType.CENTER })] })
                        ]
                    }),
                    new TableRow({
                        children: [
                            new TableCell({ children: [new Paragraph({ children: [new TextRun({ text: "3", font: "Times New Roman", size: 20 })], alignment: AlignmentType.CENTER })] }),
                            new TableCell({ children: [new Paragraph({ children: [new TextRun({ text: "Hoàn thành nhiệm vụ", bold: true, font: "Times New Roman", size: 20 })] })] }),
                            new TableCell({ children: [new Paragraph({ children: [new TextRun({ text: "Từ 50 đến dưới 75 điểm", font: "Times New Roman", size: 20 })] })] }),
                            new TableCell({ children: [new Paragraph({ children: [new TextRun({ text: check3, bold: true, font: "Times New Roman", size: 20 })], alignment: AlignmentType.CENTER })] })
                        ]
                    }),
                    new TableRow({
                        children: [
                            new TableCell({ children: [new Paragraph({ children: [new TextRun({ text: "4", font: "Times New Roman", size: 20 })], alignment: AlignmentType.CENTER })] }),
                            new TableCell({ children: [new Paragraph({ children: [new TextRun({ text: "Không hoàn thành nhiệm vụ", bold: true, font: "Times New Roman", size: 20 })] })] }),
                            new TableCell({ children: [new Paragraph({ children: [new TextRun({ text: "Dưới 50 điểm", font: "Times New Roman", size: 20 })] })] }),
                            new TableCell({ children: [new Paragraph({ children: [new TextRun({ text: check4, bold: true, font: "Times New Roman", size: 20 })], alignment: AlignmentType.CENTER })] })
                        ]
                    })
                ]
            });

            // 4. Bảng Chữ ký & Ngày tháng lệch phải
            const footerTable = new Table({
                width: { size: 100, type: WidthType.PERCENTAGE },
                borders: {
                    top: { style: BorderStyle.NONE },
                    bottom: { style: BorderStyle.NONE },
                    left: { style: BorderStyle.NONE },
                    right: { style: BorderStyle.NONE },
                    insideHorizontal: { style: BorderStyle.NONE },
                    insideVertical: { style: BorderStyle.NONE }
                },
                rows: [
                    new TableRow({
                        children: [
                            new TableCell({
                                children: [new Paragraph({})],
                                width: { size: 40, type: WidthType.PERCENTAGE }
                            }),
                            new TableCell({
                                children: [
                                    new Paragraph({
                                        children: [new TextRun({ text: "Bắc Ninh, ngày....tháng....năm 2026", italics: true, font: "Times New Roman", size: 20 })],
                                        alignment: AlignmentType.CENTER
                                    }),
                                    new Paragraph({
                                        children: [new TextRun({ text: "XÁC NHẬN CỦA LÃNH ĐẠO KHOA, TRƯỞNG KHOA", bold: true, font: "Times New Roman", size: 20 })],
                                        alignment: AlignmentType.CENTER,
                                        spacing: { before: 100 }
                                    })
                                ],
                                width: { size: 60, type: WidthType.PERCENTAGE }
                            })
                        ]
                    })
                ]
            });

            // 5. Tạo Document hoàn chỉnh
            const doc = new Document({
                sections: [{
                    properties: {},
                    children: [
                        headerTable,
                        new Paragraph({ spacing: { after: 200 } }),
                        new Paragraph({
                            children: [new TextRun({ text: `Họ và tên: ${kpiTargetUser.toUpperCase()}`, bold: true, font: "Times New Roman", size: 22 })],
                            spacing: { after: 100 }
                        }),
                        new Paragraph({
                            children: [new TextRun({ text: `Chức vụ: ${positionText}`, bold: true, font: "Times New Roman", size: 22 })],
                            spacing: { after: 300 }
                        }),
                        new Paragraph({
                            children: [new TextRun({ text: "PHIẾU THEO DÕI ĐÁNH GIÁ VIÊN CHỨC", bold: true, font: "Times New Roman", size: 26 })],
                            alignment: AlignmentType.CENTER
                        }),
                        new Paragraph({
                            children: [new TextRun({ text: `(Kỳ đánh giá tháng ${monthStr} năm ${yearStr})`, italics: true, font: "Times New Roman", size: 22 })],
                            alignment: AlignmentType.CENTER,
                            spacing: { after: 300 }
                        }),
                        new Table({
                            rows: kpiTableRows,
                            width: { size: 100, type: WidthType.PERCENTAGE }
                        }),
                        new Paragraph({ spacing: { after: 300 } }),
                        new Paragraph({
                            children: [new TextRun({ text: "BẢNG XẾP LOẠI", bold: true, font: "Times New Roman", size: 22 })],
                            spacing: { after: 150 }
                        }),
                        rankingTable,
                        new Paragraph({
                            children: [
                                new TextRun({ text: "Kết quả xếp loại: ", bold: true, font: "Times New Roman", size: 22 }),
                                new TextRun({ text: rankText, bold: true, italics: true, font: "Times New Roman", size: 22 })
                            ],
                            spacing: { before: 200, after: 400 }
                        }),
                        footerTable
                    ]
                }]
            });

            Packer.toBlob(doc).then(blob => {
                saveAs(blob, `Phieu_Danh_Gia_KPI_${kpiTargetUser}_Thang_${monthStr}_${yearStr}.docx`);
            });
        }
    </script>
</body>
</html>
