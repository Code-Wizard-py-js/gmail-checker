<template>
  <div class="w-full p-8 mx-auto bg-gray-100 sm:w-1/2 h-96 ">
    <h2 class="mb-4 text-2xl font-bold ">Login</h2>
    <form @submit.prevent="login" class="space-y-10 ">
      <input type="text" v-model="email" placeholder="Email" class="w-full p-2 border rounded">
      <input type="password" v-model="password" placeholder="Password" class="w-full p-2 border rounded">
      <button type="submit" class="w-full p-2 text-white bg-blue-500 rounded">Login</button>
      <p v-if="error" class="text-red-500">{{ error }}</p>
    </form>
  </div>
</template>

<script>
export default {
  data() {
    return {
      email: '',
      password: '',
      error: ''
    };
  },
  methods: {
    async login() {
      const formData = {
        email: this.email,
        password: this.password
      };

      try {
        const response = await fetch('http://localhost:3000/login', {
          method: 'POST',
          headers: {
            'Content-Type': 'application/json'
          },
          body: JSON.stringify(formData)
        });

        if (response.ok) {
          const data = await response.json();
          console.log('Response Data:', data);
          if (data.message === 'success') {
            localStorage.setItem('authenticated', 'true');
            localStorage.setItem('userid', this.email);
            this.$emit('login');
          } else {
            this.error = 'Incorrect email or password';
          }
        } else if (response.status === 401) {
          this.error = 'Incorrect email or password';
        } else {
          this.error = 'Error occurred while logging in';
          console.error('Error:', response.statusText);
        }
      } catch (error) {
        this.error = 'Error occurred while logging in';
        console.error('Error:', error);
      }
    }
  }
};

</script>
