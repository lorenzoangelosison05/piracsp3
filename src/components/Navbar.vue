<template>
  <nav class="navbar navbar-expand-lg navbar-dark bg-dark">
    <div class="container-fluid">
      <RouterLink to="/products" class="navbar-brand">E-Commerce</RouterLink>
      
      <button 
        class="navbar-toggler" 
        type="button" 
        data-bs-toggle="collapse" 
        data-bs-target="#navbarNav"
      >
        <span class="navbar-toggler-icon"></span>
      </button>
      
      <div class="collapse navbar-collapse" id="navbarNav">
        <ul class="navbar-nav ms-auto">
          <!-- NOT Logged In -->
          <template v-if="!user.token">
            <li class="nav-item">
              <RouterLink to="/register" class="nav-link">Register</RouterLink>
            </li>
            <li class="nav-item">
              <RouterLink to="/login" class="nav-link">Login</RouterLink>
            </li>
          </template>

          <!-- Logged In -->
          <template v-else>
            <li class="nav-item">
              <RouterLink to="/products" class="nav-link">Products</RouterLink>
            </li>
            
            <!-- Cart link - Only for non-admin users -->
            <li v-if="!user.isAdmin" class="nav-item">
              <RouterLink to="/cart" class="nav-link">Cart</RouterLink>
            </li>
            
            <li class="nav-item">
              <a href="#" @click.prevent="logout" class="nav-link">Logout</a>
            </li>
          </template>
        </ul>
      </div>
    </div>
  </nav>
</template>

<script setup>
import { RouterLink, useRouter } from 'vue-router'
import { useUserStore } from '../stores/user'

const router = useRouter()
const user = useUserStore()

const logout = () => {
  user.clearToken()
  router.push('/login')
}
</script>

<style scoped>
.nav-link {
  cursor: pointer;
}
</style>