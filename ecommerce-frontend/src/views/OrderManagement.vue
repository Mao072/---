<template>
  <div class="container">
    <div class="page-title">
      <h2> 訂單管理</h2>
    </div>
    
    <!-- User View: Create Order -->
    <div v-if="!isAdmin" class="card" style="margin-bottom: 24px;">
      <h3 style="margin-bottom: 20px;"> 選購商品</h3>
      
      <div v-if="loadingProducts" class="loading">
        <div class="spinner"></div>
      </div>
      
      <div v-else-if="products.length === 0" class="empty-state">
        <p>目前沒有可購買的商品</p>
      </div>
      
      <div v-else class="grid grid-3">
        <div 
          v-for="product in products" 
          :key="product.productId" 
          class="product-card"
          :class="{ 'out-of-stock': product.quantity === 0 }"
        >
          <h3>{{ product.productName }}</h3>
          <div class="price">NT$ {{ formatNumber(product.price) }}</div>
          <div class="stock">
            庫存: {{ product.quantity }}
            <span v-if="product.quantity === 0" style="color: #e53e3e; font-weight: bold; margin-left: 8px;">(已售完)</span>
          </div>
          
          <div class="quantity-input" style="margin-top: 16px;">
            <button @click="decreaseQuantity(product.productId)" :disabled="product.quantity === 0">−</button>
            <input 
              type="number" 
              :value="getCartQuantity(product.productId)" 
              @change="setCartQuantity(product.productId, $event.target.value, product.quantity)"
              min="0"
              :max="product.quantity"
              :disabled="product.quantity === 0"
              :placeholder="product.quantity === 0 ? '0' : ''"
            />
            <button @click="increaseQuantity(product.productId, product.quantity)" :disabled="product.quantity === 0">+</button>
          </div>
        </div>
      </div>
      
      <!-- Cart Summary -->
      <div v-if="cartTotal > 0" class="cart-summary">
        <h3> 購物車明細</h3>
        <div style="margin-bottom: 16px;">
          <div v-for="item in cartItems" :key="item.productId" style="display: flex; justify-content: space-between; margin-bottom: 8px;">
            <span>{{ item.name }} x {{ item.quantity }}</span>
            <span>NT$ {{ formatNumber(item.price * item.quantity) }}</span>
          </div>
        </div>
        <div style="display: flex; justify-content: space-between; align-items: center;">
          <span class="cart-total">總計: NT$ {{ formatNumber(cartTotal) }}</span>
          <button class="btn" style="background: white; color: #667eea; font-weight: 600;" @click="submitOrder" :disabled="submitting">
            {{ submitting ? '處理中...' : '送出訂單' }}
          </button>
        </div>
      </div>
    </div>
    
    <!-- Orders List -->
    <div class="card">
      <div style="display: flex; justify-content: space-between; align-items: center; margin-bottom: 20px;">
        <h3 style="margin: 0;">{{ isAdmin ? ' 所有訂單' : ' 我的訂單' }}</h3>
        
        <!-- Admin Filters -->
        <div v-if="isAdmin" style="display: flex; gap: 12px;">
          <input 
            v-model="searchMemberId" 
            type="text" 
            class="form-control" 
            placeholder="搜尋 Member ID..." 
            style="width: 180px; padding: 8px 12px; font-size: 14px;"
          />
          <select 
            v-model="filterStatus" 
            class="form-control" 
            style="width: 150px; padding: 8px 12px; font-size: 14px;"
          >
            <option value="">所有狀態</option>
            <option value="0">備貨中</option>
            <option value="1">運送中</option>
            <option value="2">已到達</option>
            <option value="3">已取貨</option>
            <option value="-1">已取消</option>
          </select>
        </div>
      </div>
      
      <div v-if="loadingOrders" class="loading">
        <div class="spinner"></div>
      </div>
      
      <div v-else-if="filteredOrders.length === 0" class="empty-state">
        <h3>尚無符合條件的訂單</h3>
        <p>{{ isAdmin ? '請調整搜尋條件再試一次' : '開始選購商品建立您的第一筆訂單吧！' }}</p>
      </div>
      
      <div v-else class="table-container">
        <table>
          <thead>
            <tr>
              <th>訂單編號</th>
              <th v-if="isAdmin">會員</th>
              <th>總金額</th>
              <th>付款狀態</th>
              <th>訂單狀態</th>
              <th v-if="isAdmin">操作</th>
            </tr>
          </thead>
          <tbody>
            <tr v-for="order in filteredOrders" :key="order.orderId">
              <td>{{ order.orderId }}</td>
              <td v-if="isAdmin">{{ order.memberId }}</td>
              <td>NT$ {{ formatNumber(order.totalPrice) }}</td>
              <td>
                <span class="status-badge" :class="order.payStatus === 1 ? 'status-delivered' : 'status-pending'">
                  {{ order.payStatus === 1 ? '已付款' : '未付款' }}
                </span>
                <button 
                  v-if="!isAdmin && order.payStatus === 0" 
                  class="btn btn-sm" 
                  style="margin-left: 8px; padding: 4px 8px; font-size: 12px; background: #48bb78;"
                  @click="handlePayment(order.orderId)"
                >
                  立即付款(模擬付款)
                </button>
              </td>
              <td>
                <span class="status-badge" :class="getStatusClass(order.orderStatus)">
                  {{ getStatusText(order.orderStatus) }}
                </span>
                <button 
                  v-if="!isAdmin && order.orderStatus === 2" 
                  class="btn btn-sm" 
                  style="margin-left: 8px; padding: 4px 8px; font-size: 12px; background: #ed8936;"
                  @click="handleReceive(order.orderId)"
                >
                  確認領取
                </button>
              </td>
              <td v-if="isAdmin">
                <select 
                  class="form-control" 
                  style="width: auto; padding: 8px 12px;"
                  :value="order.orderStatus"
                  @change="updateStatus(order.orderId, $event.target.value)"
                  :disabled="order.orderStatus === 3 || order.orderStatus === -1"
                >
                  <option value="0" :disabled="order.orderStatus > 0">備貨中</option>
                  <option value="1" :disabled="order.orderStatus > 1">運送中</option>
                  <option value="2" :disabled="order.orderStatus > 2">已到達</option>
                  <option value="3" :disabled="order.orderStatus > 3">已取貨</option>
                  <option value="-1" :disabled="order.orderStatus === 3">已取消</option>
                </select>
              </td>
            </tr>
          </tbody>
        </table>
      </div>
    </div>
    
    <!-- Success Modal -->
    <div v-if="showSuccess" class="modal-overlay" @click.self="showSuccess = false">
      <div class="modal" style="text-align: center;">
        <h3>訂單成立</h3>
        <p style="margin-bottom: 20px;">您的訂單已成功建立！</p>
        <button class="btn btn-primary" @click="showSuccess = false">確定</button>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, reactive, computed, onMounted } from 'vue'
