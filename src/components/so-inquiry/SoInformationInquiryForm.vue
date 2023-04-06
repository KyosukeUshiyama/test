<!-- SO情報照会画面 検索フォーム（タブ、メニューから、緊急共通） -->
<template>
  <v-card elevation="1" class="pa-5">
    <v-form ref="form" v-model="valid" @submit.prevent="submit">
      <v-row>
        <v-col cols="12" md="3">
          <v-text-field
            v-model="searchFormData.gno"
            :label="t('gno')"
            :rules="gnoRules"
            :filled="modeTab"
            :readonly="modeTab"
            @input="fieldChange"
          />
        </v-col>
        <v-col cols="12" md="2">
          <app-date-picker
            v-model="searchFormData.construction_schedule_date_from"
            :rules="scheduleFromRules"
            :label="t('constructionScheduleDateFrom')"
            @input="fieldChange"
          />
        </v-col>
        <v-col cols="12" md="1">
          <p class="charactor">
            {{ '～' }}
          </p>
        </v-col>
        <v-col cols="12" md="2">
          <app-date-picker
            v-model="searchFormData.construction_schedule_date_to"
            :rules="scheduleToRules"
            :label="t('constructionScheduleDateTo')"
            @input="fieldChange"
          />
        </v-col>
        <v-col cols="12" md="3">
          <v-select
            v-model="searchFormData.status"
            :label="t('status')"
            :items="status"
            :no-data-text="t('messages.notFound')"
            @input="fieldChange"
          />
        </v-col>
      </v-row>
      <v-row>
        <v-col cols="12" md="3">
          <v-select
            v-model="searchFormData.emergency_sending_flag"
            :label="t('emergencySendingFlag')"
            :items="kinkyu_flg"
            :no-data-text="t('messages.notFound')"
            @input="fieldChange"
          />
        </v-col>
        <v-col cols="12" md="3">
          <v-select
            v-model="searchFormData.type"
            :label="t('type')"
            :items="protcol_kubun"
            :no-data-text="t('messages.notFound')"
            @input="fieldChange"
          />
        </v-col>
        <v-col cols="12" md="3">
          <v-select
            v-model="searchFormData.new_contract_cancel_section"
            :label="t('newContractCancelSection')"
            :items="shinki_kaiyaku_kubun"
            :no-data-text="t('messages.notFound')"
            @input="fieldChange"
          />
        </v-col>
        <v-col cols="12" md="3">
          <v-select
            v-model="searchFormData.create_so_section"
            :label="t('createSoSection')"
            :items="so_sakusei_kubun"
            :no-data-text="t('messages.notFound')"
            @input="fieldChange"
          />
        </v-col>
      </v-row>
      <v-row>
        <v-col cols="12" md="2">
          <app-date-picker
            v-model="searchFormData.registration_date_from"
            :rules="registrationFromRules"
            :label="t('registrationDateFrom')"
            @input="fieldChange"
          />
        </v-col>
        <v-col cols="12" md="1">
          <p class="charactor">
            {{ '～' }}
          </p>
        </v-col>
        <v-col cols="12" md="2">
          <app-date-picker
            v-model="searchFormData.registration_date_to"
            :rules="registrationToRules"
            :label="t('registrationDateTo')"
            @input="fieldChange"
          />
        </v-col>
      </v-row>
      <v-row>
        <v-col md="8" />
        <v-col md="2">
          <v-btn class="mr-10" :disabled="loading" @click="clear">
            {{ t('clear') }}
          </v-btn>
        </v-col>
        <v-col md="2">
          <v-btn
            color="primary"
            :disabled="checkFormInvalid || loading"
            :loading="loading"
            @click="submit"
          >
            {{ t('search') }}
          </v-btn>
        </v-col>
      </v-row>
    </v-form>
  </v-card>
</template>

<script lang="ts">
import Vue from 'vue';
import {
  SearchFormData,
  SearchFormState,
  EmergencyFlagType,
  SoInquiryDataValidation,
  StatusType,
  CreateSoSectionType,
} from '@/models/so-inquiry';
import { TranslateResult } from 'vue-i18n';
import AppDatePicker from '../UI/AppDatePicker.vue';
import { numberSize, formatDate } from '@/utils/validation';
import { 
  KubunListAPI,
  ResponseData_KUBUN_KANRI,
} from '@/api/kubun-kanri';
import { AxiosError } from 'axios';
import { SelectOption } from '@/models/customer';
import { SO_TAB, SO_EMERGENCY } from '@/models/so';
import { getScreenId } from '@/utils/screenIds';

