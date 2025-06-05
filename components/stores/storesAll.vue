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
        Warehouses
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

    <!-- Info cards row -->
    <template>
      <v-row dense>
        <!-- Active Stores -->
        <v-col cols="12" sm="4" class="pr-6">
          <v-card class="pa-5" color="#6941C6" dark rounded>
            <div class="text-subtitle-1">
              Active Warehouses
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

        <!-- Inactive Stores -->
        <v-col cols="12" sm="4" class="pr-6">
          <v-card class="pa-5" rounded elevartion="4">
            <div class="text-subtitle-1">
              Inactive Warehouses
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

        <!-- Deleted Warehouses -->
        <v-col cols="12" sm="4" class="pr-6">
          <v-card class="pa-5" rounded elevartion="4">
            <div class="text-subtitle-1">
              Deleted Warehouses
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
        <strong>Active Warehouses</strong>
      </h1>
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
            @click="showAddWarehouse = true"
          >
            <v-icon left>
              mdi-plus-circle
            </v-icon>
            Add New Warehouse
          </v-btn>
          <v-btn
            color="#6941C6"
            class="mb-2 rounded-lg"
            style="color: white !important; text-transform: capitalize;"
            @click="exportWarehouses"
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
            :items="paginatedWarehouses"
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
                @click="showEditWarehouseDialog(true, item)"
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
        <v-card-title>Filter Warehouse</v-card-title>
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

    <!-- Add Warehouse Dialog -->
    <v-dialog
      v-model="showAddWarehouse"
      max-width="800px"
    >
      <v-card class="rounded-lg">
        <v-card-title class="d-flex align-center justify-space-between" style="background-color: gainsboro;">
          <span>Add New Warehouse</span>
          <div>
            <v-btn icon @click="showAddWarehouse = false">
              <v-icon>mdi-close-circle-outline</v-icon>
            </v-btn>
          </div>
        </v-card-title>
        <v-card-text>
          <v-container fluid class="pa-0 pl-5 pr-5">
            <v-form ref="addWarehouseForm" v-model="validForm">
              <v-row>
                <!-- Full Name -->
                <v-col cols="6" class="mb-1 mt-2">
                  <label class="d-block font-weight-medium mb-1">Warehouse Name</label>
                  <v-text-field
                    v-model="warehouseData.storeName"
                    placeholder="Enter Full Name"
                    outlined
                    dense
                    hide-details
                    class="rounded-lg"
                    required
                  />
                </v-col>
                <v-col cols="6" class="mb-1 mt-2">
                  <label class="d-block font-weight-medium mb-1">Contact Name</label>
                  <v-text-field
                    v-model="warehouseData.contactName"
                    placeholder="Enter Contact Name"
                    outlined
                    dense
                    hide-details
                    class="rounded-lg"
                    required
                  />
                </v-col>
                <v-col cols="12" class="mb-1 mt-2">
                  <label class="d-block font-weight-medium mb-1">Warehouse Contact Number</label>
                  <v-text-field
                    v-model="warehouseData.number"
                    placeholder="+ 1 955 000 0000 "
                    outlined
                    dense
                    hide-details
                    class="rounded-lg"
                    required
                  />
                </v-col>
                <v-col cols="12" class="mb-1 mt-2">
                  <label class="d-block font-weight-medium mb-1">Email-Id</label>
                  <v-text-field
                    v-model="warehouseData.email"
                    placeholder="Enter your Email-ID"
                    outlined
                    dense
                    hide-details
                    class="rounded-lg"
                    required
                  />
                </v-col>
                <v-col cols="6" class="mb-1 mt-2">
                  <label class="d-block font-weight-medium mb-1">State</label>
                  <v-select
                    v-model="warehouseData.state"
                    :items="states"
                    placeholder="Select your State"
                    outlined
                    dense
                    hide-details
                    class="rounded-lg"
                    required
                  />
                </v-col>
                <v-col cols="6" class="mb-1 mt-2">
                  <label class="d-block font-weight-medium mb-1">Pincode</label>
                  <v-text-field
                    v-model="warehouseData.pinCode"
                    placeholder="Enter Pincode"
                    outlined
                    dense
                    hide-details
                    class="rounded-lg"
                    required
                  />
                </v-col>
                <v-col cols="12" class="mb-1 mt-2">
                  <label class="d-block font-weight-medium mb-1">Address</label>
                  <v-text-field
                    v-model="warehouseData.address"
                    placeholder="Enter your Address"
                    outlined
                    dense
                    hide-details
                    class="rounded-lg"
                    required
                  />
                </v-col>
                <v-col cols="12" class="mb-1 mt-2">
                  <label class="d-block font-weight-medium mb-1">Capacity (in SKUs)</label>
                  <v-text-field
                    v-model="warehouseData.capacity"
                    placeholder="Enter Capacity"
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
            @click="addWarehouse"
          >
            Add Warehouse
          </v-btn>
        </v-card-actions>
      </v-card>
    </v-dialog>

    <!-- Edit Warehouse Dialog -->
    <v-dialog
      v-model="showEditWarehouse"
      max-width="800px"
    >
      <v-card class="rounded-lg">
        <v-card-title class="d-flex align-center justify-space-between" style="background-color: gainsboro;">
          <span>Edit Warehouse</span>
          <div>
            <v-btn small outlined class="mr-2 rounded-lg" style="background-color: white;">
              <v-icon small left>
                mdi-tray-arrow-up
              </v-icon>
              Bulk Upload
            </v-btn>
            <v-btn icon @click="showEditCategroy = false">
              <v-icon>mdi-close-circle-outline</v-icon>
            </v-btn>
          </div>
        </v-card-title>
        <v-card-text>
          <v-container fluid class="pa-0 pl-5 pr-5">
            <v-form ref="addWarehouseForm" v-model="validEditForm">
              <v-row>
                <!-- Full Name -->
                <v-col cols="6" class="mb-1 mt-2">
                  <label class="d-block font-weight-medium mb-1">Warehouse Name</label>
                  <v-text-field
                    v-model="warehouseData.storeName"
                    placeholder="Enter Full Name"
                    outlined
                    dense
                    hide-details
                    class="rounded-lg"
                    required
                  />
                </v-col>
                <v-col cols="6" class="mb-1 mt-2">
                  <label class="d-block font-weight-medium mb-1">Contact Name</label>
                  <v-text-field
                    v-model="warehouseData.contactName"
                    placeholder="Enter Contact Name"
                    outlined
                    dense
                    hide-details
                    class="rounded-lg"
                    required
                  />
                </v-col>
                <v-col cols="12" class="mb-1 mt-2">
                  <label class="d-block font-weight-medium mb-1">Warehouse Contact Number</label>
                  <v-text-field
                    v-model="warehouseData.number"
                    placeholder="+ 1 955 000 0000 "
                    outlined
                    dense
                    hide-details
                    class="rounded-lg"
                    required
                  />
                </v-col>
                <v-col cols="12" class="mb-1 mt-2">
                  <label class="d-block font-weight-medium mb-1">Email-Id</label>
                  <v-text-field
                    v-model="warehouseData.email"
                    placeholder="Enter your Email-ID"
                    outlined
                    dense
                    hide-details
                    class="rounded-lg"
                    required
                  />
                </v-col>
                <v-col cols="6" class="mb-1 mt-2">
                  <label class="d-block font-weight-medium mb-1">State</label>
                  <v-select
                    v-model="warehouseData.state"
                    :items="states"
                    placeholder="Select your State"
                    outlined
                    dense
                    hide-details
                    class="rounded-lg"
                    required
                  />
                </v-col>
                <v-col cols="6" class="mb-1 mt-2">
                  <label class="d-block font-weight-medium mb-1">Pincode</label>
                  <v-text-field
                    v-model="warehouseData.pinCode"
                    placeholder="Enter Pincode"
                    outlined
                    dense
                    hide-details
                    class="rounded-lg"
                    required
                  />
                </v-col>
                <v-col cols="12" class="mb-1 mt-2">
                  <label class="d-block font-weight-medium mb-1">Address</label>
                  <v-text-field
                    v-model="warehouseData.address"
                    placeholder="Enter your Address"
                    outlined
                    dense
                    hide-details
                    class="rounded-lg"
                    required
                  />
                </v-col>
                <v-col cols="12" class="mb-1 mt-2">
                  <label class="d-block font-weight-medium mb-1">Capacity (in SKUs)</label>
                  <v-text-field
                    v-model="warehouseData.capacity"
                    placeholder="Enter Capacity"
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
            @click="editWarehouse"
          >
            Update Warehouse
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
          Are you sure you want to delete "{{ warehouseToDelete?.storeName }}"? This action cannot be undone.
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
            @click="deleteWarehouse"
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
      // Warehouses
      warehouseData: {
        storeName: '',
        location: '',
        capacity: '',
        contactName: '',
        number: '',
        email: '',
        state: '',
        pinCode: '',
        address: '',
        status: ''
      },
      states: [
        'Aguascalientes',
        'Baja California',
        'Baja California Sur',
        'Campeche',
        'Chiapas',
        'Chihuahua',
        'Ciudad de México',
        'Coahuila',
        'Colima',
        'Durango',
        'Estado de México',
        'Guanajuato',
        'Guerrero',
        'Hidalgo',
        'Jalisco',
        'Michoacán',
        'Morelos',
        'Nayarit',
        'Nuevo León',
        'Oaxaca',
        'Puebla',
        'Querétaro',
        'Quintana Roo',
        'San Luis Potosí',
        'Sinaloa',
        'Sonora',
        'Tabasco',
        'Tamaulipas',
        'Tlaxcala',
        'Veracruz',
        'Yucatán',
        'Zacatecas'
      ],
      warehouses: [],
      activeCount: 0,
      inactiveCount: 0,
      deletedCount: 0,
      validForm: false,
      validEditForm: false,
      showAddWarehouse: false,
      showEditWarehouse: false,
      showDeleteConfirm: false,
      warehouseToDelete: null,

      // Filters
      searchQuery: '',
      showFilters: false,
      filters: {
        status: '',
        departmentID: '',
        warehouse: ''
      },
      statuses: ['active', 'inactive', 'deleted'],
      departmentOptions: [],
      warehouseOptions: [],

      // Tables
      itemsPerPage: 10,
      page: 1,
      maxVisiblePages: 7,
      headers: [
        { text: 'Warehouse Name', align: 'start', value: 'storeName', sortable: true, width: '20%', always: true },
        { text: 'Warehouse ID', value: 'id', width: '10%', breakpoint: 'sm' },
        { text: 'Location', value: 'state', width: '10%', breakpoint: 'md' },
        { text: 'Capacity (in SKUs)', value: 'capacity', width: '10%', breakpoint: 'sm' },
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
    filteredWarehouses () {
      return this.warehouses.filter((c) => {
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
      return Math.ceil(this.filteredWarehouses.length / this.itemsPerPage)
    },
    paginatedWarehouses () {
      const start = (this.page - 1) * this.itemsPerPage
      const end = this.page * this.itemsPerPage
      return this.filteredWarehouses.slice(start, end)
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
    this.loadWarehouses()
    window.addEventListener('resize', this.onResize)
    this.windowWidth = window.innerWidth
  },
  beforeDestroy () {
    window.removeEventListener('resize', this.onResize)
  },
  methods: {
    // Warehouses methods
    async addWarehouse () {
      try {
        this.warehouseData.status = 'active'
        await this.$axios.post('/stores/addStore', this.warehouseData)
        this.loadWarehouses()
        this.showAddWarehouse = false
      } catch (error) {
        const errorMessage = error.message || 'Error al crear la Warehouse'
        console.log('error al Crear Warehouse: ', errorMessage)
      }
    },
    async loadWarehouses () {
      try {
        const response = await this.$axios.get('/stores')

        const warehousesList = response.data.stores || response.data || []
        this.warehouses = warehousesList.filter(c => c.status !== 'deleted')
        this.departmentOptions = [...new Set(this.warehouses.map(c => c.departmentID).filter(Boolean))]
        this.warehouseOptions = [...new Set(this.warehouses.map(c => c.warehouse).filter(Boolean))]

        this.activeCount = warehousesList.filter(c => c.status === 'active').length
        this.inactiveCount = warehousesList.filter(c => c.status === 'inactive').length
        this.deletedCount = warehousesList.filter(c => c.status === 'deleted').length
      } catch (error) {
        const errorMessage = error.message || 'Error al cargar las warehouses'
        this.$store.dispatch('alert/triggerAlert', {
          message: errorMessage,
          type: 'error'
        })
      }
    },
    async editWarehouse () {
      try {
        await this.$axios.put(`/stores/updateStore/${this.warehouseData.id}`, this.warehouseData)
        /* this.$store.dispatch('alert/triggerAlert', {
          message: 'Usuario Actualizado Con Éxito',
          type: 'success'
        }) */
        this.loadWarehouses()
        this.showEditWarehouse = false
      } catch (error) {
        const errorMessage = error.message || 'Error al actualizar el usuario'
        console.log('error al Editar Usuario: ', errorMessage)
        /* this.$store.dispatch('alert/triggerAlert', {
          message: errorMessage,
          type: 'error'
        }) */
      }
    },
    async deleteWarehouse () {
      if (this.warehouseToDelete) {
        try {
          await this.$axios.patch(`/stores/toggleStoreStatus/${this.warehouseToDelete.id}`, { status: 'deleted' })
          this.loadWarehouses()
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
    async toggleStatusWarehouse (item, status) {
      try {
        await this.$axios.patch(`/stores/toggleStoreStatus/${item.id}`, { status })
        this.loadWarehouses()
        // Mensaje opcional de éxito
      } catch (error) {
        const errorMessage = error.message || 'Error al actualizar el usuario'
        console.error('Error al editar usuario:', errorMessage)
      }
    },
    handleToggleStatus (item) {
      const newStatus = item.status === 'active' ? 'inactive' : 'active'
      this.toggleStatusWarehouse(item, newStatus)
      item.status = newStatus // reflejar el cambio en la UI
    },
    exportWarehouses () {
      const dataToExport = this.filteredWarehouses.map(e => ({
        'Warehouse Name': e.storeName,
        'Warehouse ID': e.id,
        'Contact Name': e.contactName,
        'Warehouse Contact Number': e.number,
        Email: e.email,
        State: e.state,
        pinCode: e.pinCode,
        Address: e.address,
        Capacity: e.capacity,
        Status: e.status
      }))

      const worksheet = XLSX.utils.json_to_sheet(dataToExport)
      const workbook = XLSX.utils.book_new()
      XLSX.utils.book_append_sheet(workbook, worksheet, 'Warehouses')

      XLSX.writeFile(workbook, 'warehouses.xlsx')
    },
    getAvatarUrl (item) {
      return `https://i.pravatar.cc/150?u=${item.id}`
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
    showEditWarehouseDialog (mode = true, warehouse = null) {
      this.showEditWarehouse = mode
      this.warehouseData = { ...warehouse }
    },
    confirmDelete (warehouse) {
      this.warehouseToDelete = warehouse
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
</style>
