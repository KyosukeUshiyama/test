<!--SO情報登録フォーマット画面-->
<template>
  <div>
    <v-row>
      <v-col v-if="!tabMode" md="3" cols="12">
        <v-text-field
          :value="soInformation.gno"
          :label="t('gno')"
          :filled="!newMode"
          :readonly="!newMode"
          :rules="gnoRules"
          :class="{ 'required-label': newMode }"
          @input="emit('gno', $event)"
        />
      </v-col>
      <v-col v-if="newMode" md="2" cols="12">
        <v-btn
          color="primary"
          class="mt-5"
          target="_blank"
          :disabled="!soInformation.gno"
          @click="searchGno"
        >
          {{ t('searchBtn') }}
        </v-btn>
      </v-col>      
      <v-col md="5" cols="12">
        <v-text-field
          :value="soInformation.subNumber"
          :label="t('subNumber')"
          filled
          readonly
        />
      </v-col>
      <v-col v-if="!newMode" md="4" cols="12">
        <v-btn
          v-if="editMode && !tabMode && !newMode"
          color="primary"
          class="mt-5"
          @click="getServiceRegisterPage"
        >
          {{ t('serviceInfoBtn') }}
        </v-btn>
      </v-col>
    </v-row>
    <v-row>
      <v-col md="9" cols="12">
        <v-text-field
          :value="soInformation.companyName"
          :label="t('companyName')"
          filled
          readonly
        />
      </v-col>
      <v-col md="3" cols="12">
        <v-text-field
          :value="soInformation.district"
          :label="t('district')"
          filled
          readonly
        />
      </v-col>
    </v-row>
    <v-row>
      <v-col md="8" cols="12">
        <v-select
          :value="soInformation.newContractCancel"
          :label="t('newContractCancel')"
          :items="shinki_kaiyaku_kubun"
          :rules="newContractCancelRules"
          :readonly="!editMode || detail"
          :filled="!editMode || detail"
          :no-data-text="t('messages.notFound')"
          class="required-label"
          @change="emit('newContractCancel', $event)"
        />
      </v-col>
      <v-col md="4" cols="12">
        <v-select
          :value="soInformation.type"
          :label="t('type')"
          :items="protcol_kubun"
          :rules="typeRules"
          readonly
          filled
          :no-data-text="t('messages.notFound')"
          item-value="value"
          item-text="text"
          class="required-label"
          @change="emit('type', $event)"
        />
      </v-col>
    </v-row>
    <v-row>
      <v-col md="8" cols="12">
        <app-date-picker
          :value="soInformation.constructionScheduleDate"
          :label="t('constructionScheduleDate')"
          :rules="constructionScheduleDateRules"
          :filled="newContractCancelDisabled"
          :readonly="newContractCancelDisabled"
          required
          @input="emit('constructionScheduleDate', $event)"
        />
      </v-col>
      <v-col md="4" cols="12">
        <v-checkbox
          :input-value="soInformation.constructionScheduleDateUndecided"
          :readonly="!editMode"
          :label="t('constructionScheduleDateUndecided')"
          @change="changeConstructionScheduleDateUndecided($event)"
        />
      </v-col>
    </v-row>
    <v-row>
      <v-col md="8" cols="12">
        <v-select
          :value="soInformation.createSoFile"
          :label="t('createSoFile')"
          :items="so_sakusei_kubun"
          :rules="createSoFileRules"
          :readonly="!editMode || detail"
          :filled="!editMode || detail"
          :no-data-text="t('messages.notFound')"
          class="required-label"
          @change="emit('createSoFile', $event)"
        />
      </v-col>
      <v-col md="4" cols="12">
        <v-select
          :value="soInformation.emergencySendingFlag"
          :label="t('emergencySendingFlag')"
          :items="kinkyu_flg"
          :rules="emergencySendingFlagRules"
          :readonly="!editMode || detail"
          :filled="!editMode || detail"
          :no-data-text="t('messages.notFound')"
          class="required-label"
          @change="emit('emergencySendingFlag', $event)"
        />
      </v-col>
    </v-row>
    <v-row>
      <v-col md="8" cols="12">
        <v-select
          :value="soInformation.soRegistration"
          :label="t('soRegistration')"
          :items="so_toroku_kubun"
          :rules="soRegistrationRules"
          :readonly="!editMode || detail"
          :filled="!editMode || detail"
          :no-data-text="t('messages.notFound')"
          class="required-label"
          @change="emit('soRegistration', $event)"
        />
      </v-col>
    </v-row>
    <v-row>
      <v-col md="4" cols="12">
        <v-text-field
          :value="soInformation.oaOrderNumber"
          :label="t('oaOrderNumber')"
          :readonly="!editMode || detail"
          :filled="!editMode || detail"
          :rules="oaOrderNumberRules"
          @input="emit('oaOrderNumber', $event)"
        />
      </v-col>
      <v-col md="4" cols="12">
        <v-text-field
          :value="soInformation.oaOrderSubNumber"
          :label="t('oaOrderSubNumber')"
          :readonly="!editMode || detail"
          :filled="!editMode || detail"
          :rules="oaOrderSubNumberRules"
          @input="emit('oaOrderSubNumber', $event)"
        />
      </v-col>
      <v-col md="4" cols="12">
        <v-select
          :value="soInformation.registerContents"
          :label="t('registerContents')"
          :items="so_toroku_naiyo"
          :readonly="!editMode || detail"
          :filled="!editMode || detail"
          :no-data-text="t('messages.notFound')"
          @change="emit('registerContents', $event)"
        />
      </v-col>
    </v-row>
    <v-row>
      <v-col cols="12">
        <v-textarea
          :value="soInformation.comment"
          :label="t('comment')"
          :readonly="!editMode"
          :filled="!editMode"
          rows="2"
          :rules="commentRules"
          @input="emit('comment', $event)"
        />
      </v-col>
    </v-row>
  </div>
