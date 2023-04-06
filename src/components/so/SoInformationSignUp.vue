<!--SO情報新規登録画面-->
<template>
  <v-card>
    <v-container>
      <v-form ref="form" v-model="valid" @submit.prevent="submit">
        <h3 class="mb-5">
          {{ showScreenId }}: {{ $t('customerDetail.soInfoRegister.titleNew') }}
        </h3>
        <so-information-details-form
          :so-information="internalSoInformation"
          :mode="dmode"
          :new-contract-cancel-disabled="newContractCancelDisabled"
          @field-change="handleFieldChange"
          @change-new-so-flg="changeExistNewSoControlFlg"
        />
        <h3 class="mb-5">
          {{ $t('customerDetail.soInfoRegister.soDataCreation') }}
        </h3>
        <v-checkbox
          v-model="createSoData"
          :label="$t('customerDetail.soInfoRegister.createSoData')"
          class="ml-5"
          :disabled="internalSoInformation.constructionScheduleDateUndecided"
        />
        <div class="px-10">
          <so-data-create
            v-if="createSoData"
            ref="soDataCreateRefs"
            :gno-code="internalSoInformation.gno"
            :elevation="0"
            :stand-alone="false"
            :show-main-title="false"
            :mode="dmode"
          />
        </div>
        <v-card-actions class="justify-space-between">
          <v-btn @click="close">
            {{ t('closeBtn') }}
          </v-btn>
          <v-btn color="primary" @click="confirm">
            {{ $t('customerDetail.soInfoRegister.confirm') }}
          </v-btn>
        </v-card-actions>
      </v-form>
    </v-container>
  </v-card>
</template>

<script lang="ts">
import { TranslateResult } from 'vue-i18n';
import { SoInformation, SO_NEW, SO_STATUS_MISYORI } from '@/models/so';
import Vue from 'vue';
import { PropType } from 'vue/types/options';
import SoDataCreate from '../customer/SoDataCreate.vue';
import SoInformationDetailsForm from './SoInformationDetailsForm.vue';
import { CONFIRMATION, MESSAGE, REGISTER_UPDATE_EXECUTE, WHILE_INPUT, SO_KISHU_CD_NOTHING, MESSAGE_ERROR } from '@/constants/dialog';
import { ERROR_STATUS_503 } from '@/constant/common-constant';
import
{
  SoInformationDetailCreateRequestData,
  createSoIformationDetails,
} from '@/api/so-info-inquiry-api'
import {
  SoCreateOptions,
  SoCreateTarget,
  SelectTable,
  NewSoInfoRegiRequest,
  SoDataCreateForm
} from '@/models/so-batch';
import {
  SoMakeTarget,
  SoOptionAdd,
  SoSerialAndAskTelNo,
  TRUE_NUMBER,
  FALSE_NUMBER,
} from '@/models/so/index';
import {
  soInfoRegistration
} from '@/api/so-batch-api';
import { AxiosError } from 'axios';
import { KubunListAPI, ResponseData_KUBUN_KANRI } from '@/api/kubun-kanri';
import { isObjKey } from '@/utils/helper';
import { getScreenId } from '@/utils/screenIds';
import { DialogOptions } from '@/models/dialog';
import { SHINKI_KAIYAKU_KUBUN_NEW } from '@/models/so-inquiry';

