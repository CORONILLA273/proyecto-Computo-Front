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
        Products
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
          @click="showAddProduct = true"
        >
          <v-icon left>
            mdi-plus-circle
          </v-icon>
          Add New Product
        </v-btn>
        <v-btn
          color="#6941C6"
          class="mb-2 rounded-lg"
          style="color: white !important; text-transform: capitalize;"
          @click="exportProducts"
        >
          <v-icon color="white" left>
            mdi-download
          </v-icon>
          Export
        </v-btn>
      </v-col>
    </v-row>

    <client-only>
      <div class="table-responsive">
        <v-data-table
          v-model="selectedProducts"
          :headers="getVisibleHeaders"
          :items="products"
          :items-per-page="itemsPerPage"
          :page.sync="currentPage"
          :search="searchQuery"
          :custom-filter="customFilter"
          show-select
          class="elevation-1 rounded-lg"
          :footer-props="{
            'items-per-page-options': [5, 8, 10, 15, 20]
          }"
        >
          <template #[`item.image`]="{ item }">
            <v-avatar size="40">
              <v-img :src="item.image || '/placeholder.png'" :alt="item.name" />
            </v-avatar>
          </template>

          <template #[`item.name`]="{ item }">
            <div class="d-flex align-center">
              <span class="ml-2">{{ item.name }}</span>
            </div>
          </template>

          <template #[`item.supplierId`]="{ item }">
            <div class="d-flex align-center">
              <a href="#" class="text-decoration-underline primary--text">{{ item.supplierId }}</a>
              <v-icon
                v-if="item.supplierVerified"
                color="blue"
                small
                class="ml-1"
              >
                mdi-check-circle
              </v-icon>
            </div>
          </template>

          <template #[`item.category`]="{ item }">
            <a href="#" class="text-decoration-underline primary--text">{{ item.category }}</a>
          </template>

          <template #[`item.stockLevel`]="{ item }">
            {{ formatNumber(item.stockLevel) }}
          </template>

          <template #[`item.maxStock`]="{ item }">
            {{ formatNumber(item.maxStock) }}
          </template>

          <template #[`item.actions`]="{ item }">
            <v-icon
              small
              class="mr-2"
              color="#6941C6"
              @click="editProduct(item)"
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
          </template>
        </v-data-table>
      </div>
    </client-only>

    <!-- Filter Dialog -->
    <v-dialog
      v-model="showFilters"
      max-width="500px"
    >
      <v-card class="rounded-lg">
        <v-card-title>Filter Products</v-card-title>
        <v-card-text>
          <v-container>
            <v-row>
              <v-col cols="12">
                <v-select
                  v-model="filters.category"
                  :items="categories"
                  item-text="label"
                  item-value="value"
                  label="Category"
                  outlined
                  clearable
                  class="rounded-lg"
                />
              </v-col>
              <v-col cols="12">
                <v-select
                  v-model="filters.supplier"
                  :items="suppliers"
                  item-text="label"
                  item-value="value"
                  label="Supplier"
                  outlined
                  clearable
                  class="rounded-lg"
                />
              </v-col>
              <v-col cols="12">
                <v-subheader>Price Range</v-subheader>
                <v-row>
                  <v-col cols="5">
                    <v-text-field
                      v-model="filters.minPrice"
                      label="Min"
                      type="number"
                      outlined
                      dense
                      class="rounded-lg"
                    />
                  </v-col>
                  <v-col cols="2" class="d-flex justify-center align-center">
                    to
                  </v-col>
                  <v-col cols="5">
                    <v-text-field
                      v-model="filters.maxPrice"
                      label="Max"
                      type="number"
                      outlined
                      dense
                      class="rounded-lg"
                    />
                  </v-col>
                </v-row>
              </v-col>
              <v-col cols="12">
                <v-subheader>Stock Level</v-subheader>
                <v-row>
                  <v-col cols="5">
                    <v-text-field
                      v-model="filters.minStock"
                      label="Min"
                      type="number"
                      outlined
                      dense
                      class="rounded-lg"
                    />
                  </v-col>
                  <v-col cols="2" class="d-flex justify-center align-center">
                    to
                  </v-col>
                  <v-col cols="5">
                    <v-text-field
                      v-model="filters.maxStock"
                      label="Max"
                      type="number"
                      outlined
                      dense
                      class="rounded-lg"
                    />
                  </v-col>
                </v-row>
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

    <!-- Add Product Dialog -->
    <v-dialog
      v-model="showAddProduct"
      max-width="800px"
    >
      <v-card class="rounded-lg">
        <v-card-title class="d-flex align-center justify-space-between" style="background-color: gainsboro;">
          <span>Add new product</span>
          <div>
            <v-btn small outlined class="mr-2 rounded-lg">
              <v-icon small left>
                mdi-plus-circle-outline
              </v-icon>
              Add Custom Field
            </v-btn>
            <v-btn small outlined class="mr-2 rounded-lg">
              <v-icon small left>
                mdi-tray-arrow-up
              </v-icon>
              Bulk Upload
            </v-btn>
            <v-btn icon @click="showAddProduct = false">
              <v-icon>mdi-close-circle-outline</v-icon>
            </v-btn>
          </div>
        </v-card-title>
        <v-card-text>
          <v-container>
            <v-form ref="addProductForm" v-model="validForm">
              <v-row>
                <!-- First row -->
                <v-col cols="12" md="4">
                  <label>Product Name</label>
                  <v-text-field
                    v-model="newProduct.name"
                    placeholder="Ex: Bloomlign"
                    outlined
                    dense
                    class="rounded-lg"
                  />
                </v-col>
                <v-col cols="12" md="4">
                  <label>Supplier ID</label>
                  <v-text-field
                    v-model="newProduct.supplierId"
                    placeholder="Ex: TUW10234"
                    outlined
                    dense
                    class="rounded-lg"
                  />
                </v-col>
                <v-col cols="12" md="4">
                  <label>Weight (in lbs)</label>
                  <v-text-field
                    v-model="newProduct.weight"
                    placeholder="Enter Weight here"
                    outlined
                    dense
                    class="rounded-lg"
                  />
                </v-col>

                <!-- Second row -->
                <v-col cols="12" md="4">
                  <label>Category</label>
                  <v-select
                    v-model="newProduct.category"
                    :items="categories"
                    item-text="label"
                    item-value="label"
                    outlined
                    dense
                    class="rounded-lg"
                  />
                </v-col>
                <v-col cols="12" md="4">
                  <label>Dimension Unit</label>
                  <v-select
                    v-model="newProduct.dimensionUnit"
                    :items="dimensionUnits"
                    outlined
                    dense
                    class="rounded-lg"
                  />
                </v-col>
                <v-col cols="12" md="4">
                  <label>Dimensions (L x W x H)</label>
                  <v-text-field
                    v-model="newProduct.dimensions"
                    placeholder="20 × 30 × 40"
                    outlined
                    dense
                    class="rounded-lg"
                  />
                </v-col>

                <!-- Third row -->
                <v-col cols="12" md="4">
                  <label>Recorded Stock Level</label>
                  <v-text-field
                    v-model="newProduct.reorderLevel"
                    placeholder="Ex: 2000"
                    outlined
                    dense
                    class="rounded-lg"
                    type="number"
                  />
                </v-col>
                <v-col cols="12" md="4">
                  <label>Warning Threshold Stock Level</label>
                  <v-text-field
                    v-model="newProduct.warningLevel"
                    placeholder="Ex: 100"
                    outlined
                    dense
                    class="rounded-lg"
                    type="number"
                  />
                </v-col>
                <v-col cols="12" md="4">
                  <label>Auto Order Stock Level</label>
                  <v-text-field
                    v-model="newProduct.autoOrderLevel"
                    placeholder="Ex: 50"
                    outlined
                    dense
                    class="rounded-lg"
                    type="number"
                  />
                </v-col>

                <!-- Fourth row -->
                <v-col cols="12" md="4">
                  <label>SKU Code</label>
                  <v-text-field
                    v-model="newProduct.sku"
                    placeholder="RTY1234455"
                    outlined
                    dense
                    class="rounded-lg"
                  />
                </v-col>
                <v-col cols="12" md="4">
                  <label>Barcode Number</label>
                  <v-text-field
                    v-model="newProduct.barcode"
                    placeholder="QWERTY5987"
                    outlined
                    dense
                    class="rounded-lg"
                  />
                </v-col>
                <v-col cols="12" md="4">
                  <label>GRN Number</label>
                  <v-text-field
                    v-model="newProduct.gtin"
                    placeholder="QWERTY56787"
                    outlined
                    dense
                    class="rounded-lg"
                  />
                </v-col>

                <!-- Fifth row -->
                <v-col cols="12" md="4">
                  <label>Insert Image (400px x 400px)</label>
                  <div class="image-upload-container">
                    <div class="image-upload-area" @click="triggerFileInput">
                      <input
                        ref="fileInput"
                        type="file"
                        accept="image/*"
                        style="display: none"
                        @change="onFileSelected"
                      >
                      <div v-if="!previewImage" class="upload-placeholder">
                        <v-icon size="40" color="#E4E7EC">
                          mdi-plus
                        </v-icon>
                      </div>
                      <v-img
                        v-else
                        :src="previewImage"
                        max-height="150"
                        contain
                      />
                    </div>
                  </div>
                </v-col>
                <v-col cols="12" md="8">
                  <v-row>
                    <v-col cols="12" md="6">
                      <label>Purchasing Price</label>
                      <v-text-field
                        v-model="newProduct.purchasePrice"
                        placeholder="Ex: $100"
                        outlined
                        dense
                        class="rounded-lg"
                      />
                    </v-col>
                    <v-col cols="12" md="6">
                      <label>Selling Price Margin</label>
                      <v-text-field
                        v-model="newProduct.margin"
                        placeholder="Ex: 20%"
                        outlined
                        dense
                        class="rounded-lg"
                      />
                    </v-col>

                    <!-- Sixth row -->
                    <v-col cols="12">
                      <label>Product Description</label>
                      <v-textarea
                        v-model="newProduct.description"
                        label="Product Description"
                        placeholder="Ex: Type something about product here"
                        outlined
                        rows="3"
                        class="rounded-lg"
                      />
                    </v-col>
                  </v-row>
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
            @click="addProduct"
          >
            Add Product
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
          Are you sure you want to delete "{{ productToDelete?.name }}"? This action cannot be undone.
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
            @click="deleteProduct"
          >
            Delete
          </v-btn>
        </v-card-actions>
      </v-card>
    </v-dialog>
  </v-container>
