<!--システムエラー-->
<template>
  <div>
    <v-row justify="center" class="mt-4">
      <v-col cols="12">
        <h3 class="text-center">
          {{ $t('common.systemName') }}
        </h3>
        <div class="text-center">
          <v-icon large>mdi-alert</v-icon>
        </div>
      </v-col>
      <v-col cols="12">
        <h4 class="text-center red--text">
          {{ getTitle }}
        </h4>
        <h4 class="text-center red--text">
          （{{ t('alertNumberCodeLabel') + getErrorCode }} ）
        </h4>
      </v-col>
      <v-col cols="12" class="text-center mss-error">
        <div>
          {{ getErrorMessage }}
        </div>
      </v-col>
    </v-row>

    <v-card class="mx-auto" max-width="510">
      <div>
        <v-list-item>
          <v-list-item-content>
            <v-list-item-title>
              <u>{{ t('contactInquiries') }}</u>
            </v-list-item-title>
          </v-list-item-content>
        </v-list-item>
      </div>
      <div>
        <v-list-item>
          <v-list-item-content>
            <v-list-item-title>
              {{ t('contactFT') }}
            </v-list-item-title>
          </v-list-item-content>
        </v-list-item>
      </div>
      <div>
        <v-list-item>
          <v-list-item-content>
            <v-list-item-title>
              {{ t('contactTel') }}
            </v-list-item-title>
          </v-list-item-content>
          <v-row>
            <span>: {{ t('phoneContact') }}</span>
          </v-row>
        </v-list-item>
      </div>
      <div>
        <v-list-item>
          <v-list-item-content>
            <v-list-item-title>
              {{ t('contactEmail') }}
            </v-list-item-title>
          </v-list-item-content>
          <v-row>
            <span>: {{ t('emailContact') }}</span>
          </v-row>
        </v-list-item>
      </div>
    </v-card>
    <v-card-actions class="justify-center mt-5">
      <v-btn
        :loading="loading"
        color="info"
        class="m-4 justify-center"
        @click="back"
      >
        {{ t('backLoginButton') }}
      </v-btn>
    </v-card-actions>
  </div>
</template>

<script lang="ts">
import Vue from 'vue';
import { TranslateResult } from 'vue-i18n';
import { mapActions, mapGetters } from 'vuex';
export default Vue.extend({
  data() {
    return {
      loading: false,
    };
  },
  computed: {
    ...mapGetters('systemError', [
      'getTitle',
      'getErrorCode',
      'getErrorMessage',
    ]),
  },
  methods: {
    ...mapActions('auth', ['logout']),
    back() {
      this.$router.push('/login').catch(err => {console.error(err)});
    },
    t(field: string): TranslateResult {
      return this.$t(`common.systemError.${field}`);
    },
  },
});
</script>

<style lang="scss">
.mss-error {
  max-width: 30%;
  padding-bottom: 50px;
}
</style>
