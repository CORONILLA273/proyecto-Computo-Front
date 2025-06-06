<template>
  <v-container fluid class="pa-4">
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
        Sales Order
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
            @click="showAddSale = true"
          >
            <v-icon left>
              mdi-plus-circle
            </v-icon>
            Add New Sale
          </v-btn>
          <v-btn
            color="#6941C6"
            class="mb-2 rounded-lg"
            style="color: white !important; text-transform: capitalize;"
            @click="exportSales"
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
            :items="paginatedSales"
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

            <template #[`item.ETA`]>
              <span> Jan 4, 2025</span>
            </template>

            <template #[`item.status`]="{ item }">
              <v-chip
                :color="getStatusColor(item.status).background"
                :text-color="getStatusColor(item.status).text"
                small
                class="font-weight-medium"
              >
                ● {{ item.status }}
              </v-chip>
            </template>

            <template #[`item.actions`]="{ item }">
              <!-- Drafted: Eliminar y Editar-->
              <template v-if="item.status === 'Drafted'">
                <v-icon
                  small
                  class="font-weight-bold mr-2"
                  color="#6941C6"
                  @click="showEditSaleDialog(true, item)"
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

              <!-- PDF común a todos excepto Drafted -->
              <template v-else>
                <v-icon small class="mr-2" color="black" @click="generatePDF(item)">
                  mdi-file-document-check-outline
                </v-icon>

                <!-- Pending -->
                <template v-if="item.status === 'Pending'">
                  <v-icon small class="mr-2" color="black" @click="resendSale(item)">
                    mdi-email-sync-outline
                  </v-icon>
                </template>

                <!-- Received -->
                <template v-else-if="item.status === 'Received'">
                  <v-icon small class="mr-2" color="black" @click="viewReceipt(item)">
                    mdi-receipt-text-check-outline
                  </v-icon>
                </template>

                <!-- In Transit -->
                <template v-else-if="item.status === 'In Transit'">
                  <v-tooltip bottom content-class="custom-tooltip">
                    <template #activator="{ on, attrs }">
                      <v-icon small class="mr-2" color="black" v-bind="attrs" v-on="on">
                        mdi-map-marker-radius-outline
                      </v-icon>
                    </template>
                    <span style="color: #6941C6;">
                      Remarks:
                    </span>
                    <br>
                    <span>On Its Way</span>
                  </v-tooltip>
                </template>

                <!-- Rejected -->
                <template v-else-if="item.status === 'Rejected'">
                  <v-tooltip bottom content-class="custom-tooltip">
                    <template #activator="{ on, attrs }">
                      <v-icon small class="mr-2" color="black" v-bind="attrs" v-on="on">
                        mdi-alert-circle-outline
                      </v-icon>
                    </template>
                    <span style="color: red;">
                      Remarks:
                    </span>
                    <br>
                    <span>Products Stock Not Avaliable</span>
                  </v-tooltip>
                </template>
              </template>
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
        <v-card-title>Filter Sale</v-card-title>
        <v-card-text>
          <v-container>
            <v-row>
              <!-- Filtro por status -->
              <label>Status</label>
              <v-col cols="12">
                <v-select
                  v-model="filters.status"
                  :items="statuses"
                  label="Status"
                  outlined
                  clearable
                  class="rounded-lg"
                />
              </v-col>

              <!-- Filtro por Department ID -->
              <label>Department ID</label>
              <v-col cols="12">
                <v-select
                  v-model="filters.departmentID"
                  :items="departmentOptions"
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
                  v-model="filters.warehouse"
                  :items="warehouseOptions"
                  label="Warehouse ID"
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

    <!-- Add Sale Dialog -->
    <v-dialog
      v-model="showAddSale"
      max-width="800px"
    >
      <v-card class="rounded-lg">
        <v-card-title class="d-flex align-center justify-space-between" style="background-color: gainsboro;">
          <span>Add New Sale Order</span>
          <div>
            <v-btn icon @click="showAddSale = false">
              <v-icon>mdi-close-circle-outline</v-icon>
            </v-btn>
          </div>
        </v-card-title>
        <v-card-text>
          <v-container fluid class="pa-0 pl-5 pr-5">
            <v-form ref="addSaleForm" v-model="validForm">
              <v-row>
                <!-- Full Name -->
                <v-col cols="3.5" class="mb-1 mt-2">
                  <label class="d-block font-weight-medium mb-1">Customer ID</label>
                  <v-select
                    v-model="saleData.customerId"
                    :items="customers"
                    item-text="fullName"
                    item-value="id"
                    outlined
                    dense
                  />
                </v-col>
                <v-col cols="3.5" class="mb-1 mt-2">
                  <label class="d-block font-weight-medium mb-1">Quantity</label>
                  <v-text-field
                    v-model="saleData.total"
                    placeholder="Enter Total here"
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
            @click="addSale"
          >
            Add Sale
          </v-btn>
        </v-card-actions>
      </v-card>
    </v-dialog>

    <!-- Edit Catgory Dialog -->
    <v-dialog
      v-model="showEditSale"
      max-width="800px"
    >
      <v-card class="rounded-lg">
        <v-card-title class="d-flex align-center justify-space-between" style="background-color: gainsboro;">
          <span>Edit Sale</span>
          <div>
            <v-btn icon @click="showEditSale = false">
              <v-icon>mdi-close-circle-outline</v-icon>
            </v-btn>
          </div>
        </v-card-title>
        <v-card-text>
          <v-container fluid class="pa-0 pl-5 pr-5">
            <v-form ref="addSaleForm" v-model="validEditForm">
              <v-row>
                <!-- Full Name -->
                <v-col cols="3.5" class="mb-1 mt-2">
                  <label class="d-block font-weight-medium mb-1">Customer ID</label>
                  <v-select
                    v-model="saleData.customerId"
                    :items="customers"
                    item-text="fullName"
                    item-value="id"
                    outlined
                    dense
                  />
                </v-col>
                <v-col cols="3.5" class="mb-1 mt-2">
                  <label class="d-block font-weight-medium mb-1">Quantity</label>
                  <v-text-field
                    v-model="saleData.total"
                    placeholder="Enter Total here"
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
            @click="editSale"
          >
            Update Sale
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
          Are you sure you want to delete "{{ saleToDelete?.id }}"? This action cannot be undone.
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
            @click="deleteSale"
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
import { jsPDF } from 'jspdf'
import autoTable from 'jspdf-autotable'

