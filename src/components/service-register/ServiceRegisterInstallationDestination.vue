<!--サービス情報登録_設置先情報-->
<template>
  <div>
    <div class="mb-3">
      <h4>{{ t('installationDestination') }}</h4>
    </div>
    <v-form :disabled="cancelStatus">
      <v-row dense>
        <v-col cols="12" md="6">
          <v-text-field
            :label="t('kanji')"
            :value="customerData.seti_kanji_shamei"
            filled
            readonly
            dense
            hide-details="auto"
          />
        </v-col>
        <v-col cols="12" md="6">
          <v-text-field
            :label="t('furigana')"
            :value="customerData.seti_kn"
            filled
            readonly
            dense
            hide-details="auto"
          />
        </v-col>
      </v-row>
      <v-row dense>
        <v-col cols="12" md="3">
          <v-text-field
            :label="t('agency')"
            :value="customerData.dairiten_mei"
            filled
            readonly
            dense
            hide-details="auto"
          />
        </v-col>
        <v-col cols="12" md="3">
          <v-text-field
            :label="t('personInCharge')"
            :value="customerData.shain_mei"
            filled
            readonly
            dense
            hide-details="auto"
          />
        </v-col>
      </v-row>
      <v-row dense>
        <v-col cols="12" md="3">
          <v-text-field
            :label="t('representative')"
            :value="customerData.seti_tel"
            filled
            readonly
            dense
            hide-details="auto"
          />
        </v-col>
        <v-col cols="12" md="9">
          <v-row dense class="partion-search">
            <v-col cols="12" md="4">
              <v-text-field
                :label="t('partitionAreaCode')"
                :value="customerData.kyokuban"
                filled
                readonly
                dense
                hide-details="auto"
              />
            </v-col>
            <v-col cols="12" md="2">
              <v-text-field
                :label="t('partitionCode')"
                :value="customerData.kukaku_cd"
                filled
                :readonly="customerData.meeting_status==2 || !updatePermission || !createPermission"
                dense
                hide-details="auto"
                @input="emit('kukaku_cd', $event)"
              />
            </v-col>
            <v-col cols="9" md="2">
              <v-text-field
                :label="t('stationName')"
                :value="customerData.kyokusha_mei"
                filled
                readonly
                dense
                hide-details="auto"
              />
            </v-col>
            <v-col cols="3" md="2">
              <v-btn color="primary" :disabled="cancelStatus || !updatePermission || !createPermission" class="mt-2" @click="handlekukakuSerch(customerData.seti_tel)">
                {{ t('partitionSearch') }}
              </v-btn>
            </v-col>
          </v-row>
        </v-col>
      </v-row>
      <v-row dense>
        <v-col cols="12" md="3">
          <v-text-field
            :label="t('postalCode')"
            :value="customerData.seti_yubin_bng"
            filled
            readonly
            dense
            hide-details="auto"
          />
        </v-col>
        <v-col cols="12" md="3">
          <v-text-field
            :label="t('prefectures')"
            :value="customerData.seti_ken_mei"
            filled
            readonly
            dense
            hide-details="auto"
          />
        </v-col>
        <v-col cols="12" md="3">
          <v-text-field
            :label="t('cities')"
            :value="customerData.seti_sikugun_mei"
            filled
            readonly
            dense
            hide-details="auto"
          />
        </v-col>
        <v-col cols="12" md="3">
          <v-text-field
            :label="t('ward')"
            :value="customerData.seti_kmei"
            filled
            readonly
            dense
            hide-details="auto"
          />
        </v-col>
      </v-row>
      <v-row dense>
        <v-col cols="12" md="3">
          <v-text-field
            :label="t('townVillage')"
            :value="customerData.seti_choson_choz"
            filled
            readonly
            dense
            hide-details="auto"
          />
        </v-col>
        <v-col cols="12" md="3">
          <v-text-field
            :label="t('chome')"
            :value="customerData.seti_chom"
            filled
            readonly
            dense
            hide-details="auto"
          />
        </v-col>
        <v-col cols="12" md="3">
          <v-text-field
            :label="t('address')"
            :value="customerData.seti_banti"
            filled
            readonly
            dense
            hide-details="auto"
          />
        </v-col>
        <v-col cols="12" md="3">
          <v-text-field
            :label="t('issue')"
            :value="customerData.seti_go"
            filled
            readonly
            dense
            hide-details="auto"
          />
        </v-col>
      </v-row>
      <v-row dense>
        <v-col cols="12" md="9">
          <v-text-field
            :label="t('buildingName')"
            :value="customerData.seti_ttmnmei"
            filled
            readonly
            dense
            hide-details="auto"
          />
        </v-col>
        <v-col cols="12" md="3">
          <v-text-field
            :label="t('roomNumber')"
            :value="customerData.seti_gosit"
            filled
            readonly
            dense
            hide-details="auto"
          />
        </v-col>
      </v-row>
      <v-row dense>
        <v-col cols="12" md="3">
          <v-text-field
            :label="t('floorNumber')"
            :value="customerData.seti_ksu"
            filled
            readonly
            dense
            hide-details="auto"
          />
        </v-col>
        <v-col cols="12" md="3">
          <v-text-field
            :label="t('totalFloorNumber')"
            :value="customerData.seti_bil_ksu"
            filled
            readonly
            dense
            hide-details="auto"
          />
        </v-col>
      </v-row>
      <v-row dense>
        <v-col cols="12" md="3">
          <v-text-field
            :label="t('departmentInCharge')"
            :value="customerData.seti_tant_bsh_mei"
            filled
            readonly
            dense
            hide-details="auto"
          />
        </v-col>
        <v-col cols="12" md="3">
          <v-text-field
            :label="t('personInCharge')"
            :value="customerData.seti_tantoshamei"
            filled
            readonly
            dense
            hide-details="auto"
          />
        </v-col>
        <v-col cols="12" md="3">
          <v-text-field
            :label="t('phoneNumberInCharge')"
            :value="customerData.seti_tant_tel"
            filled
            readonly
            dense
            hide-details="auto"
          />
        </v-col>
      </v-row>
    </v-form>
  </div>
