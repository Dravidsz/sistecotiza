<template>
  <footer class="quotation-footer">
    <div class="footer-content">
      <div class="notes-section">
        <div class="terms-display">
          <label class="terms-label">Términos y Condiciones</label>
          <div class="terms-text">
            <p v-for="(line, i) in termsLines" :key="i">{{ line }}</p>
          </div>
        </div>

      </div>
      
      <div class="totals-section">
        <div class="total-row">
          <span class="total-label">Subtotal:</span>
          <span class="total-value">${{ formatNumber(subtotal) }}</span>
        </div>
        <div class="total-row grand-total">
          <span class="total-label">Total USD:</span>
          <span class="total-value">${{ formatNumber(total) }}</span>
        </div>
      </div>
    </div>
  </footer>
</template>

<script>
export default {
  name: 'QuotationFooter',
  props: {
    subtotal: {
      type: Number,
      required: true
    },
    total: {
      type: Number,
      required: true
    },
    terms: {
      type: String,
      default: ''
    }
  },
  computed: {
    termsLines() {
      return this.terms.split('\n').filter(l => l.trim())
    }
  },
  methods: {
    formatNumber(value) {
      return value.toFixed(2).replace(/\B(?=(\d{3})+(?!\d))/g, ',')
    }
  }
}
</script>
