<!--完了情報登録-->
<template>
  <v-form ref="form" v-model="valid">
    <h3>{{ screenId }}: {{ t('screenName') }}</h3>
    <v-row>
      <v-col md="6" cols="12">
        <v-text-field
          v-model="internalCompleteData.kanji"
          :label="t('kanji')"
          readonly
          filled
        />
      </v-col>
      <v-col md="6" cols="12">
        <v-text-field
          v-model="internalCompleteData.furigana"
          :label="t('furigana')"
          readonly
          filled
        />
      </v-col>
    </v-row>
    <v-row>
      <v-col md="3" cols="12">
        <app-date-picker
          v-model="internalCompleteData.startDate"
          :label="t('startDate')"
          :rules="startDateRule"
          :required="!internalCompleteData.startDate && !internalCompleteData.endDate"
          @input="changeDate"
        />
      </v-col>
      <v-col md="3" cols="12">
        <app-date-picker
          v-model="internalCompleteData.endDate"
          :label="t('endDate')"
          :rules="endDateRule"
          :required="!internalCompleteData.startDate && !internalCompleteData.endDate"
          @input="changeDate"
        />
      </v-col>
    </v-row>
    <v-row>
      <v-col cols="12">
        <div class="d-flex justify-space-between">
          <h3>{{ t('infoCompleteList') }}</h3>
          <v-btn 
            color="primary"
            :disabled="!createPermission"
            @click="openDialogCompleteInfo('ADD')"
          >
            {{ t('addInfo') }}
          </v-btn>
        </div>
        <v-data-table
          :headers="headers"
          :items="internalCompleteData.infoCompleteList"
          :options.sync="tableOptions"
          hide-default-footer
          class="my-5"
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
            <p>{{ t('messages.notFound') }}</p>
          </template>
          <template v-slot:[`item.action`]="{ item }">
            <v-icon :disabled="!updatePermission" @click="openDialogCompleteInfo('EDIT', item)">
              mdi-pencil
            </v-icon>
          </template>
        </v-data-table>
        <div class="d-flex justify-end">
          <v-btn
            color="primary"
            class="mr-5"
            :disabled="!allUpdatePermission || updateButton"
            @click="ikkatuUpdateButton()"
          >
            {{ t('batchUpdateConfirm') }}
          </v-btn>
          <v-btn
            color="error"
            :disabled="!allUpdatePermission || updateButton"
            @click="ikkatuUpdateButton(true)"
          >
            {{ t('forceBatchUpdateConfirm') }}
          </v-btn>
        </div>
      </v-col>
    </v-row>
    <v-dialog v-model="showDialogCompletionInforChange" width="1200" persistent>
      <completion-information-add-edit
        v-if="showDialogCompletionInforChange"
        :mode="mode"
        :information-service-data="informationServiceDataCreate"
        :complete-list="completeData.infoCompleteList"
        :max-gno-serial-no="maxGnoSerialNo"
        @close="handleCloseDialogCompletionInforChange"
        @confirm="handleAddConfirm"
      />
    </v-dialog>
  </v-form>
</template>

<script lang="ts">
import Vue, { PropType } from 'vue';
import AppDatePicker from '../UI/AppDatePicker.vue';
import {
  CompleteData,
  InfoComplete,
  InfoRegisterCompleteData,
} from '@/models/info-register-complete';
import {
  CONFIRMATION,
  REGISTER_UPDATE_EXECUTE,
  WHILE_INPUT,
} from '@/constants/dialog';
import CompletionInformationAddEdit from '@/components/completion-information/CompletionInformationAddEdit.vue';
import { TranslateResult } from 'vue-i18n';
import { DataTableHeader } from 'vuetify';
import { InformationServiceDataForm } from '@/models/completion-information';
import {
  bufferSize,
  dateRange,
  formatDate,
  groupRequired
} from '@/utils/validation';
import { FormRef, InputValidation } from '@/models/validation';
import { getScreenId } from '@/utils/screenIds';
import
{
  getKanryoJohoList,
  updateKanryoJohoAPI,
  convBillingUkewatashiFileToKanryoJohoListOne,
  BillingUkewatashiFileData,
  UpdateBillingUkewatashiFile
} from '../../api/CompletionInfomation-api';
import
{
  KubunListAPI,
} from '../../api/kubun-kanri';
import { AxiosError } from 'axios';
import { DEFAULT_HYOJI_NUM } from '@/constant/common-constant';

