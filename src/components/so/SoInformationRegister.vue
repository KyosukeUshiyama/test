<!--SO情報登録画面(顧客検索から)-->
<template>
  <v-container>
    <v-form ref="form" v-model="valid" @submit.prevent="submit">
      <h3 class="mb-5">
        {{ screenId }}: {{ $t('customerDetail.soInfoRegister.title') }}
      </h3>
      <so-information-details-form
        :so-information="internalSoInformation"
        mode="tab"
        :new-contract-cancel-disabled="newContractCancelDisabled"
        @field-change="handleFieldChange"
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
          :so-check-box="soCheckBox"
          :elevation="0"
          :stand-alone="false"
          :show-main-title="false"
          :mode="mode"
        />
      </div>
      <v-dialog v-model="showDialogSODataCreateConfirm" width="520" persistent>
        <so-data-create-confirm
          v-if="showDialogSODataCreateConfirm"
          :gno-code="internalSoInformation.gno"
          :construct-schedule-date="
            internalSoInformation.constructionScheduleDate
          "
          :so-create-data-list="inputDataList"
          :warning-message="warning_message"
          @close="handleCloseDialogSODataConfirm"
          @create="confirm"
        />
      </v-dialog>
      <div class="d-flex justify-end mt-5">
        <v-btn 
          v-if="!createSoData"
          color="primary"
          :disabled="!createPermission"
          @click="confirm"
        >
          {{ $t('customerDetail.soInfoRegister.confirm') }}
        </v-btn>
        <v-btn 
          v-if="createSoData"
          color="primary"
          :disabled="!createPermission"
          @click="soDataConfirm"
        >
          {{ $t('customerDetail.soInfoRegister.confirm') }}
        </v-btn>
      </div>
    </v-form>
  </v-container>
</template>

<script lang="ts">
import { TranslateResult } from 'vue-i18n';
import { SoInformation, SO_STATUS_MISYORI } from '@/models/so';
import Vue from 'vue';
import { PropType } from 'vue/types/options';
import SoDataCreate from '../customer/SoDataCreate.vue';
import SoInformationDetailsForm from './SoInformationDetailsForm.vue';
import { CONFIRMATION, REGISTER_UPDATE_EXECUTE, SO_KISHU_CD_NOTHING } from '@/constants/dialog';
import { ERROR_STATUS_503, JIDO_UKAI_SERVICE_CD } from '@/constant/common-constant';
import
{
  SoInformationDetailCreateRequestData,
  createSoIformationDetails,
} from '@/api/so-info-inquiry-api'
import { getSoCheckBox } from '@/api/so-checkbox-api';
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
  soInfoRegistration,
} from '@/api/so-batch-api';
import { getKanryoJohoList } from '../../api/CompletionInfomation-api';
import { AxiosError } from 'axios';
import { KubunListAPI, ResponseData_KUBUN_KANRI } from '@/api/kubun-kanri';
import { isObjKey } from '@/utils/helper';
import SoDataCreateConfirm from '@/components/customer/SoDataCreateConfirm.vue';
import { SoInfomationConfirmOneData, SO_TAB_AND_NEW} from '@/models/so/index';
import { getScreenId } from '@/utils/screenIds';
import { MESSAGE, MESSAGE_ERROR } from '@/constants/dialog';
import { DialogOptions } from '@/models/dialog';
import { SearchFormData, SHINKI_KAIYAKU_KUBUN_NEW, SO_SAKUSEI_KUBUN_NOT_CREATED } from '@/models/so-inquiry';
import { getSoInfoInquiryList } from '@/api/so-info-inquiry-api';

interface checkObject {
  key: string;
  index: number;
}

