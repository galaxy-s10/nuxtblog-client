<template>
  <div
    class="content"
    style="
      overflow: hidden;
      background: white;
      box-shadow: 0 0 5px rgba(0, 0, 0, 0.1);
      padding: 30px;
    "
  >
    <h1 style="text-align: center; padding: 10px">友链</h1>
    <hr class="hrclass" />
    <ul class="linkWrap">
      <li
        class="liitem"
        v-if="item.status"
        v-for="(item, index) in linkList"
        :key="index"
      >
        <a :href="item.url" class="alink" target="_bank">
          <div class="linkborder">
            <div style="flex: 1">
              <div style="color: #ffa7a6">{{ item.name }}</div>
              <div class="ellipsis">{{ item.description }}</div>
            </div>
            <div>
              <img
                :src="item.avatar"
                class="img"
                style="border: 2px solid #eee"
              />
            </div>
          </div>
        </a>
      </li>
    </ul>
    <div style="text-align: center; margin: 30px 0">
      <h2>欢迎大家交换友链~</h2>
    </div>
    <div style="" v-if="true">
      <el-form
        ref="linkForm"
        :model="linkForm"
        :rules="linkRules"
        label-width="100px"
      >
        <el-form-item label="网站名称" prop="name">
          <el-input
            v-model="linkForm.name"
            placeholder="输入您网站的名称"
          ></el-input>
        </el-form-item>
        <el-form-item label="网站地址" prop="url">
          <el-input
            v-model="linkForm.url"
            placeholder="输入您网站的链接"
          ></el-input>
        </el-form-item>
        <el-form-item label="网站介绍" prop="description">
          <el-input
            v-model="linkForm.description"
            placeholder="简单介绍一下您的网站"
          ></el-input>
        </el-form-item>
        <el-form-item label="网站Logo" prop="avatar">
          <el-input
            v-model="linkForm.avatar"
            placeholder="输入您网站显示的logo"
          ></el-input>
        </el-form-item>
        <el-form-item label="您的邮箱" prop="email">
          <el-input
            v-model="linkForm.email"
            placeholder="审核结果会通过邮件通知您"
          ></el-input>
        </el-form-item>
        <el-form-item label="">
          <el-button type="primary" @click="addLink()">提交申请</el-button>
        </el-form-item>
      </el-form>
    </div>

    <el-input
      type="textarea"
      resize="none"
      :rows="5"
      show-word-limit
      maxlength="200"
      v-model="content"
    ></el-input>
    <p style="text-align: right">
      <el-button type="primary" @click="addcomment()">发表评论</el-button>
    </p>
    <comment
      :list="commentList"
      :allCount="allCount"
      :pageParams="pageParams"
      @reshow="getComment"
      @handleParentPage="parentPage"
      @handleChildrenPage="childrenPage"
      v-loading="isLoading"
    />
    <!-- <comment
      :list="lists"
      :count="count"
      @reshow="commentlist"
      v-loading="isLoading"
    /> -->
  </div>
</template>

