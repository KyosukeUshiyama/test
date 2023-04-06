<!-- メディア電番検索用フォーム -->
<template>
  <v-form ref="form" v-model="valid">
    <v-card elevation="1" class="pa-5">
      <v-row>
        <v-col cols="12" md="4">
          <v-text-field
            v-model="searchFormData.jigyosha_senyo_no"
            :label="t('businessPersonPhoneNumber')"
            :rules="businessPersonPhoneRules"
          />
        </v-col>
        <v-col cols="12" md="4">
          <v-select
            v-model="searchFormData.match_type"
            :label="t('condition')"
            :items="matchOptions"
            :no-data-text="t('messages.notFound')"
          />
        </v-col>
      </v-row>
      <v-row>
        <v-col cols="12" md="4">
          <v-text-field
            v-model="searchFormData.gno"
            :label="t('GNO')"
            :rules="gnoRules"
          />
        </v-col>
        <v-col cols="12" md="4">
          <v-text-field 
            v-model="searchFormData.biko" 
            :label="t('note')" 
            :rules="bikoRules"
          />
        </v-col>
      </v-row>
      <v-row>
        <v-col cols="12" md="4">
          <v-select
            v-model="searchFormData.kanri_flg"
            :label="t('managementFlag')"
            :items="managementFlagOptions"
            :no-data-text="t('messages.notFound')"
          />
        </v-col>
        <v-col cols="12" md="4">
          <v-select
            v-model="searchFormData.protocol_kubun"
            :label="t('protocol')"
            :items="protcol_kubun"
            :no-data-text="t('messages.notFound')"
          />
        </v-col>
      </v-row>
      <v-row>
        <v-col cols="12" md="4" offset-md="8">
          <div class="d-flex justify-end">
            <v-btn
              :disabled="searchFormEmpty"
              class="mr-5"
              @click="clear"
            >
              {{ t('clear') }}
            </v-btn>
            <v-btn
              color="primary"
              :loading="loading"
              :disabled="searchFormEmpty"
              @click="submit"
            >
              {{ t('search') }}
            </v-btn>
          </div>
        </v-col>
      </v-row>
    </v-card>
  </v-form>
</template>

<script lang="ts">
import Vue from 'vue';
import {
  MatchOption,
  MatchType,
  SearchFormData,
  SearchFormState,
  SelectOption,
} from '@/models/media-phone';
import { TranslateResult } from 'vue-i18n';
import { FormRef } from '@/models/validation';
import { bufferSize, numberSize } from '@/utils/validation';
import { KubunListAPI, ResponseData_KUBUN_KANRI } from '@/api/kubun-kanri';
import { getScreenId } from '@/utils/screenIds';

export default Vue.extend({
  props: {
    loading: {
      type: Boolean,
      default: false,
    },
  },
  data(): SearchFormState {
    return {
      valid: false,
      searchFormData: {
        jigyosha_senyo_no: '',
        match_type: null,
        gno: '',
        biko: '',
        kanri_flg: null,
        protocol_kubun: null,
      },
      formReset: false,
      businessPersonPhoneRules: [(value) => bufferSize(value, 13)],
      gnoRules: [(value) => numberSize(value, 8)],
      bikoRules: [(value) => bufferSize(value, 64)],
      protcol_kubun: [],
      managementFlagOptions: [],
    };
  },
  computed: {
    matchOptions(): MatchOption[] {
      return [
        { value: MatchType.Exact, text: this.t('exact') },
        { value: MatchType.Prefix, text: this.t('prefix') },
        { value: MatchType.Suffix, text: this.t('suffix') },
      ];
    },
    searchFormEmpty(): boolean {
      let key: keyof SearchFormData
      for (key in this.searchFormData) {
        const value = this.searchFormData[key]
        // 検索方法は単体で検索できないようにする
        if ( key !== 'match_type' && (value || value === 0)) return false
      }
      return true
    }
  },
  created(){
    Promise.all([
      KubunListAPI(getScreenId(this.$route.name), "PROTCOL_KUBUN"),
      KubunListAPI(getScreenId(this.$route.name), "KANRI_FLG")
    ]).then((res) => {
      this.protcol_kubun =  this.kubunToSelect(res[0].data);
      this.managementFlagOptions = this.kubunToSelect(res[1].data);
    });
  },
  methods: {
    submit() {
      const valid = (this.$refs.form as FormRef).validate();
      if (valid) {
        this.$emit('submit', this.searchFormData);
      }
    },
    clear(): void {
      (this.$refs.form as FormRef).reset();
      this.searchFormData = {
        jigyosha_senyo_no: '',
        match_type: null,
        gno: '',
        biko: '',
        kanri_flg: null,
        protocol_kubun: null,
      };
    },
    t(field: string): TranslateResult {
      return this.$t(`mediaPhoneSearch.searchForm.${field}`);
    },
    kubunToSelect(kubunList: ResponseData_KUBUN_KANRI[]): SelectOption[] {
      // 区分管理から取得したデータは表示順カラムでソートする
      kubunList.sort(function(a, b) {
        return Number(a.hyoji_jun) < Number(b.hyoji_jun) ? -1 : 1
      })
      return kubunList.map(({ vl, hyoji_mei }) => ({
        value: vl,
        text: hyoji_mei,
      }));
    },
  },
});
</script>
