<template>
  <div class="home-wrapper">
    <!-- banner块 s-->
    <div class="banner" :class="{ 'hide-banner': !showBanner }" :style="bannerBgStyle">
      <div class="banner-conent" :style="!homeData.features && !homeData.heroImage && `padding-top: 7rem`
        ">
        <header class="hero">
          <img v-if="homeData.heroImage" :src="$withBase(homeData.heroImage)" :alt="homeData.heroAlt" />
          <h1 v-if="homeData.heroText" id="main-title">
            <span>
              {{ homeData.heroText }}
            </span>
            <transition name="fade">
              <span v-show="isShow">_</span>
            </transition>
          </h1>
          <!-- <p v-if="homeData.tagline" class="description">
            {{ homeData.tagline }}
          </p> -->

          <!-- 循环打印 -->
          <BasePrintText :text="printText" @refetch="refetch" />

          <p class="action" v-if="homeData.actionText && homeData.actionLink">
            <NavLink class="action-button" :item="actionLink" />
          </p>
        </header>

        <!-- PC端features块 s -->
        <div class="features" v-if="hasFeatures && !isMQMobile">
          <div class="feature" v-for="(feature, index) in homeData.features" :key="index">
            <router-link v-if="feature.link" :to="feature.link">
              <img class="feature-img" v-if="feature.imgUrl" :src="$withBase(feature.imgUrl)" :alt="feature.title" />
              <h2>{{ feature.title }}</h2>
              <p>{{ feature.details }}</p>
            </router-link>
            <a v-else href="javascript:;">
              <img class="feature-img" v-if="feature.imgUrl" :src="$withBase(feature.imgUrl)" :alt="feature.title" />
              <h2>{{ feature.title }}</h2>
              <p>{{ feature.details }}</p>
            </a>
          </div>
        </div>
        <!-- PC端features块 e -->

        <!-- 卡片按钮组 -->
        <div class="btns">
          <div class="card_btn" @click="openWeb('personal_daohang')">
            <div class="name">前端森林</div>
            <div class="desc">🚀 前端资源一站导航</div>
          </div>
          <button @click="scrollToMain">立即开始</button>
          <div class="card_btn" @click="openWeb('personal_photo')">
            <div class="name">默城·shuai</div>
            <div class="desc">📸 我的摄影作品集</div>
          </div>
        </div>
      </div>

      <!-- 移动端features块 s -->
      <!-- isMQMobile放到v-if上线后会报错 -->
      <div class="slide-banner" v-if="hasFeatures" v-show="isMQMobile">
        <div class="banner-wrapper">
          <div class="slide-banner-scroll" ref="slide">
            <div class="slide-banner-wrapper">
              <div class="slide-item" v-for="(feature, index) in homeData.features" :key="index">
                <router-link v-if="feature.link" :to="feature.link">
                  <img class="feature-img" v-if="feature.imgUrl" :src="$withBase(feature.imgUrl)" :alt="feature.title" />
                  <h2>{{ feature.title }}</h2>
                  <p>{{ feature.details }}</p>
                </router-link>
                <a v-else href="javascript:;">
                  <img class="feature-img" v-if="feature.imgUrl" :src="$withBase(feature.imgUrl)" :alt="feature.title" />
                  <h2>{{ feature.title }}</h2>
                  <p>{{ feature.details }}</p>
                </a>
              </div>
            </div>
          </div>
          <div class="docs-wrapper">
            <span class="doc" v-for="(item, index) in homeData.features.length" :key="index"
              :class="{ active: currentPageIndex === index }"></span>
          </div>
        </div>
      </div>
      <!-- 移动端features块 e -->
    </div>
    <!-- banner块 e -->

    <div class="main-layout-mask">
      <MainLayout>
        <template #mainLeft>
          <!-- 简约版文章列表 -->
          <UpdateArticle class="card-box" v-if="homeData.postList === 'simple'"
            :length="homeData.simplePostListLength || 10" :moreArticle="$themeConfig.updateBar && $themeConfig.updateBar.moreArticle
              " />

          <!-- 详情版文章列表 -->
          <template v-else-if="!homeData.postList || homeData.postList === 'detailed'">
            <PostList :currentPage="currentPage" :perPage="perPage" />
            <Pagination :total="total" :perPage="perPage" :currentPage="currentPage" @getCurrentPage="handlePagination"
              v-show="Math.ceil(total / perPage) > 1" />
          </template>

          <Content class="theme-vdoing-content custom card-box" />
        </template>

        <template v-if="!homeData.hideRightBar" #mainRight>
          <BloggerBar v-if="$themeConfig.blogger" />
          <CategoriesBar v-if="$themeConfig.category !== false &&
            $categoriesAndTags.categories.length
            " :categoriesData="$categoriesAndTags.categories" :length="10" />
          <TagsBar v-if="$themeConfig.tag !== false && $categoriesAndTags.tags.length" :tagsData="$categoriesAndTags.tags"
            :length="30" />
          <div class="custom-html-box card-box" v-if="homeSidebarB" v-html="homeSidebarB"></div>
        </template>
      </MainLayout>
    </div>
  </div>
