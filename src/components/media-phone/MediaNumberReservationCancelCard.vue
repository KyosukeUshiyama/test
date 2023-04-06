<!--メディア電番予約取消ダイアログ画面-->
<template>
  <v-form @submit.prevent="submit">
    <v-card>
      <v-card-title>
        <h5>
          {{ $t('mediaPhoneSearch.cancelDialog.title') }}
        </h5>
      </v-card-title>
      <v-card-title>
        <h5 class="pl-4">
          {{
            $t('mediaPhoneSearch.cancelDialog.confirmationReservationCancel')
          }}
        </h5>
      </v-card-title>
      <v-container>
        <v-card class="scroll" outlined>
          <v-container>
            <ul>
              <li v-for="item in listRemove" :key="item.jigyosha_senyo_no">
                <v-row>
                  <v-col cols="12" md="3">
                    {{ item.jigyosha_senyo_no }}
                  </v-col>
                  <v-col cols="12" md="3">
                    {{ $t('mediaPhoneSearch.cancelDialog.reservationDate') }}
                    ：{{ item.yoyaku_date }}
                  </v-col>
                  <v-col cols="12" md="3">
                    {{
                      $t('mediaPhoneSearch.cancelDialog.reservatioEmployeeCode')
                    }}：{{ item.yoyaku_shain_no }}
                  </v-col>
                  <v-col cols="12" md="3">
                    {{ $t('mediaPhoneSearch.cancelDialog.gno') }}：{{
                      item.gno
                    }}
                  </v-col>
                </v-row>
              </li>
            </ul>
          </v-container>
        </v-card>
      </v-container>
      <v-card-actions>
        <v-spacer />
        <v-container>
          <v-row>
            <v-col cols="12" md="2">
              <v-btn @click="closeDialog()">
                {{ $t('mediaPhoneSearch.cancelDialog.closeButton') }}
              </v-btn>
            </v-col>
            <v-col cols="12" md="2" offset-md="8">
              <v-btn type="submit" color="error">
                {{ $t('mediaPhoneSearch.cancelDialog.cancelButton') }}
              </v-btn>
            </v-col>
          </v-row>
        </v-container>
      </v-card-actions>
    </v-card>
  </v-form>
</template>

<script lang="ts">
import Vue, { PropType } from 'vue';
import { MediaPhoneSearchResultItem } from '@/models/media-phone';

export default Vue.extend({
  props: {
    listRemove: {
      type: Array as PropType<MediaPhoneSearchResultItem[]>,
      default: [],
    },
  },
  methods: {
    async submit() {
      this.$emit('cancel-confirm');
    },
    closeDialog() {
      this.$emit('close-dialog');
    },
  },
});
</script>

<style lang="scss">
.scroll {
  height: 100px;
  max-height: 100px;
  overflow-y: scroll;
}
</style>
