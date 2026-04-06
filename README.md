import React, { useState, useEffect } from 'react';
import { 
  Smartphone, 
  History, 
  ArrowRightLeft,
  Wallet,
  Zap,
  CheckCircle2,
  TrendingDown,
  User,
  Lock,
  LogIn,
  UserPlus,
  LogOut,
  Send,
  ArrowUpRight,
  X,
  Download,
  Share2
} from 'lucide-react';

const App = () => {
  // App States
  const [isLoggedIn, setIsLoggedIn] = useState(false);
  const [authMode, setAuthMode] = useState('login'); 
  const [activeTab, setActiveTab] = useState('topup'); 
  
  const [currentUser, setCurrentUser] = useState(null);
  const [users, setUsers] = useState([
    { phone: '09123456789', name: 'Demo User', password: '123', balance: 50000, transactions: [] },
    { phone: '09777777777', name: 'Kyaw Kyaw', password: '123', balance: 10000, transactions: [] }
  ]);

  // Auth States
  const [authPhone, setAuthPhone] = useState('');
  const [authName, setAuthName] = useState('');
  const [authPassword, setAuthPassword] = useState('');

  // Top-up/Transfer States
  const [targetPhone, setTargetPhone] = useState('');
  const [selectedOperator, setSelectedOperator] = useState('Atom');
  const [selectedAmount, setSelectedAmount] = useState(1000);
  
  const [receiverPhone, setReceiverPhone] = useState('');
  const [transferAmount, setTransferAmount] = useState('');
  const [foundRecipient, setFoundRecipient] = useState(null);

  // Receipt Modal State
  const [showReceipt, setShowReceipt] = useState(false);
  const [currentReceipt, setCurrentReceipt] = useState(null);

  useEffect(() => {
    if (isLoggedIn && currentUser) {
      setTargetPhone(currentUser.phone);
      detectOperator(currentUser.phone);
    }
  }, [isLoggedIn, currentUser]);

  // လက်ခံမယ့်သူ့ဖုန်းနံပါတ်ရိုက်ရင် နာမည်ရှာပေးခြင်း
  useEffect(() => {
    if (receiverPhone.length >= 7) {
      const recipient = users.find(u => u.phone === receiverPhone);
      setFoundRecipient(recipient || null);
    } else {
      setFoundRecipient(null);
    }
  }, [receiverPhone, users]);

  const operators = [
    { name: 'Atom', color: 'bg-blue-500' },
    { name: 'MPT', color: 'bg-yellow-500' },
    { name: 'Ooredoo', color: 'bg-red-600' },
    { name: 'MyTel', color: 'bg-cyan-500' }
  ];

  const detectOperator = (phone) => {
    if (!phone) return;
    if (/^09(2|4|5|8)/.test(phone)) setSelectedOperator('MPT');
    else if (/^09(9|7)/.test(phone)) setSelectedOperator('Atom');
    else if (/^096/.test(phone)) setSelectedOperator('MyTel');
  };

  const generateTxId = () => "TXN" + Math.floor(100000 + Math.random() * 900000).toString();

  // --- Auth Handlers ---
  const handleSignUp = (e) => {
    e.preventDefault();
    if (users.find(u => u.phone === authPhone)) {
      alert("ဤဖုန်းနံပါတ်ဖြင့် အကောင့်ရှိပြီးသားဖြစ်သည်");
      return;
    }
    const newUser = { phone: authPhone, name: authName, password: authPassword, balance: 50000, transactions: [] };
    setUsers([...users, newUser]);
    alert("အကောင့်ဖွင့်ခြင်းအောင်မြင်ပါသည်");
    setAuthMode('login');
  };

  const handleLogin = (e) => {
    e.preventDefault();
    const user = users.find(u => u.phone === authPhone && u.password === authPassword);
    if (user) {
      setCurrentUser(user);
      setIsLoggedIn(true);
    } else {
      alert("ဖုန်းနံပါတ် သို့မဟုတ် Password မှားယွင်းနေပါသည်");
    }
  };

  const handleLogout = () => {
    setIsLoggedIn(false);
    setCurrentUser(null);
  };

  // --- Transaction Handlers ---
  const handleTransfer = (e) => {
    e.preventDefault();
    const amount = parseInt(transferAmount);
    if (!foundRecipient) return;
    if (currentUser.balance < amount) { alert("လက်ကျန်ငွေ မလုံလောက်ပါ"); return; }
    
    const txId = generateTxId();
    const now = new Date();
    const timestamp = now.toLocaleString();

    const txData = {
      id: txId,
      type: 'transfer',
      amount: amount,
      senderName: currentUser.name,
      senderPhone: currentUser.phone,
      receiverName: foundRecipient.name,
      receiverPhone: foundRecipient.phone,
      timestamp: timestamp,
    };

    // Update Users List (Sender & Receiver)
    setUsers(users.map(u => {
      if (u.phone === currentUser.phone) {
        const updated = { ...u, balance: u.balance - amount, transactions: [{ ...txData, label: `Sent to ${foundRecipient.name}`, isNegative: true }, ...u.transactions] };
        setCurrentUser(updated);
        return updated;
      }
      if (u.phone === foundRecipient.phone) {
        return { ...u, balance: u.balance + amount, transactions: [{ ...txData, label: `Received from ${currentUser.name}`, isNegative: false }, ...u.transactions] };
      }
      return u;
    }));

    setCurrentReceipt(txData);
    setShowReceipt(true);
    setReceiverPhone('');
    setTransferAmount('');
  };

  const handleBuyNow = (e) => {
    e.preventDefault();
    const amount = parseInt(selectedAmount);
    if (currentUser.balance < amount) { alert("လက်ကျန်ငွေ မလုံလောက်ပါ"); return; }
    
    const txId = generateTxId();
    const txData = {
      id: txId,
      type: 'topup',
      label: `${selectedOperator} Top-up (${targetPhone})`,
      amount: amount,
      timestamp: new Date().toLocaleString(),
      isNegative: true
    };

    const updatedUser = {
      ...currentUser,
      balance: currentUser.balance - amount,
      transactions: [txData, ...currentUser.transactions]
    };
    
    setCurrentUser(updatedUser);
    setUsers(users.map(u => u.phone === currentUser.phone ? updatedUser : u));
    setCurrentReceipt(txData);
    setShowReceipt(true);
  };

  // --- Receipt Modal Component ---
  const ReceiptModal = ({ tx, onClose }) => (
    <div className="fixed inset-0 bg-black/60 backdrop-blur-sm z-[100] flex items-center justify-center p-4">
      <div className="bg-white w-full max-w-sm rounded-[2.5rem] overflow-hidden shadow-2xl animate-in zoom-in-95 duration-200">
        <div className="bg-blue-600 p-6 text-white text-center relative">
          <button onClick={onClose} className="absolute right-4 top-4 text-white/50 hover:text-white"><X size={24}/></button>
          <div className="w-16 h-16 bg-white/20 rounded-full flex items-center justify-center mx-auto mb-3">
            <CheckCircle2 size={32} />
          </div>
          <h2 className="text-xl font-black">Transaction Success</h2>
          <p className="text-blue-100 text-xs mt-1">ပြေစာ (Electronic Receipt)</p>
        </div>

        <div className="p-8 space-y-4">
          <div className="flex justify-between items-center border-b border-dashed border-slate-200 pb-4">
            <span className="text-slate-400 text-xs font-bold uppercase">ID နံပါတ်</span>
            <span className="text-slate-800 font-mono font-bold text-sm">{tx.id}</span>
          </div>

          <div className="space-y-3">
            <div className="flex justify-between">
              <span className="text-slate-400 text-xs font-bold uppercase">နေ့စွဲနှင့် အချိန်</span>
              <span className="text-slate-800 font-bold text-xs">{tx.timestamp}</span>
            </div>
            
            <div className="flex justify-between">
              <span className="text-slate-400 text-xs font-bold uppercase">ပို့သူ</span>
              <div className="text-right">
                <p className="text-slate-800 font-bold text-xs">{tx.senderName || currentUser.name}</p>
                <p className="text-slate-400 text-[10px]">{tx.senderPhone || currentUser.phone}</p>
              </div>
            </div>

            <div className="flex justify-between">
              <span className="text-slate-400 text-xs font-bold uppercase">လက်ခံသူ</span>
              <div className="text-right">
                <p className="text-slate-800 font-bold text-xs">{tx.receiverName || tx.label}</p>
                <p className="text-slate-400 text-[10px]">{tx.receiverPhone || ''}</p>
              </div>
            </div>
          </div>

          <div className="bg-slate-50 p-4 rounded-2xl flex justify-between items-center">
            <span className="text-slate-500 font-bold text-xs">စုစုပေါင်း ပမာဏ</span>
            <span className="text-blue-600 font-black text-xl">{tx.amount.toLocaleString()} <small className="text-[10px]">MMK</small></span>
          </div>

          <div className="flex gap-2 pt-2">
            <button onClick={() => alert("Image saved to gallery")} className="flex-1 bg-slate-100 text-slate-700 font-bold py-3 rounded-xl text-xs flex items-center justify-center gap-2"><Download size={16}/> Save</button>
            <button className="flex-1 bg-blue-600 text-white font-bold py-3 rounded-xl text-xs flex items-center justify-center gap-2"><Share2 size={16}/> Share</button>
          </div>
        </div>
        <div className="bg-slate-50 p-4 text-center">
            <p className="text-[9px] text-slate-300 uppercase font-black tracking-widest italic">Thank you for using Quick-Pay</p>
        </div>
      </div>
    </div>
  );

  if (!isLoggedIn) {
    return (
      <div className="min-h-screen bg-slate-50 flex items-center justify-center p-4 font-sans">
        <div className="max-w-md w-full bg-white rounded-[2.5rem] p-8 shadow-xl border border-slate-100">
          <div className="flex flex-col items-center mb-8 text-blue-600">
            <Zap size={40} fill="currentColor" />
            <h1 className="text-2xl font-black mt-2 text-slate-800 tracking-tighter">QUICK-PAY</h1>
          </div>
          <div className="flex bg-slate-100 p-1 rounded-2xl mb-6">
            <button onClick={() => setAuthMode('login')} className={`flex-1 py-2 rounded-xl text-sm font-bold transition ${authMode === 'login' ? 'bg-white text-blue-600 shadow-sm' : 'text-slate-400'}`}>အကောင့်ဝင်</button>
            <button onClick={() => setAuthMode('signup')} className={`flex-1 py-2 rounded-xl text-sm font-bold transition ${authMode === 'signup' ? 'bg-white text-blue-600 shadow-sm' : 'text-slate-400'}`}>အကောင့်ဖွင့်</button>
          </div>
          <form onSubmit={authMode === 'login' ? handleLogin : handleSignUp} className="space-y-4">
            <input type="tel" placeholder="ဖုန်းနံပါတ်" value={authPhone} onChange={(e) => setAuthPhone(e.target.value)} className="w-full px-6 py-4 bg-slate-50 rounded-2xl outline-none focus:ring-2 focus:ring-blue-500 font-bold" required />
            {authMode === 'signup' && <input type="text" placeholder="နာမည်အပြည့်အစုံ" value={authName} onChange={(e) => setAuthName(e.target.value)} className="w-full px-6 py-4 bg-slate-50 rounded-2xl outline-none focus:ring-2 focus:ring-blue-500 font-bold" required />}
            <input type="password" placeholder="Password" value={authPassword} onChange={(e) => setAuthPassword(e.target.value)} className="w-full px-6 py-4 bg-slate-50 rounded-2xl outline-none focus:ring-2 focus:ring-blue-500 font-bold" required />
            <button className="w-full bg-blue-600 text-white font-bold py-4 rounded-2xl shadow-lg transition active:scale-95">{authMode === 'login' ? 'Login ဝင်မည်' : 'အကောင့်ဖွင့်မည်'}</button>
          </form>
        </div>
      </div>
    );
  }

  return (
    <div className="min-h-screen bg-slate-50 pb-24 font-sans">
      {showReceipt && <ReceiptModal tx={currentReceipt} onClose={() => setShowReceipt(false)} />}
      
      <nav className="bg-blue-600 text-white p-4 shadow-lg sticky top-0 z-50">
        <div className="max-w-md mx-auto flex justify-between items-center">
          <div className="flex items-center gap-2 font-black italic text-xl tracking-tighter">
            <Zap fill="currentColor" size={24} /> <span>QUICK-PAY</span>
          </div>
          <button onClick={handleLogout} className="bg-blue-700 p-2 rounded-xl hover:bg-blue-800 transition"><LogOut size={20} /></button>
        </div>
      </nav>

      <main className="max-w-md mx-auto p-4 space-y-6">
        {/* User Stats */}
        <div className="px-2 flex items-center justify-between">
          <div className="flex items-center gap-3">
            <div className="w-12 h-12 bg-blue-600 text-white rounded-2xl flex items-center justify-center font-black text-xl shadow-lg ring-4 ring-white">
              {currentUser?.name ? currentUser.name[0] : 'U'}
            </div>
            <div>
              <p className="text-[10px] font-black text-slate-400 uppercase tracking-widest">Welcome back,</p>
              <p className="text-base font-bold text-slate-700 leading-tight">{currentUser?.name}</p>
              <p className="text-blue-500 font-bold text-[10px]">{currentUser?.phone}</p>
            </div>
          </div>
        </div>

        {/* Card */}
        <div className="bg-gradient-to-br from-slate-900 to-slate-800 rounded-[2.5rem] p-8 text-white shadow-2xl relative overflow-hidden ring-1 ring-white/10">
          <div className="absolute right-[-20px] bottom-[-20px] opacity-10 rotate-12"><Wallet size={160} /></div>
          <p className="text-slate-400 text-xs font-bold mb-1 uppercase tracking-widest">Main Balance</p>
          <h1 className="text-4xl font-bold tracking-tight">
            {currentUser?.balance.toLocaleString()} <span className="text-lg font-normal text-slate-500">MMK</span>
          </h1>
        </div>

        {/* Tabs */}
        <div className="flex bg-white p-1 rounded-2xl border border-slate-200">
          <button onClick={() => setActiveTab('topup')} className={`flex-1 py-3 rounded-xl text-xs font-black uppercase tracking-widest flex items-center justify-center gap-2 transition ${activeTab === 'topup' ? 'bg-blue-600 text-white shadow-lg' : 'text-slate-400'}`}>
            <Smartphone size={16} /> ဘေဖြည့်
          </button>
          <button onClick={() => setActiveTab('transfer')} className={`flex-1 py-3 rounded-xl text-xs font-black uppercase tracking-widest flex items-center justify-center gap-2 transition ${activeTab === 'transfer' ? 'bg-blue-600 text-white shadow-lg' : 'text-slate-400'}`}>
            <Send size={16} /> ငွေလွှဲ
          </button>
        </div>

        {activeTab === 'topup' ? (
          <div className="bg-white rounded-[2rem] p-6 shadow-sm border border-slate-200 animate-in fade-in slide-in-from-bottom-2">
            <div className="grid grid-cols-4 gap-2 mb-8">
              {operators.map((op) => (
                <button key={op.name} onClick={() => setSelectedOperator(op.name)} className={`flex flex-col items-center gap-2 transition-all ${selectedOperator === op.name ? 'scale-110' : 'opacity-30 grayscale'}`}>
                  <div className={`w-14 h-14 rounded-2xl ${op.color} flex items-center justify-center text-white font-black shadow-lg text-lg`}>{op.name[0]}</div>
                  <span className="text-[10px] font-bold uppercase">{op.name}</span>
                </button>
              ))}
            </div>
            <form onSubmit={handleBuyNow} className="space-y-4">
              <input type="tel" value={targetPhone} onChange={(e) => setTargetPhone(e.target.value)} className="w-full bg-slate-50 p-4 rounded-2xl font-bold text-xl outline-none" required />
              <div className="grid grid-cols-3 gap-2">
                {[1000, 3000, 5000, 10000].map(amt => (
                  <button type="button" key={amt} onClick={() => setSelectedAmount(amt)} className={`py-3 rounded-xl text-xs font-bold border-2 transition ${selectedAmount === amt ? 'bg-blue-600 text-white border-blue-600' : 'bg-white text-slate-600 border-slate-100'}`}>{amt.toLocaleString()}</button>
                ))}
              </div>
              <button className="w-full bg-blue-600 text-white font-bold py-4 rounded-2xl shadow-xl flex items-center justify-center gap-2 active:scale-95 transition"><CheckCircle2 size={20} /> ဘေဖြည့်မည်</button>
            </form>
          </div>
        ) : (
          <div className="bg-white rounded-[2rem] p-6 shadow-sm border border-slate-200 animate-in fade-in slide-in-from-bottom-2">
            <form onSubmit={handleTransfer} className="space-y-4">
              <div className="space-y-2">
                <input type="tel" placeholder="လက်ခံသူ့ဖုန်းနံပါတ်" value={receiverPhone} onChange={(e) => setReceiverPhone(e.target.value)} className="w-full bg-slate-50 p-4 rounded-2xl font-bold text-xl outline-none" required />
                {foundRecipient ? (
                  <div className="bg-blue-50 p-3 rounded-xl flex items-center gap-2 text-blue-600 font-bold text-xs animate-in slide-in-from-left-4">
                    <User size={16} /> Name: {foundRecipient.name}
                  </div>
                ) : receiverPhone.length >= 7 && (
                  <div className="bg-red-50 p-3 rounded-xl flex items-center gap-2 text-red-400 font-bold text-xs">
                    <X size={16} /> အကောင့်ရှာမတွေ့ပါ
                  </div>
                )}
              </div>
              <input type="number" placeholder="လွှဲမည့်ပမာဏ" value={transferAmount} onChange={(e) => setTransferAmount(e.target.value)} className="w-full bg-slate-50 p-4 rounded-2xl font-bold text-xl outline-none" min="100" required />
              <button disabled={!foundRecipient} className={`w-full font-bold py-4 rounded-2xl shadow-xl flex items-center justify-center gap-2 transition active:scale-95 ${foundRecipient ? 'bg-blue-600 text-white' : 'bg-slate-200 text-slate-400 cursor-not-allowed'}`}>
                <ArrowUpRight size={20} /> ငွေလွှဲမည်
              </button>
            </form>
          </div>
        )}

        <div className="space-y-3">
          <h3 className="text-xs font-black text-slate-400 uppercase tracking-widest px-2 flex items-center gap-2"><History size={14} /> လတ်တလော မှတ်တမ်းများ</h3>
          {currentUser?.transactions.map(tx => (
            <div key={tx.id} onClick={() => { setCurrentReceipt(tx); setShowReceipt(true); }} className="bg-white p-4 rounded-2xl border border-slate-100 flex justify-between items-center shadow-sm hover:border-blue-200 cursor-pointer transition active:scale-[0.98]">
              <div className="flex items-center gap-3">
                <div className={`w-10 h-10 rounded-xl flex items-center justify-center ${tx.isNegative ? 'bg-red-50 text-red-500' : 'bg-green-50 text-green-500'}`}>
                  {tx.isNegative ? <TrendingDown size={20} /> : <ArrowRightLeft size={20} />}
                </div>
                <div>
                  <p className="font-bold text-sm text-slate-800">{tx.label}</p>
                  <p className="text-[9px] text-slate-400 mt-0.5 font-mono">{tx.id} • {tx.timestamp ? tx.timestamp.split(',')[0] : ''}</p>
                </div>
              </div>
              <p className={`font-bold text-sm ${tx.isNegative ? 'text-red-500' : 'text-green-600'}`}>{tx.isNegative ? '-' : '+'}{tx.amount.toLocaleString()}</p>
            </div>
          ))}
          {currentUser?.transactions.length === 0 && <p className="text-center text-slate-300 text-xs py-8 italic font-medium">မှတ်တမ်းမရှိသေးပါ</p>}
        </div>
      </main>

      <footer className="fixed bottom-0 left-0 right-0 bg-white/80 backdrop-blur-md border-t border-slate-100 p-4 flex justify-around max-w-md mx-auto z-[40]">
        <button onClick={() => setActiveTab('topup')} className={`flex flex-col items-center ${activeTab
        import React, { useState, useEffect } from 'react';
import { 
  Smartphone, 
  History, 
  ArrowRightLeft,
  Wallet,
  Zap,
  CheckCircle2,
  TrendingDown,
  User,
  Lock,
  LogIn,
  UserPlus,
  LogOut,
  Send,
  ArrowUpRight,
  X,
  Download,
  Share2
} from 'lucide-react';

const App = () => {
  // App States
  const [isLoggedIn, setIsLoggedIn] = useState(false);
  const [authMode, setAuthMode] = useState('login'); 
  const [activeTab, setActiveTab] = useState('topup'); 
  
  const [currentUser, setCurrentUser] = useState(null);
  const [users, setUsers] = useState([
    { phone: '09123456789', name: 'Demo User', password: '123', balance: 50000, transactions: [] },
    { phone: '09777777777', name: 'Kyaw Kyaw', password: '123', balance: 10000, transactions: [] }
  ]);

  // Auth States
  const [authPhone, setAuthPhone] = useState('');
  const [authName, setAuthName] = useState('');
  const [authPassword, setAuthPassword] = useState('');

  // Top-up/Transfer States
  const [targetPhone, setTargetPhone] = useState('');
  const [selectedOperator, setSelectedOperator] = useState('Atom');
  const [selectedAmount, setSelectedAmount] = useState(1000);
  
  const [receiverPhone, setReceiverPhone] = useState('');
  const [transferAmount, setTransferAmount] = useState('');
  const [foundRecipient, setFoundRecipient] = useState(null);

  // Receipt Modal State
  const [showReceipt, setShowReceipt] = useState(false);
  const [currentReceipt, setCurrentReceipt] = useState(null);

  useEffect(() => {
    if (isLoggedIn && currentUser) {
      setTargetPhone(currentUser.phone);
      detectOperator(currentUser.phone);
    }
  }, [isLoggedIn, currentUser]);

  // လက်ခံမယ့်သူ့ဖုန်းနံပါတ်ရိုက်ရင် နာမည်ရှာပေးခြင်း
  useEffect(() => {
    if (receiverPhone.length >= 7) {
      const recipient = users.find(u => u.phone === receiverPhone);
      setFoundRecipient(recipient || null);
    } else {
      setFoundRecipient(null);
    }
  }, [receiverPhone, users]);

  const operators = [
    { name: 'Atom', color: 'bg-blue-500' },
    { name: 'MPT', color: 'bg-yellow-500' },
    { name: 'Ooredoo', color: 'bg-red-600' },
    { name: 'MyTel', color: 'bg-cyan-500' }
  ];

  const detectOperator = (phone) => {
    if (!phone) return;
    if (/^09(2|4|5|8)/.test(phone)) setSelectedOperator('MPT');
    else if (/^09(9|7)/.test(phone)) setSelectedOperator('Atom');
    else if (/^096/.test(phone)) setSelectedOperator('MyTel');
  };

  const generateTxId = () => "TXN" + Math.floor(100000 + Math.random() * 900000).toString();

  // --- Auth Handlers ---
  const handleSignUp = (e) => {
    e.preventDefault();
    if (users.find(u => u.phone === authPhone)) {
      alert("ဤဖုန်းနံပါတ်ဖြင့် အကောင့်ရှိပြီးသားဖြစ်သည်");
      return;
    }
    const newUser = { phone: authPhone, name: authName, password: authPassword, balance: 50000, transactions: [] };
    setUsers([...users, newUser]);
    alert("အကောင့်ဖွင့်ခြင်းအောင်မြင်ပါသည်");
    setAuthMode('login');
  };

  const handleLogin = (e) => {
    e.preventDefault();
    const user = users.find(u => u.phone === authPhone && u.password === authPassword);
    if (user) {
      setCurrentUser(user);
      setIsLoggedIn(true);
    } else {
      alert("ဖုန်းနံပါတ် သို့မဟုတ် Password မှားယွင်းနေပါသည်");
    }
  };

  const handleLogout = () => {
    setIsLoggedIn(false);
    setCurrentUser(null);
  };

  // --- Transaction Handlers ---
  const handleTransfer = (e) => {
    e.preventDefault();
    const amount = parseInt(transferAmount);
    if (!foundRecipient) return;
    if (currentUser.balance < amount) { alert("လက်ကျန်ငွေ မလုံလောက်ပါ"); return; }
    
    const txId = generateTxId();
    const now = new Date();
    const timestamp = now.toLocaleString();

    const txData = {
      id: txId,
      type: 'transfer',
      amount: amount,
      senderName: currentUser.name,
      senderPhone: currentUser.phone,
      receiverName: foundRecipient.name,
      receiverPhone: foundRecipient.phone,
      timestamp: timestamp,
    };

    // Update Users List (Sender & Receiver)
    setUsers(users.map(u => {
      if (u.phone === currentUser.phone) {
        const updated = { ...u, balance: u.balance - amount, transactions: [{ ...txData, label: `Sent to ${foundRecipient.name}`, isNegative: true }, ...u.transactions] };
        setCurrentUser(updated);
        return updated;
      }
      if (u.phone === foundRecipient.phone) {
        return { ...u, balance: u.balance + amount, transactions: [{ ...txData, label: `Received from ${currentUser.name}`, isNegative: false }, ...u.transactions] };
      }
      return u;
    }));

    setCurrentReceipt(txData);
    setShowReceipt(true);
    setReceiverPhone('');
    setTransferAmount('');
  };

  const handleBuyNow = (e) => {
    e.preventDefault();
    const amount = parseInt(selectedAmount);
    if (currentUser.balance < amount) { alert("လက်ကျန်ငွေ မလုံလောက်ပါ"); return; }
    
    const txId = generateTxId();
    const txData = {
      id: txId,
      type: 'topup',
      label: `${selectedOperator} Top-up (${targetPhone})`,
      amount: amount,
      timestamp: new Date().toLocaleString(),
      isNegative: true
    };

    const updatedUser = {
      ...currentUser,
      balance: currentUser.balance - amount,
      transactions: [txData, ...currentUser.transactions]
    };
    
    setCurrentUser(updatedUser);
    setUsers(users.map(u => u.phone === currentUser.phone ? updatedUser : u));
    setCurrentReceipt(txData);
    setShowReceipt(true);
  };

  // --- Receipt Modal Component ---
  const ReceiptModal = ({ tx, onClose }) => (
    <div className="fixed inset-0 bg-black/60 backdrop-blur-sm z-[100] flex items-center justify-center p-4">
      <div className="bg-white w-full max-w-sm rounded-[2.5rem] overflow-hidden shadow-2xl animate-in zoom-in-95 duration-200">
        <div className="bg-blue-600 p-6 text-white text-center relative">
          <button onClick={onClose} className="absolute right-4 top-4 text-white/50 hover:text-white"><X size={24}/></button>
          <div className="w-16 h-16 bg-white/20 rounded-full flex items-center justify-center mx-auto mb-3">
            <CheckCircle2 size={32} />
          </div>
          <h2 className="text-xl font-black">Transaction Success</h2>
          <p className="text-blue-100 text-xs mt-1">ပြေစာ (Electronic Receipt)</p>
        </div>

        <div className="p-8 space-y-4">
          <div className="flex justify-between items-center border-b border-dashed border-slate-200 pb-4">
            <span className="text-slate-400 text-xs font-bold uppercase">ID နံပါတ်</span>
            <span className="text-slate-800 font-mono font-bold text-sm">{tx.id}</span>
          </div>

          <div className="space-y-3">
            <div className="flex justify-between">
              <span className="text-slate-400 text-xs font-bold uppercase">နေ့စွဲနှင့် အချိန်</span>
              <span className="text-slate-800 font-bold text-xs">{tx.timestamp}</span>
            </div>
            
            <div className="flex justify-between">
              <span className="text-slate-400 text-xs font-bold uppercase">ပို့သူ</span>
              <div className="text-right">
                <p className="text-slate-800 font-bold text-xs">{tx.senderName || currentUser.name}</p>
                <p className="text-slate-400 text-[10px]">{tx.senderPhone || currentUser.phone}</p>
              </div>
            </div>

            <div className="flex justify-between">
              <span className="text-slate-400 text-xs font-bold uppercase">လက်ခံသူ</span>
              <div className="text-right">
                <p className="text-slate-800 font-bold text-xs">{tx.receiverName || tx.label}</p>
                <p className="text-slate-400 text-[10px]">{tx.receiverPhone || ''}</p>
              </div>
            </div>
          </div>

          <div className="bg-slate-50 p-4 rounded-2xl flex justify-between items-center">
            <span className="text-slate-500 font-bold text-xs">စုစုပေါင်း ပမာဏ</span>
            <span className="text-blue-600 font-black text-xl">{tx.amount.toLocaleString()} <small className="text-[10px]">MMK</small></span>
          </div>

          <div className="flex gap-2 pt-2">
            <button onClick={() => alert("Image saved to gallery")} className="flex-1 bg-slate-100 text-slate-700 font-bold py-3 rounded-xl text-xs flex items-center justify-center gap-2"><Download size={16}/> Save</button>
            <button className="flex-1 bg-blue-600 text-white font-bold py-3 rounded-xl text-xs flex items-center justify-center gap-2"><Share2 size={16}/> Share</button>
          </div>
        </div>
        <div className="bg-slate-50 p-4 text-center">
            <p className="text-[9px] text-slate-300 uppercase font-black tracking-widest italic">Thank you for using Quick-Pay</p>
        </div>
      </div>
    </div>
  );

  if (!isLoggedIn) {
    return (
      <div className="min-h-screen bg-slate-50 flex items-center justify-center p-4 font-sans">
        <div className="max-w-md w-full bg-white rounded-[2.5rem] p-8 shadow-xl border border-slate-100">
          <div className="flex flex-col items-center mb-8 text-blue-600">
            <Zap size={40} fill="currentColor" />
            <h1 className="text-2xl font-black mt-2 text-slate-800 tracking-tighter">QUICK-PAY</h1>
          </div>
          <div className="flex bg-slate-100 p-1 rounded-2xl mb-6">
            <button onClick={() => setAuthMode('login')} className={`flex-1 py-2 rounded-xl text-sm font-bold transition ${authMode === 'login' ? 'bg-white text-blue-600 shadow-sm' : 'text-slate-400'}`}>အကောင့်ဝင်</button>
            <button onClick={() => setAuthMode('signup')} className={`flex-1 py-2 rounded-xl text-sm font-bold transition ${authMode === 'signup' ? 'bg-white text-blue-600 shadow-sm' : 'text-slate-400'}`}>အကောင့်ဖွင့်</button>
          </div>
          <form onSubmit={authMode === 'login' ? handleLogin : handleSignUp} className="space-y-4">
            <input type="tel" placeholder="ဖုန်းနံပါတ်" value={authPhone} onChange={(e) => setAuthPhone(e.target.value)} className="w-full px-6 py-4 bg-slate-50 rounded-2xl outline-none focus:ring-2 focus:ring-blue-500 font-bold" required />
            {authMode === 'signup' && <input type="text" placeholder="နာမည်အပြည့်အစုံ" value={authName} onChange={(e) => setAuthName(e.target.value)} className="w-full px-6 py-4 bg-slate-50 rounded-2xl outline-none focus:ring-2 focus:ring-blue-500 font-bold" required />}
            <input type="password" placeholder="Password" value={authPassword} onChange={(e) => setAuthPassword(e.target.value)} className="w-full px-6 py-4 bg-slate-50 rounded-2xl outline-none focus:ring-2 focus:ring-blue-500 font-bold" required />
            <button className="w-full bg-blue-600 text-white font-bold py-4 rounded-2xl shadow-lg transition active:scale-95">{authMode === 'login' ? 'Login ဝင်မည်' : 'အကောင့်ဖွင့်မည်'}</button>
          </form>
        </div>
      </div>
    );
  }

  return (
    <div className="min-h-screen bg-slate-50 pb-24 font-sans">
      {showReceipt && <ReceiptModal tx={currentReceipt} onClose={() => setShowReceipt(false)} />}
      
      <nav className="bg-blue-600 text-white p-4 shadow-lg sticky top-0 z-50">
        <div className="max-w-md mx-auto flex justify-between items-center">
          <div className="flex items-center gap-2 font-black italic text-xl tracking-tighter">
            <Zap fill="currentColor" size={24} /> <span>QUICK-PAY</span>
          </div>
          <button onClick={handleLogout} className="bg-blue-700 p-2 rounded-xl hover:bg-blue-800 transition"><LogOut size={20} /></button>
        </div>
      </nav>

      <main className="max-w-md mx-auto p-4 space-y-6">
        {/* User Stats */}
        <div className="px-2 flex items-center justify-between">
          <div className="flex items-center gap-3">
            <div className="w-12 h-12 bg-blue-600 text-white rounded-2xl flex items-center justify-center font-black text-xl shadow-lg ring-4 ring-white">
              {currentUser?.name ? currentUser.name[0] : 'U'}
            </div>
            <div>
              <p className="text-[10px] font-black text-slate-400 uppercase tracking-widest">Welcome back,</p>
              <p className="text-base font-bold text-slate-700 leading-tight">{currentUser?.name}</p>
              <p className="text-blue-500 font-bold text-[10px]">{currentUser?.phone}</p>
            </div>
          </div>
        </div>

        {/* Card */}
        <div className="bg-gradient-to-br from-slate-900 to-slate-800 rounded-[2.5rem] p-8 text-white shadow-2xl relative overflow-hidden ring-1 ring-white/10">
          <div className="absolute right-[-20px] bottom-[-20px] opacity-10 rotate-12"><Wallet size={160} /></div>
          <p className="text-slate-400 text-xs font-bold mb-1 uppercase tracking-widest">Main Balance</p>
          <h1 className="text-4xl font-bold tracking-tight">
            {currentUser?.balance.toLocaleString()} <span className="text-lg font-normal text-slate-500">MMK</span>
          </h1>
        </div>

        {/* Tabs */}
        <div className="flex bg-white p-1 rounded-2xl border border-slate-200">
          <button onClick={() => setActiveTab('topup')} className={`flex-1 py-3 rounded-xl text-xs font-black uppercase tracking-widest flex items-center justify-center gap-2 transition ${activeTab === 'topup' ? 'bg-blue-600 text-white shadow-lg' : 'text-slate-400'}`}>
            <Smartphone size={16} /> ဘေဖြည့်
          </button>
          <button onClick={() => setActiveTab('transfer')} className={`flex-1 py-3 rounded-xl text-xs font-black uppercase tracking-widest flex items-center justify-center gap-2 transition ${activeTab === 'transfer' ? 'bg-blue-600 text-white shadow-lg' : 'text-slate-400'}`}>
            <Send size={16} /> ငွေလွှဲ
          </button>
        </div>

        {activeTab === 'topup' ? (
          <div className="bg-white rounded-[2rem] p-6 shadow-sm border border-slate-200 animate-in fade-in slide-in-from-bottom-2">
            <div className="grid grid-cols-4 gap-2 mb-8">
              {operators.map((op) => (
                <button key={op.name} onClick={() => setSelectedOperator(op.name)} className={`flex flex-col items-center gap-2 transition-all ${selectedOperator === op.name ? 'scale-110' : 'opacity-30 grayscale'}`}>
                  <div className={`w-14 h-14 rounded-2xl ${op.color} flex items-center justify-center text-white font-black shadow-lg text-lg`}>{op.name[0]}</div>
                  <span className="text-[10px] font-bold uppercase">{op.name}</span>
                </button>
              ))}
            </div>
            <form onSubmit={handleBuyNow} className="space-y-4">
              <input type="tel" value={targetPhone} onChange={(e) => setTargetPhone(e.target.value)} className="w-full bg-slate-50 p-4 rounded-2xl font-bold text-xl outline-none" required />
              <div className="grid grid-cols-3 gap-2">
                {[1000, 3000, 5000, 10000].map(amt => (
                  <button type="button" key={amt} onClick={() => setSelectedAmount(amt)} className={`py-3 rounded-xl text-xs font-bold border-2 transition ${selectedAmount === amt ? 'bg-blue-600 text-white border-blue-600' : 'bg-white text-slate-600 border-slate-100'}`}>{amt.toLocaleString()}</button>
                ))}
              </div>
              <button className="w-full bg-blue-600 text-white font-bold py-4 rounded-2xl shadow-xl flex items-center justify-center gap-2 active:scale-95 transition"><CheckCircle2 size={20} /> ဘေဖြည့်မည်</button>
            </form>
          </div>
        ) : (
          <div className="bg-white rounded-[2rem] p-6 shadow-sm border border-slate-200 animate-in fade-in slide-in-from-bottom-2">
            <form onSubmit={handleTransfer} className="space-y-4">
              <div className="space-y-2">
                <input type="tel" placeholder="လက်ခံသူ့ဖုန်းနံပါတ်" value={receiverPhone} onChange={(e) => setReceiverPhone(e.target.value)} className="w-full bg-slate-50 p-4 rounded-2xl font-bold text-xl outline-none" required />
                {foundRecipient ? (
                  <div className="bg-blue-50 p-3 rounded-xl flex items-center gap-2 text-blue-600 font-bold text-xs animate-in slide-in-from-left-4">
                    <User size={16} /> Name: {foundRecipient.name}
                  </div>
                ) : receiverPhone.length >= 7 && (
                  <div className="bg-red-50 p-3 rounded-xl flex items-center gap-2 text-red-400 font-bold text-xs">
                    <X size={16} /> အကောင့်ရှာမတွေ့ပါ
                  </div>
                )}
              </div>
              <input type="number" placeholder="လွှဲမည့်ပမာဏ" value={transferAmount} onChange={(e) => setTransferAmount(e.target.value)} className="w-full bg-slate-50 p-4 rounded-2xl font-bold text-xl outline-none" min="100" required />
              <button disabled={!foundRecipient} className={`w-full font-bold py-4 rounded-2xl shadow-xl flex items-center justify-center gap-2 transition active:scale-95 ${foundRecipient ? 'bg-blue-600 text-white' : 'bg-slate-200 text-slate-400 cursor-not-allowed'}`}>
                <ArrowUpRight size={20} /> ငွေလွှဲမည်
              </button>
            </form>
          </div>
        )}

        <div className="space-y-3">
          <h3 className="text-xs font-black text-slate-400 uppercase tracking-widest px-2 flex items-center gap-2"><History size={14} /> လတ်တလော မှတ်တမ်းများ</h3>
          {currentUser?.transactions.map(tx => (
            <div key={tx.id} onClick={() => { setCurrentReceipt(tx); setShowReceipt(true); }} className="bg-white p-4 rounded-2xl border border-slate-100 flex justify-between items-center shadow-sm hover:border-blue-200 cursor-pointer transition active:scale-[0.98]">
              <div className="flex items-center gap-3">
                <div className={`w-10 h-10 rounded-xl flex items-center justify-center ${tx.isNegative ? 'bg-red-50 text-red-500' : 'bg-green-50 text-green-500'}`}>
                  {tx.isNegative ? <TrendingDown size={20} /> : <ArrowRightLeft size={20} />}
                </div>
                <div>
                  <p className="font-bold text-sm text-slate-800">{tx.label}</p>
                  <p className="text-[9px] text-slate-400 mt-0.5 font-mono">{tx.id} • {tx.timestamp ? tx.timestamp.split(',')[0] : ''}</p>
                </div>
              </div>
              <p className={`font-bold text-sm ${tx.isNegative ? 'text-red-500' : 'text-green-600'}`}>{tx.isNegative ? '-' : '+'}{tx.amount.toLocaleString()}</p>
            </div>
          ))}
          {currentUser?.transactions.length === 0 && <p className="text-center text-slate-300 text-xs py-8 italic font-medium">မှတ်တမ်းမရှိသေးပါ</p>}
        </div>
      </main>

      <footer className="fixed bottom-0 left-0 right-0 bg-white/80 backdrop-blur-md border-t border-slate-100 p-4 flex justify-around max-w-md mx-auto z-[40]">
        <button onClick={() => setActiveTab('topup')} className={`flex flex-col items-center ${activeTab
        
