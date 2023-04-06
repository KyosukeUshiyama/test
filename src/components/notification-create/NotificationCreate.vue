<!--通知書作成-->
<template>
  <div>
    <h3>{{ screenId }}: {{ t('screenName') }}</h3>
    <div class="ml-10">
      <template v-for="checkbox in checkboxes">
        <v-checkbox
          :key="checkbox.value"
          v-model="selected"
          :label="checkbox.chohyo_mei"
          :value="checkbox.chohyo_cd"
          hide-details="auto"
          @change="selectedCheckBox(checkbox)"
        />
      </template>
    </div>
    <div class="d-flex justify-end align-center" style="display: flex;">
      <div v-if="showCustomerLabel">
        {{ t('customerRabel') }}
      </div>
      <v-radio-group
        v-if="showCustomerLabel"
        v-model="rabel"
        row
      >
        <v-radio
          :label="t('mosikomi')"
          value="mosikomi"
        />
        <v-radio
          :label="t('seti')"
          value="seti"
        />
        <v-radio
          :label="t('seiq')"
          value="seiq"
        />
      </v-radio-group>
      <v-btn :disabled="!selected.length || !publishPermission || kaiyakuStatus" color="primary" @click="confirm">
        {{ t('create') }}
      </v-btn>
    </div>
  </div>
</template>

<script lang="ts">
import { CONFIRMATION, REGISTER_UPDATE_EXECUTE, MESSAGE } from '@/constants/dialog';
import { DialogOptions } from '@/models/dialog';
import { getScreenId } from '@/utils/screenIds';
import Vue, { PropType } from 'vue';
import { TranslateResult } from 'vue-i18n';
import { CustomerServiceResponse, DetailFormCustomerData } from '@/models/customer';
import { chohyoRequest } from '@/models/service/index';
import { ChohyoHakkoRequestdata, ChohyoList } from '@/models/chohyo/index';
import { ChohyoMstCustomerAPI } from '@/api/chohyo-mst-api';
import { ATENA_LABEL_HAKKO_KUBUN, DEFAULT_CHOHYO_OEM_CD } from '@/constant/common-constant';

interface Data {
  selected: TranslateResult[];
  rabel: string;
  checkBox: ChohyoList[];
  screen_id: string;
  atenaLabelChohoFilePath: string;
  showCustomerLabel: boolean;
  publishPermission: boolean;
  kaiyakuStatus: boolean;
}

