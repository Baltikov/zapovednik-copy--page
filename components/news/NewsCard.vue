<template>
  <div class="newss" >
    <nuxt-link v-if="code" :to="`/news/${code}/`"> <div class="newss__preview"><img :src="news.PREVIEW_PICTURE" alt=""> </div> </nuxt-link>
    <nuxt-link v-else-if="!code" :to="`/news/${news.CODE}/`"> <div class="newss__preview"><img :src="news.PREVIEW_PICTURE" alt=""> </div> </nuxt-link>
    {{ news.NAME }}
    <div class="newss__link"><slot name="news"></slot></div>
  </div>
</template>

<script lang="ts">
import { Vue, Component, Prop } from 'nuxt-property-decorator';
import { INewsitem } from '~/ts-entities/';

@Component({
  name: 'NewsCard',
})
export default class NewsItem extends Vue {
 @Prop({ type: Object, required: true }) readonly news: INewsitem
 @Prop({ type: String }) readonly code: string
}

</script>

<style lang="scss" scoped>

@import '/assets/scss/var';

.news__block {
  .newss {
    &__preview {
      img {
        width: 450px;
        height: 260px;

        @media screen and (max-width: 1200px) {
          width: 100%;
          height: 330px;
        }

        @media screen and (max-width: 768px) {
          height: 280px;
        }

        @media screen and (max-width: 595px) {
          height: 200px;
        }

        @media screen and (max-width: 420px) {
          width: 280px;
          height: 160px;
        }
      }
    }
  }
}

.newss {
  display: flex;
  flex-direction: column;
  box-sizing: border-box;
  &__preview {
    display: flex;
    align-items: center;
    justify-content: center;
    margin-bottom: 10px;
    overflow: hidden;
    background: rgb(238, 235, 235);
    img {
      width: 100%;
      height: 330px;

      @media screen and (max-width: 1320px) {
        height: 260px;
      }

      @media screen and (max-width: 1070px) {
        height: 320px;
      }

      @media screen and (max-width: 768px) {
        height: 280px;
      }

      @media screen and (max-width: 595px) {
        height: 200px;
      }

      @media screen and (max-width: 420px) {
        height: 160px;
      }
    }
  }
  &__details {
    margin-top: 20px;
    text-decoration:none;
    color: $branded;
  }
  &__link {
    padding-top: 15px;
  }

  @media screen and (max-width: 1320px) {
    &__preview {
        img {
        width: 100%;
      }
    }
  }

  @media screen and (max-width: 595px) {
    width: 350px;
    &__preview {
        img {
        width: 100%;
      }
    }
  }

  @media screen and (max-width: 420px) {
    width: 280px;
    &__preview {
      img {
        min-height: 145px;
      }
    }
  }
}

</style>
