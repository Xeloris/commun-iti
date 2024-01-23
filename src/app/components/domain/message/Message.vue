<script lang="ts" setup>
import RichText from "@/app/components/ui/RichText.vue";
import BgImage from "../../ui/BgImage.vue";
import ItiEmojiPicker from "../../ui/emoji-picker/EmojiPicker.vue";
import EmojiIcon from "../../ui/icons/EmojiIcon.vue";
import MessageAttachements from "./MessageAttachements.vue";
import { useProvider, useState } from "@/app/platform";
import { MessageService } from "@/modules/message/services/MessageService";
import { DateTime } from "luxon";
import MessageReactions, { type MessageReaction } from "./MessageReactions.vue";
import { type EmojiReaction, type Message } from "@/modules/message/models/domain";
import { AuthenticationStore } from "@/modules/authentication/store/AuthenticationStore";
import ImageAttachement from "@/app/components/domain/message/attachements/ImageAttachement.vue";
import VideoAttachement from "@/app/components/domain/message/attachements/VideoAttachement.vue";
import YoutubeAttachement from "@/app/components/domain/message/attachements/YoutubeAttachement.vue";

const props = defineProps<{
  message: Message;
}>();

const [messageService] = useProvider([MessageService]);
const authState = useState(AuthenticationStore);

function onEmojiPicked(emoji: string) {
  messageService.reactTo(emoji, props.message);
}

function handleReactionClick(reaction: EmojiReaction) {
  if (reaction.userReactions.some(reactionUser => reactionUser.userId === authState.loggedUser?.id)) {
    messageService.removeReaction(reaction.emoji, props.message);
  }
  else {
    onEmojiPicked(reaction.emoji);
  }

}

</script>

<template>
  <div class="message">
    <div class="message-actions">
      <iti-emoji-picker ref="emojiPicker" @pick="onEmojiPicked" />
      <el-button :icon="EmojiIcon" circle size="small" @click="$refs.emojiPicker.show()" />
    </div>

    <bg-image class="message-user-photo" :src="props.message.author.pictureUrl" />

    <div class="message-content">
      <div class="message-title">
        <span> {{ props.message.author.username + " " }} </span>
        <small class="message-date">
          {{ DateTime.fromJSDate(props.message.creationDate).toFormat("dd LLLL yyyy") }}
        </small>
        <RichText :text="props.message.text" />
        <message-reactions :reactions="props.message.reactions" @reactionClick="handleReactionClick"/>
        <li v-for="attachment in props.message.attachements" :key="attachment.type">
          <ImageAttachement v-if="attachment.type === 'image'" :src="attachment.src"/>
          <VideoAttachement v-if="attachment.type === 'video'" :src="attachment.src"/>
          <YoutubeAttachement  v-if="attachment.type === 'youtube'" :attachement="attachment"/>
        </li>
      </div>
    </div>

  </div>
</template>

<style lang="scss">
@use "sass:map";
@use "@/app/styles/var";

.message {
  margin: 0;
  padding: map-get(var.$spaces, "xs") map-get(var.$spaces, "s");
  display: flex;
  align-items: center;
  position: relative;

  &:hover {
    background-color: var.$color-lighter-gray;

    .message-actions {
      display: block;
    }
  }

  .message-actions {
    position: absolute;
    display: none;
    top: 0;
    right: 0;
    padding: map-get(var.$spaces, "xs") map-get(var.$spaces, "s");
    z-index: 10;
  }

  .message-user-photo {
    height: 40px;
    width: 40px;
    align-self: flex-start;
  }

  .message-title {
    font-weight: bold;

    .message-date {
      font-weight: normal;
      font-size: 12px;
    }
  }

  .message-content {
    padding-left: map-get(var.$spaces, "xs");
  }
}
</style>
