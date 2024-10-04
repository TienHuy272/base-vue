<template>
  <div class="hello">
    <div class="login-container">
    <h2>Login</h2>
    <form @submit.prevent="handleLogin">
      <div>
        <label for="username">Username:</label>
        <input type="text" v-model="username" id="username" required />
      </div>
      <div>
        <label for="password">Password:</label>
        <input type="password" v-model="password" id="password" required />
      </div>
      <button type="submit">Login</button>
    </form>
    <p v-if="errorMessage" class="error">{{ errorMessage }}</p>
    <p>USER DATA: {{userData}} </p>
  </div>
  </div>
</template>

<script>
import axios from 'axios';

export default {
  data() {
    return {
      username: '',
      password: '',
      errorMessage: '',
      userData: {}
    };
  },
  methods: {
    async handleLogin() {
      try {
        const response = await axios.post('http://52.77.245.55:8081/login', {
          username: this.username,
          password: this.password,
        });
        // Assuming the response contains a token and user information
        if (response.data) {
          alert('Login successful!');
          console.log('User Data:', response.data);
          // You can store the token in localStorage or Vuex if needed
          // localStorage.setItem('token', response.data.token);
        }
      } catch (error) {
        if (error.response && error.response.status === 404) {
          this.errorMessage = 'Incorrect username or password';
        } else {
          this.errorMessage = 'An error occurred, please try again later';
        }
      }
    },
  },
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
.login-container {
  max-width: 400px;
  margin: auto;
  padding: 20px;
  border: 1px solid #ccc;
  border-radius: 5px;
}

.error {
  color: red;
}
</style>
