<template>
  <div class="grid-container">
    <v-card class="card">
      <v-card-text>
        <v-form @submit.prevent="login">
          <v-text-field
            v-model="userName"
            :rules="[() => !!userName || 'This field is required']"
            variant="outlined"
            label="Username"
            clearable
          ></v-text-field>

          <v-text-field
            class="mt-2"
            v-model="password"
            :append-inner-icon="show1 ? 'mdi-eye' : 'mdi-eye-off'"
            :rules="[() => !!password || 'This field is required']"
            :type="show1 ? 'text' : 'password'"
            label="Password"
            variant="outlined"
            @click:append-inner="togglePasswordVisibility"
          ></v-text-field>

          <v-btn class="button" type="submit" block>Login</v-btn>
          <router-link class="link" to="register">
            Create a new account.
          </router-link>
        </v-form>
      </v-card-text>
    </v-card>
  </div>
</template>

<script setup>
  import { ref } from 'vue';
  import axios from 'axios';
  import { useRouter } from 'vue-router';

  const userName = ref('');
  const password = ref('');
  const show1 = ref(false);
  const router = useRouter();

  const togglePasswordVisibility = () => {
    show1.value = !show1.value;
  };

  const login = () => {
    const data = {
      userName: userName.value,
      password: password.value,
    };
    if (!data.userName || !data.password) {
      alert('Please enter all fields!');
      return;
    }
    axios
      .post('https://localhost:7107/Login', data)
      .then((response) => {
        const token = response.data.jwtToken;

        localStorage.setItem('jwtToken', token);
        router.push({ name: 'dashboard' });
      })
      .catch((err) => {
        console.error('Error:', err.response ? err.response.data : err.message);
      });
  };
</script>

<style scoped>
  .grid-container {
    display: grid;
    place-items: center;
    height: 100vh;
    background-color: #121212;
    padding: 1rem;
  }

  .card {
    display: block;
    min-width: 400px;
    padding: 2rem;
    background-color: #1e1e1e;
    border-radius: 10px;
    box-shadow: 0 4px 8px rgba(0, 0, 0, 0.2);
  }

  .button {
    background-color: #ff5252;
    color: white;
  }

  .link {
    color: #ff5252;
    text-align: center;
    display: block;
    padding-top: 1rem;
  }

  .mt-2 {
    margin-top: 1rem;
  }

  .mt-4 {
    margin-top: 2rem;
  }

  @media (max-width: 600px) {
    .card {
      min-width: auto;
      width: 100%;
    }
  }
</style>
