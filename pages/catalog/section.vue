<template>
  <CatalogPageMaster
    :products="products"
    :more-products-loader="moreProductsLoader"
    :products-list-loader="productsListLoader"
    :product-count="productCount"
    :description="description"
    :filter-tags="filterTags"
    :sort-items="sortItems"
    :viewed-products-counter="viewedProductsCounter"
    :breadcrumbs="breadcrumbs"
    :smart-filter="smartFilter"
    :active-sort="activeSort"
    :page-title="pageTitle"
    :active-menu-name="activeMenuName"
    :page-count="pageCount"
    :current-page="currentPage"
    :per-page="perPage"
    :set-sort="setSort"
    :pagination-handler="paginationHandler"
    :load-more-products="loadMoreProducts"
    :change-per-page="changePerPage"
    :on-update-product-list="onUpdateProductList"
    :tags="tags"
  />
</template>

<script lang="ts">

import catalogMiddleware from "~/middleware/catalogMiddleware";
import CatalogPageMixin from "~/mixins/ts/catalogPage.mixin";
import { Component, mixins, Provide, ProvideReactive } from "nuxt-property-decorator";
import CatalogPageMaster from "~/components/CatalogPageMaster.vue";
import { Context } from "@nuxt/types";
import {
  IBreadcrumb,
  IFilterValue, IMeta, ProductList,
  ProductListParams, ProductSortItem,
  ProductsResponse,
  RequestFilter,
  SortOrder,
  ITags
} from "~/ts-entities";

import {
  CatalogPageServiceFactory,
  PageRoute,
  Filter,
  IProductsPageService
} from '~/utils/catalog-page-accessor/'

import { getModule } from "vuex-module-decorators";
import CityModule from "~/store/city";
import CatalogFilterModule from "~/store/catalogFilter";
import { FilteredSectionPageSeo, SectionPageSeo } from '@/utils/page-seo'

import { seoTitle } from "@/helpers/breadcrumbs.js"
import CharityPageTargetMixin from "~/mixins/ts/CharityPageTarget.mixin";
import { ICatalogView } from "~/utils/mindbox";

interface ICatalogSectionData {
  pageCount: number
  products: ProductList
  description?:string
  filterTags: IFilterValue[]
  productCount: number
  activeSort: ProductSortItem
  loadMoreCounter: null
  viewedProductsCounter: number
  meta?: IMeta[]
  pageTitle: string
  metaTitle?: string
  productsListLoader: boolean
  breadcrumbs: IBreadcrumb[]
  tags?: ITags
  canonicalHref?: string
}

@Component({
  name: 'CatalogPage',
  components: { CatalogPageMaster },
  middleware: [catalogMiddleware],
  watchQuery: true
})
export default class CatalogPage extends mixins(CatalogPageMixin, CharityPageTargetMixin) {
  protected pageService!: IProductsPageService<ProductsResponse>
  description!: string
  tags!: ITags | undefined

