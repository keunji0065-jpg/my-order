<!DOCTYPE html>
<html lang="ko">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>THE SELLER KING: 황제의 장부</title>
    <!-- Core Libraries (CDN) -->
    <script src="https://unpkg.com/vue@3/dist/vue.global.prod.js"></script>
    <script src="https://cdn.tailwindcss.com"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/xlsx/0.18.5/xlsx.full.min.js"></script>
    <link href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.0.0/css/all.min.css" rel="stylesheet">
    <script src="//t1.daumcdn.net/mapjsapi/bundle/postcode/prod/postcode.v2.js"></script>

    <style>
        @import url('https://cdn.jsdelivr.net/gh/orioncactus/pretendard/dist/web/static/pretendard.css');
        
        /* Royal Gold Theme */
        body { 
            font-family: 'Pretendard', sans-serif; 
            background: linear-gradient(135deg, #1a1a2e 0%, #16213e 100%); 
            color: #e2e8f0;
            min-height: 100vh; 
        }
        
        .glass-panel {
            background: rgba(30, 41, 59, 0.7);
            backdrop-filter: blur(16px);
            -webkit-backdrop-filter: blur(16px);
            border: 1px solid rgba(255, 215, 0, 0.2); /* Gold Border */
            box-shadow: 0 8px 32px 0 rgba(0, 0, 0, 0.5);
        }

        .glass-input {
            background: rgba(15, 23, 42, 0.6);
            border: 1px solid rgba(255, 255, 255, 0.1);
            color: #fff;
            transition: all 0.3s ease;
        }
        .glass-input:focus {
            background: rgba(15, 23, 42, 0.9);
            outline: none;
            border-color: #fbbf24; /* Gold Focus */
            box-shadow: 0 0 0 3px rgba(251, 191, 36, 0.2);
        }

        /* Gold Gradient Text */
        .text-gold-gradient {
            background: linear-gradient(to right, #fbf8cc, #fbbf24, #d97706);
            -webkit-background-clip: text;
            -webkit-text-fill-color: transparent;
        }
        
        /* Gold Button */
        .btn-gold {
            background: linear-gradient(to right, #d97706, #fbbf24);
            color: #1a1a2e;
        }
        .btn-gold:hover {
            filter: brightness(1.1);
        }

        ::-webkit-scrollbar { width: 8px; height: 8px; }
        ::-webkit-scrollbar-track { background: rgba(0, 0, 0, 0.2); }
        ::-webkit-scrollbar-thumb { background: #d97706; border-radius: 4px; }
    </style>
</head>
<body class="antialiased overflow-hidden">
    <div id="app" class="h-screen flex flex-col">
        
        <!-- Header: The Throne Room -->
        <header class="glass-panel m-4 p-5 rounded-2xl flex-none z-10 relative overflow-hidden">
            <!-- Background Decoration -->
            <div class="absolute top-0 right-0 -mt-10 -mr-10 w-40 h-40 bg-yellow-500 blur-3xl opacity-10 rounded-full pointer-events-none"></div>

            <div class="flex justify-between items-center mb-5 relative z-10">
                <div class="flex items-center gap-4">
                    <!-- Logo Area -->
                    <div class="w-12 h-12 bg-gradient-to-br from-yellow-300 to-yellow-600 rounded-xl flex items-center justify-center text-slate-900 text-2xl shadow-xl shadow-yellow-900/50">
                        <i class="fas fa-crown"></i>
                    </div>
                    <div>
                        <h1 class="text-3xl font-extrabold text-gold-gradient tracking-tight">THE SELLER KING</h1>
                        <p class="text-xs text-gray-400 tracking-widest uppercase">Supreme Order Management System</p>
                    </div>
                </div>
                <div class="flex gap-3">
                    <button @click="showProductModal = true" class="px-4 py-2 bg-slate-700 hover:bg-slate-600 text-gray-200 rounded-lg text-sm font-semibold transition border border-slate-600"><i class="fas fa-box-open mr-2 text-yellow-400"></i>상품 관리</button>
                    <button @click="backupData" class="px-4 py-2 bg-slate-700 hover:bg-slate-600 text-gray-200 rounded-lg text-sm font-semibold transition border border-slate-600"><i class="fas fa-save mr-2 text-blue-400"></i>백업</button>
                    <button @click="clearAllData" class="px-4 py-2 bg-red-900/30 hover:bg-red-900/50 text-red-400 border border-red-900/50 rounded-lg text-sm font-semibold transition"><i class="fas fa-trash-alt mr-2"></i>초기화</button>
                </div>
            </div>

            <!-- Dashboard Stats (Gold & Dark) -->
            <div class="grid grid-cols-1 md:grid-cols-4 gap-4">
                <!-- Summary -->
                <div class="bg-slate-800/50 rounded-xl p-4 border border-slate-700">
                    <div class="text-xs text-gray-400 font-bold uppercase tracking-wider">총 예상 매출</div>
                    <div class="text-2xl font-bold text-yellow-400 mt-1">{{ formatCurrency(dashboard.totalRevenue) }}</div>
                    <div class="text-xs text-gray-500 mt-2">총 주문 {{ dashboard.orderCount }}건</div>
                </div>
                
                <!-- Revenue Breakdown -->
                <div class="bg-slate-800/50 rounded-xl p-4 border border-slate-700 text-sm space-y-2">
                    <div class="flex justify-between text-gray-300"><span>💳 카드</span> <span class="font-bold text-white">{{ formatCurrency(dashboard.cardRevenue) }}</span></div>
                    <div class="flex justify-between text-gray-300"><span>💵 현금</span> <span class="font-bold text-white">{{ formatCurrency(dashboard.cashRevenue) }}</span></div>
                    <div class="flex justify-between text-gray-300"><span>🚚 배송비</span> <span class="font-bold text-white">{{ formatCurrency(dashboard.shippingRevenue) }}</span></div>
                </div>

                <!-- Unpaid (Alert) -->
                <div class="bg-red-900/20 rounded-xl p-4 border border-red-900/40">
                    <div class="text-xs text-red-400 font-bold uppercase tracking-wider flex justify-between items-center">
                        미수금 경고 <i class="fas fa-exclamation-triangle"></i>
                    </div>
                    <div class="text-2xl font-bold text-red-500 mt-1">{{ formatCurrency(dashboard.totalUnpaid) }}</div>
                    <div class="text-xs text-red-400 mt-2 opacity-80">
                        상품: {{ formatCurrency(dashboard.productUnpaid) }} | 배송: {{ formatCurrency(dashboard.shippingUnpaid) }}
                    </div>
                </div>

                <!-- Export Controls -->
                <div class="bg-slate-800/50 rounded-xl p-4 border border-slate-700 flex flex-col gap-2 justify-center">
                    <button @click="exportExcel('all')" class="w-full btn-gold font-bold py-2 rounded-lg text-sm transition shadow-lg shadow-yellow-900/20 flex justify-center items-center">
                        <i class="fas fa-file-excel mr-2"></i> 전체 장부 다운로드
                    </button>
                    <div class="flex gap-2">
                        <button @click="exportExcel('invoice')" class="flex-1 bg-slate-700 hover:bg-slate-600 text-gray-300 py-1.5 rounded text-xs border border-slate-600">송장용</button>
                        <button @click="exportExcel('order')" class="flex-1 bg-slate-700 hover:bg-slate-600 text-gray-300 py-1.5 rounded text-xs border border-slate-600">발주용</button>
                    </div>
                </div>
            </div>
        </header>

        <!-- Main Workspace -->
        <main class="flex-1 flex gap-5 px-5 pb-5 overflow-hidden">
            
            <!-- Left: Command Center -->
            <div class="w-1/3 glass-panel rounded-2xl p-5 flex flex-col shadow-2xl">
                <h2 class="text-lg font-bold mb-4 text-yellow-500 flex items-center"><i class="fas fa-scroll mr-2"></i>주문 명령 입력</h2>
                
                <div class="relative flex-1 mb-5 group">
                    <textarea v-model="rawInput" 
                        placeholder="이곳에 주문 내역을 붙여넣으십시오.&#13;&#10;예) 홍길동 010-1234-5678...&#13;&#10;VIP상품 2개"
                        class="w-full h-full glass-input rounded-xl p-4 resize-none text-sm leading-relaxed placeholder-slate-500"></textarea>
                    <button @click="processInput" class="absolute bottom-4 right-4 btn-gold text-slate-900 font-bold px-5 py-2 rounded-lg text-sm shadow-lg hover:shadow-yellow-500/20 transition transform hover:-translate-y-1">
                        <i class="fas fa-check mr-1"></i> 입력 실행
                    </button>
                </div>

                <!-- Manual Input -->
                <div class="space-y-3 bg-slate-900/40 p-4 rounded-xl border border-slate-700/50">
                    <div class="flex gap-2">
                        <input v-model="manual.nickname" placeholder="닉네임" class="glass-input w-1/3 px-3 py-2 rounded-lg text-sm">
                        <div class="relative flex-1">
                            <input v-model="manual.product" @input="filterProducts" placeholder="상품 (자동완성)" class="glass-input w-full px-3 py-2 rounded-lg text-sm">
                            <div v-if="showSuggestions && filteredProducts.length" class="absolute z-50 w-full bg-slate-800 border border-slate-600 rounded-lg shadow-xl mt-1 max-h-40 overflow-y-auto text-gray-200">
                                <div v-for="p in filteredProducts" @click="selectProduct(p)" class="px-3 py-2 hover:bg-slate-700 cursor-pointer text-sm flex justify-between">
                                    <span>{{p.name}}</span>
                                    <span class="text-yellow-500">{{formatCurrency(p.price)}}</span>
                                </div>
                            </div>
                        </div>
                        <input v-model.number="manual.qty" type="number" placeholder="수" class="glass-input w-14 px-2 py-2 rounded-lg text-sm text-center">
                    </div>
                    
                    <div class="flex gap-2">
                        <input v-model="manual.receiver" placeholder="수령인" class="glass-input w-1/4 px-3 py-2 rounded-lg text-sm">
                        <input v-model="manual.phone" placeholder="연락처" class="glass-input w-1/3 px-3 py-2 rounded-lg text-sm">
                        <button @click="openDaumPostcode" class="bg-slate-700 text-gray-300 border border-slate-600 px-3 rounded-lg text-xs hover:bg-slate-600">주소찾기</button>
                    </div>
                    <input v-model="manual.address" placeholder="배송지 주소" class="glass-input w-full px-3 py-2 rounded-lg text-sm">
                    
                    <button @click="addOrder" class="w-full bg-slate-700 hover:bg-slate-600 text-yellow-400 border border-yellow-500/30 py-2 rounded-lg font-bold shadow-md transition">
                        <i class="fas fa-plus mr-1"></i> 리스트 추가
                    </button>
                </div>
            </div>

            <!-- Right: The Ledger -->
            <div class="flex-1 glass-panel rounded-2xl flex flex-col shadow-2xl overflow-hidden">
                <!-- Toolbar -->
                <div class="p-4 border-b border-slate-700 flex justify-between items-center bg-slate-800/30">
                    <div class="flex items-center gap-2">
                        <i class="fas fa-list-alt text-yellow-500"></i>
                        <span class="font-bold text-gray-200">주문 현황 ({{ filteredOrderList.length }})</span>
                    </div>
                    <div class="relative">
                        <i class="fas fa-search absolute left-3 top-1/2 transform -translate-y-1/2 text-slate-400"></i>
                        <input v-model="searchQuery" placeholder="검색..." class="glass-input pl-9 pr-4 py-1.5 rounded-full text-sm w-64 bg-slate-900/50">
                    </div>
                </div>

                <!-- Table Header -->
                <div class="grid grid-cols-12 gap-2 p-3 bg-slate-900/50 text-xs font-bold text-gray-400 border-b border-slate-700 uppercase tracking-wide">
                    <div class="col-span-1 text-center">No</div>
                    <div class="col-span-1">닉네임</div>
                    <div class="col-span-3">상품정보</div>
                    <div class="col-span-2 text-right">금액/미수</div>
                    <div class="col-span-3 text-center">결제 (배송/카드/현금)</div>
                    <div class="col-span-2 text-center">관리</div>
                </div>

                <!-- Table Body -->
                <div class="flex-1 overflow-y-auto p-2 space-y-2">
                    <div v-for="(order, index) in filteredOrderList" :key="order.id" 
                        class="grid grid-cols-12 gap-2 p-3 rounded-xl items-center transition duration-200 border border-transparent hover:border-yellow-500/30 hover:bg-slate-800/80 bg-slate-800/40 shadow-sm group">
                        
                        <div class="col-span-1 text-center text-xs text-gray-500">{{ filteredOrderList.length - index }}</div>
                        
                        <div class="col-span-1 font-bold text-gray-200 truncate" :title="order.nickname">{{ order.nickname }}</div>
                        
                        <div class="col-span-3">
                            <div class="font-medium text-sm text-yellow-100 truncate">{{ order.product }}</div>
                            <div class="text-xs text-gray-500">
                                {{ formatCurrency(order.unitPrice) }} x {{ order.qty }}
                                <span v-if="isFreeShippingGroup(order.nickname) && order.id === getShippingMasterId(order.nickname)" class="ml-1 text-yellow-500 text-[10px] border border-yellow-500/30 px-1 rounded">배송비</span>
                            </div>
                        </div>

                        <div class="col-span-2 text-right">
                            <div class="font-bold text-gray-300">{{ formatCurrency(calculateOrderTotal(order)) }}</div>
                            <div class="text-xs font-bold" :class="calculateUnpaid(order) > 0 ? 'text-red-400' : 'text-green-400'">
                                미수: {{ formatCurrency(calculateUnpaid(order)) }}
                            </div>
                        </div>

                        <!-- Payment Controls -->
                        <div class="col-span-3 flex flex-col gap-1 items-center">
                            <div v-if="shouldShowShippingToggle(order)" class="flex items-center justify-between w-full bg-slate-900/50 px-2 py-1 rounded border border-slate-700">
                                <span class="text-[10px] text-gray-400"><i class="fas fa-truck"></i></span>
                                <button @click="toggleShipping(order)" 
                                    class="w-8 h-4 rounded-full transition-colors relative"
                                    :class="order.isShippingPaid ? 'bg-green-600' : 'bg-slate-600'">
                                    <div class="absolute top-0.5 left-0.5 w-3 h-3 bg-white rounded-full transition-transform"
                                        :class="order.isShippingPaid ? 'translate-x-4' : ''"></div>
                                </button>
                            </div>
                            
                            <div class="flex gap-1 w-full">
                                <input v-model.number="order.cardAmount" placeholder="카드" class="w-1/2 text-xs p-1 rounded bg-slate-700 text-gray-200 text-right focus:bg-slate-600 border border-transparent focus:border-yellow-500/50 outline-none placeholder-slate-500">
                                <input v-model.number="order.depositAmount" placeholder="입금" class="w-1/2 text-xs p-1 rounded bg-slate-700 text-gray-200 text-right focus:bg-slate-600 border border-transparent focus:border-yellow-500/50 outline-none placeholder-slate-500">
                            </div>
                        </div>

                        <!-- Actions -->
                        <div class="col-span-2 flex justify-center gap-3">
                            <button @click="copyOrderInfo(order)" class="text-gray-500 hover:text-yellow-400 transition" title="복사">
                                <i class="fas fa-copy"></i>
                            </button>
                            <button @click="deleteOrder(order.id)" class="text-gray-500 hover:text-red-400 transition" title="삭제">
                                <i class="fas fa-times"></i>
                            </button>
                        </div>
                    </div>
                </div>
            </div>
        </main>

        <!-- Product Modal (Dark Theme) -->
        <div v-if="showProductModal" class="fixed inset-0 z-50 flex items-center justify-center bg-black/70 backdrop-blur-sm">
            <div class="glass-panel bg-slate-800 rounded-2xl shadow-2xl w-96 p-6 border border-slate-600">
                <h3 class="text-lg font-bold mb-4 text-yellow-500">상품 단가 관리</h3>
                <div class="flex gap-2 mb-4">
                    <input v-model="newProduct.name" placeholder="상품명" class="glass-input p-2 rounded w-full text-sm">
                    <input v-model.number="newProduct.price" placeholder="단가" type="number" class="glass-input p-2 rounded w-24 text-sm">
                    <button @click="saveProduct" class="btn-gold px-3 rounded hover:opacity-90 text-slate-900 font-bold"><i class="fas fa-plus"></i></button>
                </div>
                <ul class="max-h-60 overflow-y-auto space-y-2 mb-4 pr-1">
                    <li v-for="(p, i) in products" class="flex justify-between items-center p-2 bg-slate-700/50 rounded border border-slate-700">
                        <span class="text-gray-300">{{ p.name }}</span>
                        <div class="flex items-center gap-3">
                            <span class="font-bold text-yellow-500">{{ formatCurrency(p.price) }}</span>
                            <button @click="removeProduct(i)" class="text-red-400 hover:text-red-300"><i class="fas fa-times"></i></button>
                        </div>
                    </li>
                </ul>
                <div class="flex justify-end gap-2">
                     <button @click="updateExistingOrders" class="text-xs text-yellow-600 underline mr-auto self-center hover:text-yellow-500">단가 일괄 적용</button>
                    <button @click="showProductModal = false" class="bg-slate-700 px-4 py-2 rounded text-gray-300 hover:bg-slate-600 text-sm">닫기</button>
                </div>
            </div>
        </div>

    </div>

    <script>
        const { createApp } = Vue;

        createApp({
            data() {
                return {
                    orders: [],
                    products: [
                        { name: "황제의하사품", price: 50000 },
                        { name: "로열에디션", price: 120000 }
                    ],
                    manual: { nickname: '', product: '', qty: 1, receiver: '', phone: '', address: '' },
                    rawInput: '',
                    searchQuery: '',
                    showProductModal: false,
                    showSuggestions: false,
                    newProduct: { name: '', price: '' },
                    defaultShippingCost: 3000
                }
            },
            computed: {
                filteredProducts() {
                    if (!this.manual.product) return [];
                    return this.products.filter(p => p.name.includes(this.manual.product));
                },
                filteredOrderList() {
                    const query = this.searchQuery.toLowerCase();
                    return this.orders.filter(o => 
                        o.nickname.toLowerCase().includes(query) || 
                        (o.phone && o.phone.includes(query))
                    ).sort((a, b) => b.timestamp - a.timestamp);
                },
                dashboard() {
                    let totalRevenue = 0, cardRevenue = 0, cashRevenue = 0, shippingRevenue = 0;
                    let productUnpaid = 0, shippingUnpaid = 0;
                    let totalQty = 0;

                    const nicknameGroups = {};
                    
                    this.filteredOrderList.forEach(order => {
                        totalQty += order.qty;
                        
                        const itemPrice = order.unitPrice * order.qty;
                        const isShippingMaster = this.getShippingMasterId(order.nickname) === order.id;
                        const shippingCost = isShippingMaster ? this.defaultShippingCost : 0;

                        totalRevenue += itemPrice + shippingCost;
                        cardRevenue += order.cardAmount;
                        cashRevenue += order.depositAmount;

                        let sUnpaid = 0;
                        if (isShippingMaster && !order.isShippingPaid) {
                            sUnpaid = this.defaultShippingCost;
                        }
                        
                        const totalDue = (itemPrice + shippingCost) - order.cardAmount - order.depositAmount;
                        let pUnpaid = totalDue - sUnpaid;

                        shippingUnpaid += sUnpaid;
                        productUnpaid += pUnpaid;
                        
                        if (order.isShippingPaid && isShippingMaster) {
                            shippingRevenue += this.defaultShippingCost;
                        }
                    });

                    return {
                        totalRevenue, cardRevenue, cashRevenue, shippingRevenue,
                        totalUnpaid: productUnpaid + shippingUnpaid,
                        productUnpaid, shippingUnpaid,
                        orderCount: this.filteredOrderList.length,
                        totalQty
                    };
                }
            },
            mounted() {
                this.loadState();
                setInterval(this.saveState, 5000);
            },
            methods: {
                formatCurrency(value) {
                    return new Intl.NumberFormat('ko-KR', { style: 'currency', currency: 'KRW' }).format(value);
                },
                getShippingMasterId(nickname) {
                    const orders = this.orders.filter(o => o.nickname === nickname).sort((a, b) => a.timestamp - b.timestamp);
                    return orders.length > 0 ? orders[0].id : null;
                },
                isFreeShippingGroup(nickname) {
                    return true;
                },
                shouldShowShippingToggle(order) {
                    return this.getShippingMasterId(order.nickname) === order.id;
                },
                calculateOrderTotal(order) {
                    let total = order.unitPrice * order.qty;
                    if (this.shouldShowShippingToggle(order)) {
                        total += this.defaultShippingCost;
                    }
                    return total;
                },
                calculateUnpaid(order) {
                    const totalRequired = this.calculateOrderTotal(order);
                    const paid = order.cardAmount + order.depositAmount;
                    return totalRequired - paid;
                },
                toggleShipping(order) {
                    order.isShippingPaid = !order.isShippingPaid;
                },
                processInput() {
                    const lines = this.rawInput.split('\n');
                    lines.forEach(line => {
                        if(!line.trim()) return;
                        
                        let nameMatch = line.match(/^([가-힣]{2,4})/);
                        let phoneMatch = line.match(/010-?([0-9]{4})-?([0-9]{4})/);
                        
                        let foundProduct = this.products.find(p => line.includes(p.name));
                        
                        if (nameMatch || phoneMatch) {
                            this.manual.nickname = nameMatch ? nameMatch[0] : this.manual.nickname;
                            this.manual.receiver = nameMatch ? nameMatch[0] : '';
                            this.manual.phone = phoneMatch ? `010-${phoneMatch[1]}-${phoneMatch[2]}` : '';
                            this.manual.address = line.replace(nameMatch?.[0], '').replace(phoneMatch?.[0], '').trim();
                        } else if (foundProduct) {
                            this.manual.product = foundProduct.name;
                            let qtyMatch = line.match(/([0-9]+)개/) || line.match(/\s([0-9]+)\s*$/);
                            this.manual.qty = qtyMatch ? parseInt(qtyMatch[1]) : 1;
                            this.addOrder();
                        }
                    });
                    this.rawInput = '';
                },
                filterProducts() {
                    this.showSuggestions = true;
                },
                selectProduct(p) {
                    this.manual.product = p.name;
                    this.showSuggestions = false;
                },
                addOrder() {
                    if (!this.manual.nickname || !this.manual.product) {
                        alert("닉네임과 상품명 입력은 필수입니다, 폐하.");
                        return;
                    }
                    
                    const productInfo = this.products.find(p => p.name === this.manual.product) || { price: 0 };
                    
                    this.orders.push({
                        id: Date.now(),
                        timestamp: Date.now(),
                        nickname: this.manual.nickname,
                        product: this.manual.product,
                        unitPrice: productInfo.price,
                        qty: this.manual.qty,
                        receiver: this.manual.receiver,
                        phone: this.manual.phone,
                        address: this.manual.address,
                        cardAmount: 0,
                        depositAmount: 0,
                        isShippingPaid: false,
                    });
                    
                    this.manual.product = '';
                    this.manual.qty = 1;
                },
                openDaumPostcode() {
                    new daum.Postcode({
                        oncomplete: (data) => {
                            this.manual.address = data.roadAddress || data.jibunAddress;
                            if(data.buildingName) this.manual.address += ` (${data.buildingName})`;
                        }
                    }).open();
                },
                saveProduct() {
                    if(this.newProduct.name && this.newProduct.price) {
                        this.products.push({...this.newProduct});
                        this.newProduct = { name: '', price: '' };
                    }
                },
                removeProduct(index) {
                    this.products.splice(index, 1);
                },
                updateExistingOrders() {
                    if(!confirm("모든 백성의 주문 단가를 현재 시세로 갱신하시겠습니까?")) return;
                    this.orders.forEach(o => {
                        const p = this.products.find(prod => prod.name === o.product);
                        if(p) o.unitPrice = p.price;
                    });
                    alert("시세 갱신 완료");
                },
                deleteOrder(id) {
                    if(confirm("이 주문을 파기하시겠습니까?")) {
                        this.orders = this.orders.filter(o => o.id !== id);
                    }
                },
                copyOrderInfo(order) {
                    const unpaid = this.calculateUnpaid(order);
                    const text = `[THE SELLER KING 주문서]\n${order.product} ${order.qty}개\n총액: ${this.formatCurrency(this.calculateOrderTotal(order))}\n잔금: ${this.formatCurrency(unpaid)}\n계좌: 황제은행 999-999-999999`;
                    navigator.clipboard.writeText(text).then(() => alert("클립보드에 복사되었습니다."));
                },
                saveState() {
                    const data = {
                        orders: this.orders,
                        products: this.products
                    };
                    localStorage.setItem('omProData', JSON.stringify(data));
                },
                loadState() {
                    const saved = localStorage.getItem('omProData');
                    if (saved) {
                        const data = JSON.parse(saved);
                        this.orders = data.orders || [];
                        this.products = data.products || [];
                    }
                },
                backupData() {
                    const dataStr = "data:text/json;charset=utf-8," + encodeURIComponent(JSON.stringify({orders: this.orders, products: this.products}));
                    const downloadAnchorNode = document.createElement('a');
                    downloadAnchorNode.setAttribute("href", dataStr);
                    downloadAnchorNode.setAttribute("download", "SellerKing_Backup_" + new Date().toISOString().slice(0,10) + ".json");
                    document.body.appendChild(downloadAnchorNode);
                    downloadAnchorNode.click();
                    downloadAnchorNode.remove();
                },
                clearAllData() {
                    if(confirm("모든 장부를 소각하시겠습니까? (복구 불가)")) {
                        this.orders = [];
                        localStorage.removeItem('omProData');
                    }
                },
                exportExcel(type) {
                    let data = [];
                    if (type === 'all') {
                        data = this.filteredOrderList.map(o => ({
                            '닉네임': o.nickname,
                            '상품명': o.product,
                            '수량': o.qty,
                            '단가': o.unitPrice,
                            '수령인': o.receiver,
                            '전화번호': `\u200B${o.phone}`,
                            '주소': o.address,
                            '카드결제': o.cardAmount,
                            '현금입금': o.depositAmount,
                            '배송비납부': o.isShippingPaid ? 'O' : 'X',
                            '미수금': this.calculateUnpaid(o)
                        }));
                    } else if (type === 'invoice') {
                        data = this.filteredOrderList.map(o => ({
                            '수령인': o.receiver,
                            '전화번호': `\u200B${o.phone}`,
                            '주소': o.address,
                            '품목': `${o.product} (${o.qty})`,
                            '배송메세지': o.nickname
                        }));
                    } else if (type === 'order') {
                        const summary = {};
                        this.filteredOrderList.forEach(o => {
                            if (!summary[o.product]) summary[o.product] = 0;
                            summary[o.product] += o.qty;
                        });
                        data = Object.keys(summary).map(key => ({
                            '상품명': key,
                            '총수량': summary[key]
                        }));
                    }

                    const ws = XLSX.utils.json_to_sheet(data);
                    const wb = XLSX.utils.book_new();
                    XLSX.utils.book_append_sheet(wb, ws, "Royal_Export");
                    XLSX.writeFile(wb, `SellerKing_${type}_${new Date().toISOString().slice(0,10)}.xlsx`);
                }
            }
        });
    </script>
</body>
</html>
