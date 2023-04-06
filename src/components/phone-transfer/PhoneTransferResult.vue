<!--電番移行結果-->
<template>
  <v-form ref="form" class="mt-10">
    <v-data-table
      v-model="selected"
      show-select
      :headers="headers"
      :items="searchTransferPhoneList"
      item-key="ip_tel_no"
      :options.sync="tableOptions"
      hide-default-footer
      class="phone-transfer-result-table"
      :item-class="() => 'phone-transfer-result-row'"
    >
      <template v-slot:top="{ pagination, options, updateOptions }">
        <v-data-footer
          :pagination="pagination"
          :options="options"
          :items-per-page-options="[5, 10, 15, 20, -1]"
          items-per-page-text="Rows per page"
          @update:options="updateOptions"
        />
      </template>
      <template #no-data>
        <p>{{ t('messageNotFound') }}</p>
      </template>
      <template v-slot:[`item.daihyo_kubun`]="{ item }">
        <v-select
          v-model="item.daihyo_kubun"
          :items="daihyo_kubun"
          :no-data-text="t('optionNotFound')"
          @input="fieldChange"
        />
      </template>
      <template v-slot:[`item.bango_tsuchi_kubun`]="{ item }">
        <v-select
          v-model="item.bango_tsuchi_kubun"
          :items="bango_tsuchi_kubun"
          :no-data-text="t('optionNotFound')"
          @input="fieldChange"
        />
      </template>
      <template v-slot:[`item.chakushin_num`]="{ item }">
        <v-text-field v-model="item.chakushin_num" :rules="chakushinNumRule" @input="fieldChange" />
      </template>
      <template v-slot:[`item.kaisen_meigi`]="{ item }">
        <v-text-field 
          v-model="item.kaisen_meigi" 
          :rules="KaisenMeigiRules"
          @input="fieldChange"
        />
      </template>
      <template v-slot:[`item.kaisen_meigi_kana`]="{ item }">
        <v-text-field 
          v-model="item.kaisen_meigi_kana"
          :rules="KaisenMeigiKanaRules"
          @input="fieldChange"
        />
      </template>
      <template v-slot:[`item.non_bb_target_phone`]="{ item }">
        <v-text-field 
          v-model="item.non_bb_target_phone" 
          :rules="NonBBTargetPhoneRules"
          @input="fieldChange"
        />
      </template>
      <template v-slot:[`item.n104_annai_kubun`]="{ item }">
        <v-select
          v-model="item.n104_annai_kubun"
          :items="n104_annai_kubun"
          :no-data-text="t('optionNotFound')"
          @input="fieldChange"
        />
      </template>
    </v-data-table>
    <div class="d-flex justify-end my-5">
      <v-btn
        color="primary"
        :disabled="!updatePermission || !checkSelected()"
        :loading="loading"
        @click="confirm"
      >
        {{ t('btnConfirmDataCreation') }}
      </v-btn>
    </div>
  </v-form>
</template>

<script lang="ts">
import Vue, { PropType } from 'vue';
import { DataTableHeader } from 'vuetify';
import {
  SearchResultState,
  TransferPhoneResultItem,
} from '@/models/transfer-phone';
import { SelectOption, DAIHYO_DENBAN, JUCHU_STATUS } from '@/models/service-register';
import { CustomerServiceResponse } from '@/models/customer';
import { TranslateResult } from 'vue-i18n';
import { CONFIRMATION, MESSAGE, REGISTER_UPDATE_EXECUTE } from '@/constants/dialog';
import { bufferSize, formatTelNo, numberSize, formatHalfWidth } from '@/utils/validation';
import { FormRef } from '@/models/validation';
import { DialogOptions } from '@/models/dialog';
import { KubunListAPI, ResponseData_KUBUN_KANRI } from '@/api/kubun-kanri';
import { AxiosError } from 'axios';
import {
  DenbanIkoOneData,
  DenbanIkoRequestData,
  RequestDenbanIko,
} from '@/api/transfer-phone-api';
import { DEFAULT_HYOJI_NUM } from '@/constant/common-constant';

