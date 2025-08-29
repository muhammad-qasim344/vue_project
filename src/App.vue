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
    existing.quantity += 1
  } else {
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

function printInvoice() {
  let printContents = document.getElementById("invoice").innerHTML;
  let originalContents = document.body.innerHTML;
  
  document.body.innerHTML = printContents;
  window.print();
  document.body.innerHTML = originalContents;
  location.reload(); // page ko reload kare taki sab normal aa jaye
}
</script>

<template>
  <div><h1 style="text-align: center; background: #F54927; margin: 0; color:#e3f2fd">Dashboard For Retail POS Software</h1></div>
  <div style="font-family: Arial, sans-serif; padding:10px 20px 10px 20px; display: flex; gap: 10px;">
    
    <!-- Search Section -->
    <div style="max-width: 400px; width: 100%; position: relative; background:#f9f9f9; padding:15px; border-radius:8px;">
      <h2 style="font-size: 20px; font-weight: bold; margin-bottom: 12px;">🔍 Search Products</h2>

      <input
        v-model="searchQuery"
        @input="searchData"
        type="text"
        placeholder="Type to search..."
        style="width: 70%; padding: 10px; border: 1px solid #ccc; border-radius: 8px;"
      />

      <div v-if="searchQuery" style="position: absolute; width: 90%; background: #fff; border: 1px solid #ccc; border-radius: 8px; margin-top: 5px; z-index: 50;">
        <p v-if="loading" style="padding: 10px; color: blue; text-align: center;">⏳ Searching...</p>

        <ul v-else-if="results.length" style="list-style: none; margin: 0; padding: 0;">
          <li
            v-for="(item, i) in results"
            :key="i"
            style="padding: 10px; cursor: pointer; display: flex; justify-content: space-between; align-items: center; border-bottom: 1px solid #eee;"
          >
            <div>
              <span style="font-weight: bold;">{{ item.name }}</span>
              <span style="margin: 0 8px; color: gray;">|</span>
              <span style="color: #555;">Rs. {{ item.price }}</span>
            </div>
            <div><button @click="addToCart(item)" style="margin-left: 10px; background: green; color: white; padding: 4px 8px; border: none; border-radius: 5px; cursor: pointer;">+</button></div>
          </li>
        </ul>

        <p v-else style="padding: 10px; color: gray; text-align: center;">❌ No results found</p>
      </div>
    </div>

    <!-- Cart Section -->
    <div v-if="cart.length" style="max-width: 400px; width: 100%; background:#e8f5e9; padding:15px; border-radius:8px;">
      <h3 style="font-size: 18px; font-weight: bold; margin-bottom: 10px;">🛒 Cart</h3>
      <ul style="list-style: none; padding: 0; margin: 0;">
        <li
          v-for="(item, i) in cart"
          :key="i"
          style="display: flex; justify-content: space-between; align-items: center; border-bottom: 1px solid #ddd; padding: 8px 0;"
        >
          <span>{{ item.name }} (x{{ item.quantity }}) -------- Rs. {{ item.price * item.quantity }}</span>
          <button
            @click="removeFromCart(item)"
            style="color: red; font-size: 16px; cursor: pointer; border: none; background: none;"
          >
            ✖
          </button>
        </li>
      </ul>
      <p style="margin-top: 10px; font-weight: bold; text-align: right;">Total: Rs. {{ totalPrice }}</p>
    </div>

    <!-- Invoice Section -->
    <div v-if="cart.length" id="invoice" style="max-width: 600px; width: 100%; background:#e3f2fd; padding:15px; border-radius:8px;">
  <h1 style="font-size: 20px; font-weight: bold; margin-bottom: 15px;">📄 Invoice</h1>

  <h2 style="font-weight: bold;">From:</h2>
  <address contenteditable style="margin-bottom: 10px;">
    <p>Jonathan Neal</p>
    <p>101 E. Chapman Ave<br>Orange, CA 92866</p>
    <p>(800) 555-1234</p>
  </address>

  <h2 style="font-weight: bold;">Bill To:</h2>
  <address contenteditable style="margin-bottom: 10px;">
    <p>Some Company<br>c/o Some Guy</p>
  </address>

  <table style="width: 100%; border: 1px solid black; border-collapse: collapse; margin-top: 15px; text-align: left;">
    <thead>
      <tr style="background: #f3f3f3;">
        <th style="border: 1px solid black; padding: 8px;">Product</th>
        <th style="border: 1px solid black; padding: 8px;">Quantity</th>
        <th style="border: 1px solid black; padding: 8px;">Price</th>
        <th style="border: 1px solid black; padding: 8px;">Total</th>
      </tr>
    </thead>
    <tbody>
      <tr v-for="(item, i) in cart" :key="i">
        <td style="border: 1px solid black; padding: 8px;">{{ item.name }}</td>
        <td style="border: 1px solid black; padding: 8px;">{{ item.quantity }}</td>
        <td style="border: 1px solid black; padding: 8px;">{{ item.price }}</td>
        <td style="border: 1px solid black; padding: 8px;">{{ item.price * item.quantity }}</td>
      </tr>
    </tbody>
    <tfoot>
      <tr>
        <td colspan="3" style="border: 1px solid black; padding: 8px; font-weight: bold; text-align: right;">Grand Total</td>
        <td style="border: 1px solid black; padding: 8px; font-weight: bold;">Rs. {{ totalPrice }}</td>
      </tr>
    </tfoot>
  </table>

  <!-- Print Button -->
  <button @click="printInvoice" style="margin-top: 15px; padding: 10px 20px; background: #1976d2; color: #fff; border: none; border-radius: 6px; cursor: pointer;">
    🖨️ Print Invoice
  </button>
</div>


  </div>
</template>