export default Vue.extend({
  components: { AppDatePicker },
  props: {
    mode: {
      type: String,
      default: null,
    },
    loading: {
      type: Boolean,
      default: false,
    },
  },
  data(): SearchFormState {
    return {
      valid: false,
      // SO情報照会画面初期データ設定
      searchFormData: {
        
        // タブ表示の場合はGNOを入力済みとしておく（編集不可の前提）
        gno:
          this.mode === SO_TAB ?
            this.$route.query.gno.toString() :
            '' ,
        construction_schedule_date_from: '',
        construction_schedule_date_to: '',
        status:
          this.mode === SO_EMERGENCY ?
            StatusType.Notprocessed :
            null,
        // 緊急の場合は初期表示を緊急で入力済みとしておく（編集不可の前提）
        emergency_sending_flag:
          this.mode === SO_EMERGENCY ?
            EmergencyFlagType.Emergency :
            null,
        type: null,
        new_contract_cancel_section: null,
        create_so_section:
          this.mode === SO_EMERGENCY ?
            CreateSoSectionType.Created :
            null,
        registration_date_from: '',
        registration_date_to: '',
      },
      formReset: false,
      gnoRules: [(value) => numberSize(value, 8)],
      scheduleFromRules: [(value) => formatDate(value)],
      scheduleToRules: [(value) => formatDate(value)],
      registrationFromRules: [(value) => formatDate(value)],
      registrationToRules: [(value) => formatDate(value)],
      protcol_kubun: [],
      shinki_kaiyaku_kubun: [],
      kinkyu_flg: [],
      so_sakusei_kubun: [],
      status: [],
      emergencyFlag: {value: '', text: ''},
      modeTab: this.mode === SO_TAB,
      modeEmergency: this.mode === SO_EMERGENCY

    };
  },
  validations: {
    searchFormData: {
      ...new SoInquiryDataValidation(),
    },
  },
  
  computed: {
    checkFormInvalid(): boolean {
      if (
        !!this.searchFormData.gno ||
        !!this.searchFormData.construction_schedule_date_from ||
        !!this.searchFormData.construction_schedule_date_to
      ) {
        return false;
      }
      return true;
    },
  },
  watch: {
    searchFormData: {
      deep: true,
      handler() {
        if (this.formReset) {
          this.formReset = false;
          return;
        }
      },
    },
  },
  created() {
    let screen_id = ''
    switch(this.mode){
      case 'tab':
        screen_id = getScreenId(this.$route.name, 'SoCustomerSearchInquiryView')
        break
      case 'menu':
      case 'emergency':
        screen_id = getScreenId(this.$route.name)
        break
    }
    Promise.all([
      KubunListAPI(screen_id, "PROTCOL_KUBUN"),        // 種別
      KubunListAPI(screen_id, "SHINKI_KAIYAKU_KUBUN"), // 区分
      KubunListAPI(screen_id, "KINKYU_FLG"),           // 送信フラグ
      KubunListAPI(screen_id, "SO_SAKUSEI_KUBUN"),     // SO作成区分
      KubunListAPI(screen_id, "STATUS"),               // ステータス
    ])
    .then((res) => {
      this.protcol_kubun = this.kubunToSelect(res[0].data);
      this.shinki_kaiyaku_kubun = this.kubunToSelect(res[1].data);
      this.kinkyu_flg = this.kubunToSelect(res[2].data);
      this.so_sakusei_kubun = this.kubunToSelect(res[3].data);
      this.status = this.kubunToSelect(res[4].data);
      this.emergencyFlag = this.kinkyu_flg[0];
    })
    .catch((err: AxiosError) => {
      console.error(err.response)
    })
  },
  methods: {
    submit(): void {
      const valid = (
        this.$refs.form as Vue & { validate: () => boolean }
      ).validate();

      if (valid) {
        this.$emit('submit', this.searchFormData);
        this.$emit('dirty', false)
      }
    },
    // クリアボタン押下時
    clear(): void {
      this.formReset = true;
      this.$v.searchFormData.$reset();
      let key: keyof SearchFormData;
      for (key in this.searchFormData)
      {
        if(this.mode === SO_TAB && key === 'gno')
        {
            // タブ表示、かつGNOは初期化しない
        }
        else
        {
          // 上記以外は全て初期化する
          this.searchFormData[key] = null;
        }
      }
      this.$emit('dirty', false)
    },
    t(field: string): TranslateResult {
      return this.$t(`soInquiry.${field}`);
    },
    kubunToSelect(kubunList: ResponseData_KUBUN_KANRI[]): SelectOption[] {
      return kubunList.map(({ vl, hyoji_mei }) => ({
        value: vl,
        text: hyoji_mei,
        disabled: false,
      }));
    },
    fieldChange(): void{
      this.$emit('dirty', true)
    }
  },
});
</script>

<style lang="scss">
.charactor {
  font-size: 40px;
}
</style>
