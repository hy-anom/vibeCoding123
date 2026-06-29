<script setup>
import { ref, computed, onMounted, onUnmounted } from 'vue';

// 1. Reactive State Variables
const products = ref([]);
const cart = ref([]);
const isLoading = ref(false);
const searchQuery = ref('');
const isCartOpen = ref(false);
const isCheckingOut = ref(false);
const toasts = ref([]);

let toastId = 0;

// 2. Computed Properties (Replacing manual updates)
// Task 4: Filter products dynamically using a computed property
const filteredProducts = computed(() => {
  const query = searchQuery.value.trim().toLowerCase();
  if (!query) return products.value;
  return products.value.filter(product => 
    product.name.toLowerCase().includes(query)
  );
});

// Task 3: Display the total quantity of all items using a computed property
const cartQuantity = computed(() => {
  return cart.value.reduce((sum, item) => sum + item.quantity, 0);
});

// Calculate the total cart price
const cartTotal = computed(() => {
  return cart.value.reduce((sum, item) => sum + item.product.price * item.quantity, 0);
});

// 3. Methods & Logic
// Simulating products API loading
const loadProducts = () => {
  isLoading.value = true;
  setTimeout(() => {
    products.value = [
      { id: 1, name: '無線藍牙耳機', price: 2999, image: 'earphones.jpg' },
      { id: 2, name: '智慧手錶', price: 8999, image: 'smartwatch.jpg' },
      { id: 3, name: '便攜式充電器', price: 1299, image: 'powerbank.jpg' },
      { id: 4, name: '無線滑鼠', price: 899, image: 'mouse.jpg' },
      { id: 5, name: '機械鍵盤', price: 3999, image: 'keyboard.jpg' },
      { id: 6, name: '網路攝影機', price: 2199, image: 'webcam.jpg' },
      { id: 7, name: 'USB隨身碟', price: 599, image: 'usb.jpg' },
      { id: 8, name: '桌面擴音器', price: 1599, image: 'speaker.jpg' }
    ];
    isLoading.value = false;
  }, 1500);
};

// Task 2: Show toast notifications when adding products
const showToast = (message) => {
  const id = toastId++;
  toasts.value.push({ id, message, show: false });
  
  // Trigger animation next tick
  setTimeout(() => {
    const item = toasts.value.find(t => t.id === id);
    if (item) item.show = true;
  }, 10);
  
  // Fade out and auto-remove after 3 seconds
  setTimeout(() => {
    const item = toasts.value.find(t => t.id === id);
    if (item) {
      item.show = false;
      setTimeout(() => {
        const index = toasts.value.findIndex(t => t.id === id);
        if (index !== -1) toasts.value.splice(index, 1);
      }, 300);
    }
  }, 3000);
};

const addToCart = (product) => {
  const existingItem = cart.value.find(item => item.product.id === product.id);
  if (existingItem) {
    existingItem.quantity += 1;
  } else {
    cart.value.push({
      product,
      quantity: 1
    });
  }
  showToast(`已將 "${product.name}" 加入購物車！`);
};

// Task 1: Create a confirmation dialog before removing items
// Task 5: Solve quantity updating bugs in reactive state
const updateQuantity = (productId, change) => {
  const index = cart.value.findIndex(item => item.product.id === productId);
  if (index === -1) return;
  
  const item = cart.value[index];
  if (item.quantity + change <= 0) {
    if (confirm('確定要將此商品從購物車中移除嗎？')) {
      cart.value.splice(index, 1);
    }
  } else {
    item.quantity += change;
  }
};

// Task 1 & 5: Removing item confirmation dialog
const removeFromCart = (productId) => {
  const index = cart.value.findIndex(item => item.product.id === productId);
  if (index !== -1) {
    if (confirm('確定要將此商品從購物車中移除嗎？')) {
      cart.value.splice(index, 1);
    }
  }
};

