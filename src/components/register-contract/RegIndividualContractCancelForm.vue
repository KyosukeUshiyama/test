<!--個別解約登録ダイアログ-->
<template>
  <v-card elevation="1" class="pa-5">
    <v-form @submit.prevent="submit">
      <v-row>
        <v-col cols="12" md="3" class="pt-0 pb-0">
          <v-text-field
            value="43081794"
            :label="$t('individualContractCancel.gno')"
            filled
            disabled
          />
        </v-col>
        <v-col cols="12" md="3" class="pt-0 pb-0">
          <v-text-field
            value="0001"
            :label="$t('individualContractCancel.gserialNo')"
            filled
            disabled
          />
        </v-col>
        <v-col cols="12" md="3" class="pt-0 pb-0">
          <v-text-field
            value="0123"
            :label="$t('individualContractCancel.ufCode')"
            filled
            disabled
          />
        </v-col>
      </v-row>
      <v-row>
        <v-col cols="12" sm="6" class="pt-0 pb-0">
          <v-text-field
            value="株式会社フォーバルテレコムCooVo検証用"
            filled
            :label="$t('individualContractCancel.kanji')"
            disabled
          />
        </v-col>
        <v-col cols="12" sm="6" class="pt-0 pb-0">
          <v-text-field
            value="ﾌｫｰﾊﾞﾙﾃﾚｺﾑ"
            filled
            :label="$t('individualContractCancel.furigana')"
            disabled
          />
        </v-col>
      </v-row>
      <v-row>
        <v-col cols="12" sm="3" class="pt-0 pb-0">
          <app-date-picker
            :label="$t('individualContractCancel.billingEndDate')"
          />
        </v-col>
        <v-col cols="12" sm="3" class="pt-0 pb-0">
          <app-date-picker
            :label="$t('individualContractCancel.contractCancelAcceptDate')"
          />
        </v-col>
        <v-col cols="12" sm="3" class="pt-0 pb-0">
          <app-date-picker
            :label="$t('individualContractCancel.constructScheduleDate')"
          />
        </v-col>
      </v-row>
    </v-form>
  </v-card>
</template>

<script lang="ts">
import Vue from 'vue';
import AppDatePicker from '../UI/AppDatePicker.vue';
import { checkDateValid } from '@/utils/helper';
import {
  ContractValidation,
} from '@/models/register-contract';

export default Vue.extend({
  components: { AppDatePicker },
  validations() {
    const contractValidation = new ContractValidation();
    return {
      contractCancelForm: {
        ...contractValidation,
        billing_end_date: { valid: checkDateValid },
        contract_cancel_accept_date: { valid: checkDateValid },
        construct_schedule_date: { valid: checkDateValid },
      },
    };
  },
});
</script>