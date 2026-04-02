<template>
  <footer class="quotation-footer">
    <div class="footer-content">
      <div class="notes-section">
        <label for="notes">Notas y Términos</label>
        <textarea 
          id="notes"
          :value="notes"
          @input="$emit('update:notes', $event.target.value)"
          placeholder="Escriba aquí las condiciones, términos de pago, garantías u observaciones adicionales..."
        ></textarea>
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
    notes: {
      type: String,
      default: ''
    }
  },
  emits: ['update:notes'],
  methods: {
    formatNumber(value) {
      return value.toFixed(2).replace(/\B(?=(\d{3})+(?!\d))/g, ',')
    }
  }
}
</script>
