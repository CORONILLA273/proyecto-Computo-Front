<template>
  <v-container fluid class="register-page pa-0 fill-height">
    <v-row no-gutters class="fill-height">
      <!-- IZQUIERDA: Presentación -->
      <v-col
        cols="12"
        md="6"
        class="left-panel d-flex flex-column justify-start pa-10"
      >
        <img src="/logo_inverted.png" alt="Logo" class="mb-10" width="180" />
        <div class="text-left">
          <h1 class="display-1 font-weight-bold mb-4">
            Engineered to handle all<br />your inventory needs
          </h1>
          <p class="subtitle-1 mb-10">
            Your complete inventory management software to track inventory,
            streamline sales, fulfill orders, and oversee warehouses from a
            single window.
          </p>
          <v-form ref="form" class="form-container text-left">
            <h2 class="mb-6">What about yourself</h2>
            <span class="mt-6 font-weight-medium">FULL NAME*</span>
            <v-text-field
              v-model="form.fullName"
              prepend-inner-icon="mdi-account-outline"
              outlined
              dense
              flat
              background-color="white"
              color="deep-purple accent-4"
            />
            <span class="mt-6 font-weight-medium">E-MAIL*</span>
            <v-text-field
              v-model="form.email"
              prepend-inner-icon="mdi-at"
              outlined
              dense
              background-color="white"
              color="deep-purple accent-4"
            />
            <span class="mt-6 font-weight-medium">PASSWORD*</span>
            <v-text-field
              v-model="form.password"
              type="password"
              prepend-inner-icon="mdi-lock-outline"
              outlined
              dense
              background-color="white"
              color="deep-purple accent-4"
            />
          </v-form>
        </div>
        <p class="text-caption mt-auto white--text">© TheUnityWare 2024</p>
      </v-col>

      <!-- DERECHA: Formulario -->
      <v-col
        cols="12"
        md="6"
        class="pa-10 d-flex align-center"
        style="background-color: #f9fafb"
      >
        <v-form ref="form" class="form-container text-left">
          <h2 class="mb-6">Tell us a little about your business</h2>
          <span class="mt-6 font-weight-medium">BUSINESS NAME*</span>
          <v-text-field
            label="Business Name*"
            v-model="form.businessName"
            outlined
            dense
          />
          <span class="mt-6 font-weight-medium">INDUSTRY*</span>
          <v-text-field
            label="Industry*"
            v-model="form.industry"
            prepend-inner-icon="mdi-magnify"
            outlined
            dense
          />
          <span class="mt-6 font-weight-medium">DOMAIN*</span>
          <v-text-field
            label="Domain*"
            v-model="form.domain"
            prepend-inner-icon="mdi-magnify"
            outlined
            dense
          />
          <span class="mt-6 font-weight-medium">PRODUCT/SERVICES OFFERED*</span>
          <v-text-field
            label="Product/Services Offered*"
            v-model="form.product"
            prepend-inner-icon="mdi-magnify"
            outlined
            dense
          />
          <p class="mt-6 font-weight-medium">Which of the following best describes you*</p>
          <v-radio-group v-model="form.type" row>
            <v-radio label="Super-stockiest" value="Super-stockiest" />
            <v-radio label="Distributor" value="Distributor" />
            <v-radio label="Retailer" value="Retailer" />
            <v-radio label="Brand" value="Brand" />
          </v-radio-group>

          <p class="mt-6 font-weight-medium">Storage Keeping Unit (SKU) Size*</p>
          <v-radio-group v-model="form.storage_size" row>
            <v-radio label="<500 units" value="<500" />
            <v-radio label="501-1000 units" value="501-1000" />
            <v-radio label="1001-5000 units" value="1001-5000" />
            <v-radio label="5001-10000 units" value="5001-10000" />
            <v-radio label="10001-25000 units" value="10001-25000" />
            <v-radio label=">25000 units" value=">25000" />
          </v-radio-group>

          <v-btn color="deep-purple accent-4" dark class="mt-6" large @click="register">
            Get Started
            <v-icon right>mdi-arrow-right</v-icon>
          </v-btn>
        </v-form>
      </v-col>
    </v-row>
  </v-container>
</template>

<script>
export default {
  layout: 'blank',
  data () {
    return {
      form: {
        fullName: '',
        email: '',
        password: '',
        businessName: '',
        industry: '',
        domain: '',
        product: '',
        type: '',
        storage_size: ''
      }
    }
  },
  methods: {
    async register () {
      try {
        const response = await this.$axios.post('/users/create', this.form)
        console.log(response)
        this.$router.push('/login')
      } catch (error) {
        console.error(error)
      }
    }
  }
}
</script>

<style scoped>
.register-page {
  height: 100vh; /* Ocupa el alto completo */
  overflow: hidden;
}

.left-panel {
  background-color: #7c3aed;
  color: white;
  border-top-right-radius: 20px;
  border-bottom-right-radius: 20px;
}

.form-container {
  max-width: 600px;
  width: 100%;
}
</style>
