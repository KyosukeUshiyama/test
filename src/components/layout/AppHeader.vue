<!-- -->
<template>
  <v-app-bar app flat clipped-left>
    <v-app-bar-nav-icon
      v-if="$vuetify.breakpoint.mdAndDown"
      @click="toggleSidebar"
    />
    <a style="text-decoration: none" href="/">
      <h3 v-if="$vuetify.breakpoint.lgAndUp" class="title-color" :class="getTitleClass()">
        {{ $t('common.systemName') }}
      </h3>
    </a>
    <v-spacer />
    <v-switch
      v-model="theme"
      class="switch-margin"
      :prepend-icon="themeIcon"
    />
    <v-menu open-on-hover offset-y>
      <template v-slot:activator="{ on, attrs }">
        <v-btn v-bind="attrs" large text v-on="on">
          <v-icon left size="26">mdi-account-circle</v-icon>
          {{ userInfo.user_name }}
        </v-btn>
      </template>
      <v-list max-width="350px" three-line>
        <v-list-item>
          <v-list-item-content>
            <v-list-item-title>
              {{ $t('layout.header.permission') }}
            </v-list-item-title>
            <v-list-item-subtitle>
              {{ userInfo.role }}
            </v-list-item-subtitle>
          </v-list-item-content>
        </v-list-item>
        <v-list-item>
          <v-list-item-content>
            <v-list-item-title>
              {{ $t('layout.header.memberOf') }}
            </v-list-item-title>
            <v-list-item-subtitle>
              {{ dairitenList }}
            </v-list-item-subtitle>
          </v-list-item-content>
        </v-list-item>
      </v-list>
      <v-divider />
      <v-list dense>
        <v-list-item @click="handleLogout">
          <v-list-item-icon><v-icon>mdi-logout</v-icon></v-list-item-icon>
          <v-list-item-content>
            <v-list-item-title>ログアウト</v-list-item-title>
          </v-list-item-content>
        </v-list-item>
      </v-list>
    </v-menu>
  </v-app-bar>
</template>

<script lang="ts">
import Vue from 'vue';
import { mapActions } from 'vuex';
import {
  getDairitenList,
} from '@/api/auth-api';
import
{
  getLoginUser,
} from '@/utils/helper';
import { getScreenId } from '@/utils/screenIds';
import { CONFIRMATION, WHILE_INPUT } from '@/constants/dialog';
import { DialogOptions } from '@/models/dialog';
import { getModeClass } from '../../utils/helper';

let g_dairitenList:string[] = [];

export default Vue.extend({
  data() {
    return {
      loggingOut: false,
      userInfo:{
        user_name: this.$store.state.auth.user_name,
        role: this.$store.state.auth.role,
      },
      dairitenList:g_dairitenList,
      theme: false,
    };
  },
  computed: {
    themeIcon(): string {
      return this.theme ? 'mdi-weather-night' : 'mdi-weather-sunny'
    }
  },
  watch: {
    theme() {
      this.$vuetify.theme.dark = this.theme
    }
  },
  mounted() {
    this.theme = this.$vuetify.theme.dark
  },
  created()
  {
      // 代理店リストを呼びだす
      getDairitenList(getScreenId('Header'), getLoginUser())
      .then((res)=>
      {
        if(null!=res.data)
        {
          this.dairitenList = [];
          sessionStorage.setItem('dairitenData', JSON.stringify(res.data));
          res.data.forEach((el) =>
          {
            this.dairitenList.push(el?.dairiten_mei);
          })
        }
      })
      .catch(err => console.error(err.message))
  },
  methods: {
    ...mapActions('auth', ['logout']),
    toggleSidebar() {
      this.$emit('toggle-sidebar');
    },
    getTitleClass(): string {
      return getModeClass()
    },
    async handleLogout() {
      this.loggingOut = true;
      if(this.$store.state.customer.editing){
        await this.$store.dispatch('dialog/open', {
          type: CONFIRMATION,
          subType: WHILE_INPUT,
          callback: () => this.logout(),
        } as DialogOptions);
      }else{
        await this.logout();
      }
      this.loggingOut = false;
    },
  },
});
</script>

<style lang="scss" scoped>
.logout-btn {
  text-decoration: underline;
  color: rgba(4, 4, 100, 0.932);
  cursor: pointer;
}

.truncate {
  display: inline-block;
  vertical-align: middle;
  width: 400px;
  text-overflow: ellipsis;
  white-space: nowrap;
  overflow: hidden;
}

.switch-margin {
  margin-top: 20px;
}

.development {
  color: #00ff73 !important;
  caret-color: #00ff73 !important;
}
.staging {
  color: #ffc400 !important;
  caret-color: #ffc400 !important;
}

.theme--light .title-color:not(.staging):not(.development) {
    color: #000000 !important;
    caret-color: #000000 !important;
}
.theme--dark .title-color:not(.staging):not(.development) {
    color: #FFFFFF !important;
    caret-color: #FFFFFF !important;
}
</style>
