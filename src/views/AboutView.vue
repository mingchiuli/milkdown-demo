<template>
  <div class="about">
    <h1>This is an about page</h1>
  </div>
  <MilkdownProvider>
    <MilkdownEditor />
  </MilkdownProvider>
</template>

<style>
@media (min-width: 1024px) {
  .about {
    min-height: 100vh;
    display: flex;
    align-items: center;
  }
}
</style>


<script setup lang="ts">
import { MilkdownProvider } from '@milkdown/vue';
import MilkdownEditor from '@/components/MilkdownEditor.vue'


</script>
