<template>
  <section :class="{ 'search-grid': true, 'search-grid__contain-images': shouldContainImages }"
           ref="searchGrid">
    <div class="search-grid_ctr" ref="gridItems">
      <div v-show="!isFetchingImages && includeAnalytics" class="search-grid_analytics" >
        <h2>{{ searchTerm }}</h2>
        <span>{{ _imagesCount }} photos</span>
      </div>
      <div
        class="masonry-layout"
        v-masonry transition-duration="0s"
        item-selector=".item"
        :fit-width="true"
        :gutter="20">
        <masonry-search-grid-cell v-for="(image) in _images"
          v-masonry-tile class="item"
          :key="image.id"
          :image="image"
          :shouldContainImage="shouldContainImages" />
      </div>
      <div class="load-more">
        <button v-show="!isFetchingImages && includeAnalytics"
                class="clear button"
                type="button"
                :disabled="isFinished"
                @click="onLoadMoreImages">
          <p v-if="isFinished">No more images :(</p>
          <p v-else>Load more</p>
        </button>
        <loading-icon v-show="isFetchingImages" />
      </div>
      <div class="search-grid_notification callout alert" v-if="isFetchingImagesError">
        <h5>Error fetching images</h5>
      </div>
    </div>
  </section>
</template>

<script>
import Vue from 'vue';
import { SET_IMAGES } from '@/store/mutation-types';
import MasonrySearchGridCell from '@/components/MasonrySearchGridCell';
import SearchGridFilter from '@/components/SearchGridFilter';
import LoadingIcon from '@/components/LoadingIcon';
import { VueMasonryPlugin } from 'vue-masonry';

Vue.use(VueMasonryPlugin);

const DEFAULT_PAGE_SIZE = 20;

export default {
  name: 'search-grid-manual-load',
  components: {
    SearchGridFilter,
    MasonrySearchGridCell,
    LoadingIcon,
  },
  data: () => ({
    isDataInitialized: false,
    shouldContainImages: false,
    currentPage: 1,
  }),
  props: {
    imagesCount: 0,
    images: {
      default: () => ([]),
    },
    query: {},
    useInfiniteScroll: {
      default: true,
    },
    includeAnalytics: {
      default: true,
    },
    includeAddToList: {
      default: true,
    },
    searchTerm: {
      default: '',
    },
  },
  computed: {
    imagePage() {
      return this.$store.state.imagePage;
    },
    isFetchingImagesError() {
      return this.$store.state.isFetchingImagesError;
    },
    isFetchingImages() {
      return this.$store.state.isFetchingImages;
    },
    _images() {
      return this.useInfiniteScroll ? this.$store.state.images : this.images;
    },
    _imagesCount() {
      const count = this.useInfiniteScroll ? this.$store.state.imagesCount : this.imagesCount;
      return count.toLocaleString('en');
    },
    _query() {
      return this.$props.query;
    },
    isFinished() {
      return this.currentPage >= this.$store.state.pageCount;
    },
  },
  watch: {
    _images: {
      handler() {
        if (this.$state) {
          this.$state.loaded();
          if (this._imagesCount < this.currentPage * DEFAULT_PAGE_SIZE) {
            this.$state.complete();
          }
        }
        this.isDataInitialized = true;
      },
    },
    _query: {
      handler() {
        this.searchChanged();
      },
      deep: true,
    },
  },
  methods: {
    handleScalingChange() {
      setTimeout(() => {
        this.$redrawVueMasonry(); // Some elements end up taking less space
      }, 100); // One-tenth of a second should be sufficient to calculate new height
    },
    searchChanged() {
      this.$store.commit(SET_IMAGES, { images: [] });
      this.currentPage = 1;
    },
    onLoadMoreImages() {
      if (this.isFetchingImages === false) {
        this.currentPage = this.currentPage + 1;
        const searchParams = {
          page: this.currentPage,
          shouldPersistImages: true,
          ...this._query,
        };

        this.$emit('onLoadMoreImages', searchParams);
      }
    },
  },
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style lang="scss">

  .button[disabled] {
    opacity: 1;
  }

  .search-grid_analytics h5,
  .search-grid_layout-control h5 {
    padding-top: 1.36vh;
    font-size: 1rem;
    display: inline-block;
  }

  .search-grid_layout-control h5 {
    margin-right: 10px;
  }

  .search-grid_layout-control {
    text-align: right;

    fieldset {
      display: inline;
      margin-right: 5px;
    }
  }

  .infinite-loading-container {
    margin-top: 30px;
    width: 100%;
  }

  .search-grid_ctr {
    .masonry-layout {
      margin: auto;
      margin-top: 1rem;
    }

    .item {
      width: 320px;
      margin-bottom: 20px;
    }
  }

  .search-grid:after {
    content: '';
    display: block;
    clear: both;
  }

  .search-grid_notification {
    width: 50%;
    margin: auto;
    font-weight: 500;
    text-align: center;
  }

  label {
    color: #2c3e50;
  }

  h2 {
    font-size: 2rem;
  }

  .load-more {
    text-align: center;

    button {
      font-size: 1.2em;
    }
  }
</style>
