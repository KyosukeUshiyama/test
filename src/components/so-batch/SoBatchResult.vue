<!--新規申込一括SO作成_検索結果-->
<template>
  <div class="elevation-1 pa-5 mt-5">
    <div class="d-flex justify-space-between mb-5">
      <h3 class="pr-10">
        {{ t('title') }}
      </h3>
      <v-btn class="primary" :disabled="!createPermission" @click="confirm">
        {{ t('confirm') }}
      </v-btn>
    </div>
    <v-data-table
      v-model="selected"
      :headers="headers"
      :items="soBatchList"
      item-key="gno"
      :options.sync="tableOptions"
      hide-default-footer
    >
      <template v-slot:top="{ pagination, options, updateOptions }">
        <v-data-footer
          :pagination="pagination"
          :options="options"
          :items-per-page-options="[5, 10, 15, 20, -1]"
          @update:options="updateOptions"
        />
      </template>
      <template #no-data>
        <p>{{ t('messageNotFound') }}</p>
      </template>
    </v-data-table>
  </div>
</template>

<script lang="ts">
import Vue, { PropType } from 'vue';
import {
  SearchResultState,
  SoBatchNewAppSearchResult,
  SoInquirySearchResultItem,
} from '@/models/so-batch';
import { DialogOptions } from '@/models/dialog';
import { DataTableHeader } from 'vuetify';
import { TranslateResult } from 'vue-i18n';
import { CONFIRMATION, REGISTER_UPDATE_EXECUTE } from '@/constants/dialog';
import { getScreenId } from '@/utils/screenIds';
import { DEFAULT_HYOJI_NUM } from '@/constant/common-constant';

export default Vue.extend({
  props: {
    soBatchList: {
      type: Array as PropType<SoBatchNewAppSearchResult[]>,
      default: [],
    },
    newSearch: {
      type: Boolean,
    },
  },
  data(): SearchResultState {
    return {
      tableOptions: {
        page: 1,
        itemsPerPage: DEFAULT_HYOJI_NUM,
        sortBy: [],
        sortDesc: [],
        groupBy: [],
        groupDesc: [],
        multiSort: false,
        mustSort: false,
      },
      selected: [],
      dataChangeDialog: {} as SoInquirySearchResultItem,
      screenId: '',
      createPermission: false,
    };
  },
  computed: {
    headers(): (DataTableHeader | { text: TranslateResult })[] {
      return [
        {
          text: this.t('gno'),
          value: 'gno',
        },
        {
          text: this.t('customerName'),
          value: 'seti_kanji_shamei',
        },
        {
          text: this.t('prefecturesCapital'),
          value: 'seti_ken_mei',
        },
        {
          text: this.t('productName'),
          value: 'shohin_mei',
        },
        {
          text: this.t('voipInstallDate'),
          value: 'voip_seti_date',
        },
        {
          text: this.t('error'),
          value: 'status',
        },
        {
          text: this.t('errorContent'),
          value: 'error_comment',
        },
      ];
    },
  },
  watch: {
    newSearch(value) {
      if (value) this.tableOptions.page = 1;
    },
    selected: {
      deep: true,
      handler(value) {
        this.$emit('select', value);
      },
    },
  },
  created() {
    this.screenId = getScreenId(this.$route.name);
    this.createPermission = this.$$hasCreatePermission(this.screenId);
  },
  methods: {
    t(field: string): TranslateResult {
      return this.$t(`soBatch.table.${field}`);
    },
    edit(item: SoInquirySearchResultItem): void {
      this.dataChangeDialog = item;
    },
    confirm() {
      if (this.soBatchList.length > 0) {
        const requestData:SoBatchNewAppSearchResult[] = [];
        this.soBatchList.forEach((element) => {
          console.log(element)
          if(element.status != "1") {
            requestData.push(element);
          }
        })
        console.log(requestData)
        console.log(requestData.length)
        if (requestData.length === 0) {
          alert("対象データが存在しません");
          return;
        }
        this.$store.dispatch('dialog/open', {
          type: CONFIRMATION,
          subType: REGISTER_UPDATE_EXECUTE,
          messages: [this.$t('soBatch.messages.soBatchNewApp')],
          callback: () => this.$emit('confirm', requestData),
        } as DialogOptions);
      } else {
        alert("対象データが存在しません")
      }
    },
  },
});
</script>