import { 
  getAvailableProducts, 
  getAllProducts,
  createOrder, 
  getMyOrders, 
  getAllOrders,
  updateOrderStatus,
  payOrder
} from '@/services/api'

const isAdmin = computed(() => localStorage.getItem('role') === 'ADMIN')
const memberId = computed(() => localStorage.getItem('memberId'))

const products = ref([])
const orders = ref([])
const cart = reactive({})
const loadingProducts = ref(true)
const loadingOrders = ref(true)
const submitting = ref(false)
const showSuccess = ref(false)

const searchMemberId = ref('')
const filterStatus = ref('')

const filteredOrders = computed(() => {
  return orders.value.filter(order => {
    const matchMember = !searchMemberId.value || 
      order.memberId.toLowerCase().includes(searchMemberId.value.toLowerCase())
    const matchStatus = filterStatus.value === '' || 
      String(order.orderStatus) === filterStatus.value
    return matchMember && matchStatus
  })
})

const formatNumber = (num) => {
  return new Intl.NumberFormat('zh-TW').format(num)
}

const getStatusText = (status) => {
  const map = {
    '-1': '已取消',
    '0': '備貨中',
    '1': '運送中',
    '2': '已到達',
    '3': '已取貨'
  }
  return map[String(status)] || '未知'
}

