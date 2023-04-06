<!--SO情報詳細ダイアログ&SO緊急投入ダイアログ-->
<template>
  <v-form ref="form" v-model="valid" @submit.prevent="submit">
    <v-card>
      <v-card-title>
        {{ showScreenId }}:{{ modeEmergency ? t('titleEmergency') : newMode ? t('titleNew') : t('title') }}
      </v-card-title>
      <v-card-text>
        <so-information-details-form
          :mode="mode"
          :detail="detail"
          :inquiry-flg="!modeEmergency"
          :so-information="internalSoInformation"
          :new-contract-cancel-disabled="newContractCancelDisabled"
          @field-change="handleFieldChange"
        />
        <v-row>
          <v-col cols="12" class="d-flex justify-space-between">
            <v-btn @click="close">
              {{ t('closeBtn') }}
            </v-btn>
            <div>
              <v-btn
                v-if="editMode && !newMode && deletable"
                color="error"
                class="mr-5"
                :disabled="!updatePermission"
                @click="remove"
              >
                {{ t('deleteBtn') }}
              </v-btn>
              <v-btn
                v-if="editMode || newMode"
                color="primary"
                :disabled="!updatePermission"
                @click="register"
              >
                {{ t('registerBtn') }}
              </v-btn>
              <v-btn
                v-if="!editMode&& !newMode"
                color="primary"
                :disabled="disabled || isConfirmed"
                @click="confirm" 
              >
                {{ t('confirmBtn') }}
              </v-btn>
            </div>
          </v-col>
        </v-row>
        <v-row class="mt-10">
          <v-col cols="12">
            <h2 class="mb-5">
              {{ t('createSo') }}
            </h2>
            <v-btn
              v-if="editMode && deletable"
              :disabled="internalSoInformation.constructionScheduleDateUndecided || !createPermission"
              color="primary"
              @click="showDialogSODataCreate = true"
            >
              {{ t('createSo') }}
            </v-btn>
          </v-col>
        </v-row>
        <v-row>
          <v-col md="8" cols="12">
            <v-text-field
              v-model="internalSoInformation.soFileName"
              :label="t('soFileName')"
              filled
              readonly
            />
          </v-col>
          <v-col md="4" cols="12">
            <v-text-field
              v-model="internalSoInformation.sofileCreateDate"
              :label="t('sofileCreateDate')"
              filled
              readonly
            />
          </v-col>
        </v-row>
        <v-row>
          <v-col cols="12">
            <h2>{{ t('soInputInfo') }}</h2>
          </v-col>
          <v-col md="8" cols="12">
            <v-text-field
              v-model="internalSoInformation.soSendDate"
              :label="t('soSendDate')"
              filled
              readonly
            />
          </v-col>
          <v-col md="4" cols="12">
            <v-text-field
              v-model="internalSoInformation.soSender"
              :label="t('soSender')"
              filled
              readonly
            />
          </v-col>
        </v-row>
        <v-row>
          <v-col md="8" cols="12">
            <v-text-field
              v-model="internalSoInformation.resultImportDate"
              :label="t('resultImportDate')"
              filled
              readonly
            />
          </v-col>
        </v-row>
        <v-row>
          <v-col cols="12">
            <v-textarea
              v-model="internalSoInformation.errorComment"
              :label="t('errorComment')"
              filled
              readonly
              rows="2"
            />
          </v-col>
        </v-row>
        <v-row>
          <v-col cols="12">
            <h2>{{ t('updateInfo') }}</h2>
          </v-col>
          <v-col cols="8">
            <v-select
              v-model="internalSoInformation.status"
              :label="t('status')"
              :items="status"
              :rules="internalSoInformationRules"
              :readonly="!editMode"
              :filled="!editMode"
              :no-data-text="t('messages.notFound')"
              class="required-label"
            />
          </v-col>
        </v-row>
        <v-row>
          <v-col md="8" cols="12">
            <v-text-field
              v-model="internalSoInformation.registrationDate"
              :label="t('registrationDate')"
              filled
              readonly
            />
          </v-col>
          <v-col md="4" cols="12">
            <v-text-field
              v-model="internalSoInformation.registeredUser"
              :label="t('registeredUser')"
              filled
              readonly
            />
          </v-col>
        </v-row>
        <v-row>
          <v-col md="8" cols="12">
            <v-text-field
              v-model="internalSoInformation.updatedDate"
              :label="t('updatedDate')"
              filled
              readonly
            />
          </v-col>
          <v-col md="4" cols="12">
            <v-text-field
              v-model="internalSoInformation.updatedUser"
              :label="t('updatedUser')"
              filled
              readonly
            />
          </v-col>
        </v-row>
      </v-card-text>
      <!-- SOデータ作成ダイアログ呼び出し -->
      <v-dialog v-model="showDialogSODataCreate" width="1200" persistent>
        <so-data-create
          v-if="showDialogSODataCreate"
          ref="soDataCreateRefs"
          :gno-code="internalSoInformation.gno"
          :disabled="disabled"
          :mode="mode"
          @close="handleCloseDialogSOData"
          @confirm="handleConfirmDialogSOData"
        />
      </v-dialog>
      <!-- SOデータ作成確認ダイアログ呼び出し -->
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
          @create="handleCreateDialogSODataConfirm"
        />
      </v-dialog>
    </v-card>
  </v-form>
