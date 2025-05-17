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
        Departments
      </h1>
    </div>

    <!-- Info cards row -->
    <template>
      <v-row dense>
        <!-- Active Departments -->
        <v-col cols="12" sm="4" class="pr-6">
          <v-card class="pa-5" color="#6941C6" dark rounded>
            <div class="text-subtitle-1">Active Departments</div>
            <div class="display-1 font-weight-bold my-2">{{ activeCount }}</div>
            <div class="d-flex align-center text-caption">
              <v-icon small class="mr-1">mdi-arrow-up</v-icon>
              100% <span class="ml-1">vs last month</span>
            </div>
          </v-card>
        </v-col>

        <v-spacer />

        <!-- Inactive Departments -->
        <v-col cols="12" sm="4" class="pr-6">
          <v-card class="pa-5" rounded elevartion="4">
            <div class="text-subtitle-1">Inactive Departments</div>
            <div class="display-1 font-weight-bold my-2">{{ inactiveCount }}</div>
            <div class="d-flex align-center text-caption">
              <v-icon small class="mr-1">mdi-arrow-up</v-icon>
              100% <span class="ml-1">vs last month</span>
            </div>
          </v-card>
        </v-col>

        <v-spacer />

        <!-- Deleted Departments -->
        <v-col cols="12" sm="4" class="pr-6">
          <v-card class="pa-5" rounded elevartion="4">
            <div class="text-subtitle-1">Deleted Departments</div>
            <div class="display-1 font-weight-bold my-2">{{ deletedCount }}</div>
            <div class="d-flex align-center text-caption">
              <v-icon small class="mr-1">mdi-arrow-up</v-icon>
              100% <span class="ml-1">vs last month</span>
            </div>
          </v-card>
        </v-col>
      </v-row>
    </template>

    <div class="d-flex flex-wrap align-center mb-6 mt-6">
      <h1 class="text-h4 mr-4 mb-0">
        <strong>Active Departments</strong>
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
            @click="showAddDepartment = true"
          >
            <v-icon left>
              mdi-plus-circle
            </v-icon>
            Add New Department
          </v-btn>
          <v-btn
            color="#6941C6"
            class="mb-2 rounded-lg"
            style="color: white !important; text-transform: capitalize;"
            @click="exportDepartments"
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
            :items="paginatedDepartments"
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

            <template #[`item.actions`]="{ item }">
              <v-icon
                small
                class="mr-2"
                color="#6941C6"
                @click="showEditDepartmentDialog(true, item)"
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
        <v-card-title>Filter Departments</v-card-title>
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
          <v-btn text class="rounded-lg" @click="resetFilters">Reset</v-btn>
          <v-btn color="primary" class="rounded-lg" @click="applyFilters">Apply Filters</v-btn>
        </v-card-actions>
      </v-card>
    </v-dialog>

    <!-- Add Department Dialog -->
    <v-dialog
      v-model="showAddDepartment"
      max-width="800px"
    >
      <v-card class="rounded-lg">
        <v-card-title class="d-flex align-center justify-space-between" style="background-color: gainsboro;">
          <span>Add New Department</span>
          <div>
            <v-btn small outlined class="mr-2 rounded-lg" style="background-color: white;">
              <v-icon small left>
                mdi-tray-arrow-up
              </v-icon>
              Bulk Upload
            </v-btn>
            <v-btn icon @click="showAddDepartment = false">
              <v-icon>mdi-close-circle-outline</v-icon>
            </v-btn>
          </div>
        </v-card-title>
        <v-card-text>
          <v-container fluid class="pa-0 pl-5 pr-5">
            <v-form ref="addDepartmentForm" v-model="validForm">
              <v-row>
                <!-- Department Name -->
                <v-col cols="7" class="mb-1 mt-2">
                  <label class="d-block font-weight-medium mb-1">Department Name</label>
                  <v-text-field
                    v-model="departmentData.departmentName"
                    placeholder="Enter Full Name"
                    outlined
                    dense
                    hide-details
                    class="rounded-lg"
                    required
                  />
                </v-col>

                <!-- Date of Creation -->
                <v-col cols="5" class="mb-1 mt-2">
                  <label class="d-block font-weight-medium mb-1">Date of Creation</label>
                  <v-text-field
                    v-model="departmentData.creationDate"
                    placeholder="Jan 6, 2023"
                    outlined
                    dense
                    hide-details
                    class="rounded-lg"
                    required
                  />
                </v-col>

                <!-- Manager -->
                <v-col cols="7" class="mb-1">
                  <label class="d-block font-weight-medium mb-1">Assign Manager</label>
                  <v-text-field
                    v-model="departmentData.manager"
                    placeholder="Choose Manager Here"
                    outlined
                    dense
                    hide-details
                    class="rounded-lg"
                    required
                  />
                </v-col>

                <!-- Warehouse ID-->
                <v-col cols="5" class="mb-1">
                  <label class="d-block font-weight-medium mb-1">Warehouse ID</label>
                  <v-text-field
                    v-model="departmentData.warehouse"
                    placeholder="Select Specific ID"
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
            @click="addDepartment"
          >
            Add Department
          </v-btn>
        </v-card-actions>
      </v-card>
    </v-dialog>

    <!-- Edit Department Dialog -->
    <v-dialog
      v-model="showEditDepartment"
      max-width="800px"
    >
      <v-card class="rounded-lg">
        <v-card-title class="d-flex align-center justify-space-between" style="background-color: gainsboro;">
          <span>Edit Department</span>
          <div>
            <v-btn small outlined class="mr-2 rounded-lg" style="background-color: white;">
              <v-icon small left>
                mdi-tray-arrow-up
              </v-icon>
              Bulk Upload
            </v-btn>
            <v-btn icon @click="showEditDepartment = false">
              <v-icon>mdi-close-circle-outline</v-icon>
            </v-btn>
          </div>
        </v-card-title>
        <v-card-text>
          <v-container fluid class="pa-0 pl-5 pr-5">
            <v-form ref="addDepartmentForm" v-model="validEditForm">
              <v-row>
                <!-- Department Name -->
                <v-col cols="7" class="mb-1 mt-2">
                  <label class="d-block font-weight-medium mb-1">Department Name</label>
                  <v-text-field
                    v-model="departmentData.departmentName"
                    placeholder="Enter Full Name"
                    outlined
                    dense
                    hide-details
                    class="rounded-lg"
                    required
                  />
                </v-col>

                <!-- Date of Creation -->
                <v-col cols="5" class="mb-1 mt-2">
                  <label class="d-block font-weight-medium mb-1">Date of Creation</label>
                  <v-text-field
                    v-model="departmentData.creationDate"
                    placeholder="Jan 6, 2023"
                    outlined
                    dense
                    hide-details
                    class="rounded-lg"
                    required
                  />
                </v-col>

                <!-- Manager -->
                <v-col cols="7" class="mb-1">
                  <label class="d-block font-weight-medium mb-1">Assign Manager</label>
                  <v-text-field
                    v-model="departmentData.manager"
                    placeholder="Choose Manager Here"
                    outlined
                    dense
                    hide-details
                    class="rounded-lg"
                    required
                  />
                </v-col>

                <!-- Warehouse ID-->
                <v-col cols="5" class="mb-1">
                  <label class="d-block font-weight-medium mb-1">Warehouse ID</label>
                  <v-text-field
                    v-model="departmentData.warehouse"
                    placeholder="Select Specific ID"
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
            @click="editDepartment"
          >
            Update Department
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
          Are you sure you want to delete "{{ departmentToDelete?.fullName }}"? This action cannot be undone.
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
            @click="deleteDepartment"
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
      // Departments
      departmentData: {
        creationDate: '',
        departmentName: '',
        manager: '',
        managerEmail: '',
        managerID: '',
        managerPhoneNumber: '',
        warehouse: '',
        status: ''
      },
      departments: [],
      activeCount: 0,
      inactiveCount: 0,
      deletedCount: 0,
      validForm: false,
      validEditForm: false,
      showAddDepartment: false,
      showEditDepartment: false,
      showDeleteConfirm: false,
      departmentToDelete: null,

      // Filters
      searchQuery: '',
      showFilters: false,
      filters: {
        status: '',
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
        { text: 'Department Name', align: 'start', value: 'departmentName', sortable: true, width: '20%', always: true },
        { text: 'Department ID', value: 'id', width: '10%', breakpoint: 'sm' },
        { text: 'Warehouse ID', value: 'warehouse', width: '10%', breakpoint: 'sm' },
        { text: 'Manager ID', value: 'managerID', width: '8%', breakpoint: 'xs' },
        { text: 'Email-id', value: 'managerEmail', width: '8%', breakpoint: 'xs' },
        { text: 'Phone Number', value: 'managerPhoneNumber', width: '8%', breakpoint: 'xs' },
        { text: 'Actions', value: 'actions', sortable: false, align: 'center', width: '80px', always: true }
      ],

      // Quality of life
      windowWidth: 0
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
  computed: {
    filteredDepartments () {
      return this.departments.filter((c) => {
        const matchesStatus = !this.filters.status || c.status === this.filters.status
        const matchesWarehouse = !this.filters.warehouse || c.warehouse === this.filters.warehouse
        return matchesStatus && matchesWarehouse
      })
    },
    totalPages () {
      return Math.ceil(this.filteredDepartments.length / this.itemsPerPage)
    },
    paginatedDepartments () {
      const start = (this.page - 1) * this.itemsPerPage
      const end = this.page * this.itemsPerPage
      return this.filteredDepartments.slice(start, end)
    }
  },
  mounted () {
    this.loadDepartments()
    window.addEventListener('resize', this.onResize)
    this.windowWidth = window.innerWidth
  },
  beforeDestroy () {
    window.removeEventListener('resize', this.onResize)
  },
  methods: {
    // Employees Methods (Manager)
    async getManagerData () {
      try {
        const manager = await this.$axios.get('/employees/' + this.departmentData.manager)
        this.departmentData.manager = manager.data.fullName
        this.departmentData.managerEmail = manager.data.email
        this.departmentData.managerPhoneNumber = manager.data.phoneNumber
        this.departmentData.managerID = manager.data.id
      } catch (error) {
        const errorMessage = error.message || 'Error al obtener los datos del manager'
        console.log('error al obtener los datos del manager: ', errorMessage)
      }
    },

    // Departments methods
    async addDepartment () {
      try {
        this.departmentData.status = 'active'
        await this.getManagerData()
        await this.$axios.post('/departments/addDepartment', this.departmentData)
        this.loadDepartments()
        this.showAddDepartment = false
      } catch (error) {
        const errorMessage = error.message || 'Error al crear el department'
        console.log('error al Crear Department: ', errorMessage)
      }
    },
    async loadDepartments () {
      try {
        const response = await this.$axios.get('/departments')

        const departmentList = response.data.departments || response.data || []
        this.departments = departmentList.filter(c => c.status !== 'deleted')
        this.departmentOptions = [...new Set(this.departments.map(c => c.departmentID).filter(Boolean))]
        this.warehouseOptions = [...new Set(this.departments.map(c => c.warehouse).filter(Boolean))]

        this.activeCount = departmentList.filter(c => c.status === 'active').length
        this.inactiveCount = departmentList.filter(c => c.status === 'inactive').length
        this.deletedCount = departmentList.filter(c => c.status === 'deleted').length
      } catch (error) {
        const errorMessage = error.message || 'Error al cargar los departments'
        this.$store.dispatch('alert/triggerAlert', {
          message: errorMessage,
          type: 'error'
        })
      }
    },
    async editDepartment () {
      try {
        await this.getManagerData()
        await this.$axios.put(`/departments/updateDepartment/${this.departmentData.id}`, this.departmentData)
        this.loadDepartments()
        this.showEditDepartment = false
      } catch (error) {
        const errorMessage = error.message || 'Error al actualizar el usuario'
        console.log('error al Editar Usuario: ', errorMessage)
      }
    },
    async deleteDepartment () {
      if (this.departmentToDelete) {
        try {
          await this.$axios.patch(`/departments/toggleDepartmentStatus/${this.departmentToDelete.id}`, { status: 'deleted' })
          this.loadDepartments()
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
    async toggleStatusDepartment (item, status) {
      try {
        await this.$axios.patch(`/departments/toggleDepartmentStatus/${item.id}`, { status })
        this.loadDepartments()
        // Mensaje opcional de éxito
      } catch (error) {
        const errorMessage = error.message || 'Error al actualizar el usuario'
        console.error('Error al editar usuario:', errorMessage)
      }
    },
    handleToggleStatus (item) {
      const newStatus = item.status === 'active' ? 'inactive' : 'active'
      this.toggleStatusDepartment(item, newStatus)
      item.status = newStatus // reflejar el cambio en la UI
    },
    exportDepartments () {
      const dataToExport = this.filteredDepartments.map(d => ({
        'Department ID': d.id,
        'Department Name': d.fullName,
        'Date of Creation': d.creationDate,
        'Warewhouse ID': d.warehouse,
        'Manager ID': d.managerID,
        'Manager Name': d.manager,
        'Manager Email': d.managerEmail,
        'Manager Phone Number': d.managerPhoneNumber,
        Status: d.status
      }))

      const worksheet = XLSX.utils.json_to_sheet(dataToExport)
      const workbook = XLSX.utils.book_new()
      XLSX.utils.book_append_sheet(workbook, worksheet, 'Departments')

      XLSX.writeFile(workbook, 'departments.xlsx')
    },
    getAvatarUrl (item) {
      return `https://i.pravatar.cc/150?u=${item.id}`
    },

    // Filters methods
    resetFilters () {
      this.filters = {
        status: '',
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
        item.departmentName?.toLowerCase().includes(text) ||
        item.id?.toLowerCase().includes(text) ||
        item.managerEmail?.toLowerCase().includes(text) ||
        item.managerID?.toLowerCase().includes(text) ||
        item.managerPhoneNumber?.toLowerCase().includes(text)
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
    showEditDepartmentDialog (mode = true, department = null) {
      this.showEditDepartment = mode
      this.departmentData = { ...department }
    },
    confirmDelete (department) {
      this.departmentToDelete = department
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
