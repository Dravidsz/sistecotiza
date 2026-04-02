<template>
  <header class="quotation-header">
    <div class="header-content">
      <div class="logo-section">
        <img :src="logoSrc" alt="Top Rare Parts Logo" />
      </div>
      <div class="quotation-info">
        <h1 class="quotation-title">Cotización</h1>
        <div class="quotation-number">#{{ quotationNumber }}</div>
        <div class="dates-container">
          <div class="date-field">
            <label>Fecha de emisión:</label>
            <input 
              type="date" 
              :value="issueDate" 
              @input="$emit('update:issueDate', $event.target.value)"
            />
          </div>
          <div class="date-field">
            <label>Válido hasta:</label>
            <input 
              type="date" 
              :value="expirationDate" 
              @input="$emit('update:expirationDate', $event.target.value)"
            />
          </div>
        </div>
      </div>
    </div>
  </header>
</template>

<script>
export default {
  name: 'QuotationHeader',
  props: {
    quotationNumber: {
      type: String,
      required: true
    },
    issueDate: {
      type: String,
      required: true
    },
    expirationDate: {
      type: String,
      required: true
    }
  },
  emits: ['update:issueDate', 'update:expirationDate'],
  data() {
    return {
      logoSrc: '/logo/LOGO.png'
    }
  }
}
</script>
