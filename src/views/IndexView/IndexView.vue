<script setup>
import {
  categoriesAPI,
  threadTypeListAPI,
  RecommendsAPI,
  stickAPI,
  threadListAPI,
  userInfoAPI
} from "@/http/api";
import { ArrowDown, View } from "@element-plus/icons-vue";
import { reactive, onMounted, ref } from "vue";
import PostView from "../../components/PostView/PostView.vue"
import {useStore} from "@/stores/counter"
import Copyright from "@/components/Copyright/Copyright.vue";
// 仓库
const store = useStore();

// 
let pageData = ref([]);
// 类型列表
let typeList = ref([]);
// 
let recommends = ref([]);
let stickList = ref([]);
let activeName = ref("");
let ThreadList = reactive({data:[]});
let page = ref(10);
let fixedParame =  ({dzqSid: "85801523-1681349716759", dzqPf: "pc" }) 
let flexibleParame = reactive({
  parame:{
    perPage:page.value,
    page:1,
    "filter[essence]":0,
    "filter[attention]":0,
    "filter[sort]":1,
    scope:0
  }
});
const tabsList = [
  {name:"不限",type:""},
  {name:"发布时间",type:""},
  {name:"评论时间",type:""},
  {name:"热门内容",type:""}
]


// 网络请求
const getData = function () {
  // 获取个人信息
  if (localStorage.getItem("userId")) {
    userInfoAPI({userId:localStorage.getItem("userId") , ...fixedParame }).then((res) => {
      store.getUserInfo(res.Data)
    })
    .catch((err) => {
        console.log(err);
    });
  }
  // 分类
  categoriesAPI({ ...fixedParame })
    .then((res) => {
      var allNum = res.Data.reduce((acc, item, index) => {
        return acc + item.threadCount;
      }, 0);
      var categories = res.Data;
      categories.unshift({ name: "全部", threadCount: allNum });
      pageData.value = categories;
    })
    .catch((err) => {
      console.log(err);
    });
  // 导航类型
  threadTypeListAPI({ ...fixedParame })
    .then((res) => {
      typeList.value = res.Data;
    })
    .catch((err) => {
      console.log(err);
    });
  // 置顶文章
  stickAPI({ ...fixedParame }).then((res) => {
    stickList.value = res.Data;
  })
  .catch((err) => {
      console.log(err);
  });
};
// 右侧推荐内容
const getRecommends = function () {
  // 右侧推荐内容
  RecommendsAPI({ ...fixedParame }).then(
    (res) => {
      recommends.value = res.Data;
    }
  )
  .catch((err) => {
      console.log(err);
  });
};
// 获取帖子数据
const getThreadList = function (parame) {
  // 右侧推荐内容
  threadListAPI({ ...fixedParame,...parame }).then(
    (res) => {
      ThreadList.data = res.Data;
    }
  ).catch((err) => {
      console.log(err);
  });
  flexibleParame.parame = {perPage:page.value,page:1,"filter[essence]":0,"filter[attention]":0,"filter[sort]":1,scope:0};
};
// 方法
const switchClick = function () {
  getRecommends();
};
const layoutClick = function (item) {
  page.value = 10
  getThreadList({...flexibleParame.parame,"filter[categoryids][0]":item.categoryId});
}
// tabs 筛选
const handleClick = function (pane) {
  page.value = 10
  switch (pane.index) {
    case "0":
      console.log(flexibleParame.parame);
      getThreadList({...flexibleParame.parame});
      break;
    case "1":
      console.log(flexibleParame.parame);
      flexibleParame.parame["filter[essence]"] =  1;
      getThreadList({...flexibleParame.parame});
      break;
    case "2":
      flexibleParame.parame["filter[attention]"] =  1;
      getThreadList({...flexibleParame.parame});
      break;
    default:
      break;
  }
};
// 下拉菜单点击
const dropdownType = function(item) {
  page.value = 10
  getThreadList({...flexibleParame.parame,"filter[types][0]":item.type});
}
const dropdownSort = function(index) {
  page.value = 10
  flexibleParame.parame["filter[sort]"] =  index;
  getThreadList({...flexibleParame.parame});
}


// 滚动监听
const load = function() {
  page.value = page.value + 10
  getThreadList({...flexibleParame.parame});
}
onMounted(() => {
  getData();
  // 右侧推荐内容
  getRecommends();
  // 获取帖子数据
  getThreadList({...flexibleParame.parame});
});
</script>

