<template>
  <div>
    <h1 class="text-center mb-4">Admin Dashboard</h1>
    
    <div class="d-flex gap-2 mb-4">
      <RouterLink to="/add-product" class="btn btn-primary">
        Add New Product
      </RouterLink>
      <button class="btn btn-success">
        Show User Orders
      </button>
    </div>

    <div v-if="isLoading" class="text-center my-5 py-5">
      <div class="spinner-border text-primary" role="status">
        <span class="visually-hidden">Loading...</span>
      </div>
    </div>

    <div v-if="!isLoading" class="table-responsive">
      <table class="table table-bordered">
        <thead class="table-dark">
          <tr>
            <th>Name</th>
            <th>Description</th>
            <th>Price</th>
            <th>Availability</th>
            <th>Actions</th>
          </tr>
        </thead>
        <tbody>
          <tr v-for="product in products" :key="product._id">
            <td>{{ product.name }}</td>
            <td>{{ product.description }}</td>
            <td>₱{{ product.price }}</td>
            <td>
              <span :class="product.isActive ? 'badge bg-success' : 'badge bg-danger'">
                {{ product.isActive ? 'Available' : 'Unavailable' }}
              </span>
            </td>
            <td>
              <div class="d-flex gap-1">
                <RouterLink :to="`/edit-product/${product._id}`" class="btn btn-primary btn-sm">
                  Update
                </RouterLink>
                <button v-if="product.isActive" class="btn btn-danger btn-sm">
                  Disable
                </button>
                <button v-else class="btn btn-success btn-sm">
                  Activate
                </button>
              </div>
            </td>
          </tr>
        </tbody>
      </table>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted } from 'vue'
import { RouterLink } from 'vue-router'
import api from '../api'

const products = ref([])
const isLoading = ref(false)

const fetchAllProducts = async () => {
  isLoading.value = true
  try {
    const res = await api.get('/products')
    products.value = res.data
  } catch (error) {
    console.error('Failed to fetch products:', error)
  }
  isLoading.value = false
}

onMounted(() => {
  fetchAllProducts()
})
</script>