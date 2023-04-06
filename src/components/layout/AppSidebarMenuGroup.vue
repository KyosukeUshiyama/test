<!-- -->
<template>
  <v-list v-if="items.length > 0" nav expand subheader>
    <v-divider v-if="index > 0" />
    <v-subheader v-if="title" class="px-2">
      {{ title }}
    </v-subheader>
    <v-list-item
      v-for="item in items"
      :key="item.title"
      :to="item.to"
      color="primary"
    >
      <v-list-item-content>
        <v-list-item-title>{{ item.title }}</v-list-item-title>
      </v-list-item-content>
    </v-list-item>
  </v-list>
</template>

<script lang="ts">
import Vue from 'vue';

export default Vue.extend({
  name: 'AppSidebarMenuGroup',
  props: {
    index: {
      type: Number,
      default: -1,
    },
    title: {
      type: String,
      default: '',
    },
    items: {
      type: Array,
      default: () => [],
      required: true,
    },
  },
});
</script>
