<!-- 顧客検索画面_フォーム -->
<template>
  <v-form ref="form" v-model="valid" @submit.prevent="submit">
    <!-- v-card color="grey lighten-5" -->
    <v-card>
      <v-container>
        <v-row md="8" sm="12" no-gutters>
          <v-col md="4">
            <v-text-field
              v-model="searchFormData.gno"
              outlined
              :value="123"
              :label="t('gnoText')"
              :rules="gnoRules"
            />
          </v-col>
          <v-col md="4" offset="2">
            <v-text-field
              v-model="searchFormData.uf_code"
              :label="t('ufCodeText')"
              :rules="ufCodeRules"
            />
          </v-col>
        </v-row>
        <v-row no-gutters sm="12">
          <v-col md="4">
            <v-text-field
              v-model="searchFormData.tel"
              value
              :label="t('phoneNumberText')"
              :rules="phoneNumberTextRules"
            />
          </v-col>
          <v-col md="4" offset="2">
            <v-text-field
              v-model="searchFormData.customer"
              :label="t('customerNumberText')"
              :rules="customerNumberTextRules"
            />
          </v-col>
        </v-row>
        <v-row no-gutters>
          <v-col md="4">
            <v-text-field
              v-model="searchFormData.oa_customer"
              :label="t('qaCustomerNumberText')"
              :rules="qaCustomerNumberTextRules"
            />
          </v-col>
          <v-col md="4" offset="2">
            <v-text-field
              v-model="searchFormData.ip"
              :label="t('ipAddress')"
              :rules="ipAddressRules"
            />
          </v-col>
        </v-row>
        <v-row no-gutters>
          <v-col md="5">
            <v-text-field
              v-model="searchFormData.set_customer"
              :label="t('installationDestinationCustomerNameText')"
              :rules="installDestCustomerNameRules"
            />
          </v-col>
          <v-col md="4" offset="1">
            <v-checkbox
              v-model="searchFormData.cancel"
              :label="t('contractCancellationOrderCheckbox')"
              hide-details
            />
          </v-col>
        </v-row>
      </v-container>
      <v-card-actions>
        <v-spacer />
        <v-btn color="secondary" :disabled="isSearchFormEmpty" @click="clear">
          <v-icon>mdi-close</v-icon>
          {{ t('clearButton') }}
        </v-btn>
        <v-btn
          width="120px"
          color="primary"
          :loading="loading"
          :disabled="isSearchFormEmpty"
          @click="submit"
        >
          <v-icon>mdi-magnify</v-icon>
          {{ t('searchButton') }}
        </v-btn>
      </v-card-actions>
    </v-card>
  </v-form>
</template>

<script lang="ts">
import Vue from 'vue';
import { SearchFormCustomerData, SearchFormState } from '@/models/customer';
import { TranslateResult } from 'vue-i18n';
import { FormRef } from '@/models/validation';
import { bufferSize, numberSize, formatIpAddress, formatTelNo } from '@/utils/validation';
import { mapState } from 'vuex';

export default Vue.extend({
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
        gno: '',
        uf_code: '',
        tel: '',
        customer: '',
        oa_customer: '',
        ip: '',
        set_customer: '',
        cancel: false,
      },
      formReset: false,
      gnoRules: [(value) => numberSize(value, 8)],
      ufCodeRules: [(value) => bufferSize(value, 13)],
      phoneNumberTextRules: [(value) => formatTelNo(value)],
      customerNumberTextRules: [(value) => bufferSize(value, 8)],
      qaCustomerNumberTextRules: [(value) => bufferSize(value, 6)],
      ipAddressRules: [(value) => formatIpAddress(value)],
      installDestCustomerNameRules: [(value) => bufferSize(value, 72)],
    };
  },
  validations: {
    searchFormData: {},
  },
  computed: {
    ...mapState('customer', ['formData']),
    isSearchFormEmpty(): boolean {
      let key: keyof SearchFormCustomerData;
      for (key in this.searchFormData) {
        const value = this.searchFormData[key];
        if (
          key !== 'cancel' &&
          value !== null &&
          value !== '' &&
          value !== false
        ) {
          return false;
        }
      }
      return true;
    },
  },
  watch: {
    searchFormData: {
      deep: true,
      handler() {
        if (this.formReset) {
          this.formReset = false;
          return;
        }
        this.$v.$touch();
      },
    },
  },
  created() {
    if (this.formData) {
      this.searchFormData.gno = this.formData.gno;
      this.searchFormData.uf_code = this.formData.uf_code;
      this.searchFormData.tel = this.formData.tel;
      this.searchFormData.customer = this.formData.customer;
      this.searchFormData.oa_customer = this.formData.oa_customer;
      this.searchFormData.ip = this.formData.ip;
      this.searchFormData.set_customer = this.formData.set_customer;
      this.searchFormData.cancel = this.formData.cancel;
    }
  },
  methods: {
    submit() {
      const valid = (this.$refs.form as FormRef).validate();
      if (valid) {
        this.$store.dispatch('customer/saveFormData', this.searchFormData);
        this.$emit('submit', this.searchFormData);
      }
    },
    clear(): void {
      (this.$refs.form as FormRef).reset();
      this.searchFormData = {
        gno: '',
        uf_code: '',
        tel: '',
        customer: '',
        oa_customer: '',
        ip: '',
        set_customer: '',
        cancel: false,
      };
    },
    t(field: string): TranslateResult {
      return this.$t(`customerSearch.searchForm.${field}`);
    },
  },
});
</script>
