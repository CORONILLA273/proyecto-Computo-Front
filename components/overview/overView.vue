<template>
  <v-app>
    <v-main>
      <!-- Header -->
      <div>
        <h1 class="text-h4 mr-4 mb-0">
          Purchase Order
        </h1>
      </div>
      <div class="d-flex flex-wrap align-center mb-6">
        <v-btn-toggle
          v-model="selectedTimeframe"
          mandatory
          class="rounded-toggle mb-0 mt-4 custom-toggle"
          background-color="white"
        >
          <v-btn value="1d" class="toggle-btn">
            1d
          </v-btn>
          <v-btn value="7d" class="toggle-btn">
            7d
          </v-btn>
          <v-btn value="1m" class="toggle-btn">
            1m
          </v-btn>
          <v-btn value="3m" class="toggle-btn">
            3m
          </v-btn>
          <v-btn value="6m" class="toggle-btn">
            6m
          </v-btn>
          <v-btn value="1y" class="toggle-btn">
            1y
          </v-btn>
          <v-btn value="3y" class="toggle-btn">
            3y
          </v-btn>
          <v-btn value="5y" class="toggle-btn">
            5y
          </v-btn>
        </v-btn-toggle>
        <v-spacer />
        <!-- Date picker button and menu -->
        <v-menu
          :close-on-content-click="false"
          :nudge-right="40"
          transition="scale-transition"
          min-width="290px"
          offset-y
        >
          <template #activator="{ on, attrs }">
            <v-btn
              outlined
              class="ml-2 rounded-lg date-select-btn"
              style="text-transform: capitalize; border: 1px solid gray;"
              v-bind="attrs"
              v-on="on"
            >
              <v-icon left>
                mdi-calendar
              </v-icon>
              Select dates
            </v-btn>
          </template>
          <v-date-picker
            range
            color="#6941C6"
            class="rounded-lg"
          />
        </v-menu>
      </div>
      <!-- Cards -->
      <v-row>
        <!-- Average Order Volume -->
        <v-col cols="12" sm="4" class="pr-6">
          <v-card class="pa-5" rounded elevation="1">
            <div class="text-subtitle-1">Average Order Volume</div>
            <v-row align="center" justify="space-between">
              <v-col cols="8" class="pa-0">
                <div class="display-1 font-weight-bold my-2 pa-3">${{ averageOrderVolume.toFixed(2) }}</div>
                <div class="d-flex align-center text-caption" :class="{ 'text-success': trendOrderVolumeUp, 'text-error': !trendOrderVolumeUp }">
                  <v-icon small class="mr-1 ml-3 mb-2" :style="{ color: trendOrderVolumeUp ? 'green' : 'red' }">
                    {{ trendOrderVolumeUp ? 'mdi-arrow-up' : 'mdi-arrow-down' }}
                  </v-icon>
                  <span class="mb-2" :style="{ color: trendOrderVolumeUp ? 'green' : 'red' }">{{ percentOrderVolume }}%</span>
                  <span class="ml-1 mb-2">vs last month</span>
                </div>
              </v-col>
              <v-col cols="4" class="pa-3 text-right ">
                <img :src="trendOrderVolumeUp ? '/chart_up.png' : '/chart_down.png'" alt="Trend" style="max-width: 100%; height: auto;">
              </v-col>
            </v-row>
          </v-card>
        </v-col>

        <!-- Transaction Count -->
        <v-col cols="12" sm="4" class="pr-6">
          <v-card class="pa-5" rounded elevation="1">
            <div class="text-subtitle-1">Transaction Count (Orders)</div>
            <v-row align="center" justify="space-between">
              <v-col cols="8" class="pa-0">
                <div class="display-1 font-weight-bold my-2 pa-3">{{ transactionCount }}</div>
                <div class="d-flex align-center text-caption" :class="{ 'text-success': trendTransactionCountUp, 'text-error': !trendTransactionCountUp }">
                  <v-icon small class="mr-1 ml-3 mb-2" :style="{ color: trendTransactionCountUp ? 'green' : 'red' }">
                    {{ trendTransactionCountUp ? 'mdi-arrow-up' : 'mdi-arrow-down' }}
                  </v-icon>
                  <span class="mb-2" :style="{ color: trendTransactionCountUp ? 'green' : 'red' }">{{ percentTransactionCount }}%</span>
                  <span class="ml-1 mb-2">vs last month</span>
                </div>
              </v-col>
              <v-col cols="4" class="pa-3 text-right ">
                <img :src="trendTransactionCountUp ? '/chart_up.png' : '/chart_down.png'" alt="Trend" style="max-width: 100%; height: auto;">
              </v-col>
            </v-row>
          </v-card>
        </v-col>

        <!-- Products Sold -->
        <v-col cols="12" sm="4" class="pr-6">
          <v-card class="pa-5" rounded elevation="1">
            <div class="text-subtitle-1">Product Sold</div>
            <v-row align="center" justify="space-between">
              <v-col cols="8" class="pa-0">
                <div class="display-1 font-weight-bold my-2 pa-3">{{ productsSold }}</div>
                <div class="d-flex align-center text-caption" :class="{ 'text-success': trendProductsSoldUp, 'text-error': !trendProductsSoldUp }">
                  <v-icon small class="mr-1 ml-3 mb-2" :style="{ color: trendProductsSoldUp ? 'green' : 'red' }">
                    {{ trendProductsSoldUp ? 'mdi-arrow-up' : 'mdi-arrow-down' }}
                  </v-icon>
                  <span class="mb-2" :style="{ color: trendProductsSoldUp ? 'green' : 'red' }">{{ percentProductsSold }}%</span>
                  <span class="ml-1 mb-2">vs last month</span>
                </div>
              </v-col>
              <v-col cols="4" class="pa-3 text-right ">
                <img :src="trendProductsSoldUp ? '/chart_up.png' : '/chart_down.png'" alt="Trend" style="max-width: 100%; height: auto;">
              </v-col>
            </v-row>
          </v-card>
        </v-col>
      </v-row>

      <v-row>
        <!-- Gross Profit Margin -->
        <v-col cols="12" sm="4" class="pr-6">
          <v-card class="pa-5" rounded elevation="1">
            <div class="text-subtitle-1">Gross Profit Margin</div>
            <v-row align="center" justify="space-between">
              <v-col cols="8" class="pa-0">
                <div class="display-1 font-weight-bold my-2 pa-3">{{ grossProfitMargin }}%</div>
                <div class="d-flex align-center text-caption" :class="{ 'text-success': trendGrossProfitMarginUp, 'text-error': !trendGrossProfitMarginUp }">
                  <v-icon small class="mr-1 ml-3 mb-2" :style="{ color: trendGrossProfitMarginUp ? 'green' : 'red' }">
                    {{ trendGrossProfitMarginUp ? 'mdi-arrow-up' : 'mdi-arrow-down' }}
                  </v-icon>
                  <span class="mb-2" :style="{ color: trendGrossProfitMarginUp ? 'green' : 'red' }">{{ percentGrossProfitMargin }}%</span>
                  <span class="ml-1 mb-2">vs last month</span>
                </div>
              </v-col>
              <v-col cols="4" class="pa-3 text-right ">
                <img :src="trendGrossProfitMarginUp ? '/chart_up.png' : '/chart_down.png'" alt="Trend" style="max-width: 100%; height: auto;">
              </v-col>
            </v-row>
          </v-card>
        </v-col>

        <!-- Gross Profit -->
        <v-col cols="12" sm="4" class="pr-6">
          <v-card class="pa-5" rounded elevation="1">
            <div class="text-subtitle-1">Gross Profit</div>
            <v-row align="center" justify="space-between">
              <v-col cols="8" class="pa-0">
                <div class="display-1 font-weight-bold my-2 pa-3">${{ grossProfit }}</div>
                <div class="d-flex align-center text-caption" :class="{ 'text-success': trendGrossProfitUp, 'text-error': !trendGrossProfitUp }">
                  <v-icon small class="mr-1 ml-3 mb-2" :style="{ color: trendGrossProfitUp ? 'green' : 'red' }">
                    {{ trendGrossProfitUp ? 'mdi-arrow-up' : 'mdi-arrow-down' }}
                  </v-icon>
                  <span class="mb-2" :style="{ color: trendGrossProfitUp ? 'green' : 'red' }">{{ percentGrossProfit }}%</span>
                  <span class="ml-1 mb-2">vs last month</span>
                </div>
              </v-col>
              <v-col cols="4" class="pa-3 text-right ">
                <img :src="trendGrossProfitUp ? '/chart_up.png' : '/chart_down.png'" alt="Trend" style="max-width: 100%; height: auto;">
              </v-col>
            </v-row>
          </v-card>
        </v-col>

        <!-- Total Net Revenue -->
        <v-col cols="12" sm="4" class="pr-6">
          <v-card class="pa-5" rounded elevation="1">
            <div class="text-subtitle-1">Total Net Revenue</div>
            <v-row align="center" justify="space-between">
              <v-col cols="8" class="pa-0">
                <div class="display-1 font-weight-bold my-2 pa-3">${{ totalNetRevenue }}</div>
                <div class="d-flex align-center text-caption" :class="{ 'text-success': trendTotalRevenueUp, 'text-error': !trendTotalRevenueUp }">
                  <v-icon small class="mr-1 ml-3 mb-2" :style="{ color: trendTotalRevenueUp ? 'green' : 'red' }">
                    {{ trendTotalRevenueUp ? 'mdi-arrow-up' : 'mdi-arrow-down' }}
                  </v-icon>
                  <span class="mb-2" :style="{ color: trendTotalRevenueUp ? 'green' : 'red' }">{{ percentTotalRevenue }}%</span>
                  <span class="ml-1 mb-2">vs last month</span>
                </div>
              </v-col>
              <v-col cols="4" class="pa-3 text-right ">
                <img :src="trendTotalRevenueUp ? '/chart_up.png' : '/chart_down.png'" alt="Trend" style="max-width: 100%; height: auto;">
              </v-col>
            </v-row>
          </v-card>
        </v-col>
      </v-row>
      <!-- Charts -->
       <v-row>
        <!-- Monthly Sales Vs Inventory Analysis -->
        <v-col cols="12" md="12" class="mb-4">
          <v-card class="pa-5 mb-8" rounded elevation="1">
            <div class="text-subtitle-1 font-weight-bold mb-4">Monthly Sales Vs Inventory Analysis</div>
            <canvas id="monthlySalesChart" style="max-height: 300px;"></canvas>
          </v-card>
        </v-col>
       </v-row>
      <v-row>
        <!-- Top Moving Category -->
        <v-col cols="12" md="6" class="mb-4">
          <v-card class="pa-5" rounded elevation="4">
            <div class="d-flex justify-space-between align-center mb-4">
              <div class="text-subtitle-1 font-weight-medium">Top Moving Category</div>
              <v-btn outlined small color="grey" style="text-transform: none;">By Volume</v-btn>
            </div>
            <canvas id="categoryChart" style="max-height: 250px;"></canvas>
          </v-card>
        </v-col>

        <!-- Most Sold Products -->
        <v-col cols="12" md="6" class="mb-4">
          <v-card class="pa-5" rounded elevation="4">
            <div class="d-flex justify-space-between align-center mb-4">
              <div class="text-subtitle-1 font-weight-medium">Most Sold Products</div>
              <v-btn outlined small color="grey" style="text-transform: none;">By Volume</v-btn>
            </div>
            <canvas id="productChart" style="max-height: 250px;"></canvas>
          </v-card>
        </v-col>
      </v-row>
    </v-main>
  </v-app>
