<template>
  <ul :class="{ 'lg-layout': shouldSetLgBreakPoint }">
    <li v-for="item in formatedPosts" :key="item.id">
      <RdArticleListCard
        :href="item.href"
        :img="item.img.src"
        :title="item.title"
        :date="item.date"
        :readTimeText="item.readTime"
        :isReport="item.isReport"
        :shouldReverseInMobile="shouldReverseInMobile"
        :shouldHighLightReport="shouldHighLightReport"
        :shouldHideBottomInfos="shouldHideBottomInfos"
      />
    </li>
    <div v-if="!isLoading" class="position-correct" />
    <template v-if="shouldShowSkeleton && isLoading">
      <RdSkeleton v-for="n in 4" :key="`skeleton${n}`" class="skeleton-item" />
    </template>
  </ul>
</template>

<script>
import RdArticleListCard from '~/components/shared/RdArticleListCard.vue'
import RdSkeleton from '~/components/shared/RdSkeleton.vue'

export default {
  name: 'RdArticleList',

  components: {
    RdArticleListCard,
    RdSkeleton,
  },
  props: {
    posts: {
      type: Array,
      required: true,
      default: () => [],
    },
    filterNum: {
      type: Number,
      default: 0,
    },
    shouldReverseInMobile: {
      type: Boolean,
      default: false,
    },
    shouldHighLightReport: {
      type: Boolean,
      default: false,
    },
    shouldShowSkeleton: {
      type: Boolean,
      default: false,
    },
    shouldSetLgBreakPoint: {
      type: Boolean,
      default: false,
    },
    shouldHideBottomInfos: {
      type: Boolean,
      default: false,
    },
    isLoading: {
      type: Boolean,
      default: false,
    },
  },
  computed: {
    formatedPosts() {
      return this.filterNum
        ? this.posts?.filter((item, i) => i < this.filterNum) ?? []
        : this.posts
    },
  },
}
</script>

<style lang="scss" scoped>
ul {
  width: 100%;
  display: flex;
  flex-wrap: wrap;
  justify-content: space-between;
  &::after {
    content: '';
  }
  li,
  &::after,
  .position-correct,
  .skeleton-item {
    width: 100%;
    @include media-breakpoint-up(sm) {
      width: calc((100% - 24px) / 2);
    }
    @include media-breakpoint-up(xl) {
      width: calc((100% - 72px) / 4);
    }
  }
  li,
  .skeleton-item {
    margin: 0 0 16px;
    @include media-breakpoint-up(sm) {
      margin: 0 0 32px;
    }
    @include media-breakpoint-up(lg) {
      margin: 0 0 60px;
    }
  }
  .position-correct {
    margin: 0;
  }
  &.lg-layout {
    @include media-breakpoint-up(lg) {
      li,
      &::after,
      .position-correct,
      .skeleton-item {
        width: calc((100% - 48px) / 3);
      }
    }
    @include media-breakpoint-up(xl) {
      li,
      &::after,
      .position-correct,
      .skeleton-item {
        width: calc((100% - 72px) / 4);
      }
    }
  }
}
</style>
