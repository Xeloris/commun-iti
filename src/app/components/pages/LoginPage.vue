<script setup lang="ts">
import { reactive } from "vue";
import { useRouter } from "vue-router";
import { AuthenticationService } from "@/modules/authentication/services";
import { ElMessage } from "element-plus";
import { useProvider } from "@/app/platform";
import type { FormInstance, FormRules } from "element-plus";
import type { LoginModel } from "@/modules/authentication/models/LoginModel";

const [authService] = useProvider([AuthenticationService]);
const router = useRouter();

const loginModel = reactive({
  username: "",
  password: ""
});

const userNameRegex = /^(\w+)$/i;

const loginFormRules = reactive<FormRules>({
  username: [
    {
      required: true,
      message: "Pseudo obligatoire"
    },
    {
      pattern: userNameRegex,
      message: "Le pseudo doit correspondre a la regex"
    },
  ],
  password: [
    {
      required: true,
      message: "Mot de passe obligatoire"
    },
  ]
});

async function onSubmit(form?: FormInstance) {
  if (!form) {
    return;
  }

  try {
    await form.validate().then(async valid => {
      if (!valid) {
        return ;
      }
      
      const loginData: LoginModel = {
          username: form.$props.model?.username,
          password: form.$props.model?.password,
        };
        const isLogin = await authService.authenticate(loginData);
        if (isLogin) {
          router.push('/app');
        }
        else {
          ElMessage.error( "Les identifiants ne sont pas corrects");
        }
    });

  } catch (e) {
    return;
  }
}
</script>
<template>
  <div class="login center-children full-h">
    <main class="width-s">
      <h1 class="login-title">Se connecter</h1>

      <div class="login-form">
        <el-form
          ref="form"
          :model="loginModel"
          :rules="loginFormRules"
          label-position="top"
          class="login-form"
          @submit.prevent="onSubmit($refs.form)"
        >
          <el-form-item label="Pseudo" prop="username">
            <el-input v-model="loginModel.username" />
          </el-form-item>

          <el-form-item label="Mot de passe" prop="password">
            <el-input v-model="loginModel.password" type="password" />
          </el-form-item>

          <el-form-item>
            <div class="form-actions">
              <el-button type="primary" native-type="submit">Connexion</el-button>
              <router-link to="/register">Je n'ai pas de compte</router-link>
            </div>
          </el-form-item>
        </el-form>
      </div>
    </main>
  </div>
</template>
<style scoped lang="scss">
@use "@/app/styles/var";
@use "@/app/styles/mixins";
@use "sass:list";

.form-actions {
  width: 100%;
  display: flex;
  justify-content: space-between;
}
</style>
