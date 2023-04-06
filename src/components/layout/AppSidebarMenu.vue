<!-- メニュー画面 -->
<template>
  <div class="app-sidebar-menu-wrapper mt-2">
    <app-sidebar-menu-group
      v-for="(menuGroup, index) in menusFilteredByRoles"
      :key="index"
      :index="index"
      :title="menuGroup.title"
      :items="menuGroup.items"
    />
  </div>
</template>

<script lang="ts">
import Vue from 'vue';
import { TranslateResult } from 'vue-i18n';
import { mapGetters } from 'vuex';
import AppSidebarMenuGroup from './AppSidebarMenuGroup.vue';
import
{
   getMenuList,
   MenuResultData,
} from '@/api/auth-api';
import
{
  getRoleId,
  IsAdminRole,
} from '@/utils/helper'
import { getScreenId } from '@/utils/screenIds';

interface State {
  menuGroups: MenuGroup[];
}

interface MenuGroup {
  title?: TranslateResult;
  items: MenuItem[];
}

export interface MenuItem {
  title: TranslateResult;
  to?: string;
  callback?: () => void;
  children?: MenuItem[];
  roles?: string[];
  id?:string;
}

export default Vue.extend({
  components: { AppSidebarMenuGroup },
  data(): State {
    return {
      menuGroups: [],
    };
  },
  computed: {
    ...mapGetters('auth', ['roles']),
    menusFilteredByRoles(): MenuGroup[] {
      return this.filterMenuGroupsByRoles(this.menuGroups);
    },
  },
  created(){
    this.menuGroups = [];
    const roleId = getRoleId();             // ロール種別取得

    // HOMEグループ作成
    let homeGroup:MenuGroup = {items: [
            { title: this.$t('layout.menu.group1.item1'), to: '/', },
          ]};

    const adminFlag = IsAdminRole(roleId);
    this.menuGroups.push(homeGroup);

    if(true == adminFlag)
    {
      console.log("メニュー生成,管理者権限");
      this.menuGroups.push(this.getCusbomerMenuList(adminFlag));
      this.menuGroups.push(this.getSoMenuList(adminFlag));
    }
    else
    {
      console.log("メニュー生成," + roleId);
      getMenuList(getScreenId('Menu'), roleId)
        .then((res) =>
        {
          let kanriList:MenuResultData[] = [];  // 取得失敗時も表示しない
          if(null !=res.data)
          {
            kanriList = res.data;
          }
          this.menuGroups.push(this.getCusbomerMenuList(adminFlag,kanriList));
          this.menuGroups.push(this.getSoMenuList(adminFlag,kanriList));
        })
        .finally(() =>
        {
          console.log("メニュー作成終了")
        })

    }
  },
  methods: {
    // 案件管理メニュー作成
    getCusbomerMenuList(
      adminFlag:boolean,
      kanriList?:MenuResultData[]):MenuGroup
    {
      let resultGroup:MenuGroup = {
        title:this.$t('layout.menu.group2.title'),
        items:[],
      }
      const items:MenuItem[] =  [
      {
        title: this.$t('layout.menu.group2.item1'),
        to: '/customer/search',
        id: getScreenId('SearchCustomer'),
      },
      {
        title: this.$t('layout.menu.group2.item2'),
        to: '/advanced-phone-check',
        id: getScreenId('AdvancedPhoneCheck'),
      },
      {
        title: this.$t('layout.menu.group2.item3'),
        to: '/media-phone-search',
        id: getScreenId('MediaPhoneSearch'),
      },
    ];
      resultGroup.items = this.checkMenuList(items,adminFlag,kanriList);

      return resultGroup;
    },
    // SO管理メニュー作成
    getSoMenuList(
      adminFlag:boolean,
      kanriList?:MenuResultData[]):MenuGroup
    {
      let resultGroup:MenuGroup = {
        title:this.$t('layout.menu.group3.title'),
        items:[],
      }
      const items:MenuItem[] =  [
      {
        title: this.$t('layout.menu.group3.item1'),
        to: '/create-so-batch',
        id: getScreenId('CreateSoBatch') },
      {
        title: this.$t('layout.menu.group3.item2'),
        to: '/so-batch-option',
        id: getScreenId('SoBatchOption') },
      {
        title: this.$t('layout.menu.group3.item3'),
        to: '/so-information-inquiry',
        id: getScreenId('SoInformationInquiry') },
      {
        title: this.$t('layout.menu.group3.item4'),
        to: '/so-batch-input',
        id: getScreenId('SoBatchInput') },
      { 
        title: this.$t('layout.menu.group3.item5'),
        to: '/so-emergency-input-inquiry',
        id: getScreenId('SoEmergencyInputInquiry') },
      { 
        title: this.$t('layout.menu.group3.item6'),
        to: '/so-batch-call-stop',
        id: getScreenId('SoBatchCallStop') },
      ];
      resultGroup.items = this.checkMenuList(items,adminFlag,kanriList);

      return resultGroup;
    },
    // 権限にあるメニューデータを作成する
    checkMenuList(
      items:MenuItem[],
      adminFlag:boolean,
      kanriList?:MenuResultData[]):MenuItem[] 
    {
      let resultItems:MenuItem[] = [];
      items.forEach((element) =>
      {
        let tuika = false;
        if(false == adminFlag)
        {
          kanriList?.forEach((kanriEl) =>
          {
            if(element.id == kanriEl.kino_shubetsu)
            {
              tuika = true;
            }
          })
        }
        else
        {
          tuika = true;
        }
        if(true == tuika)
        {
          resultItems.push(element);
        }
      })

      return resultItems;   
    },
    filterMenuGroupsByRoles(menuGroups: MenuGroup[]): MenuGroup[] {
      console.log("filterMenuGroupsByRoles");
      const result: MenuGroup[] = [];
      for (const group of menuGroups) {
        if (group.items.length > 0) {
          const filterMenu = this.filterMenuItemsByRoles(group.items);
          if (filterMenu.length > 0) {
            result.push({ ...group, items: filterMenu });
          }
        }
      }

      return result;
    },
    filterMenuItemsByRoles(menuItems: MenuItem[]): MenuItem[] {
      const result: MenuItem[] = [];
      for (const item of menuItems) {
        if (item.children?.length && !item.roles) {
          const childrenResult = this.filterMenuItemsByRoles(item.children);
          result.push({ ...item, children: childrenResult });
        } else if (!item.roles?.length) {
          result.push(item);
        } else {
          const userRoles: string[] = this.roles;
          const intersectArray = item.roles.filter((role) =>
            userRoles.includes(role)
          );
          intersectArray.length > 0 && result.push(item);
        }
      }
      return result;
    },
  },
});
</script>
