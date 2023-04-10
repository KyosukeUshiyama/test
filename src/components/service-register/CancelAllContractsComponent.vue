<!--全解約登録ダイアログ-->
<template>
  <v-card>
    <v-card-title>
      {{ t('title') }}
    </v-card-title>
    <v-form ref="form" v-model="valid">
      <v-card-text>
        <v-row align="end">
          <v-col cols="12" md="3">
            <v-text-field
              v-model="cancelAllDataForm.gno"
              :label="t('gno')"
              filled
              readonly
              dense
            />
          </v-col>
          <v-col cols="12" md="3" offset="6">
            <v-text-field
              v-model="cancelAllDataForm.ufCode"
              :label="t('ufCode')"
              filled
              readonly
              dense
            />
          </v-col>
        </v-row>
        <v-row>
          <v-col cols="12" md="6">
            <v-text-field
              v-model="cancelAllDataForm.kanji"
              :label="t('kanji')"
              filled
              readonly
              dense
            />
          </v-col>
          <v-col cols="12" md="6">
            <v-text-field
              v-model="cancelAllDataForm.furigana"
              :label="t('furigana')"
              filled
              readonly
              dense
            />
          </v-col>
        </v-row>
        <v-row>
          <v-container fluid>
            <label class="required-label-radio-group">
              {{ t('cancellationRequester') }}
            </label>
            <v-radio-group
              v-model="cancelAllDataForm.cancellationRequester"
              :rules="cancellationRequesterRule"
              row
              :disabled="cancelStatus"
            >
              <v-radio
                v-for="requester in cancellationRequester"
                :key="requester.value"
                :label="requester.label"
                :value="requester.value"
              />
            </v-radio-group>
          </v-container>
        </v-row>
        <v-row>
          <v-container fluid>
            <label class="required-label-radio-group">
              {{ t('cancellationSection') }}
            </label>
            <v-radio-group
              v-model="cancelAllDataForm.cancellationSection"
              :rules="cancellationSectionRule"
              row
              :disabled="cancelStatus"
            >
              <v-radio
                v-for="section in cancellationSection"
                :key="section.value"
                :label="section.label"
                :value="section.value"
              />
            </v-radio-group>
          </v-container>
        </v-row>
        <v-row>
          <v-col cols="12" md="3">
            <v-combobox
              v-model="cancelAllDataForm.cancellationReasonCode"
              :items="cancelRiyuCodeList"
              :label="t('cancellationReasonCode')"
              :no-data-text="t('messages.notFound')"
              class="required-label"
              hide-details="auto"
              :rules="cancellationReasonCodeRule"
              @input="changeRiyuMst"
            />
          </v-col>
          <v-col cols="12" md="6">
            <v-text-field
              v-model="cancelAllDataForm.cancellationReasonContent"
              filled
              readonly
            />
          </v-col>
        </v-row>
        <v-row>
          <v-col cols="12" md="12">
            <v-textarea
              v-model="cancelAllDataForm.cancellationRequesterMemo"
              maxlength="400"
              :label="t('cancellationRequesterMemo')"
              rows="2"
            />
          </v-col>
        </v-row>
        <v-row>
          <v-col cols="12" md="3">
            <v-select
              v-model="cancelAllDataForm.cancellationSectionCode"
              :items="sectionOptions"
              item-value="value"
              item-text="value"
              :label="t('cancellationSectionCode')"
              :no-data-text="t('messages.notFound')"
              class="required-label"
              hide-details="auto"
              :rules="cancellationSectionCodeRule"
              :disabled="cancelStatus"
              @input="changeSection"
            />
          </v-col>
          <v-col cols="12" md="6">
            <v-text-field
              v-model="cancelAllDataForm.cancellationSectionContent"
              filled
              readonly
            />
          </v-col>
        </v-row>
        <v-row>
          <v-col cols="12" md="6">
            <app-date-picker
              v-model="cancelAllDataForm.dateOfService"
              :rules="dateOfServiceRule"
              :label="t('dateOfService')"
            />
          </v-col>
          <v-col cols="12" md="6">
            <app-date-picker
              v-model="cancelAllDataForm.removalDeadline"
              :rules="removalDeadlineRule"
              :label="t('removalDeadline')"
            />
          </v-col>
        </v-row>
        <v-row>
          <v-col cols="12" md="6">
            <v-text-field
              v-model="cancelAllDataForm.contactOfConstructionRemoval"
              maxlength="150"
              :label="t('contactOfConstructionRemoval')"
            />
          </v-col>
          <v-col cols="12" md="6">
            <v-text-field
              v-model="cancelAllDataForm.ipLine"
              maxlength="150"
              :label="t('ipLine')"
            />
          </v-col>
        </v-row>
        <v-row>
          <v-col cols="12" md="6">
            <app-date-picker
              v-model="cancelAllDataForm.billingEndDate"
              required
              :rules="billingEndDateRule"
              :disabled="cancelStatus"
              :label="t('billingEndDate')"
            />
          </v-col>
          <v-col cols="12" md="6">
            <app-date-picker
              v-model="cancelAllDataForm.contractCancellationAcceptanceDate"
              required
              :rules="contractCancellationAcceptanceDateRule"
              :label="t('contractCancellationAcceptanceDate')"
            />
          </v-col>
        </v-row>
        <v-row>
          <v-col cols="12" md="6">
            <app-date-picker
              :rules="constructionScheduleDateRule"
              :label="t('constructionScheduleDate')"
            />
          </v-col>
        </v-row>
      </v-card-text>
    </v-form>
    <v-card-actions class="justify-space-between">
      <v-btn @click="close">
        {{ t('closeButton') }}
      </v-btn>
      <div class="d-flex">
        <v-btn :disabled="!cancelStatus" color="info" class="mr-5" @click="cancel(true)">
          {{ t('cancelCancellationButton') }}
        </v-btn>
        <v-btn color="error" @click="cancel()">
          {{ t('cancelButton') }}
        </v-btn>
      </div>
    </v-card-actions>
  </v-card>
