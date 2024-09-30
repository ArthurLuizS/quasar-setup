<template>
  <q-page class="flex flex-center">
    <google-login />
  </q-page>
</template>

<script setup>
import GoogleLogin from 'components/googleLogin.vue'
defineOptions({
  name: 'IndexPage'
});
</script>
