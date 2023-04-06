<!--SIPオプション追加ダイアログ&SIPオプション変更ダイアログ-->
<template>
  <v-form ref="form" v-model="valid" :disabled="cancelStatus || !updatePermission || !createPermission">
    <v-card>
      <v-card-title>
        {{ t(editMode ? 'editSip' : 'addSip') }}
      </v-card-title>
      <v-card-text>
        <v-row>
          <v-col md="3" cols="12">
            <v-text-field
              v-model="internalSipData.gno"
              filled
              readonly
              :label="t('GNO')"
            />
          </v-col>
          <v-col md="3" cols="12" offset-md="6">
            <v-text-field
              v-model="internalSipData.uf_cd"
              filled
              readonly
              :label="t('ufCode')"
            />
          </v-col>
        </v-row>
        <v-row>
          <v-col md="6" cols="12">
            <v-text-field
              v-model="internalSipData.sechimoto_kanji_kaisyamei"
              filled
              readonly
              :label="t('kanji')"
            />
          </v-col>
          <v-col md="6" cols="12">
            <v-text-field
              v-model="internalSipData.sechimoto_kana_kaisyamei"
              filled
              readonly
              :label="t('furigana')"
            />
          </v-col>
        </v-row>
        <v-row align="center">
          <v-col md="6" cols="12">
            <v-select
              v-model="internalSipData.sip_option_service_cd"
              :items="sipServiceCodeOptions"
              item-text="text"
              item-value="value"
              :rules="sipServiceNameRule"
              :label="t('sipServiceName')"
              :no-data-text="t('messages.notFound')"
              :readonly="editMode"
              :filled="editMode"
              class="required-label"
              @input="changeSipService"
            />
          </v-col>
          <v-col md="6" cols="12">
            <v-text-field
              v-model="internalSipData.sip_option_service_cd"
              filled
              readonly
              :label="t('sipServiceCode')"
            />
          </v-col>
        </v-row>
        <v-row>
          <v-col md="6" cols="12">
            <app-date-picker
              v-model="internalSipData.moshikomi_uketsuke_date"
              :readonly="isCreatedSoFile()"
              :label="t('acceptDate')"
              :rules="acceptDateRule"
              required
            />
          </v-col>
          <v-col md="6" cols="12">
            <app-date-picker
              v-model="internalSipData.kaiyaku_uketsuke_date"
              :readonly="isCreatedKaiyakuSoFile()"
              :label="t('cancelAcceptDate')"
              :rules="cancelAcceptDateRule"
            />
          </v-col>
        </v-row>
        <v-row>
          <v-col md="6" cols="12">
            <app-date-picker
              v-model="internalSipData.service_start_date"
              :readonly="isCreatedSoFile()"
              :label="t('startDate')"
              :rules="startDateRule"
              :required="checkRequired(startDateRule)"
            />
          </v-col>
          <v-col md="6" cols="12">
            <app-date-picker
              v-model="internalSipData.service_end_date"
              :readonly="isCreatedKaiyakuSoFile()"
              :label="t('endDate')"
              :rules="endDateRule"
            />
          </v-col>
        </v-row>
        <v-row>
          <v-col md="6" cols="12">
            <v-text-field
              v-if="internalSipData.sip_option_service_cd==='10080060'"
              v-model="internalSipData.jido_ukai_tel_no"
              :label="t('autoDetourNumber')"
              :readonly="!otsuka_flg"
              :filled="!otsuka_flg"
              :rules="autoDetourNumberRule"
            />
          </v-col>
        </v-row>
      </v-card-text>
      <v-card-actions class="justify-space-between">
        <v-btn @click="close">
          {{ t('close') }}
        </v-btn>
        <v-btn :disabled="cancelStatus || (editMode ? !updatePermission : !createPermission)" color="primary" @click="confirm">
          {{ t(editMode ? 'change' : 'add') }}
        </v-btn>
      </v-card-actions>
    </v-card>
  </v-form>
</template>

