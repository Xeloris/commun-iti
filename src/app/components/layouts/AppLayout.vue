<script lang="ts" setup>
import LeftMenu from "@/app/components/domain/menu/LeftMenu.vue";
import RightMenu from "@/app/components/domain/menu/RightMenu.vue";
import MessageInput from "@/app/components/domain/message/MessageInput.vue";
import Room from "@/app/components/domain/room/Room.vue";
import { useProvider, useState, useStore } from "@/app/platform";
import { AuthenticationStore } from "@/modules/authentication/store/AuthenticationStore";
import { WebsocketConnection } from "@/modules/infrastructure/socket";
import { RoomStore } from "@/modules/room/store";

const [socketConnection] = useProvider([WebsocketConnection]);
const authState = useState(AuthenticationStore);
const store = useStore(RoomStore);

if (authState.token?.bearer) {
  socketConnection.connect(authState.token?.bearer);
} else {
  console.warn("Socket connection failed : User not authenticated");
}
</script>
<template>
  <section class="app-layout full-wh">
    <aside>
      <left-menu />
    </aside>

    <main>
      <main>
        <router-view />
      </main>

      <footer>
        <message-input />
      </footer>
    </main>

    <right-menu />
  </section>
</template>
<style lang="scss" scoped>
@use "@/app/styles/var";

.app-layout {
  display: flex;

  >aside {
    width: var.$layout-left-menu-width;
    height: 100%;
    background-color: var.$color-primary-darker;
  }

  >main {
    flex: 1;
    display: flex;
    flex-direction: column;

    >main {
      flex: 1;
    }

    >footer {
      height: var.$layout-footer-height;
    }
  }
}
</style>
