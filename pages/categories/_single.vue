<template>
  <div id="category-page" class="page-wrapper category-page">
    <site-hero
      :title="getTitle"
      :subtitle="getSubTitle"
      :image="getImage"
    />
    <div @click="handleUpdate">
      <main-section theme="sidebar-right">
        <template v-slot:default>
          <!-- Posts in Category -->
          <posts-grid :category="articles" :id="getCategory" :per-row="2"/>
        </template>
        <template v-slot:sidebar>
          <h3 class="subtitle">
            カテゴリ
          </h3>
          <div class="panel">
            <nuxt-link
              v-for="cat in mainMenu"
              :key="cat.title"
              :to="`/categories/${cat.slug}`"
              :class="{
              'panel-block': true,
              'is-active': cat.slug === getActive
            }"
            >
              {{ cat.title }}
            </nuxt-link>
          </div>
        </template>
      </main-section>
    </div>
  </div>
</template>
<script>
import {mapGetters, mapActions} from "vuex";
import {getLoadIcon} from '@/helpers/notication';

export default {
  data() {
    return {
      title: '',
      subtitle: '',
      image: '',
    }
  },
  head() {
    return {
      title: `${this.title} | ${this.$siteConfig.siteName}`
    }
  },
  fetchOnServer: false,
  async fetch({params, store, error}) {
    await store.dispatch('actMainMenus');
    const slug = (params.single)?params.single:'';
    const res = await store.dispatch('category/actGetCategoryBySlug', {slug});
    if (res.ok) {
      await store.dispatch('posts/actFetchArticlesList', {
        categories: [res.data.id] // lấy bài viết có id của categories
      });
    } else {
      return error({statusCode: 402, message: 'Post not found'})
    }

  },

  computed: {
    ...mapGetters({
      articles       : 'posts/getArticleList',
      category       : 'category/getSlugCategories',
      mainMenu       : 'mainMenuItems',
    }),
    getTitle(){
      if(this.title){
        return this.title
      }else{
        return this.$siteConfig.siteName
      }
    },
    getActive(){
      return  this.$route.params.single
    },
    getCategory(){
      return  this.category
    },
    getSubTitle(){
      if(this.subtitle){
        return this.subtitle
      }else{
        return this.$siteConfig.tagline
      }
    },
    getImage(){
      if(this.image){
        return this.image
      }else{
        return this.$siteConfig.featureImage
      }
    }
  },
  methods: {
    ...mapActions({
      setLoading: 'setLoading'
    }),
    handleUpdate(e) {
      //  bật loading
      getLoadIcon(this, 3000);
    },
  },
  created() {
    this.mainMenu.filter(value => {
      if (value.slug === this.$route.params.single) {
        this.title     = value.title;
        this.image     = value.image;
        this.subtitle  = value.description;
      }
    })
  }
}
</script>
<style>
.panel-block.is-active {
  background-color: #00d1b285;
}
</style>