</template>

<script lang="ts">
import { SoInformation, SelectOptions, SO_TAB, SO_EMERGENCY, SO_NEW, SO_MENU, UNDECIDED_DATE } from '@/models/so';
import Vue, { PropType } from 'vue';
import { TranslateResult } from 'vue-i18n';
import AppDatePicker from '../UI/AppDatePicker.vue';
import {
  bufferSize,
  required,
  numberSize,
  formatDate,
  datePast,
} from '@/utils/validation';
import { InputValidation } from '@/models/validation';
import { KubunListAPI, ResponseData_KUBUN_KANRI } from '@/api/kubun-kanri';
import { getAllSoTorokuNaiyoAPI } from '@/api/so-toroku-naiyo-mst';
import { AxiosError } from 'axios';
import { getSoInquiryByGNOAndDate, getSoInfoInquiryList } from '@/api/so-info-inquiry-api';
import {
    getCustomerListAPI,
    convertCustomerSearchResultItemFromResponse,
} from '@/api/customer-api';
import { getScreenId } from '@/utils/screenIds';
import dateFormat from "dateformat";
import { getCompanyInfo } from '@/api/so-control'
import { SearchFormData, SHINKI_KAIYAKU_KUBUN_NEW, SO_SAKUSEI_KUBUN_NOT_CREATED } from '@/models/so-inquiry';

