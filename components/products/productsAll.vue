<template>
  <v-container>
    <v-row align="center">
      <v-btn icon @click="$router.push('/overview')">
        <v-icon color="#6941C6">
          mdi-arrow-left
        </v-icon>
        <v-text class="btnBackText">
          Back
        </v-text>
      </v-btn>
    </v-row>
    <v-row align="center" justify="end">
      <div>
        <span class="title" style="font-size: 400px;">
          Products
        </span>
      </div>
      <v-spacer />
      <v-btn-toggle
        v-model="selectedPeriod"
        mandatory
        color="#6941C6"
        class="period-selector"
      >
        <v-btn value="1d">
          1d
        </v-btn>
        <v-btn value="7d">
          7d
        </v-btn>
        <v-btn value="1m">
          1m
        </v-btn>
        <v-btn value="3m">
          3m
        </v-btn>
        <v-btn value="6m">
          6m
        </v-btn>
        <v-btn value="1y">
          1y
        </v-btn>
        <v-btn value="3y">
          3y
        </v-btn>
        <v-btn value="5y">
          5y
        </v-btn>
      </v-btn-toggle>
      <v-btn class="btnCalendar" style="background-color: white;">
        <v-icon color="">
          mdi-calendar-blank-outline
        </v-icon>
        <span>
          Select Dates
        </span>
      </v-btn>
    </v-row>
    <v-row align="center" justify="end">
      <v-data-table class="mt-4" />
    </v-row>
    <v-dialog v-model="dialogAddPacient" persistent max-width="800px">
      <v-card color="indigo lighten-5" elevation="0">
        <v-card-title>
          Add Patient
          <v-spacer />
          <v-btn icon class="mr-2" color="grey darken-1" @click="dialogAddPacient = false">
            <v-icon>
              mdi-close
            </v-icon>
          </v-btn>
        </v-card-title>
        <v-card-text>
          <v-tabs v-model="activeTab">
            <v-tab>Demographics</v-tab>
            <v-tab>Adress</v-tab>
            <v-tab>Photo</v-tab>
          </v-tabs>

          <v-tabs-items v-model="activeTab">
            <!-- Tab Demographics -->
            <v-tab-item>
              <v-form style="margin: 10px !important;">
                <v-row>
                  <v-col cols="12">
                    <v-text-field v-model="paciente.fullName" label="Full Name" required outlined />
                  </v-col>
                </v-row>
                <v-row>
                  <v-col cols="4">
                    <v-text-field v-model="paciente.email" label="Email" required outlined />
                  </v-col>
                  <v-col cols="4">
                    <v-text-field v-model="paciente.mobile" label="Mobile" required outlined />
                  </v-col>
                  <v-col cols="4">
                    <v-menu ref="menu" :close-on-content-click="false" transition="scale-transition" offset-y min-width="auto">
                      <template #activator="{ on, attrs }">
                        <v-text-field
                          v-model="paciente.dateBirth"
                          label="Date of Birth"
                          prepend-inner-icon="mdi-calendar"
                          readonly
                          outlined
                          v-bind="attrs"
                          v-on="on"
                        />
                      </template>
                      <v-date-picker v-model="paciente.dateBirth" @input="menu = false" />
                    </v-menu>
                  </v-col>
                </v-row>
                <v-row>
                  <v-col cols="6">
                    <v-text-field v-model="paciente.motherTongue" label="Mother Tongue" required outlined />
                  </v-col>
                  <v-col cols="6">
                    <v-text-field v-model="paciente.govId" label="Goberment ID" required outlined />
                  </v-col>
                </v-row>
                <v-row>
                  <v-col cols="6">
                    <v-text-field v-model="paciente.sex" label="sex" required outlined />
                  </v-col>
                  <v-col cols="6">
                    <v-text-field v-model="paciente.maritalStatus" label="Marital Status" required outlined />
                  </v-col>
                </v-row>
              </v-form>
            </v-tab-item>

            <!-- Address -->
            <v-tab-item>
              <v-form style="margin: 10px !important;">
                <v-row>
                  <v-col cols="12">
                    <v-text-field v-model="paciente.address" label="Address" required outlined />
                  </v-col>
                </v-row>
                <v-row>
                  <v-col cols="4">
                    <v-text-field v-model="paciente.city" label="City" required outlined />
                  </v-col>
                  <v-col cols="4">
                    <v-text-field v-model="paciente.zipCode" label="Zip Code" required outlined />
                  </v-col>
                  <v-col cols="4">
                    <v-text-field v-model="paciente.state" label="State" required outlined />
                  </v-col>
                </v-row>
                <v-row>
                  <v-col cols="6">
                    <v-text-field v-model="paciente.country" label="Country" required outlined />
                  </v-col>
                  <v-col cols="6">
                    <v-text-field v-model="paciente.district" label="District" required outlined />
                  </v-col>
                </v-row>
              </v-form>
            </v-tab-item>

            <!-- Photo -->
            <v-tab-item>
              <v-form style="margin: 10px !important;">
                <v-row>
                  <v-col cols="12">
                    <v-file-input
                      v-model="photoFile"
                      label="Upload Photo"
                      accept="image/*"
                      outlined
                      prepend-inner-icon="mdi-camera"
                      @change="handleFileUpload"
                    />
                  </v-col>
                </v-row>
                <v-row>
                  <v-col cols="12">
                    <v-img v-if="paciente.photo" :src="'data:image/jpeg;base64,' + paciente.photo" max-width="200" max-height="200" class="mt-2" />
                  </v-col>
                </v-row>
              </v-form>
            </v-tab-item>
          </v-tabs-items>
        </v-card-text>
        <v-card-actions>
          <v-spacer />
          <v-btn class="btnPatient" @click="savePatient">
            <span>Add Patient</span>
          </v-btn>
        </v-card-actions>
      </v-card>
    </v-dialog>
  </v-container>
