#import React, { useState, useEffect } from 'react';
import { 
  Map, 
  Wallet, 
  CheckSquare, 
  Plus, 
  Edit2, 
  Sun, 
  CloudSun,
  CloudRain,
  CalendarDays
} from 'lucide-react';

// --- 初始資料 ---

const INITIAL_ITINERARY = [
  {
    day: 1,
    date: "5/1",
    fullDate: "2025-05-01",
    weather: "🌤️ 19°C",
    title: "抵達與適應",
    activities: [
      { time: "14:00", title: "濟州機場取車", type: "交通", desc: "確認兒童座椅與保險，機場附近取車。" },
      { time: "15:30", title: "龍頭岩海岸", type: "景點", desc: "平緩步道，適合長輩小孩暖身。" },
      { time: "17:30", title: "晚餐：鮑魚粥", type: "美食", desc: "濟州市區，清淡好入口。" },
    ]
  },
  {
    day: 2,
    date: "5/2",
    fullDate: "2025-05-02",
    weather: "☀️ 21°C",
    title: "童趣與森林",
    activities: [
      { time: "10:30", title: "Snoopy Garden", type: "主景點", desc: "戶外花園平坦好走，室內有冷氣，適合拍照。" },
      { time: "13:30", title: "午餐：黑豬肉", type: "美食", desc: "舊左邑附近，選座位寬敞店家。" },
      { time: "15:30", title: "月汀里海邊", type: "休閒", desc: "大人喝咖啡，小孩玩沙。" }
    ]
  },
  {
    day: 3,
    date: "5/3",
    fullDate: "2025-05-03",
    weather: "⛅ 20°C",
    title: "海洋奇觀",
    activities: [
      { time: "10:00", title: "Aqua Planet 水族館", type: "主景點", desc: "室內無障礙設施，適合雨備與小孩。" },
      { time: "13:00", title: "午餐：白帶魚", type: "美食", desc: "無刺料理，方便老少食用。" },
      { time: "15:00", title: "光之地堡", type: "藝文", desc: "沉浸式展覽，席地而坐欣賞即可。" }
    ]
  },
  {
    day: 4,
    date: "5/4",
    fullDate: "2025-05-04",
    weather: "🌧️ 18°C",
    title: "花卉與瀑布",
    activities: [
      { time: "10:30", title: "山茶花之丘", type: "主景點", desc: "平緩賞花步道，適合拍全家福。" },
      { time: "13:00", title: "每日偶來市場", type: "美食", desc: "西歸浦傳統市場體驗。" },
      { time: "15:00", title: "天地淵瀑布", type: "景點", desc: "步道最平緩的瀑布景點。" }
    ]
  },
  {
    day: 5,
    date: "5/5",
    fullDate: "2025-05-05",
    weather: "☀️ 22°C",
    title: "茶香與樂園",
    activities: [
      { time: "10:30", title: "雪綠茶博物館", type: "主景點", desc: "吃冰淇淋，做手工皂體驗。" },
      { time: "13:00", title: "午餐：刀削麵", type: "美食", desc: "茶園周邊特色美食。" },
      { time: "15:00", title: "神話世界 / 泳池", type: "休閒", desc: "視體力決定是否進遊樂園。" }
    ]
  },
  {
    day: 6,
    date: "5/6",
    fullDate: "2025-05-06",
    weather: "🌤️ 20°C",
    title: "光影與夕陽",
    activities: [
      { time: "10:30", title: "Arte Museum", type: "主景點", desc: "震撼的媒體藝術展，全室內。" },
      { time: "13:00", title: "午餐：海鮮拉麵", type: "美食", desc: "涯月海邊景觀餐廳。" },
      { time: "15:30", title: "涯月咖啡街", type: "景點", desc: "看海喝咖啡，不需要走太多路。" }
    ]
  },
  {
    day: 7,
    date: "5/7",
    fullDate: "2025-05-07",
    weather: "☀️ 21°C",
    title: "採買與返程",
    activities: [
      { time: "10:00", title: "東門市場", type: "購物", desc: "最後伴手禮採買。" },
      { time: "12:00", title: "午餐：機場附近", type: "美食", desc: "簡單用餐。" },
      { time: "14:00", title: "前往機場", type: "交通", desc: "還車、退稅、登機。" }
    ]
  }
];

