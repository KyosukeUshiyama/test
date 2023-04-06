<!--申込サービス追加ダイアログ&申込サービス変更ダイアログ-->
<template>
  <v-form ref="form" v-model="valid" :disabled="cancelStatus || !updatePermission || !createPermission">
    <v-card>
      <v-card-title>
        {{ !editMode ? t('addService') : t('editService') }}
      </v-card-title>
      <v-card-text>
        <v-row>
          <v-col md="3" cols="12">
            <v-text-field
              v-model="serviceForm.gno"
              filled
              readonly
              :label="t('GNO')"
            />
          </v-col>
          <v-col md="3" cols="12">
            <v-text-field
              v-model="serviceForm.serial_no"
              filled
              readonly
              :label="t('serialNo')"
            />
          </v-col>
          <v-col md="3" offset-md="3" cols="12">
            <v-text-field
              v-model="serviceForm.uf_cd"
              filled
              readonly
              :label="t('UFCode')"
            />
          </v-col>
        </v-row>
        <v-row>
          <v-col md="6" cols="12">
            <v-text-field
              v-model="serviceForm.sechimoto_kanji_kaisyamei"
              filled
              readonly
              :label="t('kanji')"
            />
          </v-col>
          <v-col md="6" cols="12">
            <v-text-field
              v-model="serviceForm.sechimoto_kana_kaisyamei"
              filled
              readonly
              :label="t('Furigana')"
            />
          </v-col>
        </v-row>
        <v-row align="center">
          <v-col md="6" cols="12">
            <v-tooltip :disabled="!serviceForm.fuka_service_mei" bottom>
              <template v-slot:activator="{ on, attrs }">
                <div v-on="on">
                  <v-select
                    v-model="serviceForm.fuka_service_cd"
                    :items="fukaServiceOptions"
                    item-text="text"
                    item-value="value"
                    class="required-label"
                    :label="t('additionServiceName')"
                    :rules="additionServiceNameRule"
                    :no-data-text="t('messages.notFound')"
                    v-bind="attrs"
                    @input="changeFukaService(serviceForm.fuka_service_cd)"
                  />
                </div>
              </template>
              <span>{{ serviceForm.fuka_service_mei }}</span>
            </v-tooltip>
          </v-col>
          <v-col md="2" cols="12">
            <v-text-field
              v-model="serviceForm.fuka_service_cd"
              filled
              readonly
              :label="t('additionServiceCode')"
            />
          </v-col>
          <v-col md="4" cols="12">
            <v-text-field
              v-model="protcol_kubun"
              filled
              readonly
              :label="t('protocolName')"
            />
          </v-col>
        </v-row>
        <v-row>
          <v-col md="3" cols="12">
            <app-date-picker
              v-model="serviceForm.moshikomi_date"
              :label="t('applicationDate')"
              :rules="applicationDateRule"
              required
            />
          </v-col>
          <v-col md="3" cols="12">
            <app-date-picker
              v-model="serviceForm.koji_kanryo_date"
              :label="t('completionDate')"
              :rules="completionDateRule"
            />
          </v-col>
          <v-col md="3" cols="12">
            <app-date-picker
              v-model="serviceForm.kaiyaku_date"
              :label="t('contractCancelDate')"
              :rules="contractCancelDateRule"
            />
          </v-col>
        </v-row>
        <v-row>
          <v-col md="3" cols="12">
            <v-text-field
              v-model="serviceForm.ch_num"
              :label="t('CHNumber')"
              :rules="CHNumberRule"
              @blur="blurCHNum()"
            />
          </v-col>
        </v-row>
        <v-row>
          <v-col cols="12">
            <v-textarea
              v-model="serviceForm.jigyosha_moshiokuri_jiko"
              rows="2"
              :label="t('itemsSentByIpLineOperator')"
              :rules="itemsSentByIpLineOperatorRule"
            />
          </v-col>
        </v-row>
        <v-row>
          <v-col cols="12">
            <div class="d-flex justify-space-between align-center mb-5">
              <h3>{{ t('phoneNoList') }}</h3>
              <v-btn :disabled="cancelStatus || (editMode ? !updatePermission : !createPermission)" color="primary" @click="confirm">
                {{ t(!editMode ? 'add' : 'change') }}
              </v-btn>
            </div>
            <v-data-table
              :headers="headers"
              :items="phoneList"
              :options.sync="tableOptions"
              hide-default-footer
            >
              <template v-slot:[`item.edit`]="{ item }">
                <v-icon
                  :disabled="cancelStatus || !(updatePermission || readPermission)"
                  @click="edit(item)"
                >
                  mdi-pencil
                </v-icon>
              </template>
              <template v-slot:[`item.daihyo_kubun`]="{ item }">
                {{ item.daihyo_kubun === "1" ? '◎' : '' }}
              </template>
              <template #top="{ pagination, options, updateOptions }">
                <v-data-footer
                  :pagination="pagination"
                  :options="options"
                  items-per-page-text="$vuetify.dataTable.itemsPerPageText"
                  :items-per-page-options="[
                    1, 5, 10, 15, 20, 25, 30, 35, 40, 45, 50,
                  ]"
                  @update:options="updateOptions"
                />
              </template>
              <template v-slot:[`item.so_shoki_data_sakusei_kubun`]="{ item }">
                {{ getSoStatus(item.so_shoki_data_sakusei_kubun) }}
              </template>
              <template #no-data>
                <p>{{ t('messages.notFound') }}</p>
              </template>
            </v-data-table>
          </v-col>
        </v-row>
        <v-dialog v-model="showAddPhone" persistent width="1200">
          <service-register-add-edit-phone
            v-if="showAddPhone"
            :phone-data="phoneData"
            :customer-data="customerData"
            :list-phone-table="serviceData.list_service_data"
            :edit-mode="modeEditPhone"
            :list-numbered-tentative="numberForExcluding"
            :list-routing-numbered-tentative="routingNumberForExcluding"
            :protcol-kubun="protcol_kubun_value"
            :banpo-flg="banpoFlg"
            @confirm="confirmRegisterPhone"
            @close="showAddPhone = false"
          />
        </v-dialog>
      </v-card-text>
      <v-card-actions class="justify-space-between">
        <v-btn @click="close">
          {{ t('close') }}
        </v-btn>
        <v-btn
          color="primary"
          :disabled="cancelStatus || !createPermission_phone"
          @click="addPhone"
        >
          {{ t('addPhoneNo') }}
        </v-btn>
      </v-card-actions>
    </v-card>
  </v-form>
