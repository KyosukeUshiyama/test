<!-- バリデーションチェックコンポーネント NumberSize -->
<template>
  <v-text-field
    v-bind="$attrs"
    :label="label"
    :rules="[numbersize]"
    :size="size"
    @input="handleInput"
  />
</template>

<script lang="ts">
import Vue from 'vue';
import VueI18n from 'vue-i18n';
import { checkNumberSize } from '@/utils/helper';

export default Vue.extend({
  props: {
    label: {
      type: String,
      default: '',
    },
    size: {
      type: Number,
      default: 0
    }
  },

  methods: {
    // 入力規則: 指定されたサイズの桁数まで
    numbersize(value: string): boolean | VueI18n.TranslateResult {
      let rtn;
      rtn = checkNumberSize(value, this.size)
      // {size}桁の数値で入力してください
      return rtn || this.$t('common.UIMsgErrNumberSize', { size: this.size.toString(10) })
    },
    handleInput(value: string): void {
      this.$emit('input', value);
    },  
  }
});
</script>