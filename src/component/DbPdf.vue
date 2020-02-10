<template>
    <div :style="{cursor: loaded ? 'zoom-in' : 'default', height, width}" class="q-bordered rounded-borders">
        <pdf :src="pdfSrc" ref="pdf" @loaded="loaded = true"></pdf>
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
                loaded: false
            }
        },
        methods: {
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