export default Vue.extend({
  name: 'InfoRegisterComplete',
  components: { AppDatePicker, CompletionInformationAddEdit },
  props: {
    completeData: {
      type: Object as PropType<CompleteData>,
      required: true,
    },
  },
  data(): InfoRegisterCompleteData {
    return {
      valid: false,
      informationServiceDataCreate: null,
      showDialogCompletionInforChange: false,
      mode: '',
      internalCompleteData: this.completeData,
      maxGnoSerialNo: 0,
      updatePermission: false,
      createPermission: false,
      allUpdatePermission: false,
      updateButton: false,
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
    };
  },
  computed: {
    screenId(): string {
      return getScreenId(this.$route.name, this.$options.name);
    },
    headers(): (DataTableHeader | { text: TranslateResult })[] {
      return [
        {
          text: this.t('g_serial'),
          value: 'g_serial',      // 連番とは、DB項目としてはGシリアルNOを指す
        },
        {
          text: this.t('product'),
          value: 'product',
        },
        {
          text: this.t('productName'),
          value: 'productName',
        },
        {
          text: this.t('startDate'),
          value: 'startDate',
        },
        {
          text: this.t('endDate'),
          value: 'endDate',
        },
        {
          text: this.t('newAdded'),
          value: 'newAdded',
        },
        {
          text: this.t('billingSection'),
          value: 'billingSection',
        },
        {
          text: this.t('targetIpPhoneNumber'),
          value: 'targetIpPhoneNumber',
        },
        {
          text: this.t('edit'),
          value: 'action',
        },
      ];
    },
    startDateRule(): InputValidation {
      let message = String(this.t('startDate')) + 'または' + String(this.t('endDate'))
      return [
        groupRequired(!(!this.internalCompleteData.startDate && !this.internalCompleteData.endDate), message),
        (value) => bufferSize(value, 10),
        (value) => formatDate(value),
        dateRange(
          this.internalCompleteData.startDate,
          null,
          this.internalCompleteData.endDate
        ),
      ];
    },
    endDateRule(): InputValidation {
      let message = String(this.t('startDate')) + 'または' + String(this.t('endDate'))
      return [
        groupRequired(!(!this.internalCompleteData.startDate && !this.internalCompleteData.endDate), message),
        (value) => bufferSize(value, 10),
        (value) => formatDate(value),
        dateRange(
          this.internalCompleteData.endDate,
          this.internalCompleteData.startDate,
          null
        ),
      ];
    },
    gno(): string {
      const { gno } = this.$route.query;
      if (!gno) return '';
      if (Array.isArray(gno)) return gno[0] || '';
      return gno;
    },
    uf_cd(): string {
      const { uf_cd } = this.$route.query;
      if (!uf_cd) return '';
      if (Array.isArray(uf_cd)) return uf_cd[0] || '';
      return uf_cd;
    }
  },
  watch: {
    completeData: {
      deep: true,
      handler(value) {
        this.internalCompleteData = value;
      },
    },
  },
  created() {
    this.updateList();
    const update_name = "CompletionInformationChangeDialog"
    const create_name = "CompletionInformationAdditionDialog"
    const screen_ids = {
      "update": getScreenId(this.$route.name, update_name),
      "create": getScreenId(this.$route.name, create_name),
    }
    this.updatePermission = this.$$hasUpdatePermission(screen_ids["update"]);
    this.createPermission = this.$$hasCreatePermission(screen_ids["create"]);
    this.allUpdatePermission = this.$$hasUpdatePermission(getScreenId(this.$route.name, this.$options.name))
  },
  methods: {
    // 一括更新確認ボタン／強制一括更新確認ボタン押下処理
    //  引数:true 強制
    ikkatuUpdateButton(force = false): void
    {
      console.log("一括更新確認押下,"+force+","+ this.internalCompleteData.startDate + "," + this.internalCompleteData.endDate);
      const valid = (this.$refs.form as FormRef).validate();
      if (valid) {
        const messages = [
          this.t(`messages.${force ? 'forceUpdate' : 'update'}`),
        ];
        this.$store.dispatch('dialog/open', {
          type: CONFIRMATION,
          subType: REGISTER_UPDATE_EXECUTE,
          messages,
          callback: () =>
          {
            console.log("一括更新ボタン－OK");
            this.updateAllDate(force);
          },
        });
        this.$emit('dirty', false)
      }
    },
    // 日付一括更新処理
    //  引数:true 強制
    updateAllDate(force:boolean):void
    {
      this.updateButton = true;
      let reqData:BillingUkewatashiFileData[] = []

      this.internalCompleteData.infoCompleteList.forEach(element =>
      {
        let kousinUmu = force;  // 更新対象か
        // 入力されていても更新しない場合があるのでデフォルト空文字で定義
        let startDate = '' // 課金開始日
        let endDate = '' // 課金終了日

        // 強制一括の場合は全レコード更新
        if (force == true) {
          startDate = this.internalCompleteData.startDate
          // 対象データの開始日が登録済み || 開始日が入力されている場合に更新
          if (!(null == element.startDate || "" == element.startDate) || this.internalCompleteData.startDate){
            endDate = this.internalCompleteData.endDate
          }
        } else {
          // 対象データの開始日が空で開始日が入力されていたら更新
          if( (null ==element.startDate || "" == element.startDate) && this.internalCompleteData.startDate)
          {
            kousinUmu = true;
            startDate = this.internalCompleteData.startDate
          }
          // 終了日が入力されてる && 開始日が入力されていない場合は、対象データの開始日が入力されている && 対象データの終了日が入力されていなければ更新
          if(this.internalCompleteData.endDate && !this.internalCompleteData.startDate && (null == element.endDate || "" == element.endDate) && !(null ==element.startDate || "" == element.startDate))
          {
            kousinUmu = true;
            endDate = this.internalCompleteData.endDate
          }
          // 終了日が入力されてる && 開始日が入力されてる場合は、対象データの終了日が入力されていなければ更新
          if(this.internalCompleteData.endDate && this.internalCompleteData.startDate && (null ==element.endDate || "" == element.endDate))
          {
            kousinUmu = true;
            endDate = this.internalCompleteData.endDate
          }
        }
        if(kousinUmu == false)
        {
          return; // ループ元へ戻る
        }

        const temp:BillingUkewatashiFileData = {
          gno:this.gno,
          // 連番、シリアル番号がnullの場合は最初の0001を設定する
          gno_serial_no:element.serial==null?"0001":element.g_serial,
          serial_no:element.g_serial==null?"0001":element.serial,
          fuka_service_cd:element.product==null?"":element.product,
          shohin_mei:element.productName==null?"":element.productName,
          routing_tel_no:element.targetIpPhoneNumber==null?"":element.targetIpPhoneNumber,
          shinki_kubun:element.newAddedType==null?"":element.newAddedType,
          seikyu_kubun:element.billingSectionType==null?"":element.billingSectionType,
          kakin_start_date:startDate,
          kakin_end_date:endDate,
        }

        reqData.push(temp)
      });

      if(reqData.length>0)
      {
        console.log("更新対象数,"+reqData.length);
        const updateData: UpdateBillingUkewatashiFile = {
          all_update_flg: true,
          billingList: reqData
        }
        // 更新API呼び出し
        updateKanryoJohoAPI(getScreenId(this.$route.name, this.$options.name), updateData)
        .catch((err: AxiosError) => {
          if (err.response) {
            alert(err.response.data.title +'\n' +
            "エラー内容: " + err.response.data.detail)
          }
        }).finally(()=>
        {
          // 全て終了時、リストを更新
          console.log("全ての実施完了");
          this.updateList();
          this.$emit(
            force ? 'force-update' : 'update',
            this.internalCompleteData
            );
          this.updateButton = false;
        })
      }
      else
      {
        // 更新対象が無い場合
        console.log("更新対象-なし");
        this.$emit(
          force ? 'force-update' : 'update',
          this.internalCompleteData
          );
        this.updateButton = false;
      }
    },

    // ダイアログ呼び出し時データセット
    //  完了情報追加ダイアログ／完了情報変更ダイアログ
    openDialogCompleteInfo(mode: string, item?: InfoComplete): void {
      this.mode = mode;
      this.showDialogCompletionInforChange = true;
      this.informationServiceDataCreate = {
        information_service_data_form: {
          gno: this.gno,
          serial_number: item?.serial || '',
          gserial_no: item?.g_serial || '',
          uf_code: this.uf_cd,
          service_code: item?.product || '',
          service_contents: item?.productName || '',
          target_routing_number: item?.targetIpPhoneNumber || '',
          additional_categories: item?.newAddedType || '',
          agency_target_category: item?.billingSectionType || '',
          billing_start_date: item?.startDate || '',
          billing_end_date: item?.endDate || '',
        },
      };
    },
    handleCloseDialogCompletionInforChange(dirty: boolean): void {
      const close = (): void => {
        this.showDialogCompletionInforChange = false;
      };
      if (dirty) {
        this.$store.dispatch('dialog/open', {
          type: CONFIRMATION,
          subType: WHILE_INPUT,
          callback: close,
        });
      } else {
        close();
      }
    },
    // 完了情報追加ダイアログからデータを受け取りダイアログを閉じる
    handleAddConfirm(data: InformationServiceDataForm): void {
      console.log(data);
      this.showDialogCompletionInforChange = false;
      this.updateList();
      this.$emit('dirty', false)
    },

    // リスト内容更新
    updateList():void
    {
      this.$$openLoadingScreen()
      console.log("完了情報－リスト情報更新処理");
      this.completeData.infoCompleteList = [];
      // 完了情報検索
      const screen_id = getScreenId(this.$route.name, this.$options.name)
      Promise.all([
        getKanryoJohoList(screen_id, this.$route.query.gno.toString()),
        KubunListAPI(screen_id, 'SHINKI_KUBUN'),
        KubunListAPI(screen_id, 'SEIKYU_KUBUN'),
        ]).then((res) =>
        {
          console.log(res[0])
          if(null!=res[0].data)
          {
            res[0].data.forEach(element =>
            {
              // 連番最大値を取得
              if (this.maxGnoSerialNo <= Number(element.gno_serial_no)) {
                this.maxGnoSerialNo = Number(element.gno_serial_no) + 1;
              }
              this.completeData.infoCompleteList.push(
                convBillingUkewatashiFileToKanryoJohoListOne(
                  element,
                  res[1].data,
                  res[2].data,
                ));
            });
          }
        })
          .catch((err: AxiosError) => {
            console.log(err.message)
        }).finally(()=> this.$$closeLoadingScreen());

    },
    t(field: string): TranslateResult {
      return this.$t(`customerDetail.completeInfoRegister.${field}`);
    },
    changeDate(): void{
      this.$emit('dirty', true)
    }
  },
});
</script>
