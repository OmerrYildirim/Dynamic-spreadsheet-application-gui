<template>
   <div class="grid-container">
      <v-card class="card">

         <v-tabs class="tabs" v-model="tab">

            <v-tab class="login" value="login" width="150">Login</v-tab>
            <v-tab class="register" value="register" width="150">Register</v-tab>

         </v-tabs>

         <v-card-text>
            <v-tabs-window v-model="tab">
               <v-tabs-window-item value="login">
                  <v-form @submit.prevent="login">
                     <v-text-field v-model="userName" :rules="userNameRules" label="Username">
                     </v-text-field>

                     <v-text-field v-model="password" :rules="passwordRules" label="Password">
                     </v-text-field>

                     <v-btn class="mt-2" type="submit" block>Login</v-btn>
                  </v-form>
               </v-tabs-window-item>

               <v-tabs-window-item value="register">
                  <v-form class="form" @submit.prevent="register">

                     <v-text-field v-model="name" :rules="nameRules" label="Name">
                     </v-text-field>

                     <v-text-field v-model="surName" :rules="surNameRules" label="Surname">
                     </v-text-field>

                     <v-text-field v-model="email" :rules="emailRules" label="Email">
                     </v-text-field>

                     <v-text-field v-model="userName" :rules="userNameRules" label="Username">
                     </v-text-field>

                     <v-text-field v-model="password" :rules="passwordRules" label="Password">
                     </v-text-field>


                     <v-btn class="mt-2" type="submit" block>Register</v-btn>
                  </v-form>
               </v-tabs-window-item>

            </v-tabs-window>
         </v-card-text>


      </v-card>
   </div>
</template>

<script>
export default {
   data() {
      return {
         tab: 'login',
         userName: '',
         password: '',

      };
   },
   methods: {
      login() {

         console.log('Logging in with:', this.userName, this.password);
      },
      register() {

         console.log('Registering with:', this.userName, this.password, this.confirmPassword);
      },
   }
};
</script>


<style scoped>
.grid-container {
   display: grid;
   place-items: center;
   height: 100vh;
   background-color: black;

}

.card {
   margin: auto;
   background-color: rgb(8, 67, 67);

}

.tabs {
   margin: auto;
   border: none;

}
</style>
