<!--サービス情報登録_案件明細情報-->
<template>
  <div>
    <div class="mb-3">
      <h4>{{ t('projectDetail') }}</h4>
    </div>
    <v-form ref="form" v-model="valid" :disabled="cancelStatus || !updatePermission || !createPermission">
      <v-row align="center">
        <v-col cols="12" md="3">
          <v-tooltip :disabled="!service_name" bottom>
            <template v-slot:activator="{ on, attrs }">
              <div v-on="on">
                <v-select
                  class="required-label"
                  :items="optionsAppCode"
                  :label="t('serviceApplicationCode')"
                  :rules="serviceApplicationCodeRule"
                  :value="customerData.service_cd"
                  :loading="loadingOptions"
                  :readonly="(serviceCdBefore=='6'||serviceCdBefore=='8')||!customerData.service_code_correction"
                  :filled="(serviceCdBefore=='6'||serviceCdBefore=='8')||!customerData.service_code_correction"
                  :menu-props="{ 'content-class': 'hide-disabled' }"
                  :no-data-text="t('messages.notFound')"
                  v-bind="attrs"
                  @input="emit('service_cd', $event)"
                  @change="filterRyokinPlan()"
                />
              </div>
            </template>
            <span>{{ service_name }}</span>
          </v-tooltip>
        </v-col>
        <v-col cols="12" md="3">
          <v-checkbox
            :label="t('serviceCodeCorrection')"
            :value="customerData.service_code_correction"
            @change="emit('service_code_correction', $event ? true : false)"
          />
        </v-col>
        <v-col cols="12" md="3">
          <v-text-field
            class="required-label"
            :label="t('totalCHNumber')"
            :rules="totalCHNumberRule"
            :filled="true"
            :readonly="true"
            :value="customerData.sou_ch_num"
            @input="emit('sou_ch_num', $event)"
          />
        </v-col>
        <v-col cols="12" md="3" class="d-flex align-start">
          <v-checkbox
            :value="customerData.boards_number_editable"
            :disabled="boardCheckBoxDisable"
            @change="emit('boards_number_editable', $event || false)"
          />
          <v-text-field
            :value="boardNumberValue"
            :label="t('boardsNumber')"
            :filled="!customerData.boards_number_editable"
            :readonly="!customerData.boards_number_editable"
            :rules="boardsNumberRule"
            :error-messages="boardNumberWarning"
            @input="emit('board_num', $event)"
            @blur="blurBoardNum(boardNumberValue)"
          />
        </v-col>
      </v-row>
      <v-row align="center">
        <v-col cols="12" md="3">
          <v-text-field
            :value="customerData.oa_juchu_no"
            :label="t('phoneConstructionOrderNo')"
            :rules="phoneConstructionOrderNoRule"
            @input="emit('oa_juchu_no', $event)"
          />
        </v-col>
        <v-col cols="12" md="3">
          <v-text-field
            :value="customerData.oa_juchu_setchi_no"
            :rules="oaOrderInstallationNoRule"
            :label="''"
            @input="emit('oa_juchu_setchi_no', $event)"
          />
        </v-col>
      </v-row>
      <v-row align="center">
        <v-col cols="12" md="3" class="d-flex align-start">
          <v-checkbox
            v-model="kishu_cd_editable"
          />
          <v-combobox
            :value="customerData.kishu_cd"
            :label="t('modelId')"
            :items="optionsModel"
            :rules="modelIdRule"
            :filled="!kishu_cd_editable"
            :readonly="!kishu_cd_editable"
            autocomplete="auto-off"
            @input="emit('kishu_cd', $event.value)"
          />
        </v-col>
        <v-col cols="12" md="3">
          <v-tooltip :disabled="!kishu_name" bottom>
            <template v-slot:activator="{ on, attrs }">
              <v-text-field
                :value="kishu_name"
                :label="t('modelName')"
                filled
                readonly
                v-bind="attrs"
                v-on="on"
                @input="emit('gno', $event)"
              />
            </template>
            <span>{{ kishu_name }}</span>
          </v-tooltip>
        </v-col>
        <v-col cols="12" md="3" class="d-flex align-start">
          <v-checkbox
            v-model="norikae_moto_service_editable"
          />
          <v-select
            :value="customerData.norikae_moto_service"
            :items="optionsTransferSourceService"
            :label="t('transferSourceService')"
            :no-data-text="t('messages.notFound')"
            :filled="!norikae_moto_service_editable"
            :readonly="!norikae_moto_service_editable"
            @input="emit('norikae_moto_service', $event)"
          />
        </v-col>
        <v-col cols="12" md="3" class="d-flex align-start">
          <v-checkbox
            v-model="moto_gno_editable"
          />
          <v-text-field
            :value="customerData.moto_gno"
            :label="t('transferSourceGNO')"
            :rules="transferSourceGNORule"
            :filled="!moto_gno_editable"
            :readonly="!moto_gno_editable"
            @input="emit('moto_gno', $event)"
          />
        </v-col>
      </v-row>
      <v-row align="center">
        <v-col cols="12" md="3">
          <v-text-field
            :value="customerData.oa_juchu_no2"
            :label="t('voIPConstructionOrderNO')"
            :rules="voIPConstructionOrderNORule"
            @input="emit('oa_juchu_no2', $event)"
          />
        </v-col>
        <v-col cols="12" md="3">
          <v-text-field
            :value="customerData.oa_juchu_setchi_no2"
            :rules="oaOrderInstallationRule"
            @input="emit('oa_juchu_setchi_no2', $event)"
          />
        </v-col>
      </v-row>
      <v-row align="center">
        <v-col cols="12" md="3" class="d-flex align-start">
          <v-checkbox
            v-model="ryokin_plan_kubun_editable"
          />
          <v-select
            class="required-label"
            :items="ryokin_plan_kubun"
            :label="t('paymentPlan')"
            :rules="paymentPlanRule"
            :value="customerData.ryokin_plan_kubun"
            :loading="loadingOptions"
            :menu-props="{ 'content-class': 'hide-disabled' }"
            :no-data-text="t('messages.notFound')"
            :filled="!ryokin_plan_kubun_editable"
            :readonly="!ryokin_plan_kubun_editable"
            @input="emit('ryokin_plan_kubun', $event)"
          />
        </v-col>
        <v-col cols="12" md="3" class="d-flex align-start">
          <v-checkbox
            v-model="ryokin_plan_op_kubun_editable"
          />
          <v-select
            :value="customerData.ryokin_plan_op_kubun"
            :items="ryokin_plan_op_kubun"
            :label="t('paymentPlanOption')"
            :loading="loadingOptions"
            :no-data-text="t('messages.notFound')"
            :filled="!ryokin_plan_op_kubun_editable"
            :readonly="!ryokin_plan_op_kubun_editable"
            @input="emit('ryokin_plan_op_kubun', $event)"
          />
        </v-col>
      </v-row>
      <v-row align="center">
        <v-col cols="12" md="3">
          <v-select
            class="required-label"
            :items="mosikomi_kubun"
            :label="t('newExistingSection')"
            :rules="newExistingSectionRule"
            :value="customerData.flets_sinki_kison_flg"
            :loading="loadingOptions"
            :no-data-text="t('messages.notFound')"
            :filled="isNewExistingSectionReadOnly"
            :readonly="isNewExistingSectionReadOnly"
            @input="emit('flets_sinki_kison_flg', $event)"
          />
        </v-col>
        <v-col cols="12" md="3">
          <v-text-field
            :value="customerData.dotsu_yotei_date"
            :label="t('openConnectionScheduledDate')"
            :rules="openConnectionScheduledDateRule"
            filled
            readonly
            @input="emit('dotsu_yotei_date', $event)"
          />
        </v-col>
        <v-col cols="12" md="3">
          <v-text-field
            :value="customerData.dotsu_kanryo_date"
            :label="t('openConnectionCompletionDate')"
            :rules="openConnectionCompletionDateRule"
            filled
            readonly
            @input="emit('dotsu_kanryo_date', $event)"
          />
        </v-col>
        <v-col cols="12" md="3">
          <v-text-field
            :value="customerData.voip_setchi_date"
            :label="t('voIPInstallationDate')"
            :rules="voIPInstallationDateRule"
            filled
            readonly
            @input="emit('voip_setchi_date', $event)"
          />
        </v-col>
      </v-row>
      <v-row align="center">
        <v-col cols="12" md="3" class="d-flex align-start">
          <v-checkbox
            v-if="isPmRole && customerData.deai_status_kubun !== '2'"
            v-model="deai_status_kubun_editable"
          />
          <v-select
            :value="customerData.deai_status_kubun"
            :items="deai_status_kubun"
            :label="t('meetingStatus')"
            :loading="loadingOptions"
            :no-data-text="t('messages.notFound')"
            :filled="!deai_status_kubun_editable"
            :readonly="!deai_status_kubun_editable"
            @input="emit('deai_status_kubun', $event)"
          />
        </v-col>
      </v-row>
      <v-row align="center">
        <v-col cols="12" md="3" class="d-flex align-start">
          <v-checkbox
            v-model="shusochi_address_editable"
          />
          <v-text-field
            :value="customerData.shusochi_address"
            :label="t('IPAddress')"
            :rules="ipAddressRule"
            :filled="!shusochi_address_editable"
            :readonly="!shusochi_address_editable"
            @input="emit('shusochi_address', $event)"
          />
        </v-col>
        <v-col cols="12" md="3">
          <v-text-field
            :value="customerData.user_id"
            :label="t('id')"
            :rules="idRule"
            filled
            readonly
            @input="emit('user_id', $event)"
          />
        </v-col>
        <v-col cols="12" md="3">
          <v-text-field
            :value="customerData.password"
            :label="t('pass')"
            :rules="passRule"
            filled
            readonly
            @input="emit('password', $event)"
          />
        </v-col>
      </v-row>
      <v-row align="center">
        <v-col cols="12" md="3">
          <v-text-field
            :value="customerData.sub_domain"
            :label="t('subdomain')"
            :rules="subdomainRule"
            filled
            readonly
            @input="emit('sub_domain', $event)"
          />
        </v-col>
      </v-row>
      <v-row align="center">
        <v-col cols="12" md="3">
          <v-select
            :value="customerData.kaisen_tehai_kubun"
            :items="kaisen_tehai_kubun"
            :label="t('lineArrangementSection')"
            :loading="loadingOptions"
            :no-data-text="t('messages.notFound')"
            @input="emit('kaisen_tehai_kubun', $event)"
          />
        </v-col>
        <v-col cols="12" md="3">
          <v-select
            :value="customerData.tesuryo_kubun"
            :items="tesuryo_kubun"
            :loading="loadingOptions"
            :label="t('feeSection')"
            :no-data-text="t('messages.notFound')"
            @input="emit('tesuryo_kubun', $event)"
          />
        </v-col>
        <v-col cols="12" md="3">
          <v-text-field
            :value="customerData.tesuryo_ryokin"
            :label="t('feeAmount')"
            :rules="feeAmountRule"
            @input="emit('tesuryo_ryokin', $event)"
          />
        </v-col>
        <v-col cols="12" md="3">
          <v-text-field
            :value="customerData.tesuryo_juchu_no"
            :label="t('feeOrderNO')"
            :rules="feeOrderNORule"
            @input="emit('tesuryo_juchu_no', $event)"
          />
        </v-col>
      </v-row>
      <v-row align="center">
        <v-col cols="12" md="3">
          <app-date-picker
            :value="customerData.tesuryo_nohin_date"
            :label="t('feeDeliveryDate')"
            :rules="feeDeliveryDateRule"
            @input="emit('tesuryo_nohin_date', $event)"
          />
        </v-col>
        <v-col cols="12" md="3">
          <app-date-picker
            :value="customerData.tesuryo_seiq_date"
            :label="t('feeBillingDate')"
            :rules="feeBillingDateRule"
            @input="emit('tesuryo_seiq_date', $event)"
          />
        </v-col>
        <v-col cols="12" md="3">
          <v-select
            :value="customerData.downgrade_flg"
            :items="downgrade_flg"
            :label="t('downgrade')"
            :loading="loadingOptions"
            :no-data-text="t('messages.notFound')"
            @input="emit('downgrade_flg', $event)"
          />
        </v-col>
        <v-col cols="12" md="3">
          <v-text-field
            :value="customerData.downgrade_ryokin"
            :label="t('downgradeMoneyAmount')"
            maxlength="8"
            @input="emit('downgrade_ryokin', $event)"
          />
        </v-col>
      </v-row>
      <v-row align="center">
        <v-col cols="12" md="3">
          <v-text-field
            :value="get_tuti_kubun(customerData.tuti_hakko_kubun)"
            :label="t('customerNotificationDocument')"
            filled
            readonly
            @input="emit('tuti_hakko_kubun', $event)"
          />
        </v-col>
        <v-col cols="12" md="3">
          <v-text-field
            :value="customerData.tuti_hakko_date"
            :label="t('customerNotificationDocumentIssueDate')"
            filled
            readonly
            @input="
              emit('tuti_hakko_date', $event)
            "
          />
        </v-col>
        <v-col cols="12" md="3">
          <v-text-field
            :value="get_kaisen_kubun(customerData.kaisen_shiji_hakko_kubun)"
            :label="t('lineSettingInstructionDocumentIssueDate')"
            filled
            readonly
            @input="
              emit('kaisen_shiji_hakko_kubun', $event)
            "
          />
        </v-col>
        <v-col cols="12" md="3">
          <v-text-field
            :value="customerData.kaisen_shiji_hakko_date"
            :label="t('lineSettingDocumentIssueDate')"
            filled
            readonly
            :rules="lineSettingDocumentIssueDateRule"
            @input="emit('kaisen_shiji_hakko_date', $event)"
          />
        </v-col>
      </v-row>
    </v-form>
    <v-form ref="form" v-model="valid">
      <v-row align="center">
        <v-col cols="12">
          <v-textarea
            outlined
            :value="customerData.biko"
            :label="t('note')"
            rows="2"
            :rules="noteRule"
            @input="emit('biko', $event)"
          />
        </v-col>
      </v-row>
      <v-row align="center">
        <v-col cols="12">
          <v-textarea
            v-if="dairitenIsFT"
            outlined
            :value="customerData.naibu_memo"
            :label="t('internalSharedMemo')"
            :rules="internalSharedMemoRule"
            rows="2"
            @input="emit('naibu_memo', $event)"
          />
        </v-col>
      </v-row>
    </v-form>
  </div>
