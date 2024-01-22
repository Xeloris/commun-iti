<script lang="ts" setup>
import { MessageService } from "@/modules/message/services/MessageService";
import { useProvider, useState } from "@/app/platform";
import { RoomStore } from "@/modules/room/store";
import type { RichText } from "@/modules/message/models/domain";
import RichTextEditor from '../../ui/RichTextEditor.vue';
import { ref } from "vue";
import type { NewMessage } from "@/modules/message/models/NewMessage";

const [messageService] = useProvider([MessageService]);
const roomState = useState(RoomStore);
const richTextContent = ref<RichText | null>(null);

function sendMessage() {
  
  if (richTextContent.value && roomState.currentRoom) {
    const newMessage: NewMessage = {
      roomId: roomState.currentRoom.id,
      text: richTextContent.value
    };
    messageService.sendMessage(newMessage);
    richTextContent.value = null;
  }
}

</script>
<template>
  <div class="message-input stretch-wh">
    <rich-text-editor v-model="richTextContent" @input="sendMessage"/>
  </div>
</template>
<style lang="scss" scoped>
@use "@/app/styles/var";

.message-input {
  border-top: 1px solid var.$color-light-gray;
  background-color: var.$color-light-gray;
}

</style>