export default Vue.extend({
  components: { SoInformationDetailsForm, SoDataCreate },
  props: {
    soInformation: {
      type: Object as PropType<SoInformation>,
      required: true,
    },
    screenId: {
      type: String,
      default: ''
    }
  },
  data() {
    return {
      valid: false,
      isDirty: false,
      createSoData: false,
      initialScheduleDate: this.soInformation.constructionScheduleDate,
      internalSoInformation: this.soInformation,
      status_kubun: [] as ResponseData_KUBUN_KANRI[],
      dmode: SO_NEW,
      existNewSoControlFlg: false
    };
  },
  computed: {
    showScreenId(): string {
      console.log(this.$route.name + "!")
      return getScreenId(this.$route.name);
    },
    newContractCancelDisabled(): boolean {
      return !!this.internalSoInformation.constructionScheduleDateUndecided;
    },
    gno(): string {
      const { gno } = this.$route.query;
      if (!gno) return '';
      if (Array.isArray(gno)) return gno[0] || '';
      return gno;
    },
    setisaki_name(): string {
      const { setisaki_name } = this.$route.query;
      if (!setisaki_name) return '';
      if (Array.isArray(setisaki_name)) return setisaki_name[0] || '';
      return setisaki_name;
    },
  },
  watch: {
    internalSoInformation: {
      deep: true,
      handler() {
        this.isDirty = true;
      },
    },
    createSoData: {
      deep: true,
      handler() {
        this.isDirty = true;
      },
    },
    'internalSoInformation.constructionScheduleDateUndecided': {
      handler() {
        if (this.internalSoInformation.constructionScheduleDateUndecided) {
          this.createSoData = false;
        }
      },
    },
  },
  created() {
    this.soInformation.subNumber = "";
    this.soInformation.gno = this.gno;
    this.soInformation.companyName = this.setisaki_name;
    Promise.all([
      KubunListAPI(getScreenId(this.$route.name), 'STATUS'),
    ]).then((res) =>{
      this.status_kubun = res[0].data;
    });
  },
  methods: {
    t(field: string): TranslateResult {
      return this.$t(`soInquiry.soInformationDetails.${field}`);
    },
    SoDataCreatet(field: string): TranslateResult {
      return this.$t(`customerDetail.soDataCreate.${field}`);
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
      const soCreateData = (this.$refs.soDataCreateRefs as InstanceType<typeof SoDataCreate>)
      const valid = (
        this.$refs.form as Vue & { validate: () => boolean }
      ).validate();
      if (valid) {
        // 新規解約区分が新規 && 新規でSO未作成のSOコントロールが登録されている場合エラー
        if (this.internalSoInformation.newContractCancel == SHINKI_KAIYAKU_KUBUN_NEW && this.existNewSoControlFlg == true) {
          this.$store.dispatch('dialog/open', {
            type: MESSAGE,
            subType: MESSAGE_ERROR,
            messages: [this.t('messages.error.newSoControlError')]
          })
          return
        }
        if (this.createSoData) {
          const kishu_cd = soCreateData.kishu_cd
          if("" === kishu_cd || null === kishu_cd)
          {
            this.$store.dispatch('dialog/open', {
              type: MESSAGE,
              subType: SO_KISHU_CD_NOTHING,
            } as DialogOptions)
            return
          }
          // SO作成オプション情報
          const requestOptions: SoCreateOptions = {
            new: soCreateData.form.newContract ? TRUE_NUMBER: FALSE_NUMBER,
            option_add: soCreateData.form.optionAdd ? TRUE_NUMBER: FALSE_NUMBER,
            ch_change: soCreateData.form.totalNumberChannel ? TRUE_NUMBER: FALSE_NUMBER,
            telno_add: soCreateData.form.addPhoneNumber ? TRUE_NUMBER: FALSE_NUMBER,
            stop: soCreateData.form.callSelected[0] ? TRUE_NUMBER: FALSE_NUMBER,
            resume: soCreateData.form.callSelected[1] ? TRUE_NUMBER: FALSE_NUMBER,
            password_reset: soCreateData.form.callSelected[2] ? TRUE_NUMBER: FALSE_NUMBER,
            numbering_notice: soCreateData.form.callSelected[3] ? TRUE_NUMBER: FALSE_NUMBER,
            registration_change: soCreateData.form.callSelected[4] ? TRUE_NUMBER: FALSE_NUMBER,
            single_cancel: soCreateData.form.lineCancel ? TRUE_NUMBER: FALSE_NUMBER,
            deai_gno: soCreateData.form.cancelSelected[1] && soCreateData.form.unitRadio == this.SoDataCreatet('gnoUnit') ? TRUE_NUMBER:FALSE_NUMBER,
            deai_kaisen: soCreateData.form.cancelSelected[1] && soCreateData.form.unitRadio ==this.SoDataCreatet('lineUnit') ? TRUE_NUMBER:FALSE_NUMBER,
            all_cancel: soCreateData.form.cancelSelected[0] ? TRUE_NUMBER: FALSE_NUMBER,
            detour_telno_new: soCreateData.form.detourNumber && soCreateData.form.detourNumberRadio == this.SoDataCreatet('signUp') ? TRUE_NUMBER:FALSE_NUMBER,
            detour_telno_update: soCreateData.form.detourNumber && soCreateData.form.detourNumberRadio == this.SoDataCreatet('numberUpdate') ? TRUE_NUMBER:FALSE_NUMBER,
            detour_telno_delete: soCreateData.form.detourNumber && soCreateData.form.detourNumberRadio == this.SoDataCreatet('delNumber') ? TRUE_NUMBER:FALSE_NUMBER,
            nump_on: soCreateData.form.startNumberPort && soCreateData.form.startNumPortabilityRadio == this.SoDataCreatet('on') ? TRUE_NUMBER:FALSE_NUMBER,
            nump_off: soCreateData.form.startNumberPort && soCreateData.form.startNumPortabilityRadio == this.SoDataCreatet('off') ? TRUE_NUMBER:FALSE_NUMBER,
          }
          const listPhoneChecked: SelectTable = {
            newContract: soCreateData.form.newContract,
            listServiceSelect: soCreateData.form.optionAdd ? soCreateData.listServiceSelect:[],
            sum_of_ch: soCreateData.form.totalNumberChannel,
            listPhoneSelectAddPhoneNumber: soCreateData.form.addPhoneNumber ? soCreateData.listPhoneSelectAddPhoneNumber:[],
            stop_call: soCreateData.form.callSelected[0],
            unstop: soCreateData.form.callSelected[1],
            resetPassword: soCreateData.form.callSelected[2],
            notification: soCreateData.form.callSelected[3],
            changeRegiNumber: soCreateData.form.callSelected[4],
            ukaiNumberCheck: soCreateData.form.detourNumber,
            ukaiNumber: soCreateData.form.detourNumberReg,
            ukaiRegType: soCreateData.form.detourNumberRadio,
            allCancel: soCreateData.form.cancelSelected[0],
            encounter: soCreateData.form.cancelSelected[1],
            motoGnos: soCreateData.form.oldGno[0] + " " + soCreateData.form.oldGno[1] + " " + soCreateData.form.oldGno[2],
            unit: soCreateData.form.unitRadio,
            np: soCreateData.form.startNumberPort ? soCreateData.form.startNumPortabilityRadio:'',
            listPhoneSelectEncounter: soCreateData.form.cancelSelected[1] ? soCreateData.listPhoneSelectEncounter:[],
            listPhoneSelectLineCancel: soCreateData.form.lineCancel ? soCreateData.listPhoneSelectLineCancel: [],
            listPhoneSelectStartNumberPort: soCreateData.form.startNumberPort ? soCreateData.listPhoneSelectStartNumberPort:[],
          };
          this.$store.dispatch('dialog/open', {
            type: CONFIRMATION,
            subType: REGISTER_UPDATE_EXECUTE,
            messages: [this.t('messages.register')],
            // データ受け取り
            callback: () => this.handleConfirmDialogSOData(listPhoneChecked, requestOptions, soCreateData.form),
          });
        } else {
          console.log("SOコントロール作成")
          this.$store.dispatch('dialog/open', {
            type: CONFIRMATION,
            subType: REGISTER_UPDATE_EXECUTE,
            messages: [this.t('messages.register')],
            // データ受け取り
            callback: () => this.registSoInformation(this.internalSoInformation),
          });
        }
      } else {
        this.$store.dispatch('dialog/open', {
          type: MESSAGE,
          subType: MESSAGE_ERROR,
          messages: [this.t('messages.error.validationError')]
        });
      }
    },
    //
    registSoInformation(data: SoInformation): void {
      console.log("SO情報登録画面-登録処理");
      console.log(data);

      const temp:SoInformationDetailCreateRequestData = 
      {
        koji_yotei_date:data.constructionScheduleDate,
        gno:data.gno,
        seq_no:data.subNumber,
        kaisha_mei:data.companyName,
        chiku:data.district,
        shinki_kaiyaku_kubun:data.newContractCancel,
        shubetsu:data.type,
        status:SO_STATUS_MISYORI,
        so_sakusei_kubun:data.createSoFile,
        kinkyu_flg:data.emergencySendingFlag,
        toroku_kubun:data.soRegistration,
        oa_juchu_no:data.oaOrderNumber,
        oa_juchu_edaban:data.oaOrderSubNumber,
        toroku_naiyo_id:data.registerContents,
        biko:data.comment,
        juch_no:data.juch_no,
        juch_mesai_no:data.juch_mesai_no,
      };
      createSoIformationDetails(
        getScreenId(this.$route.name),
        temp
      )
      .then((res) => {
        this.soInformation.subNumber = res.data.seq_no
        this.$emit('close');
      })
      .finally(()=> {
        console.log("更新完了");
      });
    },
    handleFieldChange<K extends keyof SoInformation>(
      field: K,
      value: SoInformation[K]
    ): void {
      console.log(field)
      if (isObjKey(field, this.internalSoInformation)) {
        this.internalSoInformation[field] = value;
      }
    },

    // SOデータ作成関連API呼び出し処理
    handleConfirmDialogSOData(listPhone: SelectTable, requestOptions: SoCreateOptions, createForm: SoDataCreateForm): void {
      console.log('CreateDialogSODataConfirm');
      let targetData: SoCreateTarget = this.makeSoTargetDetails(listPhone, createForm)
      targetData.koji_yotei_date = this.internalSoInformation.constructionScheduleDate;
      targetData.gno = this.internalSoInformation.gno;
      targetData.seq_no = null;
      // SO情報登録リクエスト用データ作成
      const NewSoRegiRequest: NewSoInfoRegiRequest[] = [];
      const convData: NewSoInfoRegiRequest = {
        gno: this.internalSoInformation.gno,
        seq_no: null,
        kaisha_mei: this.internalSoInformation.companyName,
        chiku: this.internalSoInformation.district,
        shinki_kaiyaku_kubun: this.internalSoInformation.newContractCancel,
        shubetsu: this.internalSoInformation.type,
        koji_yotei_date: this.internalSoInformation.constructionScheduleDate,
        koji_yotei_date_mitei_kubun: false,
        so_sakusei_kubun: this.internalSoInformation.createSoFile,
        kinkyu_flg: this.internalSoInformation.emergencySendingFlag,
        toroku_kubun: this.internalSoInformation.soRegistration,
        oa_juchu_no: this.internalSoInformation.oaOrderNumber,
        oa_juchu_edaban: this.internalSoInformation.oaOrderSubNumber.toString(),
        toroku_naiyo_id: this.internalSoInformation.registerContents,
        biko: this.internalSoInformation.comment,
        juch_no: this.internalSoInformation.juch_no,
        juch_mesai_no: this.internalSoInformation.juch_mesai_no,
        options: requestOptions,
        target: targetData
      };
      NewSoRegiRequest.push(convData);
      const screen_id = getScreenId(this.$route.name)
      // SO情報登録API呼び出し
      soInfoRegistration(screen_id, NewSoRegiRequest)
        .then((res) => {
          console.log(res)
          alert(this.$t(`customerDetail.soDataCreate.messageSuccess`).toString())
          // ダイアログを閉じる
          this.$emit('close');
        })
        .catch((err: AxiosError) => {
        console.error(err.response)
        if (err.response?.status === ERROR_STATUS_503) {
          console.log(err.response)
          alert(this.$t(`customerDetail.soDataCreate.messageError`).toString() + "\n" 
            + "エラー内容: " + err.response.data.failures.ErrorDetail.message)
        } else {
          this.$store.dispatch('systemError/showSystemError', err.response)
        }
      });
    },
    makeSoTargetDetails(listPhone: SelectTable | null, createForm :SoDataCreateForm | null): SoCreateTarget {
      let dataList: SoMakeTarget = {
        koji_yotei_date: "",
        gno: "",
        seq_no: null,
        option_add: [],
        telno_add: [],
        deai_gno: [],
        deai_kaisen: [],
        single_cancel: [],
        detour_telno: "",
        nump_on: [],
        nump_off: []
      };
      //console.log(listPhone)
      //console.log(createForm)
      if (listPhone != null && createForm != null) {
        // 選択されたオプションサービス
        listPhone.listServiceSelect.forEach(function(service, index) {
          let telNo: SoOptionAdd = {
            sip_option_meisai_no: service.sip_option_meisai_no,
          }
          dataList.option_add[index] = telNo;
        });

        // 選択された追加電番
        listPhone.listPhoneSelectAddPhoneNumber.forEach(function(phone, index) {
          let telNo: SoSerialAndAskTelNo = {
            serial_no: phone.serial_no,
            ip_ashk_tel_no: phone.ip_ashk_tel_no,
          }
          dataList.telno_add[index] = telNo;
        });

        // 出合い旧GNO
        createForm.oldGno.forEach(function(gNo, index) {
          if (gNo != "") {
            dataList.deai_gno[index] = gNo
          }
        });

        // 出合い回線
        listPhone.listPhoneSelectEncounter.forEach(function(phone, index) {
          let telNo: SoSerialAndAskTelNo = {
            serial_no: phone.serial_no,
            ip_ashk_tel_no: phone.ip_ashk_tel_no,
          }
          dataList.deai_kaisen[index] = telNo;
        });

        // 回線解約番号
        listPhone.listPhoneSelectLineCancel.forEach(function(phone, index) {
          let telNo: SoSerialAndAskTelNo = {
            serial_no: phone.serial_no,
            ip_ashk_tel_no: phone.ip_ashk_tel_no,
          }
          dataList.single_cancel[index] = telNo;
        });

        // 登録迂回番号
        if (createForm.detourNumberReg != "") {
          dataList.detour_telno = createForm.detourNumberReg
        }

        // 番ポ起動電番
        console.log(createForm.startNumPortabilityRadio)
        listPhone.listPhoneSelectStartNumberPort.forEach(function(phone, index) {
          let telNo: SoSerialAndAskTelNo = {
            serial_no: phone.serial_no,
            ip_ashk_tel_no: phone.ip_ashk_tel_no,
          }
          if (createForm.startNumPortabilityRadio == "オン") {
            dataList.nump_on[index] = telNo;
          } else {
            dataList.nump_off[index] = telNo;
          }
        });
      }
      return dataList;
    },
    changeExistNewSoControlFlg(existNewSoControlFlg: boolean): void {
      this.existNewSoControlFlg = existNewSoControlFlg
    }
  },
});
</script>
