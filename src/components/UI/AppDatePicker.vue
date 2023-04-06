<template>
  <v-menu
    v-model="showReservationDate"
    :close-on-content-click="false"
    transition="scale-transition"
    offset-y
    offset-overflow
    max-width="290px"
    min-width="auto"
  >
    <template v-slot:activator="{ on, attrs }">
      <v-text-field
        :value="value"
        autocomplete="date-auto-off"
        :label="label"
        :rules="rules"
        :error="error"
        :append-icon="!readonly ? 'mdi-calendar' : undefined"
        :class="{ 'required-label': required }"
        :filled="filled"
        :readonly="readonly"
        :disabled="disabled"
        v-bind="attrs"
        v-on="on"
        @input="handleDateInput"
      />
    </template>
    <v-date-picker
      v-if="!readonly"
      :value="calendarDate"
      :day-format="date => new Date(date).getDate()"
      locale="ja"
      @input="handleDatePick"
    />
  </v-menu>
</template>

<script lang="ts">
import Vue, { PropType } from 'vue';
import { checkDateValid } from '@/utils/helper';
import dateFormat from 'dateformat';
import { InputValidation } from '@/models/validation';

interface Data {
  showReservationDate: boolean;
}

export default Vue.extend({
  props: {
    value: {
      type: String,
      default: '',
    },
    errorMessages: {
      type: String,
      default: '',
    },
    required: {
      type: Boolean,
      default: false,
    },
    label: {
      type: String,
      default: '',
    },
    filled: {
      type: Boolean,
      default: false,
    },
    readonly: {
      type: Boolean,
      default: false,
    },
    disabled: {
      type: Boolean,
      default: false,
    },
    rules: {
      type: Array as PropType<InputValidation>,
      default: () => [],
    },
    error: {
      type: Boolean,
      default: false,
    },
  },
  data(): Data {
    return {
      showReservationDate: false,
    };
  },
  computed: {
    calendarDate(): string {
      return checkDateValid(this.value)
        ? dateFormat(this.value, 'yyyy-mm-dd')
        : '';
    },
  },
  methods: {
    handleDatePick(date: string): void {
      this.showReservationDate = false;
      this.$emit('input', dateFormat(date, 'yyyy/mm/dd'));
    },
    handleDateInput(date: string): void {
      this.$emit('input', date);
    },
  },
});
</script>