</template>

<script lang="ts">
import Vue, { PropType } from 'vue';
import { mapState } from 'vuex';
import AppDatePicker from '../UI/AppDatePicker.vue';
import {
  ApplicationServiceData,
  ServiceDataReponse,
  FukaServiceOptions,
  ProtcolOptions,
} from '@/models/application-service';
import { responseData_FukaServiceMst } from '@/models/service/index';
import { ServiceApplicationResponse, PhoneNumberResponse } from '@/models/customer/index';
import { TranslateResult } from 'vue-i18n';
import { DataTableHeader } from 'vuetify';
import ServiceRegisterAddEditPhone from './ServiceRegisterAddEditPhone.vue';
import {
  ADD,
  EDIT,
  PROTCOL_KUBUN_SPECIAL,
  PROTCOL_KUBUN_050,
  PROTCOL_KUBUN_050_CHECK_SERVICE_CD_LIST,
  PROTCOL_KUBUN_050_SPECIAL_RYOKIN_KUBUN,
  PROTCOL_KUBUN_050_SPECIAL_UF_TOUROKU,
  ERRMSG_PROTCOL_KUBUN_SPECIAL_CH_NUM,
  DAIHYO_DENBAN,
  KO_DENBAN,
} from '@/models/service-register';
import { FLAG_NORMAL, FLAG_ADDED, FLAG_UPDATED, DEFAULT_HYOJI_NUM } from '@/constant/common-constant';
import {
  CONFIRMATION,
  REGISTER_UPDATE_EXECUTE,
  WHILE_INPUT,
} from '@/constants/dialog';
import { FormRef, InputValidation } from '@/models/validation';
import {
  bufferSize,
  dateRange,
  formatDate,
  required,
  numberSizeMinusEnable
} from '@/utils/validation';
import { FukaServiceMstListAPI } from '@/api/fuka-service-mst-api';
import { DialogOptions } from '@/models/dialog';
import { getScreenId } from '@/utils/screenIds';
import { KubunListAPI } from '@/api/kubun-kanri';
import { ResponseData_KUBUN_KANRI } from '@/models/service';
import { MESSAGE, MESSAGE_ERROR } from '@/constants/dialog';
import { carryDownRejiNo } from '@/utils/helper';

