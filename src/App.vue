<template>
  <div id="quotation-app">
    <div class="quotation-container" ref="quotationRef">
      <QuotationHeader 
        :quotation-number="quotationNumber"
        :issue-date="issueDate"
        :expiration-date="expirationDate"
        @update:issue-date="issueDate = $event"
        @update:expiration-date="expirationDate = $event"
      />
      
      <ProductTable 
        :products="products"
        @update:products="products = $event"
      />
      
      <QuotationFooter 
        :subtotal="subtotal"
        :total="total"
        :notes="notes"
        @update:notes="notes = $event"
      />
    </div>
    
    <div class="actions-section">
      <button 
        class="btn btn-success" 
        @click="generatePDF"
        :disabled="isGenerating"
      >
        <svg xmlns="http://www.w3.org/2000/svg" width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
          <path d="M21 15v4a2 2 0 0 1-2 2H5a2 2 0 0 1-2-2v-4"></path>
          <polyline points="7 10 12 15 17 10"></polyline>
          <line x1="12" y1="15" x2="12" y2="3"></line>
        </svg>
        {{ isGenerating ? 'Generando...' : 'Descargar PDF' }}
      </button>
    </div>
  </div>
</template>

<script>
import QuotationHeader from './components/QuotationHeader.vue'
import ProductTable from './components/ProductTable.vue'
import QuotationFooter from './components/QuotationFooter.vue'
import html2pdf from 'html2pdf.js'

export default {
  name: 'App',
  components: {
    QuotationHeader,
    ProductTable,
    QuotationFooter
  },
  data() {
    return {
      quotationNumber: this.generateQuotationNumber(),
      issueDate: this.getCurrentDate(),
      expirationDate: this.getExpirationDate(),
      products: [
        {
          id: 1,
          quantity: 1,
          description: '',
          price: 0
        }
      ],
      notes: 'Precios válidos por 15 días.\nDisponibilidad sujeta a confirmación.\nTiempo de entrega: 3-5 días hábiles.',
      isGenerating: false
    }
  },
  computed: {
    subtotal() {
      return this.products.reduce((sum, product) => {
        return sum + (product.quantity || 0) * (product.price || 0)
      }, 0)
    },
    total() {
      return this.subtotal
    }
  },
  methods: {
    generateQuotationNumber() {
      const year = new Date().getFullYear()
      const random = Math.floor(Math.random() * 9000) + 1000
      return `TRP-${year}-${random}`
    },
    
    getCurrentDate() {
      const today = new Date()
      return today.toISOString().split('T')[0]
    },
    
    getExpirationDate() {
      const date = new Date()
      date.setDate(date.getDate() + 15)
      return date.toISOString().split('T')[0]
    },
    
    async generatePDF() {
      this.isGenerating = true
      
      try {
        const element = this.$refs.quotationRef
        
        const opt = {
          margin: [10, 10, 10, 10],
          filename: `Cotizacion-${this.quotationNumber}.pdf`,
          image: { type: 'jpeg', quality: 0.98 },
          html2canvas: { 
            scale: 2,
            useCORS: true,
            letterRendering: true
          },
          jsPDF: { 
            unit: 'mm', 
            format: 'a4', 
            orientation: 'portrait' 
          }
        }
        
        await html2pdf().set(opt).from(element).save()
      } catch (error) {
        console.error('Error al generar PDF:', error)
        alert('Hubo un error al generar el PDF. Por favor intente de nuevo.')
      } finally {
        this.isGenerating = false
      }
    }
  }
}
</script>

<style scoped>
#quotation-app {
  display: flex;
  flex-direction: column;
  gap: 1rem;
}
</style>
