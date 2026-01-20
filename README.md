import React, { useState, useEffect } from 'react';
import { initializeApp } from 'firebase/app';
import { 
  getAuth, 
  signInAnonymously, 
  onAuthStateChanged,
  signInWithCustomToken 
} from 'firebase/auth';
import { 
  getFirestore, 
  collection, 
  addDoc, 
  deleteDoc, 
  updateDoc,
  setDoc, // Ditambahkan
  doc, 
  onSnapshot,
  serverTimestamp 
} from 'firebase/firestore';
import { 
  School, 
  Users, 
  GraduationCap, 
  Bell, 
  Plus, 
  Trash2, 
  Edit2, 
  Save, 
  ArrowLeft,
  Phone,
  MapPin,
  Mail,
  Loader2
} from 'lucide-react';

// --- Konfigurasi Firebase ---
const firebaseConfig = JSON.parse(__firebase_config);
const app = initializeApp(firebaseConfig);
const auth = getAuth(app);
const db = getFirestore(app);
const appId = typeof __app_id !== 'undefined' ? __app_id : 'default-app-id';

// --- Komponen Utama ---
export default function App() {
  const [user, setUser] = useState(null);
  const [currentView, setCurrentView] = useState('home');
  const [schoolProfile, setSchoolProfile] = useState({
    name: 'SMA Harapan Bangsa',
    address: 'Jl. Pendidikan No. 1, Jakarta',
    contact: '(021) 123-4567',
    email: 'info@smaharapan.sch.id',
    vision: 'Mewujudkan generasi cerdas, berkarakter, dan berdaya saing global.',
    mission: '1. Menyelenggarakan pendidikan berkualitas.\n2. Membentuk karakter siswa yang religius.\n3. Mengembangkan potensi siswa secara maksimal.'
  });

  // Autentikasi Firebase
  useEffect(() => {
    const initAuth = async () => {
      try {
        if (typeof __initial_auth_token !== 'undefined' && __initial_auth_token) {
          await signInWithCustomToken(auth, __initial_auth_token);
        } else {
          await signInAnonymously(auth);
        }
      } catch (error) {
        console.error("Auth error:", error);
      }
    };
    initAuth();
    const unsubscribe = onAuthStateChanged(auth, setUser);
    return () => unsubscribe();
  }, []);

  // Fetch Profil Sekolah (Simulasi Database Tunggal untuk Profil)
  useEffect(() => {
    if (!user) return;
    
    // Perbaikan Path: Menggunakan 6 segmen (settings/profile)
    const profileRef = doc(db, 'artifacts', appId, 'public', 'data', 'settings', 'profile');

    const unsub = onSnapshot(
      profileRef,
      (docSnap) => {
        if (docSnap.exists()) {
          setSchoolProfile(docSnap.data());
        }
      },
      (error) => console.error("Error fetching profile:", error)
    );
    return () => unsub();
  }, [user]);

  // Fungsi navigasi
  const renderView = () => {
    switch (currentView) {
      case 'profile':
        return <ProfileView user={user} data={schoolProfile} />;
      case 'teachers':
        // Perbaikan: Mengirim komponen Icon (Users) bukan elemen (<Users/>)
        return <DataListView user={user} type="guru" title="Data Guru & Staf" Icon={Users} />;
      case 'students':
        return <DataListView user={user} type="siswa" title="Data Siswa" Icon={GraduationCap} />;
      case 'announcements':
        return <AnnouncementsView user={user} />;
      default:
        return <HomeView setView={setCurrentView} profile={schoolProfile} />;
    }
  };

  if (!user) {
    return (
      <div className="flex items-center justify-center min-h-screen bg-slate-50">
        <div className="flex flex-col items-center gap-4 text-slate-600">
          <Loader2 className="w-10 h-10 animate-spin text-blue-600" />
          <p>Menghubungkan ke sistem sekolah...</p>
        </div>
      </div>
    );
  }

  return (
    <div className="min-h-screen bg-slate-50 text-slate-900 font-sans">
      {/* Header Utama */}
      <header className="bg-blue-700 text-white shadow-lg sticky top-0 z-50">
        <div className="max-w-4xl mx-auto px-4 py-4 flex justify-between items-center">
          <div className="flex items-center gap-3 cursor-pointer" onClick={() => setCurrentView('home')}>
            <div className="bg-white p-2 rounded-full text-blue-700">
              <School size={24} />
            </div>
            <div>
              <h1 className="font-bold text-lg leading-tight">{schoolProfile.name}</h1>
              <p className="text-blue-200 text-xs">Portal Informasi Digital</p>
            </div>
          </div>
          
          {currentView !== 'home' && (
            <button 
              onClick={() => setCurrentView('home')}
              className="bg-blue-600 hover:bg-blue-500 px-3 py-1.5 rounded-md text-sm flex items-center gap-2 transition-colors"
            >
              <ArrowLeft size={16} /> Kembali
            </button>
          )}
        </div>
      </header>

      {/* Konten Dinamis */}
      <main className="max-w-4xl mx-auto px-4 py-6 mb-20">
        {renderView()}
      </main>

      {/* Footer */}
      <footer className="bg-slate-800 text-slate-400 py-6 text-center text-sm">
        <p>&copy; {new Date().getFullYear()} {schoolProfile.name}. All rights reserved.</p>
        <p className="mt-1 text-xs">Sistem Informasi Sekolah Terintegrasi</p>
      </footer>
    </div>
  );
}

