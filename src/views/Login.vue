<template>
  <v-app id="login">
    <v-main>
      <v-container class="fill-height" fluid>
        <v-row align="center" justify="center">
          <v-col cols="12" md="5" sm="8">
            <v-card class="elevation-12">
              <v-progress-linear
                v-if="waiting == true"
                indeterminate
              ></v-progress-linear>
              <v-window v-model="step">
                <v-window-item :value="1">
                  <v-row>
                    <v-col cols="12" md="12">
                      <v-card-text class="mt-6" v-on:keyup.enter="handleSubmit">
                        <h2 class="text-center display-1 ">
                          LOGIN
                        </h2>
                        <h6 class="text-center ">
                          Ambiente Web de Gerenciamento PBL
                        </h6>
                        <v-divider
                          justify-center
                          style="margin-top: 8px; width: 80%; margin-left: 10%"
                        ></v-divider>
                        <v-alert
                          v-show="loginError && validLogin"
                          text
                          dense
                          type="error"
                          class="mt-3 mb-3 "
                          transition="fade-transition"
                        >
                          Usuário e Senha não correspondem às informações em
                          nossos registros. Tente novamente.
                        </v-alert>
                        <v-form
                          v-model="validLogin"
                          lazy-validation
                          ref="signin"
                          class="mt-2"
                        >
                          <v-text-field
                            id="user"
                            v-model="formLogin.username"
                            label="Nome de usuário ou email"
                            prepend-icon="mdi-account"
                            type="text"
                            :rules="usernameRules"
                          />
                          <v-text-field
                            id="password"
                            v-model="formLogin.password"
                            label="Senha"
                            prepend-icon="mdi-lock"
                            type="password"
                            :rules="passwordRules"
                          />
                        </v-form>
                        <div class="text-center">
                          <v-btn text color="primary" small
                            >ESQUECEU A SENHA?</v-btn
                          >
                        </div>

                        <div class="text-center mt-3">
                          <v-btn
                            block
                            color="var(--primary-dark-color)"
                            style="color: white"
                            @click.prevent="handleSubmit"
                            >ENTRAR</v-btn
                          >
                        </div>

                        <div class="text-center mt-3">
                          Ainda não tem conta?
                          <v-btn
                            text
                            color="primary"
                            small
                            @click="
                              () => {
                                step++;
                                loginError = false;
                                formLogin = { username: '', password: '' };
                                this.$refs.signin.reset();
                              }
                            "
                            >CADASTRE-SE</v-btn
                          >
                        </div>
                      </v-card-text>
                    </v-col>
                  </v-row>
                </v-window-item>
                <v-window-item :value="2">
                  <v-row class="fill-height">
                    <v-col cols="12" md="12">
                      <v-card-text class="mt-6" v-on:keyup.enter="handleSubmit">
                        <h1 class="text-center display-1">
                          CRIAR CONTA
                        </h1>
                        <v-form
                          ref="signup"
                          v-model="validRegistration"
                          lazy-validation
                        >
                          <v-text-field
                            v-model="formRegistration.fullName"
                            :rules="nameRules"
                            label="Nome completo"
                            prepend-icon="mdi-account"
                            type="text"
                          />
                          <v-text-field
                            v-model="formRegistration.email"
                            :rules="emailRules"
                            label="Email"
                            prepend-icon="mdi-email"
                            type="email"
                          />
                          <v-text-field
                            v-model="formRegistration.enrollment"
                            :rules="enrollmentRules"
                            label="Número de matrícula"
                            prepend-icon="mdi-school"
                            type="number"
                          />
                          <v-text-field
                            :rules="passwordRules"
                            v-model="formRegistration.password"
                            label="Senha"
                            prepend-icon="mdi-lock"
                            type="password"
                          />
                        </v-form>
                        <div class="text-center mt-3">
                          <v-btn block dark @click.prevent="handleSubmit"
                            >CADASTRAR</v-btn
                          >
                        </div>
                        <div class="text-center mt-3">
                          Já é cadastrado?
                          <v-btn
                            text
                            color="primary"
                            small
                            @click="
                              () => {
                                step--;
                                formRegistration = {
                                  fullName: '',
                                  email: '',
                                  enrollment: '',
                                  password: '',
                                };
                                this.$refs.signup.reset();
                              }
                            "
                            >FAZER LOGIN</v-btn
                          >
                        </div>
                      </v-card-text>
                    </v-col>
                  </v-row>
                </v-window-item>
              </v-window>
            </v-card>
          </v-col>
        </v-row>
      </v-container>
    </v-main>
  </v-app>
</template>
<script>
import authService from "../services/authService";
// import router from "../router";
export default {
  data: () => ({
    formLogin: { username: "", password: "" },
    validLogin: undefined,
    formRegistration: { email: "", password: "", enrollment: "", fullName: "" },
    validRegistration: "",
    step: 1,
    nameRules: [],
    emailRules: [],
    enrollmentRules: [],
    passwordRules: [],
    usernameRules: [],
    loginError: false,
    waiting: false,
  }),
  methods: {
    async handleSubmit() {
      try {
        // Login Form
        if (this.step == 1) {
          this.waiting = true;
          this.usernameRules = [(v) => !!v || "Insira um nome de usuário"];
          this.passwordRules = [(v) => !!v || "Insira uma senha"];
          if (this.$refs.signin.validate()) {
            var res = await authService.login(this.formLogin);
            sessionStorage.setItem("token", res.data.token.token);
            sessionStorage.setItem("user_type", res.data.type.user_type);
            sessionStorage.setItem("type", res.data.type.type);
            sessionStorage.setItem("username", res.data.type.first_name + " " + res.data.type.surname);
            window.location.replace("http://localhost:8081/user");
          }
          this.waiting = false;
        }
        // Registration Form
        else if (this.step == 2) {
          // Rules for form validation after submit to prevent error while writing
          this.nameRules = [(v) => !!v || "Nome é requerido"];
          this.emailRules = [
            (v) => !!v || "Email é requerido",
            (v) => /.+@.+\..+/.test(v) || "Email deve ser válido",
          ];
          this.enrollmentRules = [
            (v) => !!v || "Matrícula é requerida",
            (v) => (v && v.length == 8) || "Matrícula Inválida",
          ];
          this.passwordRules = [(v) => !!v || "Senha é requerida"];

          // Form Validation
          this.$refs.signup.validate();

          //Error, estado do valid tá true desde o início
          if (this.validRegistration == true) {
            console.log(this.formRegistration);
          }
        }
      } catch (error) {
        if (this.step == 1) {
          console.log(error);
          this.waiting = false;
          if (this.validLogin) this.loginError = true;
        } else if (this.step == 2) {
          console.log(error);
        }
      }
    },
  },
};
</script>
<style scoped>
#login {
  width: 100%;
  /* background: var(--primary-light-color); */
  /*background: linear-gradient(
    90deg,
    rgba(116, 150, 212, 1) 0%,
    rgba(255, 135, 71, 1) 100%
  ); */
  background: linear-gradient(
    0deg,
    rgba(255, 135, 71, 1) 0%,
    rgba(116, 150, 212, 1) 100%
  );
}
</style>
