<!--SOデータ作成ダイアログ（顧客検索から）-->
<template>
  <v-form ref="form" v-model="valid">
    <v-card :elevation="elevation">
      <v-card-title>
        <div>
          <h2 v-if="showMainTitle" class="mb-5">
            {{ t('mainTitle') }}
          </h2>
          <h3>
            {{ t('subTitle') }}
          </h3>
        </div>
      </v-card-title>
      <v-card-text>
        <v-checkbox
          v-model="form.newContract"
          :label="t('checkBoxNewContract')"
          :rules="rulesCallSelected"
        />
        <v-checkbox
          v-model="form.optionAdd"
          :label="t('checkBoxAddOption')"
          :rules="rulesCallSelected"
        />
        <v-data-table
          :headers="headersOptionAdd"
          :items="listService"
          :hide-default-footer="true"
          class="elevation-1 ml-3"
        >
          <template v-slot:top="{ pagination, options, updateOptions }">
            <v-data-footer
              :pagination="pagination"
              :options="options"
              :items-per-page-options="[5, 10, 15, 20]"
              @update:options="updateOptions"
            />
          </template>
          <template v-slot:[`item.select_option`]="{ item }">
            <v-checkbox
              v-model="item.select_option"
              :disabled="!form.optionAdd"
              @change="selectDataService($event, item)"
            />
          </template>
          <template #no-data>
            <p>{{ t('messageNotFound') }}</p>
          </template>
        </v-data-table>
        <v-text-field
          class="custom-error"
          :rules="checkBoxAddOptionRule"
          :error="true"
        />

        <v-checkbox
          v-model="form.totalNumberChannel"
          :label="t('checkBoxChangeNumbersCH')"
          :rules="rulesCallSelected"
        />
        <v-col cols="4" class="ml-3">
          <v-subheader>{{ t('numbersOfChAfterChange') + ' ' + form.totalCH }}</v-subheader>
        </v-col>
        <v-checkbox
          v-model="form.addPhoneNumber"
          :label="t('checkBoxAddPhone')"
          :rules="rulesCallSelected"
        />
        <v-data-table
          :headers="headersAddPhoneNumber"
          :items="listPhoneNumber"
          :hide-default-footer="true"
          class="elevation-1 ml-3"
        >
          <template v-slot:top="{ pagination, options, updateOptions }">
            <v-data-footer
              :pagination="pagination"
              :options="options"
              :items-per-page-options="[5, 10, 15, 20]"
              @update:options="updateOptions"
            />
          </template>
          <template v-slot:[`item.select_add_phone`]="{ item }">
            <v-checkbox
              v-model="item.select_add_phone"
              :disabled="!form.addPhoneNumber"
              @change="selectDataAddPhoneNumber($event, item)"
            />
          </template>
          <template #no-data>
            <p>{{ t('messageNotFound') }}</p>
          </template>
        </v-data-table>
        <v-text-field
          class="custom-error"
          :rules="checkBoxChangeNumbersChRules"
          :error="true"
        />

        <v-checkbox
          v-model="form.callSelected[0]"
          :rules="rulesCallSelected"
          :label="t('checkBoxPauseCall')"
        />
        <v-checkbox
          v-model="form.callSelected[1]"
          :rules="rulesCallSelected"
          :label="t('checkBoxUnPauseCall')"
        />
        <v-checkbox
          v-model="form.callSelected[2]"
          :rules="rulesCallSelected"
          :label="t('checkBoxInitPass')"
        />
        <v-checkbox
          v-model="form.callSelected[3]"
          :rules="rulesCallSelected"
          :label="t('checkBoxCallNumNoti')"
        />
        <v-checkbox
          v-model="form.callSelected[4]"
          :rules="rulesCallSelected"
          :label="t('checkBoxChangeRegNum')"
        />

        <v-data-table
          :headers="headersCashRegister"
          :items="listCashRegister"
          :hide-default-footer="true"
          class="elevation-1 ml-3"
        >
          <template v-slot:top="{ pagination, options, updateOptions }">
            <v-data-footer
              :pagination="pagination"
              :options="options"
              :items-per-page-options="[5, 10, 15, 20]"
              @update:options="updateOptions"
            />
          </template>
          <template #no-data>
            <p>{{ t('messageNotFound') }}</p>
          </template>
        </v-data-table>

        <v-checkbox
          v-model="form.cancelSelected[0]"
          :rules="rulesCallSelected"
          :label="t('checkBoxCancelAllContract')"
        />
        <v-checkbox
          v-model="form.cancelSelected[1]"
          :rules="rulesCallSelected"
          :label="t('checkBoxMeeting')"
        />
        <v-row class="ml-3">
          <v-col md="3" cols="12">
            <v-text-field
              v-model="form.oldGno[0]"
              :rules="gnoRules1"
              :label="t('oldGNO')"
              :disabled="!form.cancelSelected[1]"
              @blur="inputOldGno"
            />
          </v-col>
          <v-col md="3" cols="12">
            <v-text-field
              v-model="form.oldGno[1]"
              :rules="gnoRules2"
              :label="t('oldGNO')"
              :disabled="!form.cancelSelected[1]"
              @blur="inputOldGno"
            />
          </v-col>
          <v-col md="3" cols="12">
            <v-text-field
              v-model="form.oldGno[2]"
              :rules="gnoRules3"
              :label="t('oldGNO')"
              :disabled="!form.cancelSelected[1]"
              @blur="inputOldGno"
            />
          </v-col>
        </v-row>
        <v-row class="ml-3">
          <v-col md="3" cols="12">
            <v-radio-group v-model="form.unitRadio" row
                           :disabled="!form.cancelSelected[1]"
            >
              <v-radio 
                :label="t('gnoUnit')" 
                :value="t('gnoUnit')" 
              />
              <v-radio
                :label="t('lineUnit')"
                :value="t('lineUnit')"
              />
            </v-radio-group>
          </v-col>
        </v-row>
        <v-data-table
          :headers="headersUnit"
          :items="listUnit"
          :hide-default-footer="true"
          class="elevation-1 ml-3"
        >
          <template v-slot:top="{ pagination, options, updateOptions }">
            <v-data-footer
              :pagination="pagination"
              :options="options"
              :items-per-page-options="[5, 10, 15, 20]"
              @update:options="updateOptions"
            />
          </template>
          <template v-slot:[`item.select_add_phone`]="{ item }">
            <v-checkbox
              v-model="item.select_add_phone"
              :disabled="form.unitRadio != t('lineUnit')"
              @change="selectDataEncounter($event, item)"
            />
          </template>
          <template #no-data>
            <p>{{ t('messageNotFound') }}</p>
          </template>
        </v-data-table>
        <v-text-field class="custom-error" :rules="checkBoxEx1" :error="true" />

        <v-checkbox
          v-model="form.lineCancel"
          :label="t('checkBoxLineContractCancel')"
          :rules="rulesCallSelected"
        />
        <v-data-table
          :headers="headersUnit"
          :items="listLineCancel"
          :hide-default-footer="true"
          class="elevation-1 ml-3"
        >
          <template v-slot:top="{ pagination, options, updateOptions }">
            <v-data-footer
              :pagination="pagination"
              :options="options"
              :items-per-page-options="[5, 10, 15, 20]"
              @update:options="updateOptions"
            />
          </template>
          <template v-slot:[`item.select_add_phone`]="{ item }">
            <v-checkbox
              v-model="item.select_add_phone"
              :disabled="!form.lineCancel"
              @change="selectDataLineCancel($event, item)"
            />
          </template>
          <template #no-data>
            <p>{{ t('messageNotFound') }}</p>
          </template>
        </v-data-table>
        <v-text-field
          class="custom-error"
          :rules="checkBoxLineContractCancelRules"
          :error="true"
        />
        <v-checkbox
          v-model="form.detourNumber"
          :label="t('checkBoxDetourNumber')"
          :rules="rulesCallSelected"
        />
        <v-row class="ml-3">
          <v-col md="4" cols="12">
            <v-text-field
              v-model="form.detourNumberReg"
              :label="t('detourNumberReg')"
              :disabled="!form.detourNumber"
              :rules="detourNumberRegRules"
              :no-data-text="t('messageNotFound')"
            />
          </v-col>
        </v-row>
        <v-row class="ml-3">
          <v-col md="9" cols="12">
            <v-radio-group v-model="form.detourNumberRadio" row :disabled="!form.detourNumber">
              <v-radio :label="t('signUp')" :value="t('signUp')" />
              <v-radio
                :label="t('numberUpdate')"
                :value="t('numberUpdate')"
                class="pl-15"
              />
              <v-radio
                :label="t('delNumber')"
                :value="t('delNumber')"
                class="pl-15"
              />
            </v-radio-group>
          </v-col>
        </v-row>
        <v-checkbox
          v-model="form.startNumberPort"
          :label="t('checkBoxStartNumPortability')"
          :rules="rulesCallSelected"
        />
        <v-row class="ml-3">
          <v-col md="9" cols="12">
            <v-radio-group
              v-model="form.startNumPortabilityRadio"
              :rules="optionStartNumPortability"
              :disabled="!form.startNumberPort"
              row
            >
              <v-radio :label="t('on')" :value="t('on')" />
              <v-radio :label="t('off')" :value="t('off')" class="pl-15" />
            </v-radio-group>
          </v-col>
        </v-row>
        <v-data-table
          :headers="headersUnit"
          :items="listStartNumberPort"
          :hide-default-footer="true"
          class="elevation-1 ml-3"
        >
          <template v-slot:top="{ pagination, options, updateOptions }">
            <v-data-footer
              :pagination="pagination"
              :options="options"
              :items-per-page-options="[5, 10, 15, 20]"
              @update:options="updateOptions"
            />
          </template>
          <template v-slot:[`item.select_add_phone`]="{ item }">
            <v-checkbox
              v-model="item.select_add_phone"
              :disabled="!form.startNumberPort"
              @change="selectDataStartNumberPort($event, item)"
            />
          </template>
          <template #no-data>
            <p>{{ t('messageNotFound') }}</p>
          </template>
        </v-data-table>
        <v-text-field
          class="custom-error"
          :rules="checkBoxStartNumPortabilityRules"
          :error="true"
        />
      </v-card-text>
      <v-card-actions v-if="standAlone">
        <v-btn @click="close">
          {{ t('btnClose') }}
        </v-btn>
        <v-spacer />
        <v-btn color="primary" :disabled="disabled || !createPermission" @click="confirm">
          {{ t('btnConfirm') }}
        </v-btn>
      </v-card-actions>
    </v-card>
  </v-form>