</template>

<script>
import NavLink from "@theme/components/NavLink";
import BScroll from "@better-scroll/core";
import Slide from "@better-scroll/slide";
import MainLayout from "@theme/components/MainLayout";
import PostList from "@theme/components/PostList";
import UpdateArticle from "@theme/components/UpdateArticle";
import Pagination from "@theme/components/Pagination";
import BloggerBar from "@theme/components/BloggerBar";
import CategoriesBar from "@theme/components/CategoriesBar";
import TagsBar from "@theme/components/TagsBar";
import BasePrintText from "@theme/components/BasePrintText";

const MOBILE_DESKTOP_BREAKPOINT = 720; // refer to config.styl

BScroll.use(Slide);

export default {
  data() {
    return {
      isMQMobile: false,

      slide: null,
      currentPageIndex: 0,
      playTimer: 0,
      mark: 0,

      total: 0, // 总长
      perPage: 10, // 每页长
      currentPage: 1, // 当前页

      printText: "",
      isShow: true,
      timer: null,
    };
  },
  computed: {
    homeData() {
      return {
        ...this.$page.frontmatter,
      };
    },
    hasFeatures() {
      return !!(this.homeData.features && this.homeData.features.length);
    },
    homeSidebarB() {
      const { htmlModules } = this.$themeConfig;
      return htmlModules ? htmlModules.homeSidebarB : "";
    },
    showBanner() {
      // 当分页不在第一页时隐藏banner栏
      return this.$route.query.p &&
        this.$route.query.p != 1 &&
        (!this.homeData.postList || this.homeData.postList === "detailed")
        ? false
        : true;
    },
    bannerBgStyle() {
      let bannerBg = this.homeData.bannerBg;
      if (!bannerBg || bannerBg === "auto") {
        // 默认
        if (this.$themeConfig.bodyBgImg) {
          // 当有bodyBgImg时，不显示背景
          return "";
        } else {
          // 网格纹背景
          return "background: rgb(40,40,45) url(data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAACMAAAAjCAYAAAAe2bNZAAAAAXNSR0IArs4c6QAAAARnQU1BAACxjwv8YQUAAAAJcEhZcwAADsMAAA7DAcdvqGQAAABOSURBVFhH7c6xCQAgDAVRR9A6E4hLu4uLiWJ7tSnuQcIvr2TRYsw3/zOGGEOMIcYQY4gxxBhiDDGGGEOMIcYQY4gxxBhiDLkx52W4Gn1tuslCtHJvL54AAAAASUVORK5CYII=)";
        }
      } else if (bannerBg === "none") {
        // 无背景
        if (this.$themeConfig.bodyBgImg) {
          return "";
        } else {
          return "background: var(--mainBg);color: var(--textColor)";
        }
      } else if (bannerBg.indexOf("background:") > -1) {
        // 自定义背景样式
        return bannerBg;
      } else if (bannerBg.indexOf(".") > -1) {
        // 大图
        return `background: url(${this.$withBase(
          bannerBg
        )}) center center / cover no-repeat`;
      }
    },
    actionLink() {
      return {
        link: this.homeData.actionLink,
        text: this.homeData.actionText,
      };
    },
  },
  components: {
    NavLink,
    MainLayout,
    PostList,
    UpdateArticle,
    BloggerBar,
    CategoriesBar,
    TagsBar,
    Pagination,
    BasePrintText,
  },
  created() {
    this.total = this.$sortPosts.length;
  },
  beforeMount() {
    document.title = "ヾ(^▽^*)))欢迎访问";
    this.isMQMobile =
      window.innerWidth < MOBILE_DESKTOP_BREAKPOINT ? true : false; // vupress在打包时不能在beforeCreate(),created()访问浏览器api（如window）
  },
  mounted() {
    this.timer = setInterval(() => {
      this.isShow = !this.isShow;
    }, 800);
    this.fetchYiYan();
    if (this.$route.query.p) {
      this.currentPage = Number(this.$route.query.p);
    }

    if (
      this.hasFeatures &&
      this.isMQMobile &&
      (!this.$route.query.p || this.$route.query.p == 1)
    ) {
      this.init();
    }

    if (this.hasFeatures) {
      window.addEventListener("resize", () => {
        this.isMQMobile =
          window.innerWidth < MOBILE_DESKTOP_BREAKPOINT ? true : false;
        if (this.isMQMobile && !this.slide && !this.mark) {
          this.mark++;
          setTimeout(() => {
            this.init();
          }, 60);
        }
      });
    }
  },
  beforeDestroy() {
    clearInterval(this.timer);
    this.timer = null;
    clearTimeout(this.playTimer);
    this.slide && this.slide.destroy();
  },
  watch: {
    "$route.query.p"() {
      if (!this.$route.query.p) {
        this.currentPage = 1;
      } else {
        this.currentPage = Number(this.$route.query.p);
      }

      if (this.hasFeatures && this.currentPage === 1 && this.isMQMobile) {
        setTimeout(() => {
          this.slide && this.slide.destroy();
          this.init();
        }, 0);
      }
    },
  },
  methods: {
    openWeb(curWeb) {
      const webType = {
        personal_daohang:
          "http://mochenghualei.gitee.io/my-personal-website/#/",
        personal_photo: "http://82.157.148.19:81/",
      };
      window.open(webType[curWeb], "_blank");
    },
    scrollToMain() {
      document
        .querySelector(".main-layout-mask")
        .scrollIntoView({ behavior: "smooth" });
    },
    refetch(state) {
      if (state) {
        this.fetchYiYan();
      }
    },
    fetchYiYan() {
      // 请求一言
      fetch("https://v1.hitokoto.cn?c=d&c=k")
        .then((response) => response.json())
        .then((data) => {
          this.printText = data.hitokoto + " -- " + (data.from_who || "佚名");
        })
        .catch(console.error);
    },
    init() {
      clearTimeout(this.playTimer);
      this.slide = new BScroll(this.$refs.slide, {
        scrollX: true, // x轴滚动
        scrollY: false, // y轴滚动
        slide: {
          loop: true,
          threshold: 100,
        },
        useTransition: true, // 使用css3 transition动画
        momentum: false,
        bounce: false, // 回弹
        stopPropagation: false, // 是否阻止事件冒泡
        probeType: 2,
        preventDefault: false,
      });

      // user touches the slide area
      this.slide.on("beforeScrollStart", () => {
        clearTimeout(this.playTimer);
      });
      // user touched the slide done
      this.slide.on("scrollEnd", () => {
        this.autoGoNext();
      });
      this.slide.on("slideWillChange", (page) => {
        this.currentPageIndex = page.pageX;
      });
      this.autoGoNext();
    },
    autoGoNext() {
      clearTimeout(this.playTimer);
      this.playTimer = setTimeout(() => {
        this.slide.next();
      }, 4000);
    },
    handlePagination(i) {
      // 分页
      this.currentPage = i;
    },
    getScrollTop() {
      return (
        window.pageYOffset ||
        document.documentElement.scrollTop ||
        document.body.scrollTop
      );
    },
  },
};
</script>

