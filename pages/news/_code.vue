<template>
  <Page :title="pageTitle" title-position="center" back-route="/news" :breadcrumbs="breadcrumbs">
    <div slot="content" class="news-detail">
      <div class="format-html" v-html="dataCode.news.DETAIL_TEXT"></div>
      <div class="news-detail__img"><img :src="dataCode.news.DETAIL_PICTURE" alt=""></div>
      <div class="news-detail__info">
        <div class="news-detail__info__date"></div>
        <div class="news-detail__info__share"><ShareBtn icon-color="#9F9F9F" class="text-big text-dark-grey"/></div>
      </div>
      <div class="news-detail__greeting">
        <FoxSvg/>
        <div class="news-detail__greeting__text">Мы всегда рады видеть вас в нашем магазине</div>
      </div>
      <LinkButton
        text="Все новости"
        route="/news/"
      >
        <span slot="beforeText"><i class="el-icon-back"></i></span>
        {{ dataCode }}
      </LinkButton>
    </div>
  </Page>
</template>

<script lang="ts">
import Page from '../../components/common/Page.vue'
import LinkButton from '../../components/LinkButton.vue'
import ShareBtn from '../../components/product/ShareBtn.vue'
import FoxSvg from '../../components/svg/FoxSvg.vue'

import { $newsService } from '@/utils/service-accessor'
import { IMeta, INewsItemCode } from '~/ts-entities/';

import { getModule } from "vuex-module-decorators";
import CityModule from "~/store/city";
import { CommonPageSeo } from '@/utils/page-seo'

import { Component, Vue } from 'nuxt-property-decorator';
import { Context } from '@nuxt/types';

@Component({
  name: "NewsItemPage",
  components: { Page, ShareBtn, LinkButton, FoxSvg },
})

export default class Usersid extends Vue {
  public readonly metaTitle!: string
  public readonly meta!: IMeta[]
  public readonly pageTitle!: string
  public readonly breadcrumbs!: []
  public readonly dataCode!: INewsItemCode

  head () {
    return {
      title: this.metaTitle,
      meta: this.meta
    }
  }

  async asyncData ({ params, error, store }: Context) {
    const code = params.code
    const dataCode = await $newsService.getCode(code);

    const breadcrumbs = [
      { route: '/', label: 'Главная' },
      { route: '/news/', label: 'Новости' },
      { route: `/news/${dataCode.news.CODE}/`, label: dataCode.news.NAME },
    ]

    // try {

    // } catch (e) {
    //   if (e.bitrixResponseError) {
    //     return error({ statusCode: 404, message: e.value });
    //   }
    // }

    const cityStore = getModule(CityModule, store)
    const seoUtil = new CommonPageSeo(cityStore)
    seoUtil.setEntity(dataCode.news)

    const metaTitle = seoUtil.getTitle()
    const pageTitle = seoUtil.getH1()
    const meta = seoUtil.getPageMeta()

    return { dataCode, code, breadcrumbs, metaTitle, meta, pageTitle }
  }
}

</script>

<style lang="scss" scoped>
  .news-detail {
    max-width: 680px;
    margin: 0 auto;
    padding-bottom: 60px;
    h2 {
      margin-bottom: 40px;
    }
    p {
      margin-bottom: 40px;
    }
    &__img {
      display: flex;
      img {
        width: 100%;
        max-width: 100%;
        height: auto;
        object-fit: cover;
        background-color: azure;
      }
    }
    &__info {
      margin: 20px 0 7px;
      display: flex;
      justify-content: space-between;
    }
    &__greeting {
      text-align: center;
      margin-bottom: 60px;
    }
  }

</style>
