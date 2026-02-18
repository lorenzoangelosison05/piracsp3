<template>
  <div class="container py-5">
    <div class="d-flex justify-content-between align-items-center mb-4">
      <h1>All User Orders</h1>
      <RouterLink to="/products" class="btn btn-secondary">
        Back to Dashboard
      </RouterLink>
    </div>

    
    <div v-if="isLoading" class="text-center my-5 py-5">
      <div class="spinner-border text-primary" role="status">
        <span class="visually-hidden">Loading...</span>
      </div>
    </div>

    
    <div v-if="!isLoading && orders.length === 0" class="text-center">
      <h5>No orders yet.</h5>
    </div>

   
    <div v-if="!isLoading && orders.length > 0" class="table-responsive">
      <table class="table table-bordered">
        <thead class="table-dark">
          <tr>
            <th>Order ID</th>
            <th>User ID</th>
            <th>Products</th>
            <th>Total Price</th>
            <th>Order Date</th>
          </tr>
        </thead>
        <tbody>
          <tr v-for="order in orders" :key="order._id">
            <td>{{ order._id }}</td>
            <td>{{ order.userId }}</td>
            <td>
              <ul class="mb-0">
                <li v-for="item in order.productsOrdered" :key="item.productId">
                  Product ID: {{ item.productId }} (Qty: {{ item.quantity }})
                </li>
              </ul>
            </td>
            <td>₱{{ order.totalPrice }}</td>
            <td>{{ formatDate(order.orderedOn) }}</td>
          </tr>
        </tbody>
      </table>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted } from 'vue'
import { useRouter, RouterLink } from 'vue-router'
import api from '../api'
import { useUserStore } from '../stores/user'

const router = useRouter()
const user = useUserStore()

const orders = ref([])
const isLoading = ref(false)

const fetchAllOrders = async () => {
  isLoading.value = true
  try {
    const res = await api.get('/orders')
    orders.value = res.data
  } catch (error) {
    console.error('Failed to fetch orders:', error)
  }
  isLoading.value = false
}

const formatDate = (date) => {
  return new Date(date).toLocaleDateString('en-US', {
    year: 'numeric',
    month: 'short',
    day: 'numeric',
    hour: '2-digit',
    minute: '2-digit'
  })
}

onMounted(() => {

  if (!user.token) {
    router.push('/login')
  } else if (!user.isAdmin) {
    router.push('/products')
  } else {
    fetchAllOrders()
  }
})
</script>