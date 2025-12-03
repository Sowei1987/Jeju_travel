<!DOCTYPE html>
<html lang="zh-TW">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>濟州慢旅規劃 App (v3.0)</title>
    <!-- 載入 Tailwind CSS CDN -->
    <script src="https://cdn.tailwindcss.com"></script>
    <!-- 載入 React, ReactDOM, and Babel CDNs -->
    <script src="https://unpkg.com/react@18/umd/react.development.js"></script>
    <script src="https://unpkg.com/react-dom@18/umd/react-dom.development.js"></script>
    <script src="https://unpkg.com/@babel/standalone/babel.min.js"></script>
    <!-- Load Lucide icons (used for icons in the original React component) -->
    <script type="module">
        import * as lucide from 'https://unpkg.com/lucide@latest/dist/esm/lucide.js';
        window.lucide = lucide;
    </script>
    <style>
        /* 確保字體在不同設備上清晰，並使用圓角 */
        body {
            font-family: 'Inter', system-ui, -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, "Helvetica Neue", Arial, "Noto Sans", sans-serif, "Apple Color Emoji", "Segoe UI Emoji", "Segoe UI Symbol", "Noto Color Emoji";
            background-color: #f3f4f6;
        }
        /* 限制主容器最大寬度，模擬手機應用程式 */
        #root > div {
            max-width: 480px; /* 略寬於 max-w-md */
            margin-left: auto;
            margin-right: auto;
        }
        /* 隱藏滾動條，保持橫向排版美觀 */
        .hide-scrollbar::-webkit-scrollbar {
            display: none;
        }
        .hide-scrollbar {
            -ms-overflow-style: none; /* IE and Edge */
            scrollbar-width: none; /* Firefox */
        }
    </style>
</head>
<body>

<div id="root"></div>

<script type="text/babel">
const { useState, useEffect, useMemo } = React;

// --- 圖標解構 (已將 Map 更名為 MapIcon 以避免與原生 Map 衝突) ---
const { 
  Map: MapIcon, Wallet, CheckSquare, Plus, Edit2, Sun, CloudSun, CloudRain,
  CalendarDays, PhoneCall, X, Clock, Truck, Home, BookOpen, Plane, Trash2 
} = window.lucide;

// --- 初始資料與常數 ---

const APP_VERSION = "v3.0"; 
// 使用新的 STORAGE_KEY 以避免與舊版本資料衝突
const STORAGE_KEY = `jeju_block_style_data_${APP_VERSION.replace('.', '_')}`; 