const getStatusClass = (status) => {
  const map = {
    '-1': 'status-cancelled',
    '0': 'status-pending',
    '1': 'status-shipping',
    '2': 'status-delivered',
    '3': 'status-completed'
  }
  return map[String(status)] || ''
}

const getCartQuantity = (productId) => cart[productId] || 0

const setCartQuantity = (productId, value, max) => {
  const num = Math.max(0, Math.min(parseInt(value) || 0, max))
  if (num > 0) {
    cart[productId] = num
  } else {
    delete cart[productId]
  }
}

const increaseQuantity = (productId, max) => {
  const current = cart[productId] || 0
  if (current < max) {
    cart[productId] = current + 1
  }
}

const decreaseQuantity = (productId) => {
  const current = cart[productId] || 0
  if (current > 1) {
    cart[productId] = current - 1
  } else {
    delete cart[productId]
  }
}

const cartItems = computed(() => {
  return Object.entries(cart).map(([productId, quantity]) => {
    const product = products.value.find(p => p.productId === productId)
    return {
      productId,
      quantity,
      name: product?.productName || '',
      price: product?.price || 0
    }
  })
})

const cartTotal = computed(() => {
  return cartItems.value.reduce((sum, item) => sum + (item.price * item.quantity), 0)
})

const fetchProducts = async () => {
  loadingProducts.value = true
  try {
    const response = await getAllProducts()
    if (response.success) {
      products.value = response.data
    }
  } catch (err) {
    console.error('Failed to fetch products:', err)
  } finally {
    loadingProducts.value = false
  }
}

const fetchOrders = async () => {
  loadingOrders.value = true
  try {
    const response = isAdmin.value ? await getAllOrders() : await getMyOrders()
    if (response.success) {
      orders.value = response.data
    }
  } catch (err) {
    console.error('Failed to fetch orders:', err)
  } finally {
    loadingOrders.value = false
  }
}

const submitOrder = async () => {
  if (cartItems.value.length === 0) return
  
  submitting.value = true
  try {
    const orderData = {
      memberId: memberId.value,
      orderItems: cartItems.value.map(item => ({
        productId: item.productId,
        quantity: item.quantity
      }))
    }
    
    const response = await createOrder(orderData)
    
    if (response.success) {
      // Clear cart
      Object.keys(cart).forEach(key => delete cart[key])
      showSuccess.value = true
      await fetchProducts()
      await fetchOrders()
    } else {
      alert(response.message || '訂單建立失敗')
    }
  } catch (err) {
    alert(err.message || '訂單建立失敗，請稍後再試')
  } finally {
    submitting.value = false
  }
}

const handlePayment = async (orderId) => {
  if (!confirm('確定要支付此訂單嗎？')) return
  
  try {
    // 呼叫專用的付款 API 來更新 pay_status
    const response = await payOrder(orderId)
    if (response.success) {
      alert('付款成功！')
      await fetchOrders()
    } else {
      alert(response.message || '付款失敗')
    }
  } catch (err) {
    alert('付款處理出錯，請稍後再試')
  }
}

const handleReceive = async (orderId) => {
  if (!confirm('您確定已收到貨品並要完成訂單嗎？')) return
  
  try {
    // 狀態 3 代表「已取貨」
    const response = await updateOrderStatus(orderId, 3)
    if (response.success) {
      alert('商品已領取！')
      await fetchOrders()
    } else {
      alert(response.message || '狀態更新失敗')
    }
  } catch (err) {
    alert('操作失敗，請稍後再試')
  }
}

const updateStatus = async (orderId, status) => {
  try {
    const response = await updateOrderStatus(orderId, parseInt(status))
    if (response.success) {
      await fetchOrders()
    } else {
      alert(response.message || '狀態更新失敗')
    }
  } catch (err) {
    alert(err.message || '狀態更新失敗')
  }
}

onMounted(() => {
  if (!isAdmin.value) {
    fetchProducts()
  }
  fetchOrders()
})
</script>

<style scoped>
.product-card.out-of-stock {
  opacity: 0.6;
  background: #f7fafc !important;
  filter: grayscale(0.5);
  cursor: not-allowed;
}

.product-card.out-of-stock h3 {
  color: #718096;
}

.product-card.out-of-stock .price {
  color: #a0aec0;
}
</style>
