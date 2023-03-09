<template>
  <Page :title="pageTitle" :breadcrumbs="breadcrumbs">
    <div slot="content" class="news">
      <div class="news__block">
        <div
          v-for="(newsitem, index) in dataNews.news"
          :key="index"
          class="news__item"
        >
          <NewsCard :news="newsitem" :code="newsitem.CODE">
            <template slot="news">
              <div class=" mb"> Дата публикации: {{ newsitem.DATE_CREATE.split(' ')[0] }}</div>
              <nuxt-link class="text-big text-branded text-bold" :to="`/news/${newsitem.CODE}/`"> Подробнее </nuxt-link>
            </template>
          </NewsCard>
        </div>
      </div>

      <Pagination
        v-if="dataNews.news"
        :count="dataNews.page_count"
        :current="currentPage"
        class="news__pagination"
        @page-change="paginationHandler"
      />
    </div>
  </Page>
</template>

<script lang="ts">

import NewsCard from '../../components/news/NewsCard.vue'
import Page from '../../components/common/Page.vue'
import Pagination from '../../components/products/Pagination.vue'
import { $newsService } from '@/utils/service-accessor'
import { CommonPage } from "~/models/ts/";
import { IMeta } from '~/ts-entities/';
import { ICity } from '~/ts-entities/city'

import { Vue, Component } from 'nuxt-property-decorator';
import { Context } from '@nuxt/types';
import { NewsResponse } from '~/ts-entities';

import { getModule } from "vuex-module-decorators";
import CityModule from "~/store/city";
import { CommonPageSeo } from '@/utils/page-seo'

@Component({
  name: 'News',
  components: { Page, NewsCard, Pagination },
  watchQuery: true,
})
export default class NewsListPage extends Vue {
  dataNews!: NewsResponse
  currentPage!: number
  public readonly metaTitle!: string
  public readonly meta!: IMeta[]
  public readonly pageTitle!: string
  public readonly href!: string

  head () {
    return {
      title: this.metaTitle,
      meta: this.meta,
      link: [{ rel: 'canonical', href: this.href }]
    }
  }

  breadcrumbs = [
    { route: '/', label: 'Главная' },
    { route: '/news/', label: 'Новости' },
  ]

  async asyncData ({ query, store, route }: Context) {
    let currentPage

    if (query.PAGEN_1) {
      currentPage = parseInt(query.PAGEN_1.toString())
    } else {
      currentPage = 1
    }

    const dataNews = await $newsService.getList(currentPage);

    const cityStore = getModule(CityModule, store)
    const seoUtil = new CommonPageSeo(cityStore)

    const title = new CommonPage()
    title.setTitle('Новости')
    seoUtil.setEntity(title)
    seoUtil.setPage(currentPage)

    const metaTitle = seoUtil.getTitle()
    const pageTitle = seoUtil.getH1()
    const meta = seoUtil.getPageMeta()
    const href = seoUtil.getCanonical(route)

    return { dataNews, currentPage, metaTitle, meta, pageTitle, href }
  }

  paginationHandler (val: number) {
    const query = { ...this.$route.query };
    if (val === 1) {
      delete query.PAGEN_1;
    } else {
      query.PAGEN_1 = val.toString()
    }
    this.$router.replace({ path: this.$route.path, query })
  }
}

</script>

<style lang="scss" scoped>

@import '/assets/scss/var';

  .news {
    display: flex;
    flex-direction: column;
    margin: 0 auto;
    max-width: 1080px;
    &__block {
      display: flex;
      flex-wrap: wrap;
      justify-content: center;

      .news__item {
        margin-bottom: 60px;
        &:nth-child(2n + 1) {
          margin-right: 80px;
        }
      }
    }
    &__pagination {
      align-self: flex-start;
      margin-left: 32px;
    }
  }
  .mb {
    margin-bottom: 5px;
    padding-top: 5px;
    font-weight: 400;
    font-size: 16px;
    line-height: 130%;
    color: #9F9F9F;
  }
  @media screen and (max-width: 1320px) {
       .news__block {
        align-items: center;
      }
    }

  @media screen and (max-width: 1200px) {
    .news {
      &__block {
        flex-direction: column;

        .news__item {
          @media screen and (max-width: 1200px) {
            &:nth-child(2n + 1) {
              margin-right: 0;
            }
          }
          @media screen and (max-width: 420px) {
            width: 100%;
          }
        }
      }
    }
  }

  @media screen and (max-width: $tabletDevice) {
    .news {
      max-width: 100%;
      display: flex;
      align-items: center;
      &__item {
        margin: 0 0 60px;
      }
      &__pagination {
        margin-left: 32px;
      }
    }
  }
  @media screen and (max-width: 570px) {
    .news__pagination {
      margin-left: 0px;
    }
  }
</style>
