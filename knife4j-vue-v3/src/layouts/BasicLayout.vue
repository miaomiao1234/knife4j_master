<template>
  <div class="BasicLayout">
    <a-layout class="ant-layout-has-sider">
      <a-layout-sider :trigger="null" collapsible :collapsed="collapsed" breakpoint="lg" @collapse="handleMenuCollapse" :width="menuWidth" class="sider">
        <div class="knife4j-logo-data" key="logo" v-if="!collapsed">
          <a to="/" style="float:left;">
            <a-select :value="defaultServiceOption" style="width: 280px" :options="serviceOptions" @change="serviceChange">
            </a-select>
          </a>
        </div>
        <div class="knife4j-logo" key="logo" v-if="collapsed">
          <a to="/" style="float:left;" v-if="collapsed">
            <img :src="logo" alt="logo" />
          </a>
        </div>
        <div class="knife4j-menu">
          <a-menu key="Menu" theme="dark" mode="inline" :inlineCollapsed="collapsed" @openChange="handleOpenChange" @select="selected" :openKeys="openKeys" :selectedKeys="selectedKeys" style="padding: 16px 0; width: 100%">
            <ThreeMenu :menuData="MenuData" :collapsed="collapsed" />
          </a-menu>
        </div>
      </a-layout-sider>
      <!-- <SiderMenu :defaultOption="defaultServiceOption" :serviceOptions="serviceOptions" @menuClick='menuClick' :logo="logo" :menuData="MenuData" :collapsed="collapsed" :location="$route" :onCollapse="handleMenuCollapse" :menuWidth="menuWidth" /> -->
      <a-layout>
        <a-layout-header style="padding: 0;background: #fff;    height: 56px; line-height: 56px;">
          <GlobalHeader @searchKey="searchKey" @searchClear="searchClear" :documentTitle="documentTitle" :collapsed="collapsed" :headerClass="headerClass" :currentUser="currentUser" :onCollapse="handleMenuCollapse" :onMenuClick="item => handleMenuClick(item)" />
        </a-layout-header>
        <context-menu :itemList="menuItemList" :visible.sync="menuVisible" @select="onMenuSelect" />
        <a-tabs hideAdd v-model="activeKey" @contextmenu.native="e => onContextmenu(e)" type="editable-card" @change="tabChange" @edit="tabEditCallback" class="knife4j-tab">
          <a-tab-pane v-for="pane in panels" :key="pane.key" :closable="pane.closable">
            <span slot="tab" :pagekey="pane.key">{{ pane.title }}</span>
            <component :is="pane.content" :data="pane" @childrenMethods="childrenEmitMethod">
            </component>
          </a-tab-pane>
        </a-tabs>
        <a-layout-footer style="padding: 0">
          <GlobalFooter />
        </a-layout-footer>
      </a-layout>
    </a-layout>
  </div>
</template>
<script>
//import logo from "@/assets/logo.png";
import logo from "@/core/logo.js";
import SiderMenu from "@/components/SiderMenu";
import GlobalHeader from "@/components/GlobalHeader";
import GlobalFooter from "@/components/GlobalFooter";
import GlobalHeaderTab from "@/components/GlobalHeaderTab";
import { getMenuData } from "./menu";
import KUtils from "@/core/utils";
import SwaggerBootstrapUi from "@/core/Knife4j.js";
import {
  findComponentsByPath,
  findMenuByKey
} from "@/components/utils/Knife4jUtils";
import { urlToList } from "@/components/utils/pathTools";
import ThreeMenu from "@/components/SiderMenu/ThreeMenu";
//????????????
import ContextMenu from "@/components/common/ContextMenu";

const constMenuWidth = 310;