<script>
import { format } from "@/utils/format.js";
import comment from "../../components/comment";
import { mapActions, mapMutations } from "vuex";
var validateEmail = (rule, value, callback) => {
  let reg = /^[A-Za-z0-9]+([_\.][A-Za-z0-9]+)*@([A-Za-z0-9\-]+\.)+[A-Za-z]{2,6}$/;
  if (value) {
    if (!reg.test(value)) {
      callback(new Error("请输入正确的邮箱！"));
    }
    return callback();
  } else {
    callback();
  }
};
export default {
  layout: "blog",
  components: {
    comment,
  },
  data() {
    return {
      linkForm: {
        name: "",
        url: "",
        description: "",
        avatar: "",
        email: "",
      },
      linkRules: {
        name: [
          { required: true, message: "网站名称不能为空", trigger: "blur" },
          {
            min: 2,
            max: 50,
            message: "网站名称要求2-50个字符",
            trigger: "blur",
          },
        ],
        url: [
          { required: true, message: "网站地址不能为空", trigger: "blur" },
          {
            max: 100,
            message: "网站地址不能超过100个字符",
            trigger: "blur",
          },
        ],
        description: [
          { required: true, message: "网站介绍不能为空", trigger: "blur" },
          {
            min: 2,
            max: 50,
            message: "网站介绍要求2-50个字符",
            trigger: "blur",
          },
        ],
        avatar: [
          { required: true, message: "网站Logo不能为空", trigger: "blur" },
          {
            max: 100,
            message: "网站Logo不能超过100个字符",
            trigger: "blur",
          },
        ],
        email: [
          {
            validator: validateEmail,
            trigger: "change",
          },
        ],
      },
      linkList: null,
      article_id: -1,
      messagecontent: "",
      isshow: "",
      isLoading: false,
      content: "",
    };
  },
  computed: {
    formatDate(time) {
      return (time) => {
        return format(time);
      };
    },
    userInfo() {
      return this.$store.state.user.id;
    },
  },
  watch: {
    userInfo(val) {
      this.getComment();
    },
  },
  head() {
    return {
      title: "友链 - 自然博客",
      meta: [{ hid: "home", name: "description", content: "自然 - 个人博客" }],
    };
  },
  async mounted() {
    const { code, state } = this.$route.query;
    if (state == 99 && code) {
      try {
        let res = await this.$axios.$get(
          `/api/link/qqlogin?code=${code}&state=${state}`
        );
        function getCookie(name) {
          var arr,
            reg = new RegExp("(^| )" + name + "=([^;]*)(;|$)");
          if ((arr = document.cookie.match(reg))) return unescape(arr[2]);
          else return null;
        }
        const token = getCookie("token");
        if (token) {
          this.setToken(token);
          this.getUserInfo()
            .then(() => {})
            .catch(() => {
              this.logout();
            });
        }
      } catch (err) {
        console.log(err);
      }
    }
  },
  async asyncData({ $axios, params, store }) {
    var id = -1;
    let query = {
      article_id: id,
      nowPage: 1, //当前父评论页数
      pageSize: 3, //当前父评论分页大小
      childrenNowPage: 1, //当前子评论页数
      childrenPageSize: 2, //当前子评论分页大小
    };
    // this.query = query;
    const { rows } = await $axios.$get("/api/link/pageList", {
      params: { nowPage: 1, pageSize: 10 },
    });
    try {
      var data = await $axios.$get(`/api/comment/comment`, {
        params: { ...query },
      });
      return {
        query,
        linkList: rows,
        commentList: data.rows,
        allCount: data.allCount,
        pageParams: {
          ...query,
          count: data.count, //父评论层数
          nowPage: data.nowPage,
          lastPage: data.lastPage,
          pageSize: data.pageSize,
        },
      };
    } catch (err) {
      console.log(err);
    }
  },
  methods: {
    ...mapActions({
      getUserInfo: "user/getUserInfo",
    }),
    ...mapMutations({
      setToken: "user/setToken",
      logout: "user/logout",
    }),
    // 申请友链
    addLink() {
      this.$refs.linkForm.validate(async (valid) => {
        if (valid) {
          try {
            let addLinkRes = await this.$axios.$post(
              "/api/link/add",
              this.linkForm
            );
            this.$newmessage(addLinkRes.message, "success");
            for (let i in this.linkForm) {
              this.linkForm[i] = "";
            }
          } catch (err) {
            this.$newmessage(err.message, "error");
          }
        } else {
          this.$newmessage("请按要求输入正确！", "error");
        }
      });
    },
    // 留言列表
    async getComment() {
      var id = -1;
      let query = {
        article_id: id,
        nowPage: 1,
        pageSize: 3,
        childrenNowPage: 1,
        childrenPageSize: 2,
      };

      try {
        this.isLoading = true;
        var data = await this.$axios.$get(`/api/comment/comment`, {
          params: { ...query },
        });
        setTimeout(() => {
          this.isLoading = false;
          this.pageParams = {
            ...query,
            count: data.count, //父评论层数
            nowPage: data.nowPage,
            lastPage: data.lastPage,
            pageSize: data.pageSize,
          };
          this.commentList = data.rows;
          this.allCount = data.allCount;
        }, 300);
      } catch (err) {
        console.log(err);
        this.isLoading = false;
      }
    },
    // 留言列表
    // async commentlist(id) {
    //   this.isLoading = true;
    //   var res = await this.$axios.$get(`/api/comment/comment?article_id=${id}`);
    //   setTimeout(() => {
    //     this.isLoading = false;
    //     this.lists = res.lists;
    //     this.count = res.count;
    //   }, 300);
    // },
    // 提交留言
    async addcomment() {
      if (this.$store.state.user.token) {
        var article_id = parseInt(this.article_id);
        var from_user_id = parseInt(this.$store.state.user.id);
        var content = this.content;
        var to_comment_id = -1;
        var to_user_id = -1;
        if (content.length >= 3) {
          var res = await this.$axios.$post("/api/comment/add", {
            id: null,
            article_id,
            from_user_id,
            content,
            to_comment_id,
            to_user_id,
          });
          if (res) {
            this.getComment();
            this.$newmessage("发表成功！", "success");
          } else {
            this.$newmessage(res.data, "success");
          }
        } else {
          this.$newmessage("请输入三个及以上内容~~~", "warning");
        }
      } else {
        this.$newmessage("暂未登录，请登录！", "warning");
      }
    },
    // 获取子评论分页
    async childrenPage(query) {
      // async childrenPage(childrenCommentId) {
      query.childrenNowPage += 1;
      let temp = {
        ...this.pageParams,
        ...query,
      };
      this.pageParams = temp;
      var data = await this.$axios.$get(`/api/comment/commentChildrenPage`, {
        params: { ...this.pageParams },
      });
      this.commentList.forEach((item) => {
        if (item.id == data.to_comment_id) {
          item.childrenNowPage = data.childrenNowPage;
          item.childrenLastPage = data.childrenLastPage;
          item.calcSurplus =
            data.count - data.childrenNowPage * data.childrenPageSize;
          item.huifu.push(...data.rows);
        }
      });
    },

    // 获取父评论分页
    async parentPage(query) {
      query.nowPage += 1;
      var data = await this.$axios.$get(`/api/comment/comment`, {
        params: { ...query },
      });
      this.commentList.push(...data.rows);
      this.allCount = data.allCount;
      this.pageParams = {
        ...query,
        count: data.count,
        nowPage: data.nowPage,
        lastPage: data.lastPage,
        pageSize: data.pageSize,
      };
    },
  },
};
</script>