export default Vue.extend({
  components: { AppDatePicker },
  props: {
    soInformation: {
      type: Object as PropType<SoInformation>,
      required: true,
    },
    mode: { 
      type: String as PropType<string>,
      default: '',
      },
    inquiryFlg: {
      type: Boolean,
      default: false
    },
    newContractCancelDisabled: { type: Boolean, required: true },
    detail: {
      type: Boolean,
      default: false
    }
  },
  data() {
    return {
      valid: false,
      gnoRules:[
        (value: InputValidation) => 
          required(
            value,
            this.$t('soInquiry.soInformationDetails.gno')
          ),
        (value: InputValidation) => numberSize(value, 8)
      ],
      constructionScheduleDateRules: [
        (value: InputValidation) => 
          required(
            value,
            this.$t('soInquiry.soInformationDetails.constructionScheduleDate')
          ),
        (value: InputValidation) => formatDate(value),
        this.detail ? true : (value: InputValidation) => datePast(value)
      ],
      newContractCancelRules: [
        (value: InputValidation) =>
          required(
            value,
            this.$t('soInquiry.soInformationDetails.newContractCancel')
          ),
      ],
      typeRules: [
        (value: InputValidation) =>
          required(value, this.$t('soInquiry.soInformationDetails.type')),
      ],
      createSoFileRules: [
        (value: InputValidation) =>
          required(
            value,
            this.$t('soInquiry.soInformationDetails.createSoFile')
          ),
      ],
      emergencySendingFlagRules: [
        (value: InputValidation) =>
          required(
            value,
            this.$t('soInquiry.soInformationDetails.emergencySendingFlag')
          ),
      ],
      soRegistrationRules: [
        (value: InputValidation) =>
          required(
            value,
            this.$t('soInquiry.soInformationDetails.soRegistration')
          ),
      ],
      oaOrderNumberRules: [(value: InputValidation) => numberSize(value, 8)],
      oaOrderSubNumberRules: [(value: InputValidation) => numberSize(value, 1)],
      commentRules: [(value: InputValidation) => bufferSize(value, 300)],
      protcol_kubun: [] as SelectOptions[],
      shinki_kaiyaku_kubun: [] as SelectOptions[],
      kinkyu_flg: [] as SelectOptions[],
      so_sakusei_kubun: [] as SelectOptions[],
      so_toroku_kubun: [] as SelectOptions[],
      so_toroku_naiyo: [] as SelectOptions[],
      path: '',
      z_constructionScheduleDate: "",
      newMode: this.mode === SO_NEW
    };
  },
  computed: {
    editMode(): boolean {
      return this.mode !== SO_EMERGENCY
    },    
    tabMode(): boolean {
      return this.mode === SO_TAB
    }
  },
  created() {
    let screen_id = ''
    switch(this.mode){
      case SO_TAB:
        screen_id = getScreenId(this.$route.name, 'SoInformationRegisterView')
        break
      case SO_NEW:
      case SO_MENU:
      case SO_EMERGENCY:
        screen_id = getScreenId(this.$route.name)
        break
    }
    Promise.all([
      KubunListAPI(screen_id, 'PROTCOL_KUBUN'), // 種別
      KubunListAPI(screen_id, 'SHINKI_KAIYAKU_KUBUN'), // 区分
      KubunListAPI(screen_id, 'KINKYU_FLG'), // 送信フラグ
      KubunListAPI(screen_id, 'SO_SAKUSEI_KUBUN'), // SO作成区分
      KubunListAPI(screen_id, 'TOROKU_KUBUN'), // SO登録区分
      getAllSoTorokuNaiyoAPI(screen_id),
    ])
      .then((res) => {
        this.protcol_kubun = this.kubunToSelect(res[0].data);
        this.shinki_kaiyaku_kubun = this.kubunToSelect(res[1].data);
        this.kinkyu_flg = this.kubunToSelect(res[2].data);
        this.so_sakusei_kubun = this.kubunToSelect(res[3].data);
        this.so_toroku_kubun = this.kubunToSelect(res[4].data);
        this.so_toroku_naiyo = res[5].data.map(
          ({ toroku_naiyo_id, toroku_naiyo }) => ({
            value: toroku_naiyo_id,
            text: toroku_naiyo,
          })
        );
        this.so_toroku_naiyo.unshift({value: "", text: ""});
        // プロトコル区分が有効かチェック
        let protcol = this.protcol_kubun.filter((item) => item.value == this.soInformation.type)
        if (protcol.length == 0) {
          this.soInformation.type = ''
        }
      })
      .catch((err: AxiosError) => {
        console.error(err.response);
      });
  },
  methods: {
    t(field: string): TranslateResult {
      return this.$t(`soInquiry.soInformationDetails.${field}`);
    },
    emit(field: string, value: string | boolean | number | null): void {
      this.$emit('field-change', field, value, this.$refs.form);
    },
    // 工事日未定時の処理
    changeConstructionScheduleDateUndecided($event: boolean) {
      console.log($event)
      if ($event == true) {
        this.z_constructionScheduleDate = this.soInformation.constructionScheduleDate
        this.emit('constructionScheduleDate', UNDECIDED_DATE)
      } else {
        this.soInformation.constructionScheduleDate = "";
        this.emit('constructionScheduleDate', this.z_constructionScheduleDate)
      }
      this.emit('constructionScheduleDateUndecided', $event)
    },
    // 区分リストの設定
    kubunToSelect(kubunList: ResponseData_KUBUN_KANRI[]): SelectOptions[] {
      return kubunList.map(({ vl, hyoji_mei }) => ({
        value: vl,
        text: hyoji_mei,
      }));
    },
    // GNOからso情報の検索
    searchGno(): void {
      console.log("searchGno")
      this.soInformation.subNumber = ""
      this.soInformation.juch_no = ""
      this.soInformation.juch_mesai_no = ""
      let koji_yotei = dateFormat(new Date(),"yyyy/mm/dd")
      // 新規でSO未作成のSOコントロールを検索
      let searchData: SearchFormData = {
        gno: this.soInformation.gno,
        construction_schedule_date_from: '',
        construction_schedule_date_to: '',
        status: null,
        emergency_sending_flag: null,
        type: null,
        new_contract_cancel_section: SHINKI_KAIYAKU_KUBUN_NEW,
        create_so_section: SO_SAKUSEI_KUBUN_NOT_CREATED,
        registration_date_from: '',
        registration_date_to: '',
      }
      let screen_id = getScreenId(this.$route.name)
      Promise.all([
        getCompanyInfo(screen_id, this.soInformation.gno),
        getSoInquiryByGNOAndDate(screen_id, this.soInformation.gno, koji_yotei),
        getSoInfoInquiryList(screen_id, searchData)
      ])
        .then((res) => {
          console.log(res)

          if ( null != res[0].data) {
            this.soInformation.juch_no = res[0].data.juch_no
            this.soInformation.juch_mesai_no = res[0].data.juch_mesai_no
            this.soInformation.companyName = res[0].data.seti_kanji_shamei
            this.soInformation.district = res[0].data.seti_ken_mei
            this.soInformation.type = res[0].data.protcol_kubun
            // プロトコル区分が有効かチェック
            let protcol = this.protcol_kubun.filter((item) => item.value == this.soInformation.type)
            if (protcol.length == 0) {
              this.soInformation.type = ''
            }

            if(0 < res[1].data.length) {
              //最終データ+1を枝番に割り当てる
              this.soInformation.subNumber = String(res[1].data[res[1].data.length - 1].seq_no + 1)
            }else {
              // SO情報ない場合は枝番に"1"を割り当てる
              this.soInformation.subNumber = "1"
            }
          } else {
            console.log("データが存在しない場合")
            alert(this.$t('soInquiry.alert_no_data'))
          }
          if (res[2].data.results.length > 0) {
            this.$emit('change-new-so-flg', true)
          } else {
            this.$emit('change-new-so-flg', false)
          }
        })
        .catch(err => console.error(err))
    },
    // サービス情報参照ボタン押下時の処理
    getServiceRegisterPage(): void{
      const searchData = {
        gno: this.soInformation.gno,
        uf_code: '',
        tel: '',
        customer: '',
        oa_customer: '',
        ip: '',
        set_customer: '',
        cancel: false,
      }
      getCustomerListAPI(getScreenId(this.$route.name), searchData)
          .then((res) =>
          {
            if(null != res.data)
            {
              let searchResult = convertCustomerSearchResultItemFromResponse(res.data.customers[0], 0);
              console.log(searchResult);
              this.path = '/customer/detail/' + searchResult.gno + '?' +
                          'gno=' +searchResult.gno + '&' +
                          'customer_no=' + searchResult.customer_no + '&' +
                          'kakin_tel_no=' + searchResult.kakin_tel_no + '&' +
                          'setisaki_jusyo=' + searchResult.setisaki_jusyo + '&' +
                          'setisaki_name=' + searchResult.setisaki_name + '&' +
                          'status=' + searchResult.status + '&' +
                          'uf_cd=' + searchResult.uf_cd + '&' +
                          'juchu_no=' + searchResult.juch_no + '&' +
                          'juchu_meisai_no=' + searchResult.juch_mesai_no + '&' +
                          'seti_no=' + searchResult.seti_no;
              window.open(this.path, '_blank');
            }
          })
          .catch((err: AxiosError) =>
          {
            console.error(err.response)
          });
    }
  },
});
</script>
