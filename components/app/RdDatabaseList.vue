<template>
  <div class="database-list">
    <p class="intro">
      READr
      致力於產製資料驅動的新聞報導，並將所使用的資料公開，以下為公開的資料，歡迎再加以利用！
    </p>

    <div class="list-title-wrapper">
      <span>資料列表</span>
      <span>資料怎麼用</span>
    </div>

    <ul>
      <li v-for="item in items" :key="item.id">
        <article>
          <a
            class="database-title"
            :href="item.href"
            target="_blank"
            rel="noopener noreferrer"
            @click="$emit('sendGaEvent:database')"
          >
            <h1>{{ item.title }}</h1>
          </a>

          <div class="gallery">
            <span v-if="item.writerName">{{ item.writerName }}這樣用</span>
            <span v-else>分享你怎麼用</span>

            <a
              v-for="gallery in item.galleries"
              :key="gallery.id"
              :href="gallery.href"
              target="_blank"
              rel="noopener noreferrer"
              @click="$emit('sendGaEvent:portfolio')"
            >
              <img v-lazy="gallery.img" alt="" />
            </a>

            <a
              class="add"
              href="https://forms.gle/2JKrGUfherYagj3P6"
              target="_blank"
              rel="noopener noreferrer"
            >
              <RdPlusIcon />
            </a>
          </div>
        </article>
      </li>
    </ul>

    <RdButtonLoadMore
      v-if="shouldLoadMore"
      :loadMore="loadMore"
      class="database-list__btn-load-more"
      @click.native="$emit('sendGaEvent:loadMore')"
    />
  </div>
</template>

<script>
import RdPlusIcon from '~/components/shared/RdPlusIcon.vue'
import RdButtonLoadMore from '~/components/shared/Button/RdButtonLoadMore.vue'

export default {
  name: 'RdDatabaseList',

  components: {
    RdPlusIcon,
    RdButtonLoadMore,
  },

  props: {
    items: {
      type: Array,
      required: true,
      default: () => [],
    },
    loadMore: {
      type: Function,
      required: true,
    },
    shouldLoadMore: {
      type: Boolean,
      required: true,
      default: false,
    },
  },
}
</script>

<style lang="scss" scoped>
.database-list {
  font-size: 15px;
  line-height: 1.5;
  color: #000928;
  padding-top: 16px;
  @include media-breakpoint-up(md) {
    font-size: 18px;
    padding-top: 30px;
  }
  &__btn-load-more {
    margin-top: 8px;
    @include media-breakpoint-up(md) {
      margin-top: 10px;
    }
  }
}
h1,
.intro {
  font-weight: 700;
  letter-spacing: 2.5px;
}
.intro {
  margin-bottom: 18px;
  @include media-breakpoint-up(md) {
    text-align: center;
    margin-bottom: 22px;
  }
}
.list-title-wrapper {
  font-size: 13px;
  line-height: 1.4;
  margin-bottom: 4px;
  @include media-breakpoint-up(md) {
    font-size: 15px;
    margin-bottom: 8px;
    display: flex;
    justify-content: space-between;
  }
  > span:last-child {
    display: none;
    @include media-breakpoint-up(md) {
      display: inline;
    }
  }
}
ul {
  border-top: 1px solid rgba(#000, 0.1);
}
li {
  border-bottom: 1px solid rgba(#000, 0.1);
  padding-top: 10px;
  padding-bottom: 10px;
}
article {
  @include media-breakpoint-up(md) {
    display: flex;
    justify-content: space-between;
    align-items: center;
  }
}
.database-title {
  display: inline-block;
  margin-bottom: 18px;
  @include media-breakpoint-up(md) {
    margin-bottom: 0;
  }
}
h1 {
  text-decoration: underline;
}
.gallery {
  display: flex;
  justify-content: flex-end;
  align-items: center;
  @include media-breakpoint-up(md) {
    flex: 0 0 auto;
  }
  span {
    margin-right: 10px;
    font-size: 13px;
    line-height: 1.4;
  }
  a {
    border-radius: 50%;
    width: 36px;
    height: 36px;
    position: relative;
    overflow: hidden;
    @include media-breakpoint-up(md) {
      width: 44px;
      height: 44px;
    }
    + a {
      margin-left: 6.5px;
      @include media-breakpoint-up(md) {
        margin-left: 18px;
      }
    }
  }
  img {
    position: absolute;
    width: 100%;
    height: 100%;
    top: 0;
    left: 0;
    object-fit: cover;
  }
}
.add {
  background-color: #f5ebff;
  display: flex;
  justify-content: center;
  align-items: center;
}
</style>
