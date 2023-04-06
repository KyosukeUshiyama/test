<!--サービス情報登録_申込サービス一覧-->
<template>
  <div>
    <div class="d-flex align-center flex-wrap mb-3">
      <h4>{{ t('serviceApplicationList') }}</h4>
      <v-spacer />
      <div>
        <v-btn 
          color="error" 
          :disabled="notSave || !updatePermission_cancel"
          @click="showDialogCancelAll = true" 
        >
          {{ t('cancelAllContracts') }}
        </v-btn>
        <v-btn
          :disabled="cancelStatus || !createPermission"
          color="primary"
          class="ml-2"
          @click="onpenDialogServiceApplication(ADD)"
        >
          {{ t('addService') }}
        </v-btn>
      </div>
    </div>
    <v-data-table
      item-key="no"
      :headers="headers"
      :items="listOfService"
      :hide-default-footer="true"
      :options.sync="tableOptions"
      class="elevation-1"
    >
      <template v-slot:[`item.action`]="{ item }">
        <v-icon :disabled="!(updatePermission_service || readPermission)" @click="onpenDialogServiceApplication(EDIT, item)">
          mdi-pencil
        </v-icon>
      </template>
      <template #top="{ pagination, options, updateOptions }">
        <v-data-footer
          :pagination="pagination"
          :options="options"
          items-per-page-text="$vuetify.dataTable.itemsPerPageText"
          @update:options="updateOptions"
        />
      </template>
      <template #no-data>
        <p>{{ t('messages.notFound') }}</p>
      </template>
    </v-data-table>
    <v-dialog v-model="showDialogAddService" width="1200" persistent>
      <service-register-add-edit-service
        v-if="showDialogAddService"
        :edit-mode="mode === EDIT"
        :service-data-list="listOfService"
        :service-data="mode === ADD ? serviceDataCreate : serviceDataEdit"
        :customer-data="customerData"
        :list-phone-number="listPhoneNumber"
        :list-numbered-tentative="listNumberedTentative"
        :list-routing-numbered-tentative="listRoutingNumberedTentative"
        :banpo-flg="banpoFlg"
        :cancel-status="cancelStatus"
        @close="closeDialog"
        @accept="acceptData"
      />
    </v-dialog>
    <v-dialog v-model="showDialogCancelAll" width="1200" persistent>
      <cancel-all-contracts-component
        v-if="showDialogCancelAll"
        :cancel-all-data="datas"
        :cancel-status="cancelStatus"
        @close="showDialogCancelAll = false"
        @cancel-cancellation="cancelCancellation"
        @cancel="cancelData"
        @change-cancel-status="changeCancelStatus"
      />
    </v-dialog>
  </div>
</template>

<script lang="ts">
import {
  RegistrationServiceApplicationTableState,
  ServiceApplicationResponse,
  PhoneNumberResponse,
} from '@/models/customer';
import Vue, { PropType } from 'vue';
import ServiceRegisterAddEditService from '@/components/service-register/ServiceRegisterAddEditService.vue';
import { TranslateResult } from 'vue-i18n';
import { DataTableHeader } from 'vuetify';
import CancelAllContractsComponent from '@/components/service-register/CancelAllContractsComponent.vue';
import { ServiceDataForm } from '@/models/application-service';
import { CancelAllDataReponse, ADD, EDIT } from '@/models/service-register';
import { allCancelAPI, allCancelCancellationAPI } from '@/api/cancel-info-api';
import { allCancelRequestdata, allCancelCancellationRequestdata } from '@/models/cancel-info';
import { FLAG_UPDATED, FLAG_ADDED, FLAG_NORMAL, DEFAULT_HYOJI_NUM } from '@/constant/common-constant';
import {
  AFTER_CANCEL,
  CONFIRMATION,
} from '@/constants/dialog';
import { getScreenId } from '@/utils/screenIds';
import dateFormat from "dateformat";

