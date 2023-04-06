<!--メディア電番検索画面-->
<template>
  <div class="elevation-1 pa-5 mt-5">
    <div class="d-flex justify-end mb-5">
      <v-btn
        :disabled="noRowSelected || !cancelPermission"
        color="error"
        class="mr-5"
        @click="remove"
      >
        {{ t('deleteBtn') }}
      </v-btn>
      <v-btn :disabled="noRowSelected || !reservePermission" color="primary" @click="reserve">
        {{ t('reserveBtn') }}
      </v-btn>
    </div>
    <v-data-table
      v-model="selected"
      show-select
      :headers="headers"
      :items="mediaPhoneList"
      item-key="jigyosha_senyo_no"
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
      <template v-slot:[`item.edit`]="{ item }">
        <v-icon :disabled="!updatePermission" @click="edit(item)">mdi-pencil</v-icon>
      </template>
    </v-data-table>
    <v-dialog v-model="isOpenDialog" width="800px" persistent>
      <!-- メディア番号予約変更ダイアログ -->
      <media-number-reservation-change-card
        v-if="isOpenDialog"
        :data="dataChangeDialog"
        @confirm="handleConfirmReserve"
        @close-dialog="() => (isOpenDialog = false)"
      />
    </v-dialog>
    <div class="d-flex justify-end mt-5">
      <v-btn
        :disabled="noRowSelected || !cancelPermission"
        color="error"
        class="mr-5"
        @click="remove"
      >
        {{ t('deleteBtn') }}
      </v-btn>
      <v-btn :disabled="noRowSelected || !reservePermission" color="primary" @click="reserve">
        {{ t('reserveBtn') }}
      </v-btn>
    </div>
  </div>
</template>

<script lang="ts">
import Vue, { PropType } from 'vue';
import {
  SearchResultState,
  MediaPhoneSearchResultItem,
} from '@/models/media-phone';
import { DataTableHeader } from 'vuetify';
import MediaNumberReservationChangeCard from '@/components/media-phone/MediaNumberReservationChangeCard.vue';
import { TranslateResult } from 'vue-i18n';
import {
  updateMediaPhoneYoyaku,
  MediaPhoneUpdateYoyakuRequestDataItem,
} from '@/api/media-phone-api';
import { getLoginUser } from '@/utils/helper';
import { DialogOptions } from '@/models/dialog';
import { MESSAGE, MESSAGE_ERROR } from '@/constants/dialog';
import { getScreenId } from '@/utils/screenIds';
import { DEFAULT_HYOJI_NUM } from '@/constant/common-constant';
import { AxiosError } from 'axios';