</template>

<script lang="ts">
import Vue, { PropType } from "vue";
import { TranslateResult } from "vue-i18n";
import { DetailFormCustomerData } from '@/models/customer';
import { getNrNKukakuMstByTaishoNo } from '@/api/nrn-kukaku-mst-api';
import { getScreenId } from '@/utils/screenIds';
import { mapState } from 'vuex';
import { advancedPhoneCheckAPI } from '@/api/advanced-phone-check';

export default Vue.extend({
  name: 'ServiceRegisterView',
  props: {
    customerData: {
      type: Object as PropType<DetailFormCustomerData>,
      default: null,
    },
    cancelStatus: {
      type: Boolean,
      default: false,
    },
    updatePermission: {
      type: Boolean,
      default: false,
    },
    createPermission: {
      type: Boolean,
      default: false,
    }
  },
  computed: {
    ...mapState('customer', ['bbJuchu']),
  },
  methods: {
    t(field: string): TranslateResult {
      return this.$t(`customerDetail.serviceRegister.${field}`);
    },
    emit(fieldName: string, value: string) {
      this.$emit('field-changed', fieldName, value);
    },
    handlekukakuSerch(representative: string): Promise<void> {
      const requestDataForNrn: string = representative.replace(/-/g,"").substr(0,6);
      const requestDataForUf: string = representative.replace(/-/g,"").substr(0,7);
      console.log(requestDataForUf)
      return Promise.all([
        getNrNKukakuMstByTaishoNo(getScreenId(this.$route.name, this.$options.name), requestDataForNrn),
        advancedPhoneCheckAPI(getScreenId(this.$route.name, this.$options.name), requestDataForUf)
      ])
      .then((res) =>
      {
        console.log("NRN区画mst");
        console.log(res);
        if (res[0].data != null) {
          this.customerData.kyokuban = requestDataForUf;
          this.customerData.kukaku_cd = res[0].data.kukaku_cd;
          this.customerData.kyokusha_mei = res[1].data.kyokusha_mei;
        }
      })
      .finally(()=>
      {
        console.log("end");
      });
    },
  },
});
</script>

<style lang="scss" scoped>
.partion-search {
  border: 1px solid #000;
  border-radius: 10px;
  opacity: 0.65;
}
</style>
