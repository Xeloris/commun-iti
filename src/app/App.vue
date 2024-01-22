<script setup lang="ts">
import { RouterView } from "vue-router";
</script>

<template>
  <main>
    <router-view v-slot="{ Component }">
      <transition name="fade">
        <component :is="Component" />
      </transition>
    </router-view>
  </main>
</template>
