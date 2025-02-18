<!--
  - coding=utf-8
  - Copyright 2021-present, the Recognai S.L. team.
  -
  - Licensed under the Apache License, Version 2.0 (the "License");
  - you may not use this file except in compliance with the License.
  - You may obtain a copy of the License at
  -
  -     http://www.apache.org/licenses/LICENSE-2.0
  -
  - Unless required by applicable law or agreed to in writing, software
  - distributed under the License is distributed on an "AS IS" BASIS,
  - WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
  - See the License for the specific language governing permissions and
  - limitations under the License.
  -->

<template>
  <div class="list">
    <slot name="header" />
    <div class="results-scroll" id="scroll">
      <div :style="{ paddingTop: `${dataset.viewSettings.headerHeight + 10}px` }" v-if="showLoader">
        <results-loading :size="dataset.viewSettings.pagination.size" />
      </div>
      <DynamicScroller
        v-else
        page-mode
        class="scroller"
        :items="visibleRecords"
        :min-item-size="150"
        :buffer="200"
        :style="{ paddingTop: `${dataset.viewSettings.headerHeight + 10}px` }"
      > 
        <template v-slot="{ item, index, active }">
          <DynamicScrollerItem
            :watch-data="true"
            class="list__li"
            :item="item"
            :active="active"
            key-field="id"
            :index="index"
            :data-index="index"
          >
            <results-record @show-metadata="onShowMetadata" :key="item.id" :dataset="dataset" :item="item">
              <slot name="record" :record="item" />
            </results-record>
          </DynamicScrollerItem>
        </template>
        <template #after>
          <pagination-end-alert :limit="paginationLimit" v-if="isLastPagePaginable" />
        </template>
      </DynamicScroller>
      <LazyReModal
        modal-class="modal-secondary"
        modal-position="modal-center"
        :modal-custom="true"
        :prevent-body-scroll="true"
        :modal-visible="selectedRecord !== undefined"
        @close-modal="onCloseMetadata"
      >
        <Metadata
          v-if="selectedRecord"
          :applied-filters="dataset.query.metadata"
          :metadata-items="selectedRecord.metadata"
          :title="selectedRecord.recordTitle()"
          @metafilterApply="onApplyMetadataFilter"
          @cancel="onCloseMetadata"
        />
      </LazyReModal>
    </div>
    <RePagination
      :total-items="dataset.results.total"
      :pagination-settings="dataset.viewSettings.pagination"
      @changePage="onPagination"
    />
  </div>
</template>
<script>
import { mapActions } from "vuex";
export default {
  props: {
    dataset: {
      type: Object,
      required: true
    }
  },
  data() {
    return {
      scrollComponent: undefined,
      selectedRecord: undefined,
    };
  },
  computed: {
    showLoader() {
      return this.dataset.viewSettings.loading;
    },
    visibleRecords() {
      return this.dataset.visibleRecords;
    },
    paginationLimit() {
      return this.dataset.viewSettings.pagination.maxRecordsLimit;
    },
    isLastPagePaginable() {
      if (this.dataset.results.total > this.paginationLimit) {
        return (
          this.dataset.viewSettings.pagination.page *
            this.dataset.viewSettings.pagination.size ===
          this.dataset.viewSettings.pagination.maxRecordsLimit
        );
      }
      return false;
    },
  },
  mounted() {
    const scroll = document.getElementById("scroll");
    if (scroll) {
      this.scrollComponent = scroll;
      this.scrollComponent.addEventListener("scroll", this.onScroll);
    }
  },
  beforeDestroy() {
    if (this.scrollComponent)
      this.scrollComponent.removeEventListener("scroll", this.onScroll);
  },
  methods: {
    ...mapActions({
      paginate: "entities/datasets/paginate",
      search: "entities/datasets/search",
    }),
    onScroll() {
      if (document.getElementById("scroll").scrollTop > 0) {
        document.getElementsByTagName("body")[0].classList.add("fixed-header");
      } else {
        document
          .getElementsByTagName("body")[0]
          .classList.remove("fixed-header");
      }
    },
    async onApplyMetadataFilter(metadata) {
      this.onCloseMetadata();
      this.search({
        dataset: this.dataset,
        query: { metadata: metadata },
      });
    },
    onShowMetadata(record) {
      this.selectedRecord = record;
    },
    onCloseMetadata() {
      this.selectedRecord = undefined;
    },
    async onPagination(page, size) {
      document.getElementById("scroll").scrollTop = 0;
      await this.paginate({
        dataset: this.dataset,
        page: page,
        size: size
      });
    }
  }
};
</script>
<style lang="scss" scoped>
.list {
  $this: &;
  padding: 0;
  width: 100%;
  position: relative;
  margin-bottom: 0;
  list-style: none;
  .results-scroll {
    height: 100vh !important;
    overflow: auto;
    padding-left: 4em;
    padding-bottom: 61px;
    padding-right: calc(4em + 45px);
    transition: padding 0s ease-in-out 0.1s;
    .fixed-header & {
      padding-bottom: 200px;
    }
    .--metrics & {
      @include media(">desktop") {
        width: 100%;
        padding-right: calc(294px + 100px);
        transition: padding 0.1s ease-in-out;
      }
    }
    @include media(">desktop") {
      transition: padding 0.1s ease-in-out;
      width: 100%;
      padding-right: 100px;
    }
  }
  &__li {
    padding-bottom: 10px;
    position: relative;
    min-height: 150px;
  }
}
</style>
<style lang="scss">
$maxItemsperPage: 20;
@for $i from 0 through $maxItemsperPage {
  .vue-recycle-scroller__item-view:nth-of-type(#{$i}) {
    z-index: $maxItemsperPage - $i
  }
}
</style>
