<template>
  Details:<br />
  {{ route.params?.id }}
</template>

<script setup>
const route = useRoute();
const details = ref("");
console.log("route parameter", route.params);
</script>
