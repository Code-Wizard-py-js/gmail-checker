<template>
  <div id="app" class="h-screen ">
    <div class="grid w-full sm:grid-col-2 grid-col-1">
      <div class="bg-gray-300 ">
        <div class="p-8 banner">
          <h1 class="text-4xl font-bold text-center">Welcome to Gmail checker and validator.</h1>
          <!-- Banner content goes here -->
        </div>
      </div>
      <div class="h-full px-1 py-5 ">
        <div v-if="!authenticated">
          <LoginForm @login="authenticate" />
        </div>
        <div class="bg-gray-500" v-else>
          <Dashboard @logout="logout" />
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import LoginForm from './components/LoginForm.vue';
import Dashboard from './components/Dashboard.vue';

export default {
  name: 'App',
  data() {
    return {
      authenticated: false
    };
  },
  components: {
    LoginForm,
    Dashboard
  },
  created() {
    // Check localStorage for authentication status
    const isAuthenticated = localStorage.getItem('authenticated') === 'true';
    if (isAuthenticated) {
      this.authenticated = true;
    }
  },
  methods: {
    authenticate() {
      // Perform authentication logic
      this.authenticated = true;
    },
    logout() {
      // Perform logout logic
      this.authenticated = false;
      // Clear authenticated flag from localStorage
      localStorage.removeItem('authenticated');
    }
  }
};
</script>

<style>
/* You can include Tailwind CSS styles here if needed */
</style>
