<template>
  <div class="hello">
    <div class="login-container">
    <h2>Login</h2>
    <div>
      <div class="mb-20">
        <label for="username">Username</label>
        <input type="text" v-model="username" id="username" required />
      </div>
      <div class="mb-20">
        <label for="password">Password</label>
        <input type="password" v-model="password" id="password" required />
      </div>
      <div>
      <button class="mb-20" @click="handleLogin">Login</button>
      </div>
      <div>
      <button class="mb-20" @click="loginWithGoogle">Login with Google</button>
      </div>
      <div>
      <button @click="fetchUsers">Fetch User (authorized)</button>
      </div>
    </div>
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
      userData: {},
      authenUrl: null
    };
  },
  methods: {
    async handleLogin() {
      try {
        localStorage.removeItem('access_token');
        const response = await axios.post('http://localhost:8081/login', {
          username: this.username,
          password: this.password,
        });
        if (response.data) {
          alert("Login successfully")
          localStorage.setItem('access_token', response.data.token);
        }
      } catch (error) {
        if (error.response && error.response.status === 404) {
          this.errorMessage = 'Incorrect username or password';
        } else {
          this.errorMessage = 'An error occurred, please try again later';
        }
      }
    },
    async loginWithGoogle() {
      if (!this.authenUrl) {
        await this.initAuthenUrl();
      }
      window.location.href = this.authenUrl; // Redirect to Google OAuth
    },
    async initAuthenUrl() {
      try {
        localStorage.removeItem("access_token");
        const {data} = await axios.get('http://localhost:8081/auth/url'); 
        this.authenUrl = data.url;
      } catch (error) {
        console.log(error);
      }
    },
    async initAccessToken(code) {
      try {
        const {data} = await axios.get(`http://localhost:8081/auth/callback?code=${code}`); 
        localStorage.setItem("access_token", (data.token))
      } catch (error) {
        console.log(error);
      }
    },
    async fetchUsers() {
      try {
        const {data} = await axios.get(`http://localhost:8081/api/v1/users`, {
          headers: {
            "Authorization": "Bearer " + localStorage.getItem("access_token")
          }
        }); 
        alert(data)
      } catch (error) {
        alert(error)
        console.log(error);
      }
    },
  },
  created() {
    let accessToken = localStorage.getItem("access_token");
    if (accessToken) {
      return;
    }
    const urlParams = new URLSearchParams(window.location.search);
    const code = urlParams.get('code');
    if (code) {
      this.initAccessToken(code);
    } else {
      this.initAuthenUrl();
    }
  }
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

button {
  padding: 10px 20px;
  background-color: #4285f4;
  color: white;
  border: none;
  border-radius: 5px;
  cursor: pointer;
  font-size: 16px;
}

.mb-20 {
  margin-bottom: 20px;
}
</style>
