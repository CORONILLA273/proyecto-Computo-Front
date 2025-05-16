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
          :headers="headers"
          :items="products"
          :items-per-page="10"
          :search="searchQuery"
          :custom-filter="customFilter"
          show-select
          class="elevation-1 rounded-lg"
          hide-default-footer
          :footer-props="{
            'items-per-page-options': [5, 10, 15, 20],
            'prev-icon': 'mdi-arrow-left',
            'next-icon': 'mdi-arrow-right',
            'show-first-last-page': false
          }"
        >
          <template #[`item.image`]="{ item }">
            <v-avatar size="40">
              <v-img :src="item.image || '/placeholder.png'" :alt="item.productName" />
            </v-avatar>
          </template>

          <template #[`item.actions`]="{ item }">
            <v-icon
              small
              class="mr-2"
              color="#6941C6"
              @click="showEditProductDialog(true, item)"
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

          <!-- Personalización del footer de paginación -->
          <template slot="footer.page-text">
            <!-- Dejamos este slot vacío para eliminar el texto por defecto -->
          </template>

          <template #footer>
            <div class="custom-pagination d-flex align-center justify-space-between py-3 px-4">
              <v-btn
                text
                :disabled="page === 1"
                class="pagination-btn"
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
            <v-btn small outlined class="mr-2 rounded-lg" style="background-color: white;">
              <v-icon small left>
                mdi-plus-circle-outline
              </v-icon>
              Add Custom Field
            </v-btn>
            <v-btn small outlined class="mr-2 rounded-lg" style="background-color: white;">
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
                    v-model="productData.productName"
                    placeholder="Ex: Bloomlign"
                    outlined
                    dense
                    class="rounded-lg"
                    required
                  />
                </v-col>
                <v-col cols="12" md="4">
                  <label>Supplier ID</label>
                  <v-text-field
                    v-model="productData.supplierId"
                    placeholder="Ex: TUW10234"
                    outlined
                    dense
                    class="rounded-lg"
                    required
                  />
                </v-col>
                <v-col cols="12" md="4">
                  <label>Weight (in lbs)</label>
                  <v-text-field
                    v-model="productData.weight"
                    placeholder="Enter Weight here"
                    outlined
                    dense
                    class="rounded-lg"
                    required
                  />
                </v-col>

                <!-- Second row -->
                <v-col cols="12" md="4">
                  <label>Category</label>
                  <v-select
                    v-model="productData.category"
                    :items="categories"
                    item-text="label"
                    item-value="label"
                    outlined
                    dense
                    class="rounded-lg"
                    required
                  />
                </v-col>
                <v-col cols="12" md="4">
                  <label>Dimension Unit</label>
                  <v-select
                    v-model="productData.dimensionUnit"
                    :items="dimensionUnits"
                    outlined
                    dense
                    class="rounded-lg"
                    required
                  />
                </v-col>
                <v-col cols="12" md="4">
                  <label>Dimensions (L x W x H)</label>
                  <v-text-field
                    v-model="productData.dimensions"
                    placeholder="20 × 30 × 40"
                    outlined
                    dense
                    class="rounded-lg"
                    required
                  />
                </v-col>

                <!-- Third row -->
                <v-col cols="12" md="4">
                  <label>Recorded Stock Level</label>
                  <v-text-field
                    v-model="productData.recordedStockLevel"
                    placeholder="Ex: 2000"
                    outlined
                    dense
                    class="rounded-lg"
                    type="number"
                    required
                  />
                </v-col>
                <v-col cols="12" md="4">
                  <label>Warning Threshold Stock Level</label>
                  <v-text-field
                    v-model="productData.warningThresholdStockLevel"
                    placeholder="Ex: 100"
                    outlined
                    dense
                    class="rounded-lg"
                    type="number"
                    required
                  />
                </v-col>
                <v-col cols="12" md="4">
                  <label>Auto Order Stock Level</label>
                  <v-text-field
                    v-model="productData.autoOrderStockLevel"
                    placeholder="Ex: 50"
                    outlined
                    dense
                    class="rounded-lg"
                    type="number"
                    required
                  />
                </v-col>

                <!-- Fourth row -->
                <v-col cols="12" md="4">
                  <label>SKU Code</label>
                  <v-text-field
                    v-model="productData.skuCode"
                    placeholder="RTY1234455"
                    outlined
                    dense
                    class="rounded-lg"
                    required
                  />
                </v-col>
                <v-col cols="12" md="4">
                  <label>Barcode Number</label>
                  <v-text-field
                    v-model="productData.barcodeNumber"
                    placeholder="QWERTY5987"
                    outlined
                    dense
                    class="rounded-lg"
                    required
                  />
                </v-col>
                <v-col cols="12" md="4">
                  <label>GRN Number (Optional)</label>
                  <v-text-field
                    v-model="productData.grnNumber"
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
                        v-model="productData.purchasingPrice"
                        placeholder="Ex: $100"
                        outlined
                        dense
                        class="rounded-lg"
                        required
                      />
                    </v-col>
                    <v-col cols="12" md="6">
                      <label>Selling Price Margin</label>
                      <v-text-field
                        v-model="productData.sellingPriceMargin"
                        placeholder="Ex: 20%"
                        outlined
                        dense
                        class="rounded-lg"
                        required
                      />
                    </v-col>

                    <!-- Sixth row -->
                    <v-col cols="12">
                      <label>Product Description</label>
                      <v-textarea
                        v-model="productData.productDescription"
                        label="Product Description"
                        placeholder="Ex: Type something about product here"
                        outlined
                        rows="3"
                        class="rounded-lg"
                        required
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

    <!-- edit product Dialog-->
    <v-dialog
      v-model="showEditProduct"
      max-width="800px"
    >
      <v-card class="rounded-lg">
        <v-card-title class="d-flex align-center justify-space-between" style="background-color: gainsboro;">
          <span>Edit product</span>
          <div>
            <v-btn small outlined class="mr-2 rounded-lg" style="background-color: white;">
              <v-icon small left>
                mdi-plus-circle-outline
              </v-icon>
              Add Custom Field
            </v-btn>
            <v-btn small outlined class="mr-2 rounded-lg" style="background-color: white;">
              <v-icon small left>
                mdi-tray-arrow-up
              </v-icon>
              Bulk Upload
            </v-btn>
            <v-btn icon @click="showEditProduct = false">
              <v-icon>mdi-close-circle-outline</v-icon>
            </v-btn>
          </div>
        </v-card-title>
        <v-card-text>
          <v-container>
            <v-form ref="editProductForm" v-model="validEditForm">
              <v-row>
                <!-- First row -->
                <v-col cols="12" md="4">
                  <label>Product Name</label>
                  <v-text-field
                    v-model="productData.productName"
                    placeholder="Ex: Bloomlign"
                    outlined
                    dense
                    class="rounded-lg"
                    required
                  />
                </v-col>
                <v-col cols="12" md="4">
                  <label>Supplier ID</label>
                  <v-text-field
                    v-model="productData.supplierId"
                    placeholder="Ex: TUW10234"
                    outlined
                    dense
                    class="rounded-lg"
                    required
                  />
                </v-col>
                <v-col cols="12" md="4">
                  <label>Weight (in lbs)</label>
                  <v-text-field
                    v-model="productData.weight"
                    placeholder="Enter Weight here"
                    outlined
                    dense
                    class="rounded-lg"
                    required
                  />
                </v-col>

                <!-- Second row -->
                <v-col cols="12" md="4">
                  <label>Category</label>
                  <v-select
                    v-model="productData.category"
                    :items="categories"
                    item-text="label"
                    item-value="label"
                    outlined
                    dense
                    class="rounded-lg"
                    required
                  />
                </v-col>
                <v-col cols="12" md="4">
                  <label>Dimension Unit</label>
                  <v-select
                    v-model="productData.dimensionUnit"
                    :items="dimensionUnits"
                    outlined
                    dense
                    class="rounded-lg"
                    required
                  />
                </v-col>
                <v-col cols="12" md="4">
                  <label>Dimensions (L x W x H)</label>
                  <v-text-field
                    v-model="productData.dimensions"
                    placeholder="20 × 30 × 40"
                    outlined
                    dense
                    class="rounded-lg"
                    required
                  />
                </v-col>

                <!-- Third row -->
                <v-col cols="12" md="4">
                  <label>Recorded Stock Level</label>
                  <v-text-field
                    v-model="productData.recordedStockLevel"
                    placeholder="Ex: 2000"
                    outlined
                    dense
                    class="rounded-lg"
                    type="number"
                    required
                  />
                </v-col>
                <v-col cols="12" md="4">
                  <label>Warning Threshold Stock Level</label>
                  <v-text-field
                    v-model="productData.warningThresholdStockLevel"
                    placeholder="Ex: 100"
                    outlined
                    dense
                    class="rounded-lg"
                    type="number"
                    required
                  />
                </v-col>
                <v-col cols="12" md="4">
                  <label>Auto Order Stock Level</label>
                  <v-text-field
                    v-model="productData.autoOrderStockLevel"
                    placeholder="Ex: 50"
                    outlined
                    dense
                    class="rounded-lg"
                    type="number"
                    required
                  />
                </v-col>

                <!-- Fourth row -->
                <v-col cols="12" md="4">
                  <label>SKU Code</label>
                  <v-text-field
                    v-model="productData.skuCode"
                    placeholder="RTY1234455"
                    outlined
                    dense
                    class="rounded-lg"
                    required
                  />
                </v-col>
                <v-col cols="12" md="4">
                  <label>Barcode Number</label>
                  <v-text-field
                    v-model="productData.barcodeNumber"
                    placeholder="QWERTY5987"
                    outlined
                    dense
                    class="rounded-lg"
                    required
                  />
                </v-col>
                <v-col cols="12" md="4">
                  <label>GRN Number (Optional)</label>
                  <v-text-field
                    v-model="productData.grnNumber"
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
                      <div v-if="!editPreviewImage && !productData.image" class="upload-placeholder">
                        <v-icon size="40" color="#E4E7EC">
                          mdi-plus
                        </v-icon>
                      </div>
                      <v-img
                        v-else
                        :src="editPreviewImage || productData.image"
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
                        v-model="productData.purchasingPrice"
                        placeholder="Ex: $100"
                        outlined
                        dense
                        class="rounded-lg"
                        required
                      />
                    </v-col>
                    <v-col cols="12" md="6">
                      <label>Selling Price Margin</label>
                      <v-text-field
                        v-model="productData.sellingPriceMargin"
                        placeholder="Ex: 20%"
                        outlined
                        dense
                        class="rounded-lg"
                        required
                      />
                    </v-col>

                    <!-- Sixth row -->
                    <v-col cols="12">
                      <label>Product Description</label>
                      <v-textarea
                        v-model="productData.productDescription"
                        label="Product Description"
                        placeholder="Ex: Type something about product here"
                        outlined
                        rows="3"
                        class="rounded-lg"
                        required
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
            @click="editProduct"
          >
            Update Product
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
          Are you sure you want to delete "{{ productToDelete?.productName }}"? This action cannot be undone.
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
      products: [],
      searchQuery: '',
      itemsPerPage: 10,
      page: 1,
      totalPages: 10, // Total de páginas (puedes calcularlo basado en tus datos)
      maxVisiblePages: 7,
      showFilters: false,
      showAddProduct: false,
      showEditProduct: false,
      showDeleteConfirm: false,
      productToDelete: null,
      validForm: false,
      validEditForm: false,
      windowWidth: 0,
      selectedTimeframe: '1m', // Default selected timeframe
      dateMenu: false, // Controls the date picker menu
      dateRange: [], // Stores the selected date range
      previewImage: null, // For image preview
      editPreviewImage: null,
      dimensionUnits: ['inch', 'cm', 'mm', 'ft'],
      filters: {
        category: '',
        supplier: '',
        minPrice: '',
        maxPrice: '',
        minStock: '',
        maxStock: ''
      },
      productData: {
        productName: '',
        supplierId: '',
        weight: '',
        category: 'Vapes',
        dimensionUnit: 'inch',
        dimensions: '',
        recordedStockLevel: '',
        warningThresholdStockLevel: '',
        autoOrderStockLevel: '',
        skuCode: '',
        barcodeNumber: '',
        grnNumber: '',
        image: null,
        purchasingPrice: '',
        sellingPriceMargin: '',
        productDescription: ''
      },
      headers: [
        {
          text: 'Product Name',
          align: 'start',
          value: 'productName',
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
          value: 'purchasingPrice',
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
          value: 'warningThresholdStockLevel',
          align: 'end',
          width: '10%',
          breakpoint: 'md'
        },
        {
          text: 'Max Stock',
          value: 'recordedStockLevel',
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
    }
  },
  watch: {
    dateRange (val) {
      // Aquí puedes implementar lógica para filtrar datos por fecha
      console.log('Date range changed:', val)
    }
  },
  mounted () {
    this.loadProducts()
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
      return item.productName.toLowerCase().includes(search) ||
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
    async addProduct () {
      try {
        await this.$axios.post('/products/addProduct', this.productData)
        /* this.$store.dispatch('alert/triggerAlert', {
          message: 'Usuario Creado Con Éxito',
          type: 'success'
        }) */
        this.loadProducts()
        this.showAddProduct = false
      } catch (error) {
        const errorMessage = error.message || 'Error al crear el usuario'
        console.log('error al Crear Usuario: ', errorMessage)

        /* this.$store.dispatch('alert/triggerAlert', {
          message: errorMessage,
          type: 'error'
        }) */
      }
    },
    showEditProductDialog (mode = true, product = null) {
      this.editPreviewImage = product.image || null
      this.showEditProduct = mode
      this.productData = { ...product }
    },
    async editProduct () {
      try {
        await this.$axios.put(`/products/updateProduct/${this.productData.id}`, this.productData)
        /* this.$store.dispatch('alert/triggerAlert', {
          message: 'Usuario Actualizado Con Éxito',
          type: 'success'
        }) */
        this.loadProducts()
        this.showEditProduct = false
      } catch (error) {
        const errorMessage = error.message || 'Error al actualizar el usuario'
        console.log('error al Editar Usuario: ', errorMessage)
        /* this.$store.dispatch('alert/triggerAlert', {
          message: errorMessage,
          type: 'error'
        }) */
      }
    },
    confirmDelete (product) {
      this.productToDelete = product
      this.showDeleteConfirm = true
    },
    async deleteProduct () {
      if (this.productToDelete) {
        try {
          await this.$axios.delete(`/products/deleteProduct/${this.productToDelete.id}`)
          /* this.$store.dispatch('alert/triggerAlert', {
            message: 'Usuario Eliminado Con Éxito',
            type: 'success'
          }) */
          this.loadProducts()
        } catch (error) {
          const errorMessage = error.message || 'Error al eliminar el usuario'
          console.log('error al Editar Usuario: ', errorMessage)

          /* this.$store.dispatch('alert/triggerAlert', {
            message: errorMessage,
            type: 'error'
          }) */
        }
      }
      this.confirmDialog = false
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
</style>
