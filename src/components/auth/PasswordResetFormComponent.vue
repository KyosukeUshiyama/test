<!--パスワード再設定画面_フォーム-->
<template>
  <v-form ref="form" v-model="valid" @submit.prevent="submit">
    <template>
      <v-container>
        <v-row>
          <v-col md="12" align="center">
            <v-card-text class="text-center">
              <v-icon>mdi-lock</v-icon>
              {{ $t('auth.resetPassword.resetPasswordTitleLabel') }}
            </v-card-text>
            <v-card-text v-if="changePassSuccess" class="text-center">
              <h3>
                {{
                  $t('auth.resetPassword.resetGuidanceMessageComplete1Label')
                }}
              </h3>
              <div>
                {{
                  $t('auth.resetPassword.resetGuidanceMessageComplete2Label')
                }}
              </div>
            </v-card-text>
            <v-card-text v-if="!changePassSuccess" class="text-center">
              {{ $t('auth.resetPassword.resetGuidanceMessageLabel') }}
            </v-card-text>
            <v-card-text v-if="vError" class="text-center red--text">
              {{ vError }}
            </v-card-text>
            <v-col md="6">
              <v-text-field
                v-if="!changePassSuccess"
                v-model="newPassword"
                :append-icon="isShowPass ? 'mdi-eye' : 'mdi-eye-off'"
                :type="isShowPass ? 'text' : 'password'"
                :label="$t('auth.resetPassword.newPasswordLabel')"
                :rules="newPassRules"
                @click:append="isShowPass = !isShowPass"
              />
              <v-text-field
                v-if="!changePassSuccess"
                v-model="reNewPassword"
                :append-icon="isShowRePass ? 'mdi-eye' : 'mdi-eye-off'"
                :type="isShowRePass ? 'text' : 'password'"
                :label="$t('auth.resetPassword.newPasswordConfirmLabel')"
                :rules="reNewPassRules"
                @click:append="isShowRePass = !isShowRePass"
              />
            </v-col>

            <v-row no-gutters>
              <v-col>
                <v-card-actions class="justify-center">
                  <v-btn type="submit" color="info" class="mr-4 justify-center">
                    {{
                      changePassSuccess
                        ? $t('auth.resetPassword.backLoginResetCompleteButton')
                        : $t('auth.resetPassword.resetPasswordButton')
                    }}
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
import { TranslateResult } from 'vue-i18n';
import { bufferSize, required, passwordPolicy, samePassword } from '@/utils/validation';
import { InputValidation } from '@/models/validation';
import { validateTokenAPI, resetPasswordAPI } from '@/api/reset-password-api';
import { ResetPasswordRequest } from '@/models/password-reset';
import { AxiosError } from 'axios';
import { getScreenId } from '@/utils/screenIds';

interface Data {
  valid: boolean;
  isShowPass: boolean;
  isShowRePass: boolean;
  newPassword: string;
  reNewPassword: string;
  changePassSuccess: boolean;
  vError: TranslateResult;
  newPassRules: InputValidation;
}

export default Vue.extend({
  data(): Data {
    return {
      valid: false,
      isShowPass: false,
      isShowRePass: false,
      newPassword: '',
      reNewPassword: '',
      changePassSuccess: false,
      vError: '',
      newPassRules: [
        (value) => passwordPolicy(value),
        (value) => required(value, this.$t('auth.resetPassword.newPasswordLabel')),
        (value) => bufferSize(value, 20),
      ]
    };
  },

  computed:{
    reNewPassRules(): InputValidation {
      return [
        samePassword(this.reNewPassword, this.newPassword),
        required(this.reNewPassword, this.$t('auth.resetPassword.newPasswordConfirmLabel')),
        bufferSize(this.reNewPassword, 20),
      ];
    },
  },
  created() {
    const { token } = this.$route.query;
    validateTokenAPI(getScreenId(this.$route.name), token?.toString())
      .then((res) => {
        console.log(res);
      })
      .catch((err: AxiosError) => {
        console.error(err.message)
      });
  },
  methods: {
    submit() {
      const valid = (
        this.$refs.form as Vue & { validate: () => boolean }
      ).validate();
      if (valid) {
        if (this.changePassSuccess) {
          this.$router.push('/login').catch(err => {console.error(err)});
        } else {
          let data: ResetPasswordRequest = {
            token: this.$route.query.token.toString(),
            pass: this.newPassword,
          };
          resetPasswordAPI(getScreenId(this.$route.name), data)
            .then(() => {
              this.vError = '';
              this.changePassSuccess = true;
            })
            .catch((err: AxiosError) => {
              console.error(err.message)
            });
        }
      }
    },
  },
});
</script>