</template>

<script>
export default {
  name: 'ProductsPage',
  layout: 'default',
  data () {
    return {
      searchQuery: '',
      selectedProducts: [],
      currentPage: 1,
      itemsPerPage: 8,
      showFilters: false,
      showAddProduct: false,
      showDeleteConfirm: false,
      productToDelete: null,
      validForm: false,
      windowWidth: 0,
      selectedTimeframe: '1m', // Default selected timeframe
      dateMenu: false, // Controls the date picker menu
      dateRange: [], // Stores the selected date range
      previewImage: null, // For image preview
      dimensionUnits: ['inch', 'cm', 'mm', 'ft'],
      filters: {
        category: '',
        supplier: '',
        minPrice: '',
        maxPrice: '',
        minStock: '',
        maxStock: ''
      },
      newProduct: {
        name: '',
        id: '',
        supplierId: '',
        category: 'Vapes',
        dimensionUnit: 'inch',
        dimensions: '',
        weight: '',
        reorderLevel: '',
        warningLevel: '',
        autoOrderLevel: '',
        sku: '',
        barcode: '',
        gtin: '',
        purchasePrice: '',
        margin: '',
        stockLevel: '',
        maxStock: '',
        description: ''
      },
      headers: [
        {
          text: 'Product Name',
          align: 'start',
          value: 'name',
          sortable: true,
          width: '20%',
          always: true
        },
        {
          text: 'Product ID',
          value: 'id',
          width: '10%',
          breakpoint: 'sm'
        },
        {
          text: 'Supplier ID',
          value: 'supplierId',
          width: '10%',
          breakpoint: 'md'
        },
        {
          text: 'Category',
          value: 'category',
          width: '10%',
          breakpoint: 'sm'
        },
        {
          text: 'Price',
          value: 'price',
          width: '8%',
          breakpoint: 'xs'
        },
        {
          text: 'Weight',
          value: 'weight',
          width: '8%',
          breakpoint: 'lg'
        },
        {
          text: 'Stock Level',
          value: 'stockLevel',
          align: 'end',
          width: '10%',
          breakpoint: 'md'
        },
        {
          text: 'Max Stock',
          value: 'maxStock',
          align: 'end',
          width: '10%',
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
      ],
      categories: [
        { value: 'traditional-vapes', label: 'Traditional Vapes' },
        { value: 'e-cigarettes', label: 'E-Cigarettes' },
        { value: 'edibles', label: 'Edibles' },
        { value: 'vapes', label: 'Vapes' }
      ],
      suppliers: [
        { value: 'rema0123', label: 'REMA0123' },
        { value: 'supp0456', label: 'SUPP0456' },
        { value: 'vend0789', label: 'VEND0789' }
      ],
      products: [
        {
          id: 'TUX001234',
          name: 'Droned Vape',
          supplierId: 'REMA0123',
          supplierVerified: true,
          category: 'Traditional Vapes',
          price: '$45.00',
          weight: '3 lb',
          stockLevel: 12000,
          maxStock: 15000,
          image: '/placeholder.png'
        },
        {
          id: 'TUX001234',
          name: 'Crosscut E-Cig',
          supplierId: 'REMA0123',
          supplierVerified: false,
          category: 'E-Cigarettes',
          price: '$45.00',
          weight: '3 lb',
          stockLevel: 12000,
          maxStock: 15000,
          image: '/placeholder.png'
        },
        {
          id: 'TUX001234',
          name: 'Cultyvate',
          supplierId: 'REMA0123',
          supplierVerified: true,
          category: 'Edibles',
          price: '$45.00',
          weight: '3 lb',
          stockLevel: 12000,
          maxStock: 15000,
          image: '/placeholder.png'
        },
        {
          id: 'TUX001234',
          name: 'Demi High',
          supplierId: 'REMA0123',
          supplierVerified: false,
          category: 'Traditional Vapes',
          price: '$45.00',
          weight: '3 lb',
          stockLevel: 12000,
          maxStock: 15000,
          image: '/placeholder.png'
        },
        {
          id: 'TUX001234',
          name: 'Candice Wu',
          supplierId: 'REMA0123',
          supplierVerified: true,
          category: 'Edibles',
          price: '$45.00',
          weight: '3 lb',
          stockLevel: 12000,
          maxStock: 15000,
          image: '/placeholder.png'
        },
        {
          id: 'TUX001234',
          name: 'Natali Craig',
          supplierId: 'REMA0123',
          supplierVerified: false,
          category: 'E-Cigarettes',
          price: '$45.00',
          weight: '3 lb',
          stockLevel: 12000,
          maxStock: 15000,
          image: '/placeholder.png'
        },
        {
          id: 'TUX001234',
          name: 'Drew Cano',
          supplierId: 'REMA0123',
          supplierVerified: true,
          category: 'Edibles',
          price: '$45.00',
          weight: '3 lb',
          stockLevel: 12000,
          maxStock: 15000,
          image: '/placeholder.png'
        },
        {
          id: 'TUX001234',
          name: 'Orlando Diggs',
          supplierId: 'REMA0123',
          supplierVerified: false,
          category: 'E-Cigarettes',
          price: '$45.00',
          weight: '3 lb',
          stockLevel: 12000,
          maxStock: 15000,
          image: '/placeholder.png'
        }
      ]
    }
  },
  computed: {
    dateRangeText () {
      if (!this.dateRange || this.dateRange.length === 0) {
        return 'Select dates'
      }

      if (this.dateRange.length === 1) {
        return this.dateRange[0]
      }

      return `${this.dateRange[0]} - ${this.dateRange[1]}`
    },
    getVisibleHeaders () {
      // Determina qué columnas mostrar según el ancho de la ventana
      return this.headers.filter((header) => {
        if (header.always) { return true }
        if (!header.breakpoint) { return true }

        // Lógica para mostrar/ocultar columnas según el ancho
        switch (header.breakpoint) {
          case 'xs': return this.windowWidth > 0
          case 'sm': return this.windowWidth >= 600
          case 'md': return this.windowWidth >= 960
          case 'lg': return this.windowWidth >= 1264
          case 'xl': return this.windowWidth >= 1904
          default: return true
        }
      })
    }
  },
  watch: {
    dateRange (val) {
      // Aquí puedes implementar lógica para filtrar datos por fecha
      console.log('Date range changed:', val)
    }
  },
  mounted () {
    window.addEventListener('resize', this.onResize)
    this.windowWidth = window.innerWidth
  },
  beforeDestroy () {
    window.removeEventListener('resize', this.onResize)
  },
  methods: {
    triggerFileInput () {
      this.$refs.fileInput.click()
    },
    onFileSelected (event) {
      const file = event.target.files[0]
      if (file) {
        this.previewImage = URL.createObjectURL(file)
      }
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
      return item.name.toLowerCase().includes(search) ||
             item.id.toLowerCase().includes(search) ||
             item.category.toLowerCase().includes(search)
    },
    resetFilters () {
      this.filters = {
        category: '',
        supplier: '',
        minPrice: '',
        maxPrice: '',
        minStock: '',
        maxStock: ''
      }
    },
    applyFilters () {
      // In a real app, you would apply filters to the data source
      // For now, we'll just close the dialog
      this.showFilters = false
    },
    addProduct () {
      if (this.$refs.addProductForm.validate()) {
        // Format the new product data
        const product = {
          id: this.newProduct.sku || `PROD${Math.floor(Math.random() * 10000)}`,
          name: this.newProduct.name,
          supplierId: this.newProduct.supplierId,
          supplierVerified: false,
          category: this.newProduct.category,
          price: this.newProduct.purchasePrice ? `$${this.newProduct.purchasePrice}` : '$0.00',
          weight: this.newProduct.weight ? `${this.newProduct.weight} lb` : '0 lb',
          stockLevel: parseInt(this.newProduct.reorderLevel) || 0,
          maxStock: parseInt(this.newProduct.autoOrderLevel) || 0,
          image: this.previewImage || '/placeholder.png',
          dimensions: this.newProduct.dimensions,
          dimensionUnit: this.newProduct.dimensionUnit,
          warningLevel: this.newProduct.warningLevel,
          barcode: this.newProduct.barcode,
          gtin: this.newProduct.gtin,
          margin: this.newProduct.margin,
          description: this.newProduct.description
        }

        // Add to products array
        this.products.unshift(product)

        // Reset form and close dialog
        this.$refs.addProductForm.reset()
        this.previewImage = null
        this.showAddProduct = false

        // Reset newProduct object
        this.newProduct = {
          name: '',
          id: '',
          supplierId: '',
          category: 'Vapes',
          dimensionUnit: 'inch',
          dimensions: '',
          weight: '',
          reorderLevel: '',
          warningLevel: '',
          autoOrderLevel: '',
          sku: '',
          barcode: '',
          gtin: '',
          purchasePrice: '',
          margin: '',
          stockLevel: '',
          maxStock: '',
          description: ''
        }
      }
    },
    editProduct (product) {
      // In a real app, you would implement edit functionality
      console.log('Edit product:', product)
    },
    confirmDelete (product) {
      this.productToDelete = product
      this.showDeleteConfirm = true
    },
    deleteProduct () {
      if (this.productToDelete) {
        // Find and remove the product
        const index = this.products.findIndex(p =>
          p.id === this.productToDelete.id && p.name === this.productToDelete.name
        )
        if (index !== -1) {
          this.products.splice(index, 1)
        }

        // Close dialog and reset
        this.showDeleteConfirm = false
        this.productToDelete = null
      }
    },
    exportProducts () {
      // In a real app, you would implement export functionality
      console.log('Exporting products...')
      this.$nextTick(() => {
        // Using Vuetify's built-in snackbar would be better in a real app
        alert('Products exported successfully!')
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

/* Ajustes para pantallas pequeñas */
@media (max-width: 960px) {
  .toggle-btn {
    min-width: 36px !important;
    padding: 0 8px !important;
  }
}
</style>
