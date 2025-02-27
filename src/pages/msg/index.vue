<template>
  <div class="msg-wrap">
    <h1 class="title">留言板</h1>
    <hr class="hr-class" />
    <h2 class="title-desc">欢迎大家留言评论~</h2>

    <!-- 发表留言 -->
    <div class="comment-wrap">
      <TextareaInputCpt @contentChange="contentChange"></TextareaInputCpt>
      <div
        v-if="frontendData"
        class="btn"
      >
        <el-button
          type="primary"
          :loading="submitCommentLoading"
          :disabled="frontendData.allow_comment.value !== '1'"
          @click="addComment"
        >
          发表留言
        </el-button>
      </div>
    </div>

    <!-- 评论组件 -->
    <CommentCpt
      v-loading="isLoading"
      :list="commentList"
      :total="total"
      :sort="sort"
      :has-more="hasMore"
      :now-page="nowPage"
      :page-size="pageSize"
      :children-page-size="childrenPageSize"
      @refresh="refreshCommentList"
      @sortChange="sortChange"
      @handleParentPage="handleParentPage"
      @handleChildrenPage="handleChildrenPage"
    />
  </div>
</template>

<script>
import { mapMutations } from 'vuex';

// eslint-disable-next-line
import { Api } from '@/api';
import CommentCpt from '@/components/Comment/index.vue';
import TextareaInputCpt from '@/components/TextareaInput/index.vue';

export default {
  components: {
    CommentCpt,
    TextareaInputCpt,
  },
  layout: 'blog',
  /**
   * @typedef {Object} asyncDataType
   * @property {Api} $myaxios
   * @property {Object} store
   * @property {Object} params
   * @property {Object} req
   * @param {asyncDataType} value
   * https://nuxtjs.org/docs/concepts/context-helpers
   */
  async asyncData({ $myaxios, store, params, req }) {
    try {
      const orderName = 'created_at';
      const commentParams = {
        article_id: -1,
        nowPage: 1,
        pageSize: 10,
        childrenPageSize: store.state.comment.childrenPageSize, // 子评论分页大小
        orderName,
        orderBy: 'desc',
      };
      const { data: commentData } = await $myaxios.comment.comment(
        commentParams
      );
      return {
        sort: orderName === 'created_at' ? 'date' : 'hot',
        commentList: commentData.rows,
        total: commentData.total,
        hasMore: commentData.hasMore,
        nowPage: commentParams.nowPage,
        pageSize: commentParams.pageSize,
      };
    } catch (error) {
      console.log(error);
    }
  },
  data() {
    return {
      sort: '',
      commentList: [],
      total: 0,
      hasMore: false,
      nowPage: 1,
      pageSize: 10,
      submitCommentLoading: false,
      currentComponent: '',
      article_id: -1,
      commentContent: '',
      isshow: '',
      isLoading: false,
      content: '',
    };
  },
  head() {
    return {
      title: '留言 - Billd博客',
      meta: [
        {
          name: 'description',
          content: 'Billd博客 - 留言',
        },
        {
          name: 'keywords',
          content: 'Billd博客 - 留言',
        },
      ],
    };
  },
  computed: {
    frontendData() {
      return this.$store.state.app.frontendData;
    },
    userInfo() {
      return this.$store.state.user.userInfo;
    },
    childrenPageSize() {
      return this.$store.state.comment.childrenPageSize;
    },
  },
  watch: {
    userInfo() {
      this.refreshCommentList();
    },
  },
  mounted() {},
  methods: {
    ...mapMutations({
      setShowLoginModal: 'app/setShowLoginModal',
    }),
    contentChange(newVal, oldVal) {
      this.commentContent = newVal;
    },

    // 新增回复
    async addComment() {
      if (!this.userInfo) {
        this.setShowLoginModal(true);
        this.$newmessage('暂未登录，请登录！', 'warning');
        return;
      }
      if (this.commentContent.length < 5) {
        this.$newmessage('评论内容至少5个字符~', 'warning');
        return;
      }
      try {
        this.submitCommentLoading = true;
        await this.$myaxios.comment.create({
          article_id: -1,
          content: this.commentContent,
          parent_comment_id: -1,
          reply_comment_id: -1,
          to_user_id: -1,
        });
        this.submitCommentLoading = false;
        this.$newmessage('评论成功~', 'success');
        this.refreshCommentList();
      } catch (error) {
        console.log(error);
        this.submitCommentLoading = false;
      }
    },
    sortChange(sort) {
      this.sort = sort;
      this.refreshCommentList();
    },
    // 留言列表
    async refreshCommentList() {
      const query = {
        article_id: -1,
        nowPage: 1,
        pageSize: 10,
        childrenPageSize: this.childrenPageSize,
        orderName: this.sort === 'date' ? 'created_at' : 'star_total',
        orderBy: 'desc',
      };
      try {
        this.isLoading = true;
        const { data } = await this.$myaxios.comment.comment(query);
        this.isLoading = false;
        this.commentList = data.rows;
        this.total = data.total;
        this.hasMore = data.hasMore;
      } catch (error) {
        console.log(error);
        this.isLoading = false;
      }
    },

    // 获取子评论分页
    async handleChildrenPage(query) {
      try {
        const { data } = await this.$myaxios.comment.commentChildren({
          parent_comment_id: query.parent_comment_id,
          article_id: query.article_id,
          pageSize: query.childrenPageSize,
          childrenPageSize: this.childrenPageSize,
        });
        this.commentList.forEach((item) => {
          if (item.id === query.parent_comment_id) {
            item.children_comment.push(...data.rows);
          }
        });
      } catch (error) {
        console.log(error);
      }
    },
    // 获取父评论分页
    async handleParentPage(query) {
      try {
        const { data } = await this.$myaxios.comment.comment({
          article_id: -1,
          nowPage: query.nowPage + 1,
          pageSize: this.pageSize,
          childrenPageSize: this.childrenPageSize,
          orderName: query.orderName,
          orderBy: query.orderBy,
        });
        this.commentList.push(...data.rows);
        this.hasMore = data.hasMore;
      } catch (error) {
        console.log(error);
      }
    },
  },
};
</script>

<style lang="scss" scoped>
.msg-wrap {
  .title {
    display: block;
    text-align: center;
  }
  .title-desc {
    display: block;
    text-align: center;
  }
  .comment-wrap {
    .btn {
      margin-top: 20px;
      text-align: right;
    }
  }
}
</style>