</template>

<script lang="ts">
import Vue, { PropType } from 'vue';
import {
  SoInformation,
  SoInformationDetailsData,
  SelectOptions,
  SO_EMERGENCY,
  SO_NEW
} from '@/models/so';
import { TranslateResult } from 'vue-i18n';
import { isObjKey } from '@/utils/helper';
import {
  CANCEL_DELETE_REMOVE,
  CONFIRMATION,
  MESSAGE,
  MESSAGE_ERROR,
  REGISTER_UPDATE_EXECUTE,
  SO_LOADING,
  SO_RESULT_EXCEED_1_MIN,
  SO_RESULT_FAIL,
  SO_RESULT_SUCESS,
  WHILE_INPUT,
} from '@/constants/dialog';
import { ERROR_STATUS_503 } from '@/constant/common-constant';
import SoDataCreate from '@/components/customer/SoDataCreate.vue';
import SoDataCreateConfirm from '@/components/customer/SoDataCreateConfirm.vue';
import SoInformationDetailsForm from './SoInformationDetailsForm.vue';
import { FormRef } from '@/models/validation';
import { required } from '@/utils/validation';
import { DialogOptions } from '@/models/dialog';
import { getScreenId } from '@/utils/screenIds';
import {
  SoInformationDetailUpdateRequestData,
  SoInformationDetailUpdateSakujoFlgRequestData,
  updateSoIformationDetails,
  updateSoIformationDetailsSakujoFlg,
  getSoInfoInquiryList
} from '@/api/so-info-inquiry-api';
import {
  SoCreateOptions,
  SoCreateTarget,
  SelectTable,
  SoUrgencySendData,
  MakeSoGnoNewRequest,
  SoDataCreateForm,
  SoSoshinKanriRequest
} from '@/models/so-batch';
import {
  postSoUrgencySend,
  SoSoshinKanriCreate,
} from '@/api/so-batch-api';
import { getKanryoJohoList } from '../../api/CompletionInfomation-api';
import { KubunListAPI, ResponseData_KUBUN_KANRI, STATUS_COMPLETE } from '@/api/kubun-kanri';
import { 
  SoInfomationConfirmOneData, 
  SoMakeTarget,
  SoOptionAdd,
  SoSerialAndAskTelNo,
} from '@/models/so/index';
import { AxiosError } from 'axios';
import { FLAG_DELETED, FLAG_NORMAL, JIDO_UKAI_SERVICE_CD } from '@/constant/common-constant';
import { SearchFormData, SHINKI_KAIYAKU_KUBUN_NEW, SO_SAKUSEI_KUBUN_NOT_CREATED } from '@/models/so-inquiry';

