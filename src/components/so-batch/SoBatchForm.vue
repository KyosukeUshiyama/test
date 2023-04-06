<!--新規申込一括SO作成_フォーム-->
<template>
  <v-card elevation="1" class="pa-5">
    <v-form ref="form" v-model="valid" @submit.prevent="submit">
      <v-row align="center">
        <v-col cols="12" md="3">
          <app-date-picker
            v-model="$v.searchFormData.voip_setchi_date.$model"
            :rules="registrationDateFromRules"
            :label="t('voipInstallDate')"
            required
          />
        </v-col>
        <v-col cols="12" md="3">
          <v-btn
            color="primary"
            :disabled="loading"
            :loading="loading"
            @click="submit"
          >
            {{ t('search') }}
          </v-btn>
        </v-col>
      </v-row>
    </v-form>
  </v-card>
</template>

<script lang="ts">
import Vue from 'vue';
import { SearchFormState, SoInquiryDataValidation } from '@/models/so-batch';
import { TranslateResult } from 'vue-i18n';
import AppDatePicker from '../UI/AppDatePicker.vue';
import { formatDate, datePast, required } from '@/utils/validation';

export default Vue.extend({
  components: { AppDatePicker },
  props: {
    loading: {
      type: Boolean,
      default: false,
    },
  },
  data(): SearchFormState {
    return {
      valid: false,
      searchFormData: {
        voip_setchi_date: '',
      },
      registrationDateFromRules: [
        (value) =>
          required(
            value,
            this.$t('soBatch.voipInstallDate')
          ),
        (value) => formatDate(value),
        (value) => datePast(value),
      ]
    };
  },
  validations: {
    searchFormData: {
      ...new SoInquiryDataValidation(),
    },
  },
  watch: {
    searchFormData: {
      deep: true,
      handler() {
        this.$v.$touch();
      },
    },
  },
  methods: {
    submit(): void {
      const valid = (
        this.$refs.form as Vue & { validate: () => boolean }
      ).validate();

      if (valid && !this.$v.$invalid && this.$v.$dirty) {
        this.$emit('submit', this.searchFormData);
      }
    },
    t(field: string): TranslateResult {
      return this.$t(`soBatch.${field}`);
    },
  },
});
</script>
