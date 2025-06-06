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
        Suppliers
      </h1>
    </div>

    <!-- Stats Cards -->
    <template>
      <v-row dense>
        <!-- Active Suppliers -->
        <v-col cols="12" sm="4" class="pr-6">
          <v-card class="pa-5" color="#6941C6" dark rounded>
            <div class="text-subtitle-1">
              Active Suppliers
            </div>
            <div class="display-1 font-weight-bold my-2">
              {{ activeCount }}
            </div>
            <div class="d-flex align-center text-caption">
              <v-icon small class="mr-1">
                mdi-arrow-up
              </v-icon>
              100% <span class="ml-1">vs last month</span>
            </div>
          </v-card>
        </v-col>

        <v-spacer />

        <!-- Inactive Suppliers -->
        <v-col cols="12" sm="4" class="pr-6">
          <v-card class="pa-5" rounded elevation="4">
            <div class="text-subtitle-1">
              Inactive Supliers
            </div>
            <div class="display-1 font-weight-bold my-2">
              {{ inactiveCount }}
            </div>
            <div class="d-flex align-center text-caption">
              <v-icon small class="mr-1">
                mdi-arrow-up
              </v-icon>
              100% <span class="ml-1">vs last month</span>
            </div>
          </v-card>
        </v-col>

        <v-spacer />

        <!-- Deleted Suppliers -->
        <v-col cols="12" sm="4" class="pr-6">
          <v-card class="pa-5" rounded elevartion="4">
            <div class="text-subtitle-1">
              Deleted Suppliers
            </div>
            <div class="display-1 font-weight-bold my-2">
              {{ deletedCount }}
            </div>
            <div class="d-flex align-center text-caption">
              <v-icon small class="mr-1">
                mdi-arrow-up
              </v-icon>
              100% <span class="ml-1">vs last month</span>
            </div>
          </v-card>
        </v-col>
      </v-row>
    </template>

    <div class="d-flex flex-wrap align-center mb-6 mt-6">
      <h1 class="text-h4 mr-4 mb-0">
        <strong>Active Suppliers</strong>
      </h1>
    </div>

    <!-- Search and filters row -->
    <v-card>
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
            @click="showAddSupplier = true"
          >
            <v-icon left>
              mdi-plus-circle
            </v-icon>
            Add New Supplier
          </v-btn>
          <v-btn
            color="#6941C6"
            class="mb-2 rounded-lg"
            style="color: white !important; text-transform: capitalize;"
            @click="exportSuppliers"
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
            :items="suppliers"
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

            <template #[`item.fullName`]="{ item }">
              <div class="d-flex align-center">
                <v-avatar size="32" class="mr-3">
                  <img :src="getAvatarUrl(item)" alt="Avatar">
                </v-avatar>
                <span>{{ item.fullName }}</span>
              </div>
            </template>

            <template #[`item.actions`]="{ item }">
              <v-icon
                small
                class="mr-2"
                color="#6941C6"
                @click="showEditSupplierDialog(true, item)"
              >
                mdi-pencil-outline
              </v-icon>
              <v-icon
                small
                color="error"
                @click="confirmDelete(item)"
              >
                mdi-delete-outline
              </v-icon>
              <v-icon
                small
                class="mr-2"
                :color="item.status === 'active' ? '#6941C6' : 'grey'"
                @click="handleToggleStatus(item)"
              >
                {{ item.status === 'active' ? 'mdi-toggle-switch' : 'mdi-toggle-switch-off-outline' }}
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
    <v-dialog
      v-model="showFilters"
      max-width="500px"
    >
      <v-card class="rounded-lg">
        <v-card-title>Filter Suppliers</v-card-title>
        <v-card-text>
          <v-container>
            <v-row>
              <v-col cols="12">
                <v-text-field
                  v-model="filters.fullName"
                  label="Full Name"
                  outlined
                  dense
                  clearable
                  class="rounded-lg"
                />
              </v-col>
              <v-col cols="12">
                <v-text-field
                  v-model="filters.supplierId"
                  label="Supplier ID"
                  outlined
                  dense
                  clearable
                  class="rounded-lg"
                />
              </v-col>
              <v-col cols="12">
                <v-text-field
                  v-model="filters.contactName"
                  label="Contact Name"
                  outlined
                  dense
                  clearable
                  class="rounded-lg"
                />
              </v-col>
              <v-col cols="12">
                <v-text-field
                  v-model="filters.emailId"
                  label="Email-ID"
                  outlined
                  dense
                  clearable
                  class="rounded-lg"
                />
              </v-col>
              <v-col cols="12">
                <v-text-field
                  v-model="filters.address"
                  label="Address"
                  outlined
                  dense
                  clearable
                  class="rounded-lg"
                />
              </v-col>
              <v-col cols="12">
                <v-text-field
                  v-model="filters.phoneNumber"
                  label="Phone Number"
                  outlined
                  dense
                  clearable
                  class="rounded-lg"
                />
              </v-col>
            </v-row>
          </v-container>
        </v-card-text>
        <v-card-actions>
          <v-spacer />
          <v-btn
            text
            class="rounded-lg"
            @click="resetFilters"
          >
            Reset
          </v-btn>
          <v-btn
            color="primary"
            class="rounded-lg"
            @click="applyFilters"
          >
            Apply Filters
          </v-btn>
        </v-card-actions>
      </v-card>
    </v-dialog>

    <!-- Add Supplier Dialog -->
    <v-dialog
      v-model="showAddSupplier"
      max-width="600px"
    >
      <v-card class="rounded-lg">
        <v-card-title class="d-flex align-center justify-space-between" style="background-color: gainsboro;">
          <span>Add new Supplier</span>
          <div>
            <v-btn icon @click="showAddSupplier = false">
              <v-icon>mdi-close-circle-outline</v-icon>
            </v-btn>
          </div>
        </v-card-title>
        <v-card-text>
          <v-container fluid class="pa-0 pl-5 pr-5">
            <v-form ref="addSupplierForm" v-model="validForm" class="compact-form">
              <v-row>
                <!-- First row -->
                <v-col cols="6" class="mb-1 mt-2">
                  <label>Full Name</label>
                  <v-text-field
                    v-model="supplierData.fullName"
                    placeholder="Enter Full Name"
                    outlined
                    dense
                    class="rounded-lg"
                    required
                  />
                </v-col>
                <v-col cols="6" class="mb-1 mt-2">
                  <label>Contact Name</label>
                  <v-text-field
                    v-model="supplierData.contactName"
                    placeholder="Enter Contact Name"
                    outlined
                    dense
                    class="rounded-lg"
                    required
                  />
                </v-col>

                <!-- Second row -->
                <v-col cols="12" class="mb-1 mt-2">
                  <label>Phone Number</label>
                  <v-text-field
                    v-model="supplierData.phoneNumber"
                    placeholder="1 955 000 000"
                    outlined
                    dense
                    class="rounded-lg"
                    required
                  />
                </v-col>

                <!-- Third row -->
                <v-col cols="12" class="mb-1 mt-2">
                  <label>Email-Id</label>
                  <v-text-field
                    v-model="supplierData.emailId"
                    placeholder="Enter your email-ID"
                    outlined
                    dense
                    class="rounded-lg"
                    required
                  />
                </v-col>

                <!-- Fourth row -->
                <v-col cols="6" class="mb-1 mt-2">
                  <label>State</label>
                  <v-text-field
                    v-model="supplierData.state"
                    placeholder="Enter your state"
                    outlined
                    dense
                    class="rounded-lg"
                    required
                  />
                </v-col>
                <v-col cols="6" class="mb-1 mt-2">
                  <label>Pincode</label>
                  <v-text-field
                    v-model="supplierData.pinCode"
                    placeholder="Enter Pincode"
                    outlined
                    dense
                    class="rounded-lg"
                    required
                  />
                </v-col>

                <!-- Fifth row -->
                <v-col cols="12" class="mb-1 mt-2">
                  <label>Address</label>
                  <v-text-field
                    v-model="supplierData.address"
                    placeholder="Enter your Address"
                    outlined
                    dense
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
            width="500"
            @click="addSupplier"
          >
            Add Supplier
          </v-btn>
        </v-card-actions>
      </v-card>
    </v-dialog>

    <!-- edit Supplier Dialog-->
    <v-dialog
      v-model="showEditSupplier"
      max-width="600px"
    >
      <v-card class="rounded-lg">
        <v-card-title class="d-flex align-center justify-space-between" style="background-color: gainsboro;">
          <span>Edit Supplier</span>
          <div>
            <v-btn icon @click="showEditSupplier = false">
              <v-icon>mdi-close-circle-outline</v-icon>
            </v-btn>
          </div>
        </v-card-title>
        <v-card-text>
          <v-container>
            <v-form ref="editSupplierForm" v-model="validEditForm">
              <v-row>
                <!-- First row -->
                <v-col cols="6" class="mb-1 mt-2">
                  <label>Full Name</label>
                  <v-text-field
                    v-model="supplierData.fullName"
                    placeholder="Enter Full Name"
                    outlined
                    dense
                    class="rounded-lg"
                    required
                  />
                </v-col>
                <v-col cols="6" class="mb-1 mt-2">
                  <label>Contact Name</label>
                  <v-text-field
                    v-model="supplierData.contactName"
                    placeholder="Enter Contact Name"
                    outlined
                    dense
                    class="rounded-lg"
                    required
                  />
                </v-col>

                <!-- Second row -->
                <v-col cols="12" class="mb-1 mt-2">
                  <label>Phone Number</label>
                  <v-text-field
                    v-model="supplierData.phoneNumber"
                    placeholder="1 955 000 000"
                    outlined
                    dense
                    class="rounded-lg"
                    required
                  />
                </v-col>

                <!-- Third row -->
                <v-col cols="12" class="mb-1 mt-2">
                  <label>Email-Id</label>
                  <v-text-field
                    v-model="supplierData.emailId"
                    placeholder="Enter your email-ID"
                    outlined
                    dense
                    class="rounded-lg"
                    required
                  />
                </v-col>

                <!-- Fourth row -->
                <v-col cols="12" md="6">
                  <label>State</label>
                  <v-text-field
                    v-model="supplierData.state"
                    placeholder="Enter your state"
                    outlined
                    dense
                    class="rounded-lg"
                    required
                  />
                </v-col>
                <v-col cols="6" class="mb-1 mt-2">
                  <label>Pincode</label>
                  <v-text-field
                    v-model="supplierData.pinCode"
                    placeholder="Enter Pincode"
                    outlined
                    dense
                    class="rounded-lg"
                    required
                  />
                </v-col>

                <!-- Fifth row -->
                <v-col cols="12" class="mb-1 mt-2">
                  <label>Address</label>
                  <v-text-field
                    v-model="supplierData.address"
                    placeholder="Enter your Address"
                    outlined
                    dense
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
            width="500"
            @click="editSupplier"
          >
            Update Supplier
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
          Are you sure you want to delete "{{ supplierToDelete?.fullName }}"? This action cannot be undone.
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
            @click="deleteSupplier"
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
  name: 'SuppliersPage',
  layout: 'default',
  data () {
    return {
      suppliers: [],
      searchQuery: '',
      activeCount: 0,
      inactiveCount: 0,
      deletedCount: 0,
      itemsPerPage: 10,
      page: 1,
      maxVisiblePages: 7,
      showFilters: false,
      showAddSupplier: false,
      showEditSupplier: false,
      showDeleteConfirm: false,
      supplierToDelete: null,
      validForm: false,
      validEditForm: false,
      windowWidth: 0,
      dimensionUnits: ['inch', 'cm', 'mm', 'ft'],
      filters: {
        fullName: '',
        supplierId: '',
        contactName: '',
        emailId: '',
        address: '',
        phoneNumber: ''
      },
      statuses: ['active', 'inactive', 'deleted'],
      supplierData: {
        fullName: '',
        contactName: '',
        phoneNumber: '',
        emailId: '',
        state: '',
        pinCode: '',
        address: '',
        status: ''
      },
      headers: [
        {
          text: 'Full Name',
          align: 'start',
          value: 'fullName',
          sortable: true,
          width: '20%',
          always: true
        },
        {
          text: 'Supplier ID',
          value: 'id',
          width: '10%',
          breakpoint: 'sm'
        },
        {
          text: 'Contact Name',
          value: 'contactName',
          width: '10%',
          breakpoint: 'md'
        },
        {
          text: 'email-Id',
          value: 'emailId',
          width: '10%',
          breakpoint: 'sm'
        },
        {
          text: 'Address',
          value: 'address',
          width: '8%',
          breakpoint: 'xs'
        },
        {
          text: 'Phone Number',
          value: 'phoneNumber',
          width: '8%',
          breakpoint: 'lg'
        },
        {
          text: 'Actions',
          value: 'actions',
          sortable: false,
          align: 'center',
          width: '80px',
          always: true
        }
      ]
    }
  },
  computed: {
    filteredSuppliers () {
      return this.suppliers.filter((c) => {
        const matchesStatus = !this.filters.status || c.status === this.filters.status
        const matchesDepartmentID = !this.filters.departmentID || c.departmentID === this.filters.departmentID
        const matchesWarehouse = !this.filters.warehouse || c.warehouse === this.filters.warehouse
        return matchesStatus && matchesDepartmentID && matchesWarehouse
      })
    },
    totalPages () {
      if (!this.filteredSuppliers || !Array.isArray(this.filteredSuppliers)) {
        return 0
      }
      return Math.ceil(this.filteredSuppliers.length / this.itemsPerPage)
    },
    paginatedSuppliers () {
      const start = (this.page - 1) * this.itemsPerPage
      const end = this.page * this.itemsPerPage
      return this.filteredSuppliers.slice(start, end)
    },
    dateRangeText () {
      if (!this.dateRange || this.dateRange.length === 0) {
        return 'Select dates'
      }

      if (this.dateRange.length === 1) {
        return this.dateRange[0]
      }

      return `${this.dateRange[0]} - ${this.dateRange[1]}`
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
    this.loadSuppliers()
    window.addEventListener('resize', this.onResize)
    this.windowWidth = window.innerWidth
  },
  beforeDestroy () {
    window.removeEventListener('resize', this.onResize)
  },
  methods: {
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
    async loadSuppliers () {
      try {
        const response = await this.$axios.get('/suppliers')

        const suppliersList = response.data.suppliers || response.data || []
        this.suppliers = suppliersList.filter(c => c.status !== 'deleted')
        this.activeCount = suppliersList.filter(c => c.status === 'active').length
        this.inactiveCount = suppliersList.filter(c => c.status === 'inactive').length
        this.deletedCount = suppliersList.filter(c => c.status === 'deleted').length
      } catch (error) {
        const errorMessage = error.message || 'Error al cargar los usuarios'
        this.$store.dispatch('alert/triggerAlert', {
          message: errorMessage,
          type: 'error'
        })
      }
    },
    triggerFileInput () {
      this.$refs.fileInput.click()
    },
    onFileSelected (event) {
      const file = event.target.files[0]

      console.log('➡️ Archivo recibido:', file)
      console.log('➡️ Es instancia de Blob:', file instanceof Blob)
      console.log('➡️ Es instancia de File:', file instanceof File)

      if (!file || !(file instanceof Blob)) {
        console.error('Archivo inválido o no es tipo Blob:', file)
        return
      }

      const maxSize = 500000
      if (file.size > maxSize) {
        this.$store.dispatch('alert/triggerAlert', {
          message: 'Image cannot be bigger than 500KB.',
          type: 'error'
        })
        this.photoFile = null
        return
      }

      this.compressImage(file)
        .then((base64) => {
          console.log('Imagen comprimida con éxito', base64)
          this.paciente.photo = base64
        })
        .catch((err) => {
          console.error('Error al comprimir imagen:', err.message) // Mostrar mensaje del error
        })
    },
    // Método para comprimir la imagen que devuelve una cadena más corta
    compressImage (file) {
      return new Promise((resolve, reject) => {
        // Verificar si el archivo es un Blob antes de continuar
        if (!(file instanceof Blob)) {
          console.error('Archivo no es tipo Blob:', file)
          reject(new Error('Archivo no es tipo Blob')) // Rechazo con un objeto Error
          return
        }

        const reader = new FileReader()

        reader.onload = (event) => {
          const img = new Image()
          img.src = event.target.result

          img.onload = () => {
            const canvas = document.createElement('canvas')
            const ctx = canvas.getContext('2d')

            const MAX_WIDTH = 800
            const scale = MAX_WIDTH / img.width
            canvas.width = MAX_WIDTH
            canvas.height = img.height * scale

            ctx.drawImage(img, 0, 0, canvas.width, canvas.height)

            canvas.toBlob((blob) => {
              const reader = new FileReader()
              reader.onloadend = () => {
                resolve(reader.result.split(',')[1]) // Base64 result
              }
              reader.readAsDataURL(blob)
            }, 'image/jpeg', 0.7)
          }
        }

        reader.onerror = () => {
          reject(new Error('Error en la lectura de archivo')) // Rechazo con un objeto Error
        }

        reader.readAsDataURL(file)
      })
    },
    goBack () {
      this.$router.back()
    },
    onResize () {
      this.windowWidth = window.innerWidth
    },
    formatNumber (num) {
      return num.toLocaleString()
    },
    customFilter (value, search, item) {
      // Custom filter logic for v-data-table
      if (search.trim() === '') {
        return true
      }

      search = search.toLowerCase()
      return item.fullName.toLowerCase().includes(search) ||
             item.id.toLowerCase().includes(search) ||
             item.category.toLowerCase().includes(search)
    },
    resetFilters () {
      this.filters = {
        fullName: '',
        supplierId: '',
        contactName: '',
        emailId: '',
        address: '',
        phoneNumber: ''
      }
    },
    resetDataSupplier () {
      this.supplierData = {
        fullName: '',
        contactName: '',
        phoneNumber: '',
        emailId: '',
        state: '',
        pinCode: '',
        address: '',
        active: true
      }
    },
    applyFilters () {
      // In a real app, you would apply filters to the data source
      // For now, we'll just close the dialog
      this.showFilters = false
    },
    async addSupplier () {
      try {
        await this.$axios.post('/suppliers/addSupplier', this.supplierData)
        /* this.$store.dispatch('alert/triggerAlert', {
          message: 'Usuario Creado Con Éxito',
          type: 'success'
        }) */
        this.loadSuppliers()
        this.resetDataSupplier()
        this.showAddSupplier = false
      } catch (error) {
        const errorMessage = error.message || 'Error al crear el usuario'
        console.log('error al Crear Usuario: ', errorMessage)

        /* this.$store.dispatch('alert/triggerAlert', {
          message: errorMessage,
          type: 'error'
        }) */
      }
    },
    showEditSupplierDialog (mode = true, supplier = null) {
      this.showEditSupplier = mode
      this.supplierData = { ...supplier }
    },
    async editSupplier () {
      try {
        await this.$axios.put(`/suppliers/updateSupplier/${this.supplierData.id}`, this.supplierData)
        /* this.$store.dispatch('alert/triggerAlert', {
          message: 'Usuario Actualizado Con Éxito',
          type: 'success'
        }) */
        this.loadSuppliers()
        this.resetDataSupplier()
        this.showEditSupplier = false
      } catch (error) {
        const errorMessage = error.message || 'Error al actualizar el usuario'
        console.log('error al Editar Usuario: ', errorMessage)
        /* this.$store.dispatch('alert/triggerAlert', {
          message: errorMessage,
          type: 'error'
        }) */
      }
    },
    confirmDelete (supplier) {
      this.supplierToDelete = supplier
      this.showDeleteConfirm = true
    },
    async deleteSupplier () {
      if (this.supplierToDelete) {
        try {
          await this.$axios.delete(`/suppliers/deleteSupplier/${this.supplierToDelete.id}`)
          /* this.$store.dispatch('alert/triggerAlert', {
            message: 'Usuario Eliminado Con Éxito',
            type: 'success'
          }) */
          this.loadSuppliers()
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
    async toggleStatusSupplier (item, status) {
      try {
        await this.$axios.patch(`/suppliers/toggleSupplierStatus/${item.id}`, { status })
        this.loadSuppliers()
        // Mensaje opcional de éxito
      } catch (error) {
        const errorMessage = error.message || 'Error al actualizar el usuario'
        console.error('Error al editar usuario:', errorMessage)
      }
    },
    handleToggleStatus (item) {
      const newStatus = item.status === 'active' ? 'inactive' : 'active'
      this.toggleStatusSupplier(item, newStatus)
      item.status = newStatus // reflejar el cambio en la UI
    },
    exportSuppliers () {
      const dataToExport = this.filteredSuppliers.map(e => ({
        'Supplier Name': e.fullName,
        'Supplier ID': e.id,
        'Contact Name': e.contactName,
        'Phone Number': e.phoneNumber,
        Email: e.emailId,
        State: e.state,
        Pincode: e.pinCode,
        Address: e.address,
        Status: e.active
      }))

      const worksheet = XLSX.utils.json_to_sheet(dataToExport)
      const workbook = XLSX.utils.book_new()
      XLSX.utils.book_append_sheet(workbook, worksheet, 'Employees')

      XLSX.writeFile(workbook, 'suppliers.xlsx')
    },
    getAvatarUrl (item) {
      return `https://i.pravatar.cc/150?u=${item.id}`
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

.compact-form .v-row {
  margin-top: -8px !important;
  margin-bottom: -8px;
}

.compact-form .v-col {
  padding-top: 4px !important;
  padding-bottom: 4px !important;
}

.compact-form label {
  margin-bottom: 2px;
}

.compact-form .v-text-field {
  margin-top: 2px !important;
  margin-bottom: 2px !important;
}

.compact-form .v-input {
  margin-top: 0 !important;
}
</style>
