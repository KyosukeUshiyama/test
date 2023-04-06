<!--電番情報検索-->
<template>
  <v-card>
    <v-form ref="form" v-model="valid" @submit.prevent="submit">
      <v-card-text>
        <div class="d-flex align-center">
          <v-text-field
            v-model="areaCode"
            :label="$t('advancedPhoneCheck.areaCode')"
            :rules="advancedPhoneRules"
            class="required-label"
          />
          <v-spacer />
          <v-btn color="primary" @click="submit">
            {{ $t('advancedPhoneCheck.searchBtn') }}
          </v-btn>
        </div>
      </v-card-text>
    </v-form>
  </v-card>
</template>

<script lang="ts">
import Vue from 'vue';
import { FormRef, InputValidation } from '@/models/validation';
import { numberSize, required } from '@/utils/validation';

export default Vue.extend({
  props: {
    screenId: {
      type: String,
      default: '',
    },
  },  
  data(): {
    areaCode: string;
    valid: boolean;
    advancedPhoneRules: InputValidation;
  } {
    return {
      valid: false,
      areaCode: '',
      advancedPhoneRules: [
        (value) => required(value, this.$t('advancedPhoneCheck.areaCode')),
        (value) => numberSize(value, 7, 7),
      ],
    };
  },
  methods: {
    submit() {
      const valid = (this.$refs.form as FormRef).validate();
      if (valid) {
        this.$emit('submit', this.areaCode);
      }
    },
  },
});
</script>