</template>

<script lang="ts">
import AppDatePicker from '../UI/AppDatePicker.vue';
import Vue, { PropType } from 'vue';
import store from '@/store/index'
import {
  DetailCustomerState,
  DetailFormCustomerData,
  SelectOption,
  SelectOptionKishuMst,
  SelectOptionServiceCd,
  PhoneNumberResponse,
} from '@/models/customer';
import { TranslateResult } from 'vue-i18n';
import {
  bufferSize,
  dateRange,
  formatDate,
  formatIpAddress,
  numberSize,
  required,
} from '@/utils/validation';
import { InputValidation } from '@/models/validation';
import { KubunListAPI } from '@/api/kubun-kanri';
import { ResponseData_KUBUN_KANRI } from '@/models/service';
import { DairitenResultData } from '@/api/auth-api';
import { getScreenId } from '@/utils/screenIds';
import { AUTO_CALC_ILLREGULAR_FLG, DIVISION_IN_CALC_BOARD_NUM, ADDITION_IN_CALC_BOARD_NUM } from '@/models/service-register'
import { FLAG_UPDATED } from '@/constant/common-constant';
import {
  DEAI_STATUS_KUBUN_NORMAL,
  DEAI_STATUS_KUBUN_DEAI,
  DEAI_STATUS_KUBUN_REGISTED
} from '@/models/display-matter-details';

