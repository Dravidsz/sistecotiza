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
        :terms="terms"
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
import { jsPDF } from 'jspdf'

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
      terms: 'Precios válidos por 15 días.\nDisponibilidad sujeta a confirmación.\nTiempo de entrega: 3-5 días hábiles.\nPedidos USA: 10 a 15 días hábiles.\nSe requiere abono del 50% para iniciar el pedido.\nTodas las piezas tienen 1 mes de garantía.\nPiezas eléctricas: solo 24 horas para reclamos después de la entrega.',
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
        const doc = new jsPDF({
          orientation: 'portrait',
          unit: 'mm',
          format: 'a4'
        })
        
        const pageWidth = doc.internal.pageSize.getWidth()
        const pageHeight = doc.internal.pageSize.getHeight()
        const margin = 15
        const contentWidth = pageWidth - (margin * 2)
        let y = margin
        
        doc.setFont('helvetica')
        doc.setTextColor(30, 41, 59)
        
        doc.setFillColor(26, 58, 92)
        doc.rect(0, 0, pageWidth, 45, 'F')
        
        try {
          const logoImg = document.querySelector('.logo-img')
          if (logoImg && logoImg.src) {
            doc.addImage(logoImg.src, 'PNG', margin, 8, 28, 28)
          }
        } catch (e) {
          console.log('Logo no encontrado')
        }
        
        doc.setTextColor(255, 255, 255)
        doc.setFontSize(18)
        doc.setFont('helvetica', 'bold')
        doc.text('COTIZACIÓN', pageWidth - margin, 18, { align: 'right' })
        
        doc.setFontSize(12)
        doc.setFont('helvetica', 'normal')
        doc.text(`#${this.quotationNumber}`, pageWidth - margin, 28, { align: 'right' })
        
        doc.setFontSize(9)
        doc.text(`F. Emisión: ${this.formatDate(this.issueDate)}`, pageWidth - margin, 36, { align: 'right' })
        doc.text(`Válido hasta: ${this.formatDate(this.expirationDate)}`, pageWidth - margin, 42, { align: 'right' })
        
        y = 55
        
        doc.setTextColor(30, 41, 59)
        doc.setFontSize(12)
        doc.setFont('helvetica', 'bold')
        doc.text('PRODUCTOS / SERVICIOS', margin, y)
        
        y += 8
        
        doc.setFillColor(26, 58, 92)
        doc.rect(margin, y, contentWidth, 8, 'F')
        
        doc.setTextColor(255, 255, 255)
        doc.setFontSize(8)
        doc.setFont('helvetica', 'bold')
        doc.text('CANT.', margin + 3, y + 5.5)
        doc.text('DESCRIPCIÓN', margin + 25, y + 5.5)
        doc.text('PRECIO', pageWidth - margin - 38, y + 5.5)
        doc.text('SUBTOTAL', pageWidth - margin - 3, y + 5.5, { align: 'right' })
        
        y += 8
        
        doc.setTextColor(30, 41, 59)
        doc.setFont('helvetica', 'normal')
        
        const productsWithData = this.products.filter(p => p.description || p.price > 0)
        
        if (productsWithData.length === 0) {
          doc.setFontSize(10)
          doc.setTextColor(100, 100, 100)
          doc.text('Sin productos agregados', margin, y + 10)
          y += 20
        } else {
          productsWithData.forEach((product, index) => {
            const subtotal = (product.quantity || 0) * (product.price || 0)
            const descLines = doc.splitTextToSize((product.description || '-').toUpperCase(), 100)
            const rowHeight = Math.max(10, descLines.length * 5 + 6)
            
            if (y + rowHeight > pageHeight - 60) {
              doc.addPage()
              y = margin
            }
            
            doc.setFillColor(245, 245, 245)
            if (index % 2 === 0) {
              doc.rect(margin, y, contentWidth, rowHeight, 'F')
            }
            
            doc.setFontSize(9)
            doc.setTextColor(30, 41, 59)
            doc.text(String(product.quantity || 0), margin + 3, y + 5)
            
            doc.setFontSize(9)
            let descY = y + 5
            descLines.forEach(line => {
              doc.text(line, margin + 25, descY)
              descY += 5
            })
            
            doc.text(`$${this.formatNumber(product.price || 0)}`, pageWidth - margin - 38, y + 5)
            doc.text(`$${this.formatNumber(subtotal)}`, pageWidth - margin - 3, y + 5, { align: 'right' })
            
            y += rowHeight
          })
        }
        
        y += 5
        
        doc.setDrawColor(200, 200, 200)
        doc.line(margin, y, pageWidth - margin, y)
        y += 5
        
        doc.setFillColor(245, 245, 245)
        doc.rect(pageWidth - margin - 60, y, 60, 20, 'F')
        
        doc.setFontSize(10)
        doc.setTextColor(100, 100, 100)
        doc.text('Subtotal:', pageWidth - margin - 55, y + 7)
        doc.setTextColor(30, 41, 59)
        doc.text(`$${this.formatNumber(this.subtotal)}`, pageWidth - margin - 3, y + 7, { align: 'right' })
        
        doc.setFontSize(12)
        doc.setFont('helvetica', 'bold')
        doc.text('Total USD:', pageWidth - margin - 55, y + 15)
        doc.text(`$${this.formatNumber(this.total)}`, pageWidth - margin - 3, y + 15, { align: 'right' })
        
        y += 28
        
        doc.setFontSize(10)
        doc.setFont('helvetica', 'bold')
        doc.setTextColor(30, 41, 59)
        doc.text('TÉRMINOS Y CONDICIONES', margin, y)
        y += 6
        
        doc.setFont('helvetica', 'normal')
        doc.setFontSize(8)
        doc.setTextColor(80, 80, 80)
        
        const termsLines = this.terms.split('\n').filter(l => l.trim())
        termsLines.forEach(line => {
          if (y > pageHeight - 20) {
            doc.addPage()
            y = margin
          }
          doc.text('• ' + line, margin, y)
          y += 4
        })
        
        doc.save(`Cotizacion-${this.quotationNumber}.pdf`)
        
      } catch (error) {
        console.error('Error al generar PDF:', error)
        alert('Hubo un error al generar el PDF.')
      } finally {
        this.isGenerating = false
      }
    },
    
    formatDate(dateStr) {
      if (!dateStr) return ''
      const date = new Date(dateStr + 'T00:00:00')
      return date.toLocaleDateString('es-ES', { year: 'numeric', month: 'short', day: 'numeric' })
    },
    
    formatNumber(value) {
      return value.toFixed(2).replace(/\B(?=(\d{3})+(?!\d))/g, ',')
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