<script lang="ts">
import Vue, { PropType } from 'vue';
import {
  AddEditSipData,
  SipData,
  ADD, EDIT,
  KINKYU_TSUHO_SERVICE,
  DEAI_NORMAL, DEAI_DEAI
} from '@/models/service-register';
import AppDatePicker from '../UI/AppDatePicker.vue';
import {
  MESSAGE,
  MESSAGE_ERROR,
  CONFIRMATION,
  REGISTER_UPDATE_EXECUTE,
  WHILE_INPUT,
} from '@/constants/dialog';
import { TranslateResult } from 'vue-i18n';
import {
  conditionRequired,
  dateRange,
  formatDate,
  formatTelNo,
  required,
} from '@/utils/validation';
import { SipOptionsResponse } from '@/models/customer';
import { FormRef, InputValidation } from '@/models/validation';
import { FukaServiceMstListAPI } from '@/api/fuka-service-mst-api';
import { advancedPhoneCheckAPI } from '@/api/advanced-phone-check';
import { DialogOptions } from '@/models/dialog';
import { getDateString, changeDateFormat } from '@/utils/helper';
import { ResponseData_UF_KYOKUSHA_DATA, responseData_FukaServiceMst }  from '@/models/service';
import { getScreenId } from '@/utils/screenIds';
import { dairitenCheckAPI } from '@/api/dairiten-check-api';
import { KishuMstAPI } from '@/api/kishu-mst-api';

