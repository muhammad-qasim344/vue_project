<script setup>
import { ref, computed } from 'vue'

// variables
const searchQuery = ref('')
const results = ref([])
const loading = ref(false)
const cart = ref([])

// search function
const searchData = async () => {
  if (!searchQuery.value) {
    results.value = []
    return
  }
  loading.value = true
  try {
    const res = await fetch("http://localhost/myapp/api/search.php?q=" + encodeURIComponent(searchQuery.value))
    const data = await res.json()
    results.value = data
  } catch (err) {
    console.error("Error:", err)
  }
  loading.value = false
}

// add to cart
const addToCart = (product) => {
  const existing = cart.value.find(item => item.id === product.id)
  if (existing) {
    // same product hai → quantity badhao
    existing.quantity += 1
  } else {
    // naya product hai → cart mein push karo
    cart.value.push({ ...product, quantity: 1 })
  }
}

// remove from cart
const removeFromCart = (product) => {
  const index = cart.value.findIndex(item => item.id === product.id)
  if (index !== -1) {
    cart.value.splice(index, 1)
  }
}

// calculate total
const totalPrice = computed(() =>
  cart.value.reduce((sum, item) => sum + (Number(item.price) * item.quantity), 0)
)
</script>

<template>
  <div class="p-4 flex justify-between items-center" style="font-family: Arial, sans-serif; display: flex; gap:2%">
    <!-- Search Section -->
    <div class="relative max-w-md mx-auto w-33">
      <h2 class="text-xl font-bold mb-3">Search Products</h2>

      <!-- Input -->
      <input
        v-model="searchQuery"
        @input="searchData"
        type="text"
        placeholder="Type to search..."
        class="border p-2 rounded w-full shadow-sm focus:outline-none focus:ring-2 focus:ring-blue-400"
      />

      <!-- Dropdown -->
      <div v-if="searchQuery" class="absolute w-full bg-white border rounded shadow mt-1 z-50">
        <p v-if="loading" class="p-2 text-blue-500">Searching...</p>

        <ul v-else-if="results.length">
          <li
            v-for="(item, i) in results"
            :key="i"
            class="px-3 py-2 hover:bg-blue-50 cursor-pointer flex justify-between items-center"
          >
            <div>
              <span class="font-medium">{{ item.name }}</span>
              <span>----</span>
              <span class="text-gray-600 ml-2">Rs. {{ item.price }}</span><span>----</span>
              <button
              @click="addToCart(item)"
              class="bg-green-500 text-white px-2 py-1 rounded text-sm hover:bg-green-600"
            >
              +
            </button>
            </div>
            
          </li>
        </ul>

        <p v-else class="p-2 text-gray-500">No results found</p>
      </div>
    </div>

    <!-- Cart Section -->
    <div v-if="cart.length" class="mt-6 max-w-md mx-auto border-t pt-4 ">
      <h3 class="text-lg font-semibold mb-2">🛒 Cart</h3>
      <ul>
        <li
          v-for="(item, i) in cart"
          :key="i"
          class="flex justify-between items-center border-b py-2"
        >
          <span>{{ item.name }} (x{{ item.quantity }}) -------- Rs. {{ item.price * item.quantity }}</span>
          <button
            @click="removeFromCart(item)"
            class="text-red-500 hover:text-red-700 text-lg"
          >
            ✖
          </button>
        </li>
      </ul>
      <p class="mt-2 font-bold text-right">Total: Rs. {{ totalPrice }}</p>
    </div>
    <div>
          <!-- Invoice Section -->
    <div v-if="cart.length" class="mt-6 max-w-2xl mx-auto border-t pt-4">
      <h1 class="text-xl font-bold mb-4">📄 Invoice</h1>

      <!-- Sender -->
      <h2 class="font-semibold">From:</h2>
      <address contenteditable class="mb-4">
        <p>Jonathan Neal</p>
        <p>101 E. Chapman Ave<br>Orange, CA 92866</p>
        <p>(800) 555-1234</p>
      </address>

      <!-- Recipient -->
      <h2 class="font-semibold">Bill To:</h2>
      <address contenteditable class="mb-4">
        <p>Some Company<br>c/o Some Guy</p>
      </address>

      <!-- Invoice Table -->
      <!-- Invoice Table -->
<table class="w-full border border-black border-collapse text-left mt-4">
  <thead>
    <tr class="bg-gray-100">
      <th class="border border-black py-2">Product</th>
      <th class="border border-black px-4 py-2">Quantity</th>
      <th class="border border-black px-4 py-2">Price</th>
      <th class="border border-black px-4 py-2">Total</th>
    </tr>
  </thead>
  <tbody>
    <tr v-for="(item, i) in cart" :key="i">
      <td class="border border-black px-4 py-2">{{ item.name }}</td>
      <td class="border border-black px-4 py-2">x{{ item.quantity }}</td>
      <td class="border border-black px-4 py-2">Rs. {{ item.price }}</td>
      <td class="border border-black px-4 py-2">Rs. {{ item.price * item.quantity }}</td>
    </tr>
  </tbody>
  <tfoot>
    <tr>
      <td colspan="3" class="border border-black px-4 py-2 font-bold text-right">Grand Total</td>
      <td class="border border-black px-4 py-2 font-bold">Rs. {{ totalPrice }}</td>
    </tr>
  </tfoot>
</table>

    </div>


    </div>
  </div>
</template>
