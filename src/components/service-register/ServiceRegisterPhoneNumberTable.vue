<!--サービス情報登録_申込電番一覧-->
<template>
  <div>
    <div class="d-flex align-center flex-wrap mb-3">
      <h4>
        {{ t('applicationPhoneNumberList') }}
      </h4>
      <v-spacer />
      <div>
        <v-btn 
          color="error" 
          :disabled="notSave || cancelStatus || !updatePermission_cancel"
          @click="handleShowCancelIndividual()"
        >
          {{ t('individualContractCancellation') }}
        </v-btn>
      </div>
    </div>
    <v-data-table
      v-if="headers"
      ref="phoneTableRefs"
      :headers="headers"
      :items="listPhoneNumber"
      :hide-default-footer="true"
      :options.sync="tableOptions"
      class="elevation-1"
      :item-class="rowClass"
    >
      <template v-slot:[`item.so_shoki_data_sakusei_kubun`]="{ item }">
        {{ getSoStatus(item.so_shoki_data_sakusei_kubun) }}
      </template>
      <template v-slot:[`item.action`]="{ item }">
        <v-icon :disabled="!(updatePermission_phone || readPermission)" @click="onpenDialogListPhone(EDIT, item)">mdi-pencil</v-icon>
      </template>
      <template #top="{ pagination, options, updateOptions }">
        <v-data-footer
          :pagination="pagination"
          :options="options"
          @update:options="updateOptions"
        />
      </template>
      <template #no-data>
        <p>{{ t('messages.notFound') }}</p>
      </template>
    </v-data-table>
    <v-dialog v-model="showPhoneEdit" persistent width="1100">
      <service-register-add-edit-phone
        v-if="showPhoneEdit"
        :phone-data="phoneData"
        :cancel-status="cancelStatus"
        :customer-data="customerData"
        :list-phone-table="listPhoneNumber"
        :list-numbered-tentative="listNumberedTentative"
        :list-routing-numbered-tentative="listRoutingNumberedTentative"
        :protcol-kubun="customerData.protcol_kubun"
        :banpo-flg="banpoFlg"
        edit-mode
        @close="closeDialog"
        @confirm="handleConfirm"
      />
    </v-dialog>
    <v-dialog v-model="showCancelIndividual" persistent width="1100">
      <service-register-cancel-individual-contract
        v-if="showCancelIndividual"
        :individual-contract-cancel-data="individualContractCancel"
        @close="showCancelIndividual = false"
        @cancel-cancellation="handleCancelCancellation"
        @cancel="handleCancellation"
      />
    </v-dialog>
  </div>
</template>

<script lang="ts">
import { MODE_CREATE, FLAG_ADDED, FLAG_UPDATED, DEFAULT_HYOJI_NUM } from '@/constant/common-constant';
import Vue, { PropType } from 'vue';
import { PhoneNumberResponse } from '@/models/customer';
import ServiceRegisterAddEditPhone from '@/components/service-register/ServiceRegisterAddEditPhone.vue';
import { TranslateResult } from 'vue-i18n';
import { DataTableHeader } from 'vuetify';
import { mapState } from 'vuex';
import ServiceRegisterCancelIndividualContract from '@/components/service-register/ServiceRegisterCancelIndividualContract.vue';
import {
  IndividualContractCancelData,
  IndividualContractCancelPhone,
  EDIT,
  STATUS_SHINKI,
  STATUS_KAIYAKU,
  SO_NOT_CREATE,
  SO_TARGET_KAIYAKU,
  SIP,
  DAIHYO_DENBAN,
  KO_DENBAN,
} from '@/models/service-register';
import { cancelAPI, cancelCancellationAPI } from '@/api/cancel-info-api';
import { cancelRequestdata, cancelCancellationRequestdata } from '@/models/cancel-info';
import {
  AFTER_CANCEL,
  CONFIRMATION,
} from '@/constants/dialog';
import { getScreenId } from '@/utils/screenIds';
import { ServiceApplicationResponse } from '@/models/customer'
import { carryDownRejiNo, checkTelNoCancelStatus } from '@/utils/helper';
import { KubunListAPI, ResponseData_KUBUN_KANRI } from '@/api/kubun-kanri';
import { AxiosError } from 'axios';
import { SelectOption } from '@/models/service-register';
import dateFormat from "dateformat";

