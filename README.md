<html lang="zh-TW">
<head>
    <meta charset="UTF-8">
    <!-- 優化手機顯示：禁止縮放、適配瀏海螢幕 -->
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no, viewport-fit=cover">
    <title>忠義國小 躲避球隊點名表</title>
    
    <!-- 引入 Tailwind CSS -->
    <script src="https://cdn.tailwindcss.com"></script>
    
    <!-- 設定 Tailwind 自定義顏色與字體 -->
    <script>
        tailwind.config = {
            theme: {
                extend: {
                    colors: {
                        stone: {
                            50: '#fafaf9',
                            100: '#f5f5f4',
                            200: '#e7e5e4',
                            300: '#d6d3d1',
                            400: '#a8a29e',
                            500: '#78716c',
                            600: '#57534e',
                            700: '#44403c',
                            800: '#292524',
                            900: '#1c1917',
                        }
                    },
                    fontFamily: {
                        sans: ['-apple-system', 'BlinkMacSystemFont', '"Segoe UI"', 'Roboto', '"Helvetica Neue"', 'Arial', '"Noto Sans"', 'sans-serif'],
                        mono: ['ui-monospace', 'SFMono-Regular', 'Menlo', 'Monaco', 'Consolas', 'monospace'],
                    }
                }
            }
        }
    </script>

    <!-- 自定義 CSS 動畫與捲軸隱藏 -->
    <style>
        @keyframes fadeIn {
            from { opacity: 0; }
            to { opacity: 1; }
        }
        @keyframes fadeInDown {
            from { opacity: 0; transform: translateY(-10px); }
            to { opacity: 1; transform: translateY(0); }
        }
        .animate-fade-in { animation: fadeIn 0.3s ease-out; }
        .animate-fade-in-down { animation: fadeInDown 0.3s ease-out; }
        
        /* 隱藏捲軸但保留功能 */
        .no-scrollbar::-webkit-scrollbar { display: none; }
        .no-scrollbar { -ms-overflow-style: none; scrollbar-width: none; }
        
        /* 防止 iOS 點擊高亮 */
        * { -webkit-tap-highlight-color: transparent; }
    </style>

    <!-- 引入 Babel 用於解析 JSX -->
    <script src="https://unpkg.com/@babel/standalone/babel.min.js"></script>

    <!-- Import Map: 告訴瀏覽器哪裡載入 React 和 Lucide 圖示 -->
    <script type="importmap">
    {
        "imports": {
            "react": "https://esm.sh/react@18.2.0",
            "react-dom/client": "https://esm.sh/react-dom@18.2.0/client",
            "lucide-react": "https://esm.sh/lucide-react@0.292.0"
        }
    }
    </script>
