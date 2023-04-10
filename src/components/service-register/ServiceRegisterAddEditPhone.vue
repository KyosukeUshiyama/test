<!--申込電番追加ダイアログ&申込電番変更ダイアログ-->
<template>
  <v-form ref="form" v-model="valid" :disabled="cancelStatus || !updatePermission || !createPermission">
    <v-card>
      <v-card-title>
        {{ t(editMode ? 'editPhone' : 'addPhone') }}
      </v-card-title>
      <v-card-text>
        <v-row>
          <v-col md="3" cols="12">
            <v-text-field
              v-model="internalPhoneData.gno"
              filled
              readonly
              :label="t('GNO')"
            />
          </v-col>
          <v-col md="3" cols="12">
            <v-text-field
              v-model="internalPhoneData.g_serial"
              filled
              readonly
              :label="t('gnoSerialNo')"
            />
          </v-col>
          <v-col md="3" cols="12" offset-md="3">
            <v-text-field
              v-model="internalPhoneData.uf_cd"
              filled
              readonly
              :label="t('ufCode')"
            />
          </v-col>
        </v-row>
        <v-row>
          <v-col md="6" cols="12">
            <v-text-field
              v-model="internalPhoneData.kanji"
              filled
              readonly
              :label="t('kanji')"
            />
          </v-col>
          <v-col md="6" cols="12">
            <v-text-field
              v-model="internalPhoneData.furigana"
              filled
              readonly
              :label="t('furigana')"
            />
          </v-col>
        </v-row>
        <v-row align="center">
          <v-col md="3" cols="12">
            <v-combobox
              v-model="internalPhoneData.ip_tel_no"
              :disabled="himozukiCheck || isCreatedSoFile()"
              :label="t('billingNumber')"
              :rules="billingNumberRule"
              :items="billingNumberOptions"
              append-outer-icon="mdi-pencil"
              autocomplete="auto-off"
              @update:search-input="updateIpTelNoInput"
              @blur="blurIpTelNo"
            />
          </v-col>
          <v-col md="3" cols="12">
            <v-text-field
              v-model="internalPhoneData.routing_tel_no"
              filled
              readonly
              :label="t('routingNumber')"
            />
          </v-col>
          <v-col md="3" cols="12">
            <v-select
              v-model="internalPhoneData.reji_no"
              :filled="registerNumberReadonly"
              :readonly="registerNumberReadonly"
              :items="registerNumberOptions"
              :label="t('registrationNumber')"
              :no-data-text="t('messages.notFound')"
            />
          </v-col>
        </v-row>
        <v-row align="center" :style="{ height: thirdRowHeight }">
          <v-col md="3" cols="12">
            <v-select
              v-model="internalPhoneData.daihyo_kubun"
              :label="t('representativeNumber')"
              :items="daihyo_kubun"
              :rules="representativeNumberRule"
              :menu-props="{ 'content-class': 'hide-disabled' }"
              :no-data-text="t('messages.notFound')"
              class="required-label"
            />
          </v-col>
          <v-col md="3" cols="12">
            <v-select
              v-model="internalPhoneData.bango_tsuchi_kubun"
              :label="t('callerNotification')"
              :items="bango_tsuchi_kubun"
              :rules="callerNotificationRule"
              :menu-props="{ 'content-class': 'hide-disabled' }"
              :no-data-text="t('messages.notFound')"
              class="required-label"
            />
          </v-col>
          <v-col md="3" cols="12" class="d-flex">
            <v-checkbox
              v-model="internalPhoneData.himozuki_change"
              :disabled="himozukiChangeDisable"
            />
            <v-select
              v-model.trim="internalPhoneData.himozuki_tel_no"
              :filled="!internalPhoneData.himozuki_change"
              :readonly="!internalPhoneData.himozuki_change"
              :label="t('linkedNumber')"
              :class="{
                'required-label': internalPhoneData.himozuki_ashk_tel_no,
              }"
              :rules="himozukiTelNoRule"
              :items="himozukiTelNoOptions"
              :no-data-text="t('messages.notFound')"
            />
          </v-col>
        </v-row>
        <v-row align="center" :style="{ height: thirdRowHeight }">
          <v-col md="3" cols="12">
            <v-text-field
              v-model.trim="internalPhoneData.chakushin_num"
              :label="t('incomingCallNumber')"
              :rules="incomingCallNumberRule"
              class="required-label"
            />
          </v-col>
          <v-col md="6" cols="12">
            <v-select
              v-model="internalPhoneData.n104_annai_kubun"
              :label="t('list104PhoneBook')"
              :items="n104_annai_kubun"
              :rules="list104PhoneBookRule"
              :no-data-text="t('messages.notFound')"
              class="required-label"
            />
          </v-col>
        </v-row>
        <v-row>
          <v-col md="6" cols="12">
            <v-text-field
              v-model.trim="internalPhoneData.kaisen_meigi"
              :label="t('lineName')"
              :rules="lineNameRule"
              class="required-label"
            />
          </v-col>
          <v-col md="6" cols="12">
            <v-text-field
              v-model.trim="internalPhoneData.kaisen_meigi_kana"
              :label="t('lineNameKana')"
              :rules="lineNameKanaRule"
              class="required-label"
            />
          </v-col>
        </v-row>
        <v-row>
          <v-col md="6" cols="12">
            <v-text-field
              v-model.trim="internalPhoneData.keisai_mei"
              :label="t('phoneBookName')"
              :rules="phoneBookNameRule"
            />
          </v-col>
          <v-col md="6" cols="12">
            <v-text-field
              v-model.trim="internalPhoneData.keisai_mei_kana"
              :label="t('phoneBookNameKana')"
              :rules="phoneBookNameKanaRule"
            />
          </v-col>
        </v-row>
        <v-row align="center">
          <v-col md="3" cols="12">
            <v-select
              v-model="internalPhoneData.ntt_np"
              :label="t('NTTPortabilityRequest')"
              :items="ntt_np_moshikomi_hakko_kubun"
              :no-data-text="t('messages.notFound')"
            />
          </v-col>
          <v-col md="3" cols="12">
            <v-text-field
              v-model="internalPhoneData.ntt_np_date"
              filled
              readonly
              :label="t('NTTPortabilityRequestDate')"
              :rule="NTTPortabilityRequestDateRule"
            />
          </v-col>
          <v-col md="3" cols="12">
            <v-text-field
              v-model="juchu_kubun"
              filled
              readonly
              :label="t('orderSection')"
            />
          </v-col>
          <v-col md="3" cols="12">
            <v-text-field
              v-model="internalPhoneData.kaiyaku_uketsuke_date"
              filled
              readonly
              :label="t('contractCancelAcceptDate')"
            />
          </v-col>
        </v-row>
        <v-row align="center">
          <v-col md="3" cols="12">
            <v-text-field
              :value="getSoStatus(internalPhoneData.so_shoki_data_sakusei_kubun)"
              filled
              readonly
              :label="t('soShokiDataSakuseiKubun')"
            />
          </v-col>
        </v-row>
      </v-card-text>
      <v-card-actions class="justify-space-between">
        <v-btn @click="close">
          {{ t('close') }}
        </v-btn>
        <v-btn
          color="primary"
          :disabled="cancelStatus || (editMode ? !updatePermission : !createPermission)"
          @click="confirm"
        >
          {{ t(`${editMode ? 'edit' : 'add'}`) }}
        </v-btn>
      </v-card-actions>
    </v-card>
  </v-form>