<style lang="stylus" scoped>
.home-wrapper
  .banner
    width 100%
    min-height calc(100vh - 3rem)
    margin-top $navbarHeight
    color $bannerTextColor
    position relative
    overflow hidden
    .banner-conent
      max-width $homePageWidth
      margin 0px auto
      position relative
      z-index 1
      overflow hidden
      .hero
        text-align center
        // margin-top 3.5rem
        img
          max-width 100%
          max-height 240px
          display block
          margin 2rem auto 1.5rem
        h1
          margin-bottom 3rem
          span:first-child
            font-size 4.2rem
          span:last-child
            font-size 2.5rem

        .description, .action
          margin 1.5rem auto
        .description
          max-width 40rem
          font-size 1.1rem
          line-height 1.3
          opacity 0.9
        .action-button
          display inline-block
          font-size 1.2rem
          background-color $accentColor
          padding 0.8rem 1.6rem
          border-radius 4px
          transition background-color 0.1s ease
          box-sizing border-box
          border-bottom 1px solid darken($accentColor, 10%)
          color #fff
          &:hover
            background-color lighten($accentColor, 10%)
      // pc端features
      .features
        padding 2rem 0
        margin-top 4rem
        margin-bottom 2rem
        display flex
        flex-wrap wrap
        align-items flex-start
        align-content stretch
        justify-content space-between
      .feature
        flex-grow 1
        flex-basis 30%
        max-width 30%
        text-align center
        a
          // color lighten($bannerTextColor,10%)
          color inherit
          .feature-img
            width 10rem
            height 10rem
            animation heart 1.2s ease-in-out 0s infinite alternate
            animation-play-state paused
          h2
            font-weight 500
            font-size 1.3rem
            border-bottom none
            padding-bottom 0
          p
            opacity 0.8
            padding 0 0.8rem
      .feature:hover
        .feature-img
          animation-play-state running
        h2, p
          color $accentColor
    // 移动端滑动图标
    .slide-banner
      margin-top 2rem
      .banner-wrapper
        position relative
      .slide-banner-scroll
        min-height 1px
        overflow hidden
      .slide-banner-wrapper
        height 300px
        .slide-item
          display inline-block
          height 300px
          width 100%
          text-align center
          a
            // color lighten($bannerTextColor,10%)
            color inherit
            .feature-img
              width 10rem
              height 10rem
            h2
              font-size 1.1rem
              font-weight 500
              border-bottom none
              padding-bottom 0
            p
              opacity 0.8
              padding 0 0.8rem
      .docs-wrapper
        position absolute
        bottom 25px
        left 50%
        transform translateX(-50%)
        .doc
          display inline-block
          margin 0 4px
          width 8px
          height 8px
          border-radius 50%
          background var(--textColor)
          opacity 0.9
          &.active
            opacity 0.5
  // 分页不在第一页时，隐藏banner栏
  .banner.hide-banner
    display none
    & + .main-wrapper
      margin-top: ($navbarHeight + 0.9rem)
  .main-wrapper
    margin-top 2rem
    .main-left
      .card-box
        margin-bottom 2rem
      .pagination
        margin-bottom 3rem
      .theme-vdoing-content
        padding 0 2rem
        overflow hidden
        border none
        &>:first-child
          padding-top 2rem
        &>:last-child
          padding-bottom 2rem
    .main-right
      .custom-html-box
        padding 0
        overflow hidden
