<script setup>
import { ref, computed, onMounted } from 'vue'

// ================= Variables =================
const searchQuery = ref('')
const results = ref([])
const loading = ref(false)
const cart = ref([])
const currentPage = ref('dashboard')

const products = ref([])      // DB से सभी products
const errorMsg = ref('')      // error message
const selectedFile = ref(null)
const uploadMessage = ref('')

// ================= Search Function =================
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

// ================= Cart Functions =================
const addToCart = (product) => {
  const existing = cart.value.find(item => item.id === product.id)
  if (existing) {
    if (existing.quantity < product.stock) {
      existing.quantity += 1
    } else {
      alert("⚠️ Stock limit reached!")
    }
  } else {
    if (product.stock > 0) {
      cart.value.push({ ...product, quantity: 1 })
    } else {
      alert("❌ Out of stock!")
    }
  }
}

const increaseQty = (item) => {
  if (item.quantity < item.stock) {
    item.quantity++
  } else {
    alert("⚠️ Stock limit reached!")
  }
}

const decreaseQty = (item) => {
  if (item.quantity > 1) {
    item.quantity--
  } else {
    removeFromCart(item)
  }
}

const removeFromCart = (product) => {
  const index = cart.value.findIndex(item => item.id === product.id)
  if (index !== -1) {
    cart.value.splice(index, 1)
  }
}

const totalPrice = computed(() =>
  cart.value.reduce((sum, item) => sum + (Number(item.price) * item.quantity), 0)
)

// ================= Invoice =================
async function printInvoice() {
  let printContents = document.getElementById("invoice").innerHTML
  let originalContents = document.body.innerHTML

  document.body.innerHTML = printContents
  window.print()
  document.body.innerHTML = originalContents

  try {
    const res = await fetch("http://localhost/myapp/api/update_cart.php", {
      method: "POST",
      headers: { "Content-Type": "application/json" },
      body: JSON.stringify(cart.value)
    })

    const result = await res.json()
    if (result.status === "success") {
      cart.value = []
    } else {
      alert("⚠️ DB update failed: " + result.message)
    }
  } catch (err) {
    console.error("Error updating DB:", err)
    alert("❌ Failed to connect with API")
  }

  location.reload()
}

// ================= Excel Upload =================
const handleFileChange = (event) => {
  selectedFile.value = event.target.files[0]
}

const uploadFile = async () => {
  if (!selectedFile.value) {
    uploadMessage.value = "⚠️ Please select an Excel file"
    return
  }

  const formData = new FormData()
  formData.append('file', selectedFile.value)

  try {
    const res = await fetch("http://localhost/myapp/api/upload_excel.php", {
      method: "POST",
      body: formData
    })
    const data = await res.json()
    uploadMessage.value = data.message
    if (data.status === "success") {
      fetchProducts() 
    }
  } catch (error) {
    uploadMessage.value = "❌ Upload failed!"
  }
}

// ================= fetchProducts =================
// const fetchProducts = async () => {
//   loading.value = true
//   try {
//     const res = await fetch("http://localhost/myapp/api/get_products.php")
//     console.log("Response Status:", res.status)

//     const data = await res.json()
//     console.log("API Response Data:", data) 

//     products.value = data.products || []
//   } catch (error) {
//     console.error("❌ Error:", error)
//     errorMsg.value = "❌ Failed to load products"
//   } finally {
//     loading.value = false
//   }
// }

// onMounted(() => {
//   fetchProducts()
// })

const fetchProducts = async () => {
  loading.value = true
  try {
    const res = await fetch("http://localhost/myapp/api/get_products.php")
    const data = await res.json()

    // Har product me editing flag add karenge
    products.value = (data.products || []).map(p => ({
      ...p,
      editing: false
    }))
  } catch (error) {
    errorMsg.value = "❌ Failed to load products"
  } finally {
    loading.value = false
  }
}

// Save product to DB
const saveProduct = async (product) => {
  console.log("🔄 Sending to API:", product) // 👈 ye add karo
  try {
    const res = await fetch("http://localhost/myapp/api/update_product.php", {
      method: "POST",
      headers: { "Content-Type": "application/json" },
      body: JSON.stringify(product)
    })
    const data = await res.json()
    console.log("✅ API Response:", data) // 👈 ye bhi add karo

    if (data.success) {
      product.editing = false
    } else {
      alert("Update failed ❌ " + data.message)
    }
  } catch (error) {
    console.error("❌ Error updating:", error)
  }
}

onMounted(() => {
  fetchProducts()
})


</script>