export default Vue.extend({
  components: {
    ServiceRegisterAddEditPhone,
    ServiceRegisterCancelIndividualContract,
  },
  props: {
    customerData: {
      type: Object,
      default: () => ({}),
    },
    mode: {
      type: String,
      default: MODE_CREATE,
    },
    listPhoneNumber: {
      type: Array as PropType<PhoneNumberResponse[]>,
      default: null,
    },
    notSave: {
      type: Boolean,
      default: null
    },
    listOfService: {
      type: Array as PropType<ServiceApplicationResponse[]>,
      default: null,
    },
    cancelStatus: {
      type: Boolean,
      default: false,
    },
    screenId: {
      type: String,
      default: null,
    },
    listNumberedTentative: {
      type: [] as PropType<string[]>,
      default: null,
    },
    listRoutingNumberedTentative: {
      type: [] as PropType<string[]>,
      default: null
    },
    banpoFlg: {
      type: Boolean,
      default: false
    },
  },
  data() {
    return {
      showCancelIndividual: false,
      showPhoneEdit: false,
      dataService: null,
      modeDialog: '',
      phoneData: {} as PhoneNumberResponse,
      updatePermission_phone: false,
      readPermission: false,
      updatePermission_cancel: false,
      motoRejiNo: null as null | number,
      tableOptions: {
        page: 1,
        itemsPerPage: DEFAULT_HYOJI_NUM,
        sortBy: ['g_serial'],
        sortDesc: [],
        groupBy: [],
        groupDesc: [],
        multiSort: true,
        mustSort: false,
      },
      so_sakusei_kubun: [] as SelectOption[],
    };
  },
  computed: {
    ...mapState('customer', ['bbJuchu']),
    headers(): (DataTableHeader | { text: TranslateResult })[] {
      return [
        {
          text: this.t('no'),
          value: 'g_serial',
        },
        {
          text: this.t('IPPhoneNumber'),
          value: 'ip_tel_no',
        },
        {
          text: this.t('businessPersonPhoneNumber'),
          value: 'routing_tel_no',
        },
        {
          text: this.t('linkedNumber'),
          value: 'himozuki_tel_no',
        },
        {
          text: this.t('registrationNumber'),
          value: 'reji_no',
        },
        {
          text: this.t('representativeSection'),
          value: 'daihyo_kubun',
        },
        {
          text: this.t('phoneNumberIssuanceNotification'),
          value: 'bango_tsuchi_kubun',
        },
        {
          text: this.t('incomingCallsNumber'),
          value: 'chakushin_num',
        },
        {
          text: this.t('phoneBook104GuidancePostNumber'),
          value: 'n104_annai_kubun',
        },
        {
          text: this.t('name'),
          value: 'kaisen_meigi',
        },
        {
          text: this.t('contractCancellationAcceptanceDate'),
          value: 'kaiyaku_uketsuke_date',
        },
        {
          text: this.t('soShokiDataSakuseiKubun'),
          value: 'so_shoki_data_sakusei_kubun',
        },
        {
          text: this.t('applicationPhoneNumberEdit'),
          value: 'action',
        },
      ];
    },
    individualContractCancel(): IndividualContractCancelData {
      const {juchu_no, juchu_meisai_no } = this.$route.query;
      return {
        juch_no: juchu_no.toString(),
        juch_mesai_no: juchu_meisai_no.toString(),
        gno: this.customerData.gno,
        gserialNo: this.customerData.serial_no,
        ufCode: this.customerData.uf_cd,
        kanji: this.customerData.seti_kanji_shamei,
        furigana: this.customerData.seti_kn,
        billingEndDate: "",
        contractCancelAcceptDate: dateFormat(new Date(),"yyyy/mm/dd"),
        phonelist: this.convPhonelist(),
        editStatus: "",
      }
    },
    EDIT() {
      return EDIT;
    }
  },
  created() {
    const update_name = "ApplicationPhoneNumberChangeDialog"
    const cancel_name = "IndividualCancellationRegistrationDialog"
    const screen_ids = { 
      "update": getScreenId(this.$route.name, update_name),
      "cancel": getScreenId(this.$route.name, cancel_name),
    }
    this.updatePermission_phone = this.$$hasUpdatePermission(screen_ids["update"]);
    this.readPermission = this.$$checkPermission(screen_ids["update"]);
    this.updatePermission_cancel = this.$$hasUpdatePermission(screen_ids["cancel"]);
    KubunListAPI(getScreenId(this.$route.name), 'SO_DATA_SAKUSEI_KUBUN')
    .then((res) => {
      this.so_sakusei_kubun = this.kubunToSelect(res.data);
    })
    .catch((err: AxiosError) => {
      console.error(err.response);
    });
  },
  methods: {
    onpenDialogListPhone(mode: string, item: PhoneNumberResponse): void {
      this.modeDialog = mode;
      console.log(item);
      let service_cd = "";
      this.listOfService.forEach((service) => {
        if(service.no === item.g_serial){
          service_cd = service.serviceApplicationCode;
        }
      })
      if (mode == EDIT) {
        this.phoneData = {
          id: item.id,
          gno: item.gno,
          g_serial: item.g_serial,
          uf_cd: item.uf_cd,
          kanji: item.kanji,
          furigana: item.furigana,
          ip_ashk_tel_no: item.ip_ashk_tel_no,
          ip_tel_no: item.ip_tel_no,
          routing_tel_no: item.routing_tel_no,
          routing_ashk_tel_no: item.routing_ashk_tel_no,
          reji_no: item.reji_no,
          daihyo_kubun: item.daihyo_kubun,
          bango_tsuchi_kubun: item.bango_tsuchi_kubun,
          himozuki_change: item.himozuki_change,
          himozuki_tel_no: item.himozuki_tel_no,
          himozuki_ashk_tel_no: item.himozuki_ashk_tel_no,
          chakushin_num: item.chakushin_num,
          n104_annai_kubun: item.n104_annai_kubun,
          kaisen_meigi: item.kaisen_meigi,
          kaisen_meigi_kana: item.kaisen_meigi_kana,
          keisai_mei: item.keisai_mei,
          keisai_mei_kana: item.keisai_mei_kana,
          ntt_np: item.ntt_np,
          ntt_np_date: item.ntt_np_date,
          juchu_kubun: item.juchu_kubun,
          kaiyaku_uketsuke_date: item.kaiyaku_uketsuke_date,
          so_kaiyaku_data_sakusei_kubun: item.so_kaiyaku_data_sakusei_kubun,
          so_shoki_data_sakusei_kubun: item.so_shoki_data_sakusei_kubun,
          update_flg: item.update_flg,
          add_flg: item.add_flg,
          service_cd: service_cd,
          editStatus: item.editStatus,
          hold_flg: item.hold_flg,
          old_ip_ashk_tel_no: item.old_ip_ashk_tel_no,
          old_himozuki_tel_no: item.himozuki_tel_no,
        }
      }
      this.motoRejiNo = item.reji_no // 変更前のレジ番を持っておく
      this.showPhoneEdit = true;
    },
    handleConfirm(data: PhoneNumberResponse) {
      console.log("confirm")
      console.log(data)
      // 紐づき番号チェックがない場合は空文字を登録
      if (data.himozuki_change == false && this.customerData.protcol_kubun != '5') {
        data.himozuki_tel_no = ''
      }
      if (this.modeDialog === EDIT) {
        const dataEdit = this.listPhoneNumber.find(
          (item) => item.id === data.id
        );
        if (dataEdit) {
          dataEdit.gno = data.gno;
          dataEdit.g_serial = data.g_serial;
          dataEdit.uf_cd = data.uf_cd;
          dataEdit.kanji = data.kanji;
          dataEdit.furigana = data.furigana;
          dataEdit.ip_tel_no = data.ip_tel_no;
          dataEdit.ip_ashk_tel_no = data.ip_tel_no.replace(/-/g,"");
          dataEdit.routing_tel_no = data.routing_tel_no;
          dataEdit.routing_ashk_tel_no = data.routing_tel_no.replace(/-/g,"");
          dataEdit.reji_no = data.reji_no;
          dataEdit.daihyo_kubun = data.daihyo_kubun;
          dataEdit.bango_tsuchi_kubun = data.bango_tsuchi_kubun;
          dataEdit.himozuki_ashk_tel_no = data.himozuki_tel_no == null ? "":data.himozuki_tel_no.replace(/-/g,"");
          dataEdit.himozuki_tel_no = data.himozuki_tel_no;
          dataEdit.himozuki_change = data.himozuki_change
          dataEdit.chakushin_num = data.chakushin_num == null ? "":data.chakushin_num.toString();
          dataEdit.n104_annai_kubun = data.n104_annai_kubun;
          dataEdit.kaisen_meigi = data.kaisen_meigi;
          dataEdit.kaisen_meigi_kana = data.kaisen_meigi_kana;
          dataEdit.keisai_mei = data.keisai_mei;
          dataEdit.keisai_mei_kana = data.keisai_mei_kana;
          dataEdit.ntt_np = data.ntt_np;
          dataEdit.ntt_np_date = data.ntt_np_date;
          dataEdit.juchu_kubun = data.juchu_kubun;
          dataEdit.kaiyaku_uketsuke_date = data.kaiyaku_uketsuke_date;
          if(FLAG_ADDED != dataEdit.add_flg)
          {
            dataEdit.update_flg = FLAG_UPDATED;
          }
          dataEdit.hold_flg = data.hold_flg;
          // レジ番登録されている場合繰り下げ
          if (data.reji_no != null) {
            // SO作成済みの場合かつレジ番が変更されているときのみレジ番変更フラグを立てる
            if (this.customerData.deai_status_kubun == '2' && dataEdit.reji_no !== data.reji_no) {
              // レジ番変更フラグをON
              this.$emit('listOfphoneNumber-changed')
            }
            let carryDownList = carryDownRejiNo(data.reji_no,data.id,this.motoRejiNo,this.listPhoneNumber,this.customerData.board_num)
            for (let i=0; this.listPhoneNumber.length>i; i++) {
              this.listPhoneNumber[i] = carryDownList[i]
            }
          }
          // 代表区分修正
          if( data.daihyo_kubun === DAIHYO_DENBAN)
          {
            this.listPhoneNumber.forEach((item)=>{
              if(data.id !== item.id)
              {
                item.daihyo_kubun = KO_DENBAN
                item.update_flg = FLAG_UPDATED
                console.log("子電番へ変更," + item.ip_ashk_tel_no)
              }
            })
          }
        }
      }
      // 紐づき番号の変更があった場合古い紐づき番号をリセットする
      if(data.himozuki_tel_no !== data.old_himozuki_tel_no){
        this.listPhoneNumber.forEach((target) => {
          if(target.ip_tel_no === data.old_himozuki_tel_no && target.himozuki_tel_no === data.ip_tel_no){
            target.himozuki_tel_no = ""
            target.himozuki_ashk_tel_no = ""
            target.update_flg = FLAG_UPDATED
          }
        })
      }
      // 紐づき番号が設定されている場合、その紐づき番号と設定している番号で互いに紐づける
      if(data.himozuki_tel_no){
        this.listPhoneNumber.forEach((target) => {
          if(target.ip_tel_no === data.himozuki_tel_no){
            target.himozuki_tel_no = data.ip_tel_no
            target.himozuki_ashk_tel_no = data.ip_ashk_tel_no
            target.update_flg = FLAG_UPDATED
          }
        })
      }
      this.showPhoneEdit = false;
      this.$emit("numbered")
    },
    closeDialog() {
      this.showPhoneEdit = false;
    },
    t(field: string): TranslateResult {
      return this.$t(`customerDetail.serviceRegister.${field}`);
    },
    handleShowCancelIndividual() {
      this.showCancelIndividual = true;
    },
    // 個別解約取消実行
    handleCancelCancellation(data: IndividualContractCancelData) {
      console.log("handleCancelCancellation");
      // ダイアログを閉じる
      this.showCancelIndividual = false;
      const requestDatas:cancelCancellationRequestdata = this.convertCancelCancellationRequestdata(data);
      console.log(requestDatas)
      Promise.all([
        cancelCancellationAPI(getScreenId(this.$route.name), requestDatas),
      ]).then((res) => {
        console.log(res);
        this.updatePhoneList(data, false)
        if(this.$emit('auth', 'InfoRegisterComplete')){
          this.$store.dispatch('dialog/open', {
            type: CONFIRMATION,
            subType: AFTER_CANCEL,
            messages: [
              this.t(`messages.cancel`),
            ],
            callback: () =>
              this.$emit('move-completion'),
          });
        }
        this.$emit('reload')
      });
    },
    // 個別解約実行
    handleCancellation(data: IndividualContractCancelData) {
      console.log("handleCancellation");
      // ダイアログを閉じる
      this.showCancelIndividual = false;
      const requestDatas:cancelRequestdata = this.convertCancelRequestdata(data);
      console.log(requestDatas)
      const screen_id = getScreenId(this.$route.name)
      Promise.all([
        cancelAPI(screen_id, requestDatas),
      ]).then((res) => {
        console.log(res);
        this.updatePhoneList(data, true)
        if(this.$emit('auth', 'InfoRegisterComplete')){
          this.$store.dispatch('dialog/open', {
            type: CONFIRMATION,
            subType: AFTER_CANCEL,
            messages: [
              this.t(`messages.cancel`),
            ],
            callback: () =>
              this.$emit('move-completion'),
          });
          this.$emit('reload')
        }
      });
    },
    // 個別解約取消リクエスト用の構造体に変換
    convertCancelCancellationRequestdata(data :IndividualContractCancelData): cancelCancellationRequestdata {
      const convDatas: cancelCancellationRequestdata = {
        gno: data.gno,
        ip_ashk_tel_no_values: [],
        juch_no: data.juch_no,
        juch_mesai_no: data.juch_mesai_no,
      };
      data.phonelist.forEach((element) => {
        convDatas.ip_ashk_tel_no_values.push(
          {
            ip_ashk_tel_no: element.publishedNo.replace(/-/g, ""),
            serial_no: element.no
          }
        )
      });
      return convDatas;
    },
    // 個別解約リクエスト用の構造体に変換
    convertCancelRequestdata(data :IndividualContractCancelData): cancelRequestdata {
      const convDatas: cancelRequestdata = {
        gno: data.gno,
        ip_ashk_tel_no_values: [],
        kaiyaku_uketsuke_date: data.contractCancelAcceptDate,
        kakin_end_date: data.billingEndDate,
        juch_no: data.juch_no,
        juch_mesai_no: data.juch_mesai_no,
      };
      data.phonelist.forEach((element) => {
        convDatas.ip_ashk_tel_no_values.push(
          {
            ip_ashk_tel_no: element.publishedNo.replace(/-/g, ""),
            serial_no: element.no
          }
        )
      });
      return convDatas;
    },
    convPhonelist(): IndividualContractCancelPhone[] {
      const convData: IndividualContractCancelPhone[] = [];
      let i = 0;
      this.listPhoneNumber.forEach((element) => {
        const data: IndividualContractCancelPhone = {
          id: i + "",
          no: element.g_serial,
          publishedNo: element.ip_tel_no,
          routingNo: element.routing_tel_no,
          linkedNo: element.himozuki_tel_no,
          registrationNo: element.reji_no,
          daihyo_kubun: element.daihyo_kubun,
          callNoNotificationSection: element.bango_tsuchi_kubun,
          noIncomingCalls: element.chakushin_num,
          name: element.kaisen_meigi,
          contractCancelDate: element.kaiyaku_uketsuke_date,
          juchu_kubun: element.juchu_kubun,
          so_kaiyaku_sakusei_kubun: element.so_kaiyaku_data_sakusei_kubun,
          so_shoki_data_sakusei_kubun: element.so_shoki_data_sakusei_kubun,
        }
        convData.push(data)
        i++;
      })
      return convData;
    },
    // 解約・解約取消時,フロント上の申込電番情報を合わせて変更
    // cancel_statusは解約時true,取消時falseとして判定する
    updatePhoneList(data: IndividualContractCancelData, cancel_status: boolean){
      let kaiyaku_uketsuke_date = data.contractCancelAcceptDate;
      data.phonelist.forEach((c_phone) => {
        let so_kaiyaku_data_sakusei_kubun = c_phone.so_kaiyaku_sakusei_kubun;
        if(this.customerData.protcol_kubun != SIP && c_phone.so_shoki_data_sakusei_kubun != SO_NOT_CREATE){
          so_kaiyaku_data_sakusei_kubun = SO_TARGET_KAIYAKU;
        }
        this.listPhoneNumber.forEach((phone) => {
          if(phone.ip_tel_no === c_phone.publishedNo){
            phone.juchu_kubun = cancel_status ? STATUS_KAIYAKU : STATUS_SHINKI;
            phone.kaiyaku_uketsuke_date = cancel_status ? kaiyaku_uketsuke_date : "";
            phone.so_kaiyaku_data_sakusei_kubun = cancel_status ? so_kaiyaku_data_sakusei_kubun : SO_NOT_CREATE;
          }
        })
      })
    },
    rowClass(rowItem: PhoneNumberResponse): string{
      let sameHimozukitelNo = this.listPhoneNumber.filter((item) => (checkTelNoCancelStatus(item.so_kaiyaku_data_sakusei_kubun) && item.juchu_kubun !== '9') && item.himozuki_tel_no == rowItem.himozuki_tel_no)
      if (rowItem.himozuki_tel_no && sameHimozukitelNo.length >= 2 && (checkTelNoCancelStatus(rowItem.so_kaiyaku_data_sakusei_kubun) && rowItem.juchu_kubun !== '9')) {
        return 'red-row'
      }
      // 解約電番は背景を灰色にする
      if (rowItem.juchu_kubun === STATUS_KAIYAKU){
        return 'kaiyaku-style'
      }
      return ''
    },
    // 区分管理からセレクトボックスへ変換
    kubunToSelect(kubunList: ResponseData_KUBUN_KANRI[]): SelectOption[] {
      return kubunList.map(({ vl, hyoji_mei }) => ({
        value: vl,
        text: hyoji_mei,
      }))
    },
    getSoStatus(status: string): TranslateResult{
      let so_sakusei_kubun = this.so_sakusei_kubun.filter((item: SelectOption) => item.value === status)
      if (so_sakusei_kubun.length > 0) {
        return so_sakusei_kubun[0].text
      }
      return ''
    }
  },
});
</script>

<style>
.red-row {
  background: #f77b88 !important;
}
.kaiyaku-style {
  background-color: #afafaf !important;
}
.kaiyaku-style td {
  color: #ffffff;
}
</style>