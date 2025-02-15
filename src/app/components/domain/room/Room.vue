<script lang="ts" setup>
import Message from "@/app/components/domain/message/Message.vue";
import { useProvider, useState, useStore } from "@/app/platform";
import { MessageService } from "@/modules/message/services/MessageService";
import { RoomSocketService } from "@/modules/room/services";
import { MessageStore } from "@/modules/message/store";
import { watch, ref } from "vue";
import { useIntersectionObserver } from "@vueuse/core";
import { MessageSocketService } from "@/modules/message/services/MessageSocketService";
import { AuthenticationStore } from "@/modules/authentication/store/AuthenticationStore";
import { ElNotification } from "element-plus";
import type { Room } from "@/modules/room/models/domain/Room";

const props = defineProps<{ room: Room }>();
const state = useState(MessageStore);
const authState = useState(AuthenticationStore);
const store = useStore(MessageStore);

const [messageSerivce, messageSocket, roomSocket] = useProvider([
  MessageService,
  MessageSocketService,
  RoomSocketService
]);

const loading = ref(false);

// Element used to know if we should load more message
const top = ref<HTMLDivElement | null>(null);

// Should be use to know the size in px of all the messages displayed
const container = ref<HTMLDivElement | null>(null);

// Element that have the scrollbar
const root = ref<HTMLDivElement | null>(null);

subscribeToIncomingMessage();


messageSocket.onNewReaction((reaction) => {
  if (reaction.message.author.id === authState.loggedUser?.id) {
    ElNotification({ message: `${reaction.user.username} a reagit a un de vos message`, type: "info"});
  }

  messageSerivce.reloadMessages();
});


watch(
  () => props.room,
  async () => {
    store.reset();
    await fetchMore();

    roomSocket.onRoomJoined((reaction) => {
     this.$notify({
      title: 'Notification',
      message: "Quelqu'un a rejoint un salon",
      type: 'info'
      });
    });

    subscribeToIncomingMessage();
  },
);

function subscribeToIncomingMessage() {
  messageSocket.onNewMessage(props.room.id, () => {
    messageSerivce.reloadMessages();
  })
}

async function fetchMore() {
  if (loading.value) {
    return;
  }

  try {
    loading.value = true;

    await messageSerivce.fetchMore(props.room.id);
  } catch (e) {
    console.error(e);
  } finally {
    loading.value = false;
  }
}

const { stop } = useIntersectionObserver(
  top,
  ([{ isIntersecting }], observerElement) => {
    if (isIntersecting) {
      if (state.messagesPagination.page + 1 < state.messagesPagination.total / state.messagesPagination.perPage) {
        fetchMore();
      }
      else {
        stop();
      }
    }
  },
)

</script>

<template>
  <div class="room stretch-wh" ref="root">
    <div class="room-container" ref="container">
      <Message v-for="item in state.currentRoomMessages" :key="item.id" :message="item" />
      <div ref="top">
      </div>
    </div>
  </div>
</template>

<style lang="scss" scoped>
.room {
  display: flex;
  flex-direction: column-reverse;
  overflow-y: auto;
}

.room-container {
  display: flex;
  flex-direction: column-reverse;
}
</style>