// --- Tampilan Beranda ---
function HomeView({ setView, profile }) {
  // Perbaikan: Simpan komponen ikon (School, Users, dll) langsung, bukan JSX
  const menuItems = [
    { id: 'profile', label: 'Profil Sekolah', Icon: School, color: 'bg-emerald-500', desc: 'Visi, Misi & Identitas' },
    { id: 'teachers', label: 'Guru & Staf', Icon: Users, color: 'bg-blue-500', desc: 'Direktori Pengajar' },
    { id: 'students', label: 'Data Siswa', Icon: GraduationCap, color: 'bg-purple-500', desc: 'Informasi Kesiswaan' },
    { id: 'announcements', label: 'Pengumuman', Icon: Bell, color: 'bg-orange-500', desc: 'Berita Terbaru' },
  ];

  return (
    <div className="space-y-8 animate-in fade-in duration-500">
      {/* Hero Section */}
      <div className="bg-white rounded-2xl p-6 shadow-sm border border-slate-200 text-center space-y-4">
        <h2 className="text-2xl md:text-3xl font-bold text-slate-800">Selamat Datang Warga Sekolah!</h2>
        <p className="text-slate-600 max-w-2xl mx-auto">
          Akses seluruh informasi akademik dan non-akademik melalui portal ini. 
          Website ini diperbarui secara berkala untuk menunjang kegiatan belajar mengajar.
        </p>
      </div>

      {/* Menu Grid */}
      <div className="grid grid-cols-1 sm:grid-cols-2 gap-4">
        {menuItems.map((item) => (
          <button
            key={item.id}
            onClick={() => setView(item.id)}
            className="group relative overflow-hidden bg-white p-6 rounded-xl shadow-sm border border-slate-200 hover:shadow-md hover:border-blue-300 transition-all text-left flex items-start gap-4"
          >
            <div className={`${item.color} text-white p-3 rounded-lg shadow-sm group-hover:scale-110 transition-transform`}>
              {/* Render komponen ikon di sini */}
              <item.Icon size={32} />
            </div>
            <div>
              <h3 className="font-bold text-lg text-slate-800 group-hover:text-blue-600">{item.label}</h3>
              <p className="text-sm text-slate-500">{item.desc}</p>
            </div>
          </button>
        ))}
      </div>

      {/* Quick Contact Info */}
      <div className="bg-blue-50 rounded-xl p-6 border border-blue-100 grid md:grid-cols-3 gap-6">
        <div className="flex items-center gap-3">
          <MapPin className="text-blue-600" />
          <span className="text-sm text-slate-700">{profile.address}</span>
        </div>
        <div className="flex items-center gap-3">
          <Phone className="text-blue-600" />
          <span className="text-sm text-slate-700">{profile.contact}</span>
        </div>
        <div className="flex items-center gap-3">
          <Mail className="text-blue-600" />
          <span className="text-sm text-slate-700">{profile.email}</span>
        </div>
      </div>
    </div>
  );
}

