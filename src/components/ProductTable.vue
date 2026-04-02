<template>
  <section class="products-section">
    <h2 class="section-title">Productos / Servicios</h2>
    
    <table class="products-table">
      <thead>
        <tr>
          <th class="col-quantity">Cantidad</th>
          <th class="col-description">Descripción</th>
          <th class="col-price">Precio Unit. (USD)</th>
          <th class="col-subtotal">Subtotal</th>
          <th class="col-actions"></th>
        </tr>
      </thead>
      <tbody>
        <tr 
          v-for="(product, index) in products" 
          :key="product.id"
          class="product-row"
        >
          <td class="col-quantity">
            <input 
              type="number" 
              min="1"
              :value="product.quantity"
              @input="updateProduct(index, 'quantity', $event.target.value)"
              placeholder="0"
            />
          </td>
          <td class="col-description">
            <input 
              type="text" 
              :value="product.description"
              @input="updateProduct(index, 'description', $event.target.value)"
              placeholder="Descripción del producto o servicio"
            />
          </td>
          <td class="col-price">
            <input 
              type="number" 
              min="0"
              step="0.01"
              :value="product.price"
              @input="updateProduct(index, 'price', $event.target.value)"
              placeholder="0.00"
            />
          </td>
          <td class="col-subtotal">
            <span class="subtotal-value">
              ${{ formatNumber(calculateSubtotal(product)) }}
            </span>
          </td>
          <td class="col-actions">
            <button 
              v-if="products.length > 1"
              class="btn btn-danger" 
              @click="removeProduct(index)"
              title="Eliminar producto"
            >
              <svg xmlns="http://www.w3.org/2000/svg" width="18" height="18" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
                <path d="M3 6h18"></path>
                <path d="M19 6v14c0 1-1 2-2 2H7c-1 0-2-1-2-2V6"></path>
                <path d="M8 6V4c0-1 1-2 2-2h4c1 0 2 1 2 2v2"></path>
                <line x1="10" y1="11" x2="10" y2="17"></line>
                <line x1="14" y1="11" x2="14" y2="17"></line>
              </svg>
            </button>
          </td>
        </tr>
      </tbody>
    </table>
    
    <button class="btn btn-primary btn-add" @click="addProduct">
      <svg xmlns="http://www.w3.org/2000/svg" width="18" height="18" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
        <line x1="12" y1="5" x2="12" y2="19"></line>
        <line x1="5" y1="12" x2="19" y2="12"></line>
      </svg>
      Agregar Producto
    </button>
  </section>
</template>

<script>
export default {
  name: 'ProductTable',
  props: {
    products: {
      type: Array,
      required: true
    }
  },
  emits: ['update:products'],
  methods: {
    updateProduct(index, field, value) {
      const updatedProducts = [...this.products]
      if (field === 'quantity' || field === 'price') {
        updatedProducts[index][field] = parseFloat(value) || 0
      } else {
        updatedProducts[index][field] = value
      }
      this.$emit('update:products', updatedProducts)
    },
    
    addProduct() {
      const newProduct = {
        id: Date.now(),
        quantity: 1,
        description: '',
        price: 0
      }
      this.$emit('update:products', [...this.products, newProduct])
    },
    
    removeProduct(index) {
      const updatedProducts = this.products.filter((_, i) => i !== index)
      this.$emit('update:products', updatedProducts)
    },
    
    calculateSubtotal(product) {
      return (product.quantity || 0) * (product.price || 0)
    },
    
    formatNumber(value) {
      return value.toFixed(2).replace(/\B(?=(\d{3})+(?!\d))/g, ',')
    }
  }
}
</script>