export default Vue.extend({
  props: {
    searchTransferPhoneList: {
      type: Array as PropType<TransferPhoneResultItem[]>,
      default: [],
    },
    customerDetail: {
      type: Object as PropType<CustomerServiceResponse>,
      default: null,
    },
    newSearch: {
      type: Boolean,
    },
    motoGno:{
      type: String,
      default: null,
    },
    motoJuchuNo:{
      type: String,
      default: null,
    },
    sakiSerialNo:{
      type: String,
      default: '',
    },
    motoJuchuMeisaiNo:{
      type: String,
      default: null,
    },
    screenId: {
      type: String,
      default: null,
    }
  },

  data(): SearchResultState {
    return {
      singleSelect: true,
      selected: [],
      tableOptions: {
        page: 1,
        itemsPerPage: DEFAULT_HYOJI_NUM,
        sortBy: [],
        sortDesc: [],
        groupBy: [],
        groupDesc: [],
        multiSort: false,
        mustSort: false,
      },
      tableWrapper: null,
      KaisenMeigiRules: [(value) => bufferSize(value, 512)],
      KaisenMeigiKanaRules: [
        (value) => bufferSize(value, 512),
        (value) => formatHalfWidth(value, this.$t('customerDetail.serviceRegister.addEditPhone.lineNameKana'))
      ],
      NonBBTargetPhoneRules: [(value) => formatTelNo(value)],
      chakushinNumRule: [(value) => numberSize(value, 2)],
      gno_of_transfer_source: this.motoGno,
      daihyo_kubun: [] as SelectOption[],
      bango_tsuchi_kubun: [] as SelectOption[],
      n104_annai_kubun: [] as SelectOption[],
      updatePermission: false,
      loading: false,
    };
  },
  computed: {
    headers(): (DataTableHeader | { text: TranslateResult })[] {
      return [
        {
          text: '',
          value: 'data-table-select',
          width: '65px',
          align: 'center',
          class: 'phone-transfer-result-row__checkbox',
          cellClass: 'phone-transfer-result-row__checkbox',
        },
        {
          text: this.t('headerTable.detailsNo'),
          value: 'serial_no',
          width: '100px',
          class: 'phone-transfer-result-row__serial-no',
          cellClass: 'phone-transfer-result-row__serial-no',
        },
        {
          text: this.t('headerTable.phoneNumber'),
          value: 'ip_tel_no',
          class: 'phone-transfer-result-row__phone-no',
          cellClass: 'phone-transfer-result-row__phone-no',
          width: '170px',
        },
        {
          text: this.t('headerTable.routingNumber'),
          value: 'routing_tel_no',
          width: '170px',
        },
        {
          text: this.t('headerTable.representativeSection'),
          value: 'daihyo_kubun',
          width: '170px',
        },
        {
          text: this.t('headerTable.callNumberNotification'),
          value: 'bango_tsuchi_kubun',
          width: '170px',
        },
        {
          text: this.t('headerTable.numberOfIncomingCalls'),
          value: 'chakushin_num',
          width: '120px',
        },
        {
          text: this.t('headerTable.section104'),
          value: 'n104_annai_kubun',
          width: '260px',
        },
        {
          text: this.t('headerTable.lineName'),
          value: 'kaisen_meigi',
          width: '260px',
        },
        {
          text: this.t('headerTable.lineNameKana'),
          value: 'kaisen_meigi_kana',
          width: '260px',
        },
        {
          text: this.t('headerTable.nonBBTargetPhoneNo'),
          value: 'non_bb_target_phone',
          width: '170px',
        },
        {
          text: this.t('headerTable.contractCancelDate'),
          value: 'phone_no_non_bb',
          width: '120px',
        },
        {
          text: this.t('headerTable.deleteFlag'),
          value: 'sakujo_flg',
        },
      ];
    },
  },
  watch: {
    newSearch(value) {
      if (value) this.tableOptions.page = 1;
    },
    motoGno: {
      immediate: true,
      deep: true,
      handler: function(){
        this.gno_of_transfer_source = this.motoGno;
      }
    }
  },
  created() {
    const screen_id = this.screenId;
    this.updatePermission = this.$$hasUpdatePermission(screen_id);
    // 区分管理から各種区分を取得
    Promise.all([
      KubunListAPI(screen_id, 'DAIHYO_KUBUN'), // 代表電番区分
      KubunListAPI(screen_id, 'BANGO_TSUCHI_KUBUN'), // 番号通知区分
      KubunListAPI(screen_id, 'N104_ANNAI_KUBUN'), // N104案内区分
    ])
      .then((res) => {
        this.daihyo_kubun = this.kubunToSelect(res[0].data);
        this.bango_tsuchi_kubun = this.kubunToSelect(res[1].data);
        this.n104_annai_kubun = this.kubunToSelect(res[2].data);
      })
      .catch((err: AxiosError) => {
        console.error(err.response);
      });
  },
  mounted() {
    const table = document.getElementsByClassName(
      'phone-transfer-result-table'
    )[0];
    this.tableWrapper = table?.getElementsByClassName(
      'v-data-table__wrapper'
    )[0];
    this.tableWrapper?.addEventListener('scroll', this.handleScrollTable);
  },
  destroyed() {
    this.tableWrapper?.removeEventListener('scroll', this.handleScrollTable);
  },
  methods: {
    t(field: string): TranslateResult {
      return this.$t(`customerDetail.phoneTransfer.${field}`);
    },
    resetSelectedList(): void {
      this.selected = []
    },
    checkSelected(): boolean {
      let result = false
      if(this.selected?.length>0)
      {
        result = true
      }
      return result
    },
    handleScrollTable(e: Event): void {
      const target = e.target as HTMLDivElement;
      const borderCells = target.getElementsByClassName(
        'phone-transfer-result-row__phone-no'
      );
      if (target.scrollLeft !== 0) {
        for (const cell of borderCells) {
          cell.classList.add('border-on-scroll');
        }
      } else {
        for (const cell of borderCells) {
          cell.classList.remove('border-on-scroll');
        }
      }
    },
    // データ作成確認ボタン押下→確認ダイアログ表示
    confirm(): void {
      const valid = (this.$refs.form as FormRef).validate();
      if (valid) {
        let daihyoDuplicate = 0
        // 代表電番として登録する電番を数える
        this.selected.forEach((element) => {
          if (element.daihyo_kubun == DAIHYO_DENBAN) {
            daihyoDuplicate += 1
          }
        })
        // すでに登録されている代表電番を数える
        this.customerDetail.listOfphoneNumber.forEach((element) => {
          if (element.daihyo_kubun == DAIHYO_DENBAN && element.juchu_kubun == JUCHU_STATUS) {
            daihyoDuplicate += 1
          }
        })
        // 代表電番がかぶっている場合はダイアログを出して処理を終了
        if (daihyoDuplicate >= 2) {
          this.$store.dispatch('dialog/open', {
            type: MESSAGE,
            messages:  [this.t('daihyoDuplicateMessage')],
          } as DialogOptions)
          return
        }
        this.$store.dispatch('dialog/open', {
          type: CONFIRMATION,
          subType: REGISTER_UPDATE_EXECUTE,
          messages: [this.t('message')],
          // データ受け取り
          callback: () => this.makeTransferData(),
        } as DialogOptions)
      }
    },
    // 区分管理からセレクトボックスへ変換
    kubunToSelect(kubunList: ResponseData_KUBUN_KANRI[]): SelectOption[] {
      return kubunList.map(({ vl, hyoji_mei }) => ({
        value: vl,
        text: hyoji_mei,
        disabled: false,
      }));
    },
    // 電番移行データ作成
    makeTransferData(): void 
    {
      this.loading = true
      // 登録可能なレジ番リスト作成
      let rejiNoList: number[] = []
      const board_num = Number(this.customerDetail.dataRegisterDetail.board_num)
      // ボード枚数が1以上の場合、子電番で使用可能なレジ番リストを作成
      if (board_num > 0) {
        rejiNoList = [...Array(board_num -1)].map((_, i) => i + 2)
      }
      this.customerDetail.listOfphoneNumber.forEach((element) => {
        if (element.reji_no != null) {
          // 登録済みのレジ番を排除
          rejiNoList = rejiNoList.filter((elm) => elm != element.reji_no)
        }
      })

      // 画面内のデータで処理するのでデータ受け渡しはなし
      console.log('電番移行データ作成')

      const {gno, juchu_no, juchu_meisai_no } = this.$route.query
      
      // 移行先明細Noを4桁にゼロ埋めする
      const padding_saki_serial_no = ('0000' + this.sakiSerialNo).slice(-4)

      let rqData:DenbanIkoRequestData = {
        iko_moto_gno: this.motoGno,
        iko_moto_juchu_no: this.motoJuchuNo,
        iko_moto_juchu_meisai_no: this.motoJuchuMeisaiNo,
        gno: gno.toString(),
        serial_no: padding_saki_serial_no,
        juchu_no: juchu_no.toString(),
        juchu_meisai_no: juchu_meisai_no.toString(),
        bb_juchu_meisai_values:[],
      }

      this.selected.forEach((element, i)=>
        {
          let reji_no = null
          if (element.daihyo_kubun == DAIHYO_DENBAN) {
            // 代表電番の場合はレジ番'1'
            reji_no = 1
          } else if (rejiNoList.length > i) {
            // 子電番の場合は登録可能なレジ番リストから順に採番
            reji_no = rejiNoList[i]
          }
          const rqOneData:DenbanIkoOneData =
          {
            serial_no:element.serial_no,
            ip_tel_no:element.ip_tel_no,
            reji_no: reji_no,
            daihyo_kubun:element.daihyo_kubun,
            bango_tsuchi_kubun:element.bango_tsuchi_kubun,
            chakushin_num:element.chakushin_num,
            n104_annai_kubun:element.n104_annai_kubun,
            kaisen_meigi:element.kaisen_meigi,
            kaisen_meigi_kana:element.kaisen_meigi_kana,
            himozuki_tel_no:element.non_bb_target_phone
          }
          rqData.bb_juchu_meisai_values.push(rqOneData)
        })
      RequestDenbanIko(this.screenId,rqData)
        .then((res)=>
        {

          if("" != res.data?.error_message)
          {
            // エラーメッセージ有（失敗）
            this.$store.dispatch('dialog/open', {
              type: MESSAGE,
              messages: [res.data?.error_message],
            } as DialogOptions)
            
          }else
          {
            // 選択済みデータクリア
            this.selected.splice(0, this.selected.length)

            // 読込データリロード
            this.$emit('reload')

            let messageList = []
            messageList.push(this.t('complete'))
            // 注意メッセージ有（成功）追加
            res.data?.confirm_message.forEach((item)=>{messageList.push(item)})
            
            this.$store.dispatch('dialog/open', {
              type: MESSAGE,
              messages:  messageList,
            } as DialogOptions)
          }
        })
        .finally(()=>{
          this.loading = false
        })
      this.$emit('dirty', false)
    },
    fieldChange(): void{
      console.log('fieldChange')
      this.$emit('dirty', true)
    }
  },
});
</script>

<style lang="scss">
.phone-transfer-result-table {
  // prevent line-break
  th {
    white-space: nowrap;
  }

  // freezed 3 first columns
  .phone-transfer-result-row {
    &__checkbox,
    &__serial-no,
    &__phone-no {
      position: sticky;
      z-index: 1;
    }

    &__checkbox {
      left: 0;
    }

    &__serial-no {
      left: 65px;
    }

    &__phone-no {
      left: 165px;
      border-right: 1px solid transparent;
    }
  }
}
</style>
