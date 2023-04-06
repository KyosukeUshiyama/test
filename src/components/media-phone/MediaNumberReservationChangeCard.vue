<!--メディア電番予約変更ダイアログ-->
<template>
  <v-form ref="form" v-model="valid" @submit.prevent="submit">
    <v-card>
      <v-card-title>
        <span class="text-h5">
          {{ t('title') }}
        </span>
      </v-card-title>
      <v-card-text>
        <v-container>
          <v-row>
            <v-col cols="12" md="4">
              <app-date-picker
                v-model="form.yoyaku_date"
                required
                :rules="yoyakuRules"
                :label="t('reservationdate')"
              />
            </v-col>
          </v-row>

          <v-row>
            <v-col cols="12" md="4">
              <v-text-field
                v-model="form.gno"
                :rules="gnoRules"
                :label="t('gno')"
              />
            </v-col>
          </v-row>
          <v-row>
            <v-col cols="12" md="8">
              <v-text-field
                v-model="form.biko"
                :rules="bikoRules"
                :label="t('note')"
              />
            </v-col>
          </v-row>
          <v-row>
            <v-col cols="12">
              <p>
                {{ t('targetRoutingNumber') }}
              </p>
              <ul ref="numberPhone">
                <li>
                  {{ form.jigyosha_senyo_no }}
                </li>
              </ul>
            </v-col>
          </v-row>
        </v-container>
      </v-card-text>
      <v-card-actions>
        <v-spacer />
        <v-container>
          <v-row>
            <v-col cols="12" md="2">
              <v-btn @click="closeDialog()">
                {{ t('closeButton') }}
              </v-btn>
            </v-col>
            <v-col cols="12" md="2" offset-md="8">
              <v-btn color="info" @click="confirm">
                {{ t('confirmUpdateButton') }}
              </v-btn>
            </v-col>
          </v-row>
        </v-container>
      </v-card-actions>
    </v-card>
  </v-form>
</template>

<script lang="ts">
import { MediaPhoneSearchResultItem } from '@/models/media-phone';
import AppDatePicker from '../UI/AppDatePicker.vue';
import Vue from 'vue';
import {
  required,
  datePast,
  formatDate,
  numberSize,
  bufferSize,
} from '@/utils/validation';
import { InputValidation } from '@/models/validation';
import { CONFIRMATION, REGISTER_UPDATE_EXECUTE, WHILE_INPUT } from '@/constants/dialog';
import { DialogOptions } from '@/models/dialog';
import { TranslateResult } from 'vue-i18n';

interface State {
  valid: boolean;
  isDirty: boolean;
  form: FormValue;
  isDateOfYoyakuShown: boolean;
  yoyakuRules: InputValidation;
  gnoRules: InputValidation;
  bikoRules: InputValidation;
}

interface FormValue {
  yoyaku_date: string;
  gno: string;
  biko: string;
  jigyosha_senyo_no: string;
}

export default Vue.extend({
  components: { AppDatePicker },
  props: {
    data: {
      type: Object as () => MediaPhoneSearchResultItem,
      default: null,
    },
  },
  data(): State {
    return {
      valid: false,
      isDirty: false,
      form: this.data,
      isDateOfYoyakuShown: false,
      yoyakuRules: [
        (value) =>
          required(
            value,
            this.$t('mediaPhoneSearch.changeDialog.reservationdate')
          ),
        (value) => formatDate(value),
        (value) => datePast(value),
      ],
      gnoRules: [(value) => numberSize(value, 8)],
      bikoRules: [(value) => bufferSize(value, 64)],
    };
  },
  watch: {
    form: {
      deep: true,
      handler() {
        this.isDirty = true;
      },
    },
  },
  methods: {
    // 更新確認ボタン押下
    confirm(): void {
      const valid = (
        this.$refs.form as Vue & { validate: () => boolean }
      ).validate();
      if (valid && this.$refs.numberPhone) {
        const copyText = (this.$refs.numberPhone as HTMLUListElement).innerText;
        this.$store.dispatch('dialog/open', {
          type: CONFIRMATION,
          subType: REGISTER_UPDATE_EXECUTE,
          messages: [this.t('message')],
          autoClose: false,
          callback: async (close) => {
            this.$emit('confirm', copyText, this.form, close);
          },
        } as DialogOptions);
      }
    },
    closeDialog() {
       if (this.isDirty) {
        this.$store.dispatch('dialog/open', {
          type: CONFIRMATION,
          subType: WHILE_INPUT,
          callback: () => this.$emit('close-dialog'),
        } as DialogOptions);
      } else {
        this.$emit('close-dialog');
      }
      
    },
    t(field: string): TranslateResult {
      return this.$t(`mediaPhoneSearch.changeDialog.${field}`);
    },
  },
});
</script>