</template>

<script lang="ts">
import Vue, { PropType } from 'vue';
import {
  AddEditPhoneData,
  SelectOption,
  ADD,
  EDIT,
  SPECIAL_SERVICE_CD,
  NUMBER050,
  START_REJI_NUM,
  N104_ANNAI_NOT,
  DAIHYO_DENBAN,
  KO_DENBAN,
  DAIHYO_TSUCHI,
  JIDENBAN_TSUCHI,
  MOSIKOMI_HOLD,
  MOSHIKOMI,
  PROTCOL_KUBUN_050
} from '@/models/service-register';
import {
  CONFIRMATION,
  MESSAGE,
  REGISTER_UPDATE_EXECUTE,
  WHILE_INPUT,
} from '@/constants/dialog';
import { TranslateResult } from 'vue-i18n';
import { FormRef, InputValidation } from '@/models/validation';
import {
  bufferSize,
  conditionRequired,
  formatDate,
  formatTelNo,
  numberSize,
  required,
  formatFullWidth,
  formatHalfWidth
} from '@/utils/validation';
import { DialogOptions } from '@/models/dialog';
import { SearchFormData} from '@/models/media-phone';
import { KubunListAPI, ResponseData_KUBUN_KANRI } from '@/api/kubun-kanri';
import { AxiosError } from 'axios';
import { mapState } from 'vuex';
import { numberingRoutingNoAPI, numberingRoutingNoRequestData, numberingRoutingNoResponseData } from '@/api/add-edit-phone-api';
import { getMediaPhoneListAPI, MEDIA_PHONE_KARIYOYAKU, MediaPhoneSearchResponseData } from '@/api/media-phone-api';
import { PhoneNumberResponse } from '@/models/customer';
import { getScreenId } from '@/utils/screenIds';
import { FLAG_HOLD } from '@/constant/common-constant'

const DAIHYO_DENBAN_CONFIRTM_MSG = "代表電番は1つしか登録できません。既に登録済みの代表電番を子番号へ変更しますか？"
const REJIBAN_CHANGE_CONFIRTM_MSG = "代表電番を変更するとレジ番号が自動で変更されます。確認してください。"
const DENWACHO_KESAI_CHANGE = "※104電話帳掲載ができない番号が割り当てられるため、104電話帳掲載を「9 ：104案内×・電話帳×」に変更します。"

