<!--パスワード再設定メール_フォーム -->
<template>
  <v-form
    v-if="!tokenValid"
    ref="form"
    v-model="valid"
    @submit.prevent="submit"
  >
    <template>
      <v-container>
        <v-row no-gutters>
          <v-col md="12" align="center">
            <v-card-text class="text-center">
              <v-icon>mdi-lock</v-icon>
              {{ $t('auth.requestReset.resetPasswordTitleLabel') }}
            </v-card-text>
            <v-card-text class="text-center">
              <div v-if="sendMailSuccess">
                {{ $t('auth.requestReset.guidanceMessage1Label') }}
              </div>
              <div v-if="sendMailSuccess" class="text-center red--text">
                {{ $t('auth.requestReset.guidanceMessage2Label') }}
              </div>
            </v-card-text>
            <v-card-text class="text-center">
              <div v-if="sendMailSuccess">
                {{ $t('auth.requestReset.guidanceMessage3Label') }}
                <br />
                {{ $t('auth.requestReset.guidanceMessage4Label') }}
              </div>
              <div v-if="!sendMailSuccess">
                {{ $t('auth.requestReset.guidanceMessageNotification1Label') }}
                <br />
                {{ $t('auth.requestReset.guidanceMessageNotification2Label') }}
              </div>
            </v-card-text>
            <v-card-text v-if="errorMsg" class="text-center red--text">
              {{ errorMsg }}
            </v-card-text>
            <v-col md="6">
              <v-text-field
                v-if="!sendMailSuccess"
                v-model="idLogin"
                :label="$t('auth.requestReset.loginIDText')"
                :rules="idLoginRules"
                class="required-label"
              />
            </v-col>
            <v-row no-gutters>
              <v-col>
                <v-card-actions class="justify-center">
                  <v-btn class="mr-4 justify-center" @click="back">
                    {{
                      sendMailSuccess
                        ? $t('auth.requestReset.backLoginButton')
                        : $t('auth.requestReset.backButton')
                    }}
                  </v-btn>
                </v-card-actions>
              </v-col>
              <v-col v-if="!sendMailSuccess">
                <v-card-actions class="justify-center">
                  <v-btn type="submit" color="info" class="mr-4 justify-center">
                    {{ $t('auth.requestReset.sendMailButton') }}
                  </v-btn>
                </v-card-actions>
              </v-col>
            </v-row>
          </v-col>
        </v-row>
      </v-container>
    </template>
  </v-form>
</template>

<script lang="ts">
import Vue from 'vue';
import { FormRef, InputValidation } from '@/models/validation';
import { bufferSize, required } from '@/utils/validation';
import { requestResetPasswordAPI } from '@/api/reset-password-api';
import { AxiosError } from 'axios';
import { getScreenId } from '@/utils/screenIds';

export default Vue.extend({
  data() {
    return {
      valid: false,
      idLogin: '',
      sendMailSuccess: false,
      errorMsg: '',
      tokenValid: false,
      idLoginRules: [
        (value: InputValidation) =>
          required(value, this.$t('auth.requestReset.loginIDText')),
        (value: InputValidation) => bufferSize(value, 20),
      ],
    };
  },
  methods: {
    back() {
      this.$router.push('/login').catch(err => {console.error(err)});
    },
    async submit() {
      const valid = (this.$refs.form as FormRef).validate();
      if (valid)
        requestResetPasswordAPI(getScreenId(this.$route.name), this.idLogin)
          .then((res) => {
            // メッセージがある場合は表示
            // 無い場合はリクエスト成功
            if(res.data.message){
              this.errorMsg = res.data.message;
            }else {
              this.errorMsg = ""
              this.sendMailSuccess = true;
            }
          })
          .catch((err: AxiosError) => {
            this.errorMsg = err.response?.data.message;
          });
    },
  },
});
</script>
