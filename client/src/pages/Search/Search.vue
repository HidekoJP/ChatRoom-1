<template>
  <div class="content">
    <!-- 头部 -->
    <div class="top-bar">
      <!-- 搜索框 -->
      <div class="search-div">
        <img
          class="search-img"
          src="../../static/images/search/search.png"
          alt=""
          @click="search"
        />
        <input
          type="search"
          :style="{ width: width + 'px' }"
          placeholder="搜索用户/群"
          class="search"
          v-model="find"
        />
      </div>
      <div class="top-bar-right" @click="back">取消</div>
    </div>
    <!-- 内容 -->
    <div class="main">
      <div class="search-user result">
        <div class="title" v-if="userarr.length > 0">用户</div>
        <div class="list user" v-for="(user, index) in userarr" :key="index">
          <!-- 左边是头像、名字和用户 -->
          <router-link
            :to="{
              path: '/userhome',
              query: { id: user._id, tip: user.usertip, target: 'friend' },
            }"
          >
            <img :src="user.imgurl" alt="" />
            <div class="names">
              <div class="name" v-html="user.name"></div>
              <div class="email" v-html="user.email"></div>
            </div>

            <!-- 右边是进行的操作 -->
            <div class="right-btn send" v-if="user.usertip === 1">发消息</div>
            <div class="right-btn adds" v-else>加好友</div>
          </router-link>
        </div>
      </div>
      <div class="search-user result">
        <div class="title" v-if="grouparr.length > 0">群聊</div>
        <div class="list user" v-for="(group, index) in grouparr" :key="index">
          <router-link
            :to="{
              path: '/userhome',
              query: { id: group._id, tip: group.tips, target: 'group' },
            }"
          >
            <img :src="group.imgurl" alt="" />
            <div class="names">
              <div class="name" v-html="group.name"></div>
              <div class="email" v-html="group.notice"></div>
            </div>

            <div class="right-btn send" v-if="group.tips === 1">发消息</div>
            <div class="right-btn adds" v-else>申请加群</div>
          </router-link>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import {
  findFriend,
  findGroup,
  searchUser,
  searchGroup,
} from "../../request/http";
import { throttle } from "../../util/throttle.js";
export default {
  data() {
    return {
      width: 0,
      userarr: [],
      grouparr: [],
      find: "", // 用户在input框中要搜索的东西
      friendarr: [], // 获取到跟本人id相关的好友列表
      memberarr: [], // 获取到本人添加的群
    };
  },
  created() {
    this.getWidth(); //页面创建时调用

    findFriend({
      id: localStorage.getItem("id"), // 把自己的id返回回去
    }).then((res) => {
      this.friendarr = res.data;
    });

    findGroup({
      id: localStorage.getItem("id"), // 把自己的id返回回去
    }).then((res) => {
      this.memberarr = res.data;
    });
  },
  methods: {
    //动态获取元素的宽度
    getWidth() {
      this.width = window.innerWidth - 73; // 宽
      console.log(this.width);
    },
    //获取关键词
    search: throttle(function () {
      this.userarr = [];
      this.grouparr = [];
      let searchVal = this.find;
      //当字符串长度大于0时才开始进行匹配
      if (searchVal.length > 0) {
        this.searchUser(searchVal);
        this.searchGroup(searchVal);
      }
    }, 1000),
    //寻找关键词匹配的用户.
    searchUser(e) {
      let arr = []; // 只有第一次点击搜索的时候才请求数据库
      // 获取用户列表

      searchUser({}).then((res) => {
        arr = res.data.filter((item) => {
          // 后端返回所有的数据，在这里进行过滤
          return item._id !== localStorage.getItem("id"); // 返回id不等于自身的数组
        });
        this.findHaha("usertip", arr, e);
      });
    },
    // 寻找关键词匹配的群
    searchGroup(e) {
      let arr = [];
      searchGroup({}).then((res) => {
        arr = res.data;
        this.findHaha("tips", res.data, e);
      });
    },
    // 封装函数
    findHaha(role, response, e) {
      let arr = response;
      let exp = eval("/" + e + "/g"); //封装在正则里面
      for (let i = 0; i < arr.length; i++) {
        arr[i].imgurl = this.$store.state.userImg + arr[i].imgurl;
        this.$set(arr[i], role, 0); // 默认找出来的都不是自己的群
        if (role === "usertip") {
          if (arr[i].name.search(e) != -1 || arr[i].email.search(e) != -1) {
            this.isFriend(arr[i]);
            arr[i].name = arr[i].name.replace(
              exp,
              '<span style="color:#4aaaff">' + e + "</span>"
            );
            arr[i].email = arr[i].email.replace(
              exp,
              '<span style="color:#4aaaff">' + e + "</span>"
            );
            this.userarr.push(arr[i]);
          }
        } else {
          if (arr[i].name.search(e) != -1) {
            this.isGroup(arr[i]);
            arr[i].name = arr[i].name.replace(
              exp,
              '<span style="color:#4aaaff">' + e + "</span>"
            );
            this.grouparr.push(arr[i]);
          }
        }
      }
    },
    //判断是否为为好友
    isFriend(e) {
      for (let i = 0; i < this.friendarr.length; i++) {
        // 好友表中跟本人id相关的好友数据全部取出来，即数组arr
        if (this.friendarr[i].friendID === e._id) {
          //好友表中的id与搜索出来的用户id相同，则认为是好友关系
          // tip = 1; //是好友关系  注：这样加属性没有响应式
          this.$set(e, "usertip", 1); // 这样加才有响应式
        }
      }
      // e.tip = tip; //存入搜索出来的数组中
    },
    // 判断是否为自己的群
    isGroup(e) {
      for (let i = 0; i < this.memberarr.length; i++) {
        // 好友表中跟本人id相关的好友数据全部取出来，即数组arr
        if (this.memberarr[i]._id === e._id) {
          //好友表中的id与搜索出来的用户id相同，则认为是好友关系
          // tip = 1; //是好友关系  注：这样加属性没有响应式
          this.$set(e, "tips", 1); // 这样加才有响应式
        }
      }
    },
    //返回上一页
    back() {
      this.$router.back();
    },
  },
};
</script>