</template>

<script>
import Chart from 'chart.js/auto'
import ChartDataLabels from 'chartjs-plugin-datalabels'

Chart.register(ChartDataLabels)

export default {
  data () {
    return {
      purchases: [],
      totalRevenue: 0,
      totalOrders: 0,
      totalProductsSold: 0,
      grossProfit: 0,
      grossProfitMargin: 0,
      netRevenue: 0,

      averageOrderVolume: 0,
      transactionCount: 0,
      productsSold: 0,

      percentOrderVolume: 0,
      percentTransactionCount: 0,
      percentProductsSold: 0,

      trendOrderVolumeUp: true,
      trendTransactionCountUp: true,
      trendProductsSoldUp: true,

      percentGrossProfit: 0,
      trendGrossProfitUp: true,

      totalNetRevenue: 0,
      percentTotalRevenue: 0,
      trendTotalRevenueUp: true,

      percentGrossProfitMargin: 0,
      trendGrossProfitMarginUp: true,

      chartInstanceProduct: null,
      chartInstanceCategory: null,
      monthlyChartInstance: null
    }
  },
  async mounted () {
    try {
      const purchases = await this.fetchPurchases()
      await this.renderProductChart(purchases)
      await this.renderCategoryChart(purchases)
      await this.loadDataCards()
      await this.calculateMonthlyMetrics()
      await this.renderMonthlySalesChart()
    } catch (error) {
      console.error('Error inicializando gráficos:', error)
    }
  },
  methods: {
    async fetchPurchases () {
      const res = await this.$axios.get('/purchases')
      return res.data
    },
    async loadDataCards () {
      try {
        const res = await this.$axios.get('/purchases')
        this.purchases = res.data

        let revenue = 0
        const orders = this.purchases.length
        let totalQty = 0
        let grossProfit = 0
        let netRevenue = 0

        for (const purchase of this.purchases) {
          const quantity = parseInt(purchase.quantity)
          const totalPrice = parseFloat(purchase.totalPrice)
          const productId = purchase.productId

          revenue += totalPrice
          totalQty += quantity

          try {
            const productRes = await this.$axios.get(`/products/${productId}`)
            const product = productRes.data
            const price = parseFloat(product.purchasingPrice)
            const margin = parseFloat(product.sellingPriceMargin)

            // Calcular ganancia y neto
            netRevenue += price * quantity
            grossProfit += price * quantity * (margin / 100)
          } catch (err) {
            console.error(`No se pudo obtener el producto ${productId}`, err)
          }
        }

        this.totalRevenue = revenue
        this.totalOrders = orders
        this.totalProductsSold = totalQty
        this.netRevenue = netRevenue
        this.grossProfit = grossProfit
        this.grossProfitMargin = revenue > 0 ? (grossProfit / revenue) * 100 : 0
      } catch (error) {
        console.error('Error al obtener purchases:', error)
      }
    },
    async calculateMonthlyMetrics () {
      try {
        const res = await this.$axios.get('/purchases')
        const purchases = res.data

        const monthlyTotals = {}

        for (const purchase of purchases) {
          const date = new Date(purchase.orderedDate)
          const key = `${date.getFullYear()}-${String(date.getMonth() + 1).padStart(2, '0')}`

          const total = parseFloat(purchase.totalPrice) || 0
          const quantity = parseInt(purchase.quantity) || 0

          // Obtener datos del producto
          let profitPerUnit = 0
          try {
            const productRes = await this.$axios.get(`/products/${purchase.productId}`)
            const product = productRes.data
            const margin = parseFloat(product.sellingPriceMargin) || 0
            const unitPrice = parseFloat(product.purchasingPrice) || 0
            profitPerUnit = (unitPrice * margin) / 100
          } catch (err) {
            console.warn('Producto no encontrado o incompleto:', purchase.productId)
          }

          if (!monthlyTotals[key]) {
            monthlyTotals[key] = {
              total: 0,
              quantity: 0,
              count: 0,
              profit: 0
            }
          }

          monthlyTotals[key].total += total
          monthlyTotals[key].quantity += quantity
          monthlyTotals[key].count += 1
          monthlyTotals[key].profit += profitPerUnit * quantity
        }

        // Obtener meses actual y anterior
        const today = new Date()
        const currentKey = `${today.getFullYear()}-${String(today.getMonth() + 1).padStart(2, '0')}`
        const lastMonthDate = new Date(today.getFullYear(), today.getMonth() - 1, 1)
        const lastKey = `${lastMonthDate.getFullYear()}-${String(lastMonthDate.getMonth() + 1).padStart(2, '0')}`

        const current = monthlyTotals[currentKey] || { total: 0, quantity: 0, count: 0, profit: 0 }
        const previous = monthlyTotals[lastKey] || { total: 0, quantity: 0, count: 0, profit: 0 }

        // Totales básicos
        this.averageOrderVolume = current.total
        this.transactionCount = current.count
        this.productsSold = current.quantity

        const calcPercent = (now, prev) => {
          if (prev === 0) {
            return now > 0 ? 100 : 0
          }
          return ((now - prev) / prev) * 100
        }

        // Tendencias y porcentajes de cambio
        this.percentOrderVolume = calcPercent(current.total, previous.total).toFixed(1)
        this.percentTransactionCount = calcPercent(current.count, previous.count).toFixed(1)
        this.percentProductsSold = calcPercent(current.quantity, previous.quantity).toFixed(1)

        this.trendOrderVolumeUp = current.total >= previous.total
        this.trendTransactionCountUp = current.count >= previous.count
        this.trendProductsSoldUp = current.quantity >= previous.quantity

        // Totales adicionales
        this.grossProfit = current.profit.toFixed(2)
        this.percentGrossProfit = calcPercent(current.profit, previous.profit).toFixed(1)
        this.trendGrossProfitUp = current.profit >= previous.profit

        this.totalNetRevenue = current.total.toFixed(2)
        this.percentTotalRevenue = calcPercent(current.total, previous.total).toFixed(1)
        this.trendTotalRevenueUp = current.total >= previous.total

        const currentMargin = current.total > 0 ? (current.profit / current.total) : 0
        const previousMargin = previous.total > 0 ? (previous.profit / previous.total) : 0

        this.grossProfitMargin = (currentMargin * 100).toFixed(2)
        this.percentGrossProfitMargin = calcPercent(currentMargin, previousMargin).toFixed(1)
        this.trendGrossProfitMarginUp = currentMargin >= previousMargin
      } catch (error) {
        console.error('Error calculando métricas mensuales:', error)
      }
    },
    async renderMonthlySalesChart () {
      try {
        const res = await this.$axios.get('/purchases')
        const purchases = res.data

        const monthlyData = {} // key: "2025-06", value: { revenue, count, quantity }

        for (const purchase of purchases) {
          const date = new Date(purchase.orderedDate)
          const key = `${date.getFullYear()}-${String(date.getMonth() + 1).padStart(2, '0')}`

          const price = parseFloat(purchase.totalPrice) || 0
          const qty = parseInt(purchase.quantity) || 0

          if (!monthlyData[key]) {
            monthlyData[key] = { revenue: 0, count: 0, quantity: 0 }
          }

          monthlyData[key].revenue += price
          monthlyData[key].count += 1
          monthlyData[key].quantity += qty
        }

        const monthLabels = ['Jan', 'Feb', 'Mar', 'Apr', 'May', 'Jun', 'Jul', 'Aug', 'Sep', 'Oct', 'Nov', 'Dec']
        const revenueData = []
        const countData = []
        const quantityData = []

        const currentYear = new Date().getFullYear()

        for (let i = 0; i < 12; i++) {
          const key = `${currentYear}-${String(i + 1).padStart(2, '0')}`
          const entry = monthlyData[key] || { revenue: 0, count: 0, quantity: 0 }
          revenueData.push(entry.revenue)
          countData.push(entry.count)
          quantityData.push(entry.quantity)
        }

        const ctx = document.getElementById('monthlySalesChart')
        if (this.monthlyChartInstance) {
          this.monthlyChartInstance.destroy()
        }

        this.monthlyChartInstance = new Chart(ctx, {
          type: 'bar',
          data: {
            labels: monthLabels,
            datasets: [
              {
                label: 'Gross Sales Revenue',
                data: revenueData,
                backgroundColor: '#3B82F6',
                stack: 'stack1',
                borderRadius: 10
              },
              {
                label: 'Transaction Count',
                data: countData,
                backgroundColor: '#EF4444',
                stack: 'stack1',
                borderRadius: 10
              },
              {
                label: 'Inventory Moved',
                data: quantityData,
                backgroundColor: '#F59E0B',
                stack: 'stack1',
                borderRadius: 10
              }
            ]
          },
          options: {
            responsive: true,
            maintainAspectRatio: false,
            scales: {
              x: {
                stacked: true,
                grid: { display: false }
              },
              y: {
                stacked: true,
                grid: { display: false },
                ticks: { display: false }
              }
            },
            plugins: {
              tooltip: {
                callbacks: {
                  label: (ctx) => {
                    const label = ctx.dataset.label
                    const value = ctx.parsed.y
                    return label === 'Gross Sales Revenue' ? `$${value}` : `${value} units`
                  }
                }
              },
              legend: {
                position: 'top'
              },
              // Si tienes datalabels, asegúrate de desactivarlos:
              datalabels: {
                display: false
              }
            }
          }
        })
      } catch (err) {
        console.error('Error loading monthly sales chart:', err)
      }
    },
    async renderProductChart (purchases) {
      const totalsByProduct = {}

      for (const purchase of purchases) {
        const id = purchase.productId
        const quantity = parseInt(purchase.quantity)
        if (!totalsByProduct[id]) {
          totalsByProduct[id] = 0
        }
        totalsByProduct[id] += quantity
      }

      const productLabels = []
      const productData = []
      const totalQuantity = Object.values(totalsByProduct).reduce((acc, val) => acc + val, 0)

      for (const productId in totalsByProduct) {
        try {
          const productRes = await this.$axios.get(`/products/${productId}`)
          const productName = productRes.data.productName || 'Unknown'
          productLabels.push(productName)
        } catch {
          productLabels.push('Unknown')
        }
        const percent = Math.round((totalsByProduct[productId] / totalQuantity) * 100)
        productData.push(percent)
      }

      const ctx = document.getElementById('productChart')
      if (this.chartInstanceProduct) {
        this.chartInstanceProduct.destroy()
      }
      this.chartInstanceProduct = new Chart(ctx, {
        type: 'bar',
        data: {
          labels: productLabels,
          datasets: [{
            label: 'Sales %',
            data: productData,
            backgroundColor: '#7F56D9',
            borderRadius: 10,
            barThickness: 12
          }]
        },
        options: {
          indexAxis: 'y',
          maintainAspectRatio: false,
          scales: {
            x: {
              grid: { display: false },
              beginAtZero: true,
              max: 100,
              ticks: { display: false }
            },
            y: {
              grid: { display: false },
              ticks: { color: '#333' }
            }
          },
          plugins: {
            legend: { display: false },
            tooltip: { enabled: false },
            datalabels: {
              anchor: 'end',
              align: 'end',
              color: '#333',
              font: { weight: 'bold' },
              formatter: val => `${val}%`
            }
          }
        }
      })
    },
    async renderCategoryChart (purchases) {
      const categoryTotals = {}

      for (const purchase of purchases) {
        const productId = purchase.productId
        const quantity = parseInt(purchase.quantity)

        try {
          const productRes = await this.$axios.get(`/products/${productId}`)
          const category = productRes.data.category || 'Unknown'
          if (!categoryTotals[category]) {
            categoryTotals[category] = 0
          }
          categoryTotals[category] += quantity
        } catch (err) {
          console.error(`Producto no encontrado: ${productId}`)
        }
      }

      const labels = Object.keys(categoryTotals)
      const data = Object.values(categoryTotals)
      const colors = ['#F04438', '#FDB022', '#12B76A', '#6941C6', '#667085', '#7F56D9']

      const ctx = document.getElementById('categoryChart')
      if (this.chartInstanceCategory) {
        this.chartInstanceCategory.destroy()
      }
      this.chartInstanceCategory = new Chart(ctx, {
        type: 'doughnut',
        data: {
          labels,
          datasets: [{
            data,
            backgroundColor: colors.slice(0, labels.length),
            borderWidth: 0
          }]
        },
        options: {
          cutout: '70%',
          maintainAspectRatio: false,
          plugins: {
            legend: {
              display: true,
              position: 'bottom'
            },
            tooltip: {
              callbacks: {
                label: ctx => `${ctx.label}: ${ctx.parsed} unidades`
              }
            }
          }
        }
      })
    }
  }
}
</script>

