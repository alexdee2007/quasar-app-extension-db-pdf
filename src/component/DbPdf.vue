<template>
  <div :style="{cursor: loaded && !error ? 'zoom-in' : 'default', height, width}" class="q-bordered rounded-borders">
    <div v-if="error" class="absolute-center text-red q-pb-xl">
      <q-icon name="warning" style="font-size: 2rem;" />
      <div class="text-caption q-pt-md">Помилка завантаження</div>
    </div>
    <pdf :src="pdfSrc" ref="pdf" @loaded="loaded = true" @error="onError"></pdf>
    <template>
      <slot></slot>
    </template>
    <q-inner-loading :showing="!loaded">
      <q-spinner size="50px" color="primary"></q-spinner>
    </q-inner-loading>
    <q-resize-observer @resize="onResize" />
  </div>
</template>

<script>

  import pdf from 'vue-pdf';
  import minioApi from 'src/api/minio';

  export default {
    name: 'DbPdf',
    components: {
      pdf
    },
    props: {
      height: {
        type: String,
        default: '100%'
      },
      width: {
        type: String,
        default: '100%'
      },
      url: String,
      fileName: String
    },
    data() {
      return {
        pdfSrc: '',
        loaded: false,
        error: false
      }
    },
    methods: {
      onError(obj) {
        this.loaded = true;
        this.error = true;
        this.$emit('error');
      },
      onResize(size) {
        this.$refs.pdf.$el.style.display = this.$refs.pdf.$el.style.display === 'block' ? 'inline-block' : 'block'; // redraw canvas
      },
      async getPdfSrc() {
        try {
          this.pdfSrc = await minioApi.getPresignedUrlForGet(this.fileName);
        } catch (err) {
          console.error(err);
        }
      }
    },
    mounted() {
      if (this.url) {
        this.pdfSrc = this.url;
      } else {
        this.getPdfSrc();
      }
    }
  }
</script>

<style lang="stylus">

</style>
