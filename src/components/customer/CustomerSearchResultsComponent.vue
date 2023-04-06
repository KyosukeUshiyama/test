<!-- 顧客検索画面_検索結果 -->
<template>
  <div>
    <h3>{{ t('titleText') }}</h3>
    <div class="elevation-1">
      <v-data-table
        :headers="headers"
        :items="customerList"
        item-key="id"
        :item-class="itemRowBackground"
        :options.sync="tableOptions"
        hide-default-footer
      >
        <template v-slot:top="{ pagination, options, updateOptions }">
          <v-data-footer
            :pagination="pagination"
            :options="options"
            :items-per-page-options="[5, 10, 15, 20]"
            @update:options="updateOptions"
          />
        </template>
        <template v-slot:no-data>
          <p>{{ t('messageNotFound') }}</p>
        </template>
        <template v-slot:[`item.action`]="{ item }">
          <v-icon @click="redirectCustomerInfo(item)">
            mdi-format-list-bulleted-square
          </v-icon>
        </template>
        <template v-slot:[`item.status`]="{ item }">
          <div :class="{ 'red--text': item.status === '9' }">
            {{
              item.status === '1'
                ? t('underContract')
                : item.status === '9'
                  ? t('contractCancellation')
                  : t('unregistered')
            }}
          </div>
        </template>
      </v-data-table>
    </div>
  </div>
</template>

<script lang="ts">
import Vue, { PropType } from 'vue';
import { SearchResultState, CustomerSearchResultItem } from '@/models/customer';
import { DataTableHeader } from 'vuetify';
import { TranslateResult } from 'vue-i18n';
import { mapState } from 'vuex';
import { DEFAULT_HYOJI_NUM } from '@/constant/common-constant';

export default Vue.extend({
  props: {
    customerList: {
      type: Array as PropType<CustomerSearchResultItem[]>,
      default: [],
    },
    newSearch: {
      type: Boolean,
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
        multiSort: false,
        mustSort: false,
      },
    };
  },
  computed: {
     ...mapState('customer', [
      'idSession',
      'customerSearch',
    ]),
    headers(): (DataTableHeader | { text: TranslateResult })[] {
      return [
        {
          text: '',
          value: 'action',
        },
        {
          text: this.t('customerNumberText'),
          value: 'customer_no',
        },
        {
          text: this.t('ucomCodeText'),
          value: 'uf_cd',
        },
        { text: this.t('gnoText'), value: 'gno' },
        {
          text: this.t('installationDestinationNameText'),
          value: 'setisaki_name',
        },
        {
          text: this.t('statusText'),
          value: 'status',
        },
        {
          text: this.t('addressText'),
          value: 'setisaki_jusyo',
        },
        {
          text: this.t('phoneNumber'),
          value: 'kakin_tel_no',
        },
      ];
    },
  },
  watch: {
    newSearch(value) {
      if (value) this.tableOptions.page = 1;
    },
  },
  methods: {
    redirectCustomerInfo(item: string) {
      const itemString = JSON.stringify(item);
      const itemJson = JSON.parse(itemString);
      if(""==itemJson.gno)
      {
        console.log("サービス情報登録画面表示時にgnoなし");
        alert(this.t('NoGNO'));   
      }
      else
      {
        this.$store.dispatch('customer/saveIdSession', itemJson.id);
        this.$router.push({path:'/customer/detail/' + itemJson.gno, 
          query:{
            gno: itemJson.gno,
            customer_no:itemJson.customer_no,
            kakin_tel_no: itemJson.kakin_tel_no,
            setisaki_jusyo: itemJson.setisaki_jusyo,
            setisaki_name: itemJson.setisaki_name,
            status: itemJson.status,
            uf_cd: itemJson.uf_cd,
            juchu_no: itemJson.juch_no,
            juchu_meisai_no: itemJson.juch_mesai_no,
            seti_no: itemJson.seti_no,
          }
        }).catch(err => {console.error(err)});
      }
    },
    itemRowBackground(item: CustomerSearchResultItem): string {
      return item.id === this.idSession ? 'row-choose-style' : '';
    },
    t(field: string): TranslateResult {
      return this.$t(`customerSearch.searchResult.${field}`);
    },
  },
});
</script>

<style lang="scss">
.mdi-format-list-bulleted-square::before {
  content: '\F0DD0';
  background-color: lightseagreen;
  border-radius: 10px;
  color: white;
  padding: 2px;
}

.row-choose-style {
  background-color: #1976d2;
}
</style>
