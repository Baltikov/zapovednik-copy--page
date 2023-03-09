<template>
  <div class="site-container">
    <div class="page-wrapper">
      <Breadcrumbs class="page-wrapper_site-breadcrumbs" :class="{ mobile: $device.isMobileOrTablet }" :breadcrumbs="breadcrumbs"/>

      <FilterMobile
        v-if="$device.isMobileOrTablet && smartFilter && products.length"
      >
      </FilterMobile>
      <SortMobile
        v-if="$device.isMobileOrTablet"
        :sort-items="sortItems"
        :active-sort-item="activeSort"
        @change="setSort"
      ></SortMobile>
      <div v-loading="productsListLoader" :class="{ mobile: $device.isMobileOrTablet }" class="content">
        <div v-if="smartFilter && products.length" class="main-filter">
          <SmartFilterDesktop
            v-if="$device.isDesktop"
            :smart-filter="smartFilter"
          >
          </SmartFilterDesktop>

        </div>

        <div class="category">
          <div class="category_header">
            <h1 class="category_header_title">{{ pageTitle || activeMenuName }}</h1>
            <div v-show="tags && Object.keys(tags).length !== 0">
              <div class="tags">
                <div v-for="(tags, index) of viewTagProducts" :key="index">
                  <nuxt-link :to="tags[1]" class="tags-info" >
                    {{ tags[0] }}
                  </nuxt-link>
                </div>
              </div>
              <div v-if="tagsProduct.length > 4" class="tags-container-btn">
                <button class="tags-btn" @click="showAll = !showAll">
                  {{ showAll ? 'Показать меньше' : 'Показать все' }}
                </button>
              </div>
            </div>

            <slot name="hint"></slot>

            <div v-show="products.length" class="category_header_count">
              <span class="text-middle text-dark-grey">Товаров</span>
              <span class="text-middle text-dark-grey">{{ productCount | price }}</span>
            </div>

            <div v-show="products.length" class="category_header_sort">
              <CatalogSortDesktop
                v-if="$device.isDesktop"
                :sort-items="sortItems"
                :active-sort-item="activeSort"
                @change="setSort"
              >
              </CatalogSortDesktop>
              <div class="category_header_sort_paginate">
                <Pagination
                  v-if="products.length"
                  :count="pageCount"
                  :current="currentPage"
                  @page-change="paginationHandler"
                />
              </div>
            </div>

            <FilterTags
              v-show="products.length"
              :values="filterTags"
            >
            </FilterTags>
          </div>

          <div class="category_product_list">

            <ProductList :products="products" @updateProductList="onUpdateProductList"/>
          </div>

          <div v-if="products.length" class="category_controls">

            <div class="category_controls_more-products-preloder">
              <PreloaderEllipsis v-show="moreProductsLoader"/>
            </div>

            <div class="category_controls_viewed-products">
              <span class="text-small text-dark-grey">Вы посмотрели {{ viewedProductsCounter | price }} из {{ productCount | price }} товаров</span>
            </div>

            <div class="category_controls_load-more">
              <SiteButton
                v-if="viewedProductsCounter < productCount"
                color-scheme="white-branded"
                text="Загрузить ещё товары"
                font-weight="normal"
                :responsive="$device.isMobileOrTablet"
                :loading="moreProductsLoader"
                @click="loadMoreProducts"
              ></SiteButton>
            </div>

            <div class="category_controls_bottom-navigation">
              <div class="products-total">
                <div class="products-total_title">
                  <span class="text-middle">Товаров на странице:</span>
                </div>
                <ul class="products-total_counters">
                  <li class="products-total_counters_counter" :class="{ active: perPage === 40 }" @click="changePerPage(40)">
                    <span class="text-middle">40</span>
                  </li>
                  <li class="products-total_counters_counter" :class="{ active: perPage === 80 }" @click="changePerPage(80)">
                    <span class="text-middle">80</span>
                  </li>
                </ul>
              </div>
              <Pagination
                v-if="products.length"
                :count="pageCount"
                :current="currentPage"
                @page-change="paginationHandler"
              />
            </div>
            <CatalogPageDescription v-if="currentPage === 1 && !filterTags.length && description" :description="description"/>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script lang="ts">
import { Component, Vue, Prop } from "nuxt-property-decorator";
import Breadcrumbs from "~/components/Breadcrumbs.vue";
import Pagination from "~/components/products/Pagination.vue";
import ProductList from "~/components/products/ProductList.vue";
import PreloaderEllipsis from "~/components/common/PreloaderEllipsis.vue";
import CatalogPageDescription from './CatalogPageDescription.vue'
import {
  IBreadcrumb,
  IFilterValue,
  IProductPreview,
  ProductSortItem,
  SmartFilter,
  ITags,
} from "~/ts-entities";

@Component({
  name: 'CatalogPageMaster',
  components: {
    Breadcrumbs,
    Pagination,
    ProductList,
    PreloaderEllipsis,
    CatalogPageDescription,
    FilterMobile: () => import('@/components/filter/mobile/FilterMobile.vue'),
    SortMobile: () => import('@/components/sort/SortMobile.vue'),
    SmartFilterDesktop: () => import('@/components/filter/SmartFilter.vue'),
    CatalogSortDesktop: () => import('@/components/sort/CatalogSortDesktop.vue'),
    FilterTags: () => import('@/components/filter/FilterTags.vue')
  }
})
export default class CatalogPageMaster extends Vue {
  showAll = false

  @Prop({ type: Array, required: true })
  readonly products!:IProductPreview[]

  @Prop({ type: Object, required: false })
  readonly tags!: ITags

  @Prop({ type: String, required: false })
  readonly description!: string

