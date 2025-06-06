<template>
  <v-container fluid class="pa-4" style="max-width: calc(100vw - 280px); overflow-x: hidden;">
    <!-- Back button -->
    <div class="mb-2">
      <v-btn text class="pl-0" style="color: #6941C6; text-transform: capitalize;" @click="goBack">
        <v-icon color="#6941C6" left>
          mdi-arrow-left
        </v-icon>
        Back
      </v-btn>
    </div>

    <!-- Title and toggle buttons row -->
    <div class="d-flex flex-wrap align-center mb-6">
      <h1 class="text-h4 mr-4 mb-0">
        Stock
      </h1>
      <v-btn-toggle
        v-model="selectedTimeframe"
        mandatory
        class="rounded-toggle ml-auto mb-0 mt-0 custom-toggle"
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

      <!-- Date picker button and menu -->
      <v-menu
        v-model="dateMenu"
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
            {{ dateRangeText }}
          </v-btn>
        </template>
        <v-date-picker
          v-model="dateRange"
          range
          color="#6941C6"
          class="rounded-lg"
          @input="dateMenu = false"
        />
      </v-menu>
    </div>

    <!-- Search and filters row -->
    <v-card class="pa-6 rounded-xs elevation-1">
      <v-row class="mb-6">
        <v-col cols="12" sm="6" md="4" class="d-flex align-center">
          <v-text-field
            v-model="searchQuery"
            label="Search"
            prepend-inner-icon="mdi-magnify"
            outlined
            dense
            hide-details
            class="rounded-lg"
          />
          <v-btn
            class="ml-2 rounded-lg"
            outlined
            style="border: 1px solid gray; text-transform: capitalize;"
            @click="showFilters = true"
          >
            <v-icon left>
              mdi-filter
            </v-icon>
            Filters
          </v-btn>
        </v-col>
        <v-col cols="12" sm="6" md="8" class="d-flex justify-end flex-wrap">
          <v-btn
            class="mr-2 mb-2 rounded-lg"
            style="border: 1px solid gray; text-transform: capitalize;"
            outlined
            @click="showAddStock = true"
          >
            <v-icon left>
              mdi-plus-circle
            </v-icon>
            Add New Stock
          </v-btn>
          <v-btn
            color="#6941C6"
            class="mb-2 rounded-lg"
            style="color: white !important; text-transform: capitalize;"
            @click="exportStocks"
          >
            <v-icon color="white" left>
              mdi-download
            </v-icon>
            Export
          </v-btn>
        </v-col>
      </v-row>

      <v-divider class="my-2" />

      <client-only>
        <div class="table-responsive">
          <v-data-table
            :headers="headers"
            :items="paginatedStocks"
            :items-per-page="itemsPerPage"
            :search="searchQuery"
            :custom-filter="customFilter"
            single-select
            show-select
            hide-default-header
            class="elevation-1 rounded-lg"
            hide-default-footer
            :footer-props="{
              'items-per-page-options': [5, 10, 15, 20],
              'prev-icon': 'mdi-arrow-left',
              'next-icon': 'mdi-arrow-right',
              'show-first-last-page': false
            }"
          >
            <template #header>
              <thead class="custom-header">
                <tr>
                  <th class="select-column" />
                  <th
                    v-for="header in headers"
                    :key="header.value"
                    :class="[header.align ? `text-${header.align}` : '', header.width ? `width-${header.width}` : '']"
                  >
                    {{ header.text }}
                  </th>
                </tr>
              </thead>
            </template>

            <template #[`item.category`]="{ item }">
              {{ getProductProperty(item.productId, 'category') }}
            </template>

            <template #[`item.weight`]="{ item }">
              {{ getProductProperty(item.productId, 'weight') }}
            </template>

            <template #[`item.stockLevel`]="{ item }">
              {{ getProductProperty(item.productId, 'recordedStockLevel') }}
            </template>

            <template #[`item.recLevel`]="{ item }">
              {{ getProductProperty(item.productId, 'warningThresholdStockLevel') }}
            </template>

            <template #[`item.actions`]="{ item }">
              <v-icon
                small
                class="mr-2"
                color="#6941C6"
                @click="showEditStockDialog(true, item)"
              >
                mdi-pencil-outline
              </v-icon>
              <v-icon
                small
                class="mr-2"
                color="error"
                @click="confirmDelete(item)"
              >
                mdi-delete-outline
              </v-icon>
            </template>

            <!-- Personalización del footer de paginación -->
            <template slot="footer.page-text">
              <!-- Dejamos este slot vacío para eliminar el texto por defecto -->
            </template>

            <template #footer>
              <v-divider class="my-2" />
              <div class="custom-pagination d-flex align-center justify-space-between py-3 px-4">
                <v-btn
                  text
                  :disabled="page === 1"
                  class="pagination-btn"
                  outlined
                  @click="page = page - 1"
                >
                  <v-icon small class="mr-1">
                    mdi-arrow-left
                  </v-icon>
                  Previous
                </v-btn>
                <div class="pagination-numbers">
                  <template v-for="n in totalPages">
                    <v-btn
                      v-if="shouldShowPageNumber(n)"
                      :key="n"
                      text
                      small
                      min-width="32"
                      height="32"
                      class="mx-1 pagination-number-btn"
                      :class="{ 'selected-page': n === page }"
                      @click="page = n"
                    >
                      {{ n }}
                    </v-btn>
                    <span v-else-if="n === ellipsisPosition" :key="'ellipsis-' + n" class="mx-1">...</span>
                  </template>
                </div>
                <v-btn
                  text
                  :disabled="page === totalPages"
                  class="pagination-btn"
                  outlined
                  @click="page = page + 1"
                >
                  Next
                  <v-icon small class="ml-1">
                    mdi-arrow-right
                  </v-icon>
                </v-btn>
              </div>
            </template>
          </v-data-table>
        </div>
      </client-only>
    </v-card>

    <!-- Filter Dialog -->
    <v-dialog v-model="showFilters" max-width="500px">
      <v-card class="rounded-lg">
        <v-card-title>Filter Stock</v-card-title>
        <v-card-text>
          <v-container>
            <v-row>
              <!-- Filtro por Department ID -->
              <label>Product ID</label>
              <v-col cols="12">
                <v-select
                  v-model="filters.productId"
                  :items="products"
                  item-text="productName"
                  item-value="id"
                  label="Department ID"
                  outlined
                  clearable
                  class="rounded-lg"
                />
              </v-col>

              <!-- Filtro por Warehouse ID -->
              <label>Warehouse ID</label>
              <v-col cols="12">
                <v-select
                  v-model="filters.warehouseId"
                  :items="warehouses"
                  label="Warehouses"
                  item-text="storeName"
                  item-value="id"
                  outlined
                  clearable
                  class="rounded-lg"
                />
              </v-col>
            </v-row>
          </v-container>
        </v-card-text>

        <v-card-actions>
          <v-spacer />
          <v-btn text class="rounded-lg" @click="resetFilters">
            Reset
          </v-btn>
          <v-btn color="primary" class="rounded-lg" @click="applyFilters">
            Apply Filters
          </v-btn>
        </v-card-actions>
      </v-card>
    </v-dialog>

    <!-- Add Stock Dialog -->
    <v-dialog
      v-model="showAddStock"
      max-width="800px"
    >
      <v-card class="rounded-lg">
        <v-card-title class="d-flex align-center justify-space-between" style="background-color: gainsboro;">
          <span>Add New Stock</span>
          <div>
            <v-btn icon @click="showAddStock = false">
              <v-icon>mdi-close-circle-outline</v-icon>
            </v-btn>
          </div>
        </v-card-title>
        <v-card-text>
          <v-container fluid class="pa-0 pl-5 pr-5">
            <v-form ref="addStockForm" v-model="validForm">
              <v-row>
                <!-- Full Name -->
                <v-col cols="12" class="mb-1 mt-2">
                  <label class="d-block font-weight-medium mb-1">Product ID</label>
                  <v-select
                    v-model="stockData.productId"
                    :items="products"
                    placeholder="Select Product"
                    item-text="productName"
                    item-value="id"
                    outlined
                    dense
                  />
                </v-col>
                <v-col cols="12" class="mb-1 mt-2">
                  <label class="d-block font-weight-medium mb-1">Warehouse ID</label>
                  <v-select
                    v-model="stockData.warehouseId"
                    :items="warehouses"
                    placeholder="Select Warehouse"
                    item-text="storeName"
                    item-value="id"
                    outlined
                    dense
                  />
                </v-col>
                <v-col cols="3.5" class="mb-1 mt-2">
                  <label class="d-block font-weight-medium mb-1">Quantity</label>
                  <v-text-field
                    v-model="stockData.quantity"
                    placeholder="Enter Quantity here"
                    outlined
                    dense
                    hide-details
                    class="rounded-lg"
                    required
                  />
                </v-col>
              </v-row>
            </v-form>
          </v-container>
        </v-card-text>
        <v-card-actions class="justify-center pb-6">
          <v-btn
            color="#6941C6"
            class="rounded-lg white--text px-8"
            width="700"
            @click="addStock"
          >
            Add Stock
          </v-btn>
        </v-card-actions>
      </v-card>
    </v-dialog>

    <!-- Edit Catgory Dialog -->
    <v-dialog
      v-model="showEditStock"
      max-width="800px"
    >
      <v-card class="rounded-lg">
        <v-card-title class="d-flex align-center justify-space-between" style="background-color: gainsboro;">
          <span>Edit Stock</span>
          <div>
            <v-btn icon @click="showEditStock = false">
              <v-icon>mdi-close-circle-outline</v-icon>
            </v-btn>
          </div>
        </v-card-title>
        <v-card-text>
          <v-container fluid class="pa-0 pl-5 pr-5">
            <v-form ref="addStockForm" v-model="validEditForm">
              <v-row>
                <!-- Full Name -->
                <v-col cols="12" class="mb-1 mt-2">
                  <label class="d-block font-weight-medium mb-1">Product ID</label>
                  <v-select
                    v-model="stockData.productId"
                    :items="products"
                    placeholder="Select Product"
                    item-text="productName"
                    item-value="id"
                    outlined
                    dense
                  />
                </v-col>
                <v-col cols="12" class="mb-1 mt-2">
                  <label class="d-block font-weight-medium mb-1">Warehouse ID</label>
                  <v-select
                    v-model="stockData.warehouseId"
                    :items="warehouses"
                    placeholder="Select Warehouse"
                    item-text="storeName"
                    item-value="id"
                    outlined
                    dense
                  />
                </v-col>
                <v-col cols="3.5" class="mb-1 mt-2">
                  <label class="d-block font-weight-medium mb-1">Quantity</label>
                  <v-text-field
                    v-model="stockData.quantity"
                    placeholder="Enter Quantity here"
                    outlined
                    dense
                    hide-details
                    class="rounded-lg"
                    required
                  />
                </v-col>
              </v-row>
            </v-form>
          </v-container>
        </v-card-text>
        <v-card-actions class="justify-center pb-6">
          <v-btn
            color="#6941C6"
            class="rounded-lg white--text px-8"
            width="700"
            @click="editStock"
          >
            Update Stock
          </v-btn>
        </v-card-actions>
      </v-card>
    </v-dialog>

    <!-- Delete Confirmation Dialog -->
    <v-dialog
      v-model="showDeleteConfirm"
      max-width="500px"
    >
      <v-card class="rounded-lg">
        <v-card-title>Confirm Delete</v-card-title>
        <v-card-text>
          Are you sure you want to delete "{{ stockToDelete?.id }}"? This action cannot be undone.
        </v-card-text>
        <v-card-actions>
          <v-spacer />
          <v-btn
            text
            class="rounded-lg"
            @click="showDeleteConfirm = false"
          >
            Cancel
          </v-btn>
          <v-btn
            color="error"
            class="rounded-lg"
            @click="deleteStock"
          >
            Delete
          </v-btn>
        </v-card-actions>
      </v-card>
    </v-dialog>
  </v-container>
