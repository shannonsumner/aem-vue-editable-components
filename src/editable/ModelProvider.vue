<template>
  <component :is="wrappedComponent" :key="updateTime" v-bind="computedProps" />
</template>

<script lang="ts">
  /* eslint-disable vue/no-setup-props-destructure,prefer-const */
  import { Component, defineComponent, PropType, VNode } from 'vue';
  import { ModelManager, PathUtils } from '@adobe/aem-spa-page-model-manager';
  import Utils from '@/utils/Utils';

  declare global {
    interface Window {
      ModelManager: typeof ModelManager;
    }
  }

  export default defineComponent({
    name: 'ModelProvider',
    inheritAttrs: false,
    props: {
      cqForceReload: {
        type: Boolean,
        default: false,
      },
      cqPath: {
        type: String,
        default: '',
      },
      injectPropsOnInit: {
        type: Boolean,
        default: false,
      },
      isInEditor: {
        type: Boolean,
        default: Utils.isInEditor(),
      },
      itemPath: {
        type: String,
        default: '',
      },
      pagePath: {
        type: String,
        default: '',
      },
      wrappedComponent: {
        type: Object as PropType<Component | VNode>,
        default: () => {},
      },
    },
    setup(props, context) {
      // console.log('ModelProvider properties: ', props);
      // console.log('ModelProvider attributes: ', context.attrs);
      let modelManager = ModelManager;
      if (typeof window.ModelManager !== 'undefined') {
        modelManager = window.ModelManager;
      }

      return { modelManager };
    },
    data() {
      return {
        updatedProps: {},
        updateTime: new Date().getTime(),
      };
    },
    computed: {
      computedProps() {
        return {
          ...this.$attrs,
          pagePath: this.pagePath,
          itemPath: this.itemPath,
          cqPath: this.updatedCqPath,
          ...this.updatedProps,
        };
      },
      updatedCqPath() {
        const {
          pagePath,
          itemPath,
          injectPropsOnInit,
          cqPath,
        }: {
          pagePath: string;
          itemPath: string;
          injectPropsOnInit: boolean;
          cqPath: string;
        } = this;
        return Utils.getCQPath({
          pagePath,
          itemPath,
          injectPropsOnInit,
          cqPath,
        });
      },
    },
    mounted() {
      if (this.injectPropsOnInit) {
        this.updateData(this.updatedCqPath);
      }
      this.modelManager.addListener(
        this.updatedCqPath,
        this.updateDataListener
      );
    },
    unmounted() {
      this.modelManager.removeListener(
        this.updatedCqPath,
        this.updateDataListener
      );
    },
    methods: {
      async updateData(cqPath: string) {
        const {
          pagePath,
          itemPath,
          injectPropsOnInit,
        }: {
          pagePath: string;
          itemPath: string;
          injectPropsOnInit: boolean;
        } = this;

        const path =
          cqPath ||
          this.cqPath ||
          (this.pagePath &&
            Utils.getCQPath({ pagePath, itemPath, injectPropsOnInit }));

        if (path) {
          this.modelManager
            .getData({ path, forceReload: this.cqForceReload })
            .then((data) => {
              if (data && Object.keys(data).length > 0) {
                Object.assign(this.updatedProps, Utils.modelToProps(data));
                this.updateTime = new Date().getTime();

                // Fire event once component model has been fetched and rendered to enable editing on AEM
                if (injectPropsOnInit && Utils.isInEditor()) {
                  PathUtils.dispatchGlobalCustomEvent(
                    'cq-async-content-loaded',
                    {}
                  );
                }
              }
            })
            .catch((error) => {
              console.log(error);
            });
        }
      },
      updateDataListener() {
        this.updateData(this.updatedCqPath);
      },
    },
  });
</script>
