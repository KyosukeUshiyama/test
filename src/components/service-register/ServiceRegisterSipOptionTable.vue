<!--サービス情報登録_SIPオプション一覧-->
<template>
  <div>
    <div class="d-flex align-center flex-wrap mb-3">
      <h4>{{ t('SIPOptionsList') }}</h4>
      <v-spacer />
      <div>
        <v-btn :disabled="cancelStatus || !createPermission" color="primary" class="ml-2" @click="onpenDialogSip(ADD, null)">
          {{ t('addSip') }}
        </v-btn>
      </div>
    </div>
    <v-data-table
      :headers="headers"
      :items="listSipOptions"
      :hide-default-footer="true"
      :options.sync="tableOptions"
      class="elevation-1"
    >
      <template v-slot:[`item.action`]="{ item }">
        <v-icon 
          v-if="isDeleted(item)" 
          :disabled="!(updatePermission || readPermission) || isOtsukaChakushinDisabled(item)" 
          @click="onpenDialogSip(EDIT, item)"
        >
          mdi-pencil
        </v-icon>
      </template>
      <template #top="{ pagination, options, updateOptions }">
        <v-data-footer
          :pagination="pagination"
          :options="options"
          items-per-page-text="$vuetify.dataTable.itemsPerPageText"
          @update:options="updateOptions"
        />
      </template>
      <template #no-data>
        <p>{{ t('messages.notFound') }}</p>
      </template>
    </v-data-table>
    <v-dialog v-model="showSipEdit" persistent width="1100">
      <service-register-add-edit-sip
        v-if="showSipEdit"
        :cancel-status="cancelStatus"
        :customer-data="customerData"
        :sip-data="sipData"
        :sip-data-list="listSipOptions"
        :edit-mode="mode === EDIT"
        @close="showSipEdit = false"
        @confirm="confirmAdd"
      />
    </v-dialog>
  </div>
</template>

<script lang="ts">
import { SipOptionsResponse } from '@/models/customer';
import Vue, { PropType } from 'vue';
import ServiceRegisterAddEditSip from '@/components/service-register/ServiceRegisterAddEditSip.vue';
import { TranslateResult } from 'vue-i18n';
import { DataTableHeader, DataOptions } from 'vuetify';
import { SipData, ADD, EDIT } from '@/models/service-register';
import { DEFAULT_HYOJI_NUM, FLAG_ADDED, FLAG_NORMAL, FLAG_UPDATED, FLAG_DELETED } from '@/constant/common-constant';
import { getScreenId } from '@/utils/screenIds';
import dateFormat from "dateformat";

interface ServiceRegisterSipOptionTableState{
  listSipOptionsData: Array<SipOptionsResponse>,
  showSipEdit: boolean,
  mode: string,
  sipData: SipOptionsResponse | null,
  updatePermission: boolean,
  createPermission: boolean,
  readPermission: boolean,
  tableOptions: DataOptions
}