export default Vue.extend({
  components: { SoInformationDetailsForm, SoDataCreate, SoDataCreateConfirm },
  props: {
    soInformation: {
      type: Object as PropType<SoInformation>,
      required: true,
    },
    screenId: {
      type: String,
      default: ''
    },
    kojiYoteiDate: {
      type: String,
      default: '',
    },
    chiku:{
      type: String,
      default: null,
    },
    isSoDirty:{
      type: Boolean,
      default: false,
    },
    callbackFunction: {
      type: Function,
      default: undefined,
    },
  },
  data() {
    return {
      valid: false,
      createSoData: false,
      initialScheduleDate: this.soInformation.constructionScheduleDate,
      internalSoInformation: this.soInformation,
      status_kubun: [] as ResponseData_KUBUN_KANRI[],
      inputDataList:[] as SoInfomationConfirmOneData[],
      showDialogSODataCreateConfirm: false,
      createPermission: false,
      soCheckBox: {},
      warning_message: [] as TranslateResult[],
      mode: SO_TAB_AND_NEW, // SO情報登録画面のモード判定用
      existNewSoControlFlg: false,
      kakinEndDate: ''
    };
  },
  computed: {
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
    juchu_no(): string {
      const { juchu_no } = this.$route.query;
      if (!juchu_no) return '';
      if (Array.isArray(juchu_no)) return juchu_no[0] || '';
      return juchu_no;
    },
    juchu_meisai_no(): string {
      const { juchu_meisai_no } = this.$route.query;
      if (!juchu_meisai_no) return '';
      if (Array.isArray(juchu_meisai_no)) return juchu_meisai_no[0] || '';
      return juchu_meisai_no;
    },
  },
  watch: {
    'internalSoInformation.constructionScheduleDateUndecided': {
      handler() {
        if (this.internalSoInformation.constructionScheduleDateUndecided) {
          this.createSoData = false;
        }
      },
    },
    kojiYoteiDate: {
      handler() {
        // 個別解約時の工事予定日を設定する
        if(this.kojiYoteiDate){
          this.soInformation.constructionScheduleDate = this.kojiYoteiDate
        }
      }
    }
  },
  created() {
    this.createAPI()
  },
  methods: {
    t(field: string): TranslateResult {
      return this.$t(`soInquiry.soInformationDetails.${field}`);
    },
    SoDataCreatet(field: string): TranslateResult {
      return this.$t(`customerDetail.soDataCreate.${field}`);
    },
    // チェックボックス判定&メッセージリスト生成
    checkboxMsg(): TranslateResult[] {
      const soData = this.$refs.soDataCreateRefs as InstanceType<typeof SoDataCreate>
      const soDataForm = soData.form
      const msgList = []
      let target :checkObject[] = []
      if (soDataForm.newContract == true) {
        target = [{key: 'newContract', index: -1},{key: 'optionAdd', index: -1},{key: 'addPhoneNumber', index: -1},{key: 'totalNumberChannel', index: -1}]
        if (this.checkCheckBox(target) == true) {
          msgList.push(this.t(`checkboxMsg.newContract`))
        }
      }
      if (soDataForm.optionAdd == true) {
        target = [{key: 'optionAdd', index: -1},{key: 'totalNumberChannel', index: -1},{key: 'addPhoneNumber', index: -1},
                  {key: 'lineCancel', index: -1},{key: 'newContract', index: -1},{key: 'cancelSelected', index: 1}]
        if (this.checkCheckBox(target) == true) {
          msgList.push(this.t(`checkboxMsg.optionAdd`))
        }
      }
      if (soDataForm.addPhoneNumber == true) {
        target = [{key: 'addPhoneNumber', index: -1},{key: 'optionAdd', index: -1},{key: 'totalNumberChannel', index: -1},
                  {key: 'callSelected', index: 4},{key: 'lineCancel', index: -1},{key: 'newContract', index: -1},{key: 'cancelSelected', index: 1}]
        if (this.checkCheckBox(target) == true) {
          msgList.push(this.t(`checkboxMsg.addPhoneNumber`))
        }
      }
      if (soDataForm.callSelected[0] == true) {
        target = [{key: 'callSelected', index: 0}]
        if (this.checkCheckBox(target) == true) {
          msgList.push(this.t(`checkboxMsg.callSelected0`))
        }
      }
      if (soDataForm.callSelected[1] == true) {
        target = [{key: 'callSelected', index: 1}]
        if (this.checkCheckBox(target) == true) {
          msgList.push(this.t(`checkboxMsg.callSelected1`))
        }
      }
      if (soDataForm.callSelected[2] == true) {
        target = [{key: 'callSelected', index: 2}]
        if (this.checkCheckBox(target) == true) {
          msgList.push(this.t(`checkboxMsg.callSelected2`))
        }
      }
      if (soDataForm.callSelected[3] == true) {
        target = [{key: 'callSelected', index: 3}]
        if (this.checkCheckBox(target) == true) {
          msgList.push(this.t(`checkboxMsg.callSelected3`))
        }
      }
      if (soDataForm.callSelected[4] == true) {
        target = [{key: 'callSelected', index: 4},{key: 'optionAdd', index: -1},{key: 'totalNumberChannel', index: -1},
                  {key: 'addPhoneNumber', index: -1},{key: 'lineCancel', index: -1}]
        if (this.checkCheckBox(target) == true) {
          msgList.push(this.t(`checkboxMsg.callSelected4`))
        }
      }
      if (soDataForm.cancelSelected[0] == true) {
        target = [{key: 'cancelSelected', index: 0}]
        if (this.checkCheckBox(target) == true) {
          msgList.push(this.t(`checkboxMsg.cancelSelected0`))
        }
      }
      if (soDataForm.cancelSelected[1] == true) {
        target = [{key: 'cancelSelected', index: 1},{key: 'optionAdd', index: -1},{key: 'totalNumberChannel', index: -1},
                  {key: 'addPhoneNumber', index: -1},]
        if (this.checkCheckBox(target) == true) {
          msgList.push(this.t(`checkboxMsg.cancelSelected1`))
        }
      }
      if (soDataForm.lineCancel == true) {
        target = [{key: 'lineCancel', index: -1},{key: 'optionAdd', index: -1},{key: 'totalNumberChannel', index: -1},
                  {key: 'addPhoneNumber', index: -1},{key: 'callSelected', index: 4},{key: 'cancelSelected', index: 1}]
        if (this.checkCheckBox(target) == true) {
          msgList.push(this.t(`checkboxMsg.lineCancel`))
        }
      }
      if (soDataForm.detourNumber == true) {
        target = [{key: 'detourNumber', index: -1}]
        if (this.checkCheckBox(target) == true) {
          msgList.push(this.t(`checkboxMsg.detourNumber`))
        }
      }
      if (soDataForm.startNumberPort == true) {
        target = [{key: 'startNumberPort', index: -1}]
        if (this.checkCheckBox(target) == true) {
          msgList.push(this.t(`checkboxMsg.startNumberPort`))
        }
      }
      return msgList
    },
    // チェックボックス判定
    checkCheckBox(checkItem: checkObject[]): boolean {
      const soData = this.$refs.soDataCreateRefs as InstanceType<typeof SoDataCreate>
      const soDataForm = soData.form
      let result = false
      let oneDList: string[] = []
      let twoDList: checkObject[] = []
      const checkboxList = [
        'newContract', // 新規
        'optionAdd', // オプション追加
        'totalNumberChannel', // 総CH数変更
        'addPhoneNumber', // 追加電番
        'callSelected', // 0:通話停止 1:停止解除 2:パスワード初期化 3:発番通知 4:レジ番変更(配列)
        'cancelSelected', // 0:全解約 1:出会い(配列)
        'lineCancel', // 回線解約
        'detourNumber', // 迂回番号登録
        'startNumberPort', // 番ポ起動
      ] as (keyof SoDataCreateForm)[]
      checkItem.forEach((element) => {
        // 引数に含まれる要素をチェックリストから除外
        oneDList.push(element.key)
        if (element.index != -1) {
          // 二次元配列の要素を特殊処理用のチェックリストに追加
          twoDList.push(element)
        }
      })
      if (oneDList.length > 0) {
        // チェック対象のリストを生成
        const newCheckboxList = checkboxList.filter(item => !oneDList.includes(item));
        newCheckboxList.forEach((element) => {
          if (element == 'callSelected' || element == 'cancelSelected') {
            // 二次元配列の要素はもう一度forEachで回す
            soDataForm[element].forEach((e) => {
              if (e == true) result = true
            })
          } else {
            // チェックされている要素が一つでもあればtrueを返す
            if (soDataForm[element] == true) result = true
          }
        })
      } 
      // 二次元配列用処理
      if (twoDList.length > 0) {
        twoDList.forEach((element) => {
          // チェック対象のリストを生成
          const newCheckboxList = checkboxList.filter(item => element.key.includes(item));
          newCheckboxList.forEach((elem) => {
            // 二次元配列を担保するためにif文追加
            if (elem == 'callSelected' || elem == 'cancelSelected') {
              soDataForm[elem].forEach((e, index) => {
                // 指定したインデックスはチェック対象外
                if (index == element.index) return
                if (e == true) result = true
              })
            }
          })
        })
      }
      return result
    },
    soDataConfirm(): void {
      const valid_so = (
        this.$refs.form as Vue & { validate: () => boolean }
      ).validate();
      const soData = (
        this.$refs.soDataCreateRefs as InstanceType<typeof SoDataCreate>
      )
      const kishu_cd = soData.kishu_cd
      if("" === kishu_cd || null === kishu_cd)
      {
        this.$store.dispatch('dialog/open', {
          type: MESSAGE,
          subType: SO_KISHU_CD_NOTHING,
        } as DialogOptions)
        return
      }
      // 新規解約区分が新規 && 新規でSO未作成のSOコントロールが登録されている場合エラー
      if (this.internalSoInformation.newContractCancel == '1' && this.existNewSoControlFlg == true) {
        this.$store.dispatch('dialog/open', {
          type: MESSAGE,
          subType: MESSAGE_ERROR,
          messages: [this.t('messages.error.newSoControlError')]
        })
        return
      }
      const valid_soData = (
        soData.$refs.form as Vue & { validate: () => boolean }
      ).validate();
      if(valid_so && valid_soData){
        const msgList = this.checkboxMsg()
        // メッセージがある場合ダイアログ表示
        if (msgList.length > 0) {
          this.$store.dispatch('dialog/open', {
            type: MESSAGE,
            subType: MESSAGE_ERROR,
            messages: msgList,
          } as DialogOptions)
          return
        }
        // 編集中の項目がないかチェック
        if (this.isSoDirty == true) {
          this.$store.dispatch('dialog/open', {
            type: CONFIRMATION,
            subType: REGISTER_UPDATE_EXECUTE,
            messages: [this.t(`resisterDirty`)],
            autoClose: true,
            callback: () => {
              this.soResist()
            },
          } as DialogOptions)
        } else {
          this.soResist()
        }
      } else {
        this.$store.dispatch('dialog/open', {
          type: MESSAGE,
          subType: MESSAGE_ERROR,
          messages: [this.t('messages.error.validationError')]
        });
      }
    },
    soResist(): void{
      const soCreateData = (this.$refs.soDataCreateRefs as InstanceType<typeof SoDataCreate>);
      // 新規解約区分が解約の場合、工事予定日と課金終了日を比較
      // フォーマットを合わせてDate型へ変換
      let kakin_end_date = new Date(this.kakinEndDate.replace(/-/g,"/"))
      let koji_yotei_date = new Date(this.internalSoInformation.constructionScheduleDate.replace(/-/g,"/"))
      if (this.internalSoInformation.newContractCancel == '9' && koji_yotei_date < kakin_end_date) {
        if (!window.confirm('工事予定日が課金終了日前になりますが問題ないですか？課金終了日：（' + this.kakinEndDate + '）')) {
          // いいえの場合終了
          return
        }
      }

      // ワーニングメッセージ生成
      let error_message: TranslateResult[] = []
      let warning_message:TranslateResult[] = []
      // イレギュラーフラグ2をチェック
      if (soCreateData.irregular_flg2 == '1') {
        warning_message.push(this.$t('customerDetail.soDataCreateConfirm.irregularFlagWarning'))
      }
      // 全解約がチェックされている場合、VOIP設置日をチェック
      if (soCreateData.voip_setchi_date == null && soCreateData.form.cancelSelected[0]) {
        warning_message.push(this.$t('customerDetail.soDataCreateConfirm.voipgWarning'))
      }
      // 全解約がチェックされている場合、解約済み電番をチェック
      if (soCreateData.kaiyakuList.length > 0 && soCreateData.form.cancelSelected[0]) {
        warning_message.push(this.$t('customerDetail.soDataCreateConfirm.kaiyakuFlagWarning'))
        soCreateData.kaiyakuList.forEach((element) => {
          // フォーマットを合わせてDate型へ変換
          let kaiyaku_uketsuke_date = new Date(element.so_kaiyaku_data_sakusei_date.replace(/-/g,"/"))
          let koji_yotei_date = new Date(this.internalSoInformation.constructionScheduleDate.replace(/-/g,"/"))
          // 解約受付日が工事予定日より前の場合はエラー
          if (kaiyaku_uketsuke_date < koji_yotei_date) {
            warning_message.push('解約SO作成済みの電番:' + element.ip_tel_no + ' 解約SO作成日:' + element.so_kaiyaku_data_sakusei_date)
          } else {
            error_message.push('解約SO作成済みの電番:' + element.ip_tel_no + ' 解約SO作成日:' + element.so_kaiyaku_data_sakusei_date)
          }
        })
      }
      // エラーメッセージがある場合はダイアログ出して終了
      if (error_message.length > 0) {
        error_message.push(this.$t('customerDetail.soDataCreateConfirm.kaiyakuFlagWarning2'))
        this.$store.dispatch('dialog/open', {
          type: MESSAGE,
          messages: error_message,
          } as DialogOptions);
        return
      }
      this.warning_message = warning_message
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
      this.inputDataList = this.coverDataPhoneList(listPhoneChecked);
      this.showDialogSODataCreateConfirm = true;
    },
    coverDataPhoneList(listPhone: SelectTable): SoInfomationConfirmOneData[] {
      let dataList: SoInfomationConfirmOneData[] = [];

      let listService: string[] = [];
      let listPhoneAddPhoneNumber: string[] = [];
      let listPhoneEncounter: string[] = [];
      let listPhoneLineCancel: string[] = [];
      let listPhoneStartNumberPort: string[] = [];
      let listPhoneEncounterAll: string[] = [];

      listPhone.listServiceSelect.forEach((service) => {
        let jido_ukai_tel_no = ''
        if (service.sip_option_service_cd == JIDO_UKAI_SERVICE_CD) {
          jido_ukai_tel_no = service.jido_ukai_tel_no
        }
        listService.push(service.serviceName + ' ' + jido_ukai_tel_no);
      });
      listPhone.listPhoneSelectAddPhoneNumber.forEach((phone) => {
        listPhoneAddPhoneNumber.push(
          this.$t(`customerDetail.soDataCreate.addBillPhoneNumber`) +
            ' ' +
            phone.ip_tel_no
        );
      });
      listPhone.listPhoneSelectEncounter.forEach((phone) => {
        listPhoneEncounter.push(
          listPhone.unit + " " +
          this.$t(`customerDetail.soDataCreate.addBillPhoneNumber`) +
            ' ' +
            phone.ip_tel_no
        );
      });
      listPhone.listPhoneSelectLineCancel.forEach((phone) => {
        listPhoneLineCancel.push(
          this.$t(`customerDetail.soDataCreate.addBillPhoneNumber`) +
            ' ' +
            phone.ip_tel_no
        );
      });
      listPhone.listPhoneSelectStartNumberPort.forEach((phone) => {
        listPhoneStartNumberPort.push(
          this.$t(`customerDetail.soDataCreate.addBillPhoneNumber`) +
            ' ' +
            phone.ip_tel_no
        );
      });
      listPhone.newContract &&
        dataList.push({
          title: this.$t(`customerDetail.soDataCreate.checkBoxNewContract`).toString(),
          values: [],
        });
      listService.length > 0 &&
        dataList.push({
          title: this.$t(
            `customerDetail.soDataCreate.checkBoxAddOption`
          ).toString(),
          values: listService,
        });
      listPhone.sum_of_ch &&
        dataList.push({
          title: this.$t(`customerDetail.soDataCreate.checkBoxChangeNumbersCH`).toString(),
          values: [],
        });
      listPhoneAddPhoneNumber.length > 0 &&
        dataList.push({
          title: this.$t(
            `customerDetail.soDataCreate.checkBoxAddPhone`
          ).toString(),
          values: listPhoneAddPhoneNumber,
        });
      listPhone.stop_call &&
        dataList.push({
          title: this.$t(`customerDetail.soDataCreate.checkBoxPauseCall`).toString(),
          values: [],
        });
      listPhone.unstop &&
        dataList.push({
          title: this.$t(`customerDetail.soDataCreate.checkBoxUnPauseCall`).toString(),
          values: [],
        });
      listPhone.resetPassword &&
        dataList.push({
          title: this.$t(`customerDetail.soDataCreate.checkBoxInitPass`).toString(),
          values: [],
        });
      listPhone.notification &&
        dataList.push({
          title: this.$t(`customerDetail.soDataCreate.checkBoxCallNumNoti`).toString(),
          values: [],
        });
      listPhone.changeRegiNumber &&
        dataList.push({
          title: this.$t(`customerDetail.soDataCreate.checkBoxChangeRegNum`).toString(),
          values: [],
        });
      listPhone.allCancel &&
        dataList.push({
          title: this.$t(`customerDetail.soDataCreate.checkBoxCancelAllContract`).toString(),
          values: [],
        });
      if(listPhone.encounter){
        if(listPhoneEncounter.length > 0){
          listPhoneEncounterAll = listPhoneEncounter;
          listPhoneEncounterAll.unshift(
            this.$t(`customerDetail.soDataCreate.oldGNO`).toString() + " " + listPhone.motoGnos
          )
        }else {
          listPhoneEncounterAll.push(
            this.$t(`customerDetail.soDataCreate.oldGNO`).toString() + " " + listPhone.motoGnos
          );
        }
        dataList.push({
          title: this.$t(`customerDetail.soDataCreate.checkBoxMeeting`).toString(),
          values: listPhoneEncounterAll,
        });
      }
      listPhoneLineCancel.length > 0 &&
        dataList.push({
          title: this.$t(
            `customerDetail.soDataCreate.checkBoxLineContractCancel`
          ).toString(),
          values: listPhoneLineCancel,
        });
      listPhone.ukaiNumberCheck &&
        dataList.push({
          title: this.$t(`customerDetail.soDataCreate.checkBoxDetourNumber`).toString(),
          values: [listPhone.ukaiRegType + " " + listPhone.ukaiNumber],
        })
      listPhoneStartNumberPort.length > 0 &&
        dataList.push({
          title: this.$t(
            `customerDetail.soDataCreate.checkBoxStartNumPortability`
          ).toString() + " " + listPhone.np,
          values: listPhoneStartNumberPort,
        });
      return dataList;
    },
    handleCloseDialogSODataConfirm(): void {
      console.log('handleCloseDialogSODataConfirm');
      const close = (): void => {
        this.showDialogSODataCreateConfirm = false;
      };
      close();
    },
    confirm(): void {
      this.showDialogSODataCreateConfirm = false;
      // 新規解約区分が解約の場合、工事予定日と課金終了日を比較
      // フォーマットを合わせてDate型へ変換
      let kakin_end_date = new Date(this.kakinEndDate.replace(/-/g,"/"))
      let koji_yotei_date = new Date(this.internalSoInformation.constructionScheduleDate.replace(/-/g,"/"))
      if (this.internalSoInformation.newContractCancel == '9' && koji_yotei_date < kakin_end_date) {
        if (!window.confirm('工事予定日が課金終了日前になりますが問題ないですか？課金終了日：（' + this.kakinEndDate + '）')) {
          // いいえの場合終了
          return
        }
      }
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
          const soCreateData = (this.$refs.soDataCreateRefs as InstanceType<typeof SoDataCreate>)
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
          this.handleConfirmDialogSOData(listPhoneChecked, requestOptions, soCreateData.form);
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
        this.$emit('dirty', false)
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
      this.$$openLoadingScreen() // Loading処理の呼び出し
      // 確認ダイアログ後、this.internalSoInformationをAPIに渡して登録する
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
        console.log("更新完了");
        alert(this.$t(`customerDetail.soDataCreate.messageSuccess`).toString())
        const soData = this.$refs.soDataCreateRefs as InstanceType<typeof SoDataCreate>
        soData.refresh()
      })
      .finally(()=> {
        if (this.callbackFunction) {
          this.callbackFunction()
        }
        this.$$closeLoadingScreen() // Loading処理の解除
      });
    },
    handleFieldChange<K extends keyof SoInformation>(
      field: K,
      value: SoInformation[K]
    ): void {
      console.log(field)
      if (isObjKey(field, this.internalSoInformation)) {
        this.internalSoInformation[field] = value;
        this.$emit('dirty', true)
      }
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
    // SOデータ作成関連API呼び出し処理
    handleConfirmDialogSOData(listPhone: SelectTable, requestOptions: SoCreateOptions, createForm: SoDataCreateForm): void {
      this.$$openLoadingScreen() // Loading処理の呼び出し
      console.log('CreateDialogSODataConfirm');
      let targetData: SoCreateTarget = this.makeSoTargetDetails(listPhone, createForm)
      targetData.koji_yotei_date = this.internalSoInformation.constructionScheduleDate;
      targetData.gno = this.internalSoInformation.gno;
      targetData.seq_no = null;
      const {juchu_no, juchu_meisai_no} = this.$route.query
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
        juch_no: juchu_no.toString(),
        juch_mesai_no: juchu_meisai_no.toString(),
        options: requestOptions,
        target: targetData
      };
      console.log(convData)
      NewSoRegiRequest.push(convData);
      const screen_id = this.screenId ? this.screenId : getScreenId(this.$route.name)
      // SO情報登録API呼び出し
      soInfoRegistration(screen_id, NewSoRegiRequest)
        .then((res) => {
          console.log(res)
          alert(this.$t(`customerDetail.soDataCreate.messageSuccess`).toString())
          const soData = this.$refs.soDataCreateRefs as InstanceType<typeof SoDataCreate>
          soData.refresh()
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
        })
        .finally(() => {
          if (this.callbackFunction) {
            this.callbackFunction()
          }
          this.$$closeLoadingScreen() // Loading処理の解除
        });
    },
    callApi() {
      this.soCheckBox = {}
      this.createAPI()
      let component = null
      component = (this.$refs.soDataCreateRefs as InstanceType<typeof SoDataCreate>)
      if (component) {
        component.refresh()
      }
    },
    createAPI() {
      this.soInformation.subNumber = "";
      this.soInformation.gno = this.gno;
      this.soInformation.companyName = this.setisaki_name;
      this.soInformation.juch_no = this.juchu_no;
      this.soInformation.juch_mesai_no = this.juchu_meisai_no;
      this.soInformation.district = this.chiku;
      console.log("regisuter_created " + this.soInformation.juch_no + "," + this.soInformation.juch_mesai_no)
      // 個別解約時の工事予定日を設定する すでに画面を開いている場合はwatchで設定
      if(!this.soInformation.constructionScheduleDate){
        this.soInformation.constructionScheduleDate = this.kojiYoteiDate;
      }
      const screen_id = this.screenId ? this.screenId : getScreenId(this.$route.name)
      console.log(screen_id)
      this.createPermission = this.$$hasCreatePermission(screen_id);
      // 新規でSO未作成のSOコントロールを検索
      let searchData: SearchFormData = {
        gno: this.gno,
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
      Promise.all([
        KubunListAPI(screen_id, 'STATUS'),
        getSoCheckBox(screen_id, this.internalSoInformation.gno),
        getSoInfoInquiryList(screen_id, searchData),
        getKanryoJohoList(screen_id, this.internalSoInformation.gno)
      ]).then((res) =>{
        this.status_kubun = res[0].data;
        this.soCheckBox = res[1].data
        Object.values(res[1].data).forEach((element) => {
          if (element == 1) {
            this.createSoData = true
          }
        })
        if (res[2].data.results.length > 0) {
          this.existNewSoControlFlg = true
        }
        res[3].data.forEach((element) => {
          if (element.kakin_end_date && element.kakin_end_date >= this.kakinEndDate) {
            this.kakinEndDate = element.kakin_end_date.replace(/-/g,"/")
          }
        })
      });
    }
  },
});
</script>
