<!-- -->
<template>
  <v-navigation-drawer
    clipped
    app
    :value="value"
    :width="sidebarWidth"
    @input="input"
  >
    <div
      v-if="$vuetify.breakpoint.mdAndDown"
      class="sidebar__system-name"
    >
      <h2>
        {{ $t("common.systemName") }}
      </h2>
    </div>
    <app-sidebar-menu />
  </v-navigation-drawer>
</template>

<script lang="ts">
import AppSidebarMenu from './AppSidebarMenu.vue';
import Vue from "vue";

export default Vue.extend({
  components: { AppSidebarMenu },
  props: {
    value: {
      type: Boolean,
      default: true
    }
  },
  computed: {
    screenWidth(): number {
      return this.$vuetify.breakpoint.width;
    },
    sidebarWidth() {
      if (this.screenWidth >= 1904) {
        return 1920 * 0.2;
      }
      if (this.screenWidth >= 1264 && this.screenWidth < 1904) {
        return 1280 * 0.2;
      }
      if (this.screenWidth >= 960 && this.screenWidth < 1264) {
        return 960 * 0.4;
      }
      return '100%';
    }
  },
  methods: {
    input(value: boolean) {
      this.$emit('input', value);
    }
  }
});
</script>

<style lang="scss" scoped>
.sidebar__system-name {
  height: 82px;
  padding: 4px 16px;
  display: flex;
  align-items: center;
}
</style>
