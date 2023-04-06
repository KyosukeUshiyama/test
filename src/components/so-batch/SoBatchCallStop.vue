<!--通話停止SO一括作成-->
<template>
  <div>
    <v-form ref="form" v-model="valid" @submit.prevent="submit">
      <v-row align="center">
        <v-col cols="12" md="4">
          <v-file-input
            v-model="files"
            class="required-label"
            :label="t('callStopFile')"
            :rules="fileRules"
            accept="text/csv"
          />
        </v-col>
      </v-row>
      <v-row align="center">
        <v-col cols="12" md="4">
          <app-date-picker
            v-model="callStopDate"
            :label="t('callStopDate')"
            :rules="callStopDateRules"
            required
          />
        </v-col>
        <v-col cols="6" md="2">
          <v-btn
            color="primary"
            :disabled="loading"
            :loading="loading"
            @click="submit"
          >
            {{ t('soBatchCreate') }}
          </v-btn>
        </v-col>
        <v-col cols="6" md="2">
          <v-btn
            color="primary"
            :disabled="loading"
            :loading="loading"
            @click="download"
          >
            {{ t('downloadList') }}
          </v-btn>
        </v-col>
      </v-row>
    </v-form>
    <div class="elevation-1 pa-5 mt-5">
      <v-data-table
        :items="customerList"
        :headers="headers"
        item-key="gno"
        :options.sync="tableOptions"
        hide-default-footer
      >
        <template v-slot:top="{ pagination, options, updateOptions }">
          <v-data-footer
            :pagination="pagination"
            :options="options"
            :items-per-page-options="[5, 10, 15, 20, -1]"
            @update:options="updateOptions"
          />
        </template>
        <template #no-data>
          <p>{{ t('table.messageNotFound') }}</p>
        </template>
      </v-data-table>
    </div>
  </div>
</template>

<script lang="ts">
import Vue from 'vue';
import { AxiosError } from 'axios';
import { TranslateResult } from 'vue-i18n';
import AppDatePicker from '../UI/AppDatePicker.vue';
import { DataTableHeader } from 'vuetify';
import { getScreenId } from '@/utils/screenIds';
import { DEFAULT_HYOJI_NUM } from '@/constant/common-constant';
import { callStopSoRegistration } from '@/api/so-batch-api';
import { required, formatDate, datePast, fileRequired } from '@/utils/validation';
import { FormRef, InputValidation } from '@/models/validation';
import { callStopSoResponse } from '@/models/so-batch';
import { FILE_NAME } from '@/constant/common-constant';
import { MESSAGE, MESSAGE_ERROR } from '@/constants/dialog';
import { DialogOptions } from '@/models/dialog';
import dateFormat from "dateformat";

export default Vue.extend({
  components: {
    AppDatePicker
  },
  data() {
    return {
      tableOptions: {
        page: 1,
        itemsPerPage: DEFAULT_HYOJI_NUM,
        sortBy: [],
        sortDesc: [],
        groupBy: [],
        groupDesc: [],
        multiSort: false,
        mustSort: false,
      },
      files: null as null | File,
      callStopDate: '',
      loading: false,
      customerList: [] as callStopSoResponse[],
      valid: false,
      fileRules: [
        (value: File) =>
        fileRequired(
          value,
          this.$t('soBatch.callStopFile')
        ),
      ],
      callStopDateRules: [
        (value: InputValidation) =>
          required(
            value,
            this.$t('soBatch.callStopDate')
          ),
        (value: InputValidation) => formatDate(value),
        (value: InputValidation) => datePast(value),
      ]
     };
  },
  computed: {
    headers(): (DataTableHeader | { text: TranslateResult })[] {
      return [
        {
          text: this.t('callStopTable.gno'),
          value: 'gno',
        },
        {
          text: this.t('callStopTable.customerName'),
          value: 'customer_name',
        },
        {
          text: this.t('callStopTable.status'),
          value: 'status',
        },
      ];
    },
  },
  created() {
    let date = new Date()
    date.setDate(date.getDate() + 1)
    this.callStopDate = dateFormat(date,"yyyy/mm/dd")
  },
  methods: {
    t(field: string): TranslateResult {
      return this.$t(`soBatch.${field}`);
    },
    submit(): void {
      console.log('submit')
      const valid = (this.$refs.form as FormRef).validate();
      if (!valid) {
        console.log('miss')
        return
      }
      const data = new FormData();
      if (this.files !== null) {
        data.append('file', this.files) // ファイル追加
        data.append('tsuwa_teishi_date', this.callStopDate) // 通話停止日追加
        this.$$openLoadingScreen() // Loading処理の呼び出し
        // 通話停止SO一括作成API呼び出し
        callStopSoRegistration(getScreenId(this.$route.name), data)
        .then((res) => {
          console.log(res);
          if (res.data[0].error != null) {
            this.$store.dispatch('dialog/open', {
              type: MESSAGE,
              subType: MESSAGE_ERROR,
              messages: [res.data[0].error],
            } as DialogOptions);
          }
          // SO作成リストを表示
          this.customerList = res.data
        })
        .catch((err: AxiosError) => {
          console.error(err.message)
        })
        .finally(() => {
          this.$$closeLoadingScreen() // Loading処理の解除
        })
      }
    },
    download(): void {
      let file_name = this. convertFileName(FILE_NAME)
      let csv = '\ufeff' + 'GNO,顧客名,ステータス\r\n'
      this.customerList.forEach(el => {
        let line = el['gno'] + ',' + el['customer_name'] + ',' + el['status'] + '\r\n'
        csv += line
      })
      let blob = new Blob([csv], { type: 'text/csv' })
      let link = document.createElement('a')
      link.href = window.URL.createObjectURL(blob)
      link.download = file_name + '.csv'
      link.click()
    },
    convertFileName(file_name: string): string {
      let today = new Date();
      console.log(today.getTime())
      file_name = file_name.replace(/%Y/g, today.getFullYear().toString()) // 年4桁
      file_name = file_name.replace(/%y/g, today.getFullYear().toString().slice(-2)) // 年2桁
      file_name = file_name.replace(/%m/g, ('0' + (today.getMonth()+1).toString()).slice(-2)) // 月
      file_name = file_name.replace(/%d/g, ('0' + today.getDate().toString()).slice(-2)) // 日
      file_name = file_name.replace(/%H/g, ('0' + today.getHours().toString()).slice(-2)) // 時
      file_name = file_name.replace(/%M/g, ('0' + today.getMinutes().toString()).slice(-2)) // 分
      file_name = file_name.replace(/%S/g, ('0' + today.getSeconds().toString()).slice(-2)) // 秒
      file_name = file_name.replace(/%L/g, ('0' + today.getTime().toString()).slice(-3)) // ミリ秒
      console.log(file_name)
      return file_name
    }
  },
});
</script>
