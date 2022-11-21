<template>
  <Suspense>
    <AsyncCityViewVue/>
    <template #fallback>
        <h5 class="row items-center justify-center">Loading...</h5>
    </template>
  </Suspense>
  
</template>

<script lang="ts" setup>
    import AsyncCityViewVue from 'src/components/AsyncCityView.vue';
</script>

<style>

</style>