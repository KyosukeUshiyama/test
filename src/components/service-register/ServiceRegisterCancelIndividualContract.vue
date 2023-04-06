<!--個別解約ダイアログ-->
<template>
  <v-form ref="form" v-model="valid">
    <v-card>
      <v-card-title>
        {{ t('title') }}
      </v-card-title>
      <v-card-text>
        <v-row>
          <v-col cols="12" md="3">
            <v-text-field
              v-model="internalContractCancelData.gno"
              :label="t('gno')"
              filled
              readonly
            />
          </v-col>
          <v-col cols="12" md="3">
            <v-text-field
              v-model="internalContractCancelData.gserialNo"
              :label="t('gserialNo')"
              filled
              readonly
            />
          </v-col>
          <v-col cols="12" md="3">
            <v-text-field
              v-model="internalContractCancelData.ufCode"
              :label="t('ufCode')"
              filled
              readonly
            />
          </v-col>
        </v-row>
        <v-row>
          <v-col cols="12" md="6">
            <v-text-field
              v-model="internalContractCancelData.kanji"
              :label="t('kanji')"
              filled
              readonly
            />
          </v-col>
          <v-col cols="12" md="6">
            <v-text-field
              v-model="internalContractCancelData.furigana"
              filled
              :label="t('furigana')"
              readonly
            />
          </v-col>
        </v-row>
        <v-row>
          <v-col cols="12" md="3">
            <app-date-picker
              v-model="internalContractCancelData.billingEndDate"
              :label="t('billingEndDate')"
              :rules="billingEndDateRule"
              :required="!input_validate"
            />
          </v-col>
          <v-col cols="12" md="3">
            <app-date-picker
              v-model="internalContractCancelData.contractCancelAcceptDate"
              :label="t('contractCancelAcceptDate')"
              :rules="contractCancelAcceptDateRule"
              :required="!input_validate"
            />
          </v-col>
        </v-row>
        <v-row>
          <v-col cols="12">
            <v-data-table
              v-model="selected"
              show-select
              item-key="id"
              :headers="headers"
              :items="internalContractCancelData.phonelist"
              :item-class="itemRowBackground"
              :options.sync="tableOptions"
              hide-default-footer
            >
              <template v-slot:[`item.daihyo_kubun`]="{ item }">
                {{ item.daihyo_kubun === "1" ? '◎' : '' }}
              </template>
              <template v-slot:top="{ pagination, options, updateOptions }">
                <v-data-footer
                  :pagination="pagination"
                  :options="options"
                  :items-per-page-options="[5, 10, 15, 20, -1]"
                  @update:options="updateOptions"
                />
              </template>
              <template #no-data>
                <p>{{ t('messages.notFound') }}</p>
              </template>
            </v-data-table>
          </v-col>
        </v-row>
      </v-card-text>
      <v-card-actions class="d-flex justify-space-between">
        <v-btn @click="close">
          {{ t('close') }}
        </v-btn>
        <div class="d-flex">
          <v-btn :disabled="btn_cancel_cancellation" color="primary" class="mr-5" @click="confirm(true)">
            {{ t('confirmCancelContractCancellation') }}
          </v-btn>
          <v-btn :disabled="btn_cancel" color="error" @click="confirm()">
            {{ t('confirmContractCancellation') }}
          </v-btn>
        </div>
      </v-card-actions>
    </v-card>
  </v-form>
</template>

<script lang="ts">
import { DEFAULT_HYOJI_NUM } from '@/constant/common-constant';
import {
  CANCEL_DELETE_REMOVE,
  CONFIRMATION,
  WHILE_INPUT,
} from '@/constants/dialog';
import { DialogOptions } from '@/models/dialog';
import {
  IndividualContractCancelState,
  IndividualContractCancelData,
  IndividualContractCancelPhone,
  CANCEL,
  CANCEL_CANCELLATION,
  STATUS_KAIYAKU,
} from '@/models/service-register';
import { FormRef, InputValidation } from '@/models/validation';
import { formatDate, required } from '@/utils/validation';
import Vue, { PropType } from 'vue';
import { TranslateResult } from 'vue-i18n';
import { DataTableHeader } from 'vuetify';
import AppDatePicker from '../UI/AppDatePicker.vue';

