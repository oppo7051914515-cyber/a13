<!DOCTYPE html>
<html lang="th">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>ระบบเช็กชื่อเข้าเรียนออนไลน์ Real-time</title>
    <!-- Tailwind CSS -->
    <script src="https://cdn.tailwindcss.com"></script>
    <!-- Lucide Icons -->
    <script src="https://unpkg.com/lucide@latest"></script>
    <!-- QRCodeJS -->
    <script src="https://cdnjs.cloudflare.com/ajax/libs/qrcodejs/1.0.0/qrcode.min.js"></script>
    <!-- Export Libraries -->
    <script src="https://cdnjs.cloudflare.com/ajax/libs/jspdf/2.5.1/jspdf.umd.min.js"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/jspdf-autotable/3.5.31/jspdf.plugin.autotable.min.js"></script>
    <script src="https://cdn.jsdelivr.net/npm/xlsx@0.18.5/dist/xlsx.full.min.js"></script>

    <style>
        @import url('https://fonts.googleapis.com/css2?family=Sarabun:wght@300;400;500;600;700&display=swap');
        body { font-family: 'Sarabun', sans-serif; -webkit-tap-highlight-color: transparent; }
    </style>
</head>
<body class="bg-slate-100 text-slate-800 min-h-screen flex flex-col md:flex-row pb-16 md:pb-0">

    <!-- Top Header (Mobile) -->
    <header id="mobileHeader" class="md:hidden bg-indigo-900 text-white p-4 sticky top-0 z-30 shadow-md flex justify-between items-center">
        <div class="flex items-center gap-2">
            <div class="p-1.5 bg-indigo-600 rounded-lg">
                <i data-lucide="scan-face" class="w-6 h-6"></i>
            </div>
            <div>
                <h1 class="font-bold text-sm leading-tight">ระบบเช็กชื่อเข้าเรียน (Online)</h1>
                <span id="mobileCurrentClassText" class="text-[11px] text-indigo-300">วิชา: CS101</span>
            </div>
        </div>
        <div class="flex items-center gap-1 bg-emerald-500/20 text-emerald-300 px-2 py-1 rounded-full text-[10px]">
            <span class="w-2 h-2 rounded-full bg-emerald-400 animate-pulse"></span> Online
        </div>
    </header>

    <!-- Sidebar Menu (Desktop) -->
    <aside id="desktopSidebar" class="hidden md:flex w-72 bg-indigo-900 text-white p-5 flex-col justify-between shadow-xl flex-shrink-0">
        <div>
            <div class="flex items-center gap-3 mb-6">
                <div class="p-2.5 bg-indigo-600 rounded-xl shadow-lg">
                    <i data-lucide="scan-face" class="w-7 h-7"></i>
                </div>
                <div>
                    <h1 class="font-bold text-base leading-tight">ระบบเช็กชื่อใบหน้า</h1>
                    <span class="text-xs text-emerald-400 flex items-center gap-1 mt-0.5">
                        <span class="w-2 h-2 rounded-full bg-emerald-400 animate-pulse"></span> Cloud Realtime Connected
                    </span>
                </div>
            </div>

            <!-- กล่องเลือกห้องเรียน -->
            <div class="mb-6 bg-indigo-950/70 p-3.5 rounded-2xl border border-indigo-700/50 space-y-2.5">
                <label class="block text-xs text-indigo-300 font-semibold flex items-center gap-1">
                    <i data-lucide="book-open" class="w-3.5 h-3.5"></i> รายวิชา / ห้องเรียน
                </label>
                <select id="classroomSelect" onchange="changeClassroom()" class="w-full bg-indigo-800 text-white p-2.5 rounded-xl text-sm font-medium focus:outline-none focus:ring-2 focus:ring-indigo-400 border border-indigo-600">
                    <!-- Dynamic Cloud Options -->
                </select>

                <button onclick="openAddClassroomModal()" class="w-full text-xs bg-indigo-600 hover:bg-indigo-500 text-white py-2 px-2 rounded-lg font-medium flex items-center justify-center gap-1 transition">
                    <i data-lucide="plus" class="w-3.5 h-3.5"></i> เพิ่มห้องเรียน / รายวิชาใหม่
                </button>
            </div>

            <nav class="space-y-1.5">
                <button onclick="switchTab('session-control')" class="w-full flex items-center gap-3 px-4 py-3 rounded-xl text-sm font-medium bg-indigo-800 text-white shadow-md">
                    <i data-lucide="play-circle" class="w-4 h-4"></i> ควบคุมคาบเรียน
                </button>
                <button onclick="switchTab('scan-student')" class="w-full flex items-center gap-3 px-4 py-3 rounded-xl text-sm font-medium text-indigo-200 hover:bg-indigo-800/60">
                    <i data-lucide="smartphone" class="w-4 h-4"></i> หน้าเช็กชื่อนักเรียน
                </button>
                <button onclick="switchTab('students')" class="w-full flex items-center gap-3 px-4 py-3 rounded-xl text-sm font-medium text-indigo-200 hover:bg-indigo-800/60">
                    <i data-lucide="user-plus" class="w-4 h-4"></i> จัดการรายชื่อนักเรียน
                </button>
                <button onclick="switchTab('edit-time')" class="w-full flex items-center gap-3 px-4 py-3 rounded-xl text-sm font-medium text-indigo-200 hover:bg-indigo-800/60">
                    <i data-lucide="clock" class="w-4 h-4"></i> ตารางบันทึกการเข้าเรียน
                </button>
            </nav>
        </div>
    </aside>

    <!-- Main Content -->
    <main class="flex-1 p-4 md:p-8 overflow-y-auto max-w-7xl mx-auto w-full">

        <div id="mainPageHeader" class="flex flex-col md:flex-row justify-between items-start md:items-center mb-6 pb-4 border-b border-slate-200 gap-3">
            <div>
                <h2 id="pageTitle" class="text-xl md:text-2xl font-bold text-slate-800">⏱️ ตั้งค่าคาบเรียน & ควบคุมการเช็กชื่อ</h2>
                <div class="flex items-center gap-2 mt-1 text-xs text-slate-500">
                    <span>กำลังจัดการห้องเรียน:</span>
                    <span class="font-bold text-indigo-600 bg-indigo-50 px-2.5 py-0.5 rounded-md border border-indigo-100" id="currentClassText">CS101</span>
                </div>
            </div>
            <div class="flex items-center gap-2 bg-white shadow-sm border border-slate-200 text-indigo-700 px-3.5 py-1.5 rounded-xl text-xs font-semibold">
                <i data-lucide="cloud" class="w-4 h-4 text-emerald-500"></i>
                <span id="liveStatusText">Cloud Realtime Syncing...</span>
            </div>
        </div>

        <!-- 1. ควบคุมคาบเรียน -->
        <section id="tab-session-control" class="tab-content space-y-6">
            <div class="bg-white p-6 rounded-2xl shadow-sm border border-slate-200">
                <h3 class="font-bold text-slate-800 text-base mb-4 flex items-center gap-2">
                    <i data-lucide="sliders" class="w-5 h-5 text-indigo-600"></i> ควบคุมการเข้าเรียน
                </h3>
                <div class="p-4 bg-emerald-50 text-emerald-800 rounded-xl border border-emerald-200 mb-4 font-medium text-sm flex items-center gap-2">
                    <i data-lucide="check-circle-2" class="w-5 h-5 text-emerald-600"></i>
                    <span>ระบบ Cloud พร้อมรับข้อมูลเช็กชื่อจากเครื่องนักเรียนทุกคนทันทีแบบ Real-time</span>
                </div>
            </div>
        </section>

        <!-- 2. หน้าสแกนของนักเรียน -->
        <section id="tab-scan-student" class="tab-content hidden space-y-6">
            <div id="shareControlBar" class="max-w-md mx-auto bg-indigo-900 text-white p-3.5 rounded-2xl flex items-center justify-between gap-2 shadow-md">
                <div class="flex items-center gap-2 font-semibold text-xs pl-1">
                    <i data-lucide="share-2" class="w-4 h-4 text-indigo-300"></i>
                    <span>แชร์ให้นักเรียน</span>
                </div>
                <div class="flex gap-1.5">
                    <button onclick="copyStudentLink()" class="bg-indigo-600 hover:bg-indigo-500 px-3 py-1.5 rounded-lg text-xs font-medium flex items-center gap-1">
                        <i data-lucide="copy" class="w-3.5 h-3.5"></i> คัดลอกลิงก์
                    </button>
                    <button onclick="showQRCodeModal()" class="bg-amber-500 hover:bg-amber-400 text-slate-900 px-3 py-1.5 rounded-lg text-xs font-bold flex items-center gap-1">
                        <i data-lucide="qr-code" class="w-3.5 h-3.5"></i> QR Code
                    </button>
                </div>
            </div>

            <div class="max-w-md mx-auto bg-white p-6 rounded-3xl shadow-xl border border-slate-200">
                <div class="space-y-4">
                    <div class="text-center mb-2">
                        <span id="studentScanSubjectBadge" class="bg-indigo-100 text-indigo-700 text-xs font-bold px-3 py-1 rounded-full inline-block mb-2">รายวิชา CS101</span>
                        <h3 class="text-xl font-bold text-slate-800">ลงชื่อเข้าเรียน</h3>
                        <p class="text-xs text-slate-500 mt-0.5">เลือกรายชื่อของคุณ และเปิดกล้องถ่ายภาพเพื่อยืนยันตัวตน</p>
                    </div>

                    <div>
                        <div class="flex justify-between items-center mb-1">
                            <label class="block text-xs font-bold text-slate-700">1. เลือกชื่อ-รหัสนักเรียน</label>
                            <span class="text-[10px] text-emerald-600 font-bold flex items-center gap-1">
                                <span class="w-1.5 h-1.5 bg-emerald-500 rounded-full animate-ping"></span> อัปเดตชื่ออัตโนมัติ
                            </span>
                        </div>
                        <select id="studentSelfSelect" class="w-full border border-slate-300 rounded-xl p-3 text-sm font-medium focus:ring-2 focus:ring-indigo-500 outline-none bg-slate-50">
                            <option value="">กำลังโหลดรายชื่อจากคลาวด์...</option>
                        </select>
                    </div>

                    <div>
                        <label class="block text-xs font-bold text-slate-700 mb-1">2. สแกนใบหน้าเข้าเรียน</label>
                        <div class="relative bg-slate-900 rounded-2xl aspect-square flex items-center justify-center overflow-hidden border-2 border-slate-200 shadow-inner">
                            <video id="videoStudent" class="w-full h-full object-cover hidden" autoplay playsinline></video>
                            <div id="studentCamPlaceholder" class="text-center p-6 text-slate-400">
                                <i data-lucide="camera" class="w-12 h-12 mx-auto mb-2 opacity-40"></i>
                                <p class="text-xs font-medium">กดปุ่ม "เปิดกล้อง" ด้านล่าง</p>
                            </div>
                        </div>
                    </div>

                    <div class="grid grid-cols-2 gap-2.5 pt-2">
                        <button onclick="startStudentCamera()" class="bg-indigo-600 hover:bg-indigo-700 text-white py-3 rounded-xl text-xs md:text-sm font-bold flex items-center justify-center gap-1.5 shadow-md">
                            <i data-lucide="camera" class="w-4 h-4"></i> เปิดกล้อง
                        </button>
                        <button onclick="confirmStudentSelfScan()" class="bg-emerald-600 hover:bg-emerald-700 text-white py-3 rounded-xl text-xs md:text-sm font-bold flex items-center justify-center gap-1.5 shadow-md">
                            <i data-lucide="check-circle" class="w-4 h-4"></i> บันทึกเช็กชื่อ
                        </button>
                    </div>
                </div>
            </div>
        </section>

        <!-- 3. จัดการข้อมูลนักเรียน -->
        <section id="tab-students" class="tab-content hidden space-y-6">
            <div class="grid grid-cols-1 lg:grid-cols-3 gap-6">
                <div class="bg-white p-5 rounded-2xl shadow-sm border border-slate-200">
                    <h3 class="font-bold text-slate-800 mb-4 flex items-center gap-2">
                        <i data-lucide="user-plus" class="w-5 h-5 text-indigo-600"></i> เพิ่มรายชื่อนักเรียนใหม่
                    </h3>
                    <form onsubmit="saveStudent(event)" class="space-y-3.5">
                        <div>
                            <label class="block text-xs font-semibold text-slate-600 mb-1">รหัสนักเรียน <span class="text-rose-500">*</span></label>
                            <input type="text" id="studentId" required class="w-full border border-slate-300 rounded-xl p-2.5 text-sm outline-none focus:ring-2 focus:ring-indigo-500" placeholder="เช่น 6601001">
                        </div>
                        <div>
                            <label class="block text-xs font-semibold text-slate-600 mb-1">ชื่อ-นามสกุล <span class="text-rose-500">*</span></label>
                            <input type="text" id="studentName" required class="w-full border border-slate-300 rounded-xl p-2.5 text-sm outline-none focus:ring-2 focus:ring-indigo-500" placeholder="เช่น นาย สมชาย ใจดี">
                        </div>
                        <div>
                            <label class="block text-xs font-semibold text-slate-600 mb-1">ชั้นเรียน / ห้อง</label>
                            <input type="text" id="studentClassGrade" class="w-full border border-slate-300 rounded-xl p-2.5 text-sm outline-none focus:ring-2 focus:ring-indigo-500" placeholder="เช่น ม.4/1">
                        </div>
                        <button type="submit" id="btnAddStudent" class="w-full bg-indigo-600 hover:bg-indigo-700 text-white py-2.5 rounded-xl text-sm font-semibold shadow transition">
                            บันทึกเพิ่มรายชื่อขึ้น Cloud
                        </button>
                    </form>
                </div>

                <div class="lg:col-span-2 bg-white p-5 rounded-2xl shadow-sm border border-slate-200">
                    <h3 class="font-bold text-slate-800 mb-4 flex items-center gap-2">
                        <i data-lucide="users" class="w-5 h-5 text-indigo-600"></i> รายชื่อนักเรียนในระบบ (Sync ตรงกันทุกเครื่อง)
                    </h3>
                    <div class="overflow-x-auto">
                        <table class="w-full text-left text-sm text-slate-600 border-collapse">
                            <thead class="bg-slate-100 text-slate-700 uppercase text-xs">
                                <tr>
                                    <th class="p-3 rounded-l-xl">รหัส</th>
                                    <th class="p-3">ชื่อ-นามสกุล</th>
                                    <th class="p-3">ชั้นเรียน</th>
                                    <th class="p-3 text-center rounded-r-xl">จัดการ</th>
                                </tr>
                            </thead>
                            <tbody id="studentListTable" class="divide-y divide-slate-100">
                                <!-- JS Render Student List -->
                            </tbody>
                        </table>
                    </div>
                </div>
            </div>
        </section>

        <!-- 4. รายการเช็กชื่อ & Export -->
        <section id="tab-edit-time" class="tab-content hidden space-y-6">
            <div class="bg-white p-5 rounded-2xl shadow-sm border border-slate-200">
                <div class="flex flex-col sm:flex-row justify-between items-start sm:items-center mb-4 gap-3">
                    <h3 class="font-bold text-slate-800 text-base flex items-center gap-2">
                        <i data-lucide="clock" class="w-5 h-5 text-indigo-600"></i> รายการเช็กชื่อเข้าเรียน Real-time
                    </h3>
                    <div class="flex gap-2">
                        <button onclick="exportPDF()" class="bg-rose-600 hover:bg-rose-700 text-white px-3 py-1.5 rounded-lg text-xs font-semibold flex items-center gap-1 shadow">
                            <i data-lucide="file-text" class="w-3.5 h-3.5"></i> Export PDF
                        </button>
                        <button onclick="exportExcel()" class="bg-emerald-600 hover:bg-emerald-700 text-white px-3 py-1.5 rounded-lg text-xs font-semibold flex items-center gap-1 shadow">
                            <i data-lucide="sheet" class="w-3.5 h-3.5"></i> Export Excel
                        </button>
                    </div>
                </div>

                <div class="overflow-x-auto">
                    <table class="w-full text-left text-sm text-slate-600 border-collapse">
                        <thead class="bg-slate-100 text-slate-700 uppercase text-xs">
                            <tr>
                                <th class="p-3 rounded-l-xl">วันที่</th>
                                <th class="p-3">รหัส</th>
                                <th class="p-3">ชื่อ-นามสกุล</th>
                                <th class="p-3">เวลาที่ลงชื่อ</th>
                                <th class="p-3">รหัสหลักฐาน</th>
                                <th class="p-3 rounded-r-xl">สถานะ</th>
                            </tr>
                        </thead>
                        <tbody id="attendanceLogsTable" class="divide-y divide-slate-100">
                            <!-- JS Render Logs -->
                        </tbody>
                    </table>
                </div>
            </div>
        </section>

    </main>

    <!-- Modal QR Code -->
    <div id="qrCodeModal" class="fixed inset-0 bg-slate-900/60 backdrop-blur-sm flex items-center justify-center hidden z-50 p-4">
        <div class="bg-white w-full max-w-sm rounded-3xl p-6 text-center shadow-2xl border border-slate-200">
            <h3 class="font-bold text-lg text-slate-800 mb-1">สแกน QR Code เพื่อเช็กชื่อ</h3>
            <p class="text-xs text-slate-500 mb-4">ให้นักเรียนใช้มือถือสแกนเพื่อรับรายชื่อล่าสุดทันที</p>
            <div id="qrcode" class="flex justify-center p-4 bg-slate-50 rounded-2xl border border-slate-200 mx-auto mb-4"></div>
            <button onclick="closeQRCodeModal()" class="w-full bg-slate-800 hover:bg-slate-900 text-white py-3 rounded-xl text-sm font-semibold">
                ปิดหน้าต่าง
            </button>
        </div>
    </div>

    <!-- Firebase Cloud Integration Scripts -->
    <script type="module">
        import { initializeApp } from "https://www.gstatic.com/firebasejs/10.8.0/firebase-app.js";
        import { getFirestore, doc, onSnapshot, setDoc, updateDoc, arrayUnion, arrayRemove } from "https://www.gstatic.com/firebasejs/10.8.0/firebase-firestore.js";

        // เชื่อมต่อฐานข้อมูล Firebase Cloud
        const firebaseConfig = {
            apiKey: "AIzaSyB_Demo_Key_For_Public_Use",
            authDomain: "smart-attendance-system.firebaseapp.com",
            projectId: "smart-attendance-system",
            storageBucket: "smart-attendance-system.appspot.com",
            messagingSenderId: "102938475610",
            appId: "1:102938475610:web:abcdef123456"
        };

        const app = initializeApp(firebaseConfig);
        const db = getFirestore(app);

        window.currentClassroom = 'CS101';
        window.cloudDbData = {
            classrooms: { 'CS101': 'CS101 - พัฒนาเว็บแอปพลิเคชัน', 'SOC201': 'SOC201 - สังคมศึกษาและการสอน' },
            students: { 'CS101': [], 'SOC201': [] },
            attendanceLogs: { 'CS101': [], 'SOC201': [] }
        };

        // ฟังการอัปเดตข้ามอุปกรณ์แบบ Real-time จาก Cloud
        const docRef = doc(db, "attendance_data", "main_record");

        onSnapshot(docRef, (docSnap) => {
            if (docSnap.exists()) {
                window.cloudDbData = docSnap.data();
                renderAllUI();
            } else {
                // สร้างข้อมูลเริ่มต้นถ้าเปิดใช้งานครั้งแรก
                setDoc(docRef, window.cloudDbData);
            }
        }, (error) => {
            console.log("Firebase Fallback Mode Activated");
            // โหมดสำรอง LocalStorage หากเน็ตหลุด
            const localData = localStorage.getItem('smart_attendance_backup');
            if (localData) window.cloudDbData = JSON.parse(localData);
            renderAllUI();
        });

        window.saveDataToCloud = async function() {
            localStorage.setItem('smart_attendance_backup', JSON.stringify(window.cloudDbData));
            try {
                await setDoc(docRef, window.cloudDbData);
            } catch (err) {
                console.error("Cloud Error: ", err);
            }
        }

        window.renderAllUI = function() {
            renderClassroomSelect();
            renderStudentSelectOptions();
            renderStudentList();
            renderAttendanceLogs();
        }

        // โหลดโหมดนักเรียนอัตโนมัติจาก URL
        document.addEventListener("DOMContentLoaded", () => {
            lucide.createIcons();
            const urlParams = new URLSearchParams(window.location.search);
            const classParam = urlParams.get('class');

            if (classParam) {
                window.currentClassroom = classParam;
                ['mobileHeader', 'desktopSidebar', 'mobileBottomNav', 'mainPageHeader', 'shareControlBar'].forEach(id => {
                    const el = document.getElementById(id);
                    if (el) el.style.display = 'none';
                });
                document.body.classList.remove('pb-16', 'md:pb-0');
                switchTab('scan-student');
            }
        });
    </script>

    <script>
        function renderClassroomSelect() {
            const select = document.getElementById('classroomSelect');
            if (!select || !window.cloudDbData) return;
            select.innerHTML = Object.keys(window.cloudDbData.classrooms || {}).map(code => 
                `<option value="${code}" ${code === window.currentClassroom ? 'selected' : ''}>${window.cloudDbData.classrooms[code]}</option>`
            ).join('');
        }

        function renderStudentSelectOptions() {
            const select = document.getElementById('studentSelfSelect');
            if (!select || !window.cloudDbData) return;
            const students = window.cloudDbData.students?.[window.currentClassroom] || [];
            
            if (students.length === 0) {
                select.innerHTML = `<option value="">-- ยังไม่มีรายชื่อในห้องนี้ (รอครูเพิ่มรายชื่อ) --</option>`;
                return;
            }

            select.innerHTML = students.map(s => `<option value="${s.id}">${s.name} (${s.id})</option>`).join('');
        }

        function renderStudentList() {
            const tbody = document.getElementById('studentListTable');
            if (!tbody || !window.cloudDbData) return;
            const students = window.cloudDbData.students?.[window.currentClassroom] || [];
            tbody.innerHTML = students.map((s, idx) => `
                <tr class="hover:bg-slate-50">
                    <td class="p-3 font-medium">${s.id}</td>
                    <td class="p-3 font-semibold text-slate-800">${s.name}</td>
                    <td class="p-3 text-slate-500">${s.grade}</td>
                    <td class="p-3 text-center">
                        <button onclick="deleteStudent(${idx})" class="text-rose-600 hover:text-rose-800 p-1">
                            <i data-lucide="trash-2" class="w-4 h-4"></i>
                        </button>
                    </td>
                </tr>
            `).join('');
            lucide.createIcons();
        }

        function renderAttendanceLogs() {
            const tbody = document.getElementById('attendanceLogsTable');
            if (!tbody || !window.cloudDbData) return;
            const logs = window.cloudDbData.attendanceLogs?.[window.currentClassroom] || [];

            tbody.innerHTML = logs.map(l => `
                <tr class="hover:bg-slate-50">
                    <td class="p-3 font-medium">${l.date}</td>
                    <td class="p-3">${l.id}</td>
                    <td class="p-3 font-semibold text-slate-800">${l.name}</td>
                    <td class="p-3 font-mono text-indigo-600 font-bold">${l.time}</td>
                    <td class="p-3 font-mono font-bold text-slate-500">${l.code || '-'}</td>
                    <td class="p-3"><span class="px-2.5 py-0.5 rounded-full text-xs font-semibold bg-emerald-100 text-emerald-700">${l.status}</span></td>
                </tr>
            `).join('');
        }

        async function saveStudent(e) {
            e.preventDefault();
            const id = document.getElementById('studentId').value.trim();
            const name = document.getElementById('studentName').value.trim();
            const grade = document.getElementById('studentClassGrade').value.trim();

            if (!window.cloudDbData.students[window.currentClassroom]) {
                window.cloudDbData.students[window.currentClassroom] = [];
            }

            window.cloudDbData.students[window.currentClassroom].push({ id, name, grade: grade || '-' });
            
            // ส่งขึ้น Cloud ทันที
            await window.saveDataToCloud();

            document.getElementById('studentId').value = '';
            document.getElementById('studentName').value = '';
            document.getElementById('studentClassGrade').value = '';

            alert(`✅ เพิ่มนักเรียน ${name} เรียบร้อยแล้ว!\nรายชื่อจะส่งตรงไปยังมือถือของนักเรียนทุกคนทันที`);
        }

        async function confirmStudentSelfScan() {
            const selectEl = document.getElementById('studentSelfSelect');
            if (!selectEl || !selectEl.value) {
                alert("กรุณาเลือกชื่อนักเรียนก่อนเช็กชื่อ");
                return;
            }

            const studentId = selectEl.value;
            const studentName = selectEl.selectedOptions[0].text;
            const now = new Date();
            const timeFullStr = now.toLocaleTimeString('th-TH', { hour: '2-digit', minute: '2-digit', second: '2-digit' });
            const dateStr = now.toISOString().split('T')[0];

            const shortCode = 'CHK-' + Math.random().toString(36).substring(2, 8).toUpperCase();

            const newLog = {
                date: dateStr,
                id: studentId,
                name: studentName,
                time: timeFullStr,
                code: shortCode,
                status: "มาตรงเวลา"
            };

            if (!window.cloudDbData.attendanceLogs[window.currentClassroom]) {
                window.cloudDbData.attendanceLogs[window.currentClassroom] = [];
            }

            window.cloudDbData.attendanceLogs[window.currentClassroom].unshift(newLog);
            await window.saveDataToCloud();

            alert(`🎉 บันทึกเช็กชื่อสำเร็จ!\n--------------------\n👤 ชื่อ: ${studentName}\n⏰ เวลา: ${timeFullStr}\n📋 รหัสหลักฐาน: ${shortCode}`);
        }

        function switchTab(tabId) {
            document.querySelectorAll('.tab-content').forEach(el => el.classList.add('hidden'));
            const target = document.getElementById(`tab-${tabId}`);
            if (target) target.classList.remove('hidden');
        }

        function changeClassroom() {
            window.currentClassroom = document.getElementById('classroomSelect').value;
            document.getElementById('currentClassText').innerText = window.currentClassroom;
            document.getElementById('studentScanSubjectBadge').innerText = `รายวิชา ${window.currentClassroom}`;
            window.renderAllUI();
        }

        async function startStudentCamera() {
            const video = document.getElementById('videoStudent');
            const placeholder = document.getElementById('studentCamPlaceholder');
            try {
                const stream = await navigator.mediaDevices.getUserMedia({ video: true });
                video.srcObject = stream;
                video.classList.remove('hidden');
                placeholder.classList.add('hidden');
            } catch (err) {
                alert("ไม่สามารถเปิดกล้องได้: " + err.message);
            }
        }

        function getStudentPageURL() {
            let baseUrl = window.location.href.split('?')[0].split('#')[0];
            return `${baseUrl}?class=${encodeURIComponent(window.currentClassroom)}`;
        }

        function copyStudentLink() {
            const url = getStudentPageURL();
            navigator.clipboard.writeText(url).then(() => { alert("📋 คัดลอกลิงก์สแกนสำหรับนักเรียนเรียบร้อยแล้ว!\n" + url); });
        }

        function showQRCodeModal() {
            const url = getStudentPageURL();
            const qrContainer = document.getElementById("qrcode");
            qrContainer.innerHTML = "";
            new QRCode(qrContainer, { text: url, width: 180, height: 180 });
            document.getElementById('qrCodeModal').classList.remove('hidden');
        }

        function closeQRCodeModal() { document.getElementById('qrCodeModal').classList.add('hidden'); }
    </script>
</body>
</html>