// --- Tampilan Profil Sekolah (Editable) ---
function ProfileView({ user, data }) {
  const [isEditing, setIsEditing] = useState(false);
  const [formData, setFormData] = useState(data);
  const [loading, setLoading] = useState(false);

  // Update state lokal jika data props berubah (saat loading awal)
  useEffect(() => {
    setFormData(data);
  }, [data]);

  const handleSave = async () => {
    if (!user) return;
    setLoading(true);
    try {
      // Perbaikan Path: Gunakan path 6 segmen
      const docRef = doc(db, 'artifacts', appId, 'public', 'data', 'settings', 'profile');
      
      // Gunakan setDoc dengan merge agar aman jika dokumen belum ada
      await setDoc(docRef, formData, { merge: true });
      
      setIsEditing(false);
    } catch (e) {
      console.error(e);
      alert("Gagal menyimpan profil.");
    }
    setLoading(false);
  };

  return (
    <div className="bg-white rounded-xl shadow-sm border border-slate-200 overflow-hidden">
      <div className="p-6 border-b border-slate-100 flex justify-between items-center bg-emerald-50">
        <div className="flex items-center gap-3">
          <School className="text-emerald-600" size={28} />
          <h2 className="text-xl font-bold text-emerald-900">Profil Sekolah</h2>
        </div>
        <button 
          onClick={() => isEditing ? handleSave() : setIsEditing(true)}
          disabled={loading}
          className={`flex items-center gap-2 px-4 py-2 rounded-lg text-sm font-medium transition-colors ${
            isEditing 
              ? 'bg-emerald-600 text-white hover:bg-emerald-700' 
              : 'bg-white text-emerald-700 border border-emerald-200 hover:bg-emerald-50'
          }`}
        >
          {loading ? <Loader2 className="animate-spin" size={16} /> : isEditing ? <><Save size={16} /> Simpan</> : <><Edit2 size={16} /> Edit Profil</>}
        </button>
      </div>

      <div className="p-6 space-y-6">
        {/* Identitas */}
        <div className="grid md:grid-cols-2 gap-6">
          <div className="space-y-4">
            <div>
              <label className="block text-sm font-semibold text-slate-700 mb-1">Nama Sekolah</label>
              {isEditing ? (
                <input 
                  type="text" 
                  value={formData.name} 
                  onChange={e => setFormData({...formData, name: e.target.value})}
                  className="w-full p-2 border rounded-md focus:ring-2 focus:ring-emerald-500 outline-none"
                />
              ) : (
                <p className="text-lg text-slate-900">{formData.name}</p>
              )}
            </div>
            <div>
              <label className="block text-sm font-semibold text-slate-700 mb-1">Alamat</label>
              {isEditing ? (
                <input 
                  type="text" 
                  value={formData.address} 
                  onChange={e => setFormData({...formData, address: e.target.value})}
                  className="w-full p-2 border rounded-md focus:ring-2 focus:ring-emerald-500 outline-none"
                />
              ) : (
                <p className="text-slate-600 flex items-center gap-2"><MapPin size={16}/> {formData.address}</p>
              )}
            </div>
          </div>
          <div className="space-y-4">
            <div>
              <label className="block text-sm font-semibold text-slate-700 mb-1">Kontak Telepon</label>
              {isEditing ? (
                <input 
                  type="text" 
                  value={formData.contact} 
                  onChange={e => setFormData({...formData, contact: e.target.value})}
                  className="w-full p-2 border rounded-md focus:ring-2 focus:ring-emerald-500 outline-none"
                />
              ) : (
                <p className="text-slate-600 flex items-center gap-2"><Phone size={16}/> {formData.contact}</p>
              )}
            </div>
            <div>
              <label className="block text-sm font-semibold text-slate-700 mb-1">Email</label>
              {isEditing ? (
                <input 
                  type="text" 
                  value={formData.email} 
                  onChange={e => setFormData({...formData, email: e.target.value})}
                  className="w-full p-2 border rounded-md focus:ring-2 focus:ring-emerald-500 outline-none"
                />
              ) : (
                <p className="text-slate-600 flex items-center gap-2"><Mail size={16}/> {formData.email}</p>
              )}
            </div>
          </div>
        </div>

        <hr />

        {/* Visi Misi */}
        <div className="grid md:grid-cols-2 gap-8">
          <div>
            <h3 className="text-lg font-bold text-emerald-800 mb-3">Visi</h3>
            {isEditing ? (
              <textarea 
                rows={4}
                value={formData.vision}
                onChange={e => setFormData({...formData, vision: e.target.value})}
                className="w-full p-2 border rounded-md focus:ring-2 focus:ring-emerald-500 outline-none"
              />
            ) : (
              <p className="italic text-slate-700 bg-emerald-50 p-4 rounded-lg border border-emerald-100">"{formData.vision}"</p>
            )}
          </div>
          <div>
            <h3 className="text-lg font-bold text-emerald-800 mb-3">Misi</h3>
            {isEditing ? (
              <textarea 
                rows={6}
                value={formData.mission}
                onChange={e => setFormData({...formData, mission: e.target.value})}
                className="w-full p-2 border rounded-md focus:ring-2 focus:ring-emerald-500 outline-none"
              />
            ) : (
              <div className="text-slate-700 whitespace-pre-line bg-slate-50 p-4 rounded-lg border border-slate-100">{formData.mission}</div>
            )}
          </div>
        </div>
      </div>
    </div>
  );
}

