<template>
  <div>
    <img
      :src="`${publicPath}assets/logo_fwb_loading.svg`"
      alt="Fédération Wallonie-Bruxelles"
      class="w-1/4 mx-auto"
    >
    <p class="text-center">
      Veuillez patienter...
    </p>
  </div>
</template>

<script>
export default {
  data() {
    return {
      publicPath: process.env.BASE_URL
    };
  }
};
</script>
