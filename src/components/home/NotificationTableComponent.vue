<!--メニュー&Home画面_お知らせ一覧-->
<template>
  <v-container class="mt-10">
    <v-row>
      <v-col cols="12">
        <h3>{{ $t('home.notificationTitleLabel') }}</h3>
      </v-col>
      <v-col cols="12">
        <v-data-table
          dense
          :headers="headers"
          :items="informations"
          :options="tableOptions"
          :loading="loadingInformations"
          class="elevation-1"
          style="white-space:pre-wrap; word-wrap:break-word;"
        >
          <template v-slot:[`item.start_dt`]="{ item }">
            {{ item.start_dt.substr(0, 10) }}
          </template>
          <template #no-data>
            <p>{{ $t('home.messageNotFound') }}</p>
          </template>
        </v-data-table>
      </v-col>
    </v-row>
  </v-container>
</template>


<script lang="ts">
import { InformationItem } from '@/models/service';
import { InformationData } from '@/models/home';
import Vue, { PropType } from 'vue';
import { DataOptions } from 'vuetify';

export default Vue.extend({
  props: {
    tableOptions: {
      type: Object as PropType<DataOptions>,
      default: null,
    },
    informations: {
      type: Array as PropType<InformationItem[]>,
      default: [],
    },
    loadingInformations: {
      type: Boolean,
      default: false,
    },
  },
  data(): InformationData {
    return {
      headers: [
        {
          text: this.$t('home.postDateLabelTable'),
          value: 'start_dt',
          width: '150px',
        },
        {
          text: this.$t('home.postDateMessageLabelTable'),
          value: 'message',
        },
      ],
    };
  },
});
</script>
