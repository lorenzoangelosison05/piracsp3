<template>
  <div class="container py-5">
    <div class="card border-0 shadow-sm">
      <div class="card-body p-4">
        <div class="d-flex justify-content-between align-items-center mb-4">
          <h1 class="mb-0">All User Orders</h1>
          <RouterLink to="/products" class="btn btn-dark">
            Back to Dashboard
          </RouterLink>
        </div>

        <div v-if="isLoading" class="text-center my-5 py-5">
          <div class="spinner-border text-primary" role="status"></div>
        </div>

        <div v-else-if="orders.length === 0" class="text-center">
          <h5>No orders yet.</h5>
        </div>

        <div v-else class="table-responsive">
          <table class="table table-hover table-striped align-middle mb-0">
            <thead class="table-dark">
              <tr>
                <th>Order ID</th>
                <th>User</th>
                <th>Products</th>
                <th>Total Price</th>
                <th>Order Date</th>
              </tr>
            </thead>

            <tbody>
              <tr v-for="order in orders" :key="order._id">
                <td>{{ order._id }}</td>

                <td>
                  {{ displayUserEmail(order.userId) }}
                </td>

                <td>
                  <ul class="mb-0">
                    <li v-for="item in order.productsOrdered" :key="item.productId">
                      {{ productName(item.productId) }} (Qty: {{ item.quantity }})
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
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted } from "vue"
import { useRouter, RouterLink } from "vue-router"
import api from "../api"
import { useUserStore } from "../stores/user"

const router = useRouter()
const userStore = useUserStore()

const orders = ref([])
const productsById = ref({})
const usersById = ref({}) // { [userId]: email }
const isLoading = ref(false)

/* products */
const fetchProducts = async () => {
  try {
    const res = await api.get("/products")
    const list = res.data?.products || res.data?.data || res.data || []
    const map = {}
    for (const p of list) {
      const id = p._id || p.id
      const name = p.name || p.title
      if (id && name) map[id] = name
    }
    productsById.value = map
  } catch (e) {
    productsById.value = {}
  }
}

const productName = (id) => productsById.value?.[id] || id

/* orders */
const fetchAllOrders = async () => {
  isLoading.value = true
  try {
    const res = await api.get("/orders")
    orders.value = res.data?.orders || res.data || []
  } catch (e) {
    orders.value = []
  }
  isLoading.value = false
}


const fetchUsersListIfPossible = async () => {
  try {
    const res = await api.get("/users") // might fail in your backend
    const list = res.data?.users || res.data?.data || res.data || []
    const map = {}
    for (const u of list) {
      const id = u._id || u.id
      const email = u.email
      if (id && email) map[id] = email
    }
    usersById.value = { ...usersById.value, ...map }
  } catch (e) {
    // ignore if not supported
  }
}

const tryGetUserEmailById = async (userId) => {
  if (!userId) return
  if (usersById.value[userId]) return

  const endpointsToTry = [
    `/users/${userId}`,
    `/users/details/${userId}`,
    `/users/profile/${userId}`,
  ]

  for (const url of endpointsToTry) {
    try {
      const res = await api.get(url)
      const u = res.data?.user || res.data?.data || res.data
      const email = u?.email
      if (email) {
        usersById.value = { ...usersById.value, [userId]: email }
        return
      }
    } catch (e) {
      // try next endpoint
    }
  }
}

const displayUserEmail = (userId) => {
  const email = usersById.value?.[userId]
  if (email) return email

  // lazy fetch in background
  tryGetUserEmailById(userId)

  // fallback while loading
  return userId
}

/* date */
const formatDate = (date) => {
  return new Date(date).toLocaleDateString("en-US", {
    year: "numeric",
    month: "short",
    day: "numeric",
    hour: "2-digit",
    minute: "2-digit",
  })
}

onMounted(async () => {
  if (!userStore.token) return router.push("/login")
  if (!userStore.isAdmin) return router.push("/products")

  await fetchProducts()
  await fetchUsersListIfPossible()
  await fetchAllOrders()

  
  for (const o of orders.value) {
    if (o?.userId) tryGetUserEmailById(o.userId)
  }
})
</script>