</head>
<body class="bg-[#f5f5f0]">
    <div id="root"></div>

    <!-- React 應用程式邏輯 -->
    <script type="text/babel" data-type="module">
        import React, { useState, useEffect, useRef } from 'react';
        import { createRoot } from 'react-dom/client';
        import { Calendar, UserPlus, Trash2, Users, CheckCircle, Edit3, Save, AlertCircle, Download, Upload, History, X, ChevronRight } from 'lucide-react';

        const App = () => {
            // --- 狀態管理 ---
            const getToday = () => new Date().toISOString().split('T')[0];
            const getFirstDayOfMonth = () => {
                const date = new Date();
                return new Date(date.getFullYear(), date.getMonth(), 1).toISOString().split('T')[0];
            };
            
            const [currentDate, setCurrentDate] = useState(getToday());
            const [students, setStudents] = useState([]);
            const [attendance, setAttendance] = useState({}); 
            const [isEditMode, setIsEditMode] = useState(false);
            
            // 報表相關狀態
            const [showReport, setShowReport] = useState(false);
            const [reportStartDate, setReportStartDate] = useState(getFirstDayOfMonth());
            const [reportEndDate, setReportEndDate] = useState(getToday());

            // 新增學生表單狀態
            const [newStudentName, setNewStudentName] = useState("");
            const [newStudentGender, setNewStudentGender] = useState("M");
            const [newStudentClass, setNewStudentClass] = useState("三年一班");
            
            const [genderFilter, setGenderFilter] = useState("ALL"); 

            // 對話框狀態
            const [modal, setModal] = useState({
                isOpen: false,
                type: 'info',
                message: '',
                onConfirm: null
            });

            const fileInputRef = useRef(null);

            // --- 產生班級選項 (三到六年級，每級三班) ---
            const classOptions = [];
            const grades = ['三', '四', '五', '六'];
            const classes = ['一', '二', '三'];
            
            grades.forEach(g => {
                classes.forEach(c => {
                    classOptions.push(`${g}年${c}班`);
                });
            });

            // --- 初始化與儲存 (LocalStorage) ---
            useEffect(() => {
                const savedStudents = localStorage.getItem('zy_dodgeball_students');
                const savedAttendance = localStorage.getItem('zy_dodgeball_attendance');

                if (savedStudents) {
                    let parsedStudents = JSON.parse(savedStudents);
                    if (parsedStudents.length > 0 && typeof parsedStudents[0] === 'string') {
                        parsedStudents = parsedStudents.map(name => ({ name, gender: 'M', class: '未分類' }));
                        showInfo("系統已更新！舊名單已轉換格式，請記得補上性別與班級。");
                    } else if (parsedStudents.length > 0 && !parsedStudents[0].class) {
                        parsedStudents = parsedStudents.map(s => ({ ...s, class: s.class || '未分類' }));
                    }
                    setStudents(parsedStudents);
                } else {
                    setStudents([
                        { name: "陳小明", gender: "M", class: "五年二班" },
                        { name: "林大華", gender: "M", class: "六年一班" },
                        { name: "張志豪", gender: "M", class: "四年三班" },
                        { name: "王曉美", gender: "F", class: "五年二班" },
                        { name: "李強", gender: "M", class: "三年一班" },
                        { name: "林小玉", gender: "F", class: "六年三班" }
                    ]);
                }

                if (savedAttendance) {
                    setAttendance(JSON.parse(savedAttendance));
                }
            }, []);

            useEffect(() => {
                localStorage.setItem('zy_dodgeball_students', JSON.stringify(students));
            }, [students]);

            useEffect(() => {
                localStorage.setItem('zy_dodgeball_attendance', JSON.stringify(attendance));
            }, [attendance]);

            // --- 輔助功能 ---
            const showInfo = (message) => {
                setModal({ isOpen: true, type: 'info', message, onConfirm: () => setModal(prev => ({ ...prev, isOpen: false })) });
            };

            const showConfirm = (message, onConfirmAction) => {
                setModal({
                    isOpen: true, 
                    type: 'confirm', 
                    message, 
                    onConfirm: () => {
                        onConfirmAction();
                        setModal(prev => ({ ...prev, isOpen: false }));
                    }
                });
            };

            const closeModal = () => setModal(prev => ({ ...prev, isOpen: false }));

            const getWeekDay = (dateStr) => {
                if (!dateStr) return '';
                const [y, m, d] = dateStr.split('-').map(Number);
                const date = new Date(y, m - 1, d);
                const days = ['週日', '週一', '週二', '週三', '週四', '週五', '週六'];
                return days[date.getDay()];
            };

            // --- 匯入匯出功能 ---
            const exportToCSV = () => {
                const allDates = Object.keys(attendance).sort();
                let csvContent = "\uFEFF"; 
                csvContent += `班級,姓名,性別,${allDates.join(",")}\n`;

                const sortedStudents = [...students].sort((a, b) => a.class.localeCompare(b.class));

                sortedStudents.forEach(student => {
                    const row = [
                        student.class || "未分類",
                        student.name,
                        student.gender === 'M' ? '男' : '女'
                    ];
                    allDates.forEach(date => {
                        const isPresent = attendance[date]?.includes(student.name);
                        row.push(isPresent ? "O" : "");
                    });
                    csvContent += row.join(",") + "\n";
                });

                const blob = new Blob([csvContent], { type: "text/csv;charset=utf-8;" });
                const url = URL.createObjectURL(blob);
                const link = document.createElement("a");
                link.setAttribute("href", url);
                link.setAttribute("download", `忠義國小躲避球點名表_${currentDate}.csv`);
                document.body.appendChild(link);
                link.click();
                document.body.removeChild(link);
            };

            const handleImportClick = () => fileInputRef.current.click();

            const handleFileUpload = (e) => {
                const file = e.target.files[0];
                if (!file) return;

                const reader = new FileReader();
                reader.onload = (event) => {
                    const text = event.target.result;
                    const rows = text.split('\n');
                    const newStudents = [];
                    let importCount = 0;

                    rows.forEach((row, index) => {
                        if (index === 0 && (row.includes("姓名") || row.includes("Name"))) return;
                        const cols = row.split(',').map(c => c.trim());
                        if (cols.length < 2) return;

                        let name, genderStr, className;
                        if (cols[0].includes('班')) {
                            className = cols[0]; name = cols[1]; genderStr = cols[2];
                        } else {
                            name = cols[0]; genderStr = cols[1]; className = cols[2]; 
                        }

                        if (!name) return;
                        let gender = 'M';
                        if (genderStr) {
                            const g = genderStr.toUpperCase();
                            if (g === 'F' || g === '女' || g === 'GIRL') gender = 'F';
                        }
                        if (!className) className = "未分類";

                        if (!students.some(s => s.name === name) && !newStudents.some(s => s.name === name)) {
                            newStudents.push({ name, gender, class: className });
                            importCount++;
                        }
                    });

                    if (importCount > 0) {
                        setStudents(prev => [...prev, ...newStudents]);
                        showInfo(`成功匯入 ${importCount} 位新隊員！`);
                    } else {
                        showInfo("未發現新隊員。請確保 CSV 格式為：班級, 姓名, 性別");
                    }
                };
                reader.readAsText(file);
                e.target.value = '';
            };

            // --- 核心操作 ---
            const toggleAttendance = (studentName) => {
                if (isEditMode) return; 
                setAttendance(prev => {
                    const currentList = prev[currentDate] || [];
                    const isPresent = currentList.includes(studentName);
                    let newList = isPresent ? currentList.filter(name => name !== studentName) : [...currentList, studentName];
                    return { ...prev, [currentDate]: newList };
                });
            };

            const addStudent = (e) => {
                e.preventDefault();
                if (!newStudentName.trim()) return;
                if (students.some(s => s.name === newStudentName.trim())) {
                    showInfo("這位隊員已經在名單內囉！");
                    return;
                }
                setStudents([...students, { name: newStudentName.trim(), gender: newStudentGender, class: newStudentClass }]);
                setNewStudentName("");
            };

            const removeStudent = (nameToRemove) => {
                showConfirm(`確定要將「${nameToRemove}」從球隊名單中移除嗎？`, () => {
                    setStudents(prev => prev.filter(s => s.name !== nameToRemove));
                });
            };

            const toggleStudentGender = (student) => {
                const newGender = student.gender === 'M' ? 'F' : 'M';
                setStudents(prev => prev.map(s => s.name === student.name ? { ...s, gender: newGender } : s));
            };

            // --- 顯示運算 ---
            const filteredStudents = students.filter(s => genderFilter === 'ALL' || s.gender === genderFilter);
            const sortedAndFilteredStudents = filteredStudents.sort((a, b) => {
                if (a.class === b.class) return a.name.localeCompare(b.name);
                return (a.class || "").localeCompare(b.class || "");
            });

            const presentList = attendance[currentDate] || [];
            const displayTotalCount = sortedAndFilteredStudents.length;
            const displayPresentCount = sortedAndFilteredStudents.filter(s => presentList.includes(s.name)).length;
            const displayAbsentCount = displayTotalCount - displayPresentCount;

            // --- 報表運算 ---
            const getReportData = () => {
                const allRecordedDates = Object.keys(attendance).sort();
                const rangeDates = allRecordedDates.filter(date => date >= reportStartDate && date <= reportEndDate);
                const reportStudents = [...students].sort((a, b) => a.class.localeCompare(b.class));
                return { rangeDates, reportStudents };
            };

            const colors = { bg: "bg-[#f5f5f0]", textMain: "text-[#333333]", accent: "bg-[#7f1d1d]" };

            return (
                <div className={`min-h-[100dvh] ${colors.bg} font-sans selection:bg-stone-300 selection:text-stone-800 pb-[env(safe-area-inset-bottom)] relative`}>
                    <input type="file" ref={fileInputRef} onChange={handleFileUpload} accept=".csv" className="hidden" />

                    {/* 對話框 */}
                    {modal.isOpen && (
                        <div className="fixed inset-0 z-[60] flex items-center justify-center px-4 bg-stone-800/40 backdrop-blur-sm animate-fade-in">
                            <div className="bg-white rounded-xl shadow-xl max-w-sm w-full p-6 border border-[#dcdcdc] transform transition-all scale-100">
                                <div className="flex flex-col items-center text-center gap-4">
                                    <div className="p-3 bg-stone-100 rounded-full text-stone-600"><AlertCircle size={32} /></div>
                                    <h3 className="text-lg font-bold text-[#333333]">提示</h3>
                                    <p className="text-stone-600 text-base">{modal.message}</p>
                                    <div className="flex gap-3 w-full mt-2">
                                        {modal.type === 'confirm' && (
                                            <button onClick={closeModal} className="flex-1 py-3 px-4 rounded-lg border border-[#dcdcdc] text-stone-600 hover:bg-stone-50 active:bg-stone-100 touch-manipulation">取消</button>
                                        )}
                                        <button onClick={modal.onConfirm || closeModal} className={`flex-1 py-3 px-4 rounded-lg text-white shadow-sm active:opacity-90 touch-manipulation ${modal.type === 'confirm' ? 'bg-[#7f1d1d]' : 'bg-stone-700'}`}>
                                            {modal.type === 'confirm' ? '確認' : '知道了'}
                                        </button>
                                    </div>
                                </div>
                            </div>
                        </div>
                    )}

                    {/* 報表全螢幕 Modal */}
                    {showReport && (
                        <div className="fixed inset-0 z-[55] bg-[#f5f5f0] flex flex-col animate-fade-in">
                            {/* 報表 Header */}
                            <div className="bg-white border-b border-[#dcdcdc] px-4 py-3 pt-[max(0.75rem,env(safe-area-inset-top))] flex items-center justify-between shadow-sm">
                                <h2 className="text-lg font-bold text-[#333333] flex items-center gap-2">
                                    <History size={20} className="text-[#7f1d1d]" />
                                    <span>出席紀錄表</span>
                                </h2>
                                <button onClick={() => setShowReport(false)} className="p-2 bg-stone-100 rounded-full text-stone-500 hover:bg-stone-200">
                                    <X size={20} />
                                </button>
                            </div>

                            {/* 報表控制區 (日期選擇) - 修復遮擋問題 */}
                            <div className="p-4 bg-white border-b border-[#dcdcdc]">
                                <div className="flex flex-col gap-3">
                                    <div className="grid grid-cols-[1fr,auto,1fr] gap-2 items-center">
                                        <div className="flex flex-col">
                                            <label className="text-xs text-stone-400 mb-1">開始日期</label>
                                            {/* 使用 Flex 佈局替代 Absolute 定位，防止遮擋 */}
                                            <div className="flex items-center border border-[#dcdcdc] rounded-lg overflow-hidden bg-stone-50">
                                                <input 
                                                    type="date" 
                                                    value={reportStartDate} 
                                                    onChange={(e) => setReportStartDate(e.target.value)} 
                                                    className="flex-1 bg-transparent p-2 text-sm outline-none text-stone-700 min-w-0" 
                                                />
                                                <div className="bg-stone-100 text-[#7f1d1d] text-xs font-bold px-3 py-2 border-l border-[#dcdcdc] shrink-0">
                                                    {getWeekDay(reportStartDate)}
                                                </div>
                                            </div>
                                        </div>
                                        <div className="text-stone-300 pt-5"><ChevronRight size={20} /></div>
                                        <div className="flex flex-col">
                                            <label className="text-xs text-stone-400 mb-1">結束日期</label>
                                            <div className="flex items-center border border-[#dcdcdc] rounded-lg overflow-hidden bg-stone-50">
                                                <input 
                                                    type="date" 
                                                    value={reportEndDate} 
                                                    onChange={(e) => setReportEndDate(e.target.value)} 
                                                    className="flex-1 bg-transparent p-2 text-sm outline-none text-stone-700 min-w-0" 
                                                />
                                                <div className="bg-stone-100 text-[#7f1d1d] text-xs font-bold px-3 py-2 border-l border-[#dcdcdc] shrink-0">
                                                    {getWeekDay(reportEndDate)}
                                                </div>
                                            </div>
                                        </div>
                                    </div>
                                </div>
                            </div>

                            {/* 報表內容 (表格) */}
                            <div className="flex-1 overflow-auto p-4">
                                {(() => {
                                    const { rangeDates, reportStudents } = getReportData();
                                    
                                    if (rangeDates.length === 0) {
                                        return (
                                            <div className="h-full flex flex-col items-center justify-center text-stone-400">
                                                <Calendar size={48} className="mb-4 opacity-50" />
                                                <p>所選區間沒有點名紀錄</p>
                                            </div>
                                        );
                                    }

                                    return (
                                        <div className="bg-white rounded-lg border border-[#dcdcdc] shadow-sm overflow-hidden">
                                            <div className="overflow-x-auto">
                                                <table className="w-full text-sm border-collapse">
                                                    <thead>
                                                        <tr>
                                                            <th className="sticky left-0 z-10 bg-stone-100 border-b border-r border-[#dcdcdc] p-3 text-left min-w-[100px] font-bold text-stone-700">姓名</th>
                                                            <th className="bg-stone-50 border-b border-r border-[#dcdcdc] p-3 min-w-[80px] text-center font-normal text-stone-500">出席率</th>
                                                            {rangeDates.map(date => (
                                                                <th key={date} className="bg-stone-50 border-b border-[#dcdcdc] p-2 min-w-[60px] text-center font-normal text-stone-500 border-r last:border-r-0">
                                                                    <div className="text-[10px]">{date.slice(5)}</div>
                                                                    <div className="font-bold text-[#333333]">{getWeekDay(date)}</div>
                                                                </th>
                                                            ))}
                                                        </tr>
                                                    </thead>
                                                    <tbody>
                                                        {reportStudents.map((student, idx) => {
                                                            let presentCount = 0;
                                                            rangeDates.forEach(date => {
                                                                if (attendance[date]?.includes(student.name)) presentCount++;
                                                            });
                                                            const rate = Math.round((presentCount / rangeDates.length) * 100);

                                                            return (
                                                                <tr key={student.name} className={idx % 2 === 0 ? 'bg-white' : 'bg-[#fafaf9]'}>
                                                                    <td className="sticky left-0 z-10 bg-inherit border-b border-r border-[#dcdcdc] p-3 font-medium text-stone-800 shadow-[2px_0_5px_-2px_rgba(0,0,0,0.05)]">
                                                                        <div className="flex flex-col">
                                                                            <span>{student.name}</span>
                                                                            <span className="text-[10px] text-stone-400 font-normal">{student.class}</span>
                                                                        </div>
                                                                    </td>
                                                                    <td className="border-b border-r border-[#dcdcdc] p-2 text-center">
                                                                        <span className={`px-2 py-0.5 rounded-full text-xs ${rate >= 80 ? 'bg-green-100 text-green-700' : rate < 50 ? 'bg-red-50 text-red-600' : 'bg-stone-100 text-stone-600'}`}>
                                                                            {rate}%
                                                                        </span>
                                                                    </td>
                                                                    {rangeDates.map(date => {
                                                                        const isPresent = attendance[date]?.includes(student.name);
                                                                        return (
                                                                            <td key={date} className="border-b border-[#dcdcdc] border-r last:border-r-0 p-2 text-center">
                                                                                {isPresent ? (
                                                                                    <CheckCircle size={16} className="mx-auto text-[#7f1d1d]" />
                                                                                ) : (
                                                                                    <span className="text-stone-200 text-xl leading-none">·</span>
                                                                                )}
                                                                            </td>
                                                                        );
                                                                    })}
                                                                </tr>
                                                            );
                                                        })}
                                                    </tbody>
                                                </table>
                                            </div>
                                        </div>
                                    );
                                })()}
                            </div>
                        </div>
                    )}

                    {/* 頂部 Header */}
                    <header className="sticky top-0 z-50 bg-[#f5f5f0]/95 backdrop-blur-sm border-b border-[#dcdcdc] px-4 py-3 shadow-sm pt-[max(0.75rem,env(safe-area-inset-top))]">
                        <div className="max-w-5xl mx-auto flex items-center justify-between gap-2">
                            <div className="flex-1 min-w-0">
                                <h1 className={`text-xl font-bold tracking-wide ${colors.textMain} flex items-center gap-2 truncate`}>
                                    <span className="bg-[#7f1d1d] text-white p-0.5 px-2 text-xs font-light rounded-sm shrink-0">忠義</span>
                                    <span className="truncate">點名表</span>
                                </h1>
                            </div>
                            
                            {/* 頂部控制區：日期選擇 + 星期幾顯示 (修復版面) */}
                            <div className="flex items-center bg-white rounded-md border border-[#dcdcdc] shadow-sm shrink-0 overflow-hidden">
                                <div className="pl-2 pr-1 text-stone-500 flex items-center justify-center">
                                    <Calendar size={18} />
                                </div>
                                <input 
                                    type="date" 
                                    value={currentDate} 
                                    onChange={(e) => setCurrentDate(e.target.value)} 
                                    className="outline-none bg-transparent text-stone-700 py-1.5 px-1 text-base w-[125px] text-center border-none focus:ring-0" 
                                />
                                <div className="bg-stone-100 text-[#7f1d1d] text-xs font-bold px-2 py-2 border-l border-[#dcdcdc] flex items-center justify-center min-w-[3rem]">
                                    {getWeekDay(currentDate)}
                                </div>
                            </div>
                        </div>
                    </header>

                    {/* 主內容 */}
                    <main className="max-w-5xl mx-auto px-4 py-4 pb-32">
                        {/* 統計與按鈕 */}
                        <div className="bg-white p-4 rounded-xl shadow-sm border border-[#dcdcdc] mb-4">
                            <div className="flex flex-col gap-4">
                                <div className="grid grid-cols-3 gap-2 divide-x divide-stone-100">
                                    <div className="text-center px-2">
                                        <p className="text-[10px] text-stone-400 uppercase tracking-wider mb-1">應到</p>
                                        <p className="text-2xl font-light text-stone-800">{displayTotalCount}</p>
                                    </div>
                                    <div className="text-center px-2">
                                        <p className="text-[10px] text-stone-400 uppercase tracking-wider mb-1">實到</p>
                                        <p className="text-2xl font-bold text-[#7f1d1d]">{displayPresentCount}</p>
                                    </div>
                                    <div className="text-center px-2">
                                        <p className="text-[10px] text-stone-400 uppercase tracking-wider mb-1">缺席</p>
                                        <p className="text-2xl font-light text-stone-400">{displayAbsentCount}</p>
                                    </div>
                                </div>
                                <div className="h-px bg-stone-100 w-full"></div>
                                <div className="flex gap-2 overflow-x-auto pb-1 no-scrollbar">
                                    {/* 紀錄檢視按鈕 */}
                                    <button onClick={() => setShowReport(true)} className="flex items-center justify-center gap-1.5 px-3 py-2.5 text-sm border border-[#dcdcdc] rounded-lg text-stone-600 active:bg-stone-50 touch-manipulation whitespace-nowrap min-w-fit bg-stone-50">
                                        <History size={16} /> <span>紀錄</span>
                                    </button>

                                    <button onClick={exportToCSV} className="flex-1 flex items-center justify-center gap-1.5 px-3 py-2.5 text-sm border border-[#dcdcdc] rounded-lg text-stone-600 active:bg-stone-50 touch-manipulation whitespace-nowrap min-w-fit">
                                        <Download size={16} /> <span className="hidden sm:inline">匯出</span>
                                    </button>
                                    {isEditMode && (
                                        <button onClick={handleImportClick} className="flex-1 flex items-center justify-center gap-1.5 px-3 py-2.5 text-sm border border-[#dcdcdc] rounded-lg text-stone-600 active:bg-stone-50 touch-manipulation whitespace-nowrap min-w-fit">
                                            <Upload size={16} /> <span className="hidden sm:inline">匯入</span>
                                        </button>
                                    )}
                                    <button onClick={() => setIsEditMode(!isEditMode)} className={`flex-[2] flex items-center justify-center gap-2 px-4 py-2.5 text-sm rounded-lg transition-colors whitespace-nowrap min-w-fit touch-manipulation font-medium shadow-sm ${isEditMode ? 'bg-stone-800 text-white' : 'bg-stone-100 text-stone-600 active:bg-stone-200'}`}>
                                        {isEditMode ? <Save size={16} /> : <Edit3 size={16} />}
                                        {isEditMode ? "完成編輯" : "管理名單"}
                                    </button>
                                </div>
                            </div>
                        </div>

                        {/* 篩選 Tabs */}
                        <div className="flex justify-center mb-6 sticky top-[3.5rem] z-40 py-2 -mx-4 px-4 bg-[#f5f5f0]/90 backdrop-blur-sm">
                            <div className="inline-flex bg-[#e5e5e5] p-1 rounded-xl shadow-inner w-full md:w-auto">
                                <button onClick={() => setGenderFilter('ALL')} className={`flex-1 md:flex-none px-4 py-2.5 rounded-lg text-sm font-medium transition-all touch-manipulation ${genderFilter === 'ALL' ? 'bg-white text-stone-800 shadow-sm scale-[1.02]' : 'text-stone-500 active:text-stone-700'}`}>全部</button>
                                <button onClick={() => setGenderFilter('M')} className={`flex-1 md:flex-none px-4 py-2.5 rounded-lg text-sm font-medium transition-all touch-manipulation ${genderFilter === 'M' ? 'bg-white text-blue-800 shadow-sm scale-[1.02]' : 'text-stone-500 active:text-stone-700'}`}>男生</button>
                                <button onClick={() => setGenderFilter('F')} className={`flex-1 md:flex-none px-4 py-2.5 rounded-lg text-sm font-medium transition-all touch-manipulation ${genderFilter === 'F' ? 'bg-white text-red-800 shadow-sm scale-[1.02]' : 'text-stone-500 active:text-stone-700'}`}>女生</button>
                            </div>
                        </div>

                        {/* 新增表單 */}
                        {isEditMode && (
                            <form onSubmit={addStudent} className="mb-6 p-4 bg-white rounded-xl shadow-sm border border-stone-200 flex flex-col gap-3 animate-fade-in-down">
                                <div className="flex gap-3">
                                    <div className="flex-1 relative border border-[#dcdcdc] rounded-lg bg-stone-50">
                                        <select value={newStudentClass} onChange={(e) => setNewStudentClass(e.target.value)} className="w-full h-full p-3 bg-transparent outline-none text-stone-800 text-base appearance-none rounded-lg">
                                            {classOptions.map(opt => <option key={opt} value={opt}>{opt}</option>)}
                                        </select>
                                        <div className="absolute right-3 top-1/2 -translate-y-1/2 pointer-events-none text-stone-400">
                                            <svg width="10" height="6" viewBox="0 0 10 6" fill="none" stroke="currentColor" strokeWidth="2" strokeLinecap="round" strokeLinejoin="round"><path d="M1 1L5 5L9 1"/></svg>
                                        </div>
                                    </div>
                                    <div className="w-24 relative border border-[#dcdcdc] rounded-lg bg-stone-50">
                                        <select value={newStudentGender} onChange={(e) => setNewStudentGender(e.target.value)} className="w-full h-full p-3 bg-transparent outline-none text-stone-800 text-base appearance-none rounded-lg">
                                            <option value="M">男生</option>
                                            <option value="F">女生</option>
                                        </select>
                                        <div className="absolute right-3 top-1/2 -translate-y-1/2 pointer-events-none text-stone-400">
                                            <svg width="10" height="6" viewBox="0 0 10 6" fill="none" stroke="currentColor" strokeWidth="2" strokeLinecap="round" strokeLinejoin="round"><path d="M1 1L5 5L9 1"/></svg>
                                        </div>
                                    </div>
                                </div>
                                <input type="text" placeholder="輸入姓名..." value={newStudentName} onChange={(e) => setNewStudentName(e.target.value)} className="w-full p-3 rounded-lg border border-[#dcdcdc] focus:outline-none focus:border-stone-400 focus:ring-2 focus:ring-stone-100 bg-white text-base appearance-none" />
                                <button type="submit" className="w-full bg-stone-700 text-white p-3 rounded-lg active:bg-stone-800 transition-colors flex items-center justify-center gap-2 touch-manipulation mt-1">
                                    <UserPlus size={20} /> <span className="font-medium">新增隊員</span>
                                </button>
                            </form>
                        )}

                        {/* 名單列表 */}
                        {sortedAndFilteredStudents.length === 0 ? (
                            <div className="text-center py-20 text-stone-400 bg-white rounded-xl border border-dashed border-stone-300 mx-2">
                                <Users size={48} className="mx-auto mb-4 opacity-50" />
                                <p>目前沒有資料。</p>
                            </div>
                        ) : (
                            <div className="grid grid-cols-2 md:grid-cols-3 lg:grid-cols-4 gap-3">
                                {sortedAndFilteredStudents.map((student) => {
                                    const isPresent = attendance[currentDate]?.includes(student.name);
                                    const isMale = student.gender === 'M';
                                    
                                    return (
                                        <div key={student.name} onClick={() => toggleAttendance(student.name)} className={`
                                            relative group flex flex-col items-center justify-center p-3 rounded-xl border cursor-pointer transition-all duration-200 select-none h-32 touch-manipulation
                                            ${isEditMode ? 'cursor-default' : 'active:scale-95'}
                                            ${!isEditMode && isPresent ? 'bg-white border-[#7f1d1d] shadow-md ring-1 ring-[#7f1d1d]/20' : 'bg-white border-[#dcdcdc] active:bg-stone-50'}
                                        `}>
                                            <div className={`absolute top-2 left-3 text-[10px] font-bold opacity-40 ${isMale ? 'text-blue-800' : 'text-red-800'}`}>{isMale ? 'BOY' : 'GIRL'}</div>
                                            {!isEditMode && (
                                                <div className={`absolute top-2 right-2 transition-opacity duration-300 ${isPresent ? 'opacity-100' : 'opacity-10'}`}>
                                                    <CheckCircle size={20} className={isPresent ? 'text-[#7f1d1d] fill-[#7f1d1d] text-white' : 'text-stone-300'} />
                                                </div>
                                            )}
                                            <span className="text-[10px] text-stone-500 mb-1 mt-3 bg-stone-100 px-2 py-0.5 rounded-full truncate max-w-[90%]">{student.class || "未分類"}</span>
                                            <span className={`text-lg font-medium tracking-wide truncate max-w-full px-1 ${isPresent && !isEditMode ? 'text-[#7f1d1d] font-bold' : 'text-stone-700'}`}>{student.name}</span>
                                            {!isEditMode && (
                                                <span className={`text-[10px] mt-1.5 px-2 py-0.5 rounded-full border ${isPresent ? 'bg-[#7f1d1d] text-white border-[#7f1d1d]' : 'bg-stone-50 text-stone-400 border-stone-200'}`}>
                                                    {isPresent ? "已到" : "未到"}
                                                </span>
                                            )}
                                            {isEditMode && (
                                                <div className="absolute inset-0 bg-white/90 backdrop-blur-[1px] flex items-center justify-center gap-4 rounded-xl border-2 border-dashed border-stone-300 z-10">
                                                    <button onClick={(e) => { e.stopPropagation(); toggleStudentGender(student); }} className={`p-3 rounded-full text-xs font-bold border shadow-sm active:scale-90 transition-transform ${isMale ? 'bg-blue-50 text-blue-600 border-blue-200' : 'bg-red-50 text-red-600 border-red-200'}`} title="切換性別">{isMale ? "男" : "女"}</button>
                                                    <button onClick={(e) => { e.stopPropagation(); removeStudent(student.name); }} className="p-3 bg-white border border-stone-200 text-stone-500 hover:text-red-600 active:bg-red-50 active:border-red-200 rounded-full shadow-sm active:scale-90 transition-all" title="刪除"><Trash2 size={18} /></button>
                                                </div>
                                            )}
                                        </div>
                                    );
                                })}
                            </div>
                        )}
                    </main>
                    <footer className="text-center text-stone-400 text-xs py-6 pb-[calc(1.5rem+env(safe-area-inset-bottom))]">© {new Date().getFullYear()} 忠義國小 躲避球隊</footer>
                </div>
            );
        };

        const root = createRoot(document.getElementById('root'));
        root.render(<App />);
    </script>
</body>
</html>