export default Vue.extend({
  components: { AppDatePicker },
  props: {
    individualContractCancelData: {
      type: Object as PropType<IndividualContractCancelData>,
      required: true,
    },
  },
  data(): IndividualContractCancelState {
    return {
      selected: [],
      internalContractCancelData: this.individualContractCancelData,
      valid: false,
      isDirty: false,
      btn_cancel: true,
      btn_cancel_cancellation: true,
      input_validate: true,
      tableOptions: {
        page: 1,
        itemsPerPage: DEFAULT_HYOJI_NUM,
        sortBy: ['no'],
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
          text: this.t('publishedNo'),
          value: 'publishedNo',
        },
        {
          text: this.t('routingNo'),
          value: 'routingNo',
        },
        {
          text: this.t('linkedNo'),
          value: 'linkedNo',
        },
        {
          text: this.t('registrationNo'),
          value: 'registrationNo',
        },
        {
          text: this.t('representative'),
          value: 'daihyo_kubun',
        },
        {
          text: this.t('callNoNotificationSection'),
          value: 'callNoNotificationSection',
        },
        {
          text: this.t('noIncomingCalls'),
          value: 'noIncomingCalls',
        },
        {
          text: this.t('name'),
          value: 'name',
          width: '150px',
        },
        {
          text: this.t('contractCancelDate'),
          value: 'contractCancelDate',
        },
      ];
    },
    billingEndDateRule(): InputValidation {
      const { billingEndDate } = this.internalContractCancelData;
      if(this.input_validate) return[];
      return [
        required(billingEndDate, this.t('billingEndDate')),
        formatDate(billingEndDate),
      ];
    },
    contractCancelAcceptDateRule(): InputValidation {
      const { contractCancelAcceptDate } = this.internalContractCancelData;
      if(this.input_validate) return[];
      return [
        required(contractCancelAcceptDate, this.t('contractCancelAcceptDate')),
        formatDate(contractCancelAcceptDate),
      ];
    },
  },
    watch: {
    internalContractCancelData: {
      deep: true,
      handler() {
        this.isDirty = true;
      }
    },
    selected:{
      // 確認ボタン活性切替
      handler(select){
        // 両方無効状態に初期化
        this.btn_cancel_cancellation = true;
        this.btn_cancel = true;
        // 日付入力不要状態に設定(バリデーション無)
        this.input_validate = true;
        select.forEach((item: IndividualContractCancelPhone) =>{
          if(item.juchu_kubun === STATUS_KAIYAKU){
            // 解約データ選択時に解約取消確認ボタン有効 バリデーション無
            this.btn_cancel_cancellation = false;
            this.input_validate = true;
          }else {
            // 未解約データ選択時に解約確認ボタン有効 バリデーション有
            this.btn_cancel = false;
            this.input_validate = false;
          }
        })
        // 両方有効のときは無効に設定
        if(!this.btn_cancel && !this.btn_cancel_cancellation){
          this.btn_cancel = true;
          this.btn_cancel_cancellation = true;
        }
      }
    },
  },
  methods: {
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
    confirm(cancelCancellation = false): void {
      const data: IndividualContractCancelData = {
        juch_no: this.internalContractCancelData.juch_no,
        juch_mesai_no: this.internalContractCancelData.juch_mesai_no,
        gno: this.internalContractCancelData.gno,
        gserialNo: this.internalContractCancelData.gserialNo,
        ufCode: this.internalContractCancelData.ufCode,
        kanji: this.internalContractCancelData.kanji,
        furigana: this.internalContractCancelData.furigana,
        billingEndDate: this.internalContractCancelData.billingEndDate,
        contractCancelAcceptDate: this.internalContractCancelData.contractCancelAcceptDate,
        phonelist: this.selected,
        editStatus: this.internalContractCancelData.editStatus,
      }
      // 編集ステータスの設定
      if (cancelCancellation) {
        this.internalContractCancelData.editStatus = CANCEL_CANCELLATION;
      } else {
        this.internalContractCancelData.editStatus = CANCEL;
      }
      const valid = (this.$refs.form as FormRef).validate();
      if (valid) {
        this.$store.dispatch('dialog/open', {
          type: CONFIRMATION,
          subType: CANCEL_DELETE_REMOVE,
          messages: [
            this.t(
              `messages.${cancelCancellation ? 'cancelCancellation' : 'cancel'}`
            ),
          ],
          callback: () =>
            this.$emit(
              cancelCancellation ? 'cancel-cancellation' : 'cancel',
              data
            ),
        });
      }
    },
    t(field: string): TranslateResult {
      return this.$t(
        `customerDetail.serviceRegister.individualContractCancel.${field}`
      );
    },
    // 解約データ用クラス追加関数
    itemRowBackground(item: IndividualContractCancelPhone): string {
      return item.juchu_kubun === STATUS_KAIYAKU ? 'kaiyaku-style' : '';
    },
  },
});
</script>

<style lang="scss">
.kaiyaku-style {
  background-color: #afafaf !important;
}
.kaiyaku-style td {
  color: #ffffff;
}
</style>