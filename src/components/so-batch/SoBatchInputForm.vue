<!-- SO一括投入フォーム -->
<template>
  <v-form ref="form" v-model="valid" class="pt-2" @submit.prevent="submit">
    <v-row>
      <v-col cols="12" md="4" offset="1">
        <app-date-picker
          v-model="inputFormData.construction_schedule_date"
          required
          :rules="scheduleDateRules"
          :label="t('constructionScheduleDate')"
        />
      </v-col>
      <v-col cols="12" md="3">
        <v-btn color="primary" class="mt-1" @click="submit">
          {{ t('sendButton') }}
        </v-btn>
      </v-col>
    </v-row>
  </v-form>
</template>

<script lang="ts">
import Vue from 'vue';
import { TranslateResult } from 'vue-i18n';
import AppDatePicker from '../UI/AppDatePicker.vue';
import { CONFIRMATION, REGISTER_UPDATE_EXECUTE } from '@/constants/dialog';
import { required, formatDate } from '@/utils/validation';
import { InputValidation } from '@/models/validation';
import { DialogOptions } from '@/models/dialog';

export default Vue.extend({
  components: { AppDatePicker },
  data() {
    return {
      valid: false,
      inputFormData: {
        construction_schedule_date: '',
      },
      scheduleDateRules: [
        (value: InputValidation) =>
          required(value, this.$t('soBatch.constructionScheduleDate')),
        (value: InputValidation) => formatDate(value),
      ],
    };
  },
  methods: {
    submit(): void {
      const valid = (
        this.$refs.form as Vue & { validate: () => boolean }
      ).validate();
      if (valid) {
        this.$store.dispatch('dialog/open', {
          type: CONFIRMATION,
          subType: REGISTER_UPDATE_EXECUTE,
          messages: [this.t("messages.soBatchInput")],
          autoClose: false,
          callback: async (close) => this.$emit('confirm', this.inputFormData.construction_schedule_date, close),
        } as DialogOptions);
      }
    },
    t(field: string): TranslateResult {
      return this.$t(`soBatch.${field}`);
    },
  },
});
</script>
