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
        Employees
      </h1>
    </div>

    <!-- Info cards row -->
    <template>
      <v-row dense>
        <!-- Active Employees -->
        <v-col cols="12" sm="4" class="pr-6">
          <v-card class="pa-5" color="#6941C6" dark rounded>
            <div class="text-subtitle-1">Active Employees</div>
            <div class="display-1 font-weight-bold my-2">{{ activeCount }}</div>
            <div class="d-flex align-center text-caption">
              <v-icon small class="mr-1">mdi-arrow-up</v-icon>
              100% <span class="ml-1">vs last month</span>
            </div>
          </v-card>
        </v-col>

        <v-spacer />

        <!-- Inactive Employees -->
        <v-col cols="12" sm="4" class="pr-6">
          <v-card class="pa-5" rounded elevartion="4">
            <div class="text-subtitle-1">Inactive Employees</div>
            <div class="display-1 font-weight-bold my-2">{{ inactiveCount }}</div>
            <div class="d-flex align-center text-caption">
              <v-icon small class="mr-1">mdi-arrow-up</v-icon>
              100% <span class="ml-1">vs last month</span>
            </div>
          </v-card>
        </v-col>

        <v-spacer />

        <!-- Deleted Employees -->
        <v-col cols="12" sm="4" class="pr-6">
          <v-card class="pa-5" rounded elevartion="4">
            <div class="text-subtitle-1">Deleted Employees</div>
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
        <strong>Active Employee</strong>
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
            @click="showAddEmployee = true"
          >
            <v-icon left>
              mdi-plus-circle
            </v-icon>
            Add New Costumer
          </v-btn>
          <v-btn
            color="#6941C6"
            class="mb-2 rounded-lg"
            style="color: white !important; text-transform: capitalize;"
            @click="exportEmployees"
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
            :items="paginatedEmployees"
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
                  <img :src="getAvatarUrl(item)" alt="Avatar" />
                </v-avatar>
                <span>{{ item.fullName }}</span>
              </div>
            </template>

            <template #[`item.actions`]="{ item }">
              <v-icon
                small
                class="mr-2"
                color="#6941C6"
                @click="showEditEmployeeDialog(true, item)"
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
        <v-card-title>Filter Employees</v-card-title>
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
          <v-btn text class="rounded-lg" @click="resetFilters">Reset</v-btn>
          <v-btn color="primary" class="rounded-lg" @click="applyFilters">Apply Filters</v-btn>
        </v-card-actions>
      </v-card>
    </v-dialog>

    <!-- Add Employee Dialog -->
    <v-dialog
      v-model="showAddEmployee"
      max-width="800px"
    >
      <v-card class="rounded-lg">
        <v-card-title class="d-flex align-center justify-space-between" style="background-color: gainsboro;">
          <span>Add New Employee</span>
          <div>
            <v-btn small outlined class="mr-2 rounded-lg" style="background-color: white;">
              <v-icon small left>
                mdi-tray-arrow-up
              </v-icon>
              Bulk Upload
            </v-btn>
            <v-btn icon @click="showAddEmployee = false">
              <v-icon>mdi-close-circle-outline</v-icon>
            </v-btn>
          </div>
        </v-card-title>
        <v-card-text>
          <v-container fluid class="pa-0 pl-5 pr-5">
            <v-form ref="addEmployeeForm" v-model="validForm">
              <v-row>
                <!-- Full Name -->
                <v-col cols="8" class="mb-1 mt-2">
                  <label class="d-block font-weight-medium mb-1">Full Name</label>
                  <v-text-field
                    v-model="employeeData.fullName"
                    placeholder="Enter Full Name"
                    outlined
                    dense
                    hide-details
                    class="rounded-lg"
                    required
                  />
                </v-col>

                <!-- Hiring Date -->
                <v-col cols="4" class="mb-1 mt-2">
                  <label class="d-block font-weight-medium mb-1">Hiring Date</label>
                  <v-text-field
                    v-model="employeeData.hiringDate"
                    placeholder="Jan 6, 2023"
                    outlined
                    dense
                    hide-details
                    class="rounded-lg"
                    required
                  />
                </v-col>

                <!-- Phone Number -->
                <v-col cols="4" class="mb-1">
                  <label class="d-block font-weight-medium mb-1">Phone number</label>
                  <v-text-field
                    v-model="employeeData.phoneNumber"
                    placeholder="+1 955 000 0000"
                    outlined
                    dense
                    hide-details
                    class="rounded-lg"
                    required
                  />
                </v-col>

                <!-- Phone Number -->
                <v-col cols="4" class="mb-1">
                  <label class="d-block font-weight-medium mb-1">Warehouse/Store ID</label>
                  <v-text-field
                    v-model="employeeData.warehouse"
                    placeholder="Ex: BRT12234"
                    outlined
                    dense
                    hide-details
                    class="rounded-lg"
                    required
                  />
                </v-col>

                <!-- Department ID -->
                <v-col cols="4" class="mb-1">
                  <label class="d-block font-weight-medium mb-1">Department ID</label>
                  <v-text-field
                    v-model="employeeData.departmentID"
                    placeholder="Ex: BRT12234"
                    outlined
                    dense
                    hide-details
                    class="rounded-lg"
                    required
                  />
                </v-col>

                <!-- Email -->
                <v-col cols="8" class="mb-1">
                  <label class="d-block font-weight-medium mb-1">Email-id</label>
                  <v-text-field
                    v-model="employeeData.email"
                    placeholder="Enter your Email-ID"
                    outlined
                    dense
                    hide-details
                    class="rounded-lg"
                    required
                  />
                </v-col>

                <!-- Government ID -->
                <v-col cols="4" class="mb-1">
                  <label class="d-block font-weight-medium mb-1">Government ID Type</label>
                  <v-text-field
                    v-model="employeeData.governmentID"
                    placeholder="Select Specific ID"
                    outlined
                    dense
                    hide-details
                    class="rounded-lg"
                    required
                  />
                </v-col>

                <v-col cols="8">
                  <!-- State and Pincode -->
                  <v-row>
                    <v-col cols="6" md="6" class="mb-1">
                      <label class="d-block font-weight-medium mb-1">State</label>
                      <v-text-field
                        v-model="employeeData.state"
                        placeholder="Select your state"
                        outlined
                        dense
                        hide-details
                        class="rounded-lg"
                        required
                      />
                    </v-col>
                    <v-col cols="6" md="6" class="mb-1">
                      <label class="d-block font-weight-medium mb-1">Pincode</label>
                      <v-text-field
                        v-model="employeeData.pincode"
                        placeholder="Enter Pincode"
                        outlined
                        dense
                        hide-details
                        class="rounded-lg"
                        required
                      />
                    </v-col>
                  </v-row>
                  <v-row>
                    <!-- Address -->
                    <v-col cols="12" class="mb-1">
                      <label class="d-block font-weight-medium mb-1">Address</label>
                      <v-text-field
                        v-model="employeeData.address"
                        placeholder="Enter your Address"
                        outlined
                        dense
                        hide-details
                        class="rounded-lg"
                        required
                      />
                    </v-col>
                  </v-row>
                </v-col>

                <v-col cols="4" class="mb-1">
                  <label class="d-block font-weight-medium mb-1">Upload Files (ID & Offer Letter)</label>
                  <div
                    class="rounded-lg mt-2 elevation-3"
                    style="border: 2px; background-color: #F1EFF5; height: 120px; padding: 10px; text-align: center; position: relative; display: flex; flex-direction: column; align-items: center; justify-content: center;"
                  >
                    <div
                      style="background-color: white; border-radius: 50%; padding: 10px; box-shadow: 0 1px 4px rgba(0,0,0,0.1);"
                      class="mb-2"
                    >
                      <v-icon size="32" color="grey">mdi-cloud-upload-outline</v-icon>
                    </div>
                    <span style="color: #6941C6; font-weight: 500; font-size: 14px;">Click to upload</span>
                    <span style="font-size: 12px; color: grey;">SVG, PNG, JPG or PDF (max. 2MB)</span>

                    <!-- 👇 Este input está perfectamente contenido en su div -->
                    <input
                      type="file"
                      multiple
                      accept=".svg,.png,.jpg,.jpeg,.pdf"
                      style="position: absolute; top: 0; left: 0; right: 0; bottom: 0; opacity: 0; cursor: pointer; z-index: 5;"
                    />
                  </div>
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
            @click="addEmployee"
          >
            Add Employee
          </v-btn>
        </v-card-actions>
      </v-card>
    </v-dialog>

    <!-- Edit Employee Dialog -->
    <v-dialog
      v-model="showEditEmployee"
      max-width="800px"
    >
      <v-card class="rounded-lg">
        <v-card-title class="d-flex align-center justify-space-between" style="background-color: gainsboro;">
          <span>EditEmployee</span>
          <div>
            <v-btn small outlined class="mr-2 rounded-lg" style="background-color: white;">
              <v-icon small left>
                mdi-tray-arrow-up
              </v-icon>
              Bulk Upload
            </v-btn>
            <v-btn icon @click="showEditEmployee = false">
              <v-icon>mdi-close-circle-outline</v-icon>
            </v-btn>
          </div>
        </v-card-title>
        <v-card-text>
          <v-container fluid class="pa-0 pl-5 pr-5">
            <v-form ref="addEmployeeForm" v-model="validEditForm">
              <v-row>
                <!-- Full Name -->
                <v-col cols="8" class="mb-1 mt-2">
                  <label class="d-block font-weight-medium mb-1">Full Name</label>
                  <v-text-field
                    v-model="employeeData.fullName"
                    placeholder="Enter Full Name"
                    outlined
                    dense
                    hide-details
                    class="rounded-lg"
                    required
                  />
                </v-col>

                <!-- Hiring Date -->
                <v-col cols="4" class="mb-1 mt-2">
                  <label class="d-block font-weight-medium mb-1">Hiring Date</label>
                  <v-text-field
                    v-model="employeeData.hiringDate"
                    placeholder="Jan 6, 2023"
                    outlined
                    dense
                    hide-details
                    class="rounded-lg"
                    required
                  />
                </v-col>

                <!-- Phone Number -->
                <v-col cols="4" class="mb-1">
                  <label class="d-block font-weight-medium mb-1">Phone number</label>
                  <v-text-field
                    v-model="employeeData.phoneNumber"
                    placeholder="+1 955 000 0000"
                    outlined
                    dense
                    hide-details
                    class="rounded-lg"
                    required
                  />
                </v-col>

                <!-- Phone Number -->
                <v-col cols="4" class="mb-1">
                  <label class="d-block font-weight-medium mb-1">Warehouse/Store ID</label>
                  <v-text-field
                    v-model="employeeData.warehouse"
                    placeholder="Ex: BRT12234"
                    outlined
                    dense
                    hide-details
                    class="rounded-lg"
                    required
                  />
                </v-col>

                <!-- Department ID -->
                <v-col cols="4" class="mb-1">
                  <label class="d-block font-weight-medium mb-1">Department ID</label>
                  <v-text-field
                    v-model="employeeData.departmentID"
                    placeholder="Ex: BRT12234"
                    outlined
                    dense
                    hide-details
                    class="rounded-lg"
                    required
                  />
                </v-col>

                <!-- Email -->
                <v-col cols="8" class="mb-1">
                  <label class="d-block font-weight-medium mb-1">Email-id</label>
                  <v-text-field
                    v-model="employeeData.email"
                    placeholder="Enter your Email-ID"
                    outlined
                    dense
                    hide-details
                    class="rounded-lg"
                    required
                  />
                </v-col>

                <!-- Government ID -->
                <v-col cols="4" class="mb-1">
                  <label class="d-block font-weight-medium mb-1">Government ID Type</label>
                  <v-text-field
                    v-model="employeeData.governmentID"
                    placeholder="Select Specific ID"
                    outlined
                    dense
                    hide-details
                    class="rounded-lg"
                    required
                  />
                </v-col>

                <v-col cols="8">
                  <!-- State and Pincode -->
                  <v-row>
                    <v-col cols="6" md="6" class="mb-1">
                      <label class="d-block font-weight-medium mb-1">State</label>
                      <v-text-field
                        v-model="employeeData.state"
                        placeholder="Select your state"
                        outlined
                        dense
                        hide-details
                        class="rounded-lg"
                        required
                      />
                    </v-col>
                    <v-col cols="6" md="6" class="mb-1">
                      <label class="d-block font-weight-medium mb-1">Pincode</label>
                      <v-text-field
                        v-model="employeeData.pincode"
                        placeholder="Enter Pincode"
                        outlined
                        dense
                        hide-details
                        class="rounded-lg"
                        required
                      />
                    </v-col>
                  </v-row>
                  <v-row>
                    <!-- Address -->
                    <v-col cols="12" class="mb-1">
                      <label class="d-block font-weight-medium mb-1">Address</label>
                      <v-text-field
                        v-model="employeeData.address"
                        placeholder="Enter your Address"
                        outlined
                        dense
                        hide-details
                        class="rounded-lg"
                        required
                      />
                    </v-col>
                  </v-row>
                </v-col>

                <v-col cols="4" class="mb-1">
                  <label class="d-block font-weight-medium mb-1">Upload Files (ID & Offer Letter)</label>
                  <div
                    class="rounded-lg mt-2 elevation-3"
                    style="border: 2px; background-color: #F1EFF5; height: 120px; padding: 10px; text-align: center; position: relative; display: flex; flex-direction: column; align-items: center; justify-content: center;"
                  >
                    <div
                      style="background-color: white; border-radius: 50%; padding: 10px; box-shadow: 0 1px 4px rgba(0,0,0,0.1);"
                      class="mb-2"
                    >
                      <v-icon size="32" color="grey">mdi-cloud-upload-outline</v-icon>
                    </div>
                    <span style="color: #6941C6; font-weight: 500; font-size: 14px;">Click to upload</span>
                    <span style="font-size: 12px; color: grey;">SVG, PNG, JPG or PDF (max. 2MB)</span>

                    <!-- 👇 Este input está perfectamente contenido en su div -->
                    <input
                      type="file"
                      multiple
                      accept=".svg,.png,.jpg,.jpeg,.pdf"
                      style="position: absolute; top: 0; left: 0; right: 0; bottom: 0; opacity: 0; cursor: pointer; z-index: 5;"
                    />
                  </div>
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
            @click="editEmployee"
          >
            Update Employee
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
          Are you sure you want to delete "{{ employeeToDelete?.fullName }}"? This action cannot be undone.
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
            @click="deleteEmployee"
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
      // Employees
      employeeData: {
        fullName: '',
        hiringDate: '',
        phoneNumber: '',
        warehouse: '',
        departmentID: '',
        email: '',
        governmentID: '',
        state: '',
        pincode: '',
        address: '',
        status: ''
      },
      employees: [],
      activeCount: 0,
      inactiveCount: 0,
      deletedCount: 0,
      validForm: false,
      validEditForm: false,
      showAddEmployee: false,
      showEditEmployee: false,
      showDeleteConfirm: false,
      employeeToDelete: null,

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
        { text: 'Employee Name', align: 'start', value: 'fullName', sortable: true, width: '20%', always: true },
        { text: 'Employee ID', value: 'id', width: '10%', breakpoint: 'sm' },
        { text: 'Department ID', value: 'departmentID', width: '10%', breakpoint: 'md' },
        { text: 'Warehouse ID', value: 'warehouse', width: '10%', breakpoint: 'sm' },
        { text: 'Hire Date', value: 'hiringDate', width: '8%', breakpoint: 'xs' },
        { text: 'Phone Number', value: 'phoneNumber', width: '8%', breakpoint: 'xs' },
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
    filteredEmployees () {
      return this.employees.filter((c) => {
        const matchesStatus = !this.filters.status || c.status === this.filters.status
        const matchesDepartmentID = !this.filters.departmentID || c.departmentID === this.filters.departmentID
        const matchesWarehouse = !this.filters.warehouse || c.warehouse === this.filters.warehouse
        return matchesStatus && matchesDepartmentID && matchesWarehouse
      })
    },
    totalPages () {
      return Math.ceil(this.filteredEmployees.length / this.itemsPerPage)
    },
    paginatedEmployees () {
      const start = (this.page - 1) * this.itemsPerPage
      const end = this.page * this.itemsPerPage
      return this.filteredEmployees.slice(start, end)
    }
  },
  mounted () {
    this.loadEmployees()
    window.addEventListener('resize', this.onResize)
    this.windowWidth = window.innerWidth
  },
  beforeDestroy () {
    window.removeEventListener('resize', this.onResize)
  },
  methods: {
    // Employees methods
    async addEmployee () {
      try {
        this.employeeData.status = 'active'
        await this.$axios.post('/employees/addEmployee', this.employeeData)
        this.loadEmployees()
        this.showAddEmployee = false
      } catch (error) {
        const errorMessage = error.message || 'Error al crear el employee'
        console.log('error al Crear Employee: ', errorMessage)
      }
    },
    async loadEmployees () {
      try {
        const response = await this.$axios.get('/employees')

        const employeeList = response.data.employees || response.data || []
        this.employees = employeeList.filter(c => c.status !== 'deleted')
        this.departmentOptions = [...new Set(this.employees.map(c => c.departmentID).filter(Boolean))]
        this.warehouseOptions = [...new Set(this.employees.map(c => c.warehouse).filter(Boolean))]

        this.activeCount = employeeList.filter(c => c.status === 'active').length
        this.inactiveCount = employeeList.filter(c => c.status === 'inactive').length
        this.deletedCount = employeeList.filter(c => c.status === 'deleted').length
      } catch (error) {
        const errorMessage = error.message || 'Error al cargar los employees'
        this.$store.dispatch('alert/triggerAlert', {
          message: errorMessage,
          type: 'error'
        })
      }
    },
    async editEmployee () {
      try {
        await this.$axios.put(`/employees/updateEmployee/${this.employeeData.id}`, this.employeeData)
        /* this.$store.dispatch('alert/triggerAlert', {
          message: 'Usuario Actualizado Con Éxito',
          type: 'success'
        }) */
        this.loadEmployees()
        this.showEditEmployee = false
      } catch (error) {
        const errorMessage = error.message || 'Error al actualizar el usuario'
        console.log('error al Editar Usuario: ', errorMessage)
        /* this.$store.dispatch('alert/triggerAlert', {
          message: errorMessage,
          type: 'error'
        }) */
      }
    },
    async deleteEmployee () {
      if (this.employeeToDelete) {
        try {
          await this.$axios.patch(`/employees/toggleEmployeeStatus/${this.employeeToDelete.id}`, { status: 'deleted' })
          this.loadEmployees()
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
    async toggleStatusEmployee (item, status) {
      try {
        await this.$axios.patch(`/employees/toggleEmployeeStatus/${item.id}`, { status })
        this.loadEmployees()
        // Mensaje opcional de éxito
      } catch (error) {
        const errorMessage = error.message || 'Error al actualizar el usuario'
        console.error('Error al editar usuario:', errorMessage)
      }
    },
    handleToggleStatus (item) {
      const newStatus = item.status === 'active' ? 'inactive' : 'active'
      this.toggleStatusEmployee(item, newStatus)
      item.status = newStatus // reflejar el cambio en la UI
    },
    exportEmployees () {
      const dataToExport = this.filteredEmployees.map(e => ({
        'Employee Name': e.fullName,
        'Employee ID': e.id,
        'Hiring Date': e.hiringDate,
        'Phone Number': e.phoneNumber,
        'Warewhouse ID': e.warehouse,
        'Department ID': e.departmentID,
        Email: e.email,
        'Government ID': e.governmentID,
        State: e.state,
        Pincode: e.pincode,
        Address: e.address,
        Status: e.status
      }))

      const worksheet = XLSX.utils.json_to_sheet(dataToExport)
      const workbook = XLSX.utils.book_new()
      XLSX.utils.book_append_sheet(workbook, worksheet, 'Employees')

      XLSX.writeFile(workbook, 'employees.xlsx')
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
    showEditEmployeeDialog (mode = true, employee = null) {
      this.showEditEmployee = mode
      this.employeeData = { ...employee }
    },
    confirmDelete (employee) {
      this.employeeToDelete = employee
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