export default Vue.extend({
  components: { MediaNumberReservationChangeCard },
  props: {
    mediaPhoneList: {
      type: Array as PropType<MediaPhoneSearchResultItem[]>,
      default: [],
    },
    newSearch: {
      type: Boolean,
    },
    select: {
      type: Array as PropType<MediaPhoneSearchResultItem[]>,
      default: [],
    },
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
        multiSort: true,
        mustSort: false,
      },
      selected: [],
      isOpenDialog: false,
      dataChangeDialog: {} as MediaPhoneSearchResultItem,
      screenId: '',
      updatePermission: false,
      reservePermission: false,
      cancelPermission: false,
    };
  },
  computed: {
    noRowSelected(): boolean {
      return this.selected.length === 0;
    },
    headers(): (DataTableHeader | { text: TranslateResult })[] {
      return [
        {
          text: this.t('businessPersonPhoneNumber'),
          value: 'jigyosha_senyo_no',
        },
        {
          text: this.t('GNO'),
          value: 'gno',
        },
        {
          text: this.t('managementFlag'),
          value: 'kanri_flg',
        },
        {
          text: this.t('protocol'),
          value: 'protocol_kubun',
        },
        {
          text: this.t('reservationEmployeeCode'),
          value: 'yoyaku_shain_no',
        },
        {
          text: this.t('reservationDate'),
          value: 'yoyaku_date',
        },
        {
          text: this.t('note'),
          value: 'biko',
        },
        {
          text: this.t('edit'),
          value: 'edit',
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
    select: {
      handler(after, before) {
        if (after.length === 0 && before.length > 0) {
          this.selected = [];
        }
      },
    },
  },
  created() {
    const reserve_name = "MediaPhoneNumberReservationDialog"
    const cancel_name = "MediaPhoneNumberReservationCancelDialog"
    const update_name = "MediaPhoneNumberReservationChangeDialog"
    const screenIds = {
      "reserve": getScreenId(this.$route.name, reserve_name),
      "cancel": getScreenId(this.$route.name, cancel_name),
      "update": getScreenId(this.$route.name, update_name),
    }
    this.reservePermission = this.$$hasUpdatePermission(screenIds["reserve"]);
    this.cancelPermission = this.$$hasUpdatePermission(screenIds["cancel"]);
    this.updatePermission = this.$$hasUpdatePermission(screenIds["update"]);
  },
  methods: {
    // メディア番号予約変更ダイアログ表示
    edit(item: MediaPhoneSearchResultItem): void {
      let itemClone: MediaPhoneSearchResultItem = Object.assign({}, item);
      this.dataChangeDialog = itemClone;
      this.isOpenDialog = true;
    },
    // 取消ボタン押下
    remove() {
      this.$emit('remove', this.selected);
    },
    // 予約ボタン押下
    reserve() {
      const selectedManagementFlags = this.selected.map(
        (item) => item.kanri_flg
      );
      const enableFlags = ['0', '1', '4', 'X'];
      const shouldDisable = selectedManagementFlags.filter(
        (item) => !enableFlags.includes(item)
      );
      if (shouldDisable.length > 0) {
        this.$store.dispatch('dialog/open', {
          type: MESSAGE,
          subType: MESSAGE_ERROR,
          messages: [this.t('reserveError')],
        } as DialogOptions);
      } else {
        this.$emit('reserve');
      }
    },
    // メディア番号予約変更ダイアログの更新確認ボタン押下を受ける
    handleConfirmReserve(
      copyText: string,
      data: MediaPhoneSearchResultItem,
      closeAppDialog: () => void
    ): void {
      // 確認ダイアログ表示
      navigator.clipboard.writeText(copyText);
      this.changeReserve(data, closeAppDialog);
    },
    // 予約変更実行
    changeReserve(
      data: MediaPhoneSearchResultItem,
      closeAppDialog: () => void
    ): void {
      console.log('changeReserve');
      console.log(data);

      const reqData: MediaPhoneUpdateYoyakuRequestDataItem = {
        yoyaku_date: data.yoyaku_date,
        gno: data.gno,
        kanri_flg: data.kanri_flg,
        biko: data.biko,
        phone: []
      };

      reqData.yoyaku_shain_no = getLoginUser(); // ログインユーザIDを取得

      let updatePhone = []
      updatePhone.push(data.jigyosha_senyo_no.split('-').join('')) // 圧縮電番に変換して追加
      reqData.phone = updatePhone
      updateMediaPhoneYoyaku(getScreenId(this.$route.name), reqData)
        .then(() => {
          const dataEdit = this.mediaPhoneList.find(
            (item: MediaPhoneSearchResultItem) =>
              item.ip_ashk_ura_tel_no === data.ip_ashk_ura_tel_no
          );
          if (dataEdit) {
            dataEdit.biko = data.biko;
            dataEdit.gno = data.gno;
            dataEdit.yoyaku_date = data.yoyaku_date;
            dataEdit.yoyaku_shain_no = getLoginUser();
          }
        })
        .catch((err: AxiosError) => {
          if (err.response) {
            alert(err.response.data.title +'\n' +
            "エラー内容: " + err.response.data.detail)
          }
        })
        .finally(() => {
          this.isOpenDialog = false;
          closeAppDialog();
        });
    },
    t(field: string): TranslateResult {
      return this.$t(`mediaPhoneSearch.searchForm.${field}`);
    },
  },
});
</script>