export default Vue.extend({
  components: { AppDatePicker },
  props: {
    sipData: {
      type: Object as PropType<SipData>,
      default: null,
    },
    sipDataList: {
      type: Array as PropType<SipOptionsResponse[]>,
      default: [],
    },
    editMode: {
      type: Boolean,
      default: false,
    },
    cancelStatus: {
      type: Boolean,
      default: false,
    },
    customerData:{
      type: Object,
      default: () => ({}),
    }
  },
  data(): AddEditSipData {
    return {
      sipServiceCodeOptions: [],
      internalSipData: this.sipData,
      listSipOptionsData: this.sipDataList,
      valid: false,
      isDirty: false,
      service_start_date_before: this.sipData.service_start_date,
      uf_kyokusha_data: {} as ResponseData_UF_KYOKUSHA_DATA,
      areaCode: this.customerData.seti_ashk_tel_tp7k,
      kishu_cd: this.customerData.kishu_cd,
      createPermission: false,
      updatePermission: false,
      otsuka_flg: true,
    };
  },
  computed: {
    sipServiceNameRule(): InputValidation{
      return[
        required(this.internalSipData.sip_option_service_cd, this.t('sipServiceName')),
      ];
    },
    acceptDateRule(): InputValidation{
        const { moshikomi_uketsuke_date } = this.internalSipData;
        return [
          required(
            moshikomi_uketsuke_date,
            this.t('acceptDate')
          ),
          formatDate(moshikomi_uketsuke_date),
        ];
    },
    cancelAcceptDateRule(): InputValidation {
      const {
        moshikomi_uketsuke_date,
        kaiyaku_uketsuke_date,
        service_end_date,
        service_start_date,
        so_kaiyaku_data_sakusei_kubun
      } = this.internalSipData;
      return [
        conditionRequired(
          kaiyaku_uketsuke_date,
          ((so_kaiyaku_data_sakusei_kubun == "0" || so_kaiyaku_data_sakusei_kubun == "2") && service_end_date != ""),
          this.t('soKaiyakuDataSakuseiKubun') + "が未作成または解約対象で" + this.t('endDate'),
          this.t('cancelAcceptDate')
        ),
        formatDate(kaiyaku_uketsuke_date),
        dateRange(kaiyaku_uketsuke_date, service_start_date, null),
        dateRange(kaiyaku_uketsuke_date, moshikomi_uketsuke_date, null),
      ];
    },
    startDateRule(): InputValidation {
      const { moshikomi_uketsuke_date, service_start_date, so_data_sakusei_kubun, service_end_date } =
        this.internalSipData;
      const today = changeDateFormat(getDateString());

      let date: boolean | TranslateResult = ""
      // SOデータ未作成かつすでに入力されていた開始日と異なる場合、現在日付より前の日付は不可
      date = (so_data_sakusei_kubun == "0" && service_start_date !== this.service_start_date_before) ? 
            dateRange(service_start_date, today, null) : true
      return[
        formatDate(service_start_date),
        // 申込受付日より前は不可
        dateRange(service_start_date, moshikomi_uketsuke_date, null),
        // サービス終了日より後は不可
        dateRange(service_start_date, null, service_end_date),
        // 出合いが通常・出合い案件以外かつ緊急通報サービス以外を選択した場合は必須
        conditionRequired(
          service_start_date,
          this.internalSipData.sip_option_service_cd !== KINKYU_TSUHO_SERVICE && ![DEAI_NORMAL, DEAI_DEAI].includes(this.customerData.deai_status_kubun),
          '出合いが' + this.t('deai.normal') + '・' + this.t('deai.deai') + '以外で、' + this.t('kinkyu_tsuho_service') + '以外',
          this.t('startDate')
        ),
        date
      ]
    },
    endDateRule(): InputValidation {
      const { service_end_date, 
              service_start_date, 
              moshikomi_uketsuke_date, 
              so_kaiyaku_data_sakusei_kubun, 
              kaiyaku_uketsuke_date 
            } = this.internalSipData;
      return [
        conditionRequired(
          service_end_date,
          ((so_kaiyaku_data_sakusei_kubun == "0" || so_kaiyaku_data_sakusei_kubun == "2") && kaiyaku_uketsuke_date != ""),
          this.t('soKaiyakuDataSakuseiKubun') + "が未作成または解約対象で" + this.t('cancelAcceptDate'),
          this.t('endDate')
        ),
        formatDate(service_end_date),
        dateRange(service_end_date, moshikomi_uketsuke_date, null),
        dateRange(service_end_date, service_start_date, null),
      ];
    },
    autoDetourNumberRule(): InputValidation {
      const { jido_ukai_tel_no } = this.internalSipData;
      return [
        required(jido_ukai_tel_no, this.t('autoDetourNumber')),
        formatTelNo(this.internalSipData.jido_ukai_tel_no),
      ]
    }
  },
  watch: {
    internalSipData: {
      deep: true,
      handler() {
        console.log('WATCH CHANGED', this.isDirty);
        this.isDirty = true;
      },
    },
  },
  created() { 
    // サービス終了日がnullのオプションのサービスコードを取得
    let unableSerciveCode = this.listSipOptionsData.filter((item) => item.service_end_date == '').map((item) => item.sip_option_service_cd)
    const name = this.editMode ? "SIPOptionChangeDialog" : "SIPOptionAdditionDialog" 
    const screen_id = getScreenId(this.$route.name, name)
    const { juchu_no } = this.$route.query;
    this.createPermission = this.$$hasCreatePermission(screen_id)
    this.updatePermission = this.$$hasUpdatePermission(screen_id)
    Promise.all([
      FukaServiceMstListAPI(screen_id),
      advancedPhoneCheckAPI(screen_id, this.areaCode),
      dairitenCheckAPI(screen_id, juchu_no.toString()),
      KishuMstAPI(screen_id, this.customerData.kishu_cd)
    ])
      .then((res) => {
        // 代理店が大塚商会系列以外の場合は自動迂回サービス（10080060）は追加させない
        let service_list: responseData_FukaServiceMst[] = [];
        if (this.editMode) {
          service_list = res[0].data.filter(({protcol_kubun}) => protcol_kubun === '3')
        } else {
          service_list = res[0].data.filter(({protcol_kubun, fuka_service_cd}) => protcol_kubun === '3' && (res[2].data.result || fuka_service_cd !== '10080060'))
        }
        this.sipServiceCodeOptions = service_list.map(
          ({ fuka_service_cd, fuka_service_mei, protcol_kubun, service_kubun }) => ({
            value: fuka_service_cd,
            text: fuka_service_mei,
            protcol_kubun: protcol_kubun,
            service_kubun: service_kubun,
          })
        );
        // サービスコードの値で表示順ソート
        this.sipServiceCodeOptions.sort(function(a, b) {
          return Number(a.value) < Number(b.value) ? -1 : 1
        })
        this.uf_kyokusha_data = res[1].data;
        if( !this.uf_kyokusha_data ||
            this.uf_kyokusha_data.kinkyu_tsuho_kubun == "" ||
            this.customerData.protcol_kubun == '5' ||
            res[3].data.kinkyu_tsuho_kubun != '1'){
          let options = this.sipServiceCodeOptions.filter(function(item) {
            return item.value != "10080065";
          });
          this.sipServiceCodeOptions = options;
        }
        if(this.customerData.listOfSipOptions == undefined){
          let options = this.sipServiceCodeOptions.filter(function(item) {
            return item.service_kubun != "1";
          });
          this.sipServiceCodeOptions = options;
        }
        // 自身のサービスコードと異なり、すでに使用されていてサービス終了日がnullのオプションは除外
        if(unableSerciveCode.length > 0){
          let service_cd = this.internalSipData.sip_option_service_cd
          let options = this.sipServiceCodeOptions.filter(function(item) {
            return !(service_cd != item.value && unableSerciveCode.includes(item.value));
          });
          this.sipServiceCodeOptions = options;
        }
        // 大塚商会フラグ
        this.otsuka_flg = res[2].data.result
      })
  },
  methods: {
    isCreatedSoFile(): boolean {
      if (this.internalSipData.so_data_sakusei_kubun === '1') {
        return true
      } else {
        return false
      }
    },
    isCreatedKaiyakuSoFile(): boolean {
      if (this.internalSipData.so_kaiyaku_data_sakusei_kubun === '1') {
        return true
      } else {
        return false
      }
    },
    close(): void {
      if (this.isDirty) {
        this.$store.dispatch('dialog/open', {
          type: CONFIRMATION,
          subType: WHILE_INPUT,
          callback: () => this.$emit('close'),
        } as DialogOptions);
      } else {
        this.$emit('close');
      }
    },
    confirm(): void {
      //編集ステータスの追加
        console.log('confirm')
      if(this.editMode){
        // edit時 ステータスがADDになっていないときのみEDITに設定
        if(this.internalSipData.editStatus != ADD){
          this.internalSipData.editStatus = EDIT;
        }
      }else {
        // add時
        this.internalSipData.editStatus = ADD;
      }
      let checkRes = this.checkUnableServiceCd()
      // 条件に合うオプションの場合メッセージを出す
      if (checkRes == true) {
        this.$store.dispatch('dialog/open', {
          type: MESSAGE,
          subType: MESSAGE_ERROR,
          messages: [this.t('duplicationError')],
        } as DialogOptions);
        return
      }
      const valid = (this.$refs.form as FormRef).validate();
      if (valid) {
        console.log('valid')
        this.$store.dispatch('dialog/open', {
          type: CONFIRMATION,
          subType: REGISTER_UPDATE_EXECUTE,
          messages: [this.t(`messages.${this.editMode ? 'change' : 'add'}`)],
          callback: () => this.$emit('confirm', this.internalSipData),
        });
      }
    },
    checkUnableServiceCd(): boolean{
      // 変更の場合はチェック内
      if (this.editMode) {
        return false
      }
      let res = false
      let service_cd = this.internalSipData.sip_option_service_cd
      // 同じオプションのサービス終了日を取得
      let service_end_date_list = this.listSipOptionsData.filter((item) => item.sip_option_service_cd == service_cd).map((item) => item.service_end_date)
      // 同じオプションなければOK
      if (service_end_date_list.length > 0) {
        let moshikomi_uketsuke_date = this.internalSipData.moshikomi_uketsuke_date
        service_end_date_list.forEach((item) => {
          if (item >= moshikomi_uketsuke_date) {
            // 申込受付日より後の日付が合ったらNG
            res = true
          }
        })
      }
      return res
    },
    t(field: string): TranslateResult {
      return this.$t(`customerDetail.serviceRegister.addEditSip.${field}`);
    },
    changeSipService(fuka_cd: string){
      console.log(this.sipServiceCodeOptions?.filter(item => item.value === fuka_cd).map(item => item.text)[0])
      this.internalSipData.fuka_service_cd = this.sipServiceCodeOptions?.filter(item => item.value === fuka_cd).map(item => item.text)[0];
    },
    // 必須アイコン表示用関数
    checkRequired(rules: InputValidation): boolean {
      for (const item of rules) {
        if (item !== true) return true;
      }
      return false;
    },
  },
});
</script>
