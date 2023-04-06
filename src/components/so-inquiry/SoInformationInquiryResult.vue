<!-- SO情報照会画面 一覧表示 -->
<template>
  <div class="elevation-1 pa-5 mt-5">
    <div class="d-flex justify-start mb-5">
      <h3 class="pr-10">
        {{ t('targetData') }}
      </h3>
      <v-btn 
        v-show="menuMode" 
        color="primary"
        :disabled="!createPermission_new_so"
        @click="showSoSignUp"
      >
        {{ t('newRegistration') }}
      </v-btn>
    </div>
    <v-data-table
      v-model="selected"
      :headers="headers"
      :items="mediaPhoneList"
      :options.sync="tableOptions"
      hide-default-footer
    >
      <template v-slot:top="{ pagination, options, updateOptions }">
        <v-data-footer
          :pagination="pagination"
          :options="options"
          :items-per-page-options="[5, 10, 15, 20, -1]"
          @update:options="updateOptions"
        />
      </template>
      <template #no-data>
        <p>{{ t('messageNotFound') }}</p>
      </template>
      <template v-slot:[`item.detail`]="{ item }">
        <v-icon :disabled="!readPermission" @click="showSo(item)">mdi-pencil</v-icon>
      </template>
    </v-data-table>
    <v-dialog v-model="show" persistent width="1100">
      <so-information-details
        v-if="show"
        :mode="dmode"
        :so-information="soInfo"
        @close="close"
        @remove="handleConfirm"
        @register="handleConfirm"
      />
    </v-dialog>
    <v-dialog v-model="showSignUp" persistent width="1100">
      <so-information-sign-up
        v-if="showSignUp"
        :mode="dmode"
        :so-information="soInfo"
        @close="close"
        @remove="handleConfirm"
        @register="handleConfirm"
      />
    </v-dialog>
  </div>
</template>

<script lang="ts">
import Vue, { PropType } from 'vue';
import {
  SearchResultState,
  SoInquirySearchResultItemDetail,
} from '@/models/so-inquiry';
import { DataTableHeader } from 'vuetify';
import { TranslateResult } from 'vue-i18n';
import SoInformationDetails from '@/components/so/SoInformationDetails.vue';
import SoInformationSignUp from '@/components/so/SoInformationSignUp.vue'
import { SoInformation, SO_MENU, SO_NEW, UNDECIDED_DATE } from '@/models/so';
import { getScreenId } from '@/utils/screenIds';
import { DEFAULT_HYOJI_NUM } from '@/constant/common-constant';

