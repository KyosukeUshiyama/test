<!--完了情報登録追加＆編集-->
<template>
  <v-form ref="form">
    <v-card>
      <v-card-title>
        {{ mode === ADD ? t('titleAdd') : t('titleEdit') }}
      </v-card-title>
      <v-card-text>
        <v-row>
          <v-col md="3" cols="12">
            <v-text-field
              v-model="informationServiceForm.gno"
              filled
              readonly
              :label="t('gno')"
            />
          </v-col>
          <v-col md="3" cols="12">
            <v-text-field
              v-model="informationServiceForm.gserial_no"
              filled
              readonly
              :label="t('gSerialNo')"
            />
          </v-col>
          <v-col md="3" cols="12">
            <v-text-field
              v-model="informationServiceForm.serial_number"
              filled
              readonly
              :label="t('serialNumber')"
            />
          </v-col>
          <v-col md="3" cols="12">
            <v-text-field
              v-model="informationServiceForm.uf_code"
              filled
              readonly
              :label="t('ufCode')"
            />
          </v-col>
        </v-row>
        <v-row align="center">
          <v-col md="6" cols="12">
            <v-select
              v-model="informationServiceForm.service_code"
              :items="serviceOptions"
              item-text="text"
              item-value="value"
              class="required-label"
              :rules="serviceCodeRule"
              :label="t('serviceName')"
              :readonly="mode === EDIT"
              :filled="mode === EDIT"
              :no-data-text="t('messages.notFound')"
              @input="onChangeServiceCode($event)"
            />
          </v-col>
          <v-col md="3" cols="12">
            <v-text-field
              v-model="informationServiceForm.service_code"
              filled
              readonly
              :label="t('serviceCode')"
            />
          </v-col>
        </v-row>
        <v-row>
          <v-col md="6" cols="12">
            <app-date-picker
              v-model="informationServiceForm.billing_start_date"
              :label="t('billingStartDate')"
              :rules="billingStartDateRule"
              :required="Boolean(informationServiceForm.billing_end_date)"
            />
          </v-col>
          <v-col md="6" cols="12">
            <app-date-picker
              v-model="informationServiceForm.billing_end_date"
              :label="t('billingEndDate')"
              :rules="billingEndDateRule"
            />
          </v-col>
        </v-row>
        <v-row>
          <v-col md="6" cols="12">
            <v-select
              v-model="informationServiceForm.additional_categories"
              :items="additionalCategoriesOptions"
              item-text="text"
              item-value="value"
              class="required-label"
              :rules="additionalCategoriesRule"
              :label="t('additionalCategories')"
              :loading="loadingOptions"
              :no-data-text="t('messages.notFound')"
            />
          </v-col>
          <v-col md="6" cols="12">
            <v-select
              v-model="informationServiceForm.agency_target_category"
              :items="agencyTargetCategoryOptions"
              item-text="text"
              item-value="value"
              class="required-label"
              :rules="agencyTargetCategoryRule"
              :label="t('agencyTargetCategory')"
              :loading="loadingOptions"
              :no-data-text="t('messages.notFound')"
            />
          </v-col>
        </v-row>
        <v-row>
          <v-col cols="12">
            <v-textarea
              v-model="informationServiceForm.target_routing_number"
              :rules="targetRoutingNumberRule"
              :class="{
                'required-label': ['10010200', '10010300'].includes(
                  informationServiceForm.service_code
                ),
              }"
              rows="2"
              :label="t('targetRoutingNumber')"
            />
          </v-col>
        </v-row>
      </v-card-text>
      <v-card-actions class="justify-space-between">
        <v-btn :disabled="loadingConfirm" @click="close">
          {{ t('btnClose') }}
        </v-btn>
        <v-btn
          :loading="loadingConfirm"
          :disabled="mode === ADD ? !createPermission : !updatePermission"  
          color="primary"
          @click="confirm"
        >
          {{ t(mode === ADD ? 'btnAdd' : 'btnChange') }}
        </v-btn>
      </v-card-actions>
    </v-card>
  </v-form>
</template>

