<template>

    <div class="modal-content fm-modal-preview">
        <div class="modal-header">
            <h5 class="modal-title w-75 text-truncate">
                {{ showCropperModule ? lang.modal.cropper.title : lang.modal.preview.title }}
                <small class="text-muted pl-3">{{ selectedItem.basename }}</small>
            </h5>
            <button type="button" class="close" aria-label="Close" v-on:click="hideModal">
                <span aria-hidden="true">&times;</span>
            </button>
        </div>
        <div class="modal-body text-center">
            <template v-if="selectedItem.extension == 'pdf'">
                <div class="wrapper">
                    <input v-model.number="page" type="number" style="width: 5em"> /{{numPages}}
                    <button @click="rotate += 90">&#x27F3;</button>
                    <button @click="rotate -= 90">&#x27F2;</button>
                    <button @click="$refs.pdf.print()">print</button>
                    <div style="width: 90%">
                        <div v-if="loadedRatio > 0 && loadedRatio < 1" style="background-color: green; color: white; text-align: center" :style="{ width: loadedRatio * 100 + '%' }">{{ Math.floor(loadedRatio * 100) }}%</div>
                        <pdf v-if="show" ref="pdf" style="border: 1px solid red" :src="src" :page="page" :rotate="rotate" @password="password" @progress="loadedRatio = $event" @error="error" @num-pages="numPages = $event" @link-clicked="page = $event"></pdf>
                    </div>
<!--                    <pdf
                            v-for="i in numPages"
                            :key="i"
                            :src="src"
                            :page="i"
                            style="height: 100%"
                    ></pdf>-->
                </div>

            </template>
            <template v-else-if="showCropperModule">
                <cropper-module v-bind:imgUrl="imgUrl"
                                v-bind:maxHeight="maxHeight"
                                v-on:closeCropper="closeCropper"></cropper-module>
            </template>
            <template v-else>
                <img v-bind:src="imgUrl"
                     v-bind:alt="selectedItem.basename"
                     v-bind:style="{'max-height': maxHeight+'px'}">
            </template>
        </div>
        <div v-if="showFooter" class="d-flex justify-content-between">
            <span class="d-block">
                <button class="btn btn-info"
                        v-bind:title="lang.modal.cropper.title" v-on:click="showCropperModule = true">
                    <i class="fas fa-crop-alt"></i>
                </button>
            </span>
            <span class="d-block">
                <button class="btn btn-default" v-on:click="hideModal">{{ lang.btn.cancel }}</button>
            </span>
        </div>
    </div>
</template>

<script>
import CropperModule from './../additions/Cropper.vue';
import modal from './../mixins/modal';
import translate from './../../../mixins/translate';
import helper from './../../../mixins/helper';
import pdf from 'vue-pdf'

export default {
  name: 'Preview',
  mixins: [modal, translate, helper],
  components: { CropperModule ,pdf},
  data() {
    return {
        show:true,
        loadedRatio: 0,
        page: 1,
        numPages: 0,
        rotate: 0,
      showCropperModule: false,
      imgUrl: '',
        numPages: undefined,
        src:null,
        srcItem:null,
        currentPage: 0,
        pageCount: 0
    };
  },
mounted() {
    if (this.srcItem) {

        this.src= pdf.createLoadingTask(this.srcItem.durl);
        this.src.then(pdf => {

            this.numPages = pdf.numPages;
        });
    }

},
  created() {
    // Create image URL

          this.setImgUrl();


  },
  computed: {
    /**
     * Selected disk
     * @returns {*}
     */
    selectedDisk() {
      return this.$store.getters['fm/selectedDisk'];
    },

    /**
     * Selected file
     * @returns {*}
     */
    selectedItem() {
        this.srcItem = this.$store.getters['fm/selectedItems'][0];
      return this.$store.getters['fm/selectedItems'][0];
    },

    /**
     * Show modal footer
     * @return boolean
     */
    showFooter() {
      return this.canCrop(this.selectedItem.extension) && !this.showCropperModule;
    },

    /**
     * Calculate the max height for image
     * @returns {number}
     */
    maxHeight() {
      if (this.$store.state.fm.modal.modalBlockHeight) {
        return this.$store.state.fm.modal.modalBlockHeight - 170;
      }

      return 300;
    },
  },
  methods: {
    /**
     * Can we crop this image?
     * @param extension
     * @returns {boolean}
     */
    canCrop(extension) {
      return this.$store.state.fm.settings.cropExtensions.includes(extension.toLowerCase());
    },

    /**
     * Set image URL
     */
    setImgUrl() {
      this.imgUrl = `${this.selectedItem.purl}`;
    },

    /**
     * Close cropper
     */
    closeCropper() {
      this.showCropperModule = false;
      this.setImgUrl();
    },
      password: function(updatePassword, reason) {

          updatePassword(prompt('password is "test"'));
      },
    error: function(err) {

        console.log(err);
    }
  },
};
</script>

<style lang="scss">
    .wrapper{
        overflow-y: auto;
        height: 100%;
    /*    position:fixed;*/


    }
    .fm-modal-preview {
z-index: 9999;
        .modal-body {
            padding: 0;

            img {
                max-width: 100%;
            }
        }

        & > .d-flex {
            padding: 1rem;
            border-top: 1px solid #e9ecef;
        }
    }
</style>