// --- Komponen List Data Umum (Guru & Siswa) ---
function DataListView({ user, type, title, Icon }) {
  const [items, setItems] = useState([]);
  const [newItem, setNewItem] = useState({ name: '', role: '', nip: '' });
  const collectionName = type === 'guru' ? 'teachers' : 'students';
  
  useEffect(() => {
    if (!user) return;
    const q = collection(db, 'artifacts', appId, 'public', 'data', collectionName);
    const unsubscribe = onSnapshot(q, (snapshot) => {
      const data = snapshot.docs.map(doc => ({ id: doc.id, ...doc.data() }));
      data.sort((a, b) => a.name.localeCompare(b.name));
      setItems(data);
    });
    return () => unsubscribe();
  }, [user, collectionName]);

  const handleAdd = async (e) => {
    e.preventDefault();
    if (!newItem.name) return;
    try {
      await addDoc(collection(db, 'artifacts', appId, 'public', 'data', collectionName), {
        ...newItem,
        createdAt: serverTimestamp()
      });
      setNewItem({ name: '', role: '', nip: '' });
    } catch (error) {
      console.error("Error adding doc: ", error);
    }
  };

  const handleDelete = async (id) => {
    if(window.confirm("Yakin ingin menghapus data ini?")) {
      await deleteDoc(doc(db, 'artifacts', appId, 'public', 'data', collectionName, id));
    }
  };

  return (
    <div className="space-y-6">
      <div className="flex items-center gap-3 mb-6">
        <div className="p-3 bg-blue-100 text-blue-700 rounded-lg">
           {/* Render Icon Komponen secara langsung */}
           <Icon size={24} />
        </div>
        <h2 className="text-2xl font-bold text-slate-800">{title}</h2>
      </div>

      {/* Form Tambah */}
      <form onSubmit={handleAdd} className="bg-white p-4 rounded-xl shadow-sm border border-slate-200 flex flex-col md:flex-row gap-3 items-end">
        <div className="flex-1 w-full">
          <label className="text-xs font-semibold text-slate-500 mb-1 block">Nama Lengkap</label>
          <input 
            type="text" 
            value={newItem.name}
            onChange={e => setNewItem({...newItem, name: e.target.value})}
            placeholder="Contoh: Budi Santoso"
            className="w-full p-2 border rounded-md text-sm outline-none focus:border-blue-500"
          />
        </div>
        <div className="flex-1 w-full">
          <label className="text-xs font-semibold text-slate-500 mb-1 block">
            {type === 'guru' ? 'Mata Pelajaran / Jabatan' : 'Kelas'}
          </label>
          <input 
            type="text" 
            value={newItem.role}
            onChange={e => setNewItem({...newItem, role: e.target.value})}
            placeholder={type === 'guru' ? "Contoh: Matematika" : "Contoh: X IPA 1"}
            className="w-full p-2 border rounded-md text-sm outline-none focus:border-blue-500"
          />
        </div>
        <div className="w-full md:w-auto">
           <button type="submit" className="w-full bg-blue-600 hover:bg-blue-700 text-white px-4 py-2 rounded-md text-sm font-medium flex items-center justify-center gap-2 transition-colors">
             <Plus size={16} /> Tambah
           </button>
        </div>
      </form>

      {/* List Data */}
      <div className="grid grid-cols-1 md:grid-cols-2 gap-4">
        {items.length === 0 ? (
          <p className="col-span-full text-center py-10 text-slate-400 italic">Belum ada data tersedia.</p>
        ) : (
          items.map((item) => (
            <div key={item.id} className="bg-white p-4 rounded-xl border border-slate-200 shadow-sm flex justify-between items-center group hover:border-blue-300 transition-all">
              <div className="flex items-center gap-4">
                <div className="w-10 h-10 rounded-full bg-slate-100 flex items-center justify-center text-slate-500 font-bold">
                  {item.name.charAt(0)}
                </div>
                <div>
                  <h4 className="font-bold text-slate-800">{item.name}</h4>
                  <p className="text-sm text-slate-500">{item.role}</p>
                </div>
              </div>
              <button 
                onClick={() => handleDelete(item.id)}
                className="text-slate-300 hover:text-red-500 p-2 transition-colors"
                title="Hapus"
              >
                <Trash2 size={18} />
              </button>
            </div>
          ))
        )}
      </div>
    </div>
  );
}

