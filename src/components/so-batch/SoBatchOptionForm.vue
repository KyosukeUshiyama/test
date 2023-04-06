<!--オプション一括SO作成_フォーム-->
<template>
  <v-form ref="form" v-model="valid" class="pt-2" @submit.prevent="submit">
    <v-row>
      <v-col cols="12" md="4" offset="1">
        <app-date-picker
          v-model="inputFormData.add_scheduled_date"
          required
          :rules="addScheduleDateRules"
          :label="t('addScheduledDate')"
        />
      </v-col>
      <v-col cols="12" md="3">
        <v-btn color="primary" class="mt-1" :disabled="!createPermission" @click="submit">
          {{ t('createButton') }}
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
import { getScreenId } from '@/utils/screenIds';

export default Vue.extend({
  components: { AppDatePicker },
  data() {
    return {
      valid: false,
      inputFormData: {
        add_scheduled_date: '',
      },
      addScheduleDateRules: [
        (value: InputValidation) =>
          required(value, this.$t('soBatch.addScheduledDate')),
        (value: InputValidation) => formatDate(value),
      ],
      screenId: '',
      createPermission: false,
    };
  },
  created() {
    this.screenId = getScreenId(this.$route.name);
    this.createPermission = this.$$hasCreatePermission(this.screenId);
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
          messages: [this.t("messages.soBatchOption")],
          callback: () => this.$emit('confirm', this.inputFormData.add_scheduled_date),
        } as DialogOptions);
      }
    },
    t(field: string): TranslateResult {
      return this.$t(`soBatch.${field}`);
    },
  },
});
</script>