export default {
  name: "BasicLayout",
  components: {
    SiderMenu,
    GlobalHeader,
    GlobalFooter,
    GlobalHeaderTab,
    ContextMenu,
    ThreeMenu
  },
  data() {
    /* const panes = [
      {
        title: "??????",
        component: "Main",
        content: "Main",
        key: "kmain",
        closable: false
      }
    ]; */
    return {
      logo: logo,
      documentTitle: "",
      menuWidth: constMenuWidth,
      headerClass: "knife4j-header-width",
      swagger: null,
      swaggerCurrentInstance: {},
      defaultServiceOption: "",
      serviceOptions: [],
      MenuData: [],
      collapsed: false,
      linkList: [],
      panels: [],
      panelIndex: 0,
      activeKey: "",
      newTabIndex: 0,
      openKeys: [],
      selectedKeys: [],
      status: false,
      menuVisible: false,
      menuItemList: [
        { key: "1", icon: "caret-left", text: "????????????" },
        { key: "2", icon: "caret-right", text: "????????????" },
        { key: "3", icon: "close-circle", text: "????????????" }
      ]
    };
  },
  beforeCreate() {},
  created() {
    this.initKnife4jSpringUi();
    //this.initKnife4jFront();
  },
  computed: {
    currentUser() {
      return this.$store.state.header.userCurrent;
    },
    cacheMenuData() {
      return this.$store.state.globals.menuData;
    }
  },
  updated() {
    this.openDefaultTabByPath();
    //this.selectDefaultMenu();
  },
  mounted() {
    //this.selectDefaultMenu();
  },
  watch: {
    $route() {
      this.watchPathMenuSelect();
    },
    swaggerCurrentInstance() {
      var title = this.swaggerCurrentInstance.title;
      if (!title) {
        title = "Knife4j ????????????";
      }
      this.documentTitle = title;
      window.document.title = title;
    }
  },
  methods: {
    initKnife4jSpringUi() {
      //???????????????????????????knife4j-spring-ui?????????,????????????????????????
      var that = this;
      //?????????swagger??????
      var url = this.$route.path;
      var plusFlag = false;
      if (url == "/plus") {
        //????????????
        plusFlag = true;
      }
      this.swagger = new SwaggerBootstrapUi({ Vue: that, plus: plusFlag });
      try {
        this.swagger.main();
      } catch (e) {
        console.error(e);
      }
      //?????????????????????
      //?????????????????????
      //this.MenuData = getMenuData();
      //????????????
      this.$store.dispatch("header/getCurrentUser");
    },
    initKnife4jFront() {
      //??????????????????Spring-ui?????????,??????????????????????????????knife4j
      var that = this;
      //?????????swagger??????
      var url = this.$route.path;
      var plusFlag = false;
      if (url == "/plus") {
        //????????????
        plusFlag = true;
      }
      this.swagger = new SwaggerBootstrapUi({
        Vue: that,
        plus: plusFlag,
        //??????config???url??????
        configSupport: false,
        //??????url??????,?????????????????????
        url: "/static/services.json"
      });
      try {
        this.swagger.main();
      } catch (e) {
        console.error(e);
      }
      //?????????????????????
      //?????????????????????
      //this.MenuData = getMenuData();
      //????????????
      this.$store.dispatch("header/getCurrentUser");
    },
    mouseMiddleCloseTab(e) {
      //??????????????????tab??????
      console.log("??????????????????tab??????");
      console.log(e);
    },
    searchClear() {
      //?????????????????????,????????????
      this.MenuData = this.cacheMenuData;
    },
    searchKey(key) {
      //??????????????????
      if (KUtils.strNotBlank(key)) {
        var tmpMenu = [];
        var regx = ".*?" + key + ".*";
        //console.log(this.cacheMenuData);
        this.cacheMenuData.forEach(function(menu) {
          if (KUtils.arrNotEmpty(menu.children)) {
            //??????children
            var tmpChildrens = [];
            menu.children.forEach(function(children) {
              var urlflag = KUtils.searchMatch(regx, children.url);
              var sumflag = KUtils.searchMatch(regx, children.name);
              var desflag = KUtils.searchMatch(regx, children.description);
              if (urlflag || sumflag || desflag) {
                tmpChildrens.push(children);
              }
            });
            if (tmpChildrens.length > 0) {
              var tmpObj = {
                groupName: menu.groupName,
                groupId: menu.groupId,
                key: menu.key,
                name: menu.name,
                icon: menu.icon,
                path: menu.path,
                hasNew: menu.hasNew,
                authority: menu.authority,
                children: tmpChildrens
              };
              tmpMenu.push(tmpObj);
            }
          }
        });
        this.MenuData = tmpMenu;
      }
    },
    serviceChange(value, option) {
      //console("??????????????????");
      var that = this;
      //id
      let swaggerIns = this.swagger.selectInstanceByGroupId(value);
      this.swagger.analysisApi(swaggerIns);
      this.defaultServiceOption = value;
      //console(value);
      //console(option);
      setTimeout(() => {
        that.updateMainTabInstance();
      }, 500);
    },
    onMenuSelect(key, target) {
      let pageKey = this.getPageKey(target);
      switch (key) {
        case "1":
          this.closeLeft(pageKey);
          break;
        case "2":
          this.closeRight(pageKey);
          break;
        case "3":
          this.closeOthers(pageKey);
          break;
        default:
          break;
      }
    },
    onContextmenu(e) {
      const pagekey = this.getPageKey(e.target);
      if (pagekey !== null) {
        e.preventDefault();
        this.menuVisible = true;
      }
    },
    getPageKey(target, depth) {
      depth = depth || 0;
      if (depth > 2) {
        return null;
      }
      let pageKey = target.getAttribute("pagekey");
      pageKey =
        pageKey ||
        (target.previousElementSibling
          ? target.previousElementSibling.getAttribute("pagekey")
          : null);
      return (
        pageKey ||
        (target.firstElementChild
          ? this.getPageKey(target.firstElementChild, ++depth)
          : null)
      );
    },
    closeOthers(pageKey) {
      //??????????????????????????????key??????kmain
      this.linkList = ["kmain", pageKey];
      let tabs = [];
      this.panels.forEach(function(panel) {
        if (panel.key == "kmain" || panel.key == pageKey) {
          tabs.push(panel);
        }
      });
      this.panels = tabs;
      this.activeKey = pageKey;
    },
    closeLeft(pageKey) {
      //????????????,?????????????????????close
      if (this.linkList.length > 2) {
        let index = this.linkList.indexOf(pageKey);
        let sliceArr = this.linkList.slice(index);
        let nLinks = ["kmain"].concat(sliceArr);
        this.linkList = nLinks;
        let kmainComp = this.panels[0];
        let tabs = [];
        tabs.push(kmainComp);
        let splicTabs = this.panels.slice(index);
        this.panels = tabs.concat(splicTabs);
        this.activeKey = pageKey;
      }
    },
    closeRight(pageKey) {
      this.activeKey = pageKey;
      let index = this.linkList.indexOf(pageKey);
      let tmpLinks = [];
      let tmpTabs = [];
      const tpLinks = this.linkList;
      const tpPanels = this.panels;
      for (var i = 0; i <= index; i++) {
        tmpLinks.push(tpLinks[i]);
        tmpTabs.push(tpPanels[i]);
      }
      this.linkList = tmpLinks;
      this.panels = tmpTabs;
    },
    childrenEmitMethod(type, data) {
      this[type](data);
    },
    addGlobalParameters(data) {
      this.swaggerCurrentInstance.globalParameters.push(data);
    },
    openDefaultTabByPath() {
      //?????????????????????Tab?????????
      var that = this;
      const panes = this.panels;
      var url = this.$route.path;
      if (url == "/plus") {
        //??????????????????????????????,?????????????????????????????????
        url = "/home";
      }
      //var menu = findComponentsByPath(url, this.MenuData);
      var menu = findComponentsByPath(url, this.swagger.globalMenuDatas);
      if (menu != null) {
        //?????????????????????????????????????????????
        const indexSize = this.panels.filter(tab => tab.key == "kmain");
        if (indexSize == 0) {
          panes.push({
            title: "??????",
            component: "Main",
            content: "Main",
            key: "kmain",
            instance: this.swaggerCurrentInstance,
            closable: false
          });
          this.linkList.push("kmain");
        }
        const tabKeys = panes.map(tab => tab.key);

        //??????tab???????????????
        if (tabKeys.indexOf(menu.key) == -1) {
          //console(menu);
          //console(this.swaggerCurrentInstance);
          //?????????,???????????????????????????tab??????
          panes.push({
            title: menu.tabName ? menu.tabName : menu.name,
            content: menu.component,
            key: menu.key,
            instance: this.swaggerCurrentInstance,
            closable: menu.key != "kmain"
          });
          this.linkList.push(menu.key);
          this.panels = panes;
        }
        this.activeKey = menu.key;
      } else {
        //??????
        this.activeKey = "kmain";
        this.updateMainTabInstance();
      }
      //this.watchPathMenuSelect();
    },
    updateMainTabInstance() {
      var that = this;
      //??????kmain??????instance????????????
      that.panels.forEach(function(panel) {
        if (panel.key == "kmain") {
          panel.instance = that.swaggerCurrentInstance;
        }
      });
    },
    watchPathMenuSelect() {
      var url = this.$route.path;
      const tmpcol = this.collapsed;
      //console("watch-------------------------");
      const pathArr = urlToList(url);
      //console(pathArr);
      var m = findComponentsByPath(url, this.MenuData);
      //???????????????????????????,???????????????openKeys
      if (!tmpcol) {
        if (pathArr.length == 2) {
          //???????????????
          var parentM = findComponentsByPath(pathArr[0], this.MenuData);
          if (parentM != null) {
            this.openKeys = [parentM.key];
          }
        } else if (pathArr.length == 3) {
          //???????????????
          var parentM = findComponentsByPath(pathArr[1], this.MenuData);
          if (parentM != null) {
            this.openKeys = [parentM.key];
          }
        } else {
          if (m != null) {
            this.openKeys = [m.key];
          }
        }
      }

      //this.selectedKeys = [this.location.path];
      if (m != null) {
        this.selectedKeys = [m.key];
      }
    },
    selectDefaultMenu() {
      var url = this.$route.path;
      const pathArr = urlToList(url);
      var m = findComponentsByPath(url, this.MenuData);
      if (pathArr.length == 2) {
        //???????????????
        var parentM = findComponentsByPath(pathArr[0], this.MenuData);
        if (parentM != null) {
          this.openKeys = [parentM.key];
        }
      } else {
        this.openKeys = [m.key];
      }
      //this.selectedKeys = [this.location.path];
      if (m != undefined && m != null) {
        this.selectedKeys = [m.key];
      }
    },
    menuClick(key) {
      //console("??????click");
      //console(key);
      const panes = this.panels;
      //console(panes);
      const tabKeys = this.panels.map(tab => tab.key);
      // var menu = findComponentsByPath(url, this.MenuData);
      var menu = findMenuByKey(key, this.MenuData);
      //console(menu);
      if (menu != null) {
        //??????tab???????????????
        if (tabKeys.indexOf(menu.key) == -1) {
          //?????????,???????????????????????????tab??????
          panes.push({
            title: menu.name,
            content: menu.component,
            key: menu.key,
            closable: true
          });
          this.linkList.push(menu.key);
          this.panels = panes;
        }
        this.activeKey = menu.key;
      } else {
        //??????
        this.activeKey = "kmain";
        this.updateMainTabInstance();
      }
    },
    tabEditCallback(targetKey, action) {
      this[action](targetKey);
    },
    tabChange(targetKey) {
      //console("tabchange------------");
      //console(targetKey);
      //var menu = findMenuByKey(targetKey, this.MenuData);
      var menu = findMenuByKey(targetKey, this.swagger.globalMenuDatas);
      //console(menu);
      if (menu != null) {
        var path = menu.path;
        this.$router.push({ path: path });
      } else {
        this.$router.push({ path: "/" });
      }
    },
    remove(targetKey) {
      let activeKey = this.activeKey;
      const flag = targetKey == activeKey;
      let lastIndex;
      this.panels.forEach((pane, i) => {
        if (pane.key === targetKey) {
          lastIndex = i - 1;
        }
      });
      const panes = this.panels.filter(pane => pane.key !== targetKey);
      if (panes.length && activeKey === targetKey) {
        if (lastIndex >= 0) {
          activeKey = panes[lastIndex].key;
        } else {
          activeKey = panes[0].key;
        }
      }
      this.panels = panes;
      this.activeKey = activeKey;
      //????????????????????????
      if (flag) {
        this.tabChange(activeKey);
      }
    },
    handleMenuCollapse(collapsed) {
      const tmpColl = this.collapsed;
      this.collapsed = !tmpColl;
      //console("??????selectDefaultMenu");
      this.selectDefaultMenu();
      setTimeout(() => {
        if (tmpColl) {
          this.headerClass = "knife4j-header-width";
          this.menuWidth = constMenuWidth;
        } else {
          this.headerClass = "knife4j-header-width-collapsed";
          this.menuWidth = 80;
          //this.openKeys = [""];
        }
      }, 10);
    },
    handleOpenChange(openKeys) {
      let keys;
      if (openKeys.length > 1) {
        if (openKeys.length > 2) {
          keys = [openKeys[openKeys.length - 1]];
        } else if (openKeys[1].indexOf(openKeys[0]) > -1) {
          keys = [openKeys[0], openKeys[1]];
        } else {
          keys = [openKeys[openKeys.length - 1]];
        }
        this.openKeys = keys;
      } else {
        this.openKeys = openKeys;
      }
    },
    // eslint-disable-next-line
    selected({ item, key, selectedKeys }) {
      this.selectedKeys = selectedKeys;
    },
    // eslint-disable-next-line
    collapsedChange(val, oldVal) {
      // eslint-disable-line
      /* if (val) {
        this.openKeys = [];
      } else {
        const pathArr = urlToList(this.location.path);
        if (pathArr[2]) {
          this.openKeys = [pathArr[0], pathArr[1]];
        } else {
          var m = findComponentsByPath(pathArr[0], this.menuData);
          //this.openKeys = [pathArr[0]];
          this.openKeys = [m.key];
        }
      } */
    }
  }
};
</script>

<style lang="less" scoped></style>
