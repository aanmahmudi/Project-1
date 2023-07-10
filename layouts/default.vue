<template>
  <v-app>
    <v-navigation-drawer disable-resize-watcher
      v-model="sidedrawer"
      fixed
      app
    >
      <v-list>
        <v-list-item
          v-for="(item, i) in sideMenu"
          :key="i"
          :to="item.to"
          router
          exact
        >
          <v-list-item-action>
            <v-icon>{{ item.icon }}</v-icon>
          </v-list-item-action>
          <v-list-item-content>
            <v-list-item-title v-text="item.title" />
          </v-list-item-content>
        </v-list-item>
      </v-list>
    </v-navigation-drawer>

    <v-main>
      <v-container fill-height fluid>
        <Nuxt />
      </v-container>
    </v-main>

    <v-bottom-navigation
      horizontal
      height="10vh"
      fixed
      color="primary"
      app
    >
      <v-app-bar-nav-icon   
        @click.stop="sidedrawer = !sidedrawer"
        v-ripple="false" 
        plain
      />
      <v-btn v-for="(item, i) in bottomMenu"
            :key="1"
            :to="item.to"
            v-ripple="false"
            plain>
        <span> 
            {{ item.title }}
        </span>
        <v-icon>
            {{ item.icon }}
        </v-icon>
      </v-btn>
      <v-spacer/>
    </v-bottom-navigation>

  </v-app>
</template>

<script>

import { mapGetters } from 'vuex'

export default {
  name: 'DefaultLayout',
  data() {
    return {
      sidedrawer: false,
      sideMenu: [],
      originalsideMenu: [
      {
          icon: 'mdi-view-dashboard-variant',
          title: 'Dashboard',
          to: '/dashboard',
          middleware: ['authenticated']
        },
        {
          icon: 'mdi-application',
          title: 'Cashier App',
          to: '/',
          middleware: ['admin', 'cashier']
        },
        {
          icon: 'mdi-account',
          title: 'User Management',
          to: '/users',
          middleware: ['admin']
        },
        {
          icon: 'mdi-fingerprint',
          title: 'Absence',
          to: '/absence',
          middleware: ['authenticated']
        },
        {
          icon: 'mdi-login',
          title: 'Login',
          to: '/login',
          middleware: ['unauthenticated']
        },
        {
          icon: 'mdi-logout',
          title: 'Logout',
          to: '/logout',
          middleware: ['authenticated']
        },
      ],
      bottomMenu: [
        {
          icon: 'mdi-application',
          title: 'App',
          to: '/',
        },
        
      ],
    }
    
  },
  computed: {
    ...mapGetters('auth', {
        authenticated : 'authenticated',
        user : 'user',
    }),
  },
  methods: {
    isWelcomeScreen() {
      if(!localStorage.welcomeScreen){
        if(this.$router.currentRoute.path != '/register' &&
           this.$router.currentRoute.path != '/Login'){
            this.$router.push('/login')
        }
      }
    },
    filterSidemenu() {
      this.sideMenu = this.originalsideMenu.filter(item => {
        if (item.middleware.includes(this.user.role)){
          return true
        }
        if(this.authenticated){
          return item.middleware.includes ('authenticated')
        } else {
          return item.middleware.includes ('unauthenticated')
        }
      })
    },
  
  },
  watch: {
    $route() {
      this.isWelcomeScreen()
    },
    authenticated() {
      this.filterSidemenu()
    }
  },
  mounted(){
    //localStorage.setItem("welcomeScreen", true)
    this.isWelcomeScreen()

    this.filterSidemenu()
  }
}
</script>
