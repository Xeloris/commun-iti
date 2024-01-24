<script lang="ts" setup>
import BgImage from "@/app/components/ui/BgImage.vue";
import { ElMessageBox } from "element-plus";
import { SwitchButton, UserFilled, Message } from "@element-plus/icons-vue";
import { AuthenticationStore } from "@/modules/authentication/store/AuthenticationStore";
import { useProvider, useState } from "@/app/platform";
import { AuthenticationService } from "@/modules/authentication/services";
import { useRightMenuState } from "@/app/components/domain/menu/useRightMenu";
import { ref } from "vue";
import EditUserProfileModal from "../user/EditUserProfileModal.vue"

const state = useState(AuthenticationStore);
const [authService] = useProvider([AuthenticationService]);

const editUserProfileModal = ref<InstanceType<typeof EditUserProfileModal>>();

async function openUserProfileModal() {
  if (state.loggedUser) {
    const user = state.loggedUser;
    await editUserProfileModal.value?.show({username: user.username, pictureUrl: user.pictureUrl, picture: null});
  }
}

function logout() {
  ElMessageBox.confirm("Souhaitez-vous vous déconnecter de la session actuelle ?", "Warning", {
    confirmButtonText: "Oui",
    cancelButtonText: "Annuler",
    type: "warning"
  })
    .then(() => {
      authService.logout();
      location.href = "/login";
    })
    .catch(() => {});
}

function toggleRightMenu() {
  useRightMenuState().toggle();
}
</script>

<template>
  <div class="user-profile-menu-item">
    <div class="user-profile-info">
      <bg-image class="user-profile-info-picture" :src="state.loggedUser?.pictureUrl!" />

      <div class="user-profile-info-username">{{ state.loggedUser?.username }}</div>
    </div>

    <div class="user-profile-actions">
      <el-button :icon="SwitchButton" type="danger" size="default" @click="logout()" />
      <el-button :icon="Message" type="info" size="default" @click="toggleRightMenu()" />
      <el-button :icon="UserFilled" type="" size="default" @click="openUserProfileModal" />
      <EditUserProfileModal ref="editUserProfileModal" />
    </div>
  </div>
</template>

<style lang="scss" scoped>
@use "sass:map";
@use "@/app/styles/var";

.user-profile-info {
  height: var.$layout-top-menu-height * 2;
  padding: map-get(var.$spaces, "xs");
  display: flex;
  color: white;
  font-size: 14px;

  gap: map-get(var.$spaces, "xs");

  > .user-profile-info-picture {
    $width: calc(var.$layout-top-menu-height * 2 - map-get(var.$spaces, "xs") * 2);
    height: 100%;
    width: $width;
    min-width: $width;
  }
}

.user-profile-actions {
  display: flex;
  justify-content: space-between;
  padding: 0 map-get(var.$spaces, "xs") map-get(var.$spaces, "xs") map-get(var.$spaces, "xs");
}
</style>
