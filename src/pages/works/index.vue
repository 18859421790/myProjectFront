<template>
  <div class="works-wrap">
    <h1 class="title">作品集</h1>
    <hr class="hr-class" />
    <h2 class="title-desc">"Less is more"</h2>
    <div class="list-wrap">
      <div
        v-for="item in worksList"
        :key="item.id"
        class="item-wrap"
      >
        <div
          class="bg-url"
          :style="{ backgroundImage: `url(${item.bg_url})` }"
        ></div>
        <div class="item">
          <h2 class="name">{{ item.name }}</h2>
          <div class="other">
            <p class="desc">{{ item.desc }}</p>
            <a
              class="btn"
              :href="item.url"
              target="_blank"
              >预览</a
            >
          </div>
        </div>
      </div>
    </div>
    <div v-if="!worksList.length">暂无作品~</div>
  </div>
</template>

<script>
// eslint-disable-next-line
import { Api } from '@/api';

export default {
  components: {},
  layout: 'blog',
  props: {},
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
      const { data: worksData } = await $myaxios.works.list({
        orderName: 'priority',
        orderBy: 'desc',
      });
      return { worksList: worksData.rows };
    } catch (error) {
      console.log(error);
    }
  },
  data() {
    return {};
  },
  head() {
    return {
      title: '作品 - Billd博客',
      meta: [
        {
          name: 'description',
          content: 'Billd博客 - 作品',
        },
        {
          name: 'keywords',
          content: 'Billd博客 - 作品',
        },
      ],
    };
  },
  computed: {},
  watch: {},
  created() {},
  mounted() {},
  methods: {},
};
</script>

<style lang="scss" scoped>
@media screen and (max-width: 720px) {
  .works-wrap {
    .list-wrap {
      .item-wrap {
        width: 100% !important;
      }
    }
  }
}
@media screen and (max-width: 540px) {
  .works-wrap {
    .list-wrap {
      .item-wrap {
        width: 100% !important;
      }
    }
  }
}

.works-wrap {
  .title {
    display: block;
    text-align: center;
  }

  .title-desc {
    text-align: center;
  }

  .list-wrap {
    display: flex;
    flex-wrap: wrap;
    justify-content: space-between;

    .item-wrap {
      position: relative;
      overflow: hidden;
      box-sizing: border-box;
      margin-bottom: 20px;
      padding: 30px 0;
      width: 48%;
      border-radius: 10px;
      color: $theme-color6;
      text-align: center;
      &:hover {
        .item {
          transform: translateY(0%);
          .other {
            opacity: 1;
          }
        }
        &::after {
          height: 200%;
        }
      }
      .item {
        position: relative;
        z-index: 20;
        transition: all 0.8s cubic-bezier(0.19, 1, 0.22, 1);
        transform: translateY(50%);
        .name {
          display: block;
          margin: 0;
        }
        .other {
          opacity: 0;
          transition: all 0.8s cubic-bezier(0.19, 1, 0.22, 1);

          .desc {
            display: block;
            margin: 10px 6px;
          }
          .btn {
            display: inline-block;
            padding: 10px 20px;
            border-radius: 999px;
            background-color: $theme-color1;
            color: $theme-color6;
            text-decoration: none;
            font-weight: bold;
            font-size: 14px;
            cursor: pointer;
          }
        }
      }
      .bg-url {
        position: absolute;
        top: 0;
        right: 0;
        bottom: 0;
        left: 0;
        background-position: 50%;
        background-size: cover;
        background-repeat: no-repeat;
      }
      &:hover:after {
        transform: translateY(-50%);
      }
      &::after {
        position: absolute;
        top: 0;
        right: 0;
        bottom: 0;
        left: 0;
        display: block;
        width: 100%;
        height: 200%;
        background-image: linear-gradient(
          180deg,
          transparent 0,
          rgba(0, 0, 0, 0.009) 11.7%,
          rgba(0, 0, 0, 0.034) 22.1%,
          rgba(0, 0, 0, 0.072) 31.2%,
          rgba(0, 0, 0, 0.123) 39.4%,
          rgba(0, 0, 0, 0.182) 46.6%,
          rgba(0, 0, 0, 0.249) 53.1%,
          rgba(0, 0, 0, 0.32) 58.9%,
          rgba(0, 0, 0, 0.394) 64.3%,
          rgba(0, 0, 0, 0.468) 69.3%,
          rgba(0, 0, 0, 0.54) 74.1%,
          rgba(0, 0, 0, 0.607) 78.8%,
          rgba(0, 0, 0, 0.668) 83.6%,
          rgba(0, 0, 0, 0.721) 88.7%,
          rgba(0, 0, 0, 0.762) 94.1%,
          rgba(0, 0, 0, 0.79)
        );
        content: '';
        transition: transform 1.4s cubic-bezier(0.19, 1, 0.22, 1);
        transform: translateY(0%);
      }
    }
  }
}
</style>