<style scoped>
.max-width-300 {
  max-width: 300px;
}

.table-responsive {
  width: 100%;
  overflow-x: auto;
}

/* Estilos para mejorar la visualización en dispositivos móviles */
@media (max-width: 600px) {
  .v-data-table >>> th,
  .v-data-table >>> td {
    padding: 0 8px !important;
  }

  .v-data-table >>> .v-data-table__wrapper {
    overflow-x: auto;
  }
}

/* Estilos para los botones redondeados */
.rounded-lg {
  border-radius: 8px !important;
}

/* Estilos para el toggle de botones */
.rounded-toggle {
  border-radius: 8px;
  overflow: hidden;
  border: 1px solid #E4E7EC;
}

.toggle-btn {
  min-width: 40px !important;
  width: auto !important;
  padding: 0 12px !important;
  background-color: white !important;
  color: #667085 !important;
  text-transform: none !important;
}

/* Estilo para el botón seleccionado */
.custom-toggle .v-btn--active {
  background-color: #F9F5FF !important;
  color: #6941C6 !important;
  font-weight: 500 !important;
}

.date-select-btn {
  border-radius: 8px !important;
  height: 36px;
  border: 1px solid #E4E7EC;
  color: #667085;
}

/* Estilos para el área de carga de imágenes */
.image-upload-container {
  height: 100%;
  display: flex;
  flex-direction: column;
}