// 豐富化的清單內容
const INITIAL_CHECKLIST = [
  // 文件/現金
  { id: 1, text: "護照 (7大1小) / 身份證", checked: false, category: "文件/現金" },
  { id: 2, text: "國際駕照 & 台灣駕照", checked: false, category: "文件/現金" },
  { id: 3, text: "WOWPASS / 韓元現金", checked: false, category: "文件/現金" },
  { id: 4, text: "信用卡 (雙幣卡)", checked: false, category: "文件/現金" },
  
  // 育兒/長者用品
  { id: 5, text: "兒童推車/背帶", checked: false, category: "育兒/長者" },
  { id: 6, text: "尿布/濕紙巾/奶粉/副食品", checked: false, category: "育兒/長者" },
  { id: 7, text: "兒童零食/水壺/玩具", checked: false, category: "育兒/長者" },
  { id: 8, text: "長輩保暖衣物/圍巾", checked: false, category: "育兒/長者" },
  
  // 健康/安全
  { id: 9, text: "常備藥品 (感冒/腸胃/過敏)", checked: false, category: "健康/安全" },
  { id: 10, text: "OK繃/消毒水/防蚊液", checked: false, category: "健康/安全" },
  { id: 11, text: "防曬乳/太陽眼鏡/帽子", checked: false, category: "健康/安全" },
  { id: 12, text: "口罩/酒精噴霧/乾洗手", checked: false, category: "健康/安全" },

  // 電子產品/衣物
  { id: 13, text: "萬用轉接頭/延長線", checked: false, category: "電子產品/衣物" },
  { id: 14, text: "手機充電線/行動電源/AirTag", checked: false, category: "電子產品/衣物" },
  { id: 15, text: "相機/額外記憶卡/自拍桿", checked: false, category: "電子產品/衣物" },
  { id: 16, text: "雨具/輕便雨衣", checked: false, category: "電子產品/衣物" },
  { id: 17, text: "換洗衣物 (含泳衣)", checked: false, category: "電子產品/衣物" },
];

