<template>
  <div :class="placeholderClassNames" :data-cq-data-path="`${cqPath}/*`" />
</template>

<script lang="ts">
  import { defineComponent, PropType } from 'vue';

  export default defineComponent({
    // eslint-disable-next-line vue/multi-word-component-names
    name: 'ContainerPlaceholder',
    inheritAttrs: false,
    props: {
      cqPath: {
        type: String,
        default: '',
      },
      placeholderClassNames: {
        type: Array as PropType<Array<string>>,
        default: () => [],
      },
    },
  });
</script>
