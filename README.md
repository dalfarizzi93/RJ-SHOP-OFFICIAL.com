<html lang="id">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>RJ SHOP - Dashboard ERP Terpadu</title>
    <!-- Tailwind CSS -->
    <script src="https://cdn.tailwindcss.com"></script>
    <!-- React & ReactDOM -->
    <script src="https://unpkg.com/react@18/umd/react.production.min.js"></script>
    <script src="https://unpkg.com/react-dom@18/umd/react-dom.production.min.js"></script>
    <!-- Babel for JSX -->
    <script src="https://unpkg.com/@babel/standalone/babel.min.js"></script>
    <!-- Lucide Icons -->
    <script src="https://unpkg.com/lucide@latest"></script>
    <!-- Chart.js -->
    <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
    <style>
        @import url('https://fonts.googleapis.com/css2?family=Plus+Jakarta+Sans:wght@300;400;500;600;700;800&display=swap');
        body { font-family: 'Plus Jakarta Sans', sans-serif; overflow: hidden; }
        .custom-scrollbar::-webkit-scrollbar { width: 4px; }
        .custom-scrollbar::-webkit-scrollbar-track { background: transparent; }
        .custom-scrollbar::-webkit-scrollbar-thumb { background: #e2e8f0; border-radius: 10px; }
        .fade-in { animation: fadeIn 0.3s ease-in-out; }
        @keyframes fadeIn { from { opacity: 0; transform: translateY(5px); } to { opacity: 1; transform: translateY(0); } }
        .glass { background: rgba(255, 255, 255, 0.8); backdrop-filter: blur(10px); }
    </style>
</head>
<body class="bg-slate-50">
    <div id="root"></div>

    <script type="text/babel">
        const { useState, useEffect, useMemo, useRef } = React;

        // --- Shared Components ---
        const Icon = ({ name, size = 20, className = "" }) => {
            const LucideIcon = lucide.icons[name];
            if (!LucideIcon) return null;
            return <i data-lucide={name} className={className} style={{ width: size, height: size }}></i>;
        };

        const Badge = ({ status }) => {
            const styles = {
                'On Progress': 'bg-blue-100 text-blue-700',
                'Completed': 'bg-emerald-100 text-emerald-700',
                'Delayed': 'bg-red-100 text-red-700',
                'Urgent': 'bg-orange-100 text-orange-700',
                'Safe': 'bg-emerald-100 text-emerald-700',
                'Low': 'bg-amber-100 text-amber-700',
                'Out of Stock': 'bg-red-100 text-red-700',
            };
            return (
                <span className={`px-2 py-0.5 rounded-full text-[10px] font-bold uppercase tracking-wider ${styles[status] || 'bg-slate-100 text-slate-600'}`}>
                    {status}
                </span>
            );
        };

        // --- Main App ---
        const App = () => {
            const [isLoggedIn, setIsLoggedIn] = useState(false);
            const [activeTab, setActiveTab] = useState('Dashboard');
            const [isSidebarOpen, setSidebarOpen] = useState(false);
            const [notifications, setNotifications] = useState([
                { id: 1, text: "Stok Kain Hitam menipis!", time: "5 mnt lalu", read: false },
                { id: 2, text: "SPK-003 telah selesai jahit", time: "1 jam lalu", read: false }
            ]);
            const [showNotifPanel, setShowNotifPanel] = useState(false);
            const [showSettings, setShowSettings] = useState(false);

            // Real-time Linked States
            const [production, setProduction] = useState([
                { id: 'SPK-001', item: 'T-Shirt Cotton', qty: 500, stage: 'Jahit', progress: 65, status: 'On Progress' },
                { id: 'SPK-002', item: 'Hoodie Black', qty: 200, stage: 'Sablon', progress: 30, status: 'Delayed' },
                { id: 'SPK-003', item: 'Chino Slim', qty: 350, stage: 'Packing', progress: 100, status: 'Completed' },
            ]);

            const [stock, setStock] = useState([
                { id: 1, name: 'Cotton Combed 30s Black', qty: 120, unit: 'Roll', status: 'Safe' },
                { id: 2, name: 'Benang Putih', qty: 5, unit: 'Box', status: 'Low' },
                { id: 3, name: 'Label RJ SHOP', qty: 50, unit: 'Pack', status: 'Safe' },
            ]);

            const [finances, setFinances] = useState({
                omset: 145000000,
                expenses: 85000000,
                profit: 60000000
            });

            useEffect(() => {
                lucide.createIcons();
            }, [activeTab, isLoggedIn, isSidebarOpen, showNotifPanel, showSettings]);

            // Chart Initialization
            useEffect(() => {
                if (isLoggedIn && activeTab === 'Dashboard') {
                    const ctx = document.getElementById('mainChart');
                    if (ctx) {
                        const chart = new Chart(ctx, {
                            type: 'line',
                            data: {
                                labels: ['Minggu 1', 'Minggu 2', 'Minggu 3', 'Minggu 4'],
                                datasets: [{
                                    label: 'Produksi Selesai',
                                    data: [400, 800, 600, 1200],
                                    borderColor: '#2563eb',
                                    backgroundColor: 'rgba(37, 99, 235, 0.1)',
                                    fill: true, tension: 0.4
                                }]
                            },
                            options: { maintainAspectRatio: false, plugins: { legend: { display: false } } }
                        });
                        return () => chart.destroy();
                    }
                }
            }, [activeTab, isLoggedIn]);

            const handleNewOrder = () => {
                // Logic Simulasi: Menambah Order Baru & Mengurangi Stok
                const newId = `SPK-00${production.length + 1}`;
                setProduction([...production, { id: newId, item: 'Custom Order', qty: 100, stage: 'Cutting', progress: 0, status: 'On Progress' }]);
                setNotifications([{ id: Date.now(), text: `Order Baru ${newId} ditambahkan`, time: "Baru saja", read: false }, ...notifications]);
                // Kurangi stok benang sebagai simulasi
                setStock(stock.map(s => s.id === 2 ? {...s, qty: Math.max(0, s.qty - 1)} : s));
            };

            if (!isLoggedIn) {
                return (
                    <div className="min-h-screen bg-slate-900 flex items-center justify-center p-4">
                        <div className="w-full max-w-md bg-white rounded-3xl shadow-2xl overflow-hidden fade-in p-10">
                            <div className="text-center mb-8">
                                <div className="w-20 h-20 bg-blue-600 rounded-2xl flex items-center justify-center mx-auto mb-4 shadow-xl rotate-3">
                                    <Icon name="Scissors" size={40} className="text-white" />
                                </div>
                                <h1 className="text-3xl font-black text-slate-800 italic">RJ <span className="text-blue-600">SHOP</span></h1>
                                <p className="text-slate-500 mt-2">Sistem Manajemen Garment & Retail</p>
                            </div>
                            <form className="space-y-4" onSubmit={(e) => { e.preventDefault(); setIsLoggedIn(true); }}>
                                <input type="text" className="w-full px-5 py-4 rounded-2xl bg-slate-50 border border-slate-200 outline-none focus:ring-2 focus:ring-blue-500 transition-all" placeholder="Username" defaultValue="admin" />
                                <input type="password" className="w-full px-5 py-4 rounded-2xl bg-slate-50 border border-slate-200 outline-none focus:ring-2 focus:ring-blue-500 transition-all" placeholder="Password" defaultValue="password" />
                                <button className="w-full bg-blue-600 text-white py-4 rounded-2xl font-bold text-lg hover:bg-blue-700 shadow-lg shadow-blue-200 transition-all active:scale-95">Masuk Sekarang</button>
                            </form>
                        </div>
                    </div>
                );
            }

            return (
                <div className="flex h-screen bg-slate-50 overflow-hidden">
                    {/* SIDEBAR */}
                    <aside className={`fixed lg:static inset-y-0 left-0 z-50 w-72 bg-white border-r border-slate-100 flex flex-col transition-transform duration-300 lg:translate-x-0 ${isSidebarOpen ? 'translate-x-0' : '-translate-x-full'}`}>
                        <div className="p-8 flex items-center justify-between">
                            <div className="flex items-center space-x-3">
                                <div className="w-8 h-8 bg-blue-600 rounded-lg flex items-center justify-center text-white font-bold">R</div>
                                <span className="text-xl font-black tracking-tighter text-slate-800">RJ SHOP</span>
                            </div>
                            <button onClick={() => setSidebarOpen(false)} className="lg:hidden"><Icon name="X" /></button>
                        </div>

                        <nav className="flex-1 px-6 space-y-1 overflow-y-auto custom-scrollbar">
                            <p className="text-[10px] font-black text-slate-400 uppercase tracking-widest mb-4 px-2">Main Menu</p>
                            {[
                                { id: 'Dashboard', icon: 'LayoutDashboard' },
                                { id: 'Produksi', icon: 'Scissors' },
                                { id: 'Gudang', icon: 'Warehouse' },
                                { id: 'Master Produk', icon: 'Package' },
                                { id: 'Penjualan', icon: 'ShoppingCart' }
                            ].map(item => (
                                <button key={item.id} onClick={() => { setActiveTab(item.id); setSidebarOpen(false); }} className={`w-full flex items-center space-x-3 px-4 py-3 rounded-xl transition-all ${activeTab === item.id ? 'bg-blue-600 text-white shadow-lg shadow-blue-100' : 'text-slate-500 hover:bg-slate-50'}`}>
                                    <Icon name={item.icon} size={18} />
                                    <span className="text-sm font-bold">{item.id}</span>
                                </button>
                            ))}

                            <p className="text-[10px] font-black text-slate-400 uppercase tracking-widest mt-8 mb-4 px-2">Operational</p>
                            {[
                                { id: 'Keuangan', icon: 'DollarSign' },
                                { id: 'Pengaturan', icon: 'Settings' }
                            ].map(item => (
                                <button key={item.id} onClick={() => { setActiveTab(item.id); setSidebarOpen(false); }} className={`w-full flex items-center space-x-3 px-4 py-3 rounded-xl transition-all ${activeTab === item.id ? 'bg-blue-600 text-white shadow-lg' : 'text-slate-500 hover:bg-slate-50'}`}>
                                    <Icon name={item.icon} size={18} />
                                    <span className="text-sm font-bold">{item.id}</span>
                                </button>
                            ))}
                        </nav>

                        <div className="p-6 mt-auto">
                            <button onClick={() => setIsLoggedIn(false)} className="w-full flex items-center space-x-3 p-3 text-red-500 hover:bg-red-50 rounded-xl font-bold transition-all">
                                <Icon name="LogOut" size={18} />
                                <span className="text-sm">Keluar Sesi</span>
                            </button>
                        </div>
                    </aside>

                    {/* MAIN CONTENT */}
                    <main className="flex-1 flex flex-col min-w-0 overflow-hidden relative">
                        {/* HEADER */}
                        <header className="h-20 bg-white border-b border-slate-100 px-8 flex items-center justify-between shrink-0">
                            <div className="flex items-center space-x-4">
                                <button onClick={() => setSidebarOpen(true)} className="lg:hidden p-2 bg-slate-100 rounded-lg"><Icon name="Menu" /></button>
                                <h2 className="text-xl font-bold text-slate-800">{activeTab}</h2>
                            </div>

                            <div className="flex items-center space-x-3">
                                {/* Notifikasi */}
                                <div className="relative">
                                    <button onClick={() => setShowNotifPanel(!showNotifPanel)} className="p-3 bg-slate-50 text-slate-600 rounded-2xl hover:bg-slate-100 relative transition-all">
                                        <Icon name="Bell" size={20} />
                                        <span className="absolute top-2 right-2 w-2 h-2 bg-red-500 rounded-full border-2 border-white"></span>
                                    </button>
                                    
                                    {showNotifPanel && (
                                        <div className="absolute right-0 mt-3 w-80 bg-white rounded-2xl shadow-2xl border border-slate-100 z-50 fade-in overflow-hidden">
                                            <div className="p-4 border-b border-slate-50 font-bold flex justify-between items-center">
                                                <span>Notifikasi Terkini</span>
                                                <button onClick={() => setShowNotifPanel(false)}><Icon name="X" size={16}/></button>
                                            </div>
                                            <div className="max-h-64 overflow-y-auto custom-scrollbar">
                                                {notifications.map(n => (
                                                    <div key={n.id} className="p-4 hover:bg-slate-50 border-b border-slate-50 last:border-0 cursor-pointer">
                                                        <p className="text-xs font-bold text-slate-800">{n.text}</p>
                                                        <p className="text-[10px] text-slate-400 mt-1">{n.time}</p>
                                                    </div>
                                                ))}
                                            </div>
                                        </div>
                                    )}
                                </div>

                                <button onClick={() => setActiveTab('Pengaturan')} className="p-3 bg-slate-50 text-slate-600 rounded-2xl hover:bg-slate-100 transition-all"><Icon name="Settings" size={20} /></button>
                                
                                <div className="hidden sm:flex items-center space-x-3 ml-4 border-l pl-6">
                                    <div className="text-right">
                                        <p className="text-xs font-black text-slate-800">Admin RJ</p>
                                        <p className="text-[10px] text-emerald-500 font-bold">Online</p>
                                    </div>
                                    <div className="w-10 h-10 bg-indigo-100 text-indigo-600 rounded-xl flex items-center justify-center font-bold">A</div>
                                </div>
                            </div>
                        </header>

                        {/* CONTENT AREA */}
                        <div className="flex-1 overflow-y-auto p-8 custom-scrollbar">
                            <div className="max-w-7xl mx-auto space-y-6">
                                
                                {activeTab === 'Dashboard' && (
                                    <div className="space-y-6 fade-in">
                                        {/* Stats */}
                                        <div className="grid grid-cols-1 md:grid-cols-3 gap-6">
                                            <div className="bg-white p-6 rounded-3xl shadow-sm border border-slate-100">
                                                <p className="text-xs font-bold text-slate-400 uppercase tracking-widest">Total Omset</p>
                                                <h3 className="text-2xl font-black text-slate-800 mt-2">Rp {finances.omset.toLocaleString('id-ID')}</h3>
                                                <div className="flex items-center mt-2 text-emerald-500 text-[10px] font-bold bg-emerald-50 w-fit px-2 py-0.5 rounded-lg">
                                                    <Icon name="TrendingUp" size={12} className="mr-1" /> +12% dari bln lalu
                                                </div>
                                            </div>
                                            <div className="bg-white p-6 rounded-3xl shadow-sm border border-slate-100">
                                                <p className="text-xs font-bold text-slate-400 uppercase tracking-widest">Produksi Aktif</p>
                                                <h3 className="text-2xl font-black text-slate-800 mt-2">{production.filter(p => p.status !== 'Completed').length} Order</h3>
                                                <div className="flex items-center mt-2 text-blue-500 text-[10px] font-bold bg-blue-50 w-fit px-2 py-0.5 rounded-lg">
                                                    85% Kapasitas Terpakai
                                                </div>
                                            </div>
                                            <div className="bg-white p-6 rounded-3xl shadow-sm border border-slate-100">
                                                <p className="text-xs font-bold text-slate-400 uppercase tracking-widest">Keuntungan Bersih</p>
                                                <h3 className="text-2xl font-black text-emerald-600 mt-2">Rp {finances.profit.toLocaleString('id-ID')}</h3>
                                                <div className="w-full bg-slate-100 h-1 mt-4 rounded-full">
                                                    <div className="bg-emerald-500 h-full w-3/4 rounded-full"></div>
                                                </div>
                                            </div>
                                        </div>

                                        <div className="grid grid-cols-1 lg:grid-cols-2 gap-6">
                                            <div className="bg-white p-8 rounded-3xl shadow-sm border border-slate-100">
                                                <div className="flex justify-between items-center mb-6">
                                                    <h3 className="font-bold text-slate-800">Performa Produksi</h3>
                                                    <button onClick={handleNewOrder} className="text-xs bg-blue-600 text-white px-4 py-2 rounded-xl font-bold shadow-lg shadow-blue-100 hover:scale-105 transition-all">Simulasi Order Baru</button>
                                                </div>
                                                <div className="h-64"><canvas id="mainChart"></canvas></div>
                                            </div>
                                            
                                            <div className="bg-white p-8 rounded-3xl shadow-sm border border-slate-100">
                                                <h3 className="font-bold text-slate-800 mb-6">Stok Menipis</h3>
                                                <div className="space-y-4">
                                                    {stock.filter(s => s.status !== 'Safe' || s.qty < 10).map(s => (
                                                        <div key={s.id} className="flex items-center justify-between p-4 bg-slate-50 rounded-2xl">
                                                            <div className="flex items-center space-x-3">
                                                                <div className="p-2 bg-white rounded-lg shadow-sm"><Icon name="Package" size={16} className="text-amber-500" /></div>
                                                                <div>
                                                                    <p className="text-sm font-bold text-slate-800">{s.name}</p>
                                                                    <p className="text-[10px] text-slate-400">Sisa: {s.qty} {s.unit}</p>
                                                                </div>
                                                            </div>
                                                            <button className="text-[10px] font-bold text-blue-600 px-3 py-1 bg-blue-50 rounded-lg hover:bg-blue-100">Restock</button>
                                                        </div>
                                                    ))}
                                                </div>
                                            </div>
                                        </div>
                                    </div>
                                )}

                                {activeTab === 'Produksi' && (
                                    <div className="bg-white rounded-3xl shadow-sm border border-slate-100 overflow-hidden fade-in">
                                        <div className="p-6 border-b border-slate-50 flex justify-between items-center">
                                            <h3 className="font-bold">Antrian Produksi RJ SHOP</h3>
                                            <div className="flex space-x-2">
                                                <div className="flex items-center bg-slate-50 rounded-xl px-3 py-2 border border-slate-100">
                                                    <Icon name="Search" size={16} className="text-slate-400 mr-2" />
                                                    <input type="text" placeholder="Cari SPK..." className="bg-transparent outline-none text-xs w-40" />
                                                </div>
                                            </div>
                                        </div>
                                        <table className="w-full text-left">
                                            <thead className="bg-slate-50 text-[10px] uppercase font-black text-slate-400">
                                                <tr>
                                                    <th className="px-8 py-4">Kode SPK</th>
                                                    <th className="px-8 py-4">Nama Produk</th>
                                                    <th className="px-8 py-4">Qty</th>
                                                    <th className="px-8 py-4">Tahapan</th>
                                                    <th className="px-8 py-4">Progress</th>
                                                    <th className="px-8 py-4">Status</th>
                                                </tr>
                                            </thead>
                                            <tbody className="divide-y divide-slate-50">
                                                {production.map(p => (
                                                    <tr key={p.id} className="hover:bg-slate-50 transition-all group">
                                                        <td className="px-8 py-5 font-bold text-blue-600 text-sm cursor-pointer underline-offset-4 hover:underline">{p.id}</td>
                                                        <td className="px-8 py-5 text-sm font-medium text-slate-700">{p.item}</td>
                                                        <td className="px-8 py-5 text-sm font-bold">{p.qty}</td>
                                                        <td className="px-8 py-5 text-xs text-slate-500 font-bold">{p.stage}</td>
                                                        <td className="px-8 py-5">
                                                            <div className="w-full max-w-[100px] bg-slate-100 h-1.5 rounded-full overflow-hidden">
                                                                <div className={`h-full transition-all duration-1000 ${p.progress === 100 ? 'bg-emerald-500' : 'bg-blue-600'}`} style={{ width: `${p.progress}%` }}></div>
                                                            </div>
                                                        </td>
                                                        <td className="px-8 py-5"><Badge status={p.status} /></td>
                                                    </tr>
                                                ))}
                                            </tbody>
                                        </table>
                                    </div>
                                )}

                                {activeTab === 'Pengaturan' && (
                                    <div className="max-w-2xl mx-auto space-y-6 fade-in">
                                        <div className="bg-white p-8 rounded-3xl shadow-sm border border-slate-100">
                                            <h3 className="font-bold mb-6">Profil Toko RJ SHOP</h3>
                                            <div className="space-y-4">
                                                <div>
                                                    <label className="block text-[10px] font-black uppercase text-slate-400 mb-2">Nama Aplikasi</label>
                                                    <input type="text" className="w-full p-4 bg-slate-50 border border-slate-100 rounded-2xl outline-none" defaultValue="RJ SHOP ERP" />
                                                </div>
                                                <div>
                                                    <label className="block text-[10px] font-black uppercase text-slate-400 mb-2">Email Notifikasi</label>
                                                    <input type="text" className="w-full p-4 bg-slate-50 border border-slate-100 rounded-2xl outline-none" defaultValue="owner@rjshop.id" />
                                                </div>
                                                <div className="flex items-center justify-between p-4 bg-blue-50 rounded-2xl">
                                                    <div>
                                                        <p className="text-sm font-bold text-blue-800">Mode Real-time</p>
                                                        <p className="text-[10px] text-blue-600">Sinkronisasi data otomatis aktif</p>
                                                    </div>
                                                    <div className="w-12 h-6 bg-blue-600 rounded-full relative cursor-pointer shadow-inner">
                                                        <div className="absolute right-1 top-1 w-4 h-4 bg-white rounded-full shadow-md"></div>
                                                    </div>
                                                </div>
                                            </div>
                                            <button className="w-full mt-8 bg-slate-800 text-white py-4 rounded-2xl font-bold hover:bg-black transition-all">Simpan Konfigurasi</button>
                                        </div>
                                    </div>
                                )}

                                {['Gudang', 'Master Produk', 'Penjualan', 'Keuangan'].includes(activeTab) && (
                                    <div className="h-96 flex flex-col items-center justify-center bg-white rounded-3xl border-2 border-dashed border-slate-200 fade-in">
                                        <div className="w-24 h-24 bg-slate-50 rounded-full flex items-center justify-center text-slate-300 mb-6">
                                            <Icon name="Construction" size={48} />
                                        </div>
                                        <h3 className="text-xl font-black text-slate-800 italic">Modul {activeTab} RJ SHOP</h3>
                                        <p className="text-slate-400 mt-2 text-center px-8">Sedang dalam proses pengembangan backend untuk sinkronisasi database SQL otomatis.</p>
                                    </div>
                                )}
                            </div>
                        </div>

                        {/* MOBILE NAV BOTTOM */}
                        <nav className="lg:hidden bg-white border-t border-slate-100 px-6 py-4 flex justify-between items-center shrink-0">
                            {[
                                { id: 'Dashboard', icon: 'LayoutDashboard' },
                                { id: 'Produksi', icon: 'Scissors' },
                                { id: 'Penjualan', icon: 'ShoppingCart' },
                                { id: 'Pengaturan', icon: 'Settings' }
                            ].map(item => (
                                <button key={item.id} onClick={() => setActiveTab(item.id)} className={`flex flex-col items-center space-y-1 ${activeTab === item.id ? 'text-blue-600' : 'text-slate-400'}`}>
                                    <Icon name={item.icon} size={20} />
                                    <span className="text-[10px] font-bold">{item.id.substring(0,4)}</span>
                                </button>
                            ))}
                        </nav>
                    </main>
                    
                    {/* Overlay Sidebar Mobile */}
                    {isSidebarOpen && <div className="fixed inset-0 bg-slate-900/60 z-40 lg:hidden backdrop-blur-sm transition-all" onClick={() => setSidebarOpen(false)}></div>}
                </div>
            );
        };

        const root = ReactDOM.createRoot(document.getElementById('root'));
        root.render(<App />);
    </script>
</body>
</html>