<style scoped>
.top-bar {
  z-index: 100;
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 44px;
  background: rgba(255, 255, 255, 1);
  box-shadow: 0 0.5px 0 0 rgba(0, 0, 0, 0.1);
  box-sizing: border-box;
}

.top-bar-right {
  padding-right: 16px;
  font-size: 18px;
  font-weight: 550;
  font-family: PingFangSC-Medium;
  color: black;
  line-height: 44px;
  float: right;
}
.search {
  float: none;
  height: 30px;
  background: rgb(243, 244, 246);
  border-radius: 5px;
  padding: 0 30px 0 6px;
}
.search-div {
  float: left;
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  z-index: -1;
  box-sizing: border-box;
  padding: 7px 59px 7px 16px;
}
.search-img {
  width: 20px;
  height: 20px;
  float: right;
  position: absolute;
  right: 65px;
  top: 12px;
}
.main {
  padding-top: 44px;
}
.result {
  padding-top: 16px;
}
.title {
  height: 30px;
  font-family: PingFangSC-Semibold;
  font-weight: 600;
  font-size: 22px;
  padding-left: 16px;
}
.list {
  width: 100%;
  height: 40px;
  padding: 10px 0;
}
.list img {
  width: 40px;
  height: 40px;
  float: left;
  padding-left: 21px;
}
.names {
  padding-left: 16px;
  float: left;
}
.name {
  font-size: 18px;
  color: rgb(39, 40, 50);
  line-height: 25px;
  font-weight: 600;
}
.right-btn {
  float: right;
  width: 60px;
  height: 24px;
  border-radius: 12px;
  font-weight: 600;
  margin-right: 16px;
  line-height: 24px;
  text-align: center;
  margin-top: 8px;
  font-size: 12px;
}
.email {
  line-height: 14px;
  font-size: 12px;
}
.send {
  background: rgb(255, 228, 49);
  color: rgb(39, 40, 50);
}
.adds {
  background: rgba(74, 170, 255, 0.1);
  color: #4aaaff;
}
</style>