export default Vue.extend({
  props: {
    phoneData: {
      type: Object as PropType<PhoneNumberResponse>,
      required: true,
    },
    cancelStatus: {
      type: Boolean,
      default: false,
    },
    customerData: {
      type: Object,
      default: () => ({}),
    },
    editMode: {
      type: Boolean,
      default: false,
    },
    listPhoneTable: {
      type: Array as PropType<PhoneNumberResponse[]>,
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
    protcolKubun: {
      type: String,
      default: null,
    },
    banpoFlg: {
      type: Boolean,
      default: false
    },
  },
  data(): AddEditPhoneData {
    return {
      internalPhoneData: this.phoneData,
      valid: false,
      representativeNumberRule: [
        (value) =>
          required(
            value,
            this.$t(
              'customerDetail.serviceRegister.addEditPhone.representativeNumber'
            )
          ),
      ],
      callerNotificationRule: [
        (value) =>
          required(
            value,
            this.$t(
              'customerDetail.serviceRegister.addEditPhone.callerNotification'
            )
          ),
      ],
      incomingCallNumberRule: [
        (value) =>
          required(
            value,
            this.$t(
              'customerDetail.serviceRegister.addEditPhone.incomingCallNumber'
            )
          ),
        (value) => numberSize(value, 3),
      ],
      lineNameRule: [
        (value) =>
          required(
            value,
            this.$t('customerDetail.serviceRegister.addEditPhone.lineName')
          ),
        (value) => bufferSize(value, 72),
        (value) => formatFullWidth(value,true,this.$t('customerDetail.serviceRegister.addEditPhone.lineName')),
      ],
      lineNameKanaRule: [
        (value) =>
          required(
            value,
            this.$t('customerDetail.serviceRegister.addEditPhone.lineNameKana')
          ),
        (value) => bufferSize(value, 72),
        (value) => formatHalfWidth(value,this.$t('customerDetail.serviceRegister.addEditPhone.lineNameKana')),
      ],
      NTTPortabilityRequestDateRule: [(value) => formatDate(value)],
      isDirty: false,
      daihyo_kubun: [] as SelectOption[],
      bango_tsuchi_kubun: [] as SelectOption[],
      n104_annai_kubun: [] as SelectOption[],
      ntt_np_moshikomi_hakko_kubun: [] as SelectOption[],
      n104list: [] as SelectOption[],
      billingNumberOptions: [],
      himozukiChangeDisable: false,
      createPermission: false,
      updatePermission: false,
      telNoAble: true,
      himozuki_tel_no_before: "",
      daihyo_kubun_before: '',
      ip_tel_no_before: "",
      juchu_kubun_list: [] as SelectOption[],
      so_sakusei_kubun: [] as SelectOption[],
      juchu_kubun: "",
    };
  },
  computed: {
    ...mapState('customer', ['bbJuchu']),
    disabled_self_no_noti(): boolean {
      let last_two_digits = ""
      if(!this.bbJuchu || !this.bbJuchu.service_cd)
      {
        return false
      }

      last_two_digits = this.bbJuchu.service_cd?.slice(-2,-1);
      if (['6', '8'].includes(last_two_digits)) return true;
      return false;
    },
    thirdRowHeight(): string {
      if (this.$vuetify.breakpoint.mdAndDown) return 'auto';
      return '110px';
    },
    registerNumberReadonly(): boolean {
      let last_two_digit = '';
      last_two_digit = this.customerData.service_cd.slice(-2,-1);
      // サービスコードが[2*,7*] AND 電番が050以外から始まる場合レジ番選択不可
      if (['2', '7'].includes(last_two_digit) && !this.checkIpTelNoStartWith050()) {
        // レジ番クリア
        this.clearRejiNo()
        return true
      }
      return this.internalPhoneData.daihyo_kubun !== KO_DENBAN;
    },
    registerNumberOptions(): number[] {
      if (this.internalPhoneData.daihyo_kubun !== KO_DENBAN) return [1];
      let reji_no = [];
      for(let i = START_REJI_NUM; i <= this.customerData.board_num; i++){
        // レジ番号をSTART_REJI_NUM(2)からボード枚数まで選べるようにセット
        reji_no.push(Number(i));
      }
      return reji_no;
    },
    billingNumberRule(): InputValidation {
      return [
        formatTelNo(this.internalPhoneData.ip_tel_no),
        this.telNoAble
      ]
    },
    list104PhoneBookRule(): InputValidation {
        return [
          required(
            this.internalPhoneData.n104_annai_kubun,
            this.$t('customerDetail.serviceRegister.addEditPhone.list104PhoneBook')
          ),
          this.valid104PhoneBook(),
        ]
    },
    phoneBookNameRule(): InputValidation {
      return [
        bufferSize(this.internalPhoneData.keisai_mei, 72),
        (value) => formatFullWidth(value,false,this.$t('customerDetail.serviceRegister.addEditPhone.phoneBookName')),
      ];
    },
    phoneBookNameKanaRule(): InputValidation {
      return [
        bufferSize(this.internalPhoneData.keisai_mei_kana, 72),
        (value) => formatFullWidth(value,false,this.$t('customerDetail.serviceRegister.addEditPhone.phoneBookNameKana')),
      ];
    },
    himozukiTelNoRule(): InputValidation {
      if (this.checkIpTelNoStartWith050() && this.checkSpecialServiceCode()) {
        // 電番が050で始まって、サービスコードが[2,5,7]の場合入力必須
        return [
          conditionRequired(
            this.internalPhoneData.himozuki_tel_no,
            this.internalPhoneData.himozuki_change,
            this.t('linkedCheckBox'),
            this.t('linkedNumber')
          ),
        ];
      }
      return [true]
    },
    himozukiTelNoOptions(): SelectOption[] {
      // 自身が050から始まる場合は紐づけされている番号を表示
      if(this.checkIpTelNoStartWith050()){
        return [{
          value: this.internalPhoneData.himozuki_tel_no ? this.internalPhoneData.himozuki_tel_no : '',
          text: this.internalPhoneData.himozuki_tel_no ? this.internalPhoneData.himozuki_tel_no : '',
        }]
      }
      // 使用されている紐づき番号を取得
      let unableHimozukiList = this.listPhoneTable.filter(
        (item) => (item.himozuki_tel_no && item.himozuki_tel_no != this.internalPhoneData.himozuki_tel_no)
      ).map(
        ({ himozuki_tel_no }) => himozuki_tel_no
      )
      // 電番リストから自身の番号以外を取得
      let himozukiList = this.listPhoneTable.filter(
        (item) => (item.ip_tel_no != this.internalPhoneData.ip_tel_no && item.ip_tel_no.startsWith(NUMBER050))
      ).map(
        ({ ip_tel_no }) => {
          let text = ip_tel_no
          // 使用中の番号はわかるようにする
          if (unableHimozukiList.includes(ip_tel_no)) text += '（使用中）'
          return {
            value: ip_tel_no,
            text: text,
          };
        }
      );
      // 紐づき番号が設定されているがBB受注にない場合はリストに追加する
      // リストにない場合は表示されないため
      if(himozukiList.find((item) => item.value == this.internalPhoneData.himozuki_tel_no) == undefined) {
        himozukiList.push({
          value: this.internalPhoneData.himozuki_tel_no,
          text: this.internalPhoneData.himozuki_tel_no,
        })
      }
      return himozukiList
    },
    // 紐づき番号として使われている場合は変更不可
    himozukiCheck(): boolean{
      if(!this.checkIpTelNoNothing() || !this.editMode) return false
      return (this.listPhoneTable.filter(
        (item) => item.himozuki_tel_no == this.internalPhoneData.ip_tel_no
      ).length) > 0
    }
  },
  watch: {
    'internalPhoneData.daihyo_kubun'(value) {
      // 代表電番が選択された場合は1固定、代表電番以外が選択された場合、かつレジ番号が1の場合はnullに変更
      console.log("internalPhoneData.daihyo_kubun change")
      if (value === DAIHYO_DENBAN) {
        this.internalPhoneData.reji_no = 1;
      } else {
        if(1==this.internalPhoneData.reji_no)this.internalPhoneData.reji_no = null;
      }
      this.updateHasshinsyaTuti()

    },
    'internalPhoneData.himozuki_change'(checked) {
      this.updateHasshinsyaTuti();
      // チェックが外れた場合は変更前の紐づき電番に戻す
      if (!checked) {
        this.internalPhoneData.himozuki_tel_no = this.himozuki_tel_no_before;
      }
    },
    internalPhoneData: {
      deep: true,
      handler() {
        this.isDirty = true;
      },
    },
  },
  created() {
    if(this.editMode) this.checkHimozukiTelNoCheckBox() // 変更時は課金電番に応じて紐づけ変更可否を決める
    this.ip_tel_no_before = this.internalPhoneData.ip_tel_no
    this.himozuki_tel_no_before = this.internalPhoneData.himozuki_tel_no
    this.daihyo_kubun_before = this.internalPhoneData.daihyo_kubun
    const name = this.editMode ? "ApplicationPhoneNumberChangeDialog" : "ApplicationPhoneNumberAdditionDialog";
    const screen_id = getScreenId(this.$route.name, name);
    this.createPermission = this.$$hasCreatePermission(screen_id);
    this.updatePermission = this.$$hasUpdatePermission(screen_id);
    // メディア電番検索(GNO以外固定)
    const mediaPhoneReqData: SearchFormData = {
      jigyosha_senyo_no: '',
      match_type: null,
      gno: this.$route.params.gno,
      biko: '',
      kanri_flg: MEDIA_PHONE_KARIYOYAKU,
      protocol_kubun: null
    };
    Promise.all([
      KubunListAPI(screen_id, 'DAIHYO_KUBUN'), // 代表電番区分
      KubunListAPI(screen_id, 'BANGO_TSUCHI_KUBUN'), // 番号通知区分
      KubunListAPI(screen_id, 'N104_ANNAI_KUBUN'), // N104案内区分
      KubunListAPI(screen_id, 'NTT_NP_MOSHIKOMI_HAKKO_KUBUN'), // NTT番ポ申込発行区分
      getMediaPhoneListAPI(screen_id, mediaPhoneReqData),
      KubunListAPI(screen_id, 'JUCHU_KUBUN'),// 受注区分
      KubunListAPI(screen_id, 'SO_DATA_SAKUSEI_KUBUN') // SOデータ作成区分
    ])
      .then((res) => {
        this.daihyo_kubun = this.kubunToSelect(res[0].data);
        this.bango_tsuchi_kubun = this.kubunToSelect(res[1].data);
        this.n104_annai_kubun = this.kubunToSelect(res[2].data);
        this.n104list = this.n104_annai_kubun;
        this.ntt_np_moshikomi_hakko_kubun = this.kubunToSelect(res[3].data);
        this.juchu_kubun_list = this.kubunToSelect(res[5].data)
        this.so_sakusei_kubun = this.kubunToSelect(res[6].data)
        // 編集時は対象の申込区分を表示
        if(this.editMode){
          this.juchu_kubun = String(this.juchu_kubun_list.filter(item => item.value === this.internalPhoneData.juchu_kubun)[0]?.text)
        }else {
          this.juchu_kubun = MOSHIKOMI
        }
        console.log(this.juchu_kubun)
        const found_bango_tsuchi_kubun = this.bango_tsuchi_kubun.find(
          (item) => item.value === JIDENBAN_TSUCHI
        );
        this.updatebillingNumberOptions(res[4].data.results)
        if (found_bango_tsuchi_kubun)
          found_bango_tsuchi_kubun.disabled = this.disabled_self_no_noti;

        // 番号の内容に応じて、代表電番区分の選択可能状態を更新する
        this.updateDaihyoKubun()
        this.update104List()
        this.updateHasshinsyaTuti()
      })
      .catch((err: AxiosError) => {
        console.error(err.response);
      });
  },
  methods: {
    isCreatedSoFile(): boolean {
      if (this.internalPhoneData.so_shoki_data_sakusei_kubun === '1' && this.editMode) {
        return true
      } else {
        return false
      }
    },
    updateIpTelNoInput(currentValue: string) {
      if (currentValue !== null && currentValue !== undefined) {
        this.internalPhoneData.ip_tel_no = currentValue
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
      console.log("confirm")
      // 編集ステータスの設定
      if (this.editMode) {
        // edit時 ステータスがaddになっていないときのみeditに設定
        this.internalPhoneData.editStatus = EDIT;
      } else {
        // add時
        this.internalPhoneData.editStatus = ADD;
      }
      if (this.internalPhoneData.daihyo_kubun === DAIHYO_DENBAN) 
      {  
        const betuDaihyoDenban = this.GetBetuDenbanDaihyo()
        if(betuDaihyoDenban != null)
        {
          if(!this.checkIpTelNoNothing())
          {
            this.$store.dispatch('dialog/open', {
              type: CONFIRMATION,
              subType: REGISTER_UPDATE_EXECUTE,
              messages: [DAIHYO_DENBAN_CONFIRTM_MSG,REJIBAN_CHANGE_CONFIRTM_MSG],
              callback: () =>
              {
                console.log("子番号へ変更－OK")
                this.confirmProc()
              },
            })
          }
          else
          {
            // 課金電番が入力されていない場合は、ここではダイアログを呼び出さない
            this.confirmProc(true)
          }
        }
        else
        {
          this.confirmProc()
        }
      }
      else
      {
        this.confirmProc()
      }

    },
    confirmProc(useCheckDaihyoDenban = false): void {
      let type = CONFIRMATION
      let messages = [this.t(`messages.${this.editMode ? 'change' : 'add'}`).toString()]
      
      if (!this.checkIpTelNoNothing() )
      {
        messages.push(this.t('messages.addMissingBillingPhoneNo').toString())
        // 課金電番登録API呼び出し
        const reqData = this.getNumberingRoutingNoRequestData()
        numberingRoutingNoAPI(getScreenId(this.$route.name), reqData)
          .then((res) => {
            console.log("numberingRoutingNoAPI")
            const data = res.data
            console.log(data)
            // メッセージがある場合は画面に表示
            if (data != null && (data.error_message != '' || data.confirm_message.length != 0)) {
              if (data.error_message == '') {
                // memo:自動採番の場合は注意文言の仕様はないが、念のため作成
                messages += data.confirm_message
              } else {
                // エラーがある場合はメッセージを表示して登録確認をしない
                this.internalPhoneData.ip_tel_no = ''
                this.$store.dispatch('dialog/open', {
                  type: MESSAGE,
                  subType: REGISTER_UPDATE_EXECUTE,
                  messages: [data.error_message],
                })
                return
              }
            }
            const valid = (this.$refs.form as FormRef).validate();
            if (valid) {
              //104電話帳掲載リストの1,2が選択可能か、既に9が選択されていないか判別
              if (this.checkSpecialServiceCode() === true && data.ip_tel_no.startsWith(NUMBER050) === true && this.internalPhoneData.n104_annai_kubun !== N104_ANNAI_NOT) {
                //104電話帳掲載リストを9に変更する旨メッセージ追加
                messages.push(DENWACHO_KESAI_CHANGE)
              }
              this.$store.dispatch('dialog/open', {
                type: type,
                subType: REGISTER_UPDATE_EXECUTE,
                messages: messages,
                callback: () => this.confirmCallback(data),
              } as DialogOptions);
            }
          })
          .catch((err: AxiosError) => {
            console.error(err.response);
          });
      }
      else
      {
        console.log("確定－自動採番なし")
        if(useCheckDaihyoDenban)
        {
          // memo ダイアログのcallback中に再度callbackは呼べないため、メッセージ連結を行う
          messages.push(DAIHYO_DENBAN_CONFIRTM_MSG)
          messages.push(REJIBAN_CHANGE_CONFIRTM_MSG)
        }
        else if (this.daihyo_kubun_before === DAIHYO_DENBAN && this.internalPhoneData.daihyo_kubun !== DAIHYO_DENBAN) {
          // 代表電番→子電番の場合
          messages.push(REJIBAN_CHANGE_CONFIRTM_MSG)
        }
        console.log(this.internalPhoneData)
        const valid = (this.$refs.form as FormRef).validate();
        if (valid) {
          this.$store.dispatch('dialog/open', {
            type: type,
            subType: REGISTER_UPDATE_EXECUTE,
            messages: messages,
            callback: () => this.$emit('confirm', this.internalPhoneData),
          } as DialogOptions);
        }
      }

      // 採番リスト更新
      this.$emit('numbered')
    },
    // 完了時コールバック関数（データ更新など実施）
    confirmCallback(data: numberingRoutingNoResponseData): void {
      console.log("confirmCallback")
      this.replaceRoutingNo(data,true)
      if (this.check104AnnaiNotOnly() === true) {
        //104電話帳掲載リストを9に変更
        this.internalPhoneData.n104_annai_kubun = N104_ANNAI_NOT
      }
      this.$emit('confirm', this.internalPhoneData)
    },
    // 課金電番入力時
    blurIpTelNo(): void {
      this.internalPhoneData.routing_tel_no = ''
      // 課金電番のエラー状態をクリア
      this.telNoAble = true
      this.updateHasshinsyaTuti()
      // 代表区分リスト制御
      this.updateDaihyoKubun()
      if (!this.checkIpTelNoNothing()) return
      // リスト内の電番と重複なし 編集している電番は除く
      if(this.listNumberedTentative.includes(this.internalPhoneData.ip_tel_no) && this.ip_tel_no_before !== this.internalPhoneData.ip_tel_no){
        // 重複している電番が解約されいる場合は登録可能
        let dupTelnoList = this.listPhoneTable.filter((item) => item.ip_tel_no === this.internalPhoneData.ip_tel_no && item.juchu_kubun != '9')
        if (dupTelnoList.length > 0) {
          this.$store.dispatch('dialog/open', {
            type: MESSAGE,
            messages: [this.t('messages.duplicateNumber')],
            } as DialogOptions);
          this.telNoAble = false
          return
        }
      }
      if (this.banpoFlg == false && this.internalPhoneData.ip_tel_no?.startsWith(NUMBER050)) {
        this.$store.dispatch('dialog/open', {
          type: MESSAGE,
          messages: [this.t('messages.banpoUnable')],
          } as DialogOptions);
        this.telNoAble = false
        return
      }
      // 課金電番登録API呼び出し
      const reqData = this.getNumberingRoutingNoRequestData()
      numberingRoutingNoAPI(getScreenId(this.$route.name), reqData)
        .then((res) => {
          console.log("numberingRoutingNoAPI戻り")
          const data = res.data
          let type = CONFIRMATION
          let messages = '';
          console.log(data)
          // メッセージがある場合は画面に表示
          if (data.error_message != '' || data.confirm_message.length != 0) {
            if (data.error_message == '') {
              messages = data.confirm_message.join('\n')
            } else {
              type = MESSAGE
              messages = data.error_message
              // エラーメッセージがある場合は登録不可
              this.telNoAble = false
            }
            this.$store.dispatch('dialog/open', {
              type: type,
              subType: REGISTER_UPDATE_EXECUTE,
              messages: [messages],
              callback: () => this.replaceRoutingNo(data),
            } as DialogOptions);
          } else {
            this.replaceRoutingNo(data)
          }
        })
        .catch((err: AxiosError) => {
          console.error(err.response);
        });
    },
    // 課金電番登録API要求用データ生成
    getNumberingRoutingNoRequestData():numberingRoutingNoRequestData {
      const gno = this.$route.params.gno;
      const { ip_tel_no, g_serial, himozuki_change } = this.internalPhoneData;
      const { service_cd, kukaku_cd, deai_status_kubun, seti_tel } = this.customerData
      const reqData: numberingRoutingNoRequestData = {
        gno: gno,
        ip_tel_no: ip_tel_no,
        serial_no: g_serial,
        service_cd: service_cd,
        kukaku_cd: kukaku_cd,
        protcol_kubun: this.protcolKubun,
        deai_status_kubun: deai_status_kubun,
        seti_tel: seti_tel,
        himozuki_ashk_tel_no: himozuki_change,
        numberedTentative: this.listNumberedTentative,
        routingNumberedTentative: this.listRoutingNumberedTentative
      }
      return reqData
    },
    // 代表区分リスト更新
    updateDaihyoKubun(): void {
      // 一度すべて有効にする
      this.daihyo_kubun.forEach((item) => {
        item.disabled = false
      })
      
      // Amavo,Coovoのサービス(プロトコル区分=5) AND 電番が050から始まらない AND 電番が入力済み 
      if (this.checkpecialProtcol() && !this.checkIpTelNoStartWith050() && this.checkIpTelNoNothing()) {
        this.daihyo_kubun.forEach((item) => {
          if (item.value == DAIHYO_DENBAN) {
            // 代表区分選択不可
            item.disabled = true
          }
        })
        if (this.internalPhoneData.daihyo_kubun == DAIHYO_DENBAN) {
          // 代表電番選択済みの場合は入力をクリア
          this.internalPhoneData.daihyo_kubun = ''
        }
      }
    },
    // 課金電番リスト更新
    updatebillingNumberOptions(data: MediaPhoneSearchResponseData[]): void {
      let phoneList: string[] = []
      data.forEach((item) => {
        phoneList.push(item.jigyosha_senyo_no)
      })
      this.billingNumberOptions = phoneList
    },
    // 104電話帳掲載リスト更新
    update104List(): void {

      console.log("update104List")
      const umu = !this.check104AnnaiNotOnly()

      this.n104_annai_kubun = this.n104_annai_kubun.map((item) =>
      {
         if(item.value !== N104_ANNAI_NOT )return { ...item, disabled: !umu}; 
         return item
      });
    },
    check104AnnaiNotOnly():boolean
    {
      let result = false
      if(this.checkSpecialServiceCode() && this.checkIpTelNoStartWith050())
      {
        result = true
      }
      return result
    },
    valid104PhoneBook():boolean| TranslateResult {
      if(false === this.check104AnnaiNotOnly())
      {
        return true
      }

      if(this.internalPhoneData.n104_annai_kubun === N104_ANNAI_NOT)
      {
        return true
      }
      
      let targetName = "選択不可"
      // エラーメッセージ用文言取得
      const targetList = this.n104_annai_kubun.filter((item) =>item.value === N104_ANNAI_NOT)
      if(targetList.length > 0 )
      {
        // 存在しない場合（=ロード前）は仮メッセージを表示
        targetName = targetList[0]?.text.toString()
      }
      return false || "「" + targetName.toString() + "」以外選択不可"
    },
    // IP_TEL_NOが050から始まるか
    checkIpTelNoStartWith050():boolean {
      let result = false
      if(this.checkIpTelNoNothing())
      {
        result = this.internalPhoneData.ip_tel_no.startsWith(NUMBER050)
      }
      return result
    },
    // IP_TEL_NOが有効な値か（null以外且つ文字なし以外）
    checkIpTelNoNothing():boolean {
      let result = true;
      if(null === this.internalPhoneData.ip_tel_no || '' === this.internalPhoneData.ip_tel_no)
      {
        result = false
      }
      return result
    },
    // 発信者番号通知リスト更新
    updateHasshinsyaTuti(): void {
      console.log("updateHasshinsyaTuti")

      // 一度すべて有効にする
      this.bango_tsuchi_kubun = this.bango_tsuchi_kubun.map((item)=>
      {
        return { ...item, disabled: false}; 
      })
      const found_bango_tsuchi_kubun = this.bango_tsuchi_kubun.find(
        (item) => item.value === JIDENBAN_TSUCHI
      );
      if (found_bango_tsuchi_kubun)
      {
        if(this.disabled_self_no_noti || this.internalPhoneData.daihyo_kubun === DAIHYO_DENBAN)
        {
          found_bango_tsuchi_kubun.disabled = true
          if(this.internalPhoneData.bango_tsuchi_kubun === JIDENBAN_TSUCHI)
          {
            console.log("発信者番号通知リストクリア（自電番通知選択不可）")
            this.internalPhoneData.bango_tsuchi_kubun = "";
          }
        }
        else if(this.internalPhoneData.ip_tel_no?.startsWith(NUMBER050))
        {
          console.log("課金電番050=>代表通知以外選択不可")
          found_bango_tsuchi_kubun.disabled = true
          if(this.internalPhoneData.bango_tsuchi_kubun === JIDENBAN_TSUCHI)
          {
            console.log("発信者番号通知リストクリア（自電番通知選択不可）")
            this.internalPhoneData.bango_tsuchi_kubun = "";
          }
        }
      }
      if(this.internalPhoneData.himozuki_change)
      {
        console.log("紐づき番号変更にチェックあり=>代表通知以外選択不可")
        this.bango_tsuchi_kubun = this.bango_tsuchi_kubun.map((item)=>
        {
          if(item.value === DAIHYO_TSUCHI) return item
          return { ...item, disabled: true}; 
        })
        if(this.internalPhoneData.bango_tsuchi_kubun !== DAIHYO_TSUCHI)
        {
          console.log("発信者番号通知リストクリア（代表通知以外選択不可）")
          this.internalPhoneData.bango_tsuchi_kubun = "";
        }
      }
    },
    // 別電番の代表区分のデータを取得（ない場合null,存在する場合、その電番)
    GetBetuDenbanDaihyo() : PhoneNumberResponse | null {
      let result: PhoneNumberResponse | null = null

      const temp = (this.listPhoneTable as PhoneNumberResponse[]).find((item)=>
      {
        return (item.daihyo_kubun == DAIHYO_DENBAN &&
        item.id !== this.internalPhoneData.id)
      });
      if(temp)
      {
        console.log("GetBetuDenbanDaihyo->別代表電番あり")
        result = temp
      }
      return result
    },
    // 特定のサービスコードか
    checkSpecialServiceCode() : boolean {
      let result = false
      let last_two_digit = "";
      last_two_digit = this.customerData.service_cd.slice(-2,-1);
      if(SPECIAL_SERVICE_CD.includes(last_two_digit)){
        result = true
      }
      return result
    },
    // 特定のプロトコル区分か
    checkpecialProtcol(): boolean {
      let result = false
      if (PROTCOL_KUBUN_050 == this.protcolKubun) {
        result = true
      }
      return result
    },
    // ルーティング番号の置き換えメソッド
    replaceRoutingNo(data: numberingRoutingNoResponseData,updateOnly = false): void {
      console.log("replaceRoutingNo")
      if (data.ip_tel_no != null && data.ip_tel_no != '') {
        this.internalPhoneData.ip_tel_no = data.ip_tel_no
      }
      if (data.routing_tel_no!= null && data.routing_tel_no != '') {
        this.internalPhoneData.routing_tel_no = data.routing_tel_no
      }
      if (data.kukaku_cd != null && data.kukaku_cd != '') {
        this.customerData.kukaku_cd = data.kukaku_cd
      }
      if (data.moshikomi_kubun === MOSIKOMI_HOLD) {
        this.internalPhoneData.hold_flg = FLAG_HOLD;
      }
      if(false == updateOnly)
      {
        // セットした番号の内容に応じて、各選択可能状態を更新する
        this.checkHimozukiTelNoCheckBox()
        this.update104List()
      }
    },
    clearRejiNo(): void {
      this.internalPhoneData.reji_no = null
    },
    t(field: string): TranslateResult {
      return this.$t(`customerDetail.serviceRegister.addEditPhone.${field}`);
    },
    checkHimozukiTelNoCheckBox(): void {
      if (this.checkSpecialServiceCode() && !this.checkIpTelNoStartWith050()) {
        this.himozukiChangeDisable = false
      } else {
        this.himozukiChangeDisable = true
        this.internalPhoneData.himozuki_change = false
      }
    },
    kubunToSelect(kubunList: ResponseData_KUBUN_KANRI[]): SelectOption[] {
      return kubunList.map(({ vl, hyoji_mei }) => ({
        value: vl,
        text: hyoji_mei,
        disabled: false,
      }));
    },
    // 掲載名のバリデーション判定
    judge_keisai_mei(kana: boolean): boolean{
      if(kana){
        return ( this.internalPhoneData.keisai_mei !== "" && this.internalPhoneData.keisai_mei !== null)
      }else{
        return ( this.internalPhoneData.keisai_mei_kana !== "" && this.internalPhoneData.keisai_mei_kana !== null)
      }
    },
    getSoStatus(status: string): TranslateResult{
      let s = this.so_sakusei_kubun.filter((item: SelectOption) => item.value === status)
      if (s.length > 0) {
        return s[0].text
      }
      return ''
    },
  },
});
</script>
