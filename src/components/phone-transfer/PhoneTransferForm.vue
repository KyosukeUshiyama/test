<!--電番移行画面-->
<template>
  <v-form ref="form" v-model="valid" @submit.prevent="submit">
    <v-card elevation="1" class="pa-5">
      <v-row>
        <v-col cols="12" md="3" class="pt-0 pb-0">
          <v-text-field
            v-model="searchFormData.gno_of_transfer_source"
            :label="t('gno')"
            :rules="gnoRules"
            class="required-label"
            @input="fieldChange"
          />
        </v-col>
      </v-row>
      <v-row>
        <v-col cols="12" sm="10" class="pt-0 pb-0">
          <v-text-field
            :value="searchFormData.iko_saki_name"
            filled
            :label="t('installationNameOfTransferSource')"
            disabled
          />
        </v-col>
      </v-row>
      <v-row>
        <v-col cols="12" sm="3" class="pt-0 pb-0">
          <v-text-field
            :value="sakiGno"
            filled
            :label="t('gnoOfTransferDestination')"
            disabled
          />
        </v-col>
        <v-col cols="12" md="3" class="pt-0 pb-0">
          <v-text-field
            v-model="searchFormData.target_details_no"
            :label="t('targetDetailsNo')"
            :rules="targetDetailsNoRules"
            class="required-label"
            @input="fieldChange"
          />
        </v-col>
      </v-row>
      <v-row>
        <v-col cols="12" sm="10" class="pt-0 pb-0">
          <v-text-field
            :value="sakiSetimei"
            filled
            :label="t('installationNameOfTransferDestination')"
            disabled
          />
        </v-col>
      </v-row>
      <v-row>
        <v-col cols="12" md="3" offset-md="9">
          <div class="d-flex justify-end">
            <v-btn
              :loading="loading"
              class="mr-5 pl-10 pr-10"
              color="primary"
              @click="submit"
            >
              {{ t('btnSearch') }}
            </v-btn>
          </div>
        </v-col>
      </v-row>
    </v-card>
  </v-form>
</template>

<script lang="ts">
import Vue from 'vue';
import { SearchFormState } from '@/models/transfer-phone';
import { TranslateResult } from 'vue-i18n';
import { InputValidation } from '@/models/validation';
import { numberSize, required } from '@/utils/validation';

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
        gno_of_transfer_source: '',
        target_details_no: '',

        iko_saki_name: '',
      },
      formReset: false,
    };
  },
  computed: {
    gnoRules(): InputValidation {
      return [
        required(
          this.searchFormData.gno_of_transfer_source, 
          this.$t('customerDetail.phoneTransfer.search.gno')
        ),
        numberSize(this.searchFormData.gno_of_transfer_source, 8),
        this.sakiGno === this.searchFormData.gno_of_transfer_source? this.$t('customerDetail.phoneTransfer.search.gnoIcchiError') : true,
      ]
    },
    targetDetailsNoRules(): InputValidation {
      return [
        required(
          this.searchFormData.target_details_no, 
          this.$t('customerDetail.phoneTransfer.search.targetDetailsNo')
        ),
        numberSize(this.searchFormData.target_details_no, 4)
      ]
    },
    // 設置先GNO（前画面から引継ぎ）
    sakiGno(): string {
      const { gno } = this.$route.query;
      if (!gno) return '';
      if (Array.isArray(gno)) return gno[0] || '';
      return gno;
    },
    // 設置先設置名（前画面から引継ぎ）
    sakiSetimei():string{
      const { setisaki_name } = this.$route.query;
      if (!setisaki_name) return '';
      if (Array.isArray(setisaki_name)) return setisaki_name[0] || '';
      return setisaki_name;
    },
  },
  watch:{
    'searchFormData.target_details_no'(value){
        this.$emit('changeserial', value);
    },
  },
  methods: {

    t(field: string): TranslateResult {
      return this.$t(`customerDetail.phoneTransfer.search.${field}`);
    },
    submit(): void {
      const valid = (
        this.$refs.form as Vue & { validate: () => boolean }
      ).validate();
      if (valid){
        this.$emit('submit', this.searchFormData);
        this.$emit('dirty', false)
      }
    },
    fieldChange(): void{
      this.$emit('dirty', true)
    }
  },
})
</script>
