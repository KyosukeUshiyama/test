<!--新規申込一括SO作成-->
<template>
  <div>
    <so-batch-form :loading="loading" @submit="handleSubmit" />
    <so-batch-result
      :so-batch-list="searchResults"
      :new-search="newSearch"
      @select="handleSelect"
      @confirm="handleConfirm"
    />
  </div>
</template>

<script lang="ts">
import Vue from 'vue';
import SoBatchResult from '@/components/so-batch/SoBatchResult.vue';
import SoBatchForm from '@/components/so-batch/SoBatchForm.vue';
import {
  SoInquirySearchResultItem,
  SearchFormData,
  SearchViewState,
  SoBatchNewAppSearchResult,
  NewSoInfoRegiRequest,
} from '@/models/so-batch';
import { 
  getSoInfoInquiryList,
  newSoInfoRegistration,
} from '@/api/so-batch-api';
import { AxiosError } from 'axios';
import { TranslateResult } from 'vue-i18n';
import { getScreenId } from '@/utils/screenIds';
import { ERROR_STATUS_503 } from '@/constant/common-constant';

export default Vue.extend({
  components: {
    SoBatchResult,
    SoBatchForm,
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
     };
  },
  methods: {
    t(field: string): TranslateResult {
      return this.$t(`soBatch.${field}`);
    },
    handleSubmit(searchData: SearchFormData): Promise<void> {
      this.loading = true;
      this.searchResults = [];
      this.$$openLoadingScreen() // Loading処理の呼び出し
      return Promise.all([
            getSoInfoInquiryList(getScreenId(this.$route.name), searchData),
        ])
        .then((res) => {
          console.log(res)
          if(null!=res[0].data)
          {
            res[0].data.forEach(element => {
              this.searchResults.push(element);
            });
          }
        })
        .catch((err: AxiosError) => {
          console.error(err.response)
        })
        .finally(() => {
          this.$$closeLoadingScreen() // Loading処理の解除
          this.loading = false;
          this.newSearch = false;
        });
    },
    handleSelect(selectedRows: SoInquirySearchResultItem[]): void {
      this.selected = selectedRows.map((item) => item.gno);
    },
    handleConfirm(data: SoBatchNewAppSearchResult[]) :void {
      console.log("handleConfirm")
      // SO情報登録リクエスト用データ作成
      const NewSoRegiRequest = this.convertNewSoInfoRegi(data);
      console.log(NewSoRegiRequest)
      // 新規一括時SO情報登録API呼び出し
      newSoInfoRegistration(getScreenId(this.$route.name), NewSoRegiRequest)
        .then((res) => {
          console.log(res)
          alert(this.$t(`customerDetail.soDataCreate.messageSuccess`).toString())
        })
        .catch((err: AxiosError) => {
          if (err.response?.status === ERROR_STATUS_503) {
            console.log(err.response)
            alert(this.$t(`customerDetail.soDataCreate.messageError`).toString() + "\n" 
              + "エラー内容: " + err.response.data.failures.ErrorDetail.message)
          } else {
            this.$store.dispatch('systemError/showSystemError', err.response)
          }
      });
    },
    convertNewSoInfoRegi(ResultData: SoBatchNewAppSearchResult[]): NewSoInfoRegiRequest[] {
      const data: NewSoInfoRegiRequest[] = [];
      ResultData.forEach((element) => {
        const convData: NewSoInfoRegiRequest = {
          gno: element.gno,
          seq_no: element.seq_no,
          kaisha_mei: element.seti_kanji_shamei,
          chiku: element.seti_ken_mei,
          shinki_kaiyaku_kubun: '1',
          shubetsu: element.protcol_kubun,
          koji_yotei_date: element.voip_seti_date,
          koji_yotei_date_mitei_kubun: false,
          so_sakusei_kubun: '0',
          kinkyu_flg: '0',
          toroku_kubun: '0',
          oa_juchu_no: element.oa_juchu_no,
          oa_juchu_edaban: element.oa_juchu_setchi_no,
          toroku_naiyo_id: '',
          biko: '',
          juch_no: element.juch_no,
          juch_mesai_no: element.juch_mesai_no,
          options: {
            new: '1'
          },
          target: {
            koji_yotei_date: element.voip_seti_date,
            gno: element.gno,
            seq_no: null
          },
        }
        data.push(convData);
      })
      return data;
    }
  },
});
</script>