<style scoped>
@media screen and (max-width: 720px) {
  .liitem {
    width: 45% !important;
  }
}
@media screen and (max-width: 540px) {
  .liitem {
    width: 90% !important;
    margin: 15px auto !important;
    float: none !important;
  }
}
.linkWrap {
  max-height: 300px;
  overflow-y: scroll;
  padding: 0;
  margin: 40px 0;
}
.liitem::before {
  content: "";
  background-color: rgba(255, 174, 173, 0.3);
  height: 100%;
  width: 0;
  left: -60px;
  position: absolute;
  transform: skewX(45deg);
  transition: all 0.5s;
  z-index: -1;
}
.liitem:hover {
  z-index: 2;
}
.liitem:hover:before {
  width: 160%;
}
.liitem:hover .linkborder {
  border-color: #ffaead;
}
.liitem:hover .img {
  transform: rotate(360deg);
  transition: all 0.5s;
}
.alink {
  color: #666;
  text-decoration: none;
}
.ellipsis {
  font-size: 13px;
  display: -webkit-box;
  -webkit-box-orient: vertical;
  -webkit-line-clamp: 1;
  overflow: hidden;
  border-top: 2px dashed #eee;
  padding-top: 10px;
  margin-top: 10px;
}
.liitem {
  width: 31.33%;
  box-sizing: border-box;
  margin: 1%;
  display: inline-block;
  position: relative;
  list-style: none;
  overflow: hidden;
}
.linkborder {
  padding: 10px 18px;
  display: flex;
  justify-content: space-between;
  border: 1px solid #eee;
  border-radius: 5px;
}
.img {
  width: 60px;
  height: 60px;
  border-radius: 50%;
  left: 25px;
  transition: all 0.5s;
}
</style>
