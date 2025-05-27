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
        Category
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
        <!-- Active Categories -->
        <v-col cols="12" sm="4" class="pr-6">
          <v-card class="pa-5" color="#6941C6" dark rounded>
            <div class="text-subtitle-1">
              Active Categories
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

        <!-- Inactive Categories -->
        <v-col cols="12" sm="4" class="pr-6">
          <v-card class="pa-5" rounded elevartion="4">
            <div class="text-subtitle-1">
              Inactive Categories
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

        <!-- Deleted Categories -->
        <v-col cols="12" sm="4" class="pr-6">
          <v-card class="pa-5" rounded elevartion="4">
            <div class="text-subtitle-1">
              Deleted Categories
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
        <strong>Active Categories</strong>
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
            @click="showAddCategory = true"
          >
            <v-icon left>
              mdi-plus-circle
            </v-icon>
            Add New Category
          </v-btn>
          <v-btn
            color="#6941C6"
            class="mb-2 rounded-lg"
            style="color: white !important; text-transform: capitalize;"
            @click="exportCategories"
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
            :items="paginatedCategories"
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
                @click="showEditCategoryDialog(true, item)"
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
        <v-card-title>Filter Category</v-card-title>
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

    <!-- Add Category Dialog -->
    <v-dialog
      v-model="showAddCategory"
      max-width="800px"
    >
      <v-card class="rounded-lg">
        <v-card-title class="d-flex align-center justify-space-between" style="background-color: gainsboro;">
          <span>Add New Category</span>
          <div>
            <v-btn icon @click="showAddCategory = false">
              <v-icon>mdi-close-circle-outline</v-icon>
            </v-btn>
          </div>
        </v-card-title>
        <v-card-text>
          <v-container fluid class="pa-0 pl-5 pr-5">
            <v-form ref="addCategoryForm" v-model="validForm">
              <v-row>
                <!-- Full Name -->
                <v-col cols="8" class="mb-1 mt-2">
                  <label class="d-block font-weight-medium mb-1">Category Name</label>
                  <v-text-field
                    v-model="categoryData.categoryName"
                    placeholder="Enter Category Name"
                    outlined
                    dense
                    hide-details
                    class="rounded-lg"
                    required
                  />
                </v-col>
                <v-col cols="12" class="mb-1 mt-2">
                  <label class="d-block font-weight-medium mb-1">Description</label>
                  <v-text-field
                    v-model="categoryData.categoryDescription"
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
            @click="addCategory"
          >
            Add Category
          </v-btn>
        </v-card-actions>
      </v-card>
    </v-dialog>

    <!-- Edit Catgory Dialog -->
    <v-dialog
      v-model="showEditCategory"
      max-width="800px"
    >
      <v-card class="rounded-lg">
        <v-card-title class="d-flex align-center justify-space-between" style="background-color: gainsboro;">
          <span>Edit Category</span>
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
            <v-form ref="addCategoryForm" v-model="validEditForm">
              <v-row>
                <!-- Full Name -->
                <v-col cols="8" class="mb-1 mt-2">
                  <label class="d-block font-weight-medium mb-1">Category Name</label>
                  <v-text-field
                    v-model="categoryData.categoryName"
                    placeholder="Enter Category Name"
                    outlined
                    dense
                    hide-details
                    class="rounded-lg"
                    required
                  />
                </v-col>
                <v-col cols="12" class="mb-1 mt-2">
                  <label class="d-block font-weight-medium mb-1">Description</label>
                  <v-text-field
                    v-model="categoryData.categoryDescription"
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
            @click="editCategory"
          >
            Update Category
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
          Are you sure you want to delete "{{ categoryToDelete?.fullName }}"? This action cannot be undone.
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
            @click="deleteCategory"
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
      // Categories
      categoryData: {
        categoryName: '',
        categoryDescription: '',
        status: ''
      },
      categories: [],
      activeCount: 0,
      inactiveCount: 0,
      deletedCount: 0,
      validForm: false,
      validEditForm: false,
      showAddCategory: false,
      showEditCategory: false,
      showDeleteConfirm: false,
      categoryToDelete: null,

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
        { text: 'Category Name', align: 'start', value: 'categoryName', sortable: true, width: '20%', always: true },
        { text: 'Category ID', value: 'id', width: '10%', breakpoint: 'sm' },
        { text: 'Description', value: 'categoryDescription', width: '10%', breakpoint: 'md' },
        { text: 'Products', value: 'products', width: '10%', breakpoint: 'sm' },
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
    filteredCategories () {
      return this.categories.filter((c) => {
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
      return Math.ceil(this.filteredCategories.length / this.itemsPerPage)
    },
    paginatedCategories () {
      const start = (this.page - 1) * this.itemsPerPage
      const end = this.page * this.itemsPerPage
      return this.filteredCategories.slice(start, end)
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
    this.loadCategories()
    window.addEventListener('resize', this.onResize)
    this.windowWidth = window.innerWidth
  },
  beforeDestroy () {
    window.removeEventListener('resize', this.onResize)
  },
  methods: {
    // Categories methods
    async addCategory () {
      try {
        this.categoryData.status = 'active'
        await this.$axios.post('/categories/addCategory', this.categoryData)
        this.loadCategories()
        this.showAddCategory = false
      } catch (error) {
        const errorMessage = error.message || 'Error al crear la Category'
        console.log('error al Crear Category: ', errorMessage)
      }
    },
    async loadCategories () {
      try {
        const response = await this.$axios.get('/categories')

        const categoriesList = response.data.categories || response.data || []
        this.categories = categoriesList.filter(c => c.status !== 'deleted')
        this.departmentOptions = [...new Set(this.categories.map(c => c.departmentID).filter(Boolean))]
        this.warehouseOptions = [...new Set(this.categories.map(c => c.warehouse).filter(Boolean))]

        this.activeCount = categoriesList.filter(c => c.status === 'active').length
        this.inactiveCount = categoriesList.filter(c => c.status === 'inactive').length
        this.deletedCount = categoriesList.filter(c => c.status === 'deleted').length
      } catch (error) {
        const errorMessage = error.message || 'Error al cargar las categories'
        this.$store.dispatch('alert/triggerAlert', {
          message: errorMessage,
          type: 'error'
        })
      }
    },
    async editCategory () {
      try {
        await this.$axios.put(`/categories/updateCategory/${this.categoryData.id}`, this.categoryData)
        /* this.$store.dispatch('alert/triggerAlert', {
          message: 'Usuario Actualizado Con Éxito',
          type: 'success'
        }) */
        this.loadCategories()
        this.showEditCategory = false
      } catch (error) {
        const errorMessage = error.message || 'Error al actualizar el usuario'
        console.log('error al Editar Usuario: ', errorMessage)
        /* this.$store.dispatch('alert/triggerAlert', {
          message: errorMessage,
          type: 'error'
        }) */
      }
    },
    async deleteCategory () {
      if (this.categoryToDelete) {
        try {
          await this.$axios.patch(`/categories/toggleCategoryStatus/${this.categoryData.id}`, { status: 'deleted' })
          this.loadCategories()
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
    async toggleStatusCategory (item, status) {
      try {
        await this.$axios.patch(`/categories/toggleCategoryStatus/${item.id}`, { status })
        this.loadCategories()
        // Mensaje opcional de éxito
      } catch (error) {
        const errorMessage = error.message || 'Error al actualizar el usuario'
        console.error('Error al editar usuario:', errorMessage)
      }
    },
    handleToggleStatus (item) {
      const newStatus = item.status === 'active' ? 'inactive' : 'active'
      this.toggleStatusCategory(item, newStatus)
      item.status = newStatus // reflejar el cambio en la UI
    },
    exportCategories () {
      const dataToExport = this.filteredCategories.map(e => ({
        'Category Name': e.categoryName,
        'Category ID': e.id,
        Description: e.categoryDescription,
        Status: e.status
      }))

      const worksheet = XLSX.utils.json_to_sheet(dataToExport)
      const workbook = XLSX.utils.book_new()
      XLSX.utils.book_append_sheet(workbook, worksheet, 'Categories')

      XLSX.writeFile(workbook, 'categories.xlsx')
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
    showEditCategryDialog (mode = true, category = null) {
      this.showEditCategory = mode
      this.categoryData = { ...category }
    },
    confirmDelete (category) {
      this.categoryToDelete = category
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