</template>

<script lang="ts">
import {
  cashRegisterData,
  PhoneNumberData,
  SelectTable,
  ServiceData,
  SoDataCreateState,
  kaiyakuList,
  MakeSoGnoNewRequest,
  kaisenList,
  sipOptionList,
} from '@/models/so-batch';
import {
  TRUE_NUMBER, FALSE_NUMBER, SoCheckBox,
  SO_TAB, SO_MENU, SO_NEW, SO_TAB_AND_NEW,
} from '@/models/so/index';
import { InputValidation } from '@/models/validation';
import {
  conditionRequired,
  required,
  numberSize,
  groupRequired,
} from '@/utils/validation';
import Vue from 'vue';
import { TranslateResult } from 'vue-i18n';
import { DataTableHeader } from 'vuetify';
import {
  FtphoneOptionAPI,
  ResponseData_FTPHONE_OPTION,
} from '@/api/ftphone-option-api';
import {
  getListPhoneByGnoAPI,
  TransferPhoneSearchRequestData,
  getBbJuchuByGnoAPI,
} from '@/api/transfer-phone-api';
import { AxiosError } from 'axios';
import { FukaServiceMstListAPI } from '@/api/fuka-service-mst-api';
import { CONFIRMATION, MESSAGE, SO_KISHU_CD_NOTHING, WHILE_INPUT } from '@/constants/dialog';
import { FLAG_NOT_CREATED, FLAG_CREATED, FLAG_CANCELL_TARGET } from '@/constant/common-constant'
import { DialogOptions } from '@/models/dialog';
import { FormRef } from '@/models/validation';
import { getScreenId } from '@/utils/screenIds';
import { APIResponse } from '@/models/service';
import { getSoCheckBox } from '@/api/so-checkbox-api';