export default Vue.extend({
  name: 'NotificationCreate',
  props: {
    customerDetail: {
      type: Object as PropType<CustomerServiceResponse>,
      default: null,
    },
    // 新規（BB受注が存在しない）場合: true
    newContract: {
      type: Boolean,
      default: false,
    },
    chohyoMstRequestData: {
      type: Object as PropType<chohyoRequest>,
      default: null
    }
  },
  data(): Data {
    return {
      selected: [],
      rabel: 'seti',
      checkBox: [],
      screen_id: '',
      atenaLabelChohoFilePath : '',
      showCustomerLabel: false,
      publishPermission: false,
      kaiyakuStatus: false,
    };
  },
  computed: {
    screenId(): string {
      return getScreenId(this.$route.name, this.$options.name);
    },
    checkboxes(): ChohyoList[] {
      return this.checkBox;
    },
    customerData(): DetailFormCustomerData {
      return this.customerDetail.dataRegisterDetail
    }
  },
  watch: {
    chohyoMstRequestData:{
      handler: function(){
        if(this.chohyoMstRequestData.ip_type && this.chohyoMstRequestData.dairiten_cd && this.chohyoMstRequestData.kanyu_service_kubun_cd)
        this.createCheckBox()
      }
    },
    selected: {
      handler: function(){
        // 初期化(客先ラベル非表示)
        this.showCustomerLabel = false;
        // 選択したもので宛名ラベル発行区分が1のものがある場合客先ラベルを表示
        this.selected.forEach((checked) => {
          this.checkBox.forEach((item) => {
            if(checked.toString() === item.chohyo_cd && item.atena_label_hakko_kubun === ATENA_LABEL_HAKKO_KUBUN ){
              this.showCustomerLabel = true;
            }
          })
        })
      }
    }
  },
  created() {
    this.screen_id = getScreenId(this.$route.name, this.$options.name)
    if(this.chohyoMstRequestData.ip_type && this.chohyoMstRequestData.dairiten_cd && this.chohyoMstRequestData.kanyu_service_kubun_cd) this.createCheckBox()
    const publish_name = "CreatingNotice-FormPrinting"
    const screen_id_publish = getScreenId(publish_name)
    this.publishPermission = this.$$checkPermission(screen_id_publish)
    this.setKaiyakuStatus()
  },
  methods: {
    t(field: string): TranslateResult {
      return this.$t(`customerDetail.noticeCreate.${field}`);
    },
    // 追加発行コードによるチェック
    selectedCheckBox(element: ChohyoList): void {
      // チェックボックスOFFの際は処理を終了
      if (!this.selected.includes(element.chohyo_cd)) return
      // 追加発行帳票コードの有無を確認
      if (element.tsuika_hakko_chohyo_cd.length > 0) {
        const selectCheck = this.checkboxes.filter(({chohyo_cd}) => element.tsuika_hakko_chohyo_cd.includes(chohyo_cd)).map((item: ChohyoList) => item.chohyo_cd)
        // 追加発行帳票コードの数だけチェックを入れる
        let addedCheckBox = this.selected.concat(selectCheck)
        this.selected = addedCheckBox
      }
      this.$emit('selected')
    },
    confirm(): void {
      // 代理店データを取得
      const dairiten_cd = this.customerDetail.dataRegisterDetail.dairiten_cd;

      const requestData: ChohyoHakkoRequestdata[] = [];
      this.checkBox.forEach((element) => {
        if (this.selected.includes(element.chohyo_cd)) {
          const chohyoRequest: ChohyoHakkoRequestdata = {
            gno: this.customerData.gno,
            chohyo_cd: element.chohyo_cd,
            chohyo_file_path: element.chohyo_file_path,
            object_name: element.object_name,
            seti_no: this.customerData.seti_no,
            mskm_no: this.customerData.mskm_no,
            kkyk_cd: this.customerData.kkyk_cd,
            jusho_flg: this.rabel,
            dairiten_cd:dairiten_cd,
            atena_label_hakko_kubun: element.atena_label_hakko_kubun,
          }
          requestData.push(chohyoRequest)
        }
      })
      let error_message: TranslateResult = ""
      // 各情報がない場合エラーを設定
      if(!this.customerData.seti_no){
        error_message = this.t('messages.no_info.seti')
      }else {
        // 客先ラベルを発行する場合
        if(this.showCustomerLabel){
          // 申込先情報がないかつ申込先を選択した場合
          if(!this.customerData.mskm_no && this.rabel === 'mosikomi'){
            error_message = this.t('messages.no_info.mskm')
          // 請求先情報がないかつ請求先を選択した場合
          }else if(!this.customerData.kkyk_cd && this.rabel === 'seiq'){
            error_message = this.t('messages.no_info.seiq')
          }
        }
      }
      // エラーが設定されている場合は帳票発行APIを実行しない
      if(error_message){
        this.$store.dispatch('dialog/open', {
          type: MESSAGE,
          messages: [error_message],
        } as DialogOptions);
      }else {
        this.$store.dispatch('dialog/open', {
          type: CONFIRMATION,
          subType: REGISTER_UPDATE_EXECUTE,
          messages: [this.t('message')],
          callback: () => {
            this.$emit('confirm', requestData);
          },
        } as DialogOptions);
      }
    },
    setKaiyakuStatus(): void {
      this.kaiyakuStatus = false
      if(this.customerDetail.dataRegisterDetail.uf_toroku_data_sakusei_status == '9') this.kaiyakuStatus = true
    },
    createCheckBox(): void{
      this.checkBox = [];
      // BB受注がなくて保存確認していない場合はチェックボックスを出さない
      if (this.newContract) return
      const requestData:chohyoRequest = this.chohyoMstRequestData

      ChohyoMstCustomerAPI(this.screen_id, requestData)
      .then((res) => {
        console.log(res)
        let defaultAriList:string[] = [];
        // デフォルト代理店CDデータのある帳票コードリストを作成する(memo:基本的には1件以上存在する)
        res.data?.forEach((el) =>
        {
          if(el.oem_cd === DEFAULT_CHOHYO_OEM_CD)
          {
            defaultAriList.push(el.chohyo_cd)
          }
        })
        // デフォルト代理店CDの帳票コードがあるものの内、デフォルト代理店CD以外のものがあるリストを作成する
        let sakujoList:string[] = [];
        res.data?.forEach((el)=>
        {
          if(defaultAriList.includes(el.chohyo_cd) && el.oem_cd !== DEFAULT_CHOHYO_OEM_CD)
          {
            sakujoList.push(el.chohyo_cd)
          }
        })
        console.log("削除対象リスト")
        console.log(sakujoList)

        res.data?.forEach((element) => {
          // 除外対象の場合は表示しない
          if(sakujoList.includes(element.chohyo_cd) && element.oem_cd === DEFAULT_CHOHYO_OEM_CD)
          {
            return
          }
          const chohyo:ChohyoList = {
            chohyo_mei: element.chohyo_mei,
            chohyo_cd: element.chohyo_cd,
            chohyo_file_path: element.chohyo_file_path,
            atena_label_hakko_kubun: element.atena_label_hakko_kubun,
            tsuika_hakko_chohyo_cd: element.tsuika_hakko_chohyo_cd? element.tsuika_hakko_chohyo_cd.split(',') : [],
            object_name: element.object_name,
          }
          this.checkBox.push(chohyo);
        })
      })
    }
  },
});
</script>