export default Vue.extend({
  components: {
    ServiceRegisterAddEditSip,
  },
  props: {
    customerData: {
      type: Object,
      default: () => ({}),
    },
    listSipOptions: {
      type: Array as PropType<SipOptionsResponse[]>,
      default: [],
    },
    cancelStatus: {
      type: Boolean,
      default: false,
    },
    screenId: {
      type: String,
      default: null,
    }
  },
  data(): ServiceRegisterSipOptionTableState {
    return {
      listSipOptionsData: this.listSipOptions,
      showSipEdit: false,
      mode: '',
      sipData: null,
      updatePermission: false,
      createPermission: false,
      readPermission: false,
      tableOptions: {
        page: 1,
        itemsPerPage: DEFAULT_HYOJI_NUM,
        sortBy: ['kaiyaku_uketsuke_date', 'service_start_date'],
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
          text: this.t('serviceName'),
          value: 'fuka_service_cd',
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
          text: this.t('contractCancellationAcceptanceDate'),
          value: 'kaiyaku_uketsuke_date',
        },
        {
          text: this.t('serviceEndDate'),
          value: 'service_end_date',
        },
        {
          text: this.t('phoneNumberDetour'),
          value: 'jido_ukai_tel_no',
        },
        {
          text: this.t('SIPOptionsEdit'),
          value: 'action',
        },
      ];
    },
    ADD() {
      return ADD;
    },
    EDIT() {
      return EDIT;
    }
  },
  created() {
    const create_name = "SIPOptionAdditionDialog"
    const update_name = "SIPOptionChangeDialog"
    const screen_ids = { 
      "create": getScreenId(this.$route.name, create_name),
      "update": getScreenId(this.$route.name, update_name),
    }
    this.updatePermission = this.$$hasUpdatePermission(screen_ids["update"]);
    this.createPermission = this.$$hasCreatePermission(screen_ids["create"]);
    this.readPermission = this.$$checkPermission(screen_ids["update"]);
  },
  methods: {
    isOtsukaChakushinDisabled(item: SipOptionsResponse | null) {
      // 代理店が大塚商会系列の場合は着信転送サービス（10080060）は編集させない
      if (item && !this.$store.state.auth.otsukaDairitenFlg && item.sip_option_service_cd === '10080060') {
        return true
      } else {
        return false
      }
    },
    onpenDialogSip(mode: string, item: SipOptionsResponse | null ): void {
      this.mode = mode;
      if( mode === EDIT ){
        let itemClone: SipOptionsResponse = Object.assign({}, item);
        this.sipData = itemClone ;
      }
      if(mode === ADD){
        let maxSerialNo = '0';
        this.listSipOptions.forEach((element) => {
          if (typeof(element.no) == "string" && element.no >= maxSerialNo){
            maxSerialNo = element.no;
          }
        })
        this.sipData = {
          id: (new Date()).getTime() + "",
          no:  (Number(maxSerialNo) + 1).toString(),
          gno: this.customerData.gno,
          uf_cd: this.customerData.uf_cd,
          sechimoto_kanji_kaisyamei: this.customerData.seti_kanji_shamei,
          sechimoto_kana_kaisyamei: this.customerData.seti_kn,
          sip_option_service_cd: '',
          fuka_service_cd: '',
          moshikomi_uketsuke_date: dateFormat(new Date(),"yyyy/mm/dd"),
          kaiyaku_uketsuke_date: '',
          service_start_date: '',
          service_end_date: '',
          jido_ukai_tel_no: '',
          so_data_sakusei_kubun: '',
          so_kaiyaku_data_sakusei_kubun: '',
          biko: '',
          sakujo_flg: '',
          add_flg:FLAG_NORMAL,
          update_flg:FLAG_NORMAL,
        }
      }
      this.showSipEdit = true;
    },
    isDeleted(item: SipOptionsResponse): boolean {
      return item.sakujo_flg !== FLAG_DELETED
    },
    t(field: string): TranslateResult {
      return this.$t(`customerDetail.serviceRegister.${field}`);
    },
    confirmAdd(data: SipData): void {
      console.log(data)
      let dataForm: SipOptionsResponse = {
        id: data.id,
        no: data.no,
        gno: data.gno,
        uf_cd: data.uf_cd,
        sechimoto_kanji_kaisyamei: data.sechimoto_kanji_kaisyamei,
        sechimoto_kana_kaisyamei: data.sechimoto_kana_kaisyamei,
        sip_option_service_cd: data.sip_option_service_cd,
        fuka_service_cd: data.fuka_service_cd,
        moshikomi_uketsuke_date: data.moshikomi_uketsuke_date,
        kaiyaku_uketsuke_date: data.kaiyaku_uketsuke_date,
        service_start_date: data.service_start_date,
        service_end_date: data.service_end_date,
        jido_ukai_tel_no: data.jido_ukai_tel_no,
        so_data_sakusei_kubun: data.so_data_sakusei_kubun,
        so_kaiyaku_data_sakusei_kubun: data.so_kaiyaku_data_sakusei_kubun,
        sakujo_flg: FLAG_NORMAL,
        biko: data.biko,
        add_flg:FLAG_ADDED,
        update_flg:FLAG_NORMAL,
      };
      if(data.editStatus === ADD){
        this.listSipOptions.push(dataForm);
      }else {
        const dataEdit = this.listSipOptions.find(
          (item) => item.id === data.id
        );
        if (dataEdit) {
          dataEdit.id = data.id;
          dataEdit.gno = data.gno;
          dataEdit.uf_cd = data.uf_cd;
          dataEdit.sechimoto_kanji_kaisyamei = data.sechimoto_kanji_kaisyamei;
          dataEdit.sechimoto_kana_kaisyamei = data.sechimoto_kana_kaisyamei;
          dataEdit.sip_option_service_cd = data.sip_option_service_cd;
          dataEdit.fuka_service_cd = data.fuka_service_cd;
          dataEdit.moshikomi_uketsuke_date = data.moshikomi_uketsuke_date;
          dataEdit.kaiyaku_uketsuke_date = data.kaiyaku_uketsuke_date;
          dataEdit.service_start_date = data.service_start_date;
          dataEdit.service_end_date = data.service_end_date;
          dataEdit.jido_ukai_tel_no = data.jido_ukai_tel_no;
          dataEdit.so_data_sakusei_kubun = data.so_data_sakusei_kubun;
          dataEdit.so_kaiyaku_data_sakusei_kubun = data.so_kaiyaku_data_sakusei_kubun;
          dataEdit.biko = data.biko;
          if(FLAG_ADDED != dataEdit.add_flg)
          {
            // 新規追加フラグがない場合のみ更新フラグを立てる
            dataEdit.update_flg = FLAG_UPDATED;
          }
        }
      }
      this.showSipEdit = false;
      this.$emit('listOfSipOptions-changed');
    },
  },
});
</script>