const INITIAL_ITINERARY = [
  {
    day: 1,
    date: "5/1",
    dayOfWeek: "五", 
    fullDate: "2025-05-01",
    weather: "🌤️ 19°C",
    location: "濟州市區 (北部)", 
    focus: "抵達與適應，確認租車與住宿", 
    title: "抵達與適應 (濟州市)",
    activities: [
      { time: "14:00", title: "抵達濟州國際機場", type: "交通", icon: Plane, desc: "確認兒童座椅與保險，機場附近取車，GPS設定正確。" },
      { time: "15:30", title: "龍頭岩海岸散步", type: "景點", icon: MapIcon, desc: "平緩步道，適合長輩小孩暖身，拍拍抵達照。" },
      { time: "17:30", title: "晚餐：鮑魚粥與海鮮湯", type: "美食", icon: Wallet, desc: "濟州市區，清淡好入口，需注意等候時間。" },
      { time: "20:00", title: "入住濟州市區酒店", type: "住宿", icon: Home, desc: "確認房間配置，嬰兒床已準備。" }
    ]
  },
  {
    day: 2,
    date: "5/2",
    dayOfWeek: "六",
    fullDate: "2025-05-02",
    weather: "☀️ 21°C",
    location: "東部海岸線",
    focus: "童趣與森林探險：史努比庭園",
    title: "東部：童趣與森林探險",
    activities: [
      { time: "09:30", title: "Snoopy Garden (史努比庭園)", type: "主景點", icon: CheckSquare, desc: "戶外花園平坦好走，室內展區有冷氣，預計停留3小時。" },
      { time: "13:30", title: "午餐：黑豬肉烤肉", type: "美食", icon: Wallet, desc: "舊左邑附近，選座位寬敞店家，避免油煙影響幼兒。" },
      { time: "15:30", title: "月汀里海邊", type: "休閒", icon: Sun, desc: "大人輪流喝咖啡 (鯨魚咖啡)，小孩在沙灘玩沙。" },
      { time: "18:30", title: "晚餐：海鮮拉麵", type: "美食", icon: Wallet, desc: "景觀餐廳，看夕陽，需先訂位。" }
    ]
  },
  {
    day: 3,
    date: "5/3",
    dayOfWeek: "日",
    fullDate: "2025-05-03",
    weather: "⛅ 20°C",
    location: "城山/西歸浦",
    focus: "海洋奇觀與藝術：水族館",
    title: "東部：海洋奇觀與藝術",
    activities: [
      { time: "10:00", title: "Aqua Planet 水族館", type: "主景點", icon: CheckSquare, desc: "室內無障礙設施，適合雨備與小孩，看海豚表演。" },
      { time: "13:00", title: "午餐：白帶魚套餐", type: "美食", icon: Wallet, desc: "無刺料理，方便老少食用，提前抵達避開人潮。" },
      { time: "15:00", title: "光之地堡 (Bunker de Lumières)", type: "藝文", icon: BookOpen, desc: "沉浸式展覽，席地而坐欣賞，無須步行太多。" },
      { time: "17:30", title: "城山日出峰 (遠觀)", type: "景點", icon: MapIcon, desc: "在海岸邊遠觀即可，不需爬山，找個平台拍照。" }
    ]
  },
  {
    day: 4,
    date: "5/4",
    dayOfWeek: "一",
    fullDate: "2025-05-04",
    weather: "🌧️ 18°C",
    location: "西歸浦市區",
    focus: "市場美食與瀑布：每日偶來市場",
    title: "南部：市場美食與瀑布",
    activities: [
      { time: "10:30", title: "山茶花之丘 (Camellia Hill)", type: "景點", icon: MapIcon, desc: "下雨天撐傘賞花，步道平緩，注意地面濕滑。" },
      { time: "13:00", title: "每日偶來市場", type: "美食", icon: Wallet, desc: "西歸浦傳統市場體驗，主要購買橘子乾等伴手禮。" },
      { time: "15:00", title: "天地淵瀑布", type: "景點", icon: MapIcon, desc: "步道最平緩的瀑布景點，入口有小橋流水。" },
      { time: "18:00", title: "晚餐：烤肉吃到飽", type: "美食", icon: Wallet, desc: "在西歸浦市區找有兒童遊戲區的餐廳。" }
    ]
  },
  {
    day: 5,
    date: "5/5",
    dayOfWeek: "二",
    fullDate: "2025-05-05",
    weather: "☀️ 22°C",
    location: "西部綠茶區",
    focus: "茶香與休閒樂園：O'sulloc",
    title: "西部：茶香與休閒樂園",
    activities: [
      { time: "10:30", title: "雪綠茶博物館 (O'sulloc)", type: "主景點", icon: CheckSquare, desc: "吃綠茶冰淇淋，逛商店，室內區適合休息。" },
      { time: "13:00", title: "午餐：海邊刀削麵", type: "美食", icon: Wallet, desc: "茶園周邊特色美食，注意麵條是否適合幼兒。" },
      { time: "15:00", title: "神話世界度假區", type: "休閒", icon: Home, desc: "視長輩體力決定是否進遊樂園，或單純在度假村內散步。" },
      { time: "18:00", title: "晚餐：度假村內精緻料理", type: "美食", icon: Wallet, desc: "在酒店附近用餐，方便休息。" }
    ]
  },
  {
    day: 6,
    date: "5/6",
    dayOfWeek: "三",
    fullDate: "2025-05-06",
    weather: "🌤️ 20°C",
    location: "涯月海岸線",
    focus: "光影與涯月海岸：Arte Museum",
    title: "西部：光影與涯月海岸",
    activities: [
      { time: "10:30", title: "Arte Museum (媒體藝術)", type: "主景點", icon: CheckSquare, desc: "震撼的媒體藝術展，全室內，注意聲光效果。" },
      { time: "13:00", title: "午餐：部隊鍋", type: "美食", icon: Wallet, desc: "在市區解決，熱食為主。" },
      { time: "15:30", title: "涯月咖啡街 (GD咖啡)", type: "景點", icon: Sun, desc: "選有戶外座位的咖啡館，看海，不需要走太多路。" },
      { time: "18:00", title: "晚餐：東門市場二訪", type: "購物/美食", icon: Wallet, desc: "採購伴手禮並解決晚餐。" }
    ]
  },
  {
    day: 7,
    date: "5/7",
    dayOfWeek: "四",
    fullDate: "2025-05-07",
    weather: "☀️ 21°C",
    location: "濟州市區",
    focus: "最終採買與返程準備",
    title: "市區：最終採買與返程",
    activities: [
      { time: "10:00", title: "樂天超市/東門市場", type: "購物", icon: Wallet, desc: "最後伴手禮採買，重點是韓國泡麵、海苔、零食。" },
      { time: "12:00", title: "午餐：機場附近簡易餐飲", type: "美食", icon: Wallet, desc: "簡單用餐，輕便為主。" },
      { time: "14:00", title: "前往機場還車", type: "交通", icon: Truck, desc: "確保油箱加滿，檢查有無遺漏物品在車內。" },
      { time: "16:30", title: "搭機返台", type: "交通", icon: Plane, desc: "辦理退稅手續，確認登機時間。" }
    ]
  }
];

