<template>
  <!-- 如果有logo 给出logo 50px的高度  -->
  <div :class="{ 'has-logo': showLogo }">
    <!-- 判断侧边栏是否塌陷 -->
    <logo v-if="showLogo" :collapse="isCollapse" />
    <el-scrollbar wrap-class="scrollbar-wrapper">
      <!--
        default-active：激活的菜单选项。 activeMenu 与 route.path 对应
        active-text-color： 激活菜单文本颜色
        collapse：侧边栏是否塌陷
        background-color： 背景颜色
        text-color：菜单文本颜色
        unique-opened： 是否只保持一个子菜单的展开
        collapse-transition： 是否开启折叠动画
        mode="vertical"：模式 判断是顶栏 还是侧边栏
       -->
      <el-menu
        :default-active="activeMenu"
        :active-text-color="variables.menuActiveText"
        :collapse="isCollapse"
        :background-color="variables.menuBg"
        :text-color="variables.menuText"
        :unique-opened="false"
        :collapse-transition="false"
        mode="vertical"
      >
        <!-- 侧边栏item组件  -->
        <sidebar-item
          v-for="route in routes"
          :key="route.path"
          :item="route"
          :base-path="route.path"
        />
      </el-menu>
    </el-scrollbar>
  </div>
</template>

<script>
import { mapGetters } from "vuex";
import Logo from "./Logo";
import SidebarItem from "./SidebarItem";
import variables from "@/styles/variables.scss";

export default {
  components: { SidebarItem, Logo },
  computed: {
    ...mapGetters(["sidebar"]),
    // 获取到所有的的路由信息 ， 做侧边栏的item，传入sidebar-item组件中
    routes() {
      console.log("this.$router?", this.$router);
      return this.$router.options.routes;
    },
    activeMenu() {
      const route = this.$route;
      // 链接地址 https://blog.csdn.net/u014395524/article/details/88194842
      // according to this.$route we can get routes info we registered in router/index.js
      console.log("获取到当前被激活的菜单的route信息", route);
      const { meta, path } = route;
      // if set path, the sidebar will highlight the path you set ,
      // ex : I'll set activeMenu: "/example/table" in dashboard , even i clicked Dashboard it will light up table section! ,
      if (meta.activeMenu) {
        return meta.activeMenu;
      }
      // if didn't set active, deafult path
      return path;
    },
    showLogo() {
      return this.$store.state.settings.sidebarLogo;
    },
    variables() {
      return variables;
    },
    isCollapse() {
      return !this.sidebar.opened;
    },
  },
};
</script>