export default Vue.extend({
  props: {
    elevation: {
      type: Number,
      default: 1,
    },
    standAlone: {
      type: Boolean,
      default: true,
    },
    showMainTitle: {
      type: Boolean,
      default: true,
    },
    gnoCode: {
      type: String,
      default: '',
    },
    disabled: {
      type: Boolean,
      default: false,
    },
    mode: {
      type: String,
      default: '',
    }
  },
  data(): SoDataCreateState {
    return {
      form: {
        newContract: false,
        optionAdd: false,
        totalNumberChannel: false,
        totalCH: 0,
        addPhoneNumber: false,
        callSelected: [false, false, false, false, false],
        oldGno: ['', '', ''],
        oldGnoBuffer: ['', '', ''],
        cancelSelected: [false, false],
        unitRadio: 'primary',
        lineCancel: false,
        detourNumber: false,
        detourNumberReg: '',
        detourNumberRadio: 'primary',
        startNumberPort: false,
        startNumPortabilityRadio: '',
      },
      isDirty: false,
      listService: [],
      listPhoneNumber: [],
      listCashRegister: [],
      listUnit: [],
      listLineCancel: [],
      listStartNumberPort: [],
      fukaServiceMstList: [],
      listServiceSelect: [],
      listPhoneSelectAddPhoneNumber: [],
      listPhoneSelectEncounter: [],
      listPhoneSelectLineCancel: [],
      listPhoneSelectStartNumberPort: [],
      createPermission: false,
      kishu_cd:"",
      irregular_flg2:'',
      voip_setchi_date: '',
      kaiyakuList: [],
      valid: false,
      soCheckBox: {} as SoCheckBox,
      screen_id: '',
    };
  },
  computed: {
    headersOptionAdd(): (DataTableHeader | { text: TranslateResult })[] {
      return [
        {
          text: this.t('selectOption'),
          value: 'select_option',
        },
        {
          text: this.t('serviceName'),
          value: 'service_name',
        },
        {
          text: this.t('applicationAcceptanceDate'),
          value: 'moshikomi_uketsuke_date',
        },
        {
          text: this.t('serviceStartDate'),
          value: 'service_start_date',
        },
        {
          text: this.t('serviceEndDate'),
          value: 'service_end_date',
        },
        {
          text: this.t('detourPhoneNumber'),
          value: 'jido_ukai_ashk_tel_no',
        },
      ];
    },
    headersAddPhoneNumber(): (DataTableHeader | { text: TranslateResult })[] {
      return [
        {
          text: this.t('selectAddPhone'),
          value: 'select_add_phone',
        },
        {
          text: this.t('addBillPhoneNumber'),
          value: 'ip_tel_no',
        },
        {
          text: this.t('addPhoneRoutingNumber'),
          value: 'routing_tel_no',
        },
        {
          text: this.t('AddNumberRoutingGKDate'),
          value: 'so_shoki_data_sakusei_date',
        },
        {
          text: this.t('AddPhoneContractCancelAcceptDate'),
          value: 'kaiyaku_uketsuke_date',
        },
      ];
    },
    headersCashRegister(): (DataTableHeader | { text: TranslateResult })[] {
      return [
        {
          text: this.t('changeRegIPPhone'),
          value: 'ip_tel_no',
        },
        {
          text: this.t('changeRegPhoneBusinessPerson'),
          value: 'routing_tel_no',
        },
        {
          text: this.t('changeRegLinkedNumber'),
          value: 'himozuki_tel_no',
        },
        {
          text: this.t('changeRegNumber'),
          value: 'reji_no',
        },
      ];
    },
    headersUnit(): (DataTableHeader | { text: TranslateResult })[] {
      return [
        {
          text: this.t('checkBoxSelectMeeting'),
          value: 'select_add_phone',
        },
        {
          text: this.t('meetBillPhone'),
          value: 'ip_tel_no',
        },
        {
          text: this.t('meetRoutingNumber'),
          value: 'routing_tel_no',
        },
        {
          text: this.t('meetRoutingGKCreateDate'),
          value: 'so_shoki_data_sakusei_date',
        },
        {
          text: this.t('MeetContractCancelAcceptDate'),
          value: 'kaiyaku_uketsuke_date',
        },
      ];
    },
    checkBoxAddOptionRule(): InputValidation {
      const array = this.listService
        .map(({ select_option }) => select_option)
        .filter((item) => !!item);
      return [
        conditionRequired(
          array,
          this.form.optionAdd,
          this.t('checkBoxAddOption'),
          this.t('chooseCheckBoxAddOption')
        ),
      ];
    },
    checkBoxEx1(): InputValidation {
      const array = this.listUnit
        .map(({ select_add_phone }) => select_add_phone)
        .filter((item) => !!item);
      return [
        conditionRequired(
          array,
          this.form.unitRadio === 'test2',
          this.t('lineUnit'),
          this.t('chooseCheckBoxMeeting')
        ),
      ];
    },
    checkBoxDetourNumberRules(): InputValidation {
      const array = this.listUnit
        .map(({ select_add_phone }) => select_add_phone)
        .filter((item) => !!item);
      return [
        conditionRequired(
          array,
          this.form.unitRadio === 'detourNumber',
          this.t('checkBoxAddPhone'),
          this.t('chooseCheckBoxAddPhone')
        ),
      ];
    },
    checkBoxLineContractCancelRules(): InputValidation {
      const array = this.listLineCancel
        .map(({ select_add_phone }) => select_add_phone)
        .filter((item) => !!item);
      return [
        conditionRequired(
          array,
          this.form.lineCancel,
          this.t('checkBoxLineContractCancel'),
          this.t('chooseCheckBoxLineContractCancel')
        ),
      ];
    },
    checkBoxStartNumPortabilityRules(): InputValidation {
      const array = this.listStartNumberPort
        .map(({ select_add_phone }) => select_add_phone)
        .filter((item) => !!item);
      return [
        conditionRequired(
          array,
          this.form.startNumberPort,
          this.t('checkBoxStartNumPortability'),
          this.t('chooseCheckBoxStartNumPortability')
        ),
      ];
    },
    checkBoxChangeNumbersChRules(): InputValidation {
      const array = this.listPhoneNumber
        .map(({ select_add_phone }) => select_add_phone)
        .filter((item) => !!item);
      return [
        conditionRequired(
          array,
          this.form.addPhoneNumber,
          this.t('checkBoxAddPhone'),
          this.t('chooseCheckBoxAddPhone')
        ),
      ];
    },
    rulesCallSelected: {
      cache: false,
      get: function () {
        let len = 0;
        if (this.form.newContract) len += 1; 
        if (this.form.optionAdd) len += 1;
        if (this.form.totalNumberChannel) len += 1;
        if (this.form.addPhoneNumber) len += 1;
        if (this.form.lineCancel) len += 1;
        if (this.form.detourNumber) len += 1;
        if (this.form.startNumberPort) len += 1;
        let result = groupRequired(
            len +
            this.form.callSelected.filter((s: boolean) => s === true).length +
            this.form.cancelSelected.filter((s: boolean) => s === true).length >
            0,
            'チェックボックス'
          )
        return [result]
      },
    },
    gnoRules1(): InputValidation {
      return this.checkOldGno(0)
    },
    gnoRules2(): InputValidation {
      return this.checkOldGno(1)
    },
    gnoRules3(): InputValidation {
      return this.checkOldGno(2)
    },
    detourNumberRegRules(): InputValidation {
      if (this.form.detourNumber) {
        return [
          required(this.form.detourNumberReg, this.t('detourNumberReg')),
          numberSize(this.form.detourNumberReg, 11),
        ];
      }
      return [numberSize(this.form.detourNumberReg, 11)];
    },
    optionStartNumPortability(): InputValidation {
      if (this.form.startNumberPort) {
        return [
          groupRequired(
            this.form.startNumPortabilityRadio,
            this.t('checkBoxStartNumPortability')
          ),
        ];
      }
      return [];
    },
  },
  watch: {
    form: {
      deep: true,
      handler() {
        this.isDirty = true;
      },
    },
  },
  created() {
    switch(this.mode){
      case SO_NEW:
        this.screen_id = getScreenId(this.$route.name, 'SoAdditionDialog')
        break
      case SO_TAB_AND_NEW:
        this.screen_id = getScreenId(this.$route.name, 'SoInformationRegisterView')
        break
      case SO_TAB:
      case SO_MENU:
        this.screen_id = getScreenId(this.$route.name, 'SODataCreationDialog', this.mode)
        break
    }
    this.callApi()
    this.form.unitRadio = String(this.t('gnoUnit'))
  },
  methods: {
    initData(): void {
      // eslint-disable-next-line
      Object.assign(this.$data, (this.$options.data as any).call(this))
    },
    refresh() {
      this.initData()
      this.callApi()
    },
    t(field: string): TranslateResult {
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
    selectDataService(isSelect: boolean, item: ServiceData): void {
      const addPhoneNumber: sipOptionList = {
        sip_option_meisai_no: item.sip_option_meisai_no,
        sip_option_service_cd: item.sip_option_service_cd,
        serviceName: item.service_name,
        jido_ukai_ashk_tel_no: item.jido_ukai_ashk_tel_no,
        jido_ukai_tel_no: item.jido_ukai_tel_no,
      };
      const index = this.listServiceSelect.findIndex(
        ({ sip_option_meisai_no }) =>
          sip_option_meisai_no === item.sip_option_meisai_no
      );
      if (isSelect) {
        this.listServiceSelect.push(addPhoneNumber);
      } else {
        if (index > -1) {
          this.listServiceSelect?.splice(index, 1);
        }
      }
    },
    selectDataAddPhoneNumber(isSelect: boolean, item: PhoneNumberData): void {
      const addPhoneNumber: kaisenList = {
        serial_no: item.serial_no,
        ip_ashk_tel_no: item.ip_ashk_tel_no,
        ip_tel_no: item.ip_tel_no,
      };
      const index = this.listPhoneSelectAddPhoneNumber.findIndex(
        ({ serial_no, ip_ashk_tel_no }) =>
          serial_no === serial_no && ip_ashk_tel_no === item.ip_ashk_tel_no
      );
      if (isSelect) {
        this.listPhoneSelectAddPhoneNumber.push(addPhoneNumber);
      } else {
        if (index > -1) {
          this.listPhoneSelectAddPhoneNumber?.splice(index, 1);
        }
      }
    },
    selectDataEncounter(isSelect: boolean, item: PhoneNumberData): void {
      const addPhoneNumber: kaisenList = {
        serial_no: item.serial_no,
        ip_ashk_tel_no: item.ip_ashk_tel_no,
        ip_tel_no: item.ip_tel_no,
      };
      const index = this.listPhoneSelectEncounter.findIndex(
        ({ serial_no, ip_ashk_tel_no }) =>
          serial_no === serial_no && ip_ashk_tel_no === item.ip_ashk_tel_no
      );
      if (isSelect) {
        this.listPhoneSelectEncounter.push(addPhoneNumber);
      } else {
        if (index > -1) {
          this.listPhoneSelectEncounter?.splice(index, 1);
        }
      }
    },
    selectDataLineCancel(isSelect: boolean, item: PhoneNumberData): void {
      const addPhoneNumber: kaisenList = {
        serial_no: item.serial_no,
        ip_ashk_tel_no: item.ip_ashk_tel_no,
        ip_tel_no: item.ip_tel_no,
      };
      const index = this.listPhoneSelectLineCancel.findIndex(
        ({ serial_no, ip_ashk_tel_no }) =>
          serial_no === serial_no && ip_ashk_tel_no === item.ip_ashk_tel_no
      );
      if (isSelect) {
        this.listPhoneSelectLineCancel.push(addPhoneNumber);
      } else {
        if (index > -1) {
          this.listPhoneSelectLineCancel?.splice(index, 1);
        }
      }
    },
    selectDataStartNumberPort(isSelect: boolean, item: PhoneNumberData): void {
      const addPhoneNumber: kaisenList = {
        serial_no: item.serial_no,
        ip_ashk_tel_no: item.ip_ashk_tel_no,
        ip_tel_no: item.ip_tel_no,
      };
      const index = this.listPhoneSelectStartNumberPort.findIndex(
        ({ serial_no, ip_ashk_tel_no }) =>
          serial_no === serial_no && ip_ashk_tel_no === item.ip_ashk_tel_no
      );
      if (isSelect) {
        this.listPhoneSelectStartNumberPort.push(addPhoneNumber);
      } else {
        if (index > -1) {
          this.listPhoneSelectStartNumberPort?.splice(index, 1);
        }
      }
    },
    // 確認ボタン→親画面のSOデータ作成確認ダイアログ呼び出し機能コール
    confirm(): void {
      if("" === this.kishu_cd || null === this.kishu_cd)
      {
        this.$store.dispatch('dialog/open', {
          type: MESSAGE,
          subType: SO_KISHU_CD_NOTHING,
        } as DialogOptions)
        return
      }
      const requestData: MakeSoGnoNewRequest = {
        options: {
          new: this.form.newContract ? TRUE_NUMBER: FALSE_NUMBER,
          option_add: this.form.optionAdd ? TRUE_NUMBER: FALSE_NUMBER,
          ch_change: this.form.totalNumberChannel ? TRUE_NUMBER: FALSE_NUMBER,
          telno_add: this.form.addPhoneNumber ? TRUE_NUMBER: FALSE_NUMBER,
          stop: this.form.callSelected[0] ? TRUE_NUMBER: FALSE_NUMBER,
          resume: this.form.callSelected[1] ? TRUE_NUMBER: FALSE_NUMBER,
          password_reset: this.form.callSelected[2] ? TRUE_NUMBER: FALSE_NUMBER,
          numbering_notice: this.form.callSelected[3] ? TRUE_NUMBER: FALSE_NUMBER,
          registration_change: this.form.callSelected[4] ? TRUE_NUMBER: FALSE_NUMBER,
          single_cancel: this.form.lineCancel ? TRUE_NUMBER: FALSE_NUMBER,
          deai_gno: this.form.cancelSelected[1] && this.form.unitRadio == this.t('gnoUnit') ? TRUE_NUMBER : FALSE_NUMBER,
          deai_kaisen: this.form.cancelSelected[1] && this.form.unitRadio == this.t('lineUnit') ? TRUE_NUMBER : FALSE_NUMBER,
          all_cancel: this.form.cancelSelected[0] ? TRUE_NUMBER: FALSE_NUMBER,
          detour_telno_new:
            this.form.detourNumber && this.form.detourNumberRadio == this.t('signUp') ? TRUE_NUMBER : FALSE_NUMBER,
          detour_telno_update:
            this.form.detourNumber && this.form.detourNumberRadio == this.t('numberUpdate')
              ? TRUE_NUMBER
              : FALSE_NUMBER,
          detour_telno_delete:
            this.form.detourNumber && this.form.detourNumberRadio == this.t('delNumber') ? TRUE_NUMBER : FALSE_NUMBER,
          nump_on:
            this.form.startNumberPort && this.form.startNumPortabilityRadio == this.t('on') ? TRUE_NUMBER : FALSE_NUMBER,
          nump_off:
            this.form.startNumberPort && this.form.startNumPortabilityRadio == this.t('off') ? TRUE_NUMBER : FALSE_NUMBER,
        },
        target: null,
      };
      const listPhoneChecked: SelectTable = {
        newContract: this.form.newContract,
        listServiceSelect: this.form.optionAdd ? this.listServiceSelect:[],
        sum_of_ch: this.form.totalNumberChannel,
        listPhoneSelectAddPhoneNumber: this.form.addPhoneNumber ? this.listPhoneSelectAddPhoneNumber:[],
        stop_call: this.form.callSelected[0],
        unstop: this.form.callSelected[1],
        resetPassword: this.form.callSelected[2],
        notification: this.form.callSelected[3],
        changeRegiNumber: this.form.callSelected[4],
        ukaiNumberCheck: this.form.detourNumber,
        ukaiNumber: this.form.detourNumberReg,
        ukaiRegType: this.form.detourNumberRadio,
        allCancel: this.form.cancelSelected[0],
        encounter: this.form.cancelSelected[1],
        motoGnos: this.form.oldGno[0] + ' ' + this.form.oldGno[1] + ' ' + this.form.oldGno[2],
        unit: this.form.unitRadio,
        np: this.form.startNumberPort ? this.form.startNumPortabilityRadio:'',
        listPhoneSelectEncounter: this.form.cancelSelected[1] ? this.listPhoneSelectEncounter:[],
        listPhoneSelectLineCancel: this.form.lineCancel ? this.listPhoneSelectLineCancel: [],
        listPhoneSelectStartNumberPort: this.form.startNumberPort ? this.listPhoneSelectStartNumberPort:[],
      };
      requestData.target = [
        {
          koji_yotei_date: '',
          gno: '',
          seq_no: null,
          option_add: [],
          telno_add: [],
          deai_gno: [],
          deai_kaisen: [],
          single_cancel: [],
          detour_telno: this.form.detourNumberReg,
          nump_on: [],
          nump_off: [],
        },
      ];
      const valid = (this.$refs.form as FormRef).validate();
      if (valid) {
        this.$emit('confirm', listPhoneChecked, requestData, this.form);
      }
    },
    // オプション追加リスト
    mapFtPhoneOption(
      ftphoneOptionList: ResponseData_FTPHONE_OPTION[]
    ): ServiceData[] {
      let listFtPhoneOption: ServiceData[] = [];
      for (let ftphoneOption of ftphoneOptionList) {
        const { sip_option_service_cd } = ftphoneOption;
        if ((ftphoneOption.so_data_sakusei_kubun == FLAG_NOT_CREATED && ftphoneOption.so_kaiyaku_data_sakusei_kubun == FLAG_NOT_CREATED)
          || (ftphoneOption.so_data_sakusei_kubun == FLAG_CREATED && ftphoneOption.so_kaiyaku_data_sakusei_kubun == FLAG_CANCELL_TARGET)) {
          // 新規以外の場合、サービス開始日がNULLのレコードはスキップ
          if (this.soCheckBox.new_register_flag != FLAG_CREATED && ftphoneOption.service_start_date == null) {
            continue
          }
          const service: ServiceData = {
            select_option: false,
            sip_option_service_cd: sip_option_service_cd,
            service_name:
              this.fukaServiceMstList.find(
                ({ fuka_service_cd }) => fuka_service_cd === sip_option_service_cd
              )?.fuka_service_mei || sip_option_service_cd,
            sip_option_meisai_no: ftphoneOption.sip_option_meisai_no,
            moshikomi_uketsuke_date: ftphoneOption.moshikomi_uketsuke_date,
            service_start_date: ftphoneOption.service_start_date,
            service_end_date: ftphoneOption.service_end_date,
            jido_ukai_ashk_tel_no: ftphoneOption.jido_ukai_ashk_tel_no,
            jido_ukai_tel_no: ftphoneOption.jido_ukai_tel_no,
          };
          listFtPhoneOption.push(service);
        }
      }
      return listFtPhoneOption;
    },
    // 追加電番リスト
    mapListPhoneNumber(
      bbJuchuMeisaiList: TransferPhoneSearchRequestData[]
    ): PhoneNumberData[] {
      let listBbJuchuMeisai: PhoneNumberData[] = [];
      for (let bbJuchuMeisai of bbJuchuMeisaiList) {
        if (bbJuchuMeisai.so_shoki_data_sakusei_kubun == FLAG_NOT_CREATED && bbJuchuMeisai.so_kaiyaku_data_sakusei_kubun == FLAG_NOT_CREATED) {
          const data: PhoneNumberData = {
            select_add_phone: false,
            serial_no: bbJuchuMeisai.serial_no,
            ip_ashk_tel_no: bbJuchuMeisai.ip_ashk_tel_no,
            ip_tel_no: bbJuchuMeisai.ip_tel_no,
            routing_tel_no:	bbJuchuMeisai.routing_tel_no,
            so_shoki_data_sakusei_date: bbJuchuMeisai.so_shoki_data_sakusei_date,
            kaiyaku_uketsuke_date: bbJuchuMeisai.kaiyaku_uketsuke_date,
          };
          listBbJuchuMeisai.push(data);
        }
      }
      return listBbJuchuMeisai;
    },
    // 出合いリスト
    mapListUnit(
      bbJuchuMeisaiList: TransferPhoneSearchRequestData[]
    ): PhoneNumberData[] {
      let listBbJuchuMeisai: PhoneNumberData[] = [];
      for (let bbJuchuMeisai of bbJuchuMeisaiList) {
        const data: PhoneNumberData = {
          select_add_phone: false,
          serial_no: bbJuchuMeisai.serial_no,
          ip_ashk_tel_no: bbJuchuMeisai.ip_ashk_tel_no,
          ip_tel_no: bbJuchuMeisai.ip_tel_no,
          routing_tel_no:	bbJuchuMeisai.routing_tel_no,
          so_shoki_data_sakusei_date: bbJuchuMeisai.so_shoki_data_sakusei_date,
          kaiyaku_uketsuke_date: bbJuchuMeisai.kaiyaku_uketsuke_date,
        };
        listBbJuchuMeisai.push(data);
      }
      return listBbJuchuMeisai;
    },
    // 回線解約リスト
    mapListLineCancel(
      bbJuchuMeisaiList: TransferPhoneSearchRequestData[]
    ): PhoneNumberData[] {
      let listBbJuchuMeisai: PhoneNumberData[] = [];
      for (let bbJuchuMeisai of bbJuchuMeisaiList) {
        if (bbJuchuMeisai.so_shoki_data_sakusei_kubun == FLAG_CREATED && bbJuchuMeisai.so_kaiyaku_data_sakusei_kubun == FLAG_CANCELL_TARGET) {
          const data: PhoneNumberData = {
            select_add_phone: false,
            serial_no: bbJuchuMeisai.serial_no,
            ip_ashk_tel_no: bbJuchuMeisai.ip_ashk_tel_no,
            ip_tel_no: bbJuchuMeisai.ip_tel_no,
            routing_tel_no:	bbJuchuMeisai.routing_tel_no,
            so_shoki_data_sakusei_date: bbJuchuMeisai.so_shoki_data_sakusei_date,
            kaiyaku_uketsuke_date: bbJuchuMeisai.kaiyaku_uketsuke_date,
          };
          listBbJuchuMeisai.push(data);
        }
      }
      return listBbJuchuMeisai;
    },
    // 番ポ起動リスト
    mapListStartNumberPort(
      bbJuchuMeisaiList: TransferPhoneSearchRequestData[]
    ): PhoneNumberData[] {
      let listBbJuchuMeisai: PhoneNumberData[] = [];
      for (let bbJuchuMeisai of bbJuchuMeisaiList) {
        if (bbJuchuMeisai.ip_tel_no !== bbJuchuMeisai.routing_tel_no){
          const data: PhoneNumberData = {
            select_add_phone: false,
            serial_no: bbJuchuMeisai.serial_no,
            ip_ashk_tel_no: bbJuchuMeisai.ip_ashk_tel_no,
            ip_tel_no: bbJuchuMeisai.ip_tel_no,
            routing_tel_no:	bbJuchuMeisai.routing_tel_no,
            so_shoki_data_sakusei_date: bbJuchuMeisai.so_shoki_data_sakusei_date,
            kaiyaku_uketsuke_date: bbJuchuMeisai.kaiyaku_uketsuke_date,
          };
          listBbJuchuMeisai.push(data);
        }
      }
      return listBbJuchuMeisai;
    },
    // レジ番変更
    mapListCashRegister(
      bbJuchuMeisaiList: TransferPhoneSearchRequestData[]
    ): cashRegisterData[] {
      let listBbJuchuMeisai: cashRegisterData[] = [];
      for (let bbJuchuMeisai of bbJuchuMeisaiList) {
        const data: cashRegisterData = {
          ip_ashk_tel_no: bbJuchuMeisai.ip_ashk_tel_no,
          ip_tel_no: bbJuchuMeisai.ip_tel_no,
          routing_tel_no:	bbJuchuMeisai.routing_tel_no,
          himozuki_tel_no: bbJuchuMeisai.himozuki_tel_no,
          reji_no: bbJuchuMeisai.reji_no,
        };
        listBbJuchuMeisai.push(data);
      }
      return listBbJuchuMeisai;
    },
    // 解約済み電番リスト生成
    getKaiyakuList(bbJuchuMeisaiList: TransferPhoneSearchRequestData[]): kaiyakuList[] {
      let res:kaiyakuList[] = []
      bbJuchuMeisaiList.forEach((element) => {
        if (element.so_kaiyaku_data_sakusei_kubun == '1') {
          let tempItem:kaiyakuList = {
            ip_tel_no: element.ip_tel_no,
            so_kaiyaku_data_sakusei_date: element.so_kaiyaku_data_sakusei_date   
          }
          res.push(tempItem)
        }
      })
      return res
    },
    // 旧GNO入力時、対象の電番を取得する
    inputOldGno(): void {

      // 前回検索結果と同じ場合は何もせず終了する
      if(this.form.oldGno.toString() === this.form.oldGnoBuffer.toString()) return

      let tempList:PhoneNumberData[] = []
      let gnoList:string[] = []

      // 検索済み結果バッファクリア
      this.form.oldGnoBuffer.splice(0,this.form.oldGnoBuffer.length)
      this.form.oldGnoBuffer = this.form.oldGno.concat() // 現状の入力内容を複製

      let promiseList: APIResponse<TransferPhoneSearchRequestData[]>[] = []
      // 電番クリア
      this.listUnit.splice(0,this.listUnit.length)
      // 選択済みリストをクリア
      this.listPhoneSelectEncounter = []

      this.form.oldGno.forEach((element) => {
        // 空白ではない、且つ既に検索対象ではない（同一GNOは重複して検索しない）
        if (element && element !== "" && !gnoList.includes(element)) {
          // 有効なGNO以外は検索しない
          // （変化でバックエンドへ要求するため、バリデーションチェックが実施されずに実行されるので、ここでもチェックしする）
          if (true === numberSize(element,8) && true === this.checkSameGno(element))
          {
            gnoList.push(element)
            promiseList.push(getListPhoneByGnoAPI(this.screen_id, element))
          }
        }
      })
      if(promiseList.length > 0)
      {
        Promise.all(promiseList)
          .then((res) => {
            res.forEach((item)=>{
              tempList = tempList.concat(this.mapListUnit(item.data));
            })
            this.listUnit = tempList
          })
      }
    },
    checkOldGno(index:number): InputValidation
    {
      if (!this.form.cancelSelected[1])
      {
        // チェックボックスに入力がない場合はなにもしない
        return []
      }
      if (index === 0) {
        return [
          required(this.form.oldGno[index], this.t('oldGNO')),
          numberSize(this.form.oldGno[index], 8),
          this.checkSameGno(this.form.oldGno[index])
          ];
      }
      return [
        numberSize(this.form.oldGno[index], 8),
        this.checkSameGno(this.form.oldGno[index])
        ];
    },
    checkSameGno(gno:string):boolean | TranslateResult
    {
      if(gno!==this.gnoCode) return true
      return false || this.t('messageSameGno')
    },
    controlCheckBox(): void {
      Object.entries(this.soCheckBox).forEach((element) => {
        if (element[1] == '1') {
          if (element[0] == 'new_register_flag') {
            this.form.newContract = true
          }
          if (element[0] == 'op_add_flag') {
            if(this.listService.length > 0){
              this.form.optionAdd = true
              this.listService.forEach((elm) => {
                elm.select_option = true
                this.selectDataService(true, elm)
              })
            }
          }
          if (element[0] == 'allch_change_flag') {
            this.form.totalNumberChannel = true
          }
          if (element[0] == 'add_telno_flag') {
            if(this.listPhoneNumber.length > 0){
              this.form.addPhoneNumber = true
              this.listPhoneNumber.forEach((elm) => {
                elm.select_add_phone = true
                this.selectDataAddPhoneNumber(true, elm)
              })
            }
          }
          if (element[0] == 'ch_regnum_flag') {
            this.form.callSelected[4] = true
          }
          if (element[0] == 'all_cancel_flag') {
            this.form.cancelSelected[0] = true
          }
          if (element[0] == 'encounter_flag') {
            this.form.cancelSelected[1] = true
          }
          if (element[0] == 'line_cancel_flag') {
            this.form.lineCancel = true
            this.listLineCancel.forEach((elm) => {
              elm.select_add_phone = true
              this.selectDataLineCancel(true, elm)
            })
          }
        }
      })
      // 一緒にチェックできないチェックボックスは外す
      // 新規のチェックがONの場合は「全解約、回線解約、レジ番」をOFFにする
      if (this.form.newContract == true) {
        this.form.cancelSelected[0] = false
        this.form.lineCancel = false
        this.form.callSelected[4] = false
      } else if (this.form.cancelSelected[1] == true) {
        // 出合いのチェックがONの場合はその他のチェックボックスを外す
        this.form.optionAdd = false
        this.form.totalNumberChannel = false
        this.form.addPhoneNumber = false
        this.form.callSelected[4] = false
        this.form.cancelSelected[0] = false
        this.form.lineCancel = false
      } else if (this.form.cancelSelected[0] == true) {
        // 全解約にチェックがついている場合はその他のチェックボックスを外す
        this.form.optionAdd = false
        this.form.totalNumberChannel = false
        this.form.addPhoneNumber = false
        this.form.callSelected[4] = false
        this.form.lineCancel = false
      }
    },
    callApi() {
      this.createPermission = this.$$hasCreatePermission(this.screen_id);
      // 選択済みリストの初期化
      this.listServiceSelect = []
      this.listPhoneSelectAddPhoneNumber = []
      this.listPhoneSelectEncounter = []
      this.listPhoneSelectLineCancel = []
      this.listPhoneSelectStartNumberPort = []

      if (this.gnoCode) {
        this.$$openLoadingScreen() // Loading処理の呼び出し
        Promise.all([
          FukaServiceMstListAPI(this.screen_id),
          FtphoneOptionAPI(this.screen_id, this.gnoCode),
          getListPhoneByGnoAPI(this.screen_id, this.gnoCode),
          getBbJuchuByGnoAPI(this.screen_id, this.gnoCode),
          getSoCheckBox(this.screen_id, this.gnoCode)
        ])
          .then((res) => {
            console.log(res[3])
            this.soCheckBox = res[4].data;
            this.fukaServiceMstList = res[0].data;
            this.listService = this.mapFtPhoneOption(res[1].data);
            this.listPhoneNumber = this.mapListPhoneNumber(res[2].data);
            this.listCashRegister = this.mapListCashRegister(res[2].data);
            this.listLineCancel = this.mapListLineCancel(res[2].data);
            this.listStartNumberPort = this.mapListStartNumberPort(res[2].data);
            this.kaiyakuList = this.getKaiyakuList(res[2].data);
            this.controlCheckBox()
            this.form.oldGno[0] = res[3].data.moto_gno;
            this.listUnit = []
            this.inputOldGno()
            this.form.totalCH = res[3].data.sou_ch_num;
            this.kishu_cd = res[3].data.kishu_cd;
            this.irregular_flg2 = res[3].data.irregular_flg2
            this.voip_setchi_date = res[3].data.voip_setchi_date
          })
          .catch((err: AxiosError) => {
            console.log(err.message);
          })
          .finally(()=>{
            this.$$closeLoadingScreen() // Loading処理の解除
            this.isDirty = false;
          });
      } else {
        this.fukaServiceMstList = [];
        this.listService = [];
        this.listPhoneNumber = [];
        this.listCashRegister = [];
        this.listLineCancel = [];
        this.listStartNumberPort = [];
        this.controlCheckBox()
        this.form.oldGno[0] = '';
        this.form.totalCH = 0;
        this.listUnit = [];
        this.isDirty = false;
      }
    }
  },
});
</script>

<style>
.custom-error .v-input__slot {
  display: none;
}
</style>