export default function JejuFinalStyleApp() {
  const [activeTab, setActiveTab] = useState('itinerary');
  // 雖然行程全部顯示，但仍保留 selectedDay 供 Header 記帳畫面使用
  const [selectedDay, setSelectedDay] = useState(1); 
  const [itinerary, setItinerary] = useState(INITIAL_ITINERARY);
  const [expenses, setExpenses] = useState([]); // { id, date, title, amount(KRW), note }
  const [checklist, setChecklist] = useState(INITIAL_CHECKLIST);
  
  // 編輯模式狀態
  const [isEditingItinerary, setIsEditingItinerary] = useState(false);
  const [showAddExpense, setShowAddExpense] = useState(false);
  
  // 匯率僅供參考顯示 (預設 1 TWD = 42.5 KRW)
  const [exchangeRate, setExchangeRate] = useState(42.5);

  // Persistence (使用 localStorage 模擬數據持久化)
  useEffect(() => {
    const savedData = localStorage.getItem('jeju_block_style_data_v2');
    if (savedData) {
      const parsed = JSON.parse(savedData);
      if(parsed.itinerary) setItinerary(parsed.itinerary);
      if(parsed.expenses) setExpenses(parsed.expenses);
      if(parsed.checklist) setChecklist(parsed.checklist);
      if(parsed.exchangeRate) setExchangeRate(parsed.exchangeRate);
    }
  }, []);

  useEffect(() => {
    localStorage.setItem('jeju_block_style_data_v2', JSON.stringify({
      itinerary, expenses, checklist, exchangeRate
    }));
  }, [itinerary, expenses, checklist, exchangeRate]);


  // --- 行程操作 Refactor: 需傳入 dayId ---
  
  const updateActivity = (dayId, activityIdx, field, value) => {
    const newItinerary = [...itinerary];
    const dayIdx = newItinerary.findIndex(d => d.day === dayId);
    if (dayIdx !== -1) {
      newItinerary[dayIdx].activities[activityIdx][field] = value;
      setItinerary(newItinerary);
    }
  };

  const addActivity = (dayId) => {
    const newItinerary = [...itinerary];
    const dayIdx = newItinerary.findIndex(d => d.day === dayId);
    if (dayIdx !== -1) {
      newItinerary[dayIdx].activities.push({
        time: "12:00", title: "新行程", type: "自訂", desc: ""
      });
      setItinerary(newItinerary);
    }
  };


  // --- Components ---

  const Header = () => (
    // Style: 更加簡潔，強調 Day 數字
    <div className="bg-white px-6 pt-12 pb-4 sticky top-0 z-10 border-b border-gray-100 shadow-md">
      <div className="flex justify-between items-end">
        <div>
          <h1 className="text-2xl font-semibold text-cyan-700 tracking-tight flex items-center gap-2">
            <span className="font-sans">濟州島慢旅</span> 
            {/* Action/Accent color: teal-600 (Blue-Green) */}
            <span className="text-sm bg-teal-600 text-white px-2 py-0.5 text-[10px] tracking-widest rounded-md font-bold">完整排程</span>
          </h1>
          <p className="text-gray-500 text-xs mt-2 font-light">五月的一場慢旅行 ‧ 7大1小</p>
        </div>
        <div className="text-right">
            {/* 當在完整行程頁時，顯示總覽標籤，否則顯示當前選定的 Day 數 */}
           {activeTab === 'itinerary' ? (
               <div className="text-xl font-bold text-gray-400 font-sans flex items-center gap-1">
                 <CalendarDays size={24} strokeWidth={2}/> 7天總覽
               </div>
           ) : (
                <div className="text-4xl font-extrabold text-cyan-700 font-sans">
                  D{selectedDay}
                </div>
           )}
        </div>
      </div>
    </div>
  );

  const BottomNav = () => (
    // Navigation: 使用實心方塊作為活躍狀態指示
    <div className="fixed bottom-0 left-0 right-0 bg-white border-t-4 border-cyan-700 flex justify-around p-3 z-50 max-w-md mx-auto shadow-2xl">
      <button onClick={() => setActiveTab('itinerary')} className={`flex flex-col items-center transition-colors p-1 rounded-md ${activeTab === 'itinerary' ? 'text-white bg-cyan-700 shadow-lg' : 'text-gray-500 hover:text-cyan-600'}`}>
        <Map strokeWidth={2.5} size={26} />
        <span className="text-[10px] mt-1 tracking-wider font-bold">行程</span>
      </button>
      <button onClick={() => setActiveTab('accounting')} className={`flex flex-col items-center transition-colors p-1 rounded-md ${activeTab === 'accounting' ? 'text-white bg-cyan-700 shadow-lg' : 'text-gray-500 hover:text-cyan-600'}`}>
        <Wallet strokeWidth={2.5} size={26} />
        <span className="text-[10px] mt-1 tracking-wider font-bold">錢包</span>
      </button>
      <button onClick={() => setActiveTab('checklist')} className={`flex flex-col items-center transition-colors p-1 rounded-md ${activeTab === 'checklist' ? 'text-white bg-cyan-700 shadow-lg' : 'text-gray-500 hover:text-cyan-600'}`}>
        <CheckSquare strokeWidth={2.5} size={26} />
        <span className="text-[10px] mt-1 tracking-wider font-bold">備忘</span>
      </button>
    </div>
  );

  // --- Views ---

  const ItineraryView = () => {
    
    // Helper to select the correct weather icon
    const getWeatherIcon = (weather) => {
      // 使用更粗的 strokeWidth 模擬方塊/實心感
      const stroke = 2.5; 
      if (weather.includes('🌧️')) return <CloudRain strokeWidth={stroke} size={32} className="text-white"/>;
      if (weather.includes('⛅')) return <CloudSun strokeWidth={stroke} size={32} className="text-white"/>;
      return <Sun strokeWidth={stroke} size={32} className="text-white"/>;
    };


    return (
      <div className="pb-24 animate-in fade-in bg-gray-100 px-4 py-6 space-y-8">

        {/* 完整的 7 天行程列表 (垂直滾動) */}
        {itinerary.map((dayData) => (
          <div key={dayData.day} className="bg-white rounded-xl shadow-lg border border-gray-200 overflow-hidden">
            
            {/* Day Header: Weather + Title/Summary (放大佈局) */}
            <div className="flex flex-col sm:flex-row mb-4">
                {/* 1. Weather Block (Prominent) - Action color: teal-600 */}
                <div className="p-4 bg-teal-600 text-white shadow-lg flex flex-row sm:flex-col items-center justify-between sm:justify-center flex-shrink-0 sm:w-1/3">
                    <div className="flex items-center gap-3">
                        {getWeatherIcon(dayData.weather)}
                        <span className="text-3xl font-extrabold">{dayData.weather.split(' ')[1]}</span>
                    </div>
                    <div className="text-right sm:text-center mt-0 sm:mt-2">
                         <span className="text-xs uppercase tracking-wider font-semibold opacity-90 block">DAY {dayData.day} ({dayData.date})</span>
                         <span className="text-sm font-semibold opacity-90">{dayData.weather.split(' ')[0]}</span>
                    </div>
                </div>

                {/* 2. Title/Summary Block - System color: cyan-700 */}
                <div className="p-4 flex-1 border-l-4 border-cyan-700 bg-white">
                    <span className="text-xs text-gray-500 font-medium block uppercase tracking-wider">今日主題</span>
                    <h2 className="text-xl font-bold text-cyan-700 leading-snug mt-1">{dayData.title}</h2>
                    <p className="text-xs text-gray-500 mt-2">
                        {dayData.activities.map(a => a.title).join(' / ')}
                    </p>
                </div>
            </div>

            {/* Activity List */}
            <div className="px-4 space-y-3 pb-4">
                {dayData.activities.map((act, activityIdx) => (
                    <div key={activityIdx} className={`p-3 bg-gray-50 rounded-lg border-l-4 transition-all hover:shadow-md ${
                        act.type === '主景點' ? 'border-cyan-700' : 'border-teal-500'
                    }`}>
                        {isEditingItinerary ? (
                            <div className="space-y-3">
                                <div className="flex gap-2">
                                    <input 
                                        value={act.time} 
                                        onChange={(e)=>updateActivity(dayData.day, activityIdx, 'time', e.target.value)}
                                        className="w-16 bg-white border border-gray-300 p-2 text-sm text-center rounded focus:border-cyan-700 focus:ring-1 focus:ring-cyan-700"
                                    />
                                    <input 
                                        value={act.type} 
                                        onChange={(e)=>updateActivity(dayData.day, activityIdx, 'type', e.target.value)}
                                        className="flex-1 bg-white border border-gray-300 p-2 text-sm text-center rounded focus:border-cyan-700 focus:ring-1 focus:ring-cyan-700"
                                    />
                                </div>
                                <input 
                                    value={act.title} 
                                    onChange={(e)=>updateActivity(dayData.day, activityIdx, 'title', e.target.value)}
                                    className="w-full bg-white border border-gray-300 p-2 font-medium rounded focus:border-cyan-700 focus:ring-1 focus:ring-cyan-700" 
                                    placeholder="標題"
                                />
                                <textarea 
                                    value={act.desc} 
                                    onChange={(e)=>updateActivity(dayData.day, activityIdx, 'desc', e.target.value)}
                                    className="w-full bg-white border border-gray-300 p-2 text-sm rounded focus:border-cyan-700 focus:ring-1 focus:ring-cyan-700" 
                                    rows={2}
                                    placeholder="描述"
                                />
                            </div>
                        ) : (
                            <div>
                                <div className="flex justify-between items-start mb-2">
                                    <h3 className="text-lg font-semibold text-gray-800">{act.title}</h3>
                                    {/* 時間標記使用方塊背景 */}
                                    <span className="font-mono text-sm bg-gray-200 text-gray-600 px-2 py-1 rounded-md">{act.time}</span>
                                </div>
                                <p className="text-sm text-gray-600 font-light leading-relaxed">{act.desc}</p>
                                {/* 類型標籤使用實心顏色 */}
                                <div className="mt-3">
                                    <span className="text-[10px] bg-teal-100 text-teal-700 px-2 py-1 rounded-md font-semibold">{act.type}</span>
                                </div>
                            </div>
                        )}
                    </div>
                ))}
                
                {/* 每日新增按鈕 (只在編輯模式下顯示) */}
                {isEditingItinerary && (
                     <button 
                        onClick={() => addActivity(dayData.day)}
                        className="w-full py-2 border-2 border-dashed border-cyan-400 text-cyan-700 text-sm rounded-lg hover:bg-cyan-50 flex items-center justify-center gap-2 transition-colors font-bold mt-4"
                    >
                        <Plus size={18}/> 新增 Day {dayData.day} 行程
                    </button>
                )}
            </div>
            
          </div>
        ))}
        
        {/* 全局編輯控制按鈕 */}
        <div className="pt-4 flex gap-3 sticky bottom-24 bg-gray-100 z-10 p-2 rounded-lg shadow-lg">
            {isEditingItinerary ? (
                <button 
                    onClick={() => setIsEditingItinerary(false)}
                    className="flex-1 py-3 bg-teal-600 text-white text-base rounded-lg shadow-xl hover:bg-teal-700 transition-colors font-bold"
                >
                    完成所有編輯
                </button>
            ) : (
                <button 
                    onClick={() => setIsEditingItinerary(true)}
                    className="flex-1 py-3 bg-cyan-700 text-white text-base rounded-lg shadow-xl hover:bg-cyan-800 flex items-center justify-center gap-2 transition-colors font-bold"
                >
                    <Edit2 size={18}/> 啟用編輯模式
                </button>
            )}
        </div>
      </div>
    );
  };

  const AccountingView = () => {
    const [newItem, setNewItem] = useState({
        date: itinerary.find(d => d.day === selectedDay)?.fullDate || "2025-05-01",
        title: "",
        amount: "", // Input as String
        note: ""
    });

    // Helper to group expenses by date
    const groupedExpenses = expenses.reduce((groups, item) => {
        const date = item.date;
        if (!groups[date]) groups[date] = [];
        groups[date].push(item);
        return groups;
    }, {});

    // Sort dates
    const sortedDates = Object.keys(groupedExpenses).sort().reverse(); // Show newest first

    const totalKRW = expenses.reduce((sum, item) => sum + Number(item.amount), 0);

    const handleAddExpense = () => {
        if (!newItem.title || !newItem.amount) return;
        setExpenses([{...newItem, id: Date.now(), amount: Number(newItem.amount)}, ...expenses]); // Ensure amount is number before saving
        setNewItem({...newItem, title:"", amount:"", note:""});
        setShowAddExpense(false);
    };

    const handleDeleteExpense = (id) => {
        setExpenses(expenses.filter(e => e.id !== id));
    };

    return (
        <div className="pb-24 bg-gray-100 min-h-screen">
            {/* Total Card - Action/Accent color: teal-600 */}
            <div className="px-4 py-8 bg-teal-600 text-white mb-4 shadow-lg">
                <div className="flex justify-between items-start">
                    <div>
                        <div className="text-xs tracking-widest uppercase mb-1 opacity-80">總花費 (韓元)</div>
                        <div className="text-4xl font-extrabold font-mono">
                            ₩ {totalKRW.toLocaleString()}
                        </div>
                        <div className="text-sm mt-2 opacity-90">
                            約 NT$ {Math.round(totalKRW / exchangeRate).toLocaleString()}
                        </div>
                    </div>
                    {/* Floating Add Button - Action color: teal-600, using square with rounded corners */}
                    <button 
                        onClick={() => setShowAddExpense(true)}
                        className="bg-white text-teal-600 rounded-lg w-12 h-12 flex items-center justify-center shadow-lg hover:bg-gray-100 transition-transform active:scale-95"
                    >
                        <Plus strokeWidth={2.5} size={24}/>
                    </button>
                </div>
            </div>

            {/* Daily Lists */}
            <div className="px-4 space-y-4">
                {sortedDates.length === 0 && (
                    <div className="text-center py-12 text-gray-400 font-light bg-white rounded-lg shadow-sm">
                        尚無消費紀錄
                    </div>
                )}
                
                {sortedDates.map(date => {
                    const dayExpenses = groupedExpenses[date];
                    const dayTotal = dayExpenses.reduce((s, i) => s + Number(i.amount), 0);
                    
                    return (
                        <div key={date} className="bg-white rounded-lg p-4 border-2 border-gray-200 shadow-md">
                            <div className="flex justify-between items-center mb-3 pb-2 border-b border-cyan-400/50">
                                {/* Daily Note/Total Tag - System color: cyan-700 */}
                                <span className="font-bold text-gray-700 text-sm bg-cyan-100 px-3 py-1 rounded-md">{date} 每日小計</span>
                                <span className="text-lg font-mono text-gray-800 font-extrabold">₩ {dayTotal.toLocaleString()}</span>
                            </div>
                            <div className="space-y-3">
                                {dayExpenses.map(exp => (
                                    // 每個支出項目也用分明的線條隔開
                                    <div key={exp.id} className="flex justify-between items-center text-sm border-b border-dashed border-gray-200 last:border-b-0 py-2">
                                        <div>
                                            <div className="text-gray-800 font-medium">{exp.title}</div>
                                            {exp.note && <div className="text-xs text-gray-500 mt-0.5">{exp.note}</div>}
                                        </div>
                                        <div className="flex items-center gap-3">
                                            <span className="font-mono text-base text-teal-600 font-semibold">₩ {Number(exp.amount).toLocaleString()}</span>
                                            {/* Delete button: bolder icon */}
                                            <button 
                                                onClick={() => handleDeleteExpense(exp.id)} 
                                                className="text-gray-400 hover:text-red-600 transition-colors"
                                                title="刪除此筆記錄"
                                            >
                                                <svg xmlns="http://www.w3.org/2000/svg" width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" strokeWidth="2.5" strokeLinecap="round" strokeLinejoin="round" className="lucide lucide-trash-2"><path d="M3 6h18"/><path d="M19 6v14c0 1-1 2-2 2H7c-1 0-2-1-2-2V6"/><path d="M8 6V4c0-1 1-2 2-2h4c1 0 2 1 2 2v2"/><line x1="10" x2="10" y1="11" y2="17"/><line x1="14" x2="14" y1="11" y2="17"/></svg>
                                            </button>
                                        </div>
                                    </div>
                                ))}
                            </div>
                        </div>
                    );
                })}
            </div>

            {/* Add Modal */}
            {showAddExpense && (
                <div className="fixed inset-0 bg-gray-800/60 backdrop-blur-sm z-[60] flex items-center justify-center p-6">
                    <div className="bg-white w-full max-w-sm rounded-xl shadow-2xl p-6 animate-in zoom-in-95 duration-200 border-t-4 border-cyan-700">
                        <h3 className="text-xl font-bold text-gray-800 mb-6 text-center border-b pb-3">新增支出項目 (韓元)</h3>
                        
                        <div className="space-y-4">
                            <div>
                                <label className="block text-xs text-gray-500 mb-1 font-semibold">日期</label>
                                <input 
                                    type="date" 
                                    value={newItem.date}
                                    onChange={e => setNewItem({...newItem, date: e.target.value})}
                                    className="w-full border-2 border-gray-300 p-2 rounded-md focus:outline-none focus:border-cyan-700 bg-gray-50"
                                />
                            </div>
                            <div>
                                <label className="block text-xs text-gray-500 mb-1 font-semibold">項目</label>
                                <input 
                                    type="text" 
                                    placeholder="晚餐、門票..."
                                    value={newItem.title}
                                    onChange={e => setNewItem({...newItem, title: e.target.value})}
                                    className="w-full border-2 border-gray-300 p-2 rounded-md focus:outline-none focus:border-cyan-700 bg-gray-50 placeholder-gray-400"
                                    autoFocus
                                />
                            </div>
                            <div>
                                <label className="block text-xs text-gray-500 mb-1 font-semibold">金額 (₩ KRW)</label>
                                <input 
                                    type="number" 
                                    placeholder="0"
                                    value={newItem.amount}
                                    onChange={e => setNewItem({...newItem, amount: e.target.value})}
                                    className="w-full border-2 border-gray-300 p-2 text-xl font-mono rounded-md focus:outline-none focus:border-cyan-700 bg-gray-50"
                                />
                                {newItem.amount && (
                                    <p className="text-right text-xs text-gray-500 mt-1 font-mono">
                                        約 TWD {Math.round(newItem.amount / exchangeRate).toLocaleString()}
                                    </p>
                                )}
                            </div>
                             <div>
                                <label className="block text-xs text-gray-500 mb-1 font-semibold">備註 (可選)</label>
                                <input 
                                    type="text" 
                                    value={newItem.note}
                                    onChange={e => setNewItem({...newItem, note: e.target.value})}
                                    className="w-full border-2 border-gray-300 p-2 rounded-md focus:outline-none focus:border-cyan-700 bg-gray-50 placeholder-gray-400"
                                />
                            </div>
                            
                            <div className="pt-4 flex gap-3">
                                <button 
                                    onClick={() => setShowAddExpense(false)}
                                    className="flex-1 py-3 text-gray-600 border-2 border-gray-300 rounded-lg hover:bg-gray-100 transition-colors font-bold"
                                >
                                    取消
                                </button>
                                <button 
                                    onClick={handleAddExpense}
                                    // Action button uses Teal-600
                                    className="flex-1 py-3 bg-teal-600 text-white rounded-lg shadow-xl hover:bg-teal-700 transition-colors font-bold"
                                >
                                    儲存
                                </button>
                            </div>
                        </div>
                    </div>
                </div>
            )}
        </div>
    );
  };

  const ChecklistView = () => {
    // 按 Category 分組清單
    const groupedChecklist = checklist.reduce((groups, item) => {
        const category = item.category || '未分類';
        if (!groups[category]) groups[category] = [];
        groups[category].push(item);
        return groups;
    }, {});
    
    // 總進度計算
    const totalItems = checklist.length;
    const checkedItems = checklist.filter(i => i.checked).length;
    const progress = totalItems > 0 ? Math.round((checkedItems / totalItems) * 100) : 0;
    
    // 勾選/取消勾選
    const toggleCheck = (id) => {
        setChecklist(checklist.map(i => i.id === id ? { ...i, checked: !i.checked } : i));
    };

    return (
        <div className="pb-24 bg-gray-100 min-h-screen px-4 py-6 animate-in fade-in">
          {/* 進度總覽卡片 - 藍綠配色 */}
          <div className="bg-cyan-700 text-white p-6 rounded-xl shadow-lg mb-6">
            <h2 className="text-xl font-bold tracking-wide flex items-center gap-2">
                <CheckSquare size={24} strokeWidth={2.5}/> 行前準備總覽
            </h2>
            <p className="text-xs opacity-90 mt-2">已完成 {checkedItems} / {totalItems} 項</p>
            <div className="w-full bg-cyan-800 rounded-full h-2.5 mt-3">
                <div 
                    className="bg-teal-400 h-2.5 rounded-full transition-all duration-500" 
                    style={{ width: `${progress}%` }}
                ></div>
            </div>
            <p className="text-sm font-bold mt-2 text-right">{progress}%</p>
          </div>
          
          {/* 清單項目列表 (按分類分組) */}
          <div className="space-y-6">
            {Object.keys(groupedChecklist).map(category => (
                <div key={category} className="bg-white rounded-xl shadow-md border-t-4 border-teal-500 overflow-hidden">
                    <div className="p-4 bg-gray-50 border-b border-gray-200">
                        <h3 className="text-lg font-bold text-gray-700">{category}</h3>
                    </div>
                    
                    <div className="divide-y divide-gray-100 px-4">
                        {groupedChecklist[category].map(item => (
                            <div 
                                key={item.id} 
                                onClick={() => toggleCheck(item.id)}
                                className={`py-3 flex items-center gap-4 cursor-pointer transition-colors ${
                                    item.checked ? 'bg-white' : 'hover:bg-gray-50'
                                }`}
                            >
                                {/* Checkbox - 使用實心方塊圖示 */}
                                <div className={`w-6 h-6 border-2 transition-colors flex items-center justify-center rounded-sm flex-shrink-0 ${
                                    item.checked ? 'bg-teal-600 border-teal-600' : 'border-gray-400'
                                }`}>
                                     {item.checked && <svg xmlns="http://www.w3.org/2000/svg" width="18" height="18" viewBox="0 0 24 24" fill="none" stroke="currentColor" strokeWidth="3" strokeLinecap="round" strokeLinejoin="round" className="lucide lucide-check text-white"><path d="M20 6 9 17l-5-5"/></svg>}
                                </div>
                                <span className={`text-base tracking-wide transition-colors font-medium ${item.checked ? 'text-gray-400 line-through' : 'text-gray-700'}`}>
                                  {item.text}
                                </span>
                            </div>
                        ))}
                    </div>
                </div>
            ))}
          </div>


          <div className="mt-8 p-6 bg-white rounded-xl text-center border-2 border-gray-300 shadow-sm">
              <p className="text-xs text-gray-500 mb-2 uppercase tracking-widest font-semibold">匯率設定 (韓元/台幣)</p>
              <div className="flex justify-center items-center gap-2">
                <span className="text-sm text-gray-600 font-bold">1 TWD = </span>
                <input 
                    type="number"
                    value={exchangeRate}
                    onChange={e => setExchangeRate(Number(e.target.value))}
                    // System color focus: cyan-700
                    className="w-20 bg-gray-100 border-2 border-gray-300 p-1 text-center font-mono text-lg text-gray-700 rounded-md focus:outline-none focus:border-cyan-700"
                />
                <span className="text-sm text-gray-600 font-bold">KRW</span>
              </div>
          </div>
        </div>
    );
  };

  return (
    <div className="min-h-screen bg-gray-100 font-sans text-gray-800 max-w-md mx-auto shadow-2xl border-x border-gray-200 overflow-hidden relative">
      <Header />
      <main>
        {activeTab === 'itinerary' && <ItineraryView />}
        {activeTab === 'accounting' && <AccountingView />}
        {activeTab === 'checklist' && <ChecklistView />}
      </main>
      <BottomNav />
    </div>
  );
}

