<template>
  <ModelProvider
    :cq-force-reload="cqForceReload"
    :inject-props-on-init="injectPropsOnInit"
    :is-in-editor="isInEditor"
    :wrapped-component="wrappedComponent"
    v-bind="attrs"
  />
</template>

<script lang="ts">
  import { Component, defineComponent, PropType, VNode } from 'vue';
  import ModelProvider from '@/editable/ModelProvider.vue';
  // eslint-disable-next-line import/no-cycle
  import Utils from '@/utils/Utils';

  export default defineComponent({
    name: 'CompositeModelProvider',
    components: {
      ModelProvider,
    },
    inheritAttrs: false,
    props: {
      wrappedComponent: {
        type: Object as PropType<Component | VNode>,
        default: () => {},
      },
    },
    setup(props, context) {
      // console.log('CompositeModelProvider properties: ', props);
      // console.log('CompositeModelProvider attributes: ', context.attrs);
    },
    computed: {
      attrs(): Record<string, any> {
        return this.$attrs;
      },
      cqForceReload() {
        return (this.$attrs?.cqForceReload as boolean) || false;
      },
      injectPropsOnInit() {
        return (this.$attrs?.injectPropsOnInit as boolean) || false;
      },
      isInEditor() {
        return Utils.isInEditor();
      },
    },
  });
</script>
