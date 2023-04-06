<!-- 汎用ダイアログ -->
<template>
  <v-dialog v-model="show" width="520" persistent>
    <v-card v-if="show">
      <v-card-title />
      <v-card-text>
        <p v-if="soMessageTitle">
          {{ soMessageTitle }}
        </p>
        <div class="d-flex align-center">
          <v-icon v-if="messageIcon && !soMessageTitle" large class="mr-3">
            {{ messageIcon }}
          </v-icon>
          <v-progress-circular
            v-if="soInputLoading"
            indeterminate
            class="mr-3"
          />
          <div>
            <span
              v-for="item in finalMessages"
              :key="item.text || item"
              :class="{ 'red--text': item.red }"
            >
              {{ item.text || item }}
              <br />
            </span>
          </div>
        </div>
      </v-card-text>
      <v-card-actions v-if="!soInputLoading">
        <v-btn
          v-if="showConfirmationBtns || showWhileInputBtns"
          :disabled="loading"
          @click="close"
        >
          {{ buttons.no }}
        </v-btn>
        <v-spacer />
        <v-btn
          :color="yesBtnPrimary"
          :loading="loading"
          @click="yesBtnCallBack"
        >
          {{ buttons.yes }}
        </v-btn>
      </v-card-actions>
    </v-card>
  </v-dialog>
</template>

<script lang="ts">
import {
  CONFIRMATION,
  MESSAGE,
  RESERVE_PHONE,
  SO_KISHU_CD_NOTHING,
  SO_LOADING,
  SO_RESULT_EXCEED_1_MIN,
  SO_RESULT_FAIL,
  SO_RESULT_SUCESS,
  WHILE_INPUT,
} from '@/constants/dialog';
import { SET_SHOW } from '@/store/dialog/mutation-types';
import { getScreenId } from '@/utils/screenIds';
import Vue from 'vue';
import { TranslateResult } from 'vue-i18n';
import { mapActions, mapGetters, mapState } from 'vuex';

type FinalMessages =
  | TranslateResult[]
  | { text: TranslateResult; red?: boolean }[];

export default Vue.extend({
  computed: {
    ...mapState('dialog', [
      'type',
      'subType',
      'messages',
      'yesBtnCallBack',
      'loading',
    ]),
    ...mapGetters('dialog', [
      'showWhileInputBtns',
      'showConfirmationBtns',
      'showMessageBtns',
      'showCompletionBtns',
      'messageIcon',
      'showSoCloseBtn',
      'showSoVerifyBtn',
    ]),
    ...mapGetters('auth', ['isAuth']),
    finalMessages(): FinalMessages {
      if (this.type === CONFIRMATION && this.subType === RESERVE_PHONE) {
        return [
          {
            text: this.$t('dialog.confirmation.messages.reservePhone1'),
          },
          {
            text: this.$t('dialog.confirmation.messages.reservePhone2'),
            red: true,
          },
          {
            text: this.$t('dialog.confirmation.messages.reservePhone3'),
            red: true,
          },
          {
            text: this.$t('dialog.confirmation.messages.reservePhone4'),
            red: true,
          },
        ];
      }
      if (this.type === CONFIRMATION && this.subType === WHILE_INPUT) {
        return [
          this.$t('dialog.confirmation.messages.closeWhileInput1'),
          this.$t('dialog.confirmation.messages.closeWhileInput2'),
        ];
      }
      if (this.type === MESSAGE && this.subType === SO_LOADING) {
        return [
          this.$t('dialog.so.messages.loading1'),
          this.$t('dialog.so.messages.loading2'),
        ];
      }
      if (this.type === MESSAGE && this.subType === SO_RESULT_SUCESS) {
        return [this.$t('dialog.so.messages.success')];
      }
      if (this.type === MESSAGE && this.subType === SO_RESULT_FAIL) {
        return [
          { text: this.$t('dialog.so.messages.fail1'), red: true },
          { text: this.$t('dialog.so.messages.fail2'), red: true },
        ];
      }
      if (this.type === MESSAGE && this.subType === SO_RESULT_EXCEED_1_MIN) {
        return [
          { text: this.$t('dialog.so.messages.exceed1'), red: true },
          { text: this.$t('dialog.so.messages.exceed2'), red: true },
        ];
      }
      if (this.type === MESSAGE && this.subType === SO_KISHU_CD_NOTHING) {
        return [
          { text: this.$t('dialog.so.messages.kishCdNothing') },
        ];
      }
      return this.messages;
    },
    show: {
      get(): boolean {
        return this.$store.state.dialog.show;
      },
      set(value): void {
        this.$store.commit(`dialog/${SET_SHOW}`, value);
      },
    },
    buttons(): { yes: TranslateResult; no?: TranslateResult } {
      if (this.showConfirmationBtns)
        return {
          yes: this.$t('dialog.confirmation.buttons.yes'),
          no: this.$t('dialog.confirmation.buttons.no'),
        };
      if (this.showWhileInputBtns)
        return {
          yes: this.$t('dialog.confirmation.buttons.yesWhileInput'),
          no: this.$t('dialog.confirmation.buttons.noWhileInput'),
        };
      if (this.showMessageBtns)
        return { yes: this.$t('dialog.message.buttons.yes') };
      if (this.showCompletionBtns)
        return { yes: this.$t('dialog.completion.buttons.yes') };
      if (this.showSoCloseBtn)
        return { yes: this.$t('dialog.so.buttons.close') };
      if (this.showSoVerifyBtn)
        return { yes: this.$t('dialog.so.buttons.verify') };
      return { yes: '', no: '' };
    },
    soInputLoading(): boolean {
      if (this.type === MESSAGE && this.subType === SO_LOADING) {
        return true;
      }
      return false;
    },
    yesBtnPrimary(): 'primary' | '' {
      if (this.showSoCloseBtn) return '';
      return 'primary';
    },
    soMessageTitle(): TranslateResult | string {
      if (this.type === MESSAGE) {
        if (this.subType === SO_LOADING) return getScreenId(this.$route.name,"InputDialog") + this.$t('dialog.so.input');
        if (
          this.subType === SO_RESULT_SUCESS ||
          this.subType === SO_RESULT_FAIL ||
          this.subType === SO_RESULT_EXCEED_1_MIN
        )
          return getScreenId(this.$route.name, "InputResultDialog") + this.$t('dialog.so.inputResult');
      }

      return '';
    },
  },
  watch: {
    // セッションが切れた際にダイアログをcloseする
    isAuth(value) {
      if(!value) this.close();
    },
  },
  methods: {
    ...mapActions('dialog', ['close']),
  },
});
</script>