  @Prop({ type: Boolean, required: true })
  readonly moreProductsLoader!:boolean

  @Prop({ type: Boolean, required: true })
  readonly productsListLoader!:boolean

  @Prop({ type: Number, required: true })
  readonly productCount!: number;

  @Prop({ type: Array, required: true })
  readonly filterTags!: IFilterValue[];

  @Prop({ type: Array, required: true })
  readonly sortItems!:ProductSortItem[]

  @Prop({ type: Number, required: true })
  readonly viewedProductsCounter!: number;

  @Prop({ type: Array, required: true })
  readonly breadcrumbs!: IBreadcrumb[]

  @Prop({ type: Array, required: false })
  readonly smartFilter!: SmartFilter

  @Prop({ type: Array, required: true })
  readonly activeSort!: ProductSortItem

  @Prop({ type: String, required: true })
  readonly pageTitle!: string;

  @Prop({ type: String, required: true })
  readonly activeMenuName!: string

  @Prop({ type: Number, required: true })
  readonly pageCount!: number

  @Prop({ type: Number, required: true })
  readonly currentPage!:number

  @Prop({ type: Number, required: true })
  readonly perPage!:number

  @Prop({ type: Function, required: true })
  setSort!: () => void

  @Prop({ type: Function, required: true })
  paginationHandler!: () => void

  @Prop({ type: Function, required: true })
  loadMoreProducts!: () => void

  @Prop({ type: Function, required: true })
  changePerPage!: (page:number) => void

  @Prop({ type: Function, required: true })
  onUpdateProductList!:() => Promise<void>

  get tagsProduct () {
    const regExp = /([a-яё0-9(),\- ]+)(.+)/i
    const rawTags = this.tags?.REFERENCES ?? []
    const transform = (rawTags: string[]) =>

      rawTags.map((el: string) => {
        const res = el.match(regExp)
        if (res && res[1] && res[2]) {
          return [res[1].trim(), res[2]]
        }
        return undefined
      })
        .filter(el => !!el)

    return transform(rawTags)
  }

  get viewTagProducts () {
    return this.showAll ? this.tagsProduct : this.tagsProduct.slice(0, 4)
  }
}

</script>

<style scoped lang="scss">
@import "./assets/scss/var";

.content {
  display: flex;
  // margin-bottom: 100px;
  .main-filter {
    flex: 0 0 240px;
    margin-right: 38px;
  }
  .category {
    flex: 1 1 auto;
    &_header {
      &_title {
        margin-bottom: 11px;
      }
      &_count {
        margin-bottom: 16px;
      }
      &_sort {
        margin-bottom: 30px;
        display: flex;
        flex-wrap: wrap;
        justify-content: space-between;
        &_paginate {
          .el-pagination {
            padding: 0;
          }
        }
      }
    }
    &_controls {
      &_load-more {
        text-align: center;
        margin-bottom: 40px;
        // .site-button {
        //   background-color: unset;
        //   color: $branded;
        //   border: 1px solid $branded;
        //   font-weight: 400;
        //   &:hover {
        //     opacity: 0.6;
        //   }
        // }

        // @media screen and (max-width: $tabletDevice) {
        //   .site-button {
        //     display: block;
        //     width: 100%;
        //   }
        // }
      }
      &_more-products-preloder {
        text-align: center;
      }

      &_viewed-products {
        margin-bottom: 10px;
        text-align: center;
      }
      &_bottom-navigation {
        display: flex;
        justify-content: space-between;
         margin-bottom: 40px;
        flex-wrap: wrap;
        .products-total {
          display: flex;
          flex-wrap: wrap;
          align-items: center;
          &_counters {
            display: flex;
            &_counter {
              margin-left: 15px;
              list-style: none;
              height: 27px;
              width: 27px;
              border-radius: 4px;
              text-align: center;
              cursor: pointer;
              span {
                line-height: 27px;
              }
              &:hover, &.active {
                color: $branded;
              }
              &.active {
                background-color: $greyMainFilter;
              }
            }

          }
        }
        @media screen and (max-width: $tabletDevice) {
          .products-total {
            display: none;
          }
        }
      }
    }
  }
}
.page-wrapper_site-breadcrumbs.mobile {
  margin-top: 10.4em;
}
.content.mobile {
  .main-filter {
    display: none;
  }
  .category {
    &_header {
      margin-left: 30px;
      margin-right: 30px;
      //&_filter-options {
      //  display: none;
      //}
    }
    &_product_list {
      margin: 0 30px;
    }
    &_controls {
      margin: 0 30px;
    }
  }
}
//.section-description {
//  margin-top: 41px;
//  padding: 60px 0px 60px 0;
//  border-top: 1px solid #E0E0E0;
//}
.active {
  cursor: default !important;
}
.tags {
  display: flex;
  width: 100%;
  margin-top: 30px;
  flex-wrap: wrap;
  &-info {
    margin-right: 8px;
    margin-bottom: 8px;
    font-size: 14px;
    color: #00B0C7;
    /* border: #009689 solid 1px; */
    border: #00B0C7 solid 1px;
    padding: 4px 20px;
    border-radius: 5px;
    display: inline-block;
    text-decoration: none;
    cursor: pointer;
    &:hover {
      background-color: #00B0C7;;
      color: #ffff;
    }
  }
  &-container-btn {
    display: flex;
    align-items: center;
    justify-content: center;
  }
  &-btn {
    margin: 15px 0 8px 0;
    cursor: pointer;
    color: #00B0C7;
    text-decoration: dotted underline;
    font-size: 14px;
    border: none;
    background: none;
    &:hover {
      text-decoration: none;
    }
  }
}
.none-description {
  width: 100%;
  margin-top: 41px;
}

</style>
