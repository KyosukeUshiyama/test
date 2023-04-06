<!--電番情報チェック-->
<template>
  <div class="mt-5 pa-5">
    <v-row>
      <v-col md="4" cols="12">
        <v-text-field
          :label="$t('advancedPhoneCheck.numberPortabilityOk')"
          :value="result.denpo_status || null"
          filled
          readonly
        />
      </v-col>
    </v-row>
    <v-row>
      <v-col md="4" cols="12">
        <v-text-field
          :label="$t('advancedPhoneCheck.station')"
          :value="result.kyokusha_mei || null"
          filled
          readonly
        />
      </v-col>
    </v-row>
    <v-row>
      <v-col md="4" cols="12">
        <v-text-field
          :label="$t('advancedPhoneCheck.100MPossibleArea')"
          :value="result.area_100m || null"
          filled
          readonly
        />
      </v-col>
      <v-col md="4" cols="12">
        <v-text-field
          :label="$t('advancedPhoneCheck.1GPossibleArea')"
          :value="result.area_1g || null"
          filled
          readonly
        />
      </v-col>
    </v-row>
    <v-row>
      <v-col md="4" cols="12">
        <v-text-field
          :label="$t('advancedPhoneCheck.emergencyCall')"
          :value="result.kinkyu_tsuho || null"
          filled
          readonly
        />
      </v-col>
    </v-row>
    <v-row>
      <v-col md="4" cols="12">
        <v-text-field
          :label="$t('advancedPhoneCheck.050NumberOnBBPossibleArea')"
          :value="result.number_050 || null"
          filled
          readonly
        />
      </v-col>
      <v-col md="4" cols="12">
        <v-text-field
          :label="$t('advancedPhoneCheck.CooVoPossibleArea')"
          :value="result.coovo || null"
          filled
          readonly
        />
      </v-col>
    </v-row>
  </div>
</template>

<script lang="ts">
import Vue from 'vue';
export default Vue.extend({
  props: {
    // eslint-disable-next-line vue/require-prop-types
    result: {
      default: () => ({}),
    },
  },
});
</script>
