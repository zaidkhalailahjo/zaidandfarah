<!DOCTYPE html>
<html lang="ar" dir="rtl">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>فرح سناكس</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <script src="https://unpkg.com/lucide@latest"></script>
    <link href="https://fonts.googleapis.com/css2?family=Inter:wght@400;600;800&display=swap" rel="stylesheet">
    <style>
        body { font-family: 'Inter', sans-serif; }
        .modal { transition: opacity 0.3s ease, visibility 0.3s ease; opacity: 0; visibility: hidden; }
        .modal.active { opacity: 1; visibility: visible; }
        .no-scrollbar::-webkit-scrollbar { display: none; }
        .no-scrollbar { -ms-overflow-style: none;  scrollbar-width: none; }
        .loader {
            border: 4px solid #f3f3f3;
            border-top: 4px solid #fe6a00;
            border-radius: 50%;
            width: 20px;
            height: 20px;
            animation: spin 1s linear infinite;
        }
        @keyframes spin {
            0% { transform: rotate(0deg); }
            100% { transform: rotate(360deg); }
        }
    </style>
</head>
<body class="bg-gray-100 flex flex-col min-h-screen text-gray-800">

    <div id="toast-container" class="fixed top-5 right-5 z-[100] flex flex-col gap-2"></div>

    <header class="bg-orange-600 text-white shadow-lg sticky top-0 z-40">
        <div class="container mx-auto px-4 py-4 flex justify-between items-center">
            <div class="flex items-center gap-2">
                <div class="bg-white p-2 rounded-full text-orange-600 font-bold text-xl">S</div>
                <h1 class="text-2xl font-bold">فرح سناكس</h1>
            </div>
            
            <div class="relative hidden md:block w-1/3">
                <input type="text" id="desktop-search" placeholder="ابحث عن منتج..." class="w-full py-2 px-4 pr-10 rounded-full text-gray-800 focus:outline-none focus:ring-2 focus:ring-orange-300">
                <i data-lucide="search" class="absolute left-3 top-2.5 text-gray-400 w-5 h-5"></i>
            </div>

            <div class="flex items-center gap-4">
                <div class="relative cursor-pointer hover:scale-105 transition-transform" id="cart-btn">
                    <i data-lucide="shopping-cart" class="w-8 h-8"></i>
                    <span id="cart-count" class="absolute -top-2 -right-2 bg-red-500 text-white text-xs font-bold w-5 h-5 flex items-center justify-center rounded-full border-2 border-orange-600 hidden">0</span>
                </div>
            </div>
        </div>
        <div class="md:hidden px-4 pb-4">
           <input type="text" id="mobile-search" placeholder="ابحث عن منتج..." class="w-full py-2 px-4 rounded-full text-gray-800 focus:outline-none">
        </div>
    </header>

    <div class="bg-orange-100 py-8 px-4 text-center">
        <h2 class="text-3xl md:text-5xl font-extrabold text-gray-800 mb-2">ألذ وأشهر أصناف السناكات!</h2>
        <p class="text-gray-600 text-lg">كل ما تشتهيه من أصناف الشيبس والشوكولاتة والقهوة تجده هنا.</p>
        <div id="offline-mode-alert" class="mt-4 p-2 bg-yellow-200 text-yellow-800 rounded-lg font-bold hidden"></div>
    </div>

    <div class="container mx-auto px-4 py-6">
        <div id="categories-container" class="flex overflow-x-auto pb-4 gap-3 no-scrollbar scroll-smooth"></div>
    </div>

    <main class="container mx-auto px-4 pb-20 flex-grow">
        <div id="loading-indicator" class="text-center p-10 text-xl font-medium text-orange-500 hidden">جاري تحميل المنتجات...</div>
        <div id="products-grid" class="grid grid-cols-2 md:grid-cols-3 lg:grid-cols-4 gap-4 md:gap-6"></div>
        <div id="empty-state" class="text-center py-20 hidden">
            <p class="text-gray-500 text-xl">عذراً، لا توجد منتجات مطابقة لبحثك.</p>
        </div>
    </main>

    <footer class="bg-gray-800 text-white py-6 text-center text-sm mt-auto">
        <p>© 2024 سناكس ماركت - جميع الحقوق محفوظة</p>
    </footer>

    <div class="fixed bottom-4 left-4 z-50">
        <button id="admin-trigger-btn" class="w-12 h-12 bg-transparent hover:bg-green-500/20 rounded-full flex items-center justify-center transition-colors cursor-default" title="منطقة الإدارة">
            <i data-lucide="lock" class="w-4 h-4 text-transparent hover:text-green-500/50"></i>
        </button>
    </div>

    <div id="product-modal" class="modal fixed inset-0 z-50 flex items-center justify-center p-4 bg-black/60 backdrop-blur-sm">
        <div class="bg-white rounded-2xl w-full max-w-lg overflow-hidden shadow-2xl relative max-h-[90vh] overflow-y-auto transform scale-95 transition-transform duration-300" id="product-modal-content">
            <button class="close-modal absolute top-4 left-4 bg-white/80 p-2 rounded-full hover:bg-gray-200 z-10">
                <i data-lucide="x" class="w-6 h-6 text-gray-700"></i>
            </button>
            <div id="product-modal-body"></div>
        </div>
    </div>

    <div id="cart-modal" class="modal fixed inset-0 z-50 flex items-center justify-center p-4 bg-black/60 backdrop-blur-sm">
        <div class="bg-white rounded-2xl w-full max-w-xl overflow-hidden shadow-2xl relative max-h-[90vh] flex flex-col transform scale-95 transition-transform duration-300">
            <div class="p-5 border-b flex justify-between items-center bg-white z-10">
                <h2 class="text-2xl font-bold text-gray-800 flex items-center gap-2">
                    <i data-lucide="shopping-cart" class="w-6 h-6 text-orange-600"></i> سلة التسوق
                </h2>
                <button class="close-modal p-2 rounded-full hover:bg-gray-100">
                    <i data-lucide="x" class="w-6 h-6 text-gray-700"></i>
                </button>
            </div>
            <div id="cart-items-container" class="p-5 space-y-4 overflow-y-auto flex-grow"></div>
            <div id="cart-footer" class="p-5 bg-gray-50 border-t space-y-4">
                <div class="space-y-2">
                    <h3 class="font-bold text-gray-700 flex items-center gap-1">
                        <i data-lucide="tag" class="w-4 h-4 text-orange-600"></i> كود الخصم
                    </h3>
                    <div class="flex gap-2">
                        <input type="text" id="coupon-input" placeholder="أدخل الرمز" class="flex-grow p-2 border rounded-lg">
                        <button id="apply-coupon-btn" class="bg-gray-900 text-white px-4 py-2 rounded-lg font-semibold hover:bg-orange-600">تطبيق</button>
                    </div>
                    <div id="coupon-message" class="hidden text-sm font-medium p-2 rounded-lg"></div>
                </div>
                <div class="space-y-2 text-sm font-medium">
                    <div class="flex justify-between"><span class="text-gray-600">المجموع الفرعي:</span><span id="cart-subtotal" class="font-bold">0.00</span></div>
                    <div class="flex justify-between text-green-600"><span class="font-semibold">الخصم:</span><span id="cart-discount" class="font-bold">0.00</span></div>
                    <div class="flex justify-between pt-2 border-t text-lg"><span class="font-extrabold text-gray-800">الكلي:</span><span id="cart-total" class="font-extrabold text-orange-600">0.00</span></div>
                </div>
                <button id="checkout-btn" class="w-full mt-4 py-3 rounded-xl font-bold text-lg shadow-lg bg-orange-600 text-white hover:bg-orange-700 flex items-center justify-center gap-2">
                    إتمام عملية الشراء
                </button>
            </div>
        </div>
    </div>

    <div id="contact-modal" class="modal fixed inset-0 z-50 flex items-center justify-center p-4 bg-black/70 backdrop-blur-sm">
        <div class="bg-white rounded-2xl w-full max-w-sm overflow-hidden shadow-2xl relative">
            <div class="p-5 border-b bg-orange-600 text-white flex justify-between items-center">
                <h2 class="text-xl font-bold flex items-center gap-2"><i data-lucide="phone" class="w-5 h-5"></i> إتمام الشراء</h2>
                <button class="close-modal p-1 rounded-full hover:bg-white/20"><i data-lucide="x" class="w-5 h-5"></i></button>
            </div>
            <div class="p-6 space-y-5">
                <div class="space-y-2">
                    <label class="block text-lg font-bold text-gray-800">الاسم (مطلوب - 3 أحرف على الأقل)</label>
                    <input type="text" id="customer-name" placeholder="أدخل اسمك الكريم" class="w-full p-3 border rounded-lg focus:ring-2 focus:ring-orange-500">
                    <p id="name-error" class="text-red-500 text-sm hidden flex items-center gap-1"><i data-lucide="alert-triangle" class="w-3 h-3"></i> الاسم قصير جداً</p>
                </div>
                <div class="p-4 bg-gray-50 rounded-lg border border-gray-200 text-center">
                    <p class="text-3xl font-extrabold text-orange-600" id="contact-total-display">0.00</p>
                    <p id="contact-coupon-note" class="text-sm font-semibold text-green-600 hidden"></p>
                </div>
                <div class="space-y-3">
                    <button id="wa-btn" class="w-full flex items-center justify-center gap-3 py-3 rounded-xl font-bold text-lg shadow-lg bg-green-500 text-white hover:bg-green-600 disabled:bg-gray-400 transition-transform active:scale-95">
                        <i data-lucide="message-square" class="w-5 h-5"></i> إرسال عبر واتساب
                    </button>
                    <button id="messenger-btn" class="w-full flex items-center justify-center gap-3 py-3 rounded-xl font-bold text-lg shadow-lg bg-blue-600 text-white hover:bg-blue-700 disabled:bg-gray-400 transition-transform active:scale-95">
                        <i data-lucide="copy" class="w-5 h-5"></i> نسخ الطلب وفتح ماسنجر
                    </button>
                </div>
            </div>
        </div>
    </div>

    <div id="admin-auth-modal" class="modal fixed inset-0 z-50 flex items-center justify-center p-4 bg-black/70 backdrop-blur-sm">
        <div class="bg-white p-6 rounded-xl shadow-2xl w-full max-w-xs">
            <h3 class="text-lg font-bold mb-4">كلمة المرور للمدير</h3>
            <input type="password" id="admin-password-input" class="w-full p-2 border rounded mb-4" placeholder="*****">
            <div class="flex justify-end gap-2">
                <button class="close-modal px-4 py-2 text-gray-600">إلغاء</button>
                <button id="admin-login-btn" class="bg-green-600 text-white px-4 py-2 rounded">دخول</button>
            </div>
        </div>
    </div>

    <div id="admin-panel-modal" class="modal fixed inset-0 z-50 flex items-center justify-center p-4 bg-black/70 backdrop-blur-sm">
        <div class="bg-white rounded-2xl w-full max-w-4xl h-[90vh] flex flex-col shadow-2xl">
            <div class="p-5 border-b bg-gray-900 text-white flex justify-between items-center">
                <h2 class="text-xl font-bold flex items-center gap-2"><i data-lucide="lock" class="w-5 h-5 text-green-400"></i> لوحة التحكم</h2>
                <button class="close-modal p-1 rounded-full hover:bg-white/20"><i data-lucide="x" class="w-5 h-5"></i></button>
            </div>
            <div class="flex-grow overflow-y-auto p-6 space-y-4" id="admin-products-list"></div>
            <div class="p-5 border-t bg-gray-50 flex justify-end">
                <button id="admin-save-btn" class="flex items-center gap-2 py-3 px-8 rounded-xl font-bold text-lg shadow-lg bg-green-600 text-white hover:bg-green-700">
                    <i data-lucide="save" class="w-5 h-5"></i> حفظ التعديلات
                </button>
            </div>
        </div>
    </div>

    <script type="module">
        import { initializeApp } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-app.js";
        import { getAuth, signInAnonymously, signInWithCustomToken, onAuthStateChanged } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-auth.js";
        import { getFirestore, doc, setDoc, deleteDoc, onSnapshot, collection, query, writeBatch, runTransaction, getDocs, updateDoc, serverTimestamp } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-firestore.js";

        const initialProductsData = [
            { id: '20', name: 'علكة نكهة النعناع برودواي', price: 0.10, currency: 'د.أ', category: 'علكة', image: 'https://uploads.onecompiler.io/43n8uttmw/445tfdw7r/WhatsApp_Image_2025-11-27_at_15.23.18_ed77c273-removebg-preview.png', productionDate: '6/6/2025', expiryDate: '6/6/2028', ingredients: 'قاعدة صمغ، مواد تحلية، نكهات نعناع.', stock: 2, calories: 5 }, 
            { id: '19', name: 'بسكوت ستارز', price: 0.10, currency: 'د.أ', category: 'بسكويت', image: 'https://uploads.onecompiler.io/43n8uttmw/445tfdw7r/WhatsApp_Image_2025-11-27_at_15.04.11_259af214-removebg-preview.png', productionDate: '10/10/2025', expiryDate: '9/10/2026', ingredients: 'دقيق القمح، سكر، زيت نباتي.', stock: 1, calories: 170 },
            { id: '18', name: 'بسكوت مانيكس', price: 0.05, currency: 'د.أ', category: 'بسكويت', image: 'https://uploads.onecompiler.io/43n8uttmw/445tfdw7r/images__18_-removebg-preview.png', productionDate: '1/11/2025', expiryDate: '1/12/2026', ingredients: 'دقيق القمح، زيت نباتي، كاكاو.', stock: 1, calories: 94 },
            { id: '17', name: 'قهوة العميد 2/1', price: 0.25, currency: 'د.أ', category: 'قهوة', image: 'https://uploads.onecompiler.io/43n8uttmw/445tfdw7r/Products_Images_online_store_1-removebg-preview.png', productionDate: '1/10/2025', expiryDate: '1/3/2027', ingredients: 'قهوة سريعة الذوبان، كريمة.', stock: 3, calories: 0 },
            { id: '16', name: 'شوكلاتة بريك تايم', price: 0.10, currency: 'د.أ', category: 'شوكولاتة', image: 'https://uploads.onecompiler.io/43n8uttmw/445tfdw7r/images__17_-removebg-preview.png', productionDate: '16.9.2025', expiryDate: '10.2.2027', ingredients: 'سكر، حليب، كاكاو.', stock: 4, calories: 85 },
            { id: '15', name: 'شوكلاتة بريك سوبا', price: 0.10, currency: 'د.أ', category: 'شوكولاتة', image: 'https://uploads.onecompiler.io/43n8uttmw/445tdx6b2/IMG_6071-IDX-removebg-preview.png', productionDate: '16.9.2025', expiryDate: '15.2.2027', ingredients: 'سكر، كاكاو، مصل الحليب.', stock: 8, calories: 93 },
            { id: '7', name: 'شيبس بيوقلز جبنة الناشو', price: 0.30, currency: 'د.أ', category: 'شيبس', image: 'https://uploads.onecompiler.io/43n8uttmw/445t3c2fr/IMG-7394_370x370-removebg-preview.png', productionDate: '3.11.2025', expiryDate: '2.6.2026', ingredients: 'ذرة، زيت، نكهة جبنة.', stock: 1, calories: 524 },
            { id: '12', name: 'شيبس روكي', price: 0.10, currency: 'د.أ', category: 'شيبس', image: 'https://uploads.onecompiler.io/43n8uttmw/445t3c2fr/06-18-2025_0139pm2dbfc7c21d37480d6397-removebg-preview.png', productionDate: '23.10.2025', expiryDate: '23.4.2026', ingredients: 'بطاطا، زيت، ملح.', stock: 1, calories: 51 },
            { id: '9', name: 'شيبس سناك ميكس', price: 0.10, currency: 'د.أ', category: 'شيبس', image: 'https://uploads.onecompiler.io/43n8uttmw/445t3c2fr/sd465465f4654dfs654-removebg-preview.png', productionDate: '13.10.2025', expiryDate: '12.7.2026', ingredients: 'زيت، بابريكا، ملح.', stock: 0, calories: 488 },
            { id: '11', name: 'شيبس كانتينا', price: 0.10, currency: 'د.أ', category: 'شيبس', image: 'https://uploads.onecompiler.io/43n8uttmw/445t3c2fr/JOR-6251040001050_800x.webp', productionDate: '13.10.2025', expiryDate: '13.4.2026', ingredients: 'ذرة، زيت، نكهة.', stock: 0, calories: 50 },
            { id: '13', name: 'شيبس بينج رينج', price: 0.10, currency: 'د.أ', category: 'شيبس', image: 'https://uploads.onecompiler.io/43n8uttmw/445t3c2fr/tBVYLZQLm7O4ul18hNZQw0M7OuQpYRmUTTSo1Ut1-removebg-preview.png', productionDate: '17.11.2025', expiryDate: '18.8.2026', ingredients: 'ذرة، زيت، نكهة جبنة.', stock: 0, calories: 470 },
            { id: '8', name: 'شيبس تريتوس', price: 0.10, currency: 'د.أ', category: 'شيبس', image: 'https://uploads.onecompiler.io/43n8uttmw/445t3c2fr/images__16_-removebg-preview.png', productionDate: '23.9.2025', expiryDate: '23.3.2026', ingredients: 'ذرة، زيت، نكهة حلوة.', stock: 0, calories: 58 },
            { id: '10', name: 'شيبس شبابيك', price: 0.10, currency: 'د.أ', category: 'شيبس', image: 'https://uploads.onecompiler.io/43n8uttmw/445t3c2fr/blackfridayoffers-2023-10-10T174155.525_800x-removebg-preview.png', productionDate: '28.10.2025', expiryDate: '28.4.2026', ingredients: 'قمح، نشا، حامض حار.', stock: 0, calories: 52 },  
            { id: '14', name: 'شيبس ريلاكس', price: 0.10, currency: 'د.أ', category: 'شيبس', image: 'https://uploads.onecompiler.io/43n8uttmw/445t3c2fr/images__15_-removebg-preview.png', productionDate: '17.11.2025', expiryDate: '16.8.2026', ingredients: 'قمح، نشا، زيت.', stock: 0, calories: 470 }, 
        ];

        const CATEGORIES = ['الكل', 'شيبس', 'شوكولاتة', 'قهوة', 'بسكويت', 'علكة']; 
        const COUPONS = { 'فرح10': 0.10, '00000000002': 0.20 }; 
        const MIN_PURCHASE = 1.00;
        const ADMIN_PASS = '452011';
        const WA_NUMBER = "0790166389";
        const MESSENGER_URL = "https://m.me/zaid.khalaileh.2025";
        const LS_STOCK_KEY = 'snack_market_stock';
        const LS_CART_KEY = 'snack_market_cart';

        let appState = {
            db: null,
            auth: null, userId: null,
            isAuthReady: false, 
            products: initialProductsData,
            cart: {}, 
            category: 'الكل',
            search: '',
            appliedCoupon: null,
            adminPendingChanges: {},
            adminClickCount: 0
        };

        const APP_ID = typeof __app_id !== 'undefined' ? __app_id : 'snack-shop-v1';
        const STOCK_PATH = `artifacts/${APP_ID}/public/data/products`;

        const loadLocalStock = () => {
            try {
                const stockJson = localStorage.getItem(LS_STOCK_KEY);
                if (stockJson) return JSON.parse(stockJson);
            } catch (e) { console.error(e); }
            const defaultStock = {};
            initialProductsData.forEach(p => defaultStock[p.id] = { productId: p.id, stock: p.stock, productionDate: p.productionDate, expiryDate: p.expiryDate });
            return defaultStock;
        };

        const saveLocalStock = () => {
            const stockMap = {};
            appState.products.forEach(p => stockMap[p.id] = { productId: p.id, stock: p.stock, productionDate: p.productionDate, expiryDate: p.expiryDate });
            localStorage.setItem(LS_STOCK_KEY, JSON.stringify(stockMap));
        };

        const loadLocalCart = () => {
            try { return JSON.parse(localStorage.getItem(LS_CART_KEY)) || {}; } catch (e) { return {}; }
        };

        const saveLocalCart = () => localStorage.setItem(LS_CART_KEY, JSON.stringify(appState.cart));

        const mergeProductsAndStock = (stockMap) => {
            appState.products = initialProductsData.map(p => {
                const live = stockMap[p.id];
                return { ...p, stock: live ? live.stock : p.stock, productionDate: live?.productionDate || p.productionDate, expiryDate: live?.expiryDate || p.expiryDate };
            });
            renderApp();
        };

        const initFirestoreStock = async (db) => {
            try {
                const stockRef = collection(db, STOCK_PATH);
                const snapshot = await getDocs(stockRef);
                if (snapshot.empty) {
                    const batch = writeBatch(db);
                    initialProductsData.forEach(p => {
                        const ref = doc(db, STOCK_PATH, p.id);
                        batch.set(ref, { productId: p.id, stock: p.stock, productionDate: p.productionDate, expiryDate: p.expiryDate });
                    });
                    await batch.commit();
                }
            } catch (e) { console.error(e); }
        };

        const initFirebase = async () => {
            let config, token, hasConfig = false;
            try {
                config = JSON.parse(typeof __firebase_config !== 'undefined' ? __firebase_config : '{}');
                token = typeof __initial_auth_token !== 'undefined' ? __initial_auth_token : null;
                if (config.projectId) hasConfig = true;
                if (!hasConfig) {
                    appState.db = 'LOCAL'; appState.isAuthReady = true;
                    mergeProductsAndStock(loadLocalStock()); appState.cart = loadLocalCart();
                    document.getElementById('offline-mode-alert').classList.remove('hidden');
                    updateCartUI(); return; 
                }
                const app = initializeApp(config);
                appState.db = getFirestore(app);
                appState.auth = getAuth(app);
                if (token) signInWithCustomToken(appState.auth, token);
                else signInAnonymously(appState.auth);
                onAuthStateChanged(appState.auth, (user) => {
                    if (user) {
                        appState.userId = user.uid;
                        initDataListeners(); initFirestoreStock(appState.db);
                    }
                    appState.isAuthReady = true; renderProducts(); 
                });
            } catch (e) {
                appState.db = 'LOCAL'; appState.isAuthReady = true;
                mergeProductsAndStock(loadLocalStock()); appState.cart = loadLocalCart();
                document.getElementById('offline-mode-alert').classList.remove('hidden');
                updateCartUI();
            }
        };

        const initDataListeners = () => {
            if (!appState.userId || appState.db === 'LOCAL') return;
            onSnapshot(query(collection(appState.db, STOCK_PATH)), (snapshot) => {
                const stockMap = {}; snapshot.docs.forEach(d => stockMap[d.id] = d.data());
                mergeProductsAndStock(stockMap); 
            });
            onSnapshot(query(collection(appState.db, `artifacts/${APP_ID}/users/${appState.userId}/cartItems`)), (snap) => {
                const newCart = {}; snap.docs.forEach(d => { const data = d.data(); newCart[d.id] = { product: data.product, quantity: data.quantity }; });
                appState.cart = newCart; updateCartUI();
            });
        };

        const els = {
            grid: document.getElementById('products-grid'), loader: document.getElementById('loading-indicator'),
            empty: document.getElementById('empty-state'), cats: document.getElementById('categories-container'),
            cartCount: document.getElementById('cart-count'), modals: { product: document.getElementById('product-modal'), cart: document.getElementById('cart-modal'), contact: document.getElementById('contact-modal'), adminAuth: document.getElementById('admin-auth-modal'), adminPanel: document.getElementById('admin-panel-modal') },
            inputs: { searchDesk: document.getElementById('desktop-search'), searchMob: document.getElementById('mobile-search') }
        };

        const calculateTotals = () => {
            const items = Object.values(appState.cart);
            const subtotal = items.reduce((sum, item) => sum + (item.product.price * item.quantity), 0);
            let discount = appState.appliedCoupon ? subtotal * appState.appliedCoupon.discount : 0;
            return { subtotal, discount, total: subtotal - discount };
        };

        const addToCart = async (product) => {
            if (!appState.isAuthReady) return showToast("انتظر قليلاً...", 'error');
            const liveProd = appState.products.find(p => p.id === product.id) || product; 
            if (liveProd.stock <= 0) return showToast("نفذت الكمية!", 'error');
            const current = appState.cart[product.id]?.quantity || 0;
            if (current + 1 > liveProd.stock) return showToast(`المتاح ${liveProd.stock} فقط`, 'error');
            const itemToSave = { product: liveProd, quantity: current + 1 };
            if (appState.db === 'LOCAL') { appState.cart[product.id] = itemToSave; saveLocalCart(); updateCartUI(); }
            else if (appState.db) { await setDoc(doc(appState.db, `artifacts/${APP_ID}/users/${appState.userId}/cartItems`, product.id), itemToSave, { merge: true }); }
            showToast(`تم إضافة ${product.name}`, 'success');
        };

        const updateCartItem = async (prodId, delta) => {
            if (!appState.isAuthReady) return;
            const item = appState.cart[prodId]; if (!item) return;
            const liveProd = appState.products.find(p => p.id === prodId) || item.product;
            const newQty = item.quantity + delta;
            if (newQty > liveProd.stock) return showToast(`المتاح ${liveProd.stock} فقط`, 'error');
            if (appState.db === 'LOCAL') { if (newQty <= 0) delete appState.cart[prodId]; else appState.cart[prodId].quantity = newQty; saveLocalCart(); updateCartUI(); }
            else { const ref = doc(appState.db, `artifacts/${APP_ID}/users/${appState.userId}/cartItems`, prodId); if (newQty <= 0) await deleteDoc(ref); else await updateDoc(ref, { quantity: newQty }); }
        };

        // FIXED TRANSACTION LOGIC
        const processOrder = async () => {
            if (!Object.keys(appState.cart).length) return false;
            if (appState.db === 'LOCAL') {
                let localStock = loadLocalStock();
                for (const item of Object.values(appState.cart)) {
                    const s = localStock[item.product.id]?.stock || 0;
                    if (s < item.quantity) return false;
                    localStock[item.product.id].stock = s - item.quantity;
                }
                localStorage.setItem(LS_STOCK_KEY, JSON.stringify(localStock));
                appState.cart = {}; saveLocalCart(); mergeProductsAndStock(localStock);
                return true;
            } 
            try {
                await runTransaction(appState.db, async (transaction) => {
                    const cartItems = Object.values(appState.cart);
                    // 1. ALL READS FIRST
                    const readTasks = cartItems.map(item => transaction.get(doc(appState.db, STOCK_PATH, item.product.id)));
                    const snapshots = await Promise.all(readTasks);
                    
                    // 2. LOGIC CHECK
                    snapshots.forEach((snap, idx) => {
                        const stock = snap.exists() ? snap.data().stock : 0;
                        if (stock < cartItems[idx].quantity) throw new Error("Stock low");
                    });

                    // 3. ALL WRITES AFTER
                    snapshots.forEach((snap, idx) => {
                        const item = cartItems[idx];
                        const newStock = snap.data().stock - item.quantity;
                        transaction.update(doc(appState.db, STOCK_PATH, item.product.id), { stock: newStock });
                        transaction.delete(doc(appState.db, `artifacts/${APP_ID}/users/${appState.userId}/cartItems`, item.product.id));
                    });
                });
                return true;
            } catch (e) { console.error(e); return false; }
        };

        const renderCategories = () => {
            els.cats.innerHTML = CATEGORIES.map(c => `<button class="px-6 py-2 rounded-full whitespace-nowrap font-medium ${appState.category === c ? 'bg-orange-600 text-white' : 'bg-white text-gray-600 border'}" onclick="window.setCategory('${c}')">${c}</button>`).join('');
        };

        const renderProducts = () => {
            els.loader.classList.add('hidden');
            let filtered = appState.products.filter(p => (appState.category === 'الكل' || p.category === appState.category) && (p.name && p.name.includes(appState.search)));
            filtered.sort((a, b) => (a.stock === 0) - (b.stock === 0));
            if (!filtered.length) { els.grid.innerHTML = ''; els.empty.classList.remove('hidden'); return; }
            els.empty.classList.add('hidden');
            els.grid.innerHTML = filtered.map(p => {
                const isOut = p.stock === 0;
                return `<div class="bg-white rounded-xl shadow-sm border p-4 flex flex-col hover:shadow-lg cursor-pointer" onclick="${isOut ? '' : `window.openDetails('${p.id}')`}">
                    <img src="${p.image}" class="h-40 object-contain mb-4 ${isOut ? 'grayscale' : ''}">
                    <h3 class="font-bold mb-1">${p.name}</h3>
                    <div class="mt-auto flex justify-between items-center">
                        <span class="font-bold text-orange-600">${p.price.toFixed(2)} د.أ</span>
                        <button class="bg-gray-900 text-white p-2 rounded-full" onclick="event.stopPropagation(); window.addItem('${p.id}')" ${isOut ? 'disabled' : ''}>${isOut ? 'نفذت' : '<i data-lucide="shopping-cart" class="w-4 h-4"></i>'}</button>
                    </div>
                </div>`;
            }).join('');
            lucide.createIcons();
        };

        const updateCartUI = () => {
            const items = Object.values(appState.cart);
            els.cartCount.textContent = items.reduce((s, i) => s + i.quantity, 0);
            els.cartCount.classList.toggle('hidden', els.cartCount.textContent == '0');
            const totals = calculateTotals();
            document.getElementById('cart-items-container').innerHTML = items.length ? items.map(item => `<div class="flex justify-between items-center bg-gray-50 p-3 rounded-lg border">
                <div><p class="font-bold">${item.product.name}</p><p class="text-sm">${item.product.price.toFixed(2)} د.أ</p></div>
                <div class="flex items-center gap-2">
                    <button onclick="window.updateQty('${item.product.id}', -1)" class="w-8 h-8 border rounded-full">-</button>
                    <span>${item.quantity}</span>
                    <button onclick="window.updateQty('${item.product.id}', 1)" class="w-8 h-8 border rounded-full">+</button>
                </div>
            </div>`).join('') : '<p class="text-center py-4">السلة فارغة</p>';
            document.getElementById('cart-subtotal').innerText = totals.subtotal.toFixed(2);
            document.getElementById('cart-total').innerText = totals.total.toFixed(2);
        };

        window.setCategory = (c) => { appState.category = c; renderApp(); };
        window.addItem = (id) => addToCart(appState.products.find(p => p.id === id));
        window.updateQty = (id, d) => updateCartItem(id, d);
        window.openDetails = (id) => {
            const p = appState.products.find(x => x.id === id); if (!p) return;
            document.getElementById('product-modal-body').innerHTML = `<div class="p-6">
                <img src="${p.image}" class="h-60 w-full object-contain mb-4">
                <h2 class="text-2xl font-bold">${p.name}</h2>
                <p class="text-orange-600 font-bold text-xl my-2">${p.price.toFixed(2)} د.أ</p>
                <p class="text-gray-600 mb-4">${p.ingredients}</p>
                <button onclick="window.addItem('${p.id}')" class="w-full bg-orange-600 text-white py-3 rounded-xl font-bold">إضافة للسلة</button>
            </div>`;
            toggleModal('product', true);
        };

        const toggleModal = (n, s) => { const e = els.modals[n]; if (s) e.classList.remove('hidden'); setTimeout(() => { if (s) e.classList.add('active'); else { e.classList.remove('active'); setTimeout(() => e.classList.add('hidden'), 300); }}, 10); };
        document.querySelectorAll('.close-modal').forEach(b => b.onclick = (e) => toggleModal(e.target.closest('.modal').id.replace('-modal',''), false));
        document.getElementById('cart-btn').onclick = () => toggleModal('cart', true);
        document.getElementById('checkout-btn').onclick = () => { toggleModal('cart', false); document.getElementById('contact-total-display').innerText = calculateTotals().total.toFixed(2) + ' د.أ'; toggleModal('contact', true); };

        const getOrderMsg = (enc = true, wa = true) => {
            const t = calculateTotals(); const name = document.getElementById('customer-name').value.trim();
            const b = (x) => wa ? `*${x}*` : x;
            const items = Object.values(appState.cart).map(i => `${b('(' + i.quantity + ')')} ${i.product.name} - ${(i.quantity * i.product.price).toFixed(2)} د.أ`).join('\n');
            let m = `${b('طلب جديد من سناكس ماركت')} 🛍️\n\n${b('👤 العميل:')} ${name}\n\n${b('📋 الطلبات:')}\n${items}\n\n${b('💰 المجموع:')} ${t.total.toFixed(2)} د.أ\n\nالرجاء التوصيل! 🚀`;
            return enc ? encodeURIComponent(m) : m;
        };

        document.getElementById('wa-btn').onclick = async () => {
            if (document.getElementById('customer-name').value.trim().length < 3) return;
            if (await processOrder()) {
                window.open(`https://wa.me/${WA_NUMBER.replace(/^0/, '962')}?text=${getOrderMsg()}`, '_blank');
                toggleModal('contact', false); showToast("تم تأكيد الطلب!", 'success');
            } else showToast("عذراً، نفذ المخزون!", 'error');
        };

        const renderApp = () => { renderCategories(); renderProducts(); };
        const showToast = (m, t) => {
            const el = document.createElement('div'); el.className = `p-4 rounded shadow-lg text-white ${t === 'success' ? 'bg-green-600' : 'bg-red-600'}`; el.textContent = m;
            document.getElementById('toast-container').appendChild(el); setTimeout(() => el.remove(), 3000);
        };

        initFirebase();
    </script>
</body>
</html>
