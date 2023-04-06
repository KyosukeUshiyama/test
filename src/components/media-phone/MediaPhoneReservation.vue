<!--メディア電番予約ダイアログ-->
<template>
  <v-form
    ref="form"
    v-model="valid"
    class="flex-grow-1"
    @submit.prevent="submit"
  >
    <h1 class="mb-5">
      {{ $t('mediaPhoneSearch.reservation.screenName') }}
    </h1>
    <v-row>
      <v-col cols="12" md="6">
        <app-date-picker
          v-model="form.reservation_date"
          required
          :rules="reservationDateRules"
          :label="$t('mediaPhoneSearch.reservation.reservationDate')"
        />
      </v-col>
    </v-row>
    <v-row>
      <v-col cols="12" md="6">
        <v-text-field
          v-model="form.gno"
          :rules="gnoRules"
          :label="$t('mediaPhoneSearch.reservation.GNO')"
        />
      </v-col>
    </v-row>
    <v-row>
      <v-col cols="12" md="6">
        <v-text-field
          v-model="form.note"
          :rules="noteRules"
          :label="$t('mediaPhoneSearch.reservation.note')"
        />
      </v-col>
    </v-row>
    <v-row>
      <v-col cols="12">
        <p class="mb-0">
          {{ $t('mediaPhoneSearch.reservation.warning1') }}
          <br />
          {{ $t('mediaPhoneSearch.reservation.warning2') }}
        </p>
      </v-col>
    </v-row>
    <v-row>
      <v-col cols="12">
        <ul ref="numberList" class="number-list">
          <li v-for="item in selected" :key="item" class="mb-3">
            {{ item }}
          </li>
        </ul>
      </v-col>
    </v-row>
    <v-row>
      <v-col cols="12">
        <div class="d-flex justify-space-between">
          <v-btn @click="cancel">
            {{ $t('mediaPhoneSearch.reservation.closeBtn') }}
          </v-btn>
          <v-btn color="primary" @click="confirm">
            {{ $t('mediaPhoneSearch.reservation.confirmBtn') }}
          </v-btn>
        </div>
      </v-col>
    </v-row>
  </v-form>
</template>

<script lang="ts">
import Vue, { PropType } from 'vue';
import { MediaPhoneReservationState } from '@/models/media-phone';
import AppDatePicker from '../UI/AppDatePicker.vue';
import { required, formatDate, bufferSize, dateRange, numberSize } from '@/utils/validation';
import { DialogOptions } from '@/models/dialog';
import { CONFIRMATION, RESERVE_PHONE, WHILE_INPUT } from '@/constants/dialog';
import dateFormat from "dateformat";

export default Vue.extend({
  components: { AppDatePicker },
  props: {
    selected: {
      type: Array as PropType<string[]>,
      default: [],
    },
  },
  data(): MediaPhoneReservationState {
    return {
      valid: false,
      isDirty: false,
      form: {
        // 現在日付を取得
        reservation_date: dateFormat(new Date(),"yyyy/mm/dd"),
        gno: '',
        note: '',
      },
      reservationDateRules: [
        (value) =>
          required(
            value,
            this.$t('mediaPhoneSearch.reservation.reservationDate')
          ),
        (value) => formatDate(value),
        (value) => dateRange(value, dateFormat(new Date(),"yyyy/mm/dd"), null)
      ],
      gnoRules: [(value) => numberSize(value, 8)],
      noteRules: [(value) => bufferSize(value, 64)],
    };
  },
    watch: {
    form: {
      deep: true,
      handler() {
        this.isDirty = true;
      }
    }
  },
  methods:
  {
    // 予約確認ボタン押下
    confirm(): void {
      const valid = (
        this.$refs.form as Vue & { validate: () => boolean }
      ).validate();
      if (valid && this.$refs.numberList) {
        const copyText = (this.$refs.numberList as HTMLUListElement).innerText;
        this.$store.dispatch('dialog/open', {
          type: CONFIRMATION,
          subType: RESERVE_PHONE,
          messages: [],
          callback: () => {
            this.$emit('confirm', copyText,this.form.reservation_date,this.form.gno,this.form.note);
          },
        } as DialogOptions);
      }
    },
    cancel(): void {
        if (this.isDirty) {
        this.$store.dispatch('dialog/open', {
          type: CONFIRMATION,
          subType: WHILE_INPUT,
          callback: () => this.$emit('cancel'),
        } as DialogOptions);
      } else {
        this.$emit('cancel');
      }
      
    },
  },
});
</script>

<style lang="scss" scoped>
.number-list {
  border: 1px solid #e9e9e9;
  height: 100px;
  overflow-y: auto;
  padding: 10px 35px;
}
</style>
