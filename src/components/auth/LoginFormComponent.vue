<!--ログイン画面_フォーム-->
<template>
  <v-container>
    <div
      v-if="error"
      class="text-center red--text"
    >
      <span>{{ error }}</span>
    </div>
    <v-form ref="form" v-model="valid" @submit.prevent="submit">
      <template>
        <v-row>
          <v-col
            md="12"
            align="center"
          >
            <v-col md="6">
              <v-text-field
                id="id_login"
                v-model="user.id"
                :label="$t('auth.login.loginIDText')"
                :rules="loginIDRules"
              />
              <v-text-field
                v-model="user.password"
                :append-icon="isShow ? 'mdi-eye' : 'mdi-eye-off'"
                :type="isShow ? 'text' : 'password'"
                :label="$t('auth.login.passwordText')"
                :rules="passwordRules"
                autocomplete="new-password"
                @click:append="isShow = !isShow"
              />
            </v-col>
            <v-card-actions class="justify-center">
              <v-btn
                type="submit"
                :loading="isFetching"
                color="info"
              >
                {{ $t('auth.login.loginButton') }}
              </v-btn>
            </v-card-actions>
            <v-card-actions class="justify-center">
              <router-link
                id="link-router"
                to="/request-reset"
              >
                {{ $t('auth.login.resetPasswordLink') }}
              </router-link>
            </v-card-actions>
          </v-col>
        </v-row>
      </template>
    </v-form>
  </v-container>
</template>

<script lang="ts">
import Vue from 'vue';
import { mapGetters, mapState } from 'vuex';
import { LoginRequest } from '@/models/service/index';
import { bufferSize, required } from '@/utils/validation';
import { FormRef, InputValidation } from '@/models/validation';

interface Data {
  valid: boolean;
  user: LoginRequest;
  isShow: boolean;
  loginIDRules: InputValidation;
  passwordRules: InputValidation;
}

export default Vue.extend({
  data(): Data {
    return {
      valid: false,
      user: {
        id: '',
        password: '',
      },
      isShow: false,
      loginIDRules: [
        (value) => required(value, this.$t('auth.login.loginIDText')),
        (value) => bufferSize(value, 20),
      ],
      passwordRules: [
        (value) => required(value, this.$t('auth.login.passwordText')),
        (value) => bufferSize(value, 20),
      ],
    };
  },
  computed: {
    ...mapState('auth', ['error', 'isFetching']),
    ...mapGetters('auth', ['isAuth']),
  },
  methods: {
    async submit() {
      const valid = (this.$refs.form as FormRef).validate();
      if (valid) await this.$store.dispatch('auth/login', this.user);
    },
  },
});
</script>

<style scoped>
#link-router {
  text-decoration-line: none;
}
</style>
