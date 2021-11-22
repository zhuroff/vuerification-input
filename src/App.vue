<template>
  <VuerificationInput
    :length="6"
    :isError="false"
    confirmable
    confirmButton="Send code"
    errorMessage="This code is invalid"
  />
</template>

<script>

import { defineComponent } from 'vue';
import VuerificationInput from '@/components/VuerificationInput.vue';

export default defineComponent({
  components: {
    VuerificationInput,
  },
});

</script>