const INITIAL_CHECKLIST = [
  // 文件/現金
  { id: 1, text: "護照 (7大1小) / 身份證", checked: false },
  { id: 2, text: "國際駕照 & 台灣駕照", checked: false },
  { id: 3, text: "WOWPASS / 韓元現金", checked: false },
  { id: 4, text: "信用卡 (雙幣卡)", checked: false },
  // 育兒/長者用品
  { id: 5, text: "兒童推車/背帶/安全座椅", checked: false },
  { id: 6, text: "尿布/濕紙巾/奶粉/副食品 (備足3天量)", checked: false },
  { id: 7, text: "長輩保暖衣物/輕薄羽絨", checked: false },
  { id: 8, text: "拐杖/助行器 (如需要)", checked: false },
  // 健康/安全
  { id: 9, text: "常備藥品 (感冒/腸胃/過敏) - 需備齊", checked: false },
  { id: 10, text: "OK繃/消毒水/防蚊液/電蚊香", checked: false },
  { id: 11, text: "防曬乳/太陽眼鏡/帽子", checked: false },
  { id: 12, text: "口罩/酒精噴霧/乾洗手", checked: false },
  // 電子產品/衣物
  { id: 13, text: "萬用轉接頭/延長線 (多孔)", checked: false },
  { id: 14, text: "手機充電線/行動電源/AirTag (重要)", checked: false },
  { id: 15, text: "相機/額外記憶卡/自拍桿", checked: false },
  { id: 16, text: "雨具/輕便雨衣/防風外套", checked: false },
  { id: 17, text: "換洗衣物 (含泳衣) - 7套", checked: false },
];

