<template>
  <div class="main">
    <div
      class="scanner-wrapper"
      v-loading="!productsLoaded"
      element-loading-text="Loading products..."
    >
      <model-scanner
        :products="products"
        v-if="productsLoaded"
        @found="setAssembly"
        @reset="resetAssembly"
      />
    </div>

    <div
      class="col-xs-12"
      style="margin-top: 20px"
      v-if="assembly.model"
    >
      <parts-checklist
        :parts="assembly.parts"
        @complete="resetPage"
      />
    </div>
  </div>
</template>

<script>
import { mapGetters, mapActions } from 'vuex';
import { Howl } from 'howler';
import { triggerFailure, triggerSuccess } from '../helpers/util';

const successSound = new Howl({
  src: ['/sound/success.webm', '/sound/success.mp3'],
  volume: 0.5,
});

export default {
  name: 'AppDashboard',

  components: {
    Part: () => import('./Part'),
    ModelScanner: () => import('./ModelScanner'),
    PartsChecklist: () => import('./PartsChecklist'),
  },

  data() {
    return {
      assembly: {},
      partModels: [],
      partScanned: {},
    };
  },

  computed: {
    ...mapGetters('products', [
      'products',
      'productsLoaded',
    ]),
  },

  methods: {
    ...mapActions('products', [
      'fetchProducts',
    ]),
    setAssembly(assembly) {
      this.assembly = assembly;
    },
    resetAssembly() {
      this.assembly = {};
      this.$eventHub.$emit('resetBarcode');
      this.$eventHub.$emit('resetSerial');
    },
    enterPartModel(index) {
      const foundPart = this.parts.find(p => p.id === this.partModels[index]);
      const fpIndex = this.parts.indexOf(foundPart);

      if (fpIndex !== -1) {
        this.parts.splice(fpIndex, 1);
      }

      if (foundPart) {
        this.$set(this.partScanned, index, true);
        // noinspection JSUnresolvedVariable
        this.$set(this.partErrors, index, false);
        this.$nextTick(() => {
          const nextPart = document.querySelector(`#part-model-input${index + 1}`);

          if (this.parts.length === 0 && Object.values(this.partScanned).every(k => k)) {
            successSound.play();
            triggerSuccess();
          } else if (nextPart) {
            nextPart.select();
          }
        });

        return;
      }

      const foundDisplayPart = this.product.parts.find(p => p.id === this.partModels[index]);
      if (foundDisplayPart) {
        triggerFailure('Duplicate detected', 'This part has been already scanned!', 2000);
      } else {
        triggerFailure();
      }
      // noinspection JSUnresolvedVariable
      this.$set(this.partErrors, index, true);
      document.querySelector(`#part-model-input${index}`).select();
    },
    resetPage() {
      this.resetAssembly();
    },
  },

  created() {
    this.fetchProducts();
  },
};
</script>

<style scoped>
  .scanner-wrapper {
    min-height : 35px;
  }

  .unauthorized {
    padding-top : 20px;
  }
</style>

<style>
  /*noinspection CssUnusedSymbol*/
  .el-loading-spinner {
    top : 30%;
  }
</style>