// Task 1: Create a confirmation dialog before checkout
const checkout = () => {
  if (cart.value.length === 0) {
    alert('購物車是空的！');
    return;
  }
  
  if (!confirm('確定要進行結帳嗎？')) {
    return;
  }
  
  isCheckingOut.value = true;
  // Simulate checkout API call
  setTimeout(() => {
    alert('結帳成功！感謝您的購買。');
    cart.value = [];
    isCartOpen.value = false;
    isCheckingOut.value = false;
  }, 2000);
};

// Handle outside clicks to close the cart sidebar
const handleOutsideClick = (e) => {
  if (!isCartOpen.value) return;
  if (e.target.closest('.cart-icon') || e.target.closest('.cart-sidebar')) {
    return;
  }
  isCartOpen.value = false;
};

onMounted(() => {
  loadProducts();
  document.addEventListener('click', handleOutsideClick);
});

onUnmounted(() => {
  document.removeEventListener('click', handleOutsideClick);
});
</script>

<template>
  <div class="container">
    <!-- Header -->
    <div class="header">
      <h1>精品商店</h1>
      <div class="cart-icon" @click.stop="isCartOpen = !isCartOpen">
        🛒 購物車
        <div class="cart-count" id="cartCount">{{ cartQuantity }}</div>
      </div>
    </div>
    
    <!-- Task 4: Search input box -->
    <div class="search-container">
      <input 
        type="text" 
        id="searchInput" 
        class="search-input" 
        placeholder="搜尋商品..." 
        v-model="searchQuery"
      >
    </div>
    
    <div class="loading" style="display: block;" v-if="isLoading">載入中...</div>
    
    <!-- Products Grid -->
    <div class="products-grid" id="productsGrid" v-else>
      <div 
        class="product-card" 
        v-for="product in filteredProducts" 
        :key="product.id"
      >
        <div class="product-image">圖片</div>
        <div class="product-info">
          <div class="product-title">{{ product.name }}</div>
          <div class="product-price">NT$ {{ product.price }}</div>
        </div>
        <button class="add-to-cart-btn" @click="addToCart(product)">加入購物車</button>
      </div>
      <div v-if="filteredProducts.length === 0" style="grid-column: 1/-1; text-align: center; color: #999; padding: 40px 0;">
        沒有找到符合的商品
      </div>
    </div>
  </div>
  
  <!-- Cart Sidebar -->
  <div class="cart-sidebar" :class="{ open: isCartOpen }" id="cartSidebar">
    <div class="cart-header">
      <h2>購物車</h2>
      <button class="close-cart" @click="isCartOpen = false">×</button>
    </div>
    <div id="cartItems">
      <div v-if="cart.length === 0" style="text-align: center; color: #999; margin-top: 50px;">
        購物車是空的
      </div>
      <div 
        class="cart-item" 
        v-for="item in cart" 
        :key="item.product.id"
        v-else
      >
        <div class="cart-item-image">圖片</div>
        <div class="cart-item-details">
          <div class="cart-item-title">{{ item.product.name }}</div>
          <div class="cart-item-price">NT$ {{ item.product.price }}</div>
          <div class="quantity-controls">
            <button class="quantity-btn" @click="updateQuantity(item.product.id, -1)">-</button>
            <span class="quantity-display">{{ item.quantity }}</span>
            <button class="quantity-btn" @click="updateQuantity(item.product.id, 1)">+</button>
          </div>
        </div>
      </div>
    </div>
    <div class="cart-total">
      <div class="total-price">總計: NT$ <span id="totalPrice">{{ cartTotal }}</span></div>
      <button 
        class="checkout-btn" 
        :disabled="isCheckingOut || cart.length === 0"
        @click="checkout"
      >
        {{ isCheckingOut ? '結帳處理中...' : '結帳' }}
      </button>
    </div>
  </div>
  
  <!-- Toast Notification Area -->
  <div class="toast-container" id="toastContainer">
    <div 
      class="toast" 
      :class="{ show: toast.show }" 
      v-for="toast in toasts" 
      :key="toast.id"
    >
      {{ toast.message }}
    </div>
  </div>
</template>
