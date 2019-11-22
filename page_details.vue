<template>
    <main> <!-- without an outer container div this component template will not render -->
        <loading-spinner v-if="!dataLoaded"></loading-spinner>
        <transition name="fade">
            <div v-if="dataLoaded" v-cloak>
                 <div class="inside_page_header">
                    <div class="main_container position_relative">
                        <h1 v-html="currentPage.title"></h1>
                    </div>
                </div>
                <div class="main_container">
                    <div class="row">
                        <div class="col-md-12">
                            <breadcrumb></breadcrumb>
                        </div>
                    </div>
                    <div class="row margin_60">
                        <div class="col-md-12">
                            <div class="page_body" v-html="currentPage.body"></div>
                        </div>
                    </div>
                </div>
            </div>
        </transition>
    </main>
</template>

<script>
    define(["Vue", "vuex", "vue-meta"], function (Vue, Vuex, Meta) {
        return Vue.component("page-details-component", {
            template: template, // the variable template will be injected,
            props: ['id'],
            data: function data() {
                return {
                    dataLoaded: false,
                    currentPage: null,
                    meta: {
                        meta_title: "",
                        meta_description: "",
                        meta_keywords: "",
                        meta_image: ""
                    }
                }
            },
            created() {
                this.updateCurrentPage(this.id);
            },
            watch: {
                $route: function () {
                    this.updateCurrentPage(this.$route.params.id);
                }
            },
            computed: {
                ...Vuex.mapGetters([
                    'property',
                    'findMetaDataByPath'
                ])
            },
            methods: {
                updateCurrentPage(id) {
                    this.property.mm_host = this.property.mm_host.replace("http:", "");
                    var _this = this;
                    this.$store.dispatch('LOAD_PAGE_DATA', { url: this.property.mm_host + "/pages/" + this.id + ".json" }).then(function (response) {
                        _this.currentPage = response.data;
                        _this.$breadcrumbs[0].meta.breadcrumb = _this.currentPage.title
                        _this.meta = _this.findMetaDataByPath(this.$route.path);
                        _this.dataLoaded = true;
                    }, function (error) {
                        console.error( "Could not retrieve data from server. Please check internet connection and try again.");
                        _this.$router.replace({ name: '404' });
                    });
                }
            },
            metaInfo () {
                return {
                    title: this.meta.meta_title,
                    meta: [
                        { name: 'description', vmid: 'description', content: this.meta.meta_description },
                        { name: 'keywords',  vmid: 'keywords', content: this.meta.meta_keywords },
                        { property: 'og:title', vmid: 'og:title', content: this.meta.meta_title },
                        { property: 'og:description', vmid: 'og:description', content: this.meta.meta_description },
                        { property: 'og:image', vmid: 'og:image', content: this.meta.meta_image }
                    ] 
                }
            }
        });
    });
</script>