</template>

<script>
import * as XLSX from 'xlsx'

export default {
  name: 'ProductsPage',
  layout: 'default',
  data () {
    return {
      // Stocks
      stockData: {
        createdDate: this.getTodayDate(),
        productId: '',
        warehouseId: '',
        quantity: ''
      },
      stocks: [],
      products: [],
      warehouses: [],
      activeCount: 0,
      inactiveCount: 0,
      deletedCount: 0,
      validForm: false,
      validEditForm: false,
      showAddStock: false,
      showEditStock: false,
      showDeleteConfirm: false,
      stockToDelete: null,

      // Filters
      searchQuery: '',
      showFilters: false,
      filters: {
        productId: '',
        warehouseId: ''
      },

      // Tables
      itemsPerPage: 10,
      page: 1,
      maxVisiblePages: 7,
      headers: [
        { text: 'Created Date', align: 'start', value: 'createdDate', sortable: true, width: '20%', always: true },
        { text: 'Stock ID', value: 'id', width: '10%', breakpoint: 'sm' },
        { text: 'Product ID', value: 'productId', width: '10%', breakpoint: 'md' },
        { text: 'Warehouse ID', value: 'warehouseId', width: '10%', breakpoint: 'md' },
        { text: 'Category', value: 'category', width: '10%', breakpoint: 'sm' },
        { text: 'Weight', value: 'weight', width: '8%', breakpoint: 'md' },
        { text: 'Stock Level (in units)', value: 'stockLevel', width: '10%', breakpoint: 'md' },
        { text: 'Rec. Level (in units)', value: 'recLevel', width: '10%', breakpoint: 'md' },
        { text: 'Actions', value: 'actions', sortable: false, align: 'center', width: '80px', always: true }
      ],
      dateMenu: false,
      dateRange: [],
      selectedTimeframe: '1m',
      // Quality of life
      windowWidth: 0
    }
  },
  computed: {
    filteredStocks () {
      return this.stocks.filter((c) => {
        const matchesProduct = !this.filters.productId || c.productId === this.filters.productId
        const matchesWarehouse = !this.filters.warehouseId || c.warehouseId === this.filters.warehouseId
        return matchesProduct && matchesWarehouse
      })
    },
    dateRangeText () {
      if (!this.dateRange || this.dateRange.length === 0) {
        return 'Select dates'
      }

      if (this.dateRange.length === 1) {
        return this.dateRange[0]
      }

      return `${this.dateRange[0]} - ${this.dateRange[1]}`
    },
    totalPages () {
      return Math.ceil(this.filteredStocks.length / this.itemsPerPage)
    },
    paginatedStocks () {
      const start = (this.page - 1) * this.itemsPerPage
      const end = this.page * this.itemsPerPage
      return this.filteredStocks.slice(start, end)
    }
  },
  watch: {
    filters: {
      deep: true,
      handler () {
        this.page = 1
      }
    },
    searchQuery () {
      this.page = 1
    }
  },
  mounted () {
    this.loadStocks()
    window.addEventListener('resize', this.onResize)
    this.windowWidth = window.innerWidth
    this.loadProducts()
    this.loadWarehouses()
  },
  beforeDestroy () {
    window.removeEventListener('resize', this.onResize)
  },
  methods: {

    getProductProperty (productId, property) {
      const product = this.products.find(p => p.id === productId)
      return product ? product[property] : 'N/A'
    },
    // Formato de la fecha para la tabla
    getFormattedDate (date = new Date()) {
      return new Date(date).toLocaleDateString('en-US', {
        year: 'numeric',
        month: 'short',
        day: 'numeric'
      })
    },
    getTodayDate () {
      const today = new Date()
      const yyyy = today.getFullYear()
      const mm = String(today.getMonth() + 1).padStart(2, '0')
      const dd = String(today.getDate()).padStart(2, '0')
      return `${yyyy}-${mm}-${dd}`
    },

    async loadProducts () {
      try {
        const response = await this.$axios.get('/products')
        console.log('@@@ response => ', response)
        console.log('Datos recibidos => ', response.data)
        this.products = response.data.products || response.data || []
      } catch (error) {
        const errorMessage = error.message || 'Error al cargar los usuarios'
        this.$store.dispatch('alert/triggerAlert', {
          message: errorMessage,
          type: 'error'
        })
      }
    },
    async loadWarehouses () {
      try {
        const response = await this.$axios.get('/stores')
        this.warehouses = response.data.stores || response.data || []
        console.log('@@@ response => ', response)
        console.log('Datos recibidos => ', response.data)
      } catch (error) {
        const errorMessage = error.message || 'Error al cargar los usuarios'
        this.$store.dispatch('alert/triggerAlert', {
          message: errorMessage,
          type: 'error'
        })
      }
    },

    onProductChange (productId) {
      const selected = this.products.find(p => p.id === productId)
      if (selected) {
        this.form.supplierId = selected.supplierId
      }
    },

    // Stocks methods
    async addStock () {
      try {
        // Buscar producto seleccionado
        const selectedProduct = this.products.find(p => p.id === this.stockData.productId)
        if (!selectedProduct) {
          alert('Select a valid product')
          return
        }

        // formatear fecha actual
        const formattedDate = new Date().toLocaleDateString('en-US', {
          year: 'numeric',
          month: 'short',
          day: 'numeric'
        })

        // Insertar campos faltantes
        this.stockData.createdDate = formattedDate
        console.log(this.stockData)
        await this.$axios.post('/stocks/addStock', this.stockData)
        this.resetDataStock()
        this.loadStocks()
        this.showAddStock = false
      } catch (error) {
        const errorMessage = error.message || 'Error al crear la Stock'
        console.log('error al Crear Stock: ', errorMessage)
      }
    },
    async loadStocks () {
      try {
        const response = await this.$axios.get('/stocks')

        const stocksList = response.data.stocks || response.data || []
        this.stocks = stocksList.filter(c => c.status !== 'deleted')
        this.departmentOptions = [...new Set(this.stocks.map(c => c.departmentID).filter(Boolean))]
        this.warehouseOptions = [...new Set(this.stocks.map(c => c.warehouse).filter(Boolean))]

        this.activeCount = stocksList.filter(c => c.status === 'active').length
        this.inactiveCount = stocksList.filter(c => c.status === 'inactive').length
        this.deletedCount = stocksList.filter(c => c.status === 'deleted').length
      } catch (error) {
        const errorMessage = error.message || 'Error al cargar las Stocks'
        this.$store.dispatch('alert/triggerAlert', {
          message: errorMessage,
          type: 'error'
        })
      }
    },
    async editStock () {
      try {
        await this.$axios.put(`/stocks/updateStock/${this.stockData.id}`, this.stockData)
        /* this.$store.dispatch('alert/triggerAlert', {
          message: 'Usuario Actualizado Con Éxito',
          type: 'success'
        }) */
        this.loadStocks()
        this.showEditStock = false
      } catch (error) {
        const errorMessage = error.message || 'Error al actualizar el usuario'
        console.log('error al Editar Usuario: ', errorMessage)
        /* this.$store.dispatch('alert/triggerAlert', {
          message: errorMessage,
          type: 'error'
        }) */
      }
    },
    async deleteStock () {
      if (this.stockToDelete) {
        try {
          await this.$axios.delete(`/stocks/deleteStock/${this.stockToDelete.id}`)
          this.loadStocks()
        } catch (error) {
          const errorMessage = error.message || 'Error al eliminar el usuario'
          console.log('error al Editar Usuario: ', errorMessage)

          /* this.$store.dispatch('alert/triggerAlert', {
            message: errorMessage,
            type: 'error'
          }) */
        }
      }
      this.showDeleteConfirm = false
    },
    async toggleStatusStock (item, status) {
      try {
        await this.$axios.patch(`/stocks/toggleStockStatus/${item.id}`, { status })
        this.loadStocks()
        // Mensaje opcional de éxito
      } catch (error) {
        const errorMessage = error.message || 'Error al actualizar el usuario'
        console.error('Error al editar usuario:', errorMessage)
      }
    },
    handleToggleStatus (item) {
      const newStatus = item.status === 'active' ? 'inactive' : 'active'
      this.toggleStatusStock(item, newStatus)
      item.status = newStatus // reflejar el cambio en la UI
    },
    exportStocks () {
      const dataToExport = this.filteredStocks.map(e => ({
        'Created Date': e.createdDate,
        'Stock ID': e.id,
        'Product ID': e.productId,
        'Wirehouse ID': e.wirehouseId,
        Category: e.category
      }))

      const worksheet = XLSX.utils.json_to_sheet(dataToExport)
      const workbook = XLSX.utils.book_new()
      XLSX.utils.book_append_sheet(workbook, worksheet, 'Stocks')

      XLSX.writeFile(workbook, 'stocks.xlsx')
    },
    getAvatarUrl (item) {
      return `https://i.pravatar.cc/150?u=${item.id}`
    },

    // Filters methods
    resetFilters () {
      this.filters = {
        productId: '',
        warehouseId: ''
      }
    },
    applyFilters () {
      this.showFilters = false
    },
    customFilter (value, search, item) {
      if (!search) {
        return true
      }
      const text = search.toLowerCase()
      return (
        item.fullName?.toLowerCase().includes(text) ||
        item.id?.toLowerCase().includes(text) ||
        item.phoneNumber?.toLowerCase().includes(text)
      )
    },
    resetDataStock () {
      this.stockData = {
        createdDate: this.getTodayDate(),
        productId: '',
        warehouseId: '',
        quantity: ''
      }
    },

    // Table methods
    shouldShowPageNumber (n) {
      // Siempre mostrar la primera y última página
      if (n === 1 || n === this.totalPages) {
        return true
      }

      // Si estamos en las primeras páginas
      if (this.page < 5) {
        return n <= 5 || n === this.totalPages - 1
      }

      // Si estamos en las últimas páginas
      if (this.page > this.totalPages - 4) {
        return n >= this.totalPages - 4 || n === 2
      }

      // Si estamos en el medio
      return Math.abs(n - this.page) < 2 || n === 2 || n === this.totalPages - 1
    },

    // Quality of life methods
    goBack () {
      this.$router.back()
    },
    onResize () {
      this.windowWidth = window.innerWidth
    },

    // Dialog methods
    showEditStockDialog (mode = true, stock = null) {
      this.showEditStock = mode
      this.stockData = { ...stock }
    },
    confirmDelete (stock) {
      this.stockToDelete = stock
      this.showDeleteConfirm = true
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
  max-width: calc(100vw - 250px - 50px); /* Resta sidebar + padding */
  overflow-x: auto;
}
.v-data-table {
  min-width: 100%;
  width: auto;
}

/* Estilos para mejorar la visualización en dispositivos móviles */
@media (max-width: 600px) {
  .v-data-table >>> th,
  .v-data-table >>> td {
    white-space: nowrap;
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
.v-container {
  max-width: 100vw;
  padding: 0 16px;
}
</style>
