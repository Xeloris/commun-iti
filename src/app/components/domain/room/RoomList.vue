<script lang="ts" setup>
import { RoomStore } from "@/modules/room/store";
import { useProvider, useState, useStore } from "@/app/platform";
import type { Room } from "@/modules/room/models/domain/Room";
import { ref, watchEffect } from "vue";
import { useRouter } from "vue-router";
import { RoomAPI } from "@/modules/room/services";
const state = useState(RoomStore);
const store = useStore(RoomStore);
const router = useRouter();
const [roomApi] = useProvider([RoomAPI]);

const redirectionDone = ref(false);

function selectRoom(room: Room) {
  store.setCurrentRoom(room);
  localStorage.setItem("lastRoom", room.id);
}

watchEffect(() => {
  if (!redirectionDone.value && (state.rooms[0] !== undefined || localStorage.getItem("lastRoom")!)) {
    if (localStorage.getItem("lastRoom")!) {
      roomApi.findById(localStorage.getItem("lastRoom")!).then(room => {
        selectRoom(room!);
        router.push('app/room/' + room!.id);
        redirectionDone.value = true;
        return ;
      })
    }
    const firstRoom = state.rooms[0];
    if (state.currentRoom === undefined && firstRoom) {
      selectRoom(firstRoom);
      router.push('app/room/' + firstRoom.id);
    }
    redirectionDone.value = true;
  }
  
});
</script>

<template>
  <nav class="room-list">
    <ul>
      <li v-for="room in state.rooms" :key="room.id">
        <router-link :to="'/app/room/' + room.id" class="room-list-item" @click="selectRoom(room)">
          {{ room.name }}
        </router-link>
      </li>
    </ul>
  </nav>
</template>

<style lang="scss" scoped>
@use "sass:map";
@use "@/app/styles/var";

.room-list {
  padding-top: 1px;
  > ul {
    margin: 0;
    list-style-type: none;
    padding-inline: 0;

    > li {
      margin: 0;
      text-decoration: none;
      display: block;

      a {
        color: var.$color-lighter;
        text-decoration: none;
        display: block;
        padding: map-get(var.$spaces, "2xs") map-get(var.$spaces, "xs");

        &:hover {
          background-color: var.$color-light;
        }

        &.router-link-active {
          color: #ffffff;
          font-weight: 500;
        }
      }
    }
  }
}
</style>
