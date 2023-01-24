<template>
  <EditableComponent
    :wrapped-component="wrappedComponent"
    v-bind="computedProps"
  />
</template>

<script lang="ts">
  import { Component, defineComponent, PropType, VNode } from 'vue';
  import EditableComponent from '@/editable/EditableComponent.vue';
  import Utils from '@/utils/Utils';

  interface ReloadForceAble {
    /*
     * Should the model cache be ignored when processing the component.
     */
    cqForceReload?: boolean;
  }

  interface MappedComponentProperties extends ReloadForceAble {
    id?: string;
    isInEditor: boolean;
    cqPath?: string;
    cqType?: string;
    appliedCssClassNames?: string;
    aemNoDecoration?: boolean;
    cqHierarchyType?: string;
  }

  interface EditConfig<P extends MappedComponentProperties> {
    emptyLabel?: string;
    resourceType?: string;

    isEmpty(props: P): boolean;
  }

  export default defineComponent({
    name: 'CompositeEditableComponent',
    components: {
      EditableComponent,
    },
    inheritAttrs: false,
    props: {
      editConfig: {
        type: Object as PropType<EditConfig<MappedComponentProperties>>,
        default: () => {},
      },
      wrappedComponent: {
        type: Object as PropType<Component | VNode>,
        default: () => {},
      },
    },
    setup(props, context) {
      // console.log('CompositeEditableComponent properties: ', props);
      // console.log('CompositeEditableComponent attributes: ', context.attrs);
    },
    computed: {
      computedProps() {
        const defaultEditConfig = this.editConfig || {
          isEmpty: () => false,
        };

        const isInEditor =
          (this.$attrs?.isInEditor as boolean) || Utils.isInEditor();
        const aemNoDecoration =
          (this.$attrs?.aemNoDecoration as boolean) || false;

        return {
          ...this.$attrs,
          componentProperties: {
            ...this.$attrs,
            isInEditor,
            aemNoDecoration,
          },
          editConfig: defaultEditConfig,
        };
      },
    },
  });
</script>