export default Vue.extend({
  components: {
    ServiceRegisterAddEditService,
    CancelAllContractsComponent,
  },
  props: {
    customerData: {
      type: Object,
      default: () => ({}),
    },
    listOfService: {
      type: Array as PropType<ServiceApplicationResponse[]>,
      default: [],
    },
    listPhoneNumber: {
      type: Array as PropType<PhoneNumberResponse[]>,
      default: null,
    },
    notSave: {
      type: Boolean,
      default: null
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
  data(): RegistrationServiceApplicationTableState {
    return {
      mode: '',
      showDialogCancelAll: false,
      serviceDataCreate: {
        service_data_form: {
          id: new Date().getTime() + '',
          gno: '',
          serial_no: '',
          uf_cd: '',
          sechimoto_kanji_kaisyamei: '',
          sechimoto_kana_kaisyamei: '',
          fuka_service_cd: '',
          fuka_service_mei: '',
          protocol_kubun: '',
          moshikomi_date: '',
          koji_kanryo_date: '',
          kaiyaku_date: '',
          ch_num: 0,
          jigyosha_moshiokuri_jiko: '',
          editStatus: '',
        },
        list_service_data: [],
      },
      serviceDataEdit: {
        service_data_form: {
          id: new Date().getTime() + '',
          gno: '',
          serial_no: '',
          uf_cd: '',
          sechimoto_kanji_kaisyamei: '',
          sechimoto_kana_kaisyamei: '',
          fuka_service_cd: '',
          fuka_service_mei: '',
          protocol_kubun: '',
          moshikomi_date: '',
          koji_kanryo_date: '',
          kaiyaku_date: '',
          ch_num: 1,
          jigyosha_moshiokuri_jiko: '',
          editStatus: '',
        },
        list_service_data: [],
      },
      showDialogAddService: false,
      isOpenDialog: false,
      dataService: null,
      listOfServiceData: this.listOfService,
      updatePermission_service: false,
      createPermission: false,
      readPermission: false,
      updatePermission_cancel: false,
      tableOptions: {
        page: 1,
        itemsPerPage: DEFAULT_HYOJI_NUM,
        sortBy: ['no', 'applicationDate'],
        sortDesc: [],
        groupBy: [],
        groupDesc: [],
        multiSort: true,
        mustSort: false,
      },
    };
  },
  computed: {
    headers(): (DataTableHeader | { text: TranslateResult })[] {
      return [
        {
          text: this.t('no'),
          value: 'no',
        },
        {
          text: this.t('serviceApplicationName'),
          value: 'serviceApplicationName',
        },
        {
          text: this.t('applicationDate'),
          value: 'applicationDate',
        },
        {
          text: this.t('registrationDate'),
          value: 'registrationDate',
        },
        {
          text: this.t('completionDate'),
          value: 'completionDate',
        },
        {
          text: this.t('contractCancellationDate'),
          value: 'contractCancellationDate',
        },
        {
          text: this.t('numberCH'),
          value: 'numberCH',
        },
        {
          text: this.t('serviceApplicationEdit'),
          value: 'action',
        },
      ];
    },
    datas(): CancelAllDataReponse {
      const {juchu_no, juchu_meisai_no } = this.$route.query;
      return {
        gno: this.customerData.gno,
        juch_no: juchu_no.toString(),
        juch_mesai_no: juchu_meisai_no.toString(),
        ufCode: this.customerData.uf_cd,
        kanji: this.customerData.seti_kanji_shamei,
        furigana: this.customerData.seti_kn,
        carrier_status: this.customerData.carrier_status,
        cancellationRequester: '',
        cancellationSection: '',
        cancellationReasonCode: '',
        cancellationReasonContent: '',
        cancellationRequesterMemo: '',
        cancellationSectionCode: '',
        cancellationSectionContent: '',
        dateOfService: '',
        removalDeadline: '',
        contactOfConstructionRemoval: '設置先と同様',
        ipLine: '',
        billingEndDate: '',
        contractCancellationAcceptanceDate: '',
        constructionScheduleDate: '',
        sakjo_flg: '',
        editStatus: '',
      };
    },
    ADD() {
      return ADD;
    },
    EDIT() {
      return EDIT;
    }
  },
  created() {
    const create_name = "ApplicationServiceAdditionDialog"
    const update_name = "ApplicationServiceChangeDialog"
    const cancel_name = "AllCancelRegistrationDialog"
    const screen_ids = { 
      "create": getScreenId(this.$route.name, create_name),
      "update": getScreenId(this.$route.name, update_name),
      "cancel": getScreenId(this.$route.name, cancel_name),
    }
    this.updatePermission_service = this.$$hasUpdatePermission(screen_ids["update"]);
    this.createPermission = this.$$hasCreatePermission(screen_ids["create"]);
    this.readPermission = this.$$checkPermission(screen_ids["update"]);
    this.updatePermission_cancel = this.$$hasUpdatePermission(screen_ids["cancel"]);
  },
  methods: {
    onpenDialogServiceApplication(
      mode: string,
      item: ServiceApplicationResponse
    ): void {
      this.mode = mode;
      this.showDialogAddService = true;
      if (mode === ADD) {
        let maxSerialNo = '0';
        this.listOfService.forEach((element) => {
          if (typeof(element.no) == "string" && Number(element.no) >= Number(maxSerialNo)){
            maxSerialNo = element.no;
          }
        })
        this.serviceDataCreate.service_data_form = this.setServiceData(null, maxSerialNo)
        this.serviceDataCreate.list_service_data = this.convertListServiceDataFromResponse();
      }
      if (mode === EDIT) {
        this.serviceDataEdit.service_data_form = this.setServiceData(item, null);
        this.serviceDataEdit.list_service_data = this.convertListServiceDataFromResponse();
      }
    },
    convertListServiceDataFromResponse(): PhoneNumberResponse[] {
      const conv: PhoneNumberResponse[] = [];
      this.listPhoneNumber.forEach((element) => {
        const convData: PhoneNumberResponse = {
          gno: element.gno,
          id: element.id,
          g_serial: element.g_serial,
          uf_cd: element.uf_cd,
          kanji: element.kanji,
          furigana: element.furigana,
          ip_tel_no: element.ip_tel_no,
          ip_ashk_tel_no: element.ip_ashk_tel_no,
          routing_tel_no: element.routing_tel_no,
          routing_ashk_tel_no: element.routing_ashk_tel_no,
          himozuki_tel_no: element.himozuki_tel_no,
          himozuki_ashk_tel_no: element.himozuki_ashk_tel_no,
          himozuki_change: element.himozuki_change,
          reji_no: element.reji_no ? Number(element.reji_no) : null,
          daihyo_kubun: element.daihyo_kubun,
          bango_tsuchi_kubun: element.bango_tsuchi_kubun,
          chakushin_num: element.chakushin_num,
          n104_annai_kubun: element.n104_annai_kubun,
          kaisen_meigi: element.kaisen_meigi,
          kaisen_meigi_kana: element.kaisen_meigi_kana,
          keisai_mei: element.keisai_mei,
          keisai_mei_kana: element.keisai_mei_kana,
          ntt_np: element.ntt_np,
          ntt_np_date: element.ntt_np_date,
          juchu_kubun: element.juchu_kubun,
          kaiyaku_uketsuke_date: element.kaiyaku_uketsuke_date,
          so_kaiyaku_data_sakusei_kubun: element.so_kaiyaku_data_sakusei_kubun,
          so_shoki_data_sakusei_kubun: element.so_shoki_data_sakusei_kubun,
          update_flg: element.update_flg,
          add_flg: element.add_flg,
          service_cd: element.service_cd,
          editStatus: element.editStatus,
          hold_flg: element.hold_flg,
          old_ip_ashk_tel_no: element.old_ip_ashk_tel_no,
          old_himozuki_tel_no: element.himozuki_tel_no,
        };
        conv.push(convData);
      });
      return conv;
    },
    closeDialog(): void {
      this.showDialogAddService = false;
    },
    t(field: string): TranslateResult {
      return this.$t(`customerDetail.serviceRegister.${field}`);
    },
    // 申込サービス追加ダイアログからデータを受け取りダイアログを閉じる
    acceptData(data: ServiceDataForm, phonedata: PhoneNumberResponse[]): void {
      console.log('acceptData');
      console.log(data);
      let dataForm: ServiceApplicationResponse = {
        id: new Date().getTime() + '',
        no: data.serial_no,
        applicationDate: data.moshikomi_date,
        serviceApplicationName: data.fuka_service_mei,
        serviceApplicationCode: data.fuka_service_cd,
        protcol_kubun: data.protocol_kubun ? data.protocol_kubun: "",
        registrationDate: data.koji_kanryo_date,
        completionDate: data.koji_kanryo_date,
        contractCancellationDate: data.kaiyaku_date,
        numberCH: data.ch_num,
        jigyosha_moshiokuri_jiko: data.jigyosha_moshiokuri_jiko,
        add_flg:FLAG_ADDED,
        update_flg:FLAG_NORMAL,
      };
      if (this.mode === EDIT) {
        const dataEdit = this.listOfService.find(
          (item) => item.no === data.serial_no
        );
        if (dataEdit) {
          dataEdit.id = data.id;
          dataEdit.applicationDate = data.moshikomi_date;
          dataEdit.serviceApplicationName = data.fuka_service_mei;
          dataEdit.serviceApplicationCode = data.fuka_service_cd;
          dataEdit.numberCH = data.ch_num;
          dataEdit.registrationDate= data.koji_kanryo_date;
          dataEdit.completionDate = data.koji_kanryo_date;
          dataEdit.contractCancellationDate = data.kaiyaku_date;
          dataEdit.jigyosha_moshiokuri_jiko = data.jigyosha_moshiokuri_jiko;
          if(FLAG_ADDED != dataEdit.add_flg)
          {
            dataEdit.update_flg = FLAG_UPDATED;
          }
        }
      } else {
        this.listOfService.push(dataForm);
      }
      phonedata.forEach((phone) => {
        if (this.listPhoneNumber.length == 0) {
            // 元のリストが空の場合は追加
            this.listPhoneNumber.push(phone);
        } else  {
          for(let index = 0; index < this.listPhoneNumber.length; index++) {
            if(this.listPhoneNumber.length != 0 && phone.id === this.listPhoneNumber[index].id){
              // 同じデータがあれば書き換える
              this.listPhoneNumber[index].ip_tel_no = phone.ip_tel_no;
              this.listPhoneNumber[index].routing_tel_no = phone.routing_tel_no;
              this.listPhoneNumber[index].reji_no = phone.reji_no;
              this.listPhoneNumber[index].daihyo_kubun = phone.daihyo_kubun;
              this.listPhoneNumber[index].bango_tsuchi_kubun = phone.bango_tsuchi_kubun;
              this.listPhoneNumber[index].himozuki_tel_no = phone.himozuki_tel_no;
              this.listPhoneNumber[index].chakushin_num = phone.chakushin_num;
              this.listPhoneNumber[index].n104_annai_kubun = phone.n104_annai_kubun;
              this.listPhoneNumber[index].kaisen_meigi = phone.kaisen_meigi;
              this.listPhoneNumber[index].kaisen_meigi_kana = phone.kaisen_meigi_kana;
              this.listPhoneNumber[index].keisai_mei = phone.keisai_mei;
              this.listPhoneNumber[index].keisai_mei_kana = phone.keisai_mei_kana;
              this.listPhoneNumber[index].ntt_np = phone.ntt_np;
              this.listPhoneNumber[index].ntt_np_date = phone.ntt_np_date;
              this.listPhoneNumber[index].juchu_kubun = phone.juchu_kubun;
              this.listPhoneNumber[index].kaiyaku_uketsuke_date = phone.kaiyaku_uketsuke_date;
              this.listPhoneNumber[index].update_flg = FLAG_UPDATED;
              break;
            }else if(index === this.listPhoneNumber.length - 1){
              // 最後まで対象のデータがない場合は追加する
              this.listPhoneNumber.push(phone);
              break;
            }
          }
        }
      })
      phonedata.forEach((phone) => {
        // 紐づき番号の変更があった場合古い紐づき番号をリセットする
        if(phone.himozuki_tel_no !== phone.old_himozuki_tel_no){
          this.listPhoneNumber.forEach((target) => {
            if(target.ip_tel_no === phone.old_himozuki_tel_no){
              target.himozuki_tel_no = ""
              target.himozuki_ashk_tel_no = ""
              target.update_flg = FLAG_UPDATED
            }
          })
        }
        // 紐づき番号が設定されている場合、その紐づき番号と設定している番号で互いに紐づける
        if(phone.himozuki_tel_no){
          this.listPhoneNumber.forEach((target) => {
            if(target.ip_tel_no === phone.himozuki_tel_no){
              target.himozuki_tel_no = phone.ip_tel_no
              target.himozuki_ashk_tel_no = phone.ip_ashk_tel_no
              target.update_flg = FLAG_UPDATED
            }
          })
        }
      })
      this.showDialogAddService = false;
      this.$emit('listOfService-changed');
      // 採番リスト更新
      this.$emit('numbered');
    },
    // 全解約取消
    cancelCancellation(data: CancelAllDataReponse): void {
      console.log('cancelCancellation');
      console.log(data);
      this.showDialogCancelAll = false;
      const requestData:allCancelCancellationRequestdata = this.convertAllCancelCancellationRequestdata(data);
      console.log(requestData)
      Promise.all([
        allCancelCancellationAPI(getScreenId(this.$route.name), requestData),
      ]).then((res) => {
        console.log(res)
        if(this.$emit('auth', 'InfoRegisterComplete')){
          this.$emit('reload')
          // 全解約取消メッセージ
          this.$store.dispatch('dialog/open', {
            type: CONFIRMATION,
            subType: AFTER_CANCEL,
            messages: [
              this.$t('customerDetail.messages.allCancelSuccess'),this.t(`messages.cancel`)
            ],
            callback: () =>
              this.$emit('move-completion'),
          });
        }
      });
    },
    // 全解約登録
    cancelData(data: CancelAllDataReponse): void {
      console.log('cancelData');
      console.log(data);
      this.showDialogCancelAll = false;
      const requestData:allCancelRequestdata = this.convertAllCancelRequestdata(data);
      console.log(requestData)
      Promise.all([
        allCancelAPI(getScreenId(this.$route.name), requestData),
      ]).then((res) => {
        console.log(res)
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
      });
    },
    convertAllCancelRequestdata(data :CancelAllDataReponse): allCancelRequestdata {
      const convData: allCancelRequestdata = {
        gno: data.gno,
        juch_no: data.juch_no,
        juch_mesai_no: data.juch_mesai_no,
        cancel_kaiyaku_kubun: data.cancellationSectionCode,
        service_teishi_kibo_date: data.dateOfService,
        tekkyo_koji_limit_date: data.removalDeadline,
        tekkyo_koji_contact: data.contactOfConstructionRemoval,
        ip_kaisen_jigyosha_tokki_jiko: data.ipLine,
        c_taish_kbn: data.cancellationSection,
        c_uktk_date: data.contractCancellationAcceptanceDate,
        c_sinsei: data.cancellationRequester,
        c_sinsei_memo: data.cancellationRequesterMemo,
        c_riyu_cd: data.cancellationReasonCode,
        kaiyaku_date: data.billingEndDate,
        kaiyaku_uketsuke_date: data.contractCancellationAcceptanceDate,
        dotsu_yotei_date: data.constructionScheduleDate,
        sakjo_flg: data.sakjo_flg,
      };
      return convData;
    },
    convertAllCancelCancellationRequestdata(data :CancelAllDataReponse): allCancelCancellationRequestdata {
      const convData: allCancelCancellationRequestdata = {
        gno: data.gno,
        juch_no: data.juch_no,
        juch_mesai_no: data.juch_mesai_no,
      };
      return convData;
    },
    // 全解約時または取消時、顧客の解約状態をその状態に変更
    // 引数は取消時にtrue,解約時にfalseが入る
    changeCancelStatus(cancelCancellation: boolean){
      this.$emit("change-cancel-status",cancelCancellation);
    },
    // サービス申込・変更ダイアログへの表示データ設定
    setServiceData(item: ServiceApplicationResponse | null, maxSerialNo: string | null): ServiceDataForm{
      let data: ServiceDataForm = {
        id: item ? item.id : new Date().getTime() + '',
        gno: this.customerData.gno,
        // 10000件目は'0000'で登録し、10001件目から一意制約違反となる
        serial_no: item ? item.no : ('0000' + (Number(maxSerialNo) + 1).toString()).slice(-4),
        uf_cd: this.customerData.uf_cd,
        sechimoto_kanji_kaisyamei: this.customerData.seti_kanji_shamei,
        sechimoto_kana_kaisyamei: this.customerData.seti_kn,
        fuka_service_cd: item ? item.serviceApplicationCode : '',
        fuka_service_mei: item ? item.serviceApplicationName : '',
        protocol_kubun: item ? item.protcol_kubun : '',
        moshikomi_date: item ? item.applicationDate : dateFormat(new Date(),"yyyy/mm/dd"),
        koji_kanryo_date: item ? item.completionDate : '',
        kaiyaku_date: item ? item.contractCancellationDate : '',
        ch_num: item ?  item.numberCH : 0,
        jigyosha_moshiokuri_jiko: item ? item.jigyosha_moshiokuri_jiko : '',
        editStatus: '',
      }
      return data;
    }
  },
});
</script>
