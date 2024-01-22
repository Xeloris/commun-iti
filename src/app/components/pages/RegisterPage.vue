<script setup lang="ts">
import { ElMessage } from "element-plus";
import { reactive } from "vue";
import { useRouter } from "vue-router";
import { UserAPI } from "@/modules/user/services";
import { useProvider } from "@/app/platform";
import type { FormRules, FormInstance } from "element-plus";
import type { UserRegistrationModel } from "@/modules/user/models";

const [userApi] = useProvider([UserAPI]);
const router = useRouter();

const registerModel = reactive({
  username: "",
  password: "",
  passwordConfirmation: ""
});

const userNameRegex = /^(\w+)$/i;

const registerFormRules = reactive<FormRules>({
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
  ],
  passwordConfirmation: [
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
      else {
        if (form.$props.model?.password !== form.$props.model?.passwordConfirmation) {
          ElMessage.error( "Les mots de passe ne sont pas identiques");
          return;
        }
        const usernameExists = await userApi.exists(form.$props.model?.username);
        if (usernameExists) {
          ElMessage.error( "Le nom d'utilisateur existe déjà");
          return;
        }
        const registrationData: UserRegistrationModel = {
          username: form.$props.model?.username,
          password: form.$props.model?.password,
        };
        userApi.register(registrationData).then(() => {
          ElMessage({
            message: 'Votre compte a été créé',
            type: 'success',
          });
          router.push('/login');
        })
        .catch(() => {
          ElMessage.error( "Une erreur est survenu lors de l'inscription du compte");
        });
      }
    });
  } catch (e) {
    return;
  }
}
</script>
<template>
  <div class="register center-children full-h">
    <main class="width-s">
      <h1 class="register-title">Créer un compte</h1>

      <div class="register-form">
        <el-form
          ref="form"
          :model="registerModel"
          :rules="registerFormRules"
          label-position="top"
          class="register-form"
          @submit.prevent="onSubmit($refs.form)"
        >
          <el-form-item label="Pseudo" prop="username">
            <el-input v-model="registerModel.username" />
          </el-form-item>

          <el-form-item label="Mot de passe" prop="password">
            <el-input v-model="registerModel.password" type="password" />
          </el-form-item>

          <el-form-item label="Confirmez votre mot de passe" prop="passwordConfirmation">
            <el-input v-model="registerModel.passwordConfirmation" type="password"/>
          </el-form-item>

          <el-form-item>
            <div class="form-actions">
              <el-button type="primary" native-type="submit"> Créer mon compte </el-button>

              <router-link to="/login">J'ai déjà un compte</router-link>
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
