<!--SOデータ作成確認ダイアログ（顧客検索から）-->
<template>
  <v-card>
    <v-card-title>
      <div>
        <h3 class="mb-5">
          {{ showScreenId }}:{{ t('mainTitle') }}
        </h3>
        <h3>
          {{ t('subTitle') }}
        </h3>
        <h3>
          {{ t('subTitle2') }}
        </h3>
      </div>
    </v-card-title>
    <v-card-text>
      <div v-if="warningMessage.length > 0" class="mt-5 red--text">
        <span
          v-for="item in warningMessage"
          :key="item"
        >
          {{ item }}
          <br />
        </span>
      </div>

      <v-subheader>{{ t('gno') }}</v-subheader>
      <span class="ml-7">
        {{ gnoCode }}
      </span>

      <v-subheader class="mt-5">
        {{ t('constructScheduleDate') }}
      </v-subheader>
      <span class="ml-7">
        {{ constructScheduleDate }}
      </span>

      <v-subheader class="mt-5">
        {{ t('listSODataCreate') }}
      </v-subheader>
      <ul class="ml-7">
        <li
          v-for="(item, index) in dataList"
          :key="index"
          :index="index"
        >
          {{ item.title }}
          <div
            v-for="(itemChild, indexChild) in item.values"
            :key="itemChild"
            :index="indexChild"
          >
            {{ itemChild }}
          </div>
        </li>
      </ul>
    </v-card-text>
    <v-card-actions class="justify-space-between">
      <v-btn @click="close">
        {{ t('btnClose') }}
      </v-btn>
      <v-btn color="primary" @click="create">
        {{ t('btnCreate') }}
      </v-btn>
    </v-card-actions>
  </v-card>
</template>

<script lang="ts">
import Vue, { PropType } from 'vue';
import { TranslateResult } from 'vue-i18n';
import { SoInfomationConfirmOneData } from "@/models/so/index"
import { getScreenId } from '@/utils/screenIds';

interface targetDataState
{
  dataList:SoInfomationConfirmOneData[];
}

export default Vue.extend({
  name: "SODataCreationConfirmationDialogMenu",
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
    constructScheduleDate: {
      type: String,
      default: '',
    },
    warningMessage: {
      type: Array as PropType<string[]>,
      default: '',
    },
    soCreateDataList:{
      type: [Object,Array],
      default:()=>[],
    }
  },
  data():targetDataState 
  {
    return {
      dataList:[],
    };
  },
  computed:
  {
    // 画面ID取得
    showScreenId(): string {
      return getScreenId(
        this.$route.name,
        this.$options.name
      );
    },
  },
  created(){
    this.dataList = this.soCreateDataList as SoInfomationConfirmOneData[];
  },
  methods: {
    t(field: string): TranslateResult {
      return this.$t(`customerDetail.soDataCreateConfirm.${field}`);
    },
    close(): void {
      this.$emit('close');
    },
    // 親画面（SoInformationDetails）の
    // handleCreateDialogSODataConfirmを呼び出す
    create(): void {
      this.$emit('create');
    },
  },
});
</script>