<template>
  <div>
    <!-- Menu -->
    <nav style="background:#333; padding:10px; display:flex; gap:15px;">
      <button @click="currentPage='dashboard'" style="color:white; background:none; border:none; cursor:pointer;">🏠 Dashboard</button>
      <button @click="currentPage='products'" style="color:white; background:none; border:none; cursor:pointer;">📦 Upload Products</button>
      <button @click="currentPage='cart'" style="color:white; background:none; border:none; cursor:pointer;">📦 Show ALL Product</button>
      <button @click="currentPage='invoice'" style="color:white; background:none; border:none; cursor:pointer;">📄 Invoice</button>
    </nav>

    <!-- Pages -->
    <div v-if="currentPage==='dashboard'">
      
      <div>
    <h1 style="text-align: center; background: #F54927; padding-block: 10px; margin: 0; color:#e3f2fd">
      Dashboard For Retail POS Software
    </h1>
  </div>

  <div style="font-family: Arial, sans-serif; padding:10px 20px; display: flex; gap: 10px;">

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
            :style="{
              padding: '10px',
              cursor: 'pointer',
              display: 'flex',
              justifyContent: 'space-between',
              alignItems: 'center',
              borderBottom: '1px solid #eee',
              background: item.quantity === 0 ? '#ffcccc' : '#fff'
            }"
          >
            <div>
              <span style="font-weight: bold;">{{ item.name }}</span>
              <span style="margin: 0 8px; color: gray;">|</span>
              <span style="color: #555;">Rs. {{ item.price }}</span>
              <span style="margin: 0 8px; color: gray;">|</span>
              <span :style="{ color: item.quantity === 0 ? 'red' : '#888' }">
                Stock: {{ item.quantity }}
              </span>
            </div>
            <div>
              <button 
                @click="addToCart({ ...item, stock: item.quantity })"
                :disabled="item.quantity === 0"
                style="margin-left: 10px; background: green; color: white; padding: 4px 8px; border: none; border-radius: 5px; cursor: pointer;"
              >
                +
              </button>
            </div>
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
          <div>
            <button @click="decreaseQty(item)" style="margin-right:5px; background:#ccc; border:none; border-radius:4px; padding:2px 6px;">-</button>
            <button @click="increaseQty(item)" style="margin-right:5px; background:#ccc; border:none; border-radius:4px; padding:2px 6px;">+</button>
            <button @click="removeFromCart(item)" style="color: red; font-size: 16px; cursor: pointer; border: none; background: none;">✖</button>
          </div>
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
    </div>

    <div v-if="currentPage==='products'">
  <div>
    <h1
      style="text-align: center; background: #F54927; padding-block: 10px; margin: 0; color:#e3f2fd"
    >
      📦 Upload Products
    </h1>
  </div>

  <!-- Excel Upload Section -->
  <div style="text-align: center; margin-top: 20px;">
  
    <div style="margin:20px; text-align:center;">
      <input type="file" accept=".xlsx,.xls" @change="handleFileChange" />
      <button @click="uploadFile" style="margin-left:10px; padding:8px 15px; background:#388e3c; color:white; border:none; border-radius:5px;">Upload</button>
      <p>{{ uploadMessage }}</p>
    </div>
  </div>
</div>


    <div v-if="currentPage==='cart'">
    <div>
  <h1
    style="text-align: center; background: #F54927; padding-block: 10px; margin: 0; color:#e3f2fd"
  >
    📦 Show All Product
  </h1>
</div>

<!-- Loader -->
<div v-if="loading" style="text-align:center; padding:20px;">⏳ Loading...</div>

<!-- Error Message -->
<div v-if="errorMsg" style="color:red; text-align:center;">{{ errorMsg }}</div>

<!-- Products List -->
<div v-if="products.length" style="display: flex; flex-wrap: wrap; gap: 15px; padding: 20px;">
  <div 
    v-for="p in products" 
    :key="p.id" 
    style="border:1px solid #ccc; padding:15px; border-radius:10px; width:220px; background:#fff"
  >
    <!-- Agar edit mode on hai to input dikhayega -->
    <div v-if="p.editing">
      <input v-model="p.name" placeholder="Name" style="width:100%; padding:5px; margin-bottom:5px;" />
      <input v-model="p.price" type="number" placeholder="Price" style="width:100%; padding:5px; margin-bottom:5px;" />
      <input v-model="p.quantity" type="number" placeholder="Quantity" style="width:100%; padding:5px; margin-bottom:5px;" />

      <button 
        @click="saveProduct(p)" 
        style="background:#28a745; color:#fff; border:none; padding:5px 10px; border-radius:6px; cursor:pointer;"
      >
        💾 Save
      </button>
      <button 
        @click="p.editing = false" 
        style="background:#6c757d; color:#fff; border:none; padding:5px 10px; border-radius:6px; cursor:pointer; margin-left:5px;"
      >
        ❌ Cancel
      </button>
    </div>

    <!-- Normal view -->
    <div v-else>
      <h3 style="margin:0; color:#F54927">{{ p.name }}</h3>
      <p>💰 Price: {{ p.price }}</p>
      <p>📦 Quantity: {{ p.quantity }}</p>

      <button 
        @click="p.editing = true" 
        style="background:#F54927; color:#fff; border:none; padding:5px 10px; border-radius:6px; cursor:pointer;"
      >
        ✏️ Edit
      </button>
    </div>
  </div>
</div>


<!-- No Products -->
<div v-else-if="!loading" style="text-align:center; padding:20px;">
  🚫 No products found
</div>

    </div>

    <div v-if="currentPage==='invoice'">
      <h2>📄 Invoice Page</h2>
      <!-- yahan tum invoice wala code rakho -->
    </div>
  </div>
  
</template>