  async asyncData ({ store, error, params, route }: Context): Promise<ICatalogSectionData | {}> {
    // console.log('ASYNC DATA', Date.now());
    const pageServiceFactory = new CatalogPageServiceFactory(route)
    const pageService = pageServiceFactory.getCatalogPageService()

    const requestFilter = await pageService.getRequestFilters(store.state.menu.activeNode.ID)
    await store.dispatch('catalogFilter/setAppliedFilters', { ...requestFilter })

    const activeSort = PageRoute.getActiveSort(route)

    let productsResponse: ProductsResponse;

    const page = PageRoute.getCurrentPage(route)
    const perPage = PageRoute.getPerPage(route)
    try {
      const [response] = await Promise.all([
        pageService.getResponse({
          section: store.state.menu.activeNode?.ID,
          page,
          perPage,
          'sort[SORT]': SortOrder.ASC,
          [activeSort[0]]: activeSort[2],
          'filter_url': route.path
        }, requestFilter),
        store.dispatch('catalogFilter/initFilter', requestFilter)
      ])

      productsResponse = response;
    } catch (e: any) {
      console.error(e.message);
      error({ message: 'Не удалось загрузить продукты' })
      return {}
    }
    const filterTags = Filter.cloneCheckedFilterValues(store.state.catalogFilter.checkedFilterValues)

    const breadcrumbs: IBreadcrumb[] = []

    let tags: ITags | undefined

    if (productsResponse.tags && !Array.isArray(productsResponse.tags)) {
      tags = Object.values(productsResponse.tags)[0]
    }

    const data: ICatalogSectionData = {
      pageCount: productsResponse.page_count || 0,
      products: productsResponse.products,
      description: productsResponse.section?.DESCRIPTION,
      filterTags,
      productCount: productsResponse.product_count ? Number(productsResponse.product_count) : 0,
      activeSort,
      loadMoreCounter: null,
      viewedProductsCounter: (page - 1) * perPage + productsResponse.products.length,
      pageTitle: productsResponse.section?.NAME ?? '',
      productsListLoader: false,
      breadcrumbs,
      tags
    }

    // seo
    if (productsResponse.section) {
      const cityModule = getModule(CityModule, store)
      const filterModule = getModule(CatalogFilterModule, store)

      const seoUtil = store.state.catalogFilter.checkedFilterValues.length === 0
        ? new SectionPageSeo(cityModule, filterModule)
        : new FilteredSectionPageSeo(cityModule, filterModule)

      seoUtil
        .setEntity(productsResponse.section)
        .setPage(page)

      data.meta = seoUtil.getPageMeta()
      data.pageTitle = seoUtil.getH1()
      data.metaTitle = seoUtil.getTitle()
      data.canonicalHref = seoUtil.getCanonical(route)

      // console.log('META', data.meta)
      // console.log('PAGE TITLE', data.pageTitle)
      // console.log('META TITLE', data.metaTitle)
      // console.log('CANONICAL', data.canonicalHref)
    }

    if (store.state.menu.activeNode) {
      // если мы в подсекции (корма). мы получаем обьект с названием
      let parent = store.getters['catalog/getNodeById'](store.state.menu.activeNode.IBLOCK_SECTION_ID);
      breadcrumbs.push({ label: data.pageTitle, route: `/catalog/${store.state.menu.activeNode.UNIQUE_CODE}/` })
      while (parent) {
        const title = seoTitle.find(el => el.id === parent.ID)?.label || parent.NAME
        breadcrumbs.push({ label: title, route: `/catalog/${parent.UNIQUE_CODE}/` })
        parent = store.getters['catalog/getNodeById'](parent.IBLOCK_SECTION_ID)
      }
      // breadcrumbs[breadcrumbs.length - 1].label = items.filter(el => el.id === )

      breadcrumbs.push({ label: 'Главная', route: '/' });

      breadcrumbs.reverse();
    }

    return data
  }

  beforeMount () {
    this.pageService = (new CatalogPageServiceFactory(this.$route)).getCatalogPageService()
  }

  mounted () {
    this.callMindBoxOperation()
  }

  // buildBreadcrumbs () {
  //   if (this.activeNode) {
  //     let parent = this.getNodeById(this.activeNode.IBLOCK_SECTION_ID);
  //     const breadcrumbs = [
  //       { label: this.activeNode.NAME, route: `/catalog/${this.activeNode.UNIQUE_CODE}/` }
  //     ]
  //     while (parent) {
  //       breadcrumbs.push({ label: parent.NAME, route: `/catalog/${parent.UNIQUE_CODE}/` })
  //       parent = this.getNodeById(parent.IBLOCK_SECTION_ID);
  //     }
  //     breadcrumbs.push({ label: 'Главная', route: '/' });
  //
  //     return breadcrumbs.reverse();
  //   }
  //   return [];
  // }