</template>

<script lang="ts">
import Vue, { PropType } from 'vue';
import AppDatePicker from '../UI/AppDatePicker.vue';
import {
  CancelAllContractlState,
  CancelAllDataReponse,
  CANCEL,
  CANCEL_CANCELLATION,
} from '@/models/service-register';
import { TranslateResult } from 'vue-i18n';
import {
  CANCEL_DELETE_REMOVE,
  CONFIRMATION,
  WHILE_INPUT,
} from '@/constants/dialog';
import { FormRef, InputValidation } from '@/models/validation';
import { dateRange, formatDate, required } from '@/utils/validation';
import { RiyuMstListAPI } from '@/api/riyu-mst-api';
import { KubunListAPI } from '@/api/kubun-kanri';
import { DialogOptions } from '@/models/dialog';
import { getScreenId } from '@/utils/screenIds';
import { getAllCancelInfoAPI } from '@/api/cancel-info-api';
import dateFormat from "dateformat";

export default Vue.extend({
  name: 'AllCancelRegistrationDialog',
  components: { AppDatePicker },
  props: {
    cancelAllData: {
      type: Object as PropType<CancelAllDataReponse>,
      default: null,
    },
    cancelStatus: {
      type: Boolean,
      default: false,
    }
  },
  data(): CancelAllContractlState {
    const { cancellationRequester, cancellationSection } = this.cancelAllData;
    return {
      cancelAllDataForm: {
        ...this.cancelAllData,
        // ラジオボタン初期値設定
        cancellationRequester: cancellationRequester || '2',
        cancellationSection: cancellationSection || '3',
      },
      isDirty: false,
      valid: false,
      cancellationRequesterRule: [
        (value) =>
          required(
            value,
            this.$t(
              'customerDetail.serviceRegister.allContractsCancel.cancellationRequester'
            )
          ),
      ],
      cancellationSectionRule: [
        (value) =>
          required(
            value,
            this.$t(
              'customerDetail.serviceRegister.allContractsCancel.cancellationSection'
            )
          ),
      ],
      cancellationReasonCodeRule: [
        (value) =>
          required(
            value,
            this.$t(
              'customerDetail.serviceRegister.allContractsCancel.cancellationReasonCode'
            )
          ),
      ],
      cancellationSectionCodeRule: [
        (value) =>
          required(
            value,
            this.$t(
              'customerDetail.serviceRegister.allContractsCancel.cancellationSectionCode'
            )
          ),
      ],
      cancelRiyuCodeList: [],
      cancelRiyuCodeOptions: [],
      sectionOptions: [],
      cancellationRequester: [],
      cancellationSection: [],
      juch_mesai_sakjo_flg: '',
    };
  },
  computed: {
    dateOfServiceRule(): InputValidation {
      const { dateOfService } = this.cancelAllDataForm;
      // 解約状態の場合は範囲判定は行わない
      if (this.cancelStatus) {
        return [
          formatDate(dateOfService)
        ]
      }
      return [
        formatDate(dateOfService),
        dateRange(dateOfService, dateFormat(new Date(),"yyyy/mm/dd"), null)
      ];
    },
    removalDeadlineRule(): InputValidation {
      const { removalDeadline } = this.cancelAllDataForm;
      // 解約状態の場合は範囲判定は行わない
      if (this.cancelStatus) {
        return [
          formatDate(removalDeadline)
        ]
      }
      return [
        formatDate(removalDeadline),
        dateRange(removalDeadline, dateFormat(new Date(),"yyyy/mm/dd"), null)
      ];
    },
    billingEndDateRule(): InputValidation {
      const { billingEndDate } = this.cancelAllDataForm;
      return [
        required(billingEndDate, this.t('billingEndDate')),
        formatDate(billingEndDate),
      ];
    },
    contractCancellationAcceptanceDateRule(): InputValidation {
      const { contractCancellationAcceptanceDate } = this.cancelAllDataForm;
      return [
        required(
          contractCancellationAcceptanceDate,
          this.t('contractCancellationAcceptanceDate')
        ),
        formatDate(contractCancellationAcceptanceDate),
      ];
    },
    constructionScheduleDateRule(): InputValidation {
      const { constructionScheduleDate } = this.cancelAllDataForm;
      return [
        formatDate(constructionScheduleDate),
      ];
    },
  },
  watch: {
    cancelAllDataForm: {
      deep: true,
      handler() {
        this.isDirty = true;
      },
    },
  },
  created() {
    const cancelStatus = this.cancelStatus;
    Promise.all([RiyuMstListAPI(getScreenId(this.$route.name, this.$options.name)),
                KubunListAPI(getScreenId(this.$route.name, this.$options.name), 'CANCEL_KAIYAKU_KUBUN'),
                KubunListAPI(getScreenId(this.$route.name, this.$options.name), 'CANCEL_KAIYAKU_SHINSEISHA'),
                KubunListAPI(getScreenId(this.$route.name, this.$options.name), 'CANCEL_KAIYAKU_TAISHO_KUBUN')
      ]).then((res) => {
        this.cancelRiyuCodeOptions = res[0].data.map(
          ({ riyu_cd, riyu_mei, juch_mesai_sakjo_flg }) => ({
            value: riyu_cd,
            text: riyu_mei,
            juch_mesai_sakjo_flg: juch_mesai_sakjo_flg,
          })
        );
        this.cancelRiyuCodeList = res[0].data.map((element) => (element.riyu_cd));
        this.sectionOptions = res[1].data.map(({ vl, hyoji_mei }) => ({
          value: vl,
          text: hyoji_mei,
        }));
        this.cancellationRequester = res[2].data.map(({ vl, hyoji_mei }) => ({
          value: vl,
          label: hyoji_mei,
        }));
        this.cancellationSection = res[3].data.map(({ vl, hyoji_mei }) => ({
          value: vl,
          label: hyoji_mei,
        }));
        this.cancelAllDataForm.contractCancellationAcceptanceDate = dateFormat(new Date(),"yyyy/mm/dd")

        if (cancelStatus) {
          // 全解約情報の取得
          getAllCancelInfoAPI(getScreenId(this.$route.name, this.$options.name), this.cancelAllData.gno).then((allCancelInfo) => {
            this.cancelAllDataForm.cancellationRequester = allCancelInfo.data.c_sinsei
            this.cancelAllDataForm.cancellationSection = allCancelInfo.data.c_taish_kbn
            this.cancelAllDataForm.cancellationReasonCode = allCancelInfo.data.c_riyu_cd
            this.cancelAllDataForm.cancellationRequesterMemo = allCancelInfo.data.c_sinsei_memo
            this.cancelAllDataForm.cancellationSectionCode = allCancelInfo.data.cancel_kaiyaku_kubun
            this.cancelAllDataForm.dateOfService = allCancelInfo.data.service_teishi_kibo_date ? dateFormat(allCancelInfo.data.service_teishi_kibo_date, 'yyyy/mm/dd') : ''
            this.cancelAllDataForm.removalDeadline = allCancelInfo.data.tekkyo_koji_limit_date ? dateFormat(allCancelInfo.data.tekkyo_koji_limit_date, 'yyyy/mm/dd') : ''
            this.cancelAllDataForm.contactOfConstructionRemoval = allCancelInfo.data.tekkyo_koji_contact
            this.cancelAllDataForm.ipLine = allCancelInfo.data.ip_kaisen_jigyosha_tokki_jiko
            this.cancelAllDataForm.billingEndDate = allCancelInfo.data.kaiyaku_date ? dateFormat(allCancelInfo.data.kaiyaku_date, 'yyyy/mm/dd') : ''
            this.cancelAllDataForm.contractCancellationAcceptanceDate = allCancelInfo.data.kaiyaku_uketsuke_date ? dateFormat(allCancelInfo.data.kaiyaku_uketsuke_date, 'yyyy/mm/dd') : ''

            this.changeRiyuMst(allCancelInfo.data.c_riyu_cd)
            this.changeSection(allCancelInfo.data.cancel_kaiyaku_kubun)
          })
        }
        if (this.cancelStatus == false) {
          let cancel_kaiyaku_kubun = ''
          cancel_kaiyaku_kubun = this.cancelAllData.carrier_status == '800'? '02':'01'
          this.cancelAllDataForm.cancellationSectionCode = cancel_kaiyaku_kubun
          this.changeSection(cancel_kaiyaku_kubun)
        }
      }
    );
  },
  methods: {
    changeRiyuMst(riyu_cd: string) {
      this.cancelAllDataForm.cancellationReasonContent =
        this.cancelRiyuCodeOptions
          ?.filter((item) => item.value === riyu_cd)
          .map((item) => item.text)[0];
      this.$nextTick(function() {
        // DOM更新後でないとクリアされないため更新後に判定
        // 対応する理由コードが無い場合はクリアする
        if (this.cancelAllDataForm.cancellationReasonContent === undefined) {
          this.cancelAllDataForm.cancellationReasonCode = ''
        }else {
          this.juch_mesai_sakjo_flg = this.cancelRiyuCodeOptions?.filter((item) => item.value === riyu_cd)[0].juch_mesai_sakjo_flg;
        }
      });
    },
    changeSection(vl: string) {
      this.cancelAllDataForm.cancellationSectionContent = this.sectionOptions
        ?.filter((item) => item.value === vl)
        .map((item) => item.text)[0];
    },
    close(): void {
      if (!this.cancelStatus && this.isDirty) {
        this.$store.dispatch('dialog/open', {
          type: CONFIRMATION,
          subType: WHILE_INPUT,
          callback: () => this.$emit('close'),
        } as DialogOptions);
      } else {
        this.$emit('close');
      }
    },
    cancel(cancelCancellation = false): void {
      console.log(cancelCancellation);
      // 全解約取消
      if (cancelCancellation) {
        console.log('全解約取消');
        this.cancelAllDataForm.editStatus = CANCEL;
      // 全解約登録
      } else {
        console.log('全解約登録');
        this.cancelAllDataForm.editStatus = CANCEL_CANCELLATION;
        this.cancelAllDataForm.sakjo_flg = this.juch_mesai_sakjo_flg;
      }
      let valid = false;
      // 解約取消の場合はバリデーション判定を行わない
      if(cancelCancellation){
        valid = true;
      }else {
        valid = (this.$refs.form as FormRef).validate();
      }
      if (valid) {
        this.$store.dispatch('dialog/open', {
          type: CONFIRMATION,
          subType: CANCEL_DELETE_REMOVE,
          messages: [
            this.t(
              `messages.${cancelCancellation ? 'cancelCancellation' : 'cancel'}`
            ),
          ],
          callback: () =>{
            this.$emit(
              cancelCancellation ? 'cancel-cancellation' : 'cancel',
              this.cancelAllDataForm
            );
            this.$emit("change-cancel-status", cancelCancellation);
          }
        });
      }
    },
    t(field: string): TranslateResult {
      return this.$t(
        `customerDetail.serviceRegister.allContractsCancel.${field}`
      );
    },
  },
});
</script>