<script lang="ts">
import Vue, { PropType } from 'vue';
import AppDatePicker from '../UI/AppDatePicker.vue';
import {
  SelectOption,
  InformationServiceData,
  InformationServiceDataReponse,
  BbJuchuOptionDataForm,
  InformationServiceDataForm,
} from '@/models/completion-information';
import { CONFIRMATION, REGISTER_UPDATE_EXECUTE, WHILE_INPUT } from '@/constants/dialog';
import { TranslateResult } from 'vue-i18n';
import {
  addKanryoJohoAPI,
  updateKanryoJohoAPI,
  getKanryoJohoList,
  getBbJuchuOptionAPI,
  BillingUkewatashiFileData,
  UpdateBillingUkewatashiFile,
} from '../../api/CompletionInfomation-api';
import { AxiosError } from 'axios';
import { FormRef, InputValidation } from '@/models/validation';
import {
  bufferSize,
  formatDate,
  dateRange,
  required,
  conditionRequired,
  numberSize
} from '@/utils/validation';
import { DialogOptions } from '@/models/dialog';
import { FukaServiceMstListAPI } from '@/api/fuka-service-mst-api';
import { ResponseData_KUBUN_KANRI,} from '@/models/service';
import { KubunListAPI } from '@/api/kubun-kanri';
import { ADD, EDIT } from '@/models/service-register';
import { mapState } from 'vuex';
import { BbJuchuOption } from '@/models/store';
import { InfoComplete } from '@/models/info-register-complete';
import { getScreenId } from '@/utils/screenIds';
import moment from 'moment';