  protected getPath () {
    const filterUriPath = this.filterURI ? this.filterURI.substring(1, this.filterURI.length - 1) : '';

    const catalogSection = PageRoute.getCatalogSection(this.$route)
    // TODO раздела каталога может не быть

    return filterUriPath
      ? `/catalog/${catalogSection}/filter/${filterUriPath}/apply/`
      : `/catalog/${catalogSection}/`;
  }

  protected async getListResponse (params: ProductListParams, applyFilters?: RequestFilter) {
    const response = await this.pageService.getResponse(params, applyFilters)
    return {
      page_count: response.page_count ?? 0,
      product_count: Number(response.product_count) ?? 0,
      products: response.products
    }
  }

  // FILTER PROVIDERS
  @ProvideReactive()
  filterLoading = false

  @ProvideReactive()
  hasCountTooltip = true

  @Provide()
  async onFiltersChange (values: IFilterValue[]) {
    let intersection: IFilterValue[]

    if (values.length > this.checkedFilterValues.length) {
      intersection = values.filter(fv => !this.checkedFilterValues.includes(fv));
    } else {
      intersection = this.checkedFilterValues.filter(fv => !values.includes(fv));
    }

    await this.setCheckedFilterValuesAction(values)
    this.setTooltip('') // reset tooltip
    await this.updateFilter()

    if (intersection[0] !== undefined) {
      setTimeout(this.setTooltip, 400, intersection[0].CONTROL_NAME)
    }
  }

  @Provide()
  async updateFilter () {
    try {
      this.filterLoading = true;
      await this.initFilter(this.requestFilter)
    } catch (e: any) {
      console.error(e.message)
      this.$notify.error('Возникла ошибка')
    }
    this.filterLoading = false;
  }

  @Provide()
  async onApplyFilter () {
    await this.setTooltip('')
    const query = { ...this.$route.query };
    delete query.PAGEN_1;
    this.productsListLoader = true;

    const onComplete = () => { this.productsListLoader = false; }
    const onAbort = onComplete;

    this.$router.replace({ path: this.getPath(), query }, onComplete, onAbort)
  }

  @Provide()
  async onResetFilter () {
    await this.setCheckedFilterValuesAction([])

    const onComplete = () => { this.productsListLoader = false; }
    const onAbort = onComplete;

    const query = { ...this.$route.query };
    delete query.PAGEN_1;
    this.productsListLoader = true;
    const catalogSection = PageRoute.getCatalogSection(this.$route)

    this.$router.replace({ path: `/catalog/${catalogSection}/`, query }, onComplete, onAbort)
  }

  @Provide()
  async onDeleteFilterTag (tag: IFilterValue) {
    this.productsListLoader = true;

    this.removeFilterValue(tag); // remove from checked (without await)
    const applied = { ...this.appliedFilters }
    delete applied[tag.CONTROL_NAME]
    await this.setAppliedFilters(applied)
    await this.initFilter(this.appliedFilters)

    const onComplete = () => {
      this.filterTags = this.filterTags.filter(fTag => fTag.CONTROL_NAME !== tag.CONTROL_NAME)
      this.productsListLoader = false;
    }
    const onAbort = () => { this.productsListLoader = false };

    const query = { ...this.$route.query };
    delete query.PAGEN_1;

    this.$router.replace({ path: this.getPath(), query }, onComplete, onAbort)
  }

  @Provide()
  async onPriceFilterUpdate (controlName: string) {
    await this.setTooltip('');
    await this.updateFilter();
    setTimeout(this.setTooltip, 400, controlName)
  }

  callMindBoxOperation () {
    if (this.$store.state.menu.activeNode) {
      const data: ICatalogView = {
        viewProductCategory: {
          productCategory: {
            ids: {
              website: this.$store.state.menu.activeNode.ID,
            }
          },
        }
      }
      this.$mindBoxExecutor.exec<ICatalogView>('Catalog.ViewCategory', data)
    }
  }
}
</script>
