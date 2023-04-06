<!-- SO情報照会画面 検索実行 -->
<template>
  <div>
    <so-information-inquiry-form :loading="loading" :mode="mode" @submit="handleSubmit" @dirty="dirtyChange" />
    <so-information-inquiry-result
      :media-phone-list="searchResults"
      :new-search="newSearch"
      :mode="mode"
      :screen-id="screenId"
      @select="handleSelect"
      @reload="callApi"
    />
  </div>
</template>

<script lang="ts">
import Vue from 'vue';
import SoInformationInquiryResult from '@/components/so-inquiry/SoInformationInquiryResult.vue';
import SoInformationInquiryForm from '@/components/so-inquiry/SoInformationInquiryForm.vue';
import {
  SoInquirySearchResultItem,
  SearchFormData,
  SearchViewState,
} from '@/models/so-inquiry';
import {
  getSoInfoInquiryList,
  convSoInfoInquiryData,
} from '@/api/so-info-inquiry-api';
import {
  KubunListAPI,
} from '@/api/kubun-kanri'
import { AxiosError } from 'axios';
import { TranslateResult } from 'vue-i18n';
import { getScreenId } from '@/utils/screenIds';
import { DialogOptions } from '@/models/dialog';
import { SO_TAB, SO_EMERGENCY, SO_NEW, SO_MENU } from '@/models/so';

export default Vue.extend({
  components: {
    SoInformationInquiryResult,
    SoInformationInquiryForm,
  },
  props: {
    mode: {
      type: String,
      default: null,
    },
  },
  data(): SearchViewState {
    return {
      shouldSearchOnInitialOptionsUpdate: false,
      searchResults: [],
      loading: false,
      newSearch: false,
      selected: [],
      searchData: null,
      screenId: '',
    };
  },
  created(){
    this.screenId = getScreenId(this.$route.name);
  },
  methods: {
    t(field: string): TranslateResult {
      return this.$t(`soInquiry.${field}`);
    },
    handleSubmit(searchData: SearchFormData): void {
      this.loading = true;
      this.searchData = searchData;
      this.callApi();
    },
    callApi(): void {
      this.searchResults = [];

      // SO情報照会検索実行＋区分取得
      if (this.searchData) {
        let screen_id = ''
        switch(this.mode){
          case SO_TAB:
            screen_id = getScreenId(this.$route.name, 'SoCustomerSearchInquiryView')
            break
          case SO_NEW:
          case SO_MENU:
          case SO_EMERGENCY:
            screen_id = getScreenId(this.$route.name)
            break
        }
        this.$$openLoadingScreen() // Loading処理の呼び出し
        Promise.all([
          getSoInfoInquiryList(screen_id, this.searchData),
          KubunListAPI(screen_id, 'PROTCOL_KUBUN'), // 種別
          KubunListAPI(screen_id, 'SHINKI_KAIYAKU_KUBUN'), // 区分
          KubunListAPI(screen_id, 'KINKYU_FLG'), // 送信フラグ
          KubunListAPI(screen_id, 'SO_SAKUSEI_KUBUN'), // SO作成区分
          KubunListAPI(screen_id, 'STATUS'), // ステータス
        ])
          .then((res) => {
            if (null != res[0].data) {
              const { results, max_count_message } = res[0].data
              if (max_count_message) {
                this.$store.dispatch('dialog/open', {
                  type: 'MESSAGE',
                  subType: 'MESSAGE_ALERT',
                  messages: [max_count_message],
                } as DialogOptions);
              }
              results.forEach((element) => {
                this.searchResults.push(
                  convSoInfoInquiryData(
                    element,
                    res[1].data,
                    res[2].data,
                    res[3].data,
                    res[4].data,
                    res[5].data
                  )
                );
              });
            }
          })
          .catch((err: AxiosError) => {
            console.error(err.response);
          })
          .finally(() => {
            this.$$closeLoadingScreen() // Loading処理の解除
            this.loading = false;
            this.newSearch = false;
          });
      }
    },
    handleSelect(selectedRows: SoInquirySearchResultItem[]): void {
      this.selected = selectedRows.map((item) => item.gno);
    },
    dirtyChange(data: boolean): void{
      this.$emit('dirty', data)
    }
  },
});
</script>