@keyframes heart
  from
    transform translate(0, 0)
  to
    transform translate(0, 8px)
// 1025px以下
@media (max-width 1025px)
  .home-wrapper
    .banner
      .banner-conent
        .hero
          h1
            font-size 2.5rem
          .description
            font-size 1rem
        .feature
          a
            h2
              font-size 1.1rem
            .feature-img
              width 9rem
              height 9rem
// 719px以下
@media (max-width $MQMobile)
  .home-wrapper
    .banner
      .banner-conent
        .features
          display none !important
// 419px以下
@media (max-width $MQMobileNarrow)
  .home-wrapper
    .banner-conent
      padding-left 1.5rem
      padding-right 1.5rem
      .hero
        img
          max-height 210px
          margin 2rem auto 1.2rem
        h1
          font-size 2rem
        h1, .description, .action
          margin 1.2rem auto
        .description
          font-size 1.2rem
        .action-button
          font-size 1rem
          padding 0.6rem 1.2rem
      .feature
        h2
          font-size 1.25rem
.theme-style-line
  .main-wrapper
    @media (max-width 719px)
      margin-top -1px


.fade-enter-active, .fade-leave-active
  transition opacity .8s

.fade-enter, .fade-leave-to
  opacity 0

.main-layout-mask
  padding-top 2rem
  width 100%



.btns
  display flex
  padding 0 15rem
  justify-content space-between
  margin-bottom 1rem

  .card_btn
    cursor pointer
    width calc(33% - 0.7rem)
    background: #9aa2df;
    border-radius: 5px;
    border 4px solid #3e3b73
    outline #f1f1f1 solid 5px
    color: #fff;
    display: flex;
    flex-direction column
    align-items center
    box-shadow: 1px 1px 2px 0 rgb(0 0 0 / 6%);
    transition: all .4s;

    .name
      font-weight 500
      margin .5rem 0 .4rem 0
    .desc
      font-size: .8rem;
      line-height: 1.1rem;
      // opacity: .8;
      margin-bottom: 0.5rem;

    &:hover
      // text-decoration: none;
      box-shadow: 0 10px 20px -10px rgba(0,0,0,.15);
      transform: translateY(-3px) scale(1.01);


  button
    margin 0 auto
    display block
    font-size 1rem
    background-color #11a8cd
    padding 0.4rem 1rem
    border-radius 4px
    transition background-color .1s ease
    box-sizing border-box
    outline none
    border none
    color #fff
    &:hover
      background-color #2e6987
</style>