export default Vue.extend({
  components: {
    SoInformationDetails,
    SoInformationSignUp,
  },
  props: {
    mediaPhoneList: {
      type: Array as PropType<SoInquirySearchResultItemDetail[]>,
      default: [],
    },
    newSearch: {
      type: Boolean,
    },
    mode: {
      type: String as PropType<string>,
      default: null,
    },
    screenId: {
      type: String,
      default: null,
    }
  },
  data(): SearchResultState {
    return {
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
      selected: [],
      dataChangeDialog: {} as SoInquirySearchResultItemDetail,
      show: false,
      showSignUp :false,
      showSoDetail: false,
      showSoDetailEditMode: false,
      soInfo: {
        ...new SoInformation(),
        constructionScheduleDate: '',
      },
      menuMode: this.mode === SO_MENU,
      dmode: this.mode,
      createPermission: false,
      readPermission: false,
      createPermission_new_so: false,
    };
  },
  computed: {
    headers(): (DataTableHeader | { text: TranslateResult })[] {
      return [
        {
          text: this.t('constructionScheduleDate'),
          value: 'construction_schedule_date',
        },
        {
          text: this.t('gno'),
          value: 'gno',
        },
        {
          text: this.t('subNumber'),
          value: 'sub_number',
        },
        {
          text: this.t('type'),
          value: 'type',
        },
        {
          text: this.t('section'),
          value: 'section',
        },
        {
          text: this.t('companyName'),
          value: 'company_name',
        },
        {
          text: this.t('sendFlag'),
          value: 'send_flag',
        },
        {
          text: this.t('createSoSection'),
          value: 'create_so_section',
        },
        {
          text: this.t('status'),
          value: 'status',
        },
        {
          text: this.t('errorContent'),
          value: 'error_content',
        },
        {
          text: this.t('displayDetail'),
          value: 'detail',
        },
      ];
    },

  },
  watch: {
    newSearch(value) {
      if (value) this.tableOptions.page = 1;
    },
    selected: {
      deep: true,
      handler(value) {
        this.$emit('select', value);
      },
    },
  },
  created() {
    const name = "SoInformationDetails"
    const screen_id = getScreenId(this.$route.name, name, this.mode);
    this.createPermission = this.$$hasCreatePermission(screen_id);
    this.readPermission = this.$$checkPermission(screen_id);
    const name_new_so = "SoAdditionDialog";
    const screen_id_new_so = getScreenId(this.$route.name, name_new_so);
    this.createPermission_new_so = this.$$hasCreatePermission(screen_id_new_so);
  },
  methods: {
    t(field: string): TranslateResult {
      return this.$t(`soInquiry.table.${field}`);
    },
    edit(item: SoInquirySearchResultItemDetail): void {
      this.dataChangeDialog = item;
    },
    // SO情報照会-検索結果
    handleConfirm() {
      console.log("SO情報照会-handleConfirm");
      this.close();
      this.$emit("reload")
    },
    close() {
      // SO新規追加時、SO情報一覧更新
      if(this.showSignUp){
        this.$emit("reload")
      }
      this.show = false;
      this.showSignUp = false;
      this.showSoDetail = false;
      this.showSoDetailEditMode = false;
    },
    showSo(item: SoInquirySearchResultItemDetail): void {
      console.log("SO情報照会-showSo");
      this.show = true;
      const soInfo = new SoInformation();
      console.log("so_item=" + item)
      if (item != null) {
        soInfo.gno = item.gno;
        soInfo.subNumber = item.sub_number;
        soInfo.companyName = item.company_name;
        soInfo.district = item.chiku;
        soInfo.newContractCancel = item.section_vl;
        soInfo.type = item.type_vl;
        soInfo.constructionScheduleDate = item.construction_schedule_date;
        soInfo.constructionScheduleDateUndecided = soInfo.constructionScheduleDate === UNDECIDED_DATE ? true : false;
        soInfo.createSoFile = item.create_so_section_vl;
        soInfo.emergencySendingFlag = item.send_flag_vl;
        soInfo.errorComment = item.error_content;
        soInfo.status = item.status_vl;
        soInfo.soRegistration = item.soRegistration;
        soInfo.oaOrderNumber = item.oa_juchu_no;
        soInfo.oaOrderSubNumber = item.oa_juchu_edaban;
        soInfo.registerContents = item.toroku_naiyo_id;
        soInfo.comment = item.comment;
        soInfo.soFileName = item.soFileName;
        soInfo.sofileCreateDate = item.sofileCreateDate;
        soInfo.soSendDate = item.soSendDate;
        soInfo.soSender = item.soSender
        soInfo.resultImportDate = item.resultImportDate
        soInfo.registrationDate = item.registrationDate;
        soInfo.registeredUser = item.registeredUser;
        soInfo.updatedDate = item.updatedDate;
        soInfo.updatedUser = item.updatedUser;
        soInfo.juch_no = item.juch_no;
        soInfo.juch_mesai_no = item.juch_mesai_no
        this.dmode = this.mode;
      } else {
        this.dmode = SO_NEW;
      }
      this.soInfo = soInfo;
    },
    showSoSignUp(): void {
      console.log("SO情報新規登録-showSoSignUp");
      // 新規登録のダイアログを開くときに初期化
      this.soInfo = new SoInformation()
      this.showSignUp = true;
    },
  },
});
</script>
