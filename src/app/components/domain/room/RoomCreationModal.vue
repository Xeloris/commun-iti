<script lang="ts" setup>
import { ref, reactive } from "vue";
import { useFormModal } from "@/app/components/ui/modal";
import type { FormInstance, FormRules } from "element-plus";
import { useProvider } from "@/app/platform";
import { RoomAPI } from "@/modules/room/services/RoomAPI";
import { RoomService } from "@/modules/room/services/RoomService";
import { ElMessage } from "element-plus";
import type { NewRoom } from "@/modules/room/models/NewRoom";

const [roomApi, roomService] = useProvider([RoomAPI, RoomService]);
const form = ref<FormInstance | null>(null);
const loading = ref(false);

const formRules = reactive<FormRules>({
  name: [
    {
      required: true,
      message: "Nom de salon obligatoire"
    },
  ],
});

const { isVisible, hide, show, formModel } = useFormModal(
  {
    name: ""
  },
  form
);

async function onSubmit(form?: FormInstance) {
  if (!form) {
    return;
  }

  try {
    loading.value = true;
    await form.validate().then(async valid => {
      if (!valid) {
        return ;
      }
      else {
        const salonExist = await roomApi.exists(form.$props.model?.name)
        if (salonExist) {
          ElMessage.error("Ce salon existe déjà");
          return ;
        }
        const roomData: NewRoom = {
          name : form.$props.model?.name
        };
        roomService.create(roomData).then(() => {
          ElMessage({
            message: "Le salon a été créé",
            type: 'success',
          });
          hide();
        })
        .catch(() => {
          ElMessage({
            message: "Une erreur s'est produite lors de la création du salon",
            type: 'error',
          });
          hide();
        });
      }
    });


  } catch (e) {
    return;
  } finally {
    loading.value = false;
  }
}

defineExpose({
  show,
  hide
});
</script>

<template>
  <el-dialog v-model="isVisible" title="Création d'un nouveau salon" width="30%">
    <el-form
      ref="form"
      :model="formModel"
      :rules="formRules"
      label-position="top"
      class="login-form"
      @submit.prevent="onSubmit(form!)"
    >
      <el-form-item label="Nom du salon" prop="name">
        <el-input v-model="formModel.name" />
      </el-form-item>
    </el-form>

    <template #footer>
      <div class="form-actions">
        <el-button native-type="reset" @click="hide()">Annuler</el-button>
        
        <el-button type="primary" native-type="submit" :loading="loading" @click="onSubmit(form!)">
          Créer le salon
        </el-button>
      </div>
    </template>
  </el-dialog>
</template>