.image-upload-area {
  border: 2px dashed #E4E7EC;
  border-radius: 8px;
  height: 150px;
  display: flex;
  align-items: center;
  justify-content: center;
  cursor: pointer;
  overflow: hidden;
}

.upload-placeholder {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  color: #667085;
}

/* Estilos para la paginación personalizada */
.custom-pagination {
  width: 100%;
}

.pagination-btn {
  text-transform: none !important;
  font-size: 14px !important;
  letter-spacing: normal !important;
}

.pagination-number-btn {
  min-width: 32px !important;
  width: 32px !important;
  padding: 0 !important;
  margin: 0 4px !important;
}

.selected-page {
  background-color: #f4f4f4 !important;
  color: #6941C6 !important;
}

/* Ajustes para pantallas pequeñas */
@media (max-width: 960px) {
  .toggle-btn {
    min-width: 36px !important;
    padding: 0 8px !important;
  }
}

/* Estilos para el encabezado personalizado */
.custom-header {
  background-color: #f5f5f5;
  width: 100%;
}

.custom-header th {
  padding: 16px;
  font-weight: 600;
  color: #333;
  text-align: left;
}

.select-column {
  width: 48px;
}
/* Estilo del tooltip*/
.custom-tooltip {
  background-color: white; /* púrpura intenso */
  color: #333;
  font-weight: bold;
}
</style>
