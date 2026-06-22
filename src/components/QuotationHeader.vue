<template>
  <header class="quotation-header">
    <div class="header-content">
      <img :src="logoSrc" alt="Top Rare Parts Logo" class="logo-img" />
      <div class="header-info">
        <div class="quotation-number">{{ quotationNumber }}</div>
        <div class="dates-row">
          <div class="date-item">
            <span class="date-label">F. Emisión:</span>
            <input 
              type="date" 
              :value="issueDate" 
              @input="$emit('update:issueDate', $event.target.value)"
            />
          </div>
          <div class="date-item">
            <span class="date-label">Válido hasta:</span>
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
