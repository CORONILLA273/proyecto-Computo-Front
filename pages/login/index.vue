<template>
  <v-container fluid class="login-page pa-0 fill-height">
    <v-row no-gutters class="fill-height">
      <!-- IZQUIERDA: Presentación -->
      <v-col
        cols="12"
        md="6"
        class="left-panel"
      >
        <div class="text-center">
          <div class=" mb-8">
            <div class="logo-badge">
              <img src="/logo.png" alt="Logo" class="logo-img">
            </div>
          </div>
          <p class="custom-subtitle text-center px-8">
            Re-imagining inventory and management experience with advance
            data analytics for optimum performance
          </p>
        </div>
        <p class="text-caption footer-text">
          © TheUnityWare 2024
        </p>
      </v-col>

      <!-- DERECHA: Formulario de Login -->
      <v-col
        cols="12"
        md="6"
        class="pa-10 d-flex align-center justify-center"
        style="background-color: #f9fafb"
      >
        <div class="form-container">
          <h2 class="text-h4 font-weight-bold mb-2">
            Welcome back
          </h2>
          <p class="text-body-2 text--secondary mb-8">
            Welcome back! Please enter your details.
          </p>

          <v-form ref="form" @submit.prevent="login">
            <div class="mb-4">
              <label class="text-body-2 font-weight-medium mb-2 d-block">Email</label>
              <v-text-field
                v-model="user.email"
                placeholder="Enter your email"
                outlined
                dense
                hide-details="auto"
                class="mb-4"
              />
            </div>

            <div class="mb-4">
              <label class="text-body-2 font-weight-medium mb-2 d-block">Password</label>
              <v-text-field
                v-model="user.password"
                type="password"
                placeholder="••••••••"
                outlined
                dense
                hide-details="auto"
                class="mb-4"
              />
            </div>

            <div class="d-flex justify-space-between align-center mb-6">
              <v-checkbox
                v-model="user.rememberMe"
                label="Remember for 30 days"
                dense
                hide-details
                class="ma-0 pa-0"
              />
              <a href="#" class="text-decoration-none" style="color: #7c3aed;">Forgot password</a>
            </div>

            <v-btn
              color="#7c3aed"
              dark
              large
              block
              class="mb-4 text-capitalize"
              @click="login"
            >
              Sign in
            </v-btn>

            <v-btn
              outlined
              large
              block
              class="mb-6 text-capitalize"
              @click="signInWithGoogle"
            >
              <v-icon left small>
                mdi-google
              </v-icon>
              Sign in with Google
            </v-btn>

            <div class="text-center">
              <span class="text-body-2 text--secondary">Don't have an account? </span>
              <a class="text-decoration-none" style="color: #7c3aed;" @click="goToSignUp">Sign up</a>
            </div>
          </v-form>
        </div>
      </v-col>
    </v-row>
  </v-container>
</template>

<script>
export default {
  layout: 'blank',
  data () {
    return {
      user: {
        email: '',
        password: '',
        rememberMe: false
      }
    }
  },
  mounted () {
    if (this.$auth.loggedIn) {
      console.log('Usuario ya logueado, redirigiendo a /employees')
      this.$router.push('/employees')
    }
  },
  methods: {
    async login () {
      try {
        console.log('Estrategia:', this.$auth.strategies.local.options)
        await this.$auth.loginWith('local', {
          data: this.user
        })

        if (this.$auth.loggedIn) {
          this.$router.push('/employees')
        }
      } catch (error) {
        console.log('@@@ error => ', error)
      }
    },
    signInWithGoogle () {
      // Lógica para login con Google
      console.log('Sign in with Google')
    },
    goToSignUp () {
      // Navegar a página de registro
      this.$router.push('/register')
    }
  }
}
</script>

<style scoped>
.login-page {
  height: 100vh;
  overflow: hidden;
}

.left-panel {
  background-color: #7c3aed;
  position: relative;
  min-height: 300px;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  border-top-right-radius: 20px;
  border-bottom-right-radius: 20px;
}

.logo-img {
  display: block;
  max-width: 100%;
  height: auto;
}

.logo-badge {
  background:white;
  backdrop-filter: blur(10px);
  border: 1px solid white;
  border-radius: 50px;
  padding: 4px;
  display: inline-block;
}

.custom-subtitle {
  font-size: 0.95rem;
  color: white;
}

.form-container {
  max-width: 400px;
  width: 100%;
}

.footer-text {
  position: absolute;
  bottom: 8px;
  left: 16px;
  color: white;
}

/* Estilos personalizados para los campos */
.v-text-field--outlined >>> .v-input__control .v-input__slot {
  min-height: 48px;
}

.v-text-field--outlined >>> fieldset {
  border-color: #e5e7eb;
}

.v-text-field--outlined:not(.v-input--is-focused):not(.v-input--has-state) >>> fieldset {
  border-color: #e5e7eb;
}

.v-text-field--outlined.v-input--is-focused >>> fieldset {
  border-color: #7c3aed;
  border-width: 2px;
}

/* Estilos para el checkbox */
.v-input--checkbox .v-input__control .v-input__slot {
  margin-bottom: 0;
}

/* Responsive */
@media (max-width: 960px) {
  .left-panel {
    order: 2;
    min-height: 300px;
  }

  .form-container {
    order: 1;
  }
}
</style>
