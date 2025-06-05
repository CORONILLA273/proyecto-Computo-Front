<template>
  <v-app>
    <!-- Header -->
    <v-app-bar app flat color="#ffffff" height="80">
      <v-container class="d-flex align-center justify-space-between pa-0">
        <div class="logo-section">
          <img src="/logo.png" alt="Logo de UnityWare" class="logo">
        </div>
        <v-btn
          rounded
          large
          color="#6941C6"
          dark
          class="sign-in-btn"
          elevation="0"
          @click="goToLogin"
        >
          Sign in
          <v-icon right>mdi-arrow-right</v-icon>
        </v-btn>
      </v-container>
    </v-app-bar>

    <!-- Main Content -->
    <v-main>
      <!-- Hero Section with Wave Background -->
      <section class="hero-section">
        <div>
          <img src="/rainbow_waves.png" alt="Wave Background" class="wave-background">
        </div>
        <v-container class="hero-content">
          <v-row align="center" justify="center">
            <v-col cols="12" class="text-center">
              <h1 class="hero-title">
                Technology doesn't have to feel like a different language
              </h1>
              <p class="hero-subtitle mt-8">
                Simplified inventory management to drive business growth and strategically scale operations
              </p>
            </v-col>
          </v-row>
        </v-container>
      </section>

      <!-- Features and Dashboard Section -->
      <section class="features-section">
        <v-container>
          <v-row>
            <!-- Features Column -->
            <v-col cols="12" md="8">
              <v-row class="mt-6">
                <!-- Order Management -->
                <v-col cols="12" sm="1" class="mb-8 mt-8">
                  <div>
                    <v-img src="/order.png" width="50"/>
                  </div>
                </v-col>
                <v-col cols="12" sm="5" class="mb-8">
                  <div>
                    <h3 class="feature-title mb-3" style="color: #6941C6;">Order Management</h3>
                    <p class="feature-description font-weight-bold">
                      Efficiently manage all your sales and purchasing activities with our platform. Handle invoices and bills seamlessly while tracking payments with ease.
                    </p>
                  </div>
                </v-col>

                <!-- Warehouse Management -->
                <v-col cols="12" sm="1" class="mb-8 mt-8">
                  <div>
                    <v-img src="/list.png" width="50"/>
                  </div>
                </v-col>
                <v-col cols="12" sm="5" class="mb-8">
                  <div>
                    <h3 class="feature-title mb-3" style="color: #6941C6;">Warehouse Management</h3>
                    <p class="feature-description font-weight-bold">
                      Centrally manage your stock across multiple warehouses and generate detailed reports specific to each warehouse.
                    </p>
                  </div>
                </v-col>

                <!-- Inventory Tracking -->
                <v-col cols="12" sm="1" class="mb-8 mt-8">
                  <div>
                    <v-img src="/tracking.png" width="50"/>
                  </div>
                </v-col>
                <v-col cols="12" sm="5" class="mb-8">
                  <div>
                    <h3 class="feature-title mb-3" style="color: #6941C6;">Inventory Tracking</h3>
                    <p class="feature-description font-weight-bold">
                      Real-time tracking of your inventory levels with automated alerts and comprehensive monitoring capabilities.
                    </p>
                  </div>
                </v-col>

                <!-- Reports & Analytics -->
                <v-col cols="12" sm="1" class="mb-8 mt-8">
                  <div>
                    <v-img src="/reports.png" width="50"/>
                  </div>
                </v-col>
                <v-col cols="12" sm="5" class="mb-8">
                  <div>
                    <h3 class="feature-title mb-3" style="color: #6941C6;">Reports & Analytics</h3>
                    <p class="feature-description font-weight-bold">
                      Comprehensive analytics and reporting tools to help you make data-driven decisions for your business.
                    </p>
                  </div>
                </v-col>
              </v-row>
            </v-col>
            <v-col cols="12" md="4 ">
              <v-img src="/dashboard-mockup.png" width="100%" />
            </v-col>
          </v-row>
        </v-container>
      </section>
    </v-main>
    <!-- Footer -->
    <v-footer color="transparent" class="footer-section">
      <v-container>
        <div class="footer-content">
          <span class="footer-text">© TheUnityWare 2024</span>
        </div>
      </v-container>
    </v-footer>
  </v-app>
</template>

<script>
export default {
  name: 'TheUnityWareLanding',
  layout: 'blank',
  data () {
    return {
    }
  },
  methods: {
    goToLogin () {
      this.$router.push('/login')
    }
  }
}
</script>

<style scoped>
/* Estilos del logo */
.logo-container {
  display: flex;
  justify-content: center;
  align-items: center;
  padding: 1rem;
}
.logo {
  max-width: 300px;
  height: auto;
}

/* Header Styles */
.sign-in-btn {
  padding: 0 24px !important;
  height: 48px !important;
  font-weight: 500;
  text-transform: none;
  letter-spacing: 0;
}

/* Hero Section */
.hero-section {
  position: relative;
  min-height: 400px;
  display: flex;
  align-items: center;
  overflow: hidden;
}

.wave-background {
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background-size: cover;
  z-index: 1;
}

.hero-content {
  position: relative;
  z-index: 2;
  padding: 80px 0;
}

.hero-title {
  font-size: 3.5rem;
  font-weight: 500;
  line-height: 1.2;
  max-width: 80vw;
  margin: 0 auto;
  background: linear-gradient(90deg, #384B97, #712020);
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
}

.hero-subtitle {
  font-size: 1.25rem;
  font-weight: 700;
  color: #000;
  max-width: 80vw;
  margin: 0 auto;
  line-height: 1.6;
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
}

/* Features Section */
.features-section {
  padding: 80px 0;
  background: white;
}

.feature-title {
  font-size: 1.25rem;
  font-weight: 600;
  color: #1e293b;
  margin: 0;
}

.feature-description {
  font-size: 0.95rem;
  color: #64748b;
  line-height: 1.6;
  margin: 0;
}

/* Dashboard Mockup */
.dashboard-mockup {
  padding: 20px;
}

.dashboard-card {
  border-radius: 12px !important;
  overflow: hidden;
}

/* Footer */
.footer-section {
  padding: 40px 0;
  border-top: 1px solid #e2e8f0;
}

.footer-content {
  text-align: center;
}

.footer-text {
  font-size: 14px;
  color: #7C3AED;
}

/* Responsive */
@media (max-width: 960px) {
  .hero-title {
    font-size: 2.5rem;
  }

  .dashboard-layout {
    flex-direction: column;
  }

  .dashboard-sidebar {
    width: 100%;
  }

  .metrics-row {
    flex-direction: column;
  }
}

@media (max-width: 600px) {
  .hero-title {
    white-space: normal;
    text-overflow: unset;
    font-size: 2rem;
  }
  .hero-subtitle {
    font-size: 1.1rem;
  }
}
</style>