export default Vue.extend({
  components: { AppDatePicker },
  props: {
    mode: {
      type: String as PropType<typeof ADD | typeof EDIT>,
      required: true,
    },
    informationServiceData: {
      type: Object as PropType<InformationServiceDataReponse>,
      required: true,
    },
    completeList: {
      type: Array as PropType<InfoComplete[]>,
      default: []
    },
    maxGnoSerialNo: {
      type: Number,
      default: 1,
    }
  },
  data(): InformationServiceData {
    const { additional_categories, agency_target_category } =
      this.informationServiceData.information_service_data_form;
    return {
      informationServiceForm: {
        ...this.informationServiceData.information_service_data_form,
        additional_categories:
          this.mode === ADD ? '2' : additional_categories,
        agency_target_category:
          this.mode === ADD ? '1' : agency_target_category,
      },
      serviceOptions: [],
      serviceName: '',
      serviceCodeRule: [
        (value) =>
          required(
            value,
            this.$t(
              'customerDetail.completeInfoRegister.addEditCompleteInfo.serviceCode'
            )
          ),
      ],
      additionalCategoriesRule: [
        (value) =>
          required(
            value,
            this.$t(
              'customerDetail.completeInfoRegister.addEditCompleteInfo.additionalCategories'
            )
          ),
      ],
      agencyTargetCategoryRule: [
        (value) =>
          required(
            value,
            this.$t(
              'customerDetail.completeInfoRegister.addEditCompleteInfo.agencyTargetCategory'
            )
          ),
      ],
      additionalCategoriesOptions:[],
      agencyTargetCategoryOptions:[],

      loadingOptions:false,
      loadingConfirm: false,
      isDirty: false,
      createPermission: false,
      updatePermission: false,
    };
  },
  
  computed: {
    ...mapState('customer', ['bbJuchuOption']),
    billingStartDateRule(): InputValidation {
      return [
        conditionRequired(this.informationServiceForm.billing_start_date, Boolean(this.informationServiceForm.billing_end_date), this.t('billingEndDate'), this.t('billingStartDate')),
        (value) => bufferSize(value, 10),
        (value) => formatDate(value),
        dateRange(
          this.informationServiceForm.billing_start_date,
          null,
          this.informationServiceForm.billing_end_date
        ),
      ];
    },
    billingEndDateRule(): InputValidation {
      return [
        (value) => bufferSize(value, 10),
        (value) => formatDate(value),
        dateRange(
          this.informationServiceForm.billing_end_date,
          this.informationServiceForm.billing_start_date,
          null
        ),
      ];
    },
    targetRoutingNumberRule(): InputValidation {
      const { service_code, target_routing_number } =
        this.informationServiceForm;
      return [
        (value) => numberSize(value, 11),
        conditionRequired(
          target_routing_number,
          ['10010200', '10010300'].includes(service_code),
          this.t('serviceCode') + "で10010200または10010300",
          this.t('targetRoutingNumber')
        ),
      ];
    },
    ADD() {
      return ADD;
    },
    EDIT() {
      return EDIT;
    }
  },
  watch: {
    informationServiceForm: {
      deep: true,
      handler() {
        this.isDirty = true;
      },
    },
  },
  created()
  {
    this.loadingOptions = true;
    if (this.mode === ADD) {
      this.informationServiceForm.gserial_no = ('0000' + this.maxGnoSerialNo.toString()).slice(-4)
    }
    const name = this.mode === ADD ? "CompletionInformationAdditionDialog" : "CompletionInformationChangeDialog"
    const screen_id = getScreenId(this.$route.name, name)
    this.createPermission = this.$$hasCreatePermission(screen_id)
    this.updatePermission = this.$$hasUpdatePermission(screen_id)
    Promise.all([
      KubunListAPI(screen_id, 'SHINKI_KUBUN'),
      KubunListAPI(screen_id, 'SEIKYU_KUBUN'),
      FukaServiceMstListAPI(screen_id),
      ])
      .then((res) => {
        this.additionalCategoriesOptions = this.kubunToSelect(res[0].data);
        this.agencyTargetCategoryOptions = this.kubunToSelect(res[1].data);
        let options = res[2].data
        // 追加の時のみフィルターをかける
        // 変更の時はメディア作成区分が0以外のものも表示される必要があるため
        if (this.mode === ADD) {
          options = options.filter(({ media_sakusei_kubun }) => media_sakusei_kubun === '0')
        }
        this.serviceOptions = options
          .map(({ fuka_service_cd, fuka_service_mei }) => {
            let disabled = false;
            const records_25030200 = this.completeList.filter(
              ({ product, endDate }) => {
                return (
                  product === '25030200' &&
                  (!endDate ||
                    moment(new Date(endDate)).isAfter(new Date(), 'day'))
                );
              }
            );
            if (records_25030200.length && fuka_service_cd === '25030200')
              disabled = true;
            return {
              value: fuka_service_cd,
              text: fuka_service_mei,
              disabled,
            };
          });
      })
      .finally(()=>
      {
        this.loadingOptions = false;
        this.isDirty = false;
      })
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
    // サービス名変更時、データ上のサービス名も変更
    onChangeServiceCode(value: string): void {
      this.informationServiceForm.service_contents =  this.serviceOptions.filter(item => item.value == value).map(item => item.text)[0];
    },
    confirm(): void {
      const valid = (this.$refs.form as FormRef).validate();
      if (valid)
      {
        const bbJuchuOptionServiceCodes = (this.bbJuchuOption as BbJuchuOption[]).map((item: BbJuchuOption) => item.fuka_service_cd)
        // 選択したサービスコードが申し込まれているものか判定 変更時は除く
        const serviceCdIncluded = bbJuchuOptionServiceCodes.includes(this.informationServiceForm.service_code) || this.mode === EDIT
        
        this.$store.dispatch('dialog/open', {
          type: CONFIRMATION,
          subType: REGISTER_UPDATE_EXECUTE,
          messages: [this.t(`messages.${serviceCdIncluded ? 'confirm' : 'serviceCdNotInBbJuChuOption'}`)],
          autoClose: false,
          callback: async (close) => {
            await this.callKanryoJohoAPI(this.mode,this.informationServiceForm, close)
          }
        } as DialogOptions)
      }
    },
    // 区分管理をselectoptionに変更
    kubunToSelect(kubunList: ResponseData_KUBUN_KANRI[]): SelectOption[] {
      return kubunList.map(({ vl, hyoji_mei }) => ({
        value: vl,
        text: hyoji_mei,
      }));
    },
    // 完了情報登録・更新API呼出
    callKanryoJohoAPI(
          mode:string,
          srcData:InformationServiceDataForm,
          closePreviousDialog: () => void
      ):Promise<void>
    {
      console.log("完了情報登録－データ受け取り、更新");
      console.log(srcData);
      this.loadingConfirm = true;
      const screen_id = getScreenId(this.$route.name)
      if(ADD == mode)
      {
        // 追加時
        return Promise.all([
          getKanryoJohoList(screen_id, srcData.gno),
          getBbJuchuOptionAPI(screen_id, srcData.gno),
        ])
          .then((getRes) =>
          {
            // フォームを新しく定義してデータをセット
            let informationServiceFormData = srcData;
            // 最大のgserial_no + 1 を取得する
            informationServiceFormData.gserial_no = this.automaticNumberingGserialNo(getRes[0].data);
            // 最大のserial_no + 1 を取得する
            const serial_no = this.automaticNumberingSerialNo(
              getRes[1].data,
              informationServiceFormData.service_code
            );
            informationServiceFormData.serial_number = serial_no;

            if (serial_no === '0001') {
              closePreviousDialog()
              this.$store.dispatch('dialog/open', {
                type: CONFIRMATION,
                subType: REGISTER_UPDATE_EXECUTE,
                messages: [this.t(`messages.noServiceCode`)],
                autoClose: false,
                callback: (close) => {
                  return this.callAddKanryoJohoAPI(informationServiceFormData).finally(
                    () => close()
                  );
                },
              } as DialogOptions);
            } else {
              return this.callAddKanryoJohoAPI(informationServiceFormData).finally(
                    () => closePreviousDialog());
            }
          })
          .catch((err: AxiosError) => 
          {
            closePreviousDialog()
            console.log(err.message);
            this.$emit('confirm', this.informationServiceForm);
          })
          .finally(() => this.loadingConfirm = false);

      }
      else
      {
        // 更新時
        let reqData:BillingUkewatashiFileData = {
          gno: this.informationServiceForm.gno,
          // 連番、シリアル番号がnullの場合は最初の0001を設定する
          gno_serial_no: this.informationServiceForm.gserial_no==null ? "0001" : this.informationServiceForm.gserial_no,
          serial_no: this.informationServiceForm.serial_number==null ? "0001" : this.informationServiceForm.serial_number,
          fuka_service_cd: this.informationServiceForm.service_code==null ? "" : this.informationServiceForm.service_code,
          shohin_mei: this.informationServiceForm.service_contents==null ? "" : this.informationServiceForm.service_contents,
          routing_tel_no: this.informationServiceForm.target_routing_number==null ? "" : this.informationServiceForm.target_routing_number,
          shinki_kubun: this.informationServiceForm.additional_categories==null ? "" : this.informationServiceForm.additional_categories,
          seikyu_kubun: this.informationServiceForm.agency_target_category==null ? "" : this.informationServiceForm.agency_target_category,
          kakin_start_date: this.informationServiceForm.billing_start_date,
          kakin_end_date: this.informationServiceForm.billing_end_date,
        }
        const updateData: UpdateBillingUkewatashiFile = {
          all_update_flg: false,
          billingList: [reqData]
        }
        return updateKanryoJohoAPI(screen_id, updateData)
          .then((res) =>
          {
            console.log(res);
          })
          .catch((err: AxiosError) => {
            if (err.response) {
                alert(err.response.data.title +'\n' +
                "エラー内容: " + err.response.data.detail)
            }
          })
          .finally(()=>
          {
            this.$emit('confirm', this.informationServiceForm);
            closePreviousDialog()
          });
      } 
    },

    // 最大のgserial_no + 1 を取得する
    // 対象のgserial_noがない場合は'1'とする
    automaticNumberingGserialNo(data :BillingUkewatashiFileData[]): string {
      let gserial_no = "0001";
      if(null!=data)
      {
        data.forEach(element => {
          if (null!=element.gno_serial_no && Number(gserial_no) <= Number(element.gno_serial_no) )
          {
            // 10000件目は'0000'で登録し、10001件目から一意制約違反となる
            gserial_no = ('0000' + (Number(element.gno_serial_no) + 1 ).toString()).slice(-4)
          }
        });
      }
      console.log("automaticNumberingGserialNo:"+gserial_no);
      return gserial_no;
    },
    // サービス区分が一致するシリアルナンバーを取得
    // サービス区分が一致する対象がない場合は'0001'とする
    automaticNumberingSerialNo(data :BbJuchuOptionDataForm[], service_code: string): string {
      let serial_number = "0001";
      if (null!=data)
      {
        // サービス区分が一致した場合ループを抜ける
        // forEachだとループの途中で抜けれず余計な処理が入る為、forを使用
        for (const element of data ) {
          if (null!=element.fuka_service_cd && element.fuka_service_cd == service_code) {
            serial_number = element.serial_no;
            break;
          }
        }
      }
      console.log("automaticNumberingSerialNo:"+serial_number);
       return serial_number;
    },

    t(field: string): TranslateResult {
      return this.$t(
        `customerDetail.completeInfoRegister.addEditCompleteInfo.${field}`
      );
    },

    callAddKanryoJohoAPI(
      informationServiceFormData: InformationServiceDataForm
    ): Promise<void> {
      return Promise.all([addKanryoJohoAPI(getScreenId(this.$route.name), informationServiceFormData)])
        .then((res) => {
          console.log('完了情報登録完了');
          console.log(res);
        })
        .catch((err: AxiosError) => {
          if (err.response) {
            alert(err.response.data.title +'\n' +
            "エラー内容: " + err.response.data.detail)
          }
        })
        .finally(() => {
          this.$emit('confirm', this.informationServiceForm);
        });
    },
  },
});
</script>
