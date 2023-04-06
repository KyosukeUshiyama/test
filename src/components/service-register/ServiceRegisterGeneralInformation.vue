<!--サービス情報登録_基本情報-->
<template>
  <div>
    <div class="mb-3">
      <h4>{{ screenId }}: {{ t('title') }}</h4>
    </div>
    <v-row dense>
      <v-col cols="12" md="3">
        <v-text-field
          :label="t('employeeCode')"
          :value="customerData.update_user_cd"
          filled
          readonly
          dense
          hide-details="auto"
        />
      </v-col>
      <v-col cols="12" md="3">
        <v-text-field
          :label="t('employeeName')"
          :value="customerData.update_user_mei"
          filled
          readonly
          dense
          hide-details="auto"
        />
      </v-col>
      <v-col cols="12" md="3">
        <v-text-field
          :label="t('customerCode')"
          :value="customerData.kkyk_cd"
          filled
          readonly
          dense
          hide-details="auto"
        />
      </v-col>
      <v-col cols="12" md="3">
        <v-text-field
          :label="t('OACustomerNumber')"
          :value="customerData.oakokyk_bng"
          filled
          readonly
          dense
          hide-details="auto"
        />
      </v-col>
    </v-row>
    <v-row dense>
      <v-col cols="12" md="3">
        <v-text-field
          :label="t('ufCode')"
          :value="customerData.uf_cd"
          filled
          readonly
          dense
          hide-details="auto"
        />
      </v-col>
    </v-row>
  </div>
</template>

<script lang="ts">
import Vue from 'vue';
import { TranslateResult } from 'vue-i18n';
export default Vue.extend({
  props: {
    customerData: {
      type: Object,
      default: () => ({}),
    },
    screenId: {
      type: String,
      default: ''
    }
  },
  created(){
    this.customerData.kkyk_cd = this.$route.query.customer_no.toString();
  },
  methods: {
    t(field: string): TranslateResult {
      return this.$t(`customerDetail.serviceRegister.${field}`);
    },
  },
});
</script>
