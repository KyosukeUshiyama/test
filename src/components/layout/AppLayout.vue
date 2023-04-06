<!-- -->
<template>
  <v-app id="inspire">
    <div v-if="loading" id="loading">
      <v-progress-circular indeterminate :size="70" :width="7" color="amber" />
    </div>
    <v-progress-linear
      v-if="!autoLoginDone"
      indeterminate
      fixed
      style="z-index: 6"
    />
    <template v-if="autoLoginDone">
      <template v-if="isAuth">
        <app-header @toggle-sidebar="drawer = !drawer" />
        <app-sidebar v-model="drawer" />
      </template>
      <v-main>
        <slot />
      </v-main>
    </template>
  </v-app>
</template>

<style lang="scss">
#loading {
  display: flex;
  justify-content: center;
  align-items: center;
  width: 100%;
  height: 100vh;
  z-index: 9999;
  position: fixed;
  background-color: rgba(#000, 0.5);
}
</style>

<script lang="ts">
import Vue from 'vue';
import { mapState } from 'vuex';
import AppHeader from './AppHeader.vue';
import AppSidebar from './AppSidebar.vue';
import { mapGetters } from 'vuex';
import store from "@/store";

export default Vue.extend({
  components: { AppHeader, AppSidebar },
  data() {
    return { drawer: !this.$vuetify.breakpoint.mdAndDown };
  },
  computed: {
    ...mapState('auth', ['autoLoginDone']),
    ...mapGetters('auth', ['isAuth']),
    loading: function () {
      return store.getters['dialog/isLoadingScreen'];
    },
  },
  watch: {
    loading() {
      // do nothing
    },
  },
});
</script>