export default Vue.extend({
  components: { AppDatePicker, ServiceRegisterAddEditPhone },
  props: {
    customerData: {
      type: Object,
      default: () => ({}),
    },
    cancelStatus: {
      type: Boolean,
      default: false,
    },
    editMode: {
      type: Boolean,
      default: false,
    },
    serviceData: {
      type: Object as PropType<ServiceDataReponse>,
      required: true,
    },
    serviceDataList: {
      type: Array as PropType<ServiceApplicationResponse[]>,
      required: true,
    },
    listPhoneNumber: {
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
    banpoFlg: {
      type: Boolean,
      default: false
    },
  },
  data(): ApplicationServiceData {
    return {
      serviceForm: this.serviceData.service_data_form,
      serviceList: this.serviceData.list_service_data,
      showAddPhone: false,
      phoneData: null,
      valid: false,
      fukaServiceOptions: [],
      fukaServiceName: '',
      modeEditPhone: false,
      additionServiceNameRule: [
        (value) =>
          required(
            value,
            this.$t(
              'customerDetail.serviceRegister.addEditService.additionServiceName'
            )
          ),
      ],
      applicationDateRule: [
        (value) =>
          required(
            value,
            this.$t(
              'customerDetail.serviceRegister.addEditService.applicationDate'
            )
          ),
        (value) => formatDate(value),
      ],
      CHNumberRule: [
        (value) => numberSizeMinusEnable(value, 3),
      ],
      itemsSentByIpLineOperatorRule: [(value) => bufferSize(value, 150)],
      isDirty: false,
      protcol_list: [],
      protcol_kubun: "", // 表示プロトコル区分名
      protcol_kubun_value: "", // プロトコル区分の値
      createPermission: false,
      updatePermission: false,
      readPermission: false,
      createPermission_phone: false,
      numberForExcluding: [], // 本画面で採番した電番＋他画面で採番された電番配列
      routingNumberForExcluding: [], // 本画面で採番した電番＋他画面で採番されたルーティング電番配列
      motoRejiNo: null,
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
      so_sakusei_kubun: [] as ProtcolOptions[],
    };
  },
  computed: {
    ...mapState('customer', ['bbJuchu']),
    headers(): (DataTableHeader | { text: TranslateResult })[] {
      return [
        {
          text: this.t('No'),
          value: 'g_serial',
        },
        {
          text: this.t('IpPhoneNo'),
          value: 'ip_tel_no',
        },
        {
          text: this.t('businessPersonPhoneNo'),
          value: 'routing_tel_no',
        },
        {
          text: this.t('linkedNo'),
          value: 'himozuki_tel_no',
        },
        {
          text: this.t('RegistrationNo'),
          value: 'reji_no',
        },
        {
          text: this.t('representativeSection'),
          value: 'daihyo_kubun',
        },
        {
          text: this.t('numberNotificationSection'),
          value: 'bango_tsuchi_kubun',
        },
        {
          text: this.t('incomingCallsNumber'),
          value: 'chakushin_num',
        },
        {
          text: this.t('list104PhoneBook'),
          value: 'n104_annai_kubun',
        },
        {
          text: this.t('name'),
          value: 'kaisen_meigi',
        },
        {
          text: this.t('contractCancelAcceptDate'),
          value: 'kaiyaku_uketsuke_date',
        },
        {
          text: this.t('soShokiDataSakuseiKubun'),
          value: 'so_shoki_data_sakusei_kubun',
        },
        {
          text: this.t('edit'),
          value: 'edit',
        },
      ];
    },
    completionDateRule(): InputValidation {
      return [
        (value) => formatDate(value),
        dateRange(
          this.serviceForm.koji_kanryo_date,
          this.serviceForm.moshikomi_date,
          null
        ),
      ];
    },
    contractCancelDateRule(): InputValidation {
      return [
        (value) => formatDate(value),
        dateRange(
          this.serviceForm.kaiyaku_date,
          this.serviceForm.koji_kanryo_date,
          null
        ),
      ];
    },
    ADD() {
      return ADD;
    },
    EDIT() {
      return EDIT;
    },
    phoneList(): PhoneNumberResponse[]{
      return this.serviceData.list_service_data.filter((item) => item.g_serial == this.serviceData.service_data_form.serial_no)
    }
  },
  watch: {
    serviceForm: {
      deep: true,
      handler() {
        console.log('WATCH CHANGED', this.isDirty);
        this.isDirty = true;
      },
    },
  },
  created() {
    // 仮採番リストをコピーする
    this.numberForExcluding = Array.from(this.listNumberedTentative)
    this.routingNumberForExcluding = Array.from(this.listRoutingNumberedTentative)

    const service_name = this.editMode ? "ApplicationServiceChangeDialog" : "ApplicationServiceAdditionDialog"
    const phone_name = "ApplicationPhoneNumberAdditionDialog"
    const screenId = getScreenId(this.$route.name, service_name);
    const screenId_phone = getScreenId(this.$route.name, phone_name);
    this.createPermission = this.$$hasCreatePermission(screenId);
    this.updatePermission = this.$$hasUpdatePermission(screenId);
    this.readPermission = this.$$checkPermission(screenId);
    this.createPermission_phone = this.$$hasCreatePermission(screenId_phone);

    Promise.all([
      FukaServiceMstListAPI(screenId),
      KubunListAPI(screenId, "PROTCOL_KUBUN"),
      KubunListAPI(screenId, 'SO_DATA_SAKUSEI_KUBUN')
    ]).then((res) => {
        const fukaServiceList: FukaServiceOptions[] = []
        console.log("BB受注プロトコル区分:"+this.customerData.protcol_kubun)

        res[0].data.filter(({juchu_type}) => juchu_type === '1').forEach(item => {
          const fukaData: FukaServiceOptions = {
            value: item.fuka_service_cd, 
            text: item.fuka_service_mei, 
            protcol: item.protcol_kubun
          }
          if(item.fuka_service_cd == this.serviceForm.fuka_service_cd) {
            // 現在の登録済みのもは無条件で表示
            fukaServiceList.push(fukaData)
          }
          else if (this.checkFukaService(item)) {
            fukaServiceList.push(fukaData)
          }
        });
        // サービスコードの値で表示順ソート 降順に修正 2023/01/13
        fukaServiceList.sort(function(a, b) {
          return Number(a.value) < Number(b.value) ? 1 : -1
        })
        this.fukaServiceOptions = fukaServiceList;
        this.protcol_list = this.kubunToSelect(res[1].data);
        this.so_sakusei_kubun = this.kubunToSelect(res[2].data)
        this.changeFukaService(this.serviceForm.fuka_service_cd);
      })
      .finally(() => {
        this.isDirty = false;
      });
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
    confirm(): void {
      const valid = (this.$refs.form as FormRef).validate();
      if (valid) {
        if(PROTCOL_KUBUN_SPECIAL == this.serviceForm.protocol_kubun &&
         0 != this.serviceForm.ch_num )
        {
          this.$store.dispatch('dialog/open', {
            type: MESSAGE,
            subType: MESSAGE_ERROR,
            messages: ERRMSG_PROTCOL_KUBUN_SPECIAL_CH_NUM,
          } as DialogOptions)
          return
        }
        this.$store.dispatch('dialog/open', {
          type: CONFIRMATION,
          subType: REGISTER_UPDATE_EXECUTE,
          messages: [this.t(`messages.${this.editMode ? 'change' : 'add'}`)],
          // データ受け取り
          callback: () => {
            // 編集ステータスの設定
            if (!this.editMode) {
              // add時
              this.serviceForm.editStatus = ADD;
            } else {
              // edit時 ステータスがaddになっていないときのみeditに設定
              if (this.serviceForm.editStatus != ADD) {
                this.serviceForm.editStatus = EDIT;
              }
            }
            if( "" == this.customerData.protcol_kubun )
            {
              if( PROTCOL_KUBUN_SPECIAL != this.serviceForm.protocol_kubun)
              {
                // BB受注のプロトコル区分がなし、かつ登録／変更サービスのプロトコル区分が9以外の場合、更新する
                this.customerData.protcol_kubun = this.serviceForm.protocol_kubun
              }
              if( PROTCOL_KUBUN_050 == this.serviceForm.protocol_kubun)
              {
                if(PROTCOL_KUBUN_050_CHECK_SERVICE_CD_LIST.includes(this.serviceForm.fuka_service_cd.substr(0,1)))
                {
                  this.customerData.ryokin_plan_kubun = PROTCOL_KUBUN_050_SPECIAL_RYOKIN_KUBUN
                }
                this.customerData.uf_toroku_data_sakusei_kubun = PROTCOL_KUBUN_050_SPECIAL_UF_TOUROKU
              }
            }
            this.$emit('accept', this.serviceForm, this.serviceData.list_service_data)
          },
        });
      }
    },
    edit(item: PhoneNumberResponse): void {
      this.modeEditPhone = true;
      this.motoRejiNo = item.reji_no
      this.showAddPhone = true;
      let itemClone: PhoneNumberResponse = Object.assign({}, item);
      itemClone.editStatus = EDIT;
      itemClone.update_flg = FLAG_UPDATED;
      this.phoneData = itemClone;
    },
    t(field: string): TranslateResult {
      return this.$t(`customerDetail.serviceRegister.addEditService.${field}`);
    },
    addPhone(): void {
      this.phoneData = null;
      let initData: PhoneNumberResponse = {
        id: (new Date()).getTime() + "",
        gno: this.customerData.gno,
        g_serial: this.serviceForm.serial_no,
        uf_cd: this.customerData.uf_cd,
        kanji: this.customerData.seti_kanji_shamei,
        furigana: this.customerData.seti_kn,
        ip_ashk_tel_no: '',
        ip_tel_no: '',
        routing_tel_no: '',
        routing_ashk_tel_no: '',
        reji_no: null,
        daihyo_kubun: '',
        bango_tsuchi_kubun: '',
        himozuki_change: false,
        himozuki_tel_no: '',
        himozuki_ashk_tel_no: '',
        chakushin_num: '',
        n104_annai_kubun: '',
        kaisen_meigi: this.customerData.seti_kanji_shamei,
        kaisen_meigi_kana: this.customerData.seti_kn,
        keisai_mei: '',
        keisai_mei_kana: '',
        ntt_np: '',
        ntt_np_date: '',
        juchu_kubun: '1',
        kaiyaku_uketsuke_date: '',
        so_kaiyaku_data_sakusei_kubun: '0',
        so_shoki_data_sakusei_kubun: '0',
        editStatus: ADD,
        service_cd: this.serviceForm.fuka_service_cd,
        add_flg: FLAG_ADDED,
        update_flg: FLAG_NORMAL,
        hold_flg: FLAG_NORMAL,
        old_ip_ashk_tel_no: '',
        old_himozuki_tel_no: '',
      }

      this.phoneData = initData;
      this.modeEditPhone = false;
      this.motoRejiNo = null
      this.showAddPhone = true;
    },
    changeFukaService(fuka_cd: string){
       this.serviceForm.fuka_service_mei = this.fukaServiceOptions.filter(item => item.value === fuka_cd).map(item => item.text)[0];
       this.serviceForm.protocol_kubun = this.fukaServiceOptions?.filter(item => item.value === fuka_cd).map(item => item.protcol)[0];
       this.protcol_kubun = this.protcol_list.filter(item => item.value === this.serviceForm.protocol_kubun).map(item => item.text)[0];
       this.protcol_kubun_value = this.protcol_list.filter(item => item.value === this.serviceForm.protocol_kubun).map(item => item.value)[0];
    },
    confirmRegisterPhone(data: PhoneNumberResponse): void {
      console.log(data)
      if (data.reji_no != null){
        let carryDownList = carryDownRejiNo(data.reji_no,data.id,this.motoRejiNo,this.serviceData.list_service_data,this.customerData.board_num)
        for (let i=0; this.listPhoneNumber.length>i; i++) {
          this.serviceData.list_service_data[i] = carryDownList[i]
        }
      }
      if(data.editStatus == ADD){
        const dataCreate: PhoneNumberResponse = {
          id: data.id,
          gno: data.gno,
          uf_cd: data.uf_cd,
          kanji: data.kanji,
          furigana: data.furigana,
          g_serial: this.serviceForm.serial_no,
          ip_tel_no: data.ip_tel_no,
          ip_ashk_tel_no: data.ip_tel_no.replace(/-/g, ""),
          routing_tel_no: data.routing_tel_no,
          routing_ashk_tel_no: data.routing_tel_no.replace(/-/g, ""),
          himozuki_tel_no: data.himozuki_tel_no,
          himozuki_ashk_tel_no: data.himozuki_tel_no?.replace(/-/g, ""),
          himozuki_change: data.himozuki_change,
          reji_no: data.reji_no ? Number(data.reji_no) : null,
          daihyo_kubun: data.daihyo_kubun,
          bango_tsuchi_kubun: data.bango_tsuchi_kubun,
          chakushin_num: data.chakushin_num + "",
          n104_annai_kubun: data.n104_annai_kubun,
          ntt_np: data.ntt_np,
          ntt_np_date: data.ntt_np_date,
          juchu_kubun: data.juchu_kubun,
          kaisen_meigi: data.kaisen_meigi,
          kaisen_meigi_kana: data.kaisen_meigi_kana,
          keisai_mei: data.keisai_mei,
          keisai_mei_kana: data.keisai_mei_kana,
          kaiyaku_uketsuke_date: "",
          so_kaiyaku_data_sakusei_kubun: "",
          so_shoki_data_sakusei_kubun: data.so_shoki_data_sakusei_kubun,
          update_flg: FLAG_NORMAL,
          add_flg: FLAG_ADDED,
          service_cd: data.service_cd,
          editStatus: ADD,
          hold_flg: data.hold_flg,
          old_ip_ashk_tel_no: "",// 新規追加時は参照先無し
          old_himozuki_tel_no: data.himozuki_tel_no,
        };
        console.log(dataCreate)
        this.serviceData.list_service_data.push(dataCreate);
      }else{
        const confirmItem = this.serviceData.list_service_data.find(({ id }) => id === data.id);
        if (confirmItem) {
          confirmItem.g_serial = this.serviceForm.serial_no;
          confirmItem.ip_tel_no = data.ip_tel_no;
          confirmItem.ip_ashk_tel_no = data.ip_tel_no.replace(/-/g, "");
          confirmItem.routing_tel_no = data.routing_tel_no;
          confirmItem.routing_ashk_tel_no = data.routing_tel_no.replace(/-/g, "")
          confirmItem.reji_no = data.reji_no ? Number(data.reji_no) : null;
          confirmItem.himozuki_tel_no = data.himozuki_tel_no;
          confirmItem.himozuki_ashk_tel_no = data.himozuki_tel_no?.replace(/-/g, "")
          confirmItem.daihyo_kubun = data.daihyo_kubun;
          confirmItem.bango_tsuchi_kubun = data.bango_tsuchi_kubun;
          confirmItem.chakushin_num = data.chakushin_num + "";
          confirmItem.n104_annai_kubun = data.n104_annai_kubun;
          confirmItem.kaisen_meigi = data.kaisen_meigi;
          confirmItem.kaisen_meigi_kana = data.kaisen_meigi_kana;
          confirmItem.keisai_mei = data.keisai_mei;
          confirmItem.keisai_mei_kana = data.keisai_mei_kana;
          confirmItem.old_ip_ashk_tel_no = data.old_ip_ashk_tel_no;
        }
      }
      if( data.daihyo_kubun === DAIHYO_DENBAN)
      {
        this.serviceData.list_service_data.forEach((item)=>{
          if(data.id !== item.id)
          {
            item.daihyo_kubun = KO_DENBAN
            item.update_flg = FLAG_UPDATED
            console.log("子電番へ変更," + item.ip_ashk_tel_no)
          }
        })
      }
      this.showAddPhone = false
      // 仮採番を更新
      this.numberForExcluding = []
      this.routingNumberForExcluding = []
      this.serviceData.list_service_data.forEach((phone) => {
        this.numberForExcluding.push(phone.ip_tel_no)
        this.routingNumberForExcluding.push(phone.routing_tel_no)
      })
    },

    // 付加サービスコード判定
    checkFukaService(data: responseData_FukaServiceMst): boolean {
      if (data.protcol_kubun == '9') {
        return true
      }
      if (this.customerData.protcol_kubun != "" && data.protcol_kubun != this.customerData.protcol_kubun) {
        console.log("プロトコル区分不一致")
        return false;
      }
      if (this.serviceForm.serial_no == "1") {
        if (data.service_kubun != "1") {
          console.log("新規登録")
          return false;
        }
      }
      if (["50","51","52"].includes(this.customerData.ryokin_plan_kubun)) {
        if (data.protcol_kubun != PROTCOL_KUBUN_050) { 
          console.log("プロトコル区分5のみ")
          return false;
        }
      }
      if (["51","52"].includes(this.customerData.ryokin_plan_kubun)) {
        if (data.fuka_service_cd == "50010070") {
          console.log("50010070除外")
          return false;
        }
      } 
      const duplicate = ["10010000","90010000","50010070","50010072","25050200","25050230"]
      if (duplicate.includes(data.fuka_service_cd)) {
        if(this.editMode){
          const checkRes: boolean = this.serviceDataList.some(item => {
            return (item.serviceApplicationCode == data.fuka_service_cd) && (item.serviceApplicationCode !== this.serviceForm.fuka_service_cd)
          })
          if (checkRes) {
            console.log("変更時重複")
            return false;
          }
        }else {
          const checkRes: boolean = this.serviceDataList.some(item => {
            return item.serviceApplicationCode == data.fuka_service_cd
          })
          if (checkRes) {
            console.log("重複")
            return false;
          }
        }
      }
      if (data.fuka_service_cd == "25030100") {
        const checkRes: boolean = this.serviceDataList.some(item => {
          return item.serviceApplicationCode == data.fuka_service_cd && !item.contractCancellationDate
        })
        if (checkRes) {
          console.log("25030100解約日チェック")
          return false;
        }
      }
      if (data.fuka_service_cd == "25030200") {
        const checkRes: boolean = this.serviceDataList.some(item => {
          return item.serviceApplicationCode == "25030100" && !item.contractCancellationDate
        })
        if (!checkRes) {
          console.log("25030200解約日チェック")
          return false;
        }
      }
      if (data.fuka_service_cd == "25030400" || data.fuka_service_cd == "25030500") {
        const checkRes: boolean = this.serviceDataList.some(item => {
          return item.serviceApplicationCode == "25030300" && !item.contractCancellationDate
        })
        if (!checkRes) {
          console.log("25030300解約日チェック")
          return false;
        }
      }
      return true;
    },
    kubunToSelect(kubunList: ResponseData_KUBUN_KANRI[]): ProtcolOptions[] {
      return kubunList.map(({ vl, hyoji_mei }) => ({
        value: vl,
        text: hyoji_mei,
      }));
    },
    getSoStatus(status: string): TranslateResult{
      let so_sakusei_kubun = this.so_sakusei_kubun.filter((item: ProtcolOptions) => item.value === status)
      if (so_sakusei_kubun.length > 0) {
        return so_sakusei_kubun[0].text
      }
      return ''
    },
    blurCHNum() {
      if (!this.serviceForm.ch_num) {
        this.serviceForm.ch_num = 0;
      }
    }
  },
});
</script>