export default {
  name: 'ProductsPage',
  layout: 'default',
  data () {
    return {
      // Sale
      saleData: {
        orderedDate: this.getTodayDate(),
        customerId: '',
        status: '',
        total: ''
      },
      sales: [],
      customers: [],
      activeCount: 0,
      inactiveCount: 0,
      deletedCount: 0,
      validForm: false,
      validEditForm: false,
      showAddSale: false,
      showEditSale: false,
      showDeleteConfirm: false,
      saleToDelete: null,

      // Filters
      searchQuery: '',
      showFilters: false,
      filters: {
        status: '',
        departmentID: '',
        warehouse: ''
      },
      statuses: ['received', 'Pending', 'Drafted', 'Rejected', 'In Transit'],
      departmentOptions: [],
      warehouseOptions: [],

      // Tables
      itemsPerPage: 10,
      page: 1,
      maxVisiblePages: 7,
      headers: [
        { text: 'Ordered Date', align: 'start', value: 'orderedDate', sortable: true, width: '20%', always: true },
        { text: 'Sale Order ID', value: 'id', width: '10%', breakpoint: 'sm' },
        { text: 'Customer ID', value: 'customerId', width: '10%', breakpoint: 'md' },
        { text: 'ETA', value: 'ETA', width: '10%', breakpoint: 'sm' },
        { text: 'Status', value: 'status', width: '10%', breakpoint: 'sm' },
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
    filteredSales () {
      return this.sales.filter((c) => {
        const matchesStatus = !this.filters.status || c.status === this.filters.status
        const matchesDepartmentID = !this.filters.departmentID || c.departmentID === this.filters.departmentID
        const matchesWarehouse = !this.filters.warehouse || c.warehouse === this.filters.warehouse
        return matchesStatus && matchesDepartmentID && matchesWarehouse
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
      return Math.ceil(this.filteredSales.length / this.itemsPerPage)
    },
    paginatedSales () {
      const start = (this.page - 1) * this.itemsPerPage
      const end = this.page * this.itemsPerPage
      return this.filteredSales.slice(start, end)
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
    this.loadSales()
    window.addEventListener('resize', this.onResize)
    this.windowWidth = window.innerWidth
    this.loadCustomers()
  },
  beforeDestroy () {
    window.removeEventListener('resize', this.onResize)
  },
  methods: {
    // Cambio de color de status
    getStatusColor (status) {
      switch (status) {
        case 'Received':
          return { background: '#E6F4EA', text: '#2E7D32' } // verde suave
        case 'Pending':
          return { background: '#FFF4E5', text: '#EF6C00' } // naranja
        case 'Rejected':
          return { background: '#FDECEA', text: '#D32F2F' } // rojo
        case 'In Transit':
          return { background: '#EFE4FF', text: '#7E57C2' } // morado claro
        default:
          return { background: '#ECEFF1', text: '#455A64' } // gris
      }
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

    async loadCustomers () {
      try {
        const response = await this.$axios.get('/customers')
        console.log('@@@ response => ', response)
        console.log('Datos recibidos => ', response.data)
        this.customers = response.data.customers || response.data || []
      } catch (error) {
        const errorMessage = error.message || 'Error al cargar los usuarios'
        this.$store.dispatch('alert/triggerAlert', {
          message: errorMessage,
          type: 'error'
        })
      }
    },

    onCustomerChange (customerId) {
      const selected = this.customers.find(p => p.id === customerId)
      if (selected) {
        this.form.supplierId = selected.supplierId
      }
    },

    // Sales methods
    async addSale () {
      try {
        // Buscar producto seleccionado
        const selectedCustomer = this.customers.find(p => p.id === this.saleData.customerId)
        if (!selectedCustomer) {
          alert('Select a valid customer')
          return
        }

        // formatear fecha actual
        const formattedDate = new Date().toLocaleDateString('en-US', {
          year: 'numeric',
          month: 'short',
          day: 'numeric'
        })

        // Insertar campos faltantes
        this.saleData.customerId = selectedCustomer.id
        this.saleData.orderedDate = formattedDate
        this.saleData.status = 'Pending'
        console.log(this.saleData)
        await this.$axios.post('/sales/addSale', this.saleData)
        this.resetDataSale()
        this.loadSales()
        this.showAddSale = false
      } catch (error) {
        const errorMessage = error.message || 'Error al crear la Sale'
        console.log('error al Crear Sale: ', errorMessage)
      }
    },
    async loadSales () {
      try {
        const response = await this.$axios.get('/sales')

        const salesList = response.data.sales || response.data || []
        this.sales = salesList.filter(c => c.status !== 'deleted')
        this.departmentOptions = [...new Set(this.sales.map(c => c.departmentID).filter(Boolean))]
        this.warehouseOptions = [...new Set(this.sales.map(c => c.warehouse).filter(Boolean))]

        this.activeCount = salesList.filter(c => c.status === 'active').length
        this.inactiveCount = salesList.filter(c => c.status === 'inactive').length
        this.deletedCount = salesList.filter(c => c.status === 'deleted').length
      } catch (error) {
        const errorMessage = error.message || 'Error al cargar las sales'
        this.$store.dispatch('alert/triggerAlert', {
          message: errorMessage,
          type: 'error'
        })
      }
    },
    async editSale () {
      try {
        await this.$axios.put(`/sales/updateSale/${this.saleData.id}`, this.saleData)
        /* this.$store.dispatch('alert/triggerAlert', {
          message: 'Usuario Actualizado Con Éxito',
          type: 'success'
        }) */
        this.loadSales()
        this.showEditSale = false
      } catch (error) {
        const errorMessage = error.message || 'Error al actualizar el usuario'
        console.log('error al Editar Usuario: ', errorMessage)
        /* this.$store.dispatch('alert/triggerAlert', {
          message: errorMessage,
          type: 'error'
        }) */
      }
    },
    async deleteSale () {
      if (this.saleToDelete) {
        try {
          await this.$axios.delete(`/sales/deleteSale/${this.saleToDelete.id}`)
          this.loadSales()
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
    async toggleStatusSale (item, status) {
      try {
        await this.$axios.patch(`/sales/toggleSaleStatus/${item.id}`, { status })
        this.loadSales()
        // Mensaje opcional de éxito
      } catch (error) {
        const errorMessage = error.message || 'Error al actualizar el usuario'
        console.error('Error al editar usuario:', errorMessage)
      }
    },
    handleToggleStatus (item) {
      const newStatus = item.status === 'active' ? 'inactive' : 'active'
      this.toggleStatusSale(item, newStatus)
      item.status = newStatus // reflejar el cambio en la UI
    },
    exportSales () {
      const dataToExport = this.filteredSales.map(e => ({
        'Ordered Date': e.orderedDate,
        'Sale Order ID': e.id,
        'Customer ID': e.supplierId,
        ETA: 'Jan 4, 2022',
        Status: e.status
      }))

      const worksheet = XLSX.utils.json_to_sheet(dataToExport)
      const workbook = XLSX.utils.book_new()
      XLSX.utils.book_append_sheet(workbook, worksheet, 'Sales')

      XLSX.writeFile(workbook, 'Sales.xlsx')
    },
    getAvatarUrl (item) {
      return `https://i.pravatar.cc/150?u=${item.id}`
    },

    generatePDF (item) {
      // eslint-disable-next-line new-cap
      const doc = new jsPDF()

      doc.setFontSize(16)
      doc.text('SALE DETAILS', 14, 20)

      doc.setFontSize(12)
      doc.text(`ID: ${item.id}`, 14, 30)
      doc.text(`Status: ${item.status}`, 14, 36)
      doc.text(`Date: ${item.orderedDate}`, 14, 42)
      autoTable(doc, {
        startY: 30,
        head: [['Customer', 'status', 'Total']],
        body: [[item.customerId, item.status, `$${item.total}`]]
      })

      doc.save(`sale${item.id}.pdf`)
    },

    viewReceipt (item) {
      // eslint-disable-next-line new-cap
      const doc = new jsPDF()
      doc.setFontSize(18)
      doc.text('SALE RECEIPT', 14, 20)

      doc.setFontSize(12)
      doc.text(`SALE ID: ${item.id}`, 14, 30)
      doc.text(`Status: ${item.status}`, 14, 36)
      doc.text(`Date: ${item.orderedDate}`, 14, 42)

      autoTable(doc, {
        startY: 45,
        head: [['Customer', 'Total']],
        body: [
          [item.customerId, `$${item.total}`]
        ]
      })

      doc.text(`Total to Pay: $${item.total}`, 14, doc.lastAutoTable.finalY + 10)
      doc.save(`receipt_${item.id}.pdf`)
    },

    resendSale () {
      alert('Sale Successfully Forwarded')
    },

    // Filters methods
    resetFilters () {
      this.filters = {
        status: '',
        departmentID: '',
        warehouse: ''
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
    resetDataSale () {
      this.saleData = {
        orderedDate: this.getTodayDate(),
        customerId: '',
        status: '',
        total: ''
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
    showEditSaleDialog (mode = true, sale = null) {
      this.showEditSale = mode
      this.saleData = { ...sale }
    },
    confirmDelete (sale) {
      this.saleToDelete = sale
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