<template>
  <main class="index">
    <div class="page">
      <!-- 左侧侧边栏 -->
      <div class="baselayout_left">
        <el-card :body-style="{ padding: '0px 0.125rem' }">
          <el-menu
            default-active="0"
            class="el-menu-vertical-demo"
          >
            <el-menu-item
              v-for="(item, index) in pageData"
              :key="item.pid"
              :index="index + ''"
              @click="layoutClick(item)"
            >
              <div class="mentItem">
                <span class="name">{{ item.name }}</span>
                <span style="color: var(--color-text-secondary)">{{
                  item.threadCount
                }}</span>
              </div>
            </el-menu-item>
          </el-menu>
        </el-card>
      </div>
      <!-- 中间内容 -->
      <div class="mian_content fixed" v-infinite-scroll="load" :infinite-scroll-distance="200" :infinite-scroll-immediate="false">
        <el-card class="content_title" :body-style="{ padding: 0 }">
          <template #header>
            <div class="card-header">
              <el-tabs
                v-model="activeName"
                @tab-click="handleClick"
                model-value="all"
              >
                <el-tab-pane label="所有" name="all"></el-tab-pane>
                <el-tab-pane label="精华" name="second"></el-tab-pane>
                <el-tab-pane label="已关注" name="third"></el-tab-pane>
                <el-tab-pane>
                  <template #label>
                    <el-dropdown
                      style="padding-bottom: 2px"
                      size="large"
                      popper-class="dropdown"
                    >
                      <span class="el-dropdown-link">
                        <span style="color: var(--el-text-color-primary)"
                          >类型</span
                        >
                        <el-icon class="el-icon--right">
                          <arrow-down />
                        </el-icon>
                      </span>
                      <template #dropdown>
                        <el-dropdown-menu>
                          <el-dropdown-item
                            v-for="(item, index) in typeList"
                            :key="item.type"
                            @click="dropdownType(item)"
                            >{{ item.name }}</el-dropdown-item
                          >
                        </el-dropdown-menu>
                      </template>
                    </el-dropdown>
                  </template>
                </el-tab-pane>
                <el-tab-pane>
                  <template #label>
                    <el-dropdown
                      style="padding-bottom: 2px"
                      size="large"
                      popper-class="dropdown"
                    >
                      <span class="el-dropdown-link">
                        <span style="color: var(--el-text-color-primary)"
                          >排序</span
                        >
                        <el-icon class="el-icon--right">
                          <arrow-down />
                        </el-icon>
                      </span>
                      <template #dropdown>
                        <el-dropdown-menu>
                          <el-dropdown-item
                            v-for="(item, index) in tabsList"
                            :key="index"
                            @click="dropdownSort(index)"
                            >{{ item.name }}</el-dropdown-item
                          >
                        </el-dropdown-menu>
                      </template>
                    </el-dropdown>
                  </template>
                </el-tab-pane>
              </el-tabs>
              <el-button size="large" type="primary" class="release"
                >发布</el-button
              >
            </div>
          </template>
          <div class="StickyPosts">
            <div class="StickyPosts-c">
              <div class="StickyPosts-m">
                <ul class="StickyPosts-list">
                  <li
                    class="StickyPosts-item flex hideEllipsis"
                    v-for="(item, index) in stickList"
                    :key="index"
                  >
                    <div class="StickyPosts-item-left">置顶</div>
                    <div class="StickyPosts-item-right hideEllipsis">
                      {{ item.title }}
                    </div>
                  </li>
                </ul>
              </div>
            </div>
          </div>
        </el-card>
        <div class="content-m">
          <PostView :data="ThreadList.data.pageData" v-if="ThreadList.data.pageData" />
          <div class="empty" v-if="ThreadList.data.pageData&&ThreadList.data.pageData.length< 1">
            <div class="empty-box">
              <div class="empty-body">
                <span>没有更多内容了</span>
              </div>
            </div>
          </div>
        </div>
      </div>
      <!-- 右侧内容 (推荐内容) -->
      <div class="baselayout_right fixed">
        <el-card
          class="recommend"
          :body-style="{ paddingTop: 0, paddingBottom: 0 }"
        >
          <template #header>
            <div class="recommend-header">
              <span>推荐内容</span>
            </div>
          </template>
          <ul class="recommend-list">
            <li
              class="recommend-list-item"
              v-for="(item, index) in recommends"
              :key="item.threadId"
            >
              <div class="recommend-title">
                <p class="recommend-index">{{ index + 1 }}</p>
                <div class="title-text">{{ item.title }}</div>
              </div>
              <div class="recommend-function">
                <div class="function-left">
                  <span class="icon"
                    ><el-icon><View /></el-icon
                  ></span>
                  <span class="num">{{ item.viewCount }}</span>
                </div>
                <div class="function-right j" v-if="item.displayTag.isEssence">
                  <span>精</span>
                </div>
                <div class="function-right jh" v-if="item.displayTag.isEssence">
                  <span>精华</span>
                </div>
              </div>
            </li>
          </ul>
          <div class="right-recommend-switch" @click="switchClick">
            <div class="recommend-switch">
              <el-icon><Refresh /></el-icon>
              换一批
            </div>
          </div>
        </el-card>
        <Copyright />
      </div>
    </div>
  </main>
</template>
<style >

</style>
<style>
@import url(@/assets/IndexAdapter.css);
@import url(./IndexView.css);
</style>
