<template>
  <v-app>
    <v-navigation-drawer
      absolute
      permanent
      left
      width="250px"
      class="sidebar"
    >
      <div class="logo-container">
        <img src="/logo.png" alt="Logo de UnityWare" class="logo">
      </div>
      <v-text-field
        dense
        solo-inverted
        flat
        hide-details
        placeholder="Search"
        class="search-field"
        prepend-inner-icon="mdi-magnify"
      />
      <v-list dense>
        <v-divider class="my-2" />
        <template v-for="(item, index) in items">
          <v-list-item
            v-if="!item.children"
            :key="`item-${index}`"
            link
            @click="goTo(item.path)"
          >
            <v-list-item-icon>
              <v-icon color="#667085">
                {{ item.icon }}
              </v-icon>
            </v-list-item-icon>
            <v-list-item-content>
              <v-list-item-title class="list-title">
                {{ item.title }}
              </v-list-item-title>
            </v-list-item-content>
          </v-list-item>
        </template>
      </v-list>
      <v-divider class="my-4" />
      <div class="user-profile">
        <img src="https://i.pravatar.cc/40" alt="User" class="avatar">
        <div class="user-info">
          <strong>Olivia Rhye</strong>
          <small>Admin</small>
        </div>
      </div>
    </v-navigation-drawer>
  </v-app>
</template>

<script>

export default {
  data () {
    return {
      items: [
        {
          title: 'Overview',
          icon: 'mdi-chart-bell-curve',
          path: '/overview'
        },
        {
          title: 'Products',
          icon: 'mdi-cart-outline',
          path: '/products'
        },
        {
          title: 'Supplier',
          icon: 'mdi-account-multiple-outline',
          path: '/suppliers'
        },
        {
          title: 'Category',
          icon: 'mdi-checkbox-marked-outline',
          path: '/category'
        },
        {
          title: 'Warehouse',
          icon: 'mdi-package-variant-closed',
          path: '/warehouse'
        },
        {
          title: 'Customer',
          icon: 'mdi-handshake-outline',
          path: '/customers'
        },
        {
          title: 'Employees',
          icon: 'mdi-account-tie-outline',
          path: '/employees'
        },
        {
          title: 'Departments',
          icon: 'mdi-desktop-tower',
          path: '/departments'
        },
        {
          title: 'Payment',
          icon: 'mdi-database-outline',
          path: '/payment'
        },
        {
          title: 'Roles',
          icon: 'mdi-layers-triple-outline',
          path: '/roles'
        },
        {
          title: 'Support',
          icon: 'mdi-lifebuoy',
          path: '/support'
        },
        {
          title: 'Settings',
          icon: 'mdi-cog-outline',
          path: '/settings'
        }
      ]
    }
  },
  methods: {
    goTo (route) {
      if (route === '/') {
        this.logout()
      } else {
        this.$router.push(route)
      }
    }
    /* async logout () {
      try {
        await this.$axios.post('/users/logout')
        window.location.href = '/'
      } catch (error) {
        const errorMessage = error.response?.data?.message || 'Ocurrio un error al cerrar sesion'
        this.$store.dispatch('alert/triggerAlert', {
          message: errorMessage,
          type: 'error'
        })
      }
    } */
  }
}
</script>

<style scoped>

/* Estilos del sidebar */
.sidebar {
  position: fixed;
  top: 0;
  left: 0;
  height: 100vh;
  z-index: 100;
  background: white;
  box-shadow: 0 0 10px rgba(0, 0, 0, 0.1);
}
.position-fixed {
  position: fixed;
  top: 20px;
  right: 20px;
  z-index: 2000;
}
.fill-height {
  height: 100%;
}

/* Estilos del logo */
.logo-container {
  display: flex;
  justify-content: center;
  align-items: center;
  padding: 1rem;
}
.logo {
  max-width: 200px;
  height: auto;
}

/* Estilos del texto */
.list-title {
  font-size: 16px;
  font-weight: 500;
  font-family: 'Inter', sans-serif; /* o la que tú uses */
  color: #344054;
}

.search {
  margin: 1rem 0;
  padding: 0.5rem;
  width: 100%;
}

/* Estilos de inputs */
.always-focused >>> .v-input__control {
  border: 2px solid #667085 !important; /* mismo color que el ícono */
  border-radius: 4px;
}
.always-focused >>> .v-label {
  color: #667085 !important;
}
.always-focused >>> input {
  color: #344054 !important;
}
.search-field >>> .v-input__control {
  background-color: white !important;
  border-radius: 6px;
}
.search-field >>> input {
  color: #344054 !important;
}
.search-field >>> .v-label {
  color: #667085 !important;
}
.search-field >>> .v-input__control {
  box-shadow: none !important;
  border: none !important;
}

/* Estilos del usuario */
.user-profile {
  display: flex;
  align-items: center;
  padding: 1rem;
  margin-top: auto;
}
.avatar {
  border-radius: 50%;
  width: 40px;
  height: 40px;
  margin-right: 10px;
}
.user-info {
  display: flex;
  flex-direction: column;
  font-size: 14px;
}
.user-info strong {
  color: #344054;
}
.user-info small {
  color: #667085;
}

</style>