// --- Tampilan Pengumuman ---
function AnnouncementsView({ user }) {
  const [announcements, setAnnouncements] = useState([]);
  const [newTitle, setNewTitle] = useState('');
  const [newContent, setNewContent] = useState('');

  // Sort manual by createdAt desc (newest first)
  useEffect(() => {
    if (!user) return;
    const q = collection(db, 'artifacts', appId, 'public', 'data', 'announcements');
    const unsubscribe = onSnapshot(q, (snapshot) => {
      const data = snapshot.docs.map(doc => ({ 
        id: doc.id, 
        ...doc.data(),
        // Handle timestamp conversion safely
        createdAt: doc.data().createdAt ? doc.data().createdAt.toDate() : new Date() 
      }));
      data.sort((a, b) => b.createdAt - a.createdAt);
      setAnnouncements(data);
    });
    return () => unsubscribe();
  }, [user]);

  const handlePost = async (e) => {
    e.preventDefault();
    if (!newTitle || !newContent) return;

    try {
      await addDoc(collection(db, 'artifacts', appId, 'public', 'data', 'announcements'), {
        title: newTitle,
        content: newContent,
        createdAt: serverTimestamp(),
        author: 'Admin Sekolah'
      });
      setNewTitle('');
      setNewContent('');
    } catch (error) {
      console.error("Error posting announcement:", error);
    }
  };

  const handleDelete = async (id) => {
    if(window.confirm("Hapus pengumuman ini?")) {
      await deleteDoc(doc(db, 'artifacts', appId, 'public', 'data', 'announcements', id));
    }
  };

  return (
    <div className="space-y-6">
      <div className="flex items-center gap-3 mb-6">
        <div className="p-3 bg-orange-100 text-orange-600 rounded-lg"><Bell /></div>
        <h2 className="text-2xl font-bold text-slate-800">Papan Pengumuman</h2>
      </div>

      {/* Form Buat Pengumuman */}
      <div className="bg-white p-5 rounded-xl shadow-sm border border-slate-200">
        <h3 className="font-semibold text-slate-700 mb-3">Buat Pengumuman Baru</h3>
        <form onSubmit={handlePost} className="space-y-3">
          <input 
            className="w-full p-2 border rounded-md text-sm outline-none focus:ring-2 focus:ring-orange-200"
            placeholder="Judul Pengumuman"
            value={newTitle}
            onChange={e => setNewTitle(e.target.value)}
          />
          <textarea 
            className="w-full p-2 border rounded-md text-sm outline-none focus:ring-2 focus:ring-orange-200"
            rows={3}
            placeholder="Isi informasi..."
            value={newContent}
            onChange={e => setNewContent(e.target.value)}
          />
          <div className="flex justify-end">
            <button type="submit" className="bg-orange-500 hover:bg-orange-600 text-white px-4 py-2 rounded-md text-sm font-medium flex items-center gap-2">
              <Plus size={16} /> Terbitkan
            </button>
          </div>
        </form>
      </div>

      {/* List Pengumuman */}
      <div className="space-y-4">
        {announcements.length === 0 ? (
           <p className="text-center py-10 text-slate-400 italic">Belum ada pengumuman terbaru.</p>
        ) : (
          announcements.map((item) => (
            <div key={item.id} className="bg-white p-6 rounded-xl border-l-4 border-orange-400 shadow-sm relative group">
               <button 
                  onClick={() => handleDelete(item.id)}
                  className="absolute top-4 right-4 text-slate-300 hover:text-red-500 opacity-0 group-hover:opacity-100 transition-opacity"
                  title="Hapus"
                >
                  <Trash2 size={16} />
                </button>
              <span className="text-xs text-orange-600 font-bold bg-orange-50 px-2 py-1 rounded-full mb-2 inline-block">
                {item.createdAt.toLocaleDateString('id-ID', { day: 'numeric', month: 'long', year: 'numeric' })}
              </span>
              <h3 className="text-xl font-bold text-slate-800 mb-2">{item.title}</h3>
              <p className="text-slate-600 whitespace-pre-line leading-relaxed">{item.content}</p>
            </div>
          ))
        )}
      </div>
    </div>
  );
}