function JejuFinalStyleApp() {
  const [activeTab, setActiveTab] = useState('itinerary');
  const [selectedDay, setSelectedDay] = useState(1); 
  const [itinerary, setItinerary] = useState(INITIAL_ITINERARY);
  const [expenses, setExpenses] = useState([]); // { id, date, title, amount(KRW), note }
  const [checklist, setChecklist] = useState(INITIAL_CHECKLIST);
  
  const [isEditingItinerary, setIsEditingItinerary] = useState(false);
  const [showAddExpense, setShowAddExpense] = useState(false);
  const [exchangeRate, setExchangeRate] = useState(42.5);
  const [newItemText, setNewItemText] = useState(''); // Checklist new item state
  
  const [newItem, setNewItem] = useState({ // Accounting new item state
        date: INITIAL_ITINERARY[0].fullDate,
        title: "",
        amount: "", // Input as String
        note: ""
    });


  // Persistence (使用 localStorage 模擬數據持久化)
  useEffect(() => {
    const savedData = localStorage.getItem(STORAGE_KEY); 
    if (savedData) {
      try {
        const parsed = JSON.parse(savedData);
        if(parsed.itinerary) setItinerary(parsed.itinerary);
        if(parsed.expenses) setExpenses(parsed.expenses);
        if(parsed.checklist) setChecklist(parsed.checklist);
        if(parsed.exchangeRate) setExchangeRate(parsed.exchangeRate);
      } catch (e) {
        console.error("Error parsing localStorage data:", e);
      }
    }
  }, []);

  useEffect(() => {
    localStorage.setItem(STORAGE_KEY, JSON.stringify({
      itinerary, expenses, checklist, exchangeRate
    }));
  }, [itinerary, expenses, checklist, exchangeRate]);


  // --- 行程操作 ---
  
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
        time: "12:00", title: "新行程", type: "自訂", icon: BookOpen, desc: "請輸入詳細描述"
      });
      setItinerary(newItinerary);
    }
  };

  const removeActivity = (dayId, activityIdx) => {
      const newItinerary = [...itinerary];
      const dayIdx = newItinerary.findIndex(d => d.day === dayId);
      if (dayIdx !== -1) {
          newItinerary[dayIdx].activities.splice(activityIdx, 1);
          setItinerary(newItinerary);
      }
  };
  
  // --- 備忘清單操作 (新增功能) ---
  
  const toggleCheck = (id) => {
      setChecklist(checklist.map(i => i.id === id ? { ...i, checked: !i.checked } : i));
  };

  const handleRemoveItem = (id) => {
      setChecklist(checklist.filter(i => i.id !== id));
  };
  
  const handleAddItem = (text) => {
      if (!text.trim()) return;
      setChecklist([...checklist, { id: Date.now(), text: text.trim(), checked: false }]);
  };
  
  // --- 記帳操作 ---
  
  const handleAddExpense = () => {
    if (!newItem.title || !newItem.amount) return;
    setExpenses([{...newItem, id: Date.now(), amount: Number(newItem.amount)}, ...expenses]); 
    setNewItem({...newItem, title:"", amount:"", note:""});
    setShowAddExpense(false);
  };

  const handleDeleteExpense = (id) => {
      setExpenses(expenses.filter(e => e.id !== id));
  };


  // --- Components & Helpers ---
  
  // 輔助函數：獲取天氣圖標
  const getWeatherIcon = (weather) => {
    const stroke = 2.5; 
    // 使用 switch/case 結構替代 if/else if 鏈
    if (weather.includes('🌧️')) return <CloudRain strokeWidth={stroke} size={24} className="text-white"/>;
    if (weather.includes('⛅')) return <CloudSun strokeWidth={stroke} size={24} className="text-white"/>;
    return <Sun strokeWidth={stroke} size={24} className="text-white"/>;
  };
  
  // 輔助函數：根據類型獲取顏色
  const getTypeColor = (type) => {
      switch (type) {
          case '交通': return 'text-purple-600 bg-purple-100';
          case '住宿': return 'text-red-600 bg-red-100';
          case '主景點': return 'text-cyan-700 bg-cyan-100';
          case '美食': return 'text-orange-600 bg-orange-100';
          default: return 'text-gray-600 bg-gray-100';
      }
  };


  const Header = () => (
    // Style: 頂部 Logo 和標題
    <div className="bg-white px-6 pt-12 pb-4 sticky top-0 z-10 border-b border-gray-100 shadow-md">
      <div className="flex justify-between items-end">
        <div className="flex items-center gap-3">
            <div className="text-cyan-700">
                {/* 簡單的三角形 Logo 符號 */}
                <svg width="24" height="24" viewBox="0 0 24 24" fill="currentColor" xmlns="http://www.w3.org/2000/svg">
                    <path d="M12 2L2 22H22L12 2ZM12 5.5L18.5 19H5.5L12 5.5Z" fill="#00BCD4"/>
                </svg>
            </div>
          <h1 className="text-2xl font-semibold text-gray-800 tracking-tight flex items-center gap-2">
            濟州島慢旅計畫
          </h1>
        </div>
        <div className="text-right">
            {/* 版本標籤修正：使用 APP_VERSION 變數，並移除重複的 'beta' */}
            <span className="text-xs bg-teal-500 text-white px-2 py-0.5 text-[10px] tracking-widest rounded-md font-bold">
                {APP_VERSION} (Cloud Sync)
            </span>
        </div>
      </div>
    </div>
  );

  const BottomNav = () => (
    // Navigation: 仿圖片中的樣式，使用文字與圖標
    <div className="fixed bottom-0 left-0 right-0 bg-white border-t border-gray-200 flex justify-around py-3 z-50 max-w-md mx-auto shadow-2xl">
      <button onClick={() => setActiveTab('itinerary')} className={`flex flex-col items-center transition-colors ${activeTab === 'itinerary' ? 'text-cyan-700' : 'text-gray-500 hover:text-cyan-600'}`}>
        <CalendarDays strokeWidth={2} size={24} />
        <span className="text-xs mt-1 font-semibold">行程</span>
      </button>
      <button onClick={() => setActiveTab('accounting')} className={`flex flex-col items-center transition-colors ${activeTab === 'accounting' ? 'text-cyan-700' : 'text-gray-500 hover:text-cyan-600'}`}>
        <Wallet strokeWidth={2} size={24} />
        <span className="text-xs mt-1 font-semibold">記帳</span>
      </button>
      <button onClick={() => setActiveTab('checklist')} className={`flex flex-col items-center transition-colors ${activeTab === 'checklist' ? 'text-cyan-700' : 'text-gray-500 hover:text-cyan-600'}`}>
        <CheckSquare strokeWidth={2} size={24} />
        <span className="text-xs mt-1 font-semibold">備忘</span>
      </button>
    </div>
  );

  const DaySelector = () => {
    const totalDays = Array.from({ length: 7 }, (_, i) => i + 1);
    const dayData = itinerary;

    return (
        // 保持橫向滾動，減少間距，使用 hide-scrollbar 隱藏滾動條
        <div className="flex px-2 py-4 bg-white border-b border-gray-100 sticky top-[72px] z-[5] overflow-x-auto whitespace-nowrap gap-1.5 justify-start hide-scrollbar">
            {totalDays.map(day => {
                const data = dayData.find(d => d.day === day);
                const isActive = day === selectedDay;
                
                return (
                    <button
                        key={day}
                        onClick={() => setSelectedDay(day)}
                        // 縮小方塊尺寸
                        className={`
                            flex-shrink-0 p-2 w-[60px] rounded-xl shadow-md transition-all duration-200 
                            ${isActive 
                                ? 'bg-cyan-700 text-white font-bold' 
                                : 'bg-gray-100 text-gray-600 hover:bg-gray-200 font-medium'
                            }
                        `}
                    >
                        {/* 縮小文字尺寸 */}
                        <div className="text-base leading-none">Day {day}</div> 
                        <div className="text-[10px] opacity-90 mt-1">{data?.date} ({data?.dayOfWeek})</div>
                    </button>
                );
            })}
        </div>
    );
  };

  const ItineraryView = () => {
    const dayData = itinerary.find(d => d.day === selectedDay);
    
    if (!dayData) return <div className="p-4 text-center text-gray-500">找不到此日行程。</div>;

    return (
      <div className="pb-24 animate-in fade-in bg-gray-100 min-h-screen">
        
        {/* 1. Day Summary Block */}
        <div className="p-4 bg-teal-500 text-white shadow-lg">
            <div className="flex justify-between items-start">
                <div>
                    {/* 地點標題 */}
                    <h2 className="text-xl font-bold tracking-wider">{dayData.location}</h2>
                    
                    {/* 日期與焦點 */}
                    <p className="text-sm mt-1 opacity-90">
                        {dayData.date}/{dayData.dayOfWeek} 今日重點：{dayData.focus}
                    </p>
                    
                    {/* 模擬用戶 ID */}
                    <p className="text-xs mt-2 opacity-70">
                        用戶ID (用於儲存): 01455216625324405696
                    </p>
                </div>
                
                {/* 天氣顯示 */}
                <div className="flex items-center gap-2 text-white/90">
                    {getWeatherIcon(dayData.weather)}
                    <span className="text-base font-semibold">{dayData.weather.split(' ')[0]} {dayData.weather.split(' ')[1]}</span>
                </div>
            </div>
        </div>

        {/* Activity List with Timeline */}
        <div className="p-4 pt-6 relative">
            {dayData.activities.map((act, activityIdx) => {
                const isLast = activityIdx === dayData.activities.length - 1;
                // 注意：這裡使用 act.icon，如果它是從 window.lucide 傳下來的，它會是一個 React 組件
                // 修正：將默認的 Map 替換為 MapIcon
                const Icon = act.icon || MapIcon; 
                
                return (
                    <div key={activityIdx} className="relative pb-8 pl-8">
                        
                        {/* 垂直線 (Timeline Line) */}
                        {!isLast && (
                            <div className="absolute top-0 left-2.5 h-full w-0.5 bg-gray-300"></div>
                        )}
                        
                        {/* 圓點 (Timeline Dot) */}
                        <div className="absolute left-0 top-0 w-5 h-5 flex items-center justify-center">
                            <div className="w-3 h-3 bg-cyan-700 rounded-full shadow-md"></div>
                        </div>

                        {/* 卡片內容 */}
                        <div className="bg-white p-4 rounded-xl shadow-md border-t-4 border-gray-100">
                            {isEditingItinerary ? (
                                // 編輯模式
                                <div className="space-y-3">
                                     <div className="flex justify-end gap-2 mb-3">
                                        <button 
                                            onClick={() => removeActivity(dayData.day, activityIdx)}
                                            className="text-red-500 hover:text-red-700 p-1"
                                            title="刪除此行程"
                                        >
                                            <Trash2 size={20} />
                                        </button>
                                     </div>
                                    <div className="flex gap-2">
                                        <input 
                                            value={act.time} 
                                            onChange={(e)=>updateActivity(dayData.day, activityIdx, 'time', e.target.value)}
                                            className="w-16 bg-gray-50 border border-gray-300 p-2 text-sm text-center rounded focus:border-cyan-700 focus:ring-1 focus:ring-cyan-700"
                                        />
                                        <input 
                                            value={act.type} 
                                            onChange={(e)=>updateActivity(dayData.day, activityIdx, 'type', e.target.value)}
                                            className="flex-1 bg-gray-50 border border-gray-300 p-2 text-sm text-center rounded focus:border-cyan-700 focus:ring-1 focus:ring-cyan-700"
                                            placeholder="類型 (e.g., 交通, 美食)"
                                        />
                                    </div>
                                    <input 
                                        value={act.title} 
                                        onChange={(e)=>updateActivity(dayData.day, activityIdx, 'title', e.target.value)}
                                        className="w-full bg-gray-50 border border-gray-300 p-2 font-medium rounded focus:border-cyan-700 focus:ring-1 focus:ring-cyan-700" 
                                        placeholder="標題"
                                    />
                                    <textarea 
                                        value={act.desc} 
                                        onChange={(e)=>updateActivity(dayData.day, activityIdx, 'desc', e.target.value)}
                                        className="w-full bg-gray-50 border border-gray-300 p-2 text-sm rounded focus:border-cyan-700 focus:ring-1 focus:ring-cyan-700" 
                                        rows={2}
                                        placeholder="描述"
                                    />
                                </div>
                            ) : (
                                // 顯示模式
                                <div>
                                    <div className="flex justify-between items-start mb-2">
                                        <div className="flex items-center gap-3">
                                            {/* 時間與類型標籤 */}
                                            <div className="flex items-center text-sm font-semibold text-gray-600">
                                                <Clock size={16} className="mr-1 text-teal-600"/>
                                                {act.time}
                                            </div>
                                            <div className={`${getTypeColor(act.type)} text-xs px-2 py-0.5 rounded-full font-bold uppercase flex items-center gap-1`}>
                                                {/* 確保 Icon 是一個有效的 React 組件 */}
                                                <Icon size={14}/>
                                                {act.type}
                                            </div>
                                        </div>
                                        {/* 編輯/刪除按鈕 */}
                                        <div className="flex gap-1 text-gray-400">
                                            <Edit2 size={20} className="hover:text-cyan-700 cursor-pointer" onClick={() => setIsEditingItinerary(true)}/>
                                            <Trash2 size={20} className="hover:text-red-500 cursor-pointer" onClick={() => removeActivity(dayData.day, activityIdx)}/>
                                        </div>
                                    </div>
                                    
                                    {/* 標題與描述 */}
                                    <h3 className="text-lg font-bold text-gray-800 mt-2">{act.title}</h3>
                                    <p className="text-sm text-gray-600 mt-1 font-light leading-relaxed">{act.desc}</p>
                                    
                                    {/* 模擬圖片中的地圖查詢連結 */}
                                    <div className="mt-3 pt-2 border-t border-dashed border-gray-100">
                                        <button className="text-xs text-teal-600 font-semibold flex items-center gap-1 hover:text-teal-700">
                                            {/* 修正：使用 MapIcon */}
                                            <MapIcon size={14} strokeWidth={2.5}/> 開啟地圖查詢
                                        </button>
                                    </div>
                                </div>
                            )}
                        </div>
                    </div>
                );
            })}
            
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
        
        {/* 全局編輯控制按鈕 */}
        <div className="px-4 pt-4 flex gap-3 pb-8">
            {isEditingItinerary ? (
                <button 
                    onClick={() => setIsEditingItinerary(false)}
                    className="flex-1 py-3 bg-teal-600 text-white text-base rounded-lg shadow-xl hover:bg-teal-700 transition-colors font-bold"
                >
                    完成編輯
                </button>
            ) : (
                <button 
                    onClick={() => setIsEditingItinerary(true)}
                    className="flex-1 py-3 bg-cyan-700 text-white text-base rounded-lg shadow-xl hover:bg-cyan-800 flex items-center justify-center gap-2 transition-colors font-bold"
                >
                    <Edit2 size={18}/> 編輯 Day {dayData.day}
                </button>
            )}
        </div>

      </div>
    );
  };

  const AccountingView = () => {
    // 必須在函數內部聲明，因為它依賴於組件狀態 `itinerary`
    const currentDayData = itinerary.find(d => d.day === selectedDay);
    const initialDate = currentDayData ? currentDayData.fullDate : "2025-05-01";
    
    // 如果 newItem 狀態還沒有初始化，則使用初始日期
    // 這裡我們在外部已經初始化了 newItem，但為了確保日期正確，我們可以在這裡處理一下
    useEffect(() => {
        // 如果使用者切換了日期，我們更新 newItem 的預設日期
        if(currentDayData && newItem.date !== currentDayData.fullDate && !showAddExpense) {
            setNewItem(prev => ({...prev, date: currentDayData.fullDate}));
        }
    }, [selectedDay, currentDayData, showAddExpense]);


    const groupedExpenses = useMemo(() => {
        return expenses.reduce((groups, item) => {
            const date = item.date;
            if (!groups[date]) groups[date] = [];
            groups[date].push(item);
            return groups;
        }, {});
    }, [expenses]);
    

    const sortedDates = Object.keys(groupedExpenses).sort().reverse(); 

    const totalKRW = expenses.reduce((sum, item) => sum + Number(item.amount), 0);
    
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
                        onClick={() => {
                            setNewItem(prev => ({...prev, date: initialDate})); // 重置日期
                            setShowAddExpense(true);
                        }}
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
                                                <Trash2 size={16} strokeWidth={2.5}/>
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
                                    onChange={e => setNewItem(prev => ({...prev, date: e.target.value}))}
                                    className="w-full border-2 border-gray-300 p-2 rounded-md focus:outline-none focus:border-cyan-700 bg-gray-50"
                                />
                            </div>
                            <div>
                                <label className="block text-xs text-gray-500 mb-1 font-semibold">項目</label>
                                <input 
                                    type="text" 
                                    placeholder="晚餐、門票..."
                                    value={newItem.title}
                                    onChange={e => setNewItem(prev => ({...prev, title: e.target.value}))}
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
                                    onChange={e => setNewItem(prev => ({...prev, amount: e.target.value}))}
                                    className="w-full border-2 border-gray-300 p-2 text-xl font-mono rounded-md focus:outline-none focus:border-cyan-700 bg-gray-50"
                                />
                                {newItem.amount && (
                                    <p className="text-right text-xs text-gray-500 mt-1 font-mono">
                                        約 TWD {Math.round(Number(newItem.amount) / exchangeRate).toLocaleString()}
                                    </p>
                                )}
                            </div>
                             <div>
                                <label className="block text-xs text-gray-500 mb-1 font-semibold">備註 (可選)</label>
                                <input 
                                    type="text" 
                                    value={newItem.note}
                                    onChange={e => setNewItem(prev => ({...prev, note: e.target.value}))}
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
    // 總進度計算
    const totalItems = checklist.length;
    const checkedItems = checklist.filter(i => i.checked).length;
    const progress = totalItems > 0 ? Math.round((checkedItems / totalItems) * 100) : 0;
    
    // Checklist new item state is handled by the outer component state 'newItemText'

    return (
        <div className="pb-24 bg-gray-100 min-h-screen px-4 py-6 animate-in fade-in">
            {/* 跨國緊急聯絡資訊區塊 */}
            <div className="bg-red-600 text-white p-6 rounded-xl shadow-lg mb-6">
                <h2 className="text-xl font-bold tracking-wide flex items-center gap-3 mb-4 border-b border-red-400 pb-2">
                    <PhoneCall size={24} strokeWidth={2.5}/> 跨國緊急聯絡資訊
                </h2>
                
                <div className="space-y-4">
                    {/* Item 1: Korea Emergency */}
                    <div className="flex justify-between items-center text-sm border-b border-red-400/50 pb-2">
                        <span className="font-semibold text-base">韓國緊急報案/火警/急救</span>
                        <div className="text-right">
                           <p className="font-extrabold text-lg tracking-wider">112 / 119</p>
                           <p className="text-xs opacity-80">(在韓國境內直撥)</p>
                        </div>
                    </div>

                    {/* Item 2: Korea Travel Hotline */}
                    <div className="flex justify-between items-center text-sm border-b border-red-400/50 pb-2">
                        <span className="font-semibold text-base">韓國旅遊諮詢熱線 (中文)</span>
                        <a href="tel:1330" className="font-extrabold text-lg tracking-wider underline hover:text-red-200">
                            1330
                        </a>
                    </div>
                    
                    {/* Item 3: Taiwan Embassy */}
                    <div className="flex justify-between items-center text-sm border-b border-red-400/50 pb-2">
                        <span className="font-semibold text-base">台灣駐韓國代表部</span>
                        <a href="tel:+8223992780" className="font-extrabold text-lg tracking-wider underline hover:text-red-200">
                            +82-2-399-2780
                        </a>
                    </div>

                    {/* Item 4: MOFA */}
                    <div className="flex justify-between items-center text-sm pt-1">
                        <span className="font-semibold text-base">外交部旅外國人急難救助</span>
                        <a href="tel:+886800085095" className="font-extrabold text-lg tracking-wider underline hover:text-red-200">
                            +886-800-085-095
                        </a>
                    </div>
                </div>
            </div>

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
            
            {/* 新增項目 UI */}
            <div className="p-4 bg-white rounded-xl shadow-md border-t-4 border-cyan-700 mb-6">
                <h3 className="text-lg font-bold text-cyan-700 mb-3">新增備忘項目</h3>
                <div className="flex gap-2">
                    <input 
                        type="text"
                        value={newItemText}
                        onChange={(e) => setNewItemText(e.target.value)}
                        onKeyDown={(e) => {
                            if (e.key === 'Enter' && newItemText.trim()) {
                                handleAddItem(newItemText);
                                setNewItemText('');
                            }
                        }}
                        placeholder="輸入新項目..."
                        className="flex-1 border-2 border-gray-300 p-2 rounded-lg focus:outline-none focus:border-teal-600"
                    />
                    <button
                        onClick={() => {
                            handleAddItem(newItemText);
                            setNewItemText('');
                        }}
                        className="bg-teal-600 text-white px-4 py-2 rounded-lg font-bold shadow-lg hover:bg-teal-700 transition-colors flex items-center gap-1 active:scale-95 disabled:opacity-50"
                        disabled={!newItemText.trim()}
                    >
                        <Plus size={20}/> 新增
                    </button>
                </div>
            </div>

            {/* 簡化清單項目列表 (含刪除功能) */}
            <div className="bg-white rounded-xl shadow-md border-t-4 border-teal-500 overflow-hidden">
                <div className="p-4 bg-gray-50 border-b border-gray-200">
                    <h3 className="text-lg font-bold text-gray-700">完整備忘清單</h3>
                </div>
                
                <div className="divide-y divide-gray-100 px-4">
                    {checklist.map(item => (
                        <div 
                            key={item.id} 
                            className={`py-3 flex items-center justify-between transition-colors ${
                                item.checked ? 'bg-white' : 'hover:bg-gray-50'
                            }`}
                        >
                            <div className="flex items-center gap-4 cursor-pointer flex-1" onClick={() => toggleCheck(item.id)}>
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
                             
                            {/* 刪除按鈕 */}
                            <button
                                onClick={(e) => {
                                    e.stopPropagation(); // 防止點擊刪除時觸發勾選
                                    handleRemoveItem(item.id);
                                }}
                                className="text-gray-300 hover:text-red-500 transition-colors p-1 flex-shrink-0"
                                title="移除此項目"
                            >
                                <X size={20} strokeWidth={2}/>
                            </button>
                        </div>
                    ))}
                </div>
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
      
      {/* 只有在行程頁面顯示 Day Selector */}
      {activeTab === 'itinerary' && <DaySelector />}

      <main>
        {activeTab === 'itinerary' && <ItineraryView />}
        {activeTab === 'accounting' && <AccountingView />}
        {activeTab === 'checklist' && <ChecklistView />}
      </main>
      
      {/* Footer for Version */}
      <div className="text-center py-2 text-xs text-gray-400 font-mono bg-white border-t sticky bottom-16 z-10">
        濟州慢旅規劃 App Version: {APP_VERSION}
      </div>

      <BottomNav />
    </div>
  );
}

// 啟動 React 應用程式
const root = ReactDOM.createRoot(document.getElementById('root'));
root.render(<JejuFinalStyleApp />);
</script>

</body>
</html>