export default Vue.extend({
  name: 'SoInformationDetails',
  components: { SoDataCreate, SoDataCreateConfirm, SoInformationDetailsForm },
  props: {
    soInformation: {
      type: Object as PropType<SoInformation>,
      required: true,
    },
    mode: {
      type: String as PropType<string>,
      required: true,
    },
  },
  data(): SoInformationDetailsData {
    return {
      valid: false,
      isDirty: false,
      showDialogSODataCreateConfirm: false,
      showDialogSODataCreate: false,
      internalSoInformation: this.soInformation,
      initialScheduleDate: this.soInformation.constructionScheduleDate,
      kakinEndDate: '',
      registerContentsOptions: [],
      internalSoInformationRules: [
        (value) =>
          required(value, this.$t('soInquiry.soInformationDetails.status')),
      ],
      status: [],
      initialConstructionScheduleDate:
        this.soInformation.constructionScheduleDate,
      callApiDone: false,
      modeEmergency: this.mode === SO_EMERGENCY,
      newMode: this.mode === SO_NEW,
      deletable: false,
      requestData: null,
      createForm: null,
      listPhone: null,
      existNewSoControlFlg: false,
      selfNewSoControlFlg: false,
      warning_message: [],
      inputDataList: [
        {
          title: 'オプション追加',
          values: ['着信転送サービス 03-1111-1111'],
        },
      ],
      createPermission: false,
      updatePermission: false,
      detail: false,
      isConfirmed: false,
    };
  },
  computed: {
    newContractCancelDisabled(): boolean {
      console.log(this.mode);
      if (!this.editMode) return true;
      return !!this.internalSoInformation.constructionScheduleDateUndecided;
    },
    // 画面ID取得
    showScreenId(): string {
      return getScreenId(this.$route.name, this.$options.name, this.mode);
    },
    editMode(): boolean {
      return this.mode !== SO_EMERGENCY
    },
    disabled(): boolean {
      return ["1","2","9"].includes(this.soInformation.status)
    }
  },
  watch: {
    internalSoInformation: {
      deep: true,
      handler() {
        this.isDirty = true;
      },
    },
  },
  created() {
    this.detail = this.$options.name == 'SoInformationDetails'
    this.createPermission = this.$$hasCreatePermission(this.showScreenId);
    this.updatePermission = this.$$hasUpdatePermission(this.showScreenId);
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
    let screen_id = getScreenId(this.$route.name, this.$options.name, this.mode)
    Promise.all([
      KubunListAPI(screen_id, 'STATUS'), //ステータス
      getSoInfoInquiryList(screen_id, searchData),
      getKanryoJohoList(screen_id, this.internalSoInformation.gno)
    ])
      .then((res) => {
        const kubun_status = this.kubunToSelect(res[0].data)
        const status_complete = kubun_status.filter((item)=> item.text == STATUS_COMPLETE)[0].value
        // ステータスが「完了」の場合削除ボタンを非表示
        this.deletable = this.soInformation.status !== status_complete ? true : false
        this.status = kubun_status
        // 新規SO未作成のレコードがないかチェック
        if (res[1].data.results.length > 0) {
          this.existNewSoControlFlg = true
        }
        // 編集中のデータが新規SO未作成かチェック
        if (this.internalSoInformation.newContractCancel == SHINKI_KAIYAKU_KUBUN_NEW && this.internalSoInformation.createSoFile == SO_SAKUSEI_KUBUN_NOT_CREATED) {
          this.selfNewSoControlFlg = true
        }
        res[2].data.forEach((element) => {
          if (element.kakin_end_date && element.kakin_end_date >= this.kakinEndDate) {
            this.kakinEndDate = element.kakin_end_date.replace(/-/g,"/")
          }
        })
      })
      .catch((err: AxiosError) => {
        console.error(err.response);
      });
  },
  methods: {
    t(field: string): TranslateResult {
      return this.$t(`soInquiry.soInformationDetails.${field}`);
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
    // 削除ボタンクリック時
    remove(): void {
      this.$store.dispatch('dialog/open', {
        type: CONFIRMATION,
        subType: CANCEL_DELETE_REMOVE,
        messages: [this.t('messages.delete')],
        callback: async (close) => {
          console.log('SoInformationDetails-削除ボタン');
          await this.updateSoInformationDetailSakujo();
          this.$emit('remove', this.internalSoInformation);
          close();
        },
      } as DialogOptions);
    },
    // 登録ボタンクリック時
    register(): void {
      const valid = (this.$refs.form as FormRef).validate();
      console.log(this.soInformation);
      if (valid) {
        // 新規解約区分が新規 && so作成区分が未作成 && 編集中のデータが新規でSO未作成でない && 新規でSO未作成のSOコントロールが登録されている場合エラー
        if (this.internalSoInformation.newContractCancel == SHINKI_KAIYAKU_KUBUN_NEW
          && this.internalSoInformation.createSoFile == SO_SAKUSEI_KUBUN_NOT_CREATED
          && this.selfNewSoControlFlg == false
          && this.existNewSoControlFlg == true) 
        {
          this.$store.dispatch('dialog/open', {
            type: MESSAGE,
            subType: MESSAGE_ERROR,
            messages: [this.t('messages.error.newSoControlError')]
          })
          return
        }
        this.$store.dispatch('dialog/open', {
          type: CONFIRMATION,
          subType: REGISTER_UPDATE_EXECUTE,
          messages: [this.t('messages.update')],
          autoClose: false,
          callback: async (close) => {
            console.log('SoInformationDetails-登録ボタン');
            await this.updateSoInformationDetail();
            this.$emit('register', this.internalSoInformation);
            close();
          },
        } as DialogOptions);
      }
    },

    callApi(): void {
      this.isConfirmed = true
      let batchRequest: SoUrgencySendData = {
        koji_yotei_date: this.soInformation.constructionScheduleDate,
        gno: this.soInformation.gno,
        seq_no: parseInt(this.soInformation.subNumber, 10),
      };
      const callback = () => this.$emit('confirm', this.internalSoInformation);
      this.callApiDone = false;

      this.$store.dispatch('dialog/open', {
        type: MESSAGE,
        subType: SO_LOADING,
      } as DialogOptions);

      postSoUrgencySend(getScreenId(this.$route.name, this.$options.name, this.mode), batchRequest)
        .then((res) => {
          if (res.data.status === 2) {
            // 成功
            this.$store.dispatch('dialog/open', {
              type: MESSAGE,
              subType: SO_RESULT_SUCESS,
              callback,
            });
          } else if (res.data.status === -1) {
            // タイムアウト
            this.$store.dispatch('dialog/open', {
              type: MESSAGE,
              subType: SO_RESULT_EXCEED_1_MIN,
              callback,
            });
          } else {
            // エラー(9がエラーのステータスだが、成功とタイムアウト以外は基本的に全てエラー扱いとする)
            this.$store.dispatch('dialog/open', {
              type: MESSAGE,
              subType: SO_RESULT_FAIL,
              callback,
            });
          }
        })
        .catch(() => {
          // エラー
          this.$store.dispatch('dialog/open', {
            type: MESSAGE,
            subType: SO_RESULT_FAIL,
            callback,
          });
        })
        .finally(() => (this.callApiDone = true));
    },
    confirm(): void {
      this.$store.dispatch('dialog/open', {
        type: CONFIRMATION,
        subType: REGISTER_UPDATE_EXECUTE,
        messages: [this.t('messages.confirm')],
        autoClose: false,
        callback: () => this.callApi(),
      } as DialogOptions);
    },
    async updateSoInformationDetail(): Promise<void> {
      const type = this.soInformation.type;
      const { data } = await KubunListAPI(getScreenId(this.$route.name, this.$options.name, this.mode), 'PROTCOL_KUBUN');
      const temp: SoInformationDetailUpdateRequestData = {
        shinki_kaiyaku_kubun: this.soInformation.newContractCancel,
        shubetsu: data.find(({ hyoji_mei }) => hyoji_mei === type)?.vl || type,
        so_sakusei_kubun: this.soInformation.createSoFile,
        kinkyu_flg: this.soInformation.emergencySendingFlag,
        toroku_kubun: this.soInformation.soRegistration,
        toroku_naiyo_id: this.soInformation.registerContents,
        biko: this.soInformation.comment,
        status: this.soInformation.status,
        koji_yotei_date: this.soInformation.constructionScheduleDate,
        oa_juchu_no: this.soInformation.oaOrderNumber,
        oa_juchu_edaban: this.soInformation.oaOrderSubNumber,
        juch_no: this.soInformation.juch_no,
        juch_mesai_no: this.soInformation.juch_mesai_no,
      };
      await updateSoIformationDetails(
        getScreenId(this.$route.name, this.$options.name, this.mode),
        this.initialConstructionScheduleDate,
        this.soInformation.gno,
        this.soInformation.subNumber,
        temp
      ).catch((err: AxiosError) => {
        if (err.response) {
            alert(err.response.data.title +'\n' +
            "エラー内容: " + err.response.data.detail)
        }
      }).finally(() => {
        console.log('更新完了');
      });
    },
    async updateSoInformationDetailSakujo(sakujo?: boolean): Promise<void> {
      const temp: SoInformationDetailUpdateSakujoFlgRequestData = {
        sakujo_flg: sakujo == null ? FLAG_DELETED : sakujo == false ? FLAG_NORMAL : FLAG_DELETED,
        juch_no: this.soInformation.juch_no,
        juch_mesai_no: this.soInformation.juch_mesai_no,
      };

      await updateSoIformationDetailsSakujoFlg(
        getScreenId(this.$route.name, this.$options.name, this.mode),
        this.soInformation.constructionScheduleDate,
        this.soInformation.gno,
        this.soInformation.subNumber,
        temp
      ).catch((err: AxiosError) => {
        if (err.response) {
          alert(err.response.data.title +'\n' +
          "エラー内容: " + err.response.data.detail)
        }
      }).finally(() => {
        console.log('削除フラグ更新完了,' + temp.sakujo_flg);
      });
    },
    handleCloseDialogSOData(): void {
      const close = (): void => {
        this.showDialogSODataCreate = false;
      };
      close();
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
    // SOデータ作成ダイアログからSOデータ作成確認ダイアログ呼び出し
    handleConfirmDialogSOData(listPhone: SelectTable, requestData: MakeSoGnoNewRequest, createform: SoDataCreateForm): void {
      this.requestData = requestData;
      this.createForm = createform;
      this.listPhone = listPhone;
      this.inputDataList = this.coverDataPhoneList(listPhone);
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
      this.showDialogSODataCreateConfirm = true;
      console.log('handleConfirmDialogSOData');
    },
    handleCloseDialogSODataConfirm(): void {
      console.log('handleCloseDialogSODataConfirm');
      const close = (): void => {
        this.showDialogSODataCreateConfirm = false;
      };
      close();
    },
    // SOデータ作成確認ダイアログから、作成処理呼び出し
    handleCreateDialogSODataConfirm(): void {
      this.showDialogSODataCreateConfirm = false;
      console.log('handleCreateDialogSODataConfirm');
      // SOデータ作成APIリクエスト用データ生成
      let requestOptions: SoCreateOptions = {}
      if (this.requestData != null && this.requestData.options != null) {
        requestOptions = this.requestData.options
      }
      let targetData: SoCreateTarget = this.makeSoTargetDetails(this.listPhone, this.createForm)
      targetData.koji_yotei_date = this.soInformation.constructionScheduleDate;
      targetData.gno = this.soInformation.gno;
      targetData.seq_no =  Number(this.soInformation.subNumber);
      const NewSoRegiRequest: SoSoshinKanriRequest = {
        gno: this.internalSoInformation.gno,
        juch_no: this.internalSoInformation.juch_no,
        juch_mesai_no: this.internalSoInformation.juch_mesai_no,
        koji_yotei_date: this.soInformation.constructionScheduleDate,
        options: requestOptions,
        target: targetData
      };
      // SOデータ作成API実施
      SoSoshinKanriCreate(getScreenId(this.$route.name, this.$options.name, this.mode), NewSoRegiRequest)
        .then((res) => {
          console.log(res)
          alert(this.$t(`customerDetail.soDataCreate.messageSuccess`).toString())
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
        this.showDialogSODataCreate = false;
    },
    handleFieldChange<K extends keyof SoInformation>(
      field: K,
      value: SoInformation[K]
    ): void {
      if (!isObjKey(field, this.internalSoInformation)) return

      this.internalSoInformation[field] = value;
      if (field === 'constructionScheduleDate' && value === '9999/12/31') {
        this.internalSoInformation.constructionScheduleDateUndecided = true;
      }
    },
    kubunToSelect(kubunList: ResponseData_KUBUN_KANRI[]): SelectOptions[] {
      return kubunList.map(({ vl, hyoji_mei }) => ({
        value: vl,
        text: hyoji_mei,
      }));
    }
  },
});
</script>