</template>

<script>
export default {
  data () {
    return {
      pacientes: [],
      dialogAddPacient: false,
      paciente: {
        fullName: '',
        email: '',
        mobile: '',
        dateBirth: '',
        motherTongue: '',
        govId: '',
        sex: '',
        maritalStatus: '',
        address: '',
        city: '',
        zipCode: '',
        state: '',
        country: '',
        district: '',
        international: 'false',
        photo: ''
      },
      photoFile: null,
      activeTab: 0,
      menu: false
    }
  },
  mounted () {
    this.loadPacientes()
  },
  methods: {
    handleFileUpload (event) {
      const file = event
      const reader = new FileReader()
      reader.onload = (e) => {
        this.paciente.photo = e.target.result.split(',')[1]
      }
      // console.log('@@@ file => ', file, this.paciente.photo)
      reader.readAsDataURL(file)
    },
    async loadPacientes () {
      try {
        const response = await this.$axios.get('/pacientes')
        console.log('@@@ response => ', response)
      } catch (error) {
        console.log(error)
      }
    },
    async savePatient () {
      try {
        await this.$axios.post('/pacientes/create', this.paciente)
        this.$store.dispatch('alert/triggerAlert', {
          message: 'Paciente Guardado Satisfactoriamente',
          type: 'success'
        })
        this.dialogAddPacient = false
        this.loadPacientes()
      } catch (error) {
        const errorMessage = error.message || 'Error al registrar los usuarios'
        this.$store.dispatch('alert/triggerAlert', {
          message: errorMessage,
          type: 'error'
        })
      }
    }
  }
}

</script>

<style scoped>
.btnBack {
  color: #6941C6;
}

.btnBackText {
  font-family: Inter;
  font-weight: 500;
  font-size: 14px;
  line-height: 20px;
  letter-spacing: 0%;
  color: #6941C6;
  text-transform: capitalize;

}
.title{
  font-family: Inter;
  font-weight: 500;
  line-height: 38px;
  letter-spacing: 0%;
  color: #101828;
}

.period-selector {
  border-radius: 4px;
  box-shadow: 0px 1px 2px 0px #1018280D;
  background-color: white;
}

.period-selector .v-btn {
  min-width: 40px !important; /* Ancho mínimo para cada botón */
  padding: 0 8px !important;
  font-size: 12px;
}

.btnCalendar {
  border-radius: 4px;
  border: #101828;
  padding: 5px 10px !important;
  font-size: 14px !important;
  text-transform: none !important;
  margin: 2;
  font-family: Inter;
  font-weight: 500;
  font-size: 14px;
  line-height: 20px;
  letter-spacing: 0%;

}

.btnFilter {
  color: white;
  color: #101828 !important;
  margin-right: 10px !important;
  border: 1px solid #101828 !important;
  border-radius: 5px !important;
  padding: 5px 10px !important;
  font-size: 14px !important;
  text-transform: none !important;
}

.btnPatient {
  text-transform: none !important;
  color: white;
  border-radius: 5px !important;
  padding: 5px 10px !important;
  background-color: #0e1680 !important;
  font-size: 14px !important;
  margin-right: 10px !important;
  font-weight: 500 !important;
}

</style>