export default Vue.extend({
  name: 'ServiceRegisterView',
  components: { AppDatePicker },
  props: {
    customerData: {
      type: Object as PropType<DetailFormCustomerData>,
      default: null,
    },
    listPhoneNumber: {
      type: Array as PropType<PhoneNumberResponse[]>,
      default: null,
    },
    optionsAppCode: {
      type: Array as PropType<SelectOptionServiceCd[]>,
      default: [],
    },
    optionsModel: {
      type: Array as PropType<SelectOptionKishuMst[]>,
      default: [],
    },
    ryokinPlanKubunList: {
      type: Array as PropType<ResponseData_KUBUN_KANRI[]>,
      default: [],
    },
    initialCustomerData: {
      type: Object,
      default: () => ({}),
    },
    sumOfCh:{
      type: Number,
      default: null,
    },
    cancelStatus:{
      type: Boolean,
      default: false,
    },
    // 新規（BB受注が存在しない）場合: true
    newContract: {
      type: Boolean,
      default: false,
    },
    ryokinPlanBefore: {
      type: String,
      default: '',
    },
    updatePermission: {
      type: Boolean,
      default: false,
    },
    createPermission: {
      type: Boolean,
      default: false,
    }
  },
  data(): DetailCustomerState {
    return {
      optionsTransferSourceService: [],
      boardsNumberRule: [(value) => numberSize(value, 1)],
      phoneConstructionOrderNoRule: [(value) => bufferSize(value, 8)],
      oaOrderInstallationNoRule: [(value) => numberSize(value, 1)],
      modelIdRule: [(value) => bufferSize(value, 10)],
      transferSourceGNORule: [(value) => numberSize(value, 8)],
      voIPConstructionOrderNORule: [(value) => bufferSize(value, 8)],
      oaOrderInstallationRule: [(value) => numberSize(value, 1)],
      openConnectionScheduledDateRule: [(value) => formatDate(value)],
      openConnectionCompletionDateRule: [(value) => formatDate(value)],
      voIPInstallationDateRule: [(value) => formatDate(value)],
      ipAddressRule: [(value) => formatIpAddress(value)],
      idRule: [(value) => bufferSize(value, 30)],
      passRule: [(value) => bufferSize(value, 30)],
      subdomainRule: [(value) => bufferSize(value, 50)],
      feeAmountRule: [(value) => numberSize(value, 8)],
      feeOrderNORule: [(value) => numberSize(value, 8)],
      feeDeliveryDateRule: [(value) => formatDate(value)],
      lineSettingDocumentIssueDateRule: [(value) => formatDate(value)],
      noteRule: [(value) => bufferSize(value, 128)],
      internalSharedMemoRule: [(value) => bufferSize(value, 128)],
      loadingOptions: false,
      ryokin_plan_kubun: [],
      ryokin_plan_op_kubun: [],
      mosikomi_kubun: [],
      deai_status_kubun: [],
      kaisen_tehai_kubun: [],
      tesuryo_kubun: [],
      downgrade_flg: [],
      tuti_hakko_kubun: [],
      kaisen_shiji_hakko_kubun: [],
      serviceCdBefore: this.customerData.service_cd,
      correctServiceCd: this.customerData.service_code_correction,
      dairitenIsFT: false,
      kishu_name: '',
      max_board_num: null,
      kishu_cd_before: null,
      ip_type: '',
      valid: false,
      board_num_before: '',
      set_board_num: '',
      sou_ch_num_before: '',
      kishu_cd_editable: false,
      norikae_moto_service_editable: false,
      moto_gno_editable: false,
      ryokin_plan_kubun_editable: false,
      ryokin_plan_op_kubun_editable: false,
      deai_status_kubun_editable: false,
      shusochi_address_editable: false,
      service_name: '',
    };
  },
  computed: {
    isPmRole(): boolean {
      if (store.getters['auth/pmRoleFlg'] === true ) {
        return true
      }
      return false
    },
    boardNumberValue() {
      if (!this.customerData.kishu_cd && this.customerData.board_num === '0') {
        return 0
      } else {
        return this.customerData.board_num
      }
    },
    serviceApplicationCodeRule(): InputValidation {
      let optionsAppCode = this.optionsAppCode.filter((item) => item.value === this.customerData.service_cd)
      console.log(optionsAppCode)
      // 選択されている申込サービスコードの可否を判定
      if (optionsAppCode.length <= 0 || optionsAppCode[0].disabled == true) {
        return [false]
      }
      return[
        required(this.customerData.service_cd, this.$t('customerDetail.serviceRegister.serviceApplicationCode')),
        ]
    },
    feeBillingDateRule(): InputValidation {
      return [
        formatDate(this.customerData.tesuryo_seiq_date),
        dateRange(
          this.customerData.tesuryo_seiq_date,
          this.customerData.tesuryo_nohin_date,
          null
        ),
      ];
    },
    totalCHNumberRule(): InputValidation {
      return [
        required(
          this.customerData.sou_ch_num,
          this.$t('customerDetail.serviceRegister.totalCHNumber')
        ),
        numberSize(this.customerData.sou_ch_num, 3),
      ];
    },
    paymentPlanRule(): InputValidation {
      let ryokin_plan = this.ryokin_plan_kubun.filter((item) => item.value === this.customerData.ryokin_plan_kubun)
      // 選択されている料金プラン区分の可否を判定
      if (ryokin_plan.length <= 0 || ryokin_plan[0].disabled == true) {
        return [false]
      }
      return [
        required(
          this.customerData.ryokin_plan_kubun,
          this.$t('customerDetail.serviceRegister.paymentPlan')
        ),
      ];
    },
    newExistingSectionRule(): InputValidation {
      let mosikomi_kubun = this.mosikomi_kubun.filter((item) => item.value === this.customerData.flets_sinki_kison_flg)
      // 選択されている新規既存区分の可否を判定
      if (mosikomi_kubun.length <= 0 || mosikomi_kubun[0].disabled == true) {
        return [false]
      }
      return [
        required(
          this.customerData.flets_sinki_kison_flg,
          this.$t('customerDetail.serviceRegister.newExistingSection')
        ),
      ];
    },
    isNewExistingSectionReadOnly(): boolean {
      let mosikomi_kubun = this.mosikomi_kubun.filter((item) => item.value === this.customerData.flets_sinki_kison_flg)
      // 選択されている新規既存区分の可否を判定
      if (mosikomi_kubun.length <= 0 || mosikomi_kubun[0].disabled == true) {
        return false
      }
      return true
    },
    boardNumberWarning() {
      // 電番の数がボード枚数以上の場合メッセージを表示
      if (Number(this.customerData.board_num) > this.listPhoneNumber.length) {
        return this.$t('customerDetail.serviceRegister.boardsNumberWarning')
      } else if (this.listPhoneNumber.filter((item) => (item.reji_no)).length !== Number(this.customerData.board_num)) {
        // レジ番が設定されている電番の数とボード枚数が一致していなければメッセージ表示
        return this.$t('customerDetail.serviceRegister.boardsNumberRejiWarning')
      } else {
        return ''
      }
    },
    boardNumberAutoCalc() {
      if (!(this.customerData.kishu_cd == null || this.customerData.kishu_cd == '')
        && AUTO_CALC_ILLREGULAR_FLG.includes(this.customerData.irregular_flg)
        && AUTO_CALC_ILLREGULAR_FLG.includes(this.customerData.irregular_flg2)) {
          console.log(true)
          return true
      }
      return false
    },
    boardCheckBoxDisable(): boolean {
      // 最大ボード枚数が0の場合はチェックを外せない
      if (this.max_board_num == 0) return true
      return false
    }
  },
  watch:{
    sumOfCh:{
      handler(value){
        this.customerData.sou_ch_num = value;
      }
    },
    // 機種プルダウン変更時
    'customerData.kishu_cd'(key, old_key) {
      if(null === this.kishu_cd_before)
      {
        console.log("以前の機種セット," + key)
        this.kishu_cd_before = key
      }
      this.onChangeModelId(key);
      // 機種コードが変わった時にボード枚数計算
      // 再度登録されている機種に戻したときも自動計算する
      if(key !== this.kishu_cd_before || (key === this.kishu_cd_before && old_key !== '')){
        if (this.max_board_num == 0) {
          // 機種マスタのmax_board_numが0の場合は自動計算しない
          return
        }
        this.customerData.board_num = this.calcBoardNum();
      }
    },
    // 総CH変更時
    'customerData.sou_ch_num'(value){
      // 変更前総CH数取得
      if(this.sou_ch_num_before === ''){
        this.sou_ch_num_before = value
      }
      // 総CH数に変更があった場合にボード枚数計算
      // valueは数値になることがあるため等価比較
      if(this.sou_ch_num_before != value){
        this.customerData.board_num = this.calcBoardNum();
      }
    },
    // 料金プランプルダウン変更時
    'customerData.ryokin_plan_kubun'(){
      this.filterRyokinPlan();
    },
    // 出会いステータスプルダウン変更時
    'customerData.deai_status_kubun'(){
      this.filterRyokinPlan();
      this.deai_status_kubun = this.deaiStatusKubunToSelect();
    },
    // 新規（BB受注が存在しない）変更時（親が非同期で情報を取得するため）
    'newContract'(){
      this.filterRyokinPlan();
    },
    // 機種リスト取得時
    'optionsModel'(){
      console.log("watch-optionsModel")
      this.onChangeModelId(this.customerData.kishu_cd)
    },
    // 料金プラン区分リスト取得時
    'ryokinPlanKubunList'(value){
      console.log("watch-ryokinPlanKubunList")
      this.ryokin_plan_kubun = this.kubunToSelect(value)
      this.filterRyokinPlan()
    },
    // 変更前サービスコード取得
    'customerData.service_cd'(value){
      if(this.serviceCdBefore === ''){
        this.serviceCdBefore = value
      }
    },
    'customerData.service_code_correction'(value){
      // チェックボックスが外れた時は変更前のサービスコードに戻す
      if(!value){
        this.customerData.service_cd = this.serviceCdBefore
      }
    },
    // ボード枚数取得
    'customerData.board_num'(value){
      // 表示上のボード枚数はboard_numと同期
      this.set_board_num = value
      // 変更前のボード枚数
      if(this.board_num_before === ''){
        this.board_num_before = value
      }
    },
    'customerData.boards_number_editable'(value){
      if(!value){
        // チェックが外された場合
        if(this.boardNumberAutoCalc === true) {
          // 条件に一致する場合は自動計算を行う
          this.calcBoardNum()
        } else {
          // 自動計算しない場合は初期値に戻す
          this.customerData.board_num = this.board_num_before
        }
      }
    }
  },
  created() {
    this.loadingOptions = true;
    const screen_id = getScreenId(this.$route.name, this.$options.name)
    Promise.all([
      KubunListAPI(screen_id, 'RYOKIN_PLAN_OP_KUBUN'),
      KubunListAPI(screen_id, 'MOSIKOMI_KUBUN'),
      KubunListAPI(screen_id, 'DEAI_STATUS_KUBUN'),
      KubunListAPI(screen_id, 'KAISEN_TEHAI_KUBUN'),
      KubunListAPI(screen_id, 'TESURYO_KUBUN'),
      KubunListAPI(screen_id, 'DOWNGRADE_FLG'),
      KubunListAPI(screen_id, 'TUTI_HAKKO_KUBUN'),
      KubunListAPI(screen_id, 'KAISEN_SHIJI_HAKKO_KUBUN'),
      KubunListAPI(screen_id, 'NORIKAE_MOTO_SERVICE'),
    ])
      .then((res) => {
        console.log("サービス情報登録画面:区分取得完了")
        this.ryokin_plan_op_kubun = this.kubunToSelect(res[0].data);
        this.mosikomi_kubun = this.kubunToSelect(res[1].data);
        this.deai_status_kubun = this.kubunToSelect(res[2].data);
        this.kaisen_tehai_kubun = this.kubunToSelect(res[3].data);
        this.tesuryo_kubun = this.kubunToSelect(res[4].data);
        this.downgrade_flg = this.kubunToSelect(res[5].data);
        this.tuti_hakko_kubun = this.kubunToSelect(res[6].data);
        this.kaisen_shiji_hakko_kubun = this.kubunToSelect(res[7].data);
        this.optionsTransferSourceService = this.kubunToSelect(res[8].data);
      })
      .finally(() => (this.loadingOptions = false));

    // 代理店判定
    const dairitenString = sessionStorage.getItem('dairitenData');
    let resultList:string[] = [];
    if( null!= dairitenString){
      const dairitenListItem = JSON.parse(dairitenString) as DairitenResultData[];
      dairitenListItem.forEach(element =>{
        resultList.push(element?.dairiten_cd);
      });
      if(resultList.includes("F90000000")){
      this.dairitenIsFT = true;
      }
    }
  },
  methods: {
    boardNumberRecalc(): void {
      // 親から呼ぶボード枚数自動計算ロジック（チェックボックスOFFで自動計算対象の場合のみ自動計算する）
      if (!this.customerData.boards_number_editable && this.boardNumberAutoCalc === true) {
        // 条件に一致する場合は自動計算を行う
        this.calcBoardNum()
      }
    },
    t(field: string): TranslateResult {
      return this.$t(`customerDetail.serviceRegister.${field}`);
    },
    emit(fieldName: string, value: string) {
      this.$emit('field-changed', fieldName, value);
    },
    kubunToSelect(kubunList: ResponseData_KUBUN_KANRI[]): SelectOption[] {
      return kubunList.map(({ vl, hyoji_mei }) => ({
        value: vl,
        text: hyoji_mei,
        disabled: false,
      }));
    },
    deaiStatusKubunToSelect(): SelectOption[] {
      return this.deai_status_kubun.filter(({ value }) => {
        if (this.customerData.deai_status_kubun !== DEAI_STATUS_KUBUN_REGISTED) {
          return (value === DEAI_STATUS_KUBUN_NORMAL || value === DEAI_STATUS_KUBUN_DEAI)
        }
        return true
      }).map(({ value, text }) => ({
        value: value,
        text: text,
        disabled: false,
      }));
    },
    // 機種ID変更時動作
    onChangeModelId(key: string | null){
      const selectObjet = this.optionsModel.filter(x => x.value === key)[0];
      if(null!=selectObjet)
      {
        this.kishu_name = selectObjet.name;
        this.max_board_num = selectObjet.max_board_num
        if (this.max_board_num == 0) {
          // MAXボード枚数0の時はチェックボックスON
          this.customerData.boards_number_editable = true
        }
        let kubun_before = this.optionsModel.find((item) => {
            return item.value == this.kishu_cd_before;
          })
        let kubun_after = this.optionsModel.find((item) => {
            return item.value == key;
          })

        if(kubun_before && kubun_after){
          // 緊急通報区分が"1"から"0"に変更された場合はアラートを表示する
          if(kubun_before.kinkyu_kubun == "1" && kubun_after.kinkyu_kubun == "0"){
            alert(this.t("messages.kishuWarning"));
          }
        }
      }
      else
      {
        this.kishu_name = ''; // 機種名クリア
        console.log("機種－対象データなし");
      }
    },
    filterRyokinPlan(){
      const optionsAppCode = this.optionsAppCode.filter((item) => item.value === this.customerData.service_cd)
      if (optionsAppCode.length > 0) {
        this.service_name = optionsAppCode[0]?.text.toString()
      } else {
        this.service_name = ''
      }
      let ryokin_plan_kubun = this.ryokin_plan_kubun
      // 初期化
      ryokin_plan_kubun.forEach((item) => {
        item.disabled = false
      })
      if (this.newContract) {
        // 新規（BB受注が存在しない）場合
        ryokin_plan_kubun.forEach((item) => {
          // [10,11]は選択不可
          if(item.value === '10' || item.value === '11') {
            item.disabled = true;
          }
          if(this.customerData.fzkhintype_cd != "67" && this.customerData.fzkhintype_cd != "68"){
            // 固定電話受注明細内容テーブルの付属品タイプコードが「67」,「68」でない場合
            ryokin_plan_kubun.forEach((item) => {
              // [50,51,52]は選択不可
              if(item.value == "50" || item.value == "51" || item.value == "52") item.disabled = true;
            })
          }
        })
      } else {
        if(this.customerData.protcol_kubun == "5") {
          // プロトコル区分が5の場合
          ryokin_plan_kubun.forEach((item) => {
            // [50,51,52]のみ選択可
            if(!(item.value == "50"|| item.value == "51" || item.value == "52")) item.disabled = true;
          })
        }else if(this.customerData.protcol_kubun != null) {
          // プロトコル区分が入力済みの場合
          ryokin_plan_kubun.forEach((item) => {
            // [50,51,52]は選択不可
            if(item.value == "50"|| item.value == "51" || item.value == "52") item.disabled = true;
          })
        }
        if(this.customerData.fzkhintype_cd != "67" && this.customerData.fzkhintype_cd != "68"){
            // 固定電話受注明細内容テーブルの付属品タイプコードが「67」,「68」でない場合
          ryokin_plan_kubun.forEach((item) => {
            // [50,51,52]は選択不可
            if(item.value == "50" || item.value == "51" || item.value == "52") item.disabled = true;
          })
        }
        if(this.customerData.deai_status_kubun != "0") {
          // 出会いステータス区分が通常以外の場合
          ryokin_plan_kubun.forEach((item) => {
            if (!(['10','11',this.ryokinPlanBefore].includes(item.value))) {
              // [10,11,元の料金プラン]以外は選択不可
              item.disabled = true;
            }
          })
          if (this.ryokinPlanBefore == '51') {
            // 元の料金プラン区分が51の場合
            ryokin_plan_kubun.forEach((item) => {
              if (item.value != '51') {
                // 51以外は選択不可
                item.disabled = true;
              }
            })
          }
          if (this.ryokinPlanBefore == '52') {
            // 元の料金プラン区分が52の場合
            ryokin_plan_kubun.forEach((item) => {
              if (item.value != '52') {
                // 52以外は選択不可
                item.disabled = true;
              }
            })
          }
        } else {
          // 出会いステータス区分が通常の場合
          ryokin_plan_kubun.forEach((item) => {
            // [10,11]は選択不可
            if(item.value === '10' || item.value === '11') {
              item.disabled = true;
            }
          })
        }
      }
      // サービスコードによる制御
      let lastTwoDigit = ''
      if(this.customerData.service_cd !== '' && this.customerData.service_cd != null){
        lastTwoDigit = this.customerData.service_cd.slice(-2,-1);
      }
      if(lastTwoDigit == "6" || lastTwoDigit == "8"){
        // 申込サービスコードの末尾2桁が「6*」「8*」の場合
        ryokin_plan_kubun.forEach((item) => {
          if(!(item.value == "02" || item.value == "03")) {
            // [02,03]のみ選択可
            item.disabled = true;
          }
        })
      }else if(lastTwoDigit == "7"){
        // 申込サービスコードの末尾2桁が「7*」の場合
        ryokin_plan_kubun.forEach((item) => {
          // [51]のみ選択可
          if(item.value != "51") {
            item.disabled = true;
          }
        })
      }else if(lastTwoDigit == "2"){
        // 申込サービスコードの末尾2桁が「2*」の場合
        ryokin_plan_kubun.forEach((item) => {
          // [52]のみ選択可
          if(item.value != "52") {
            item.disabled = true;
          }
        })
      }else if(lastTwoDigit == "5"){
        // 申込サービスコードの末尾2桁が「5*」の場合
        ryokin_plan_kubun.forEach((item) => {
          // [50]のみ選択可
          if(item.value != "50") {
            item.disabled = true;
          }
        })
      }else if(lastTwoDigit == "3"){
        // 申込サービスコードの末尾2桁が「3*」の場合
        ryokin_plan_kubun.forEach((item) => {
          // [00,01]のみ選択可
          if(!(item.value == "00" || item.value == "01")) {
            item.disabled = true;
          }
        })
      }
      this.ryokin_plan_kubun = ryokin_plan_kubun;
    },
    // ボード枚数自動計算
    calcBoardNum(): string{
      // イレギュラーフラグ,フラグ2が0または空かつ機種コードが入力済みの場合ボード枚数を自動計算する
      if(this.boardNumberAutoCalc === true)
      {
        this.customerData.boards_number_editable = false
        console.log("ボード枚数自動計算実行")
        let ch_cnt = Number(this.customerData.sou_ch_num) / DIVISION_IN_CALC_BOARD_NUM;
        console.log("ch_cnt="+ ch_cnt)
        let c = Math.sign(ch_cnt);
        let x = Math.floor(Math.abs(ch_cnt) + ADDITION_IN_CALC_BOARD_NUM);
        return String(c * x);
      }else{
      // 自動計算しない場合はそのままボード枚数を返す
        console.log("ボード枚数自動計算なし")
        this.customerData.boards_number_editable = true
        return this.customerData.board_num;
      }
      
    },
    // 保存後は比較用の値をそれぞれ保存時のデータに設定する
    setBeforeSection(){
      this.kishu_cd_before = this.customerData.kishu_cd
      this.serviceCdBefore = this.customerData.service_cd
      this.board_num_before = this.customerData.board_num
      this.sou_ch_num_before = this.customerData.sou_ch_num
    },
    blurBoardNum(value: string){
      this.listPhoneNumber.forEach((phone) => {
        // 既に設定されているレジ番号がボード枚数より大きい場合nullにする
        if(Number(value) < Number(phone.reji_no)){
          phone.reji_no = null
          phone.update_flg = FLAG_UPDATED
        }
      })
    },
    get_kaisen_kubun(value: string): string{
      let res: TranslateResult = value
      this.kaisen_shiji_hakko_kubun.forEach((val) => {
        if (val.value == value){
          res = val.text
        }
      })
      return res
    },
    get_tuti_kubun(value: string): string{
      let res: TranslateResult = value
      this.tuti_hakko_kubun.forEach((val) => {
        if (val.value == value){
          res = val.text
        }
      })
      return res
    }
  },
});

</script>
