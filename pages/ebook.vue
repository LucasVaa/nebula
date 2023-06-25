<template>
  <div class="ebook">
    <el-row>
      <el-col :span="24"
        ><div class="title">
          {{ bookMeta.title }} &nbsp-&nbsp {{ bookMeta.creator }}
        </div></el-col
      >
    </el-row>
    <el-row class="read-wrapper">
      <el-col :span="2"><div class="left" @click="prevPage"></div></el-col>
      <el-col :span="20"><div id="read" class="read"></div></el-col>
      <el-col :span="2"><div class="right" @click="nextPage"></div></el-col>
    </el-row>
    <!-- <div class="read-wrapper">
      <div class="mask">
        <div class="center" @click="toggleTitleAndMenu"></div>
      </div>
      <div id="extras">
        <ul id="highlights"></ul>
      </div>
    </div>
    <menu-bar
      :ifTitleAndMenuShow="ifTitleAndMenuShow"
      :fontSizeList="fontSizeList"
      :defaultFontSize="defaultFontSize"
      @setFontSize="setFontSize"
      :themeList="themeList"
      :defaultTheme="defaultTheme"
      @setTheme="setTheme"
      :bookAvailable="bookAvailable"
      @onProgressChange="onProgressChange"
      :navigation="navigation"
      @jumpTo="jumpTo"
      ref="menuBar"
    ></menu-bar> -->
  </div>
</template>

<script>
import MenuBar from "../components/ebook/MenuBar.vue";
import Epub from "epubjs";
// const DOWNLOAD_URL =
//   "http://110.42.197.57:8080/epub/穿越寒冬：创业者的融资策略与独角兽思维.epub";
export default {
  layout: "read",
  name: "Ebook",
  components: {
    MenuBar,
  },
  data() {
    return {
      ifTitleAndMenuShow: false,
      fontSizeList: [
        { fontSize: 12 },
        { fontSize: 14 },
        { fontSize: 16 },
        { fontSize: 18 },
        { fontSize: 20 },
        { fontSize: 22 },
        { fontSize: 24 },
      ],
      defaultFontSize: 16,
      themeList: [
        {
          name: "default",
          style: {
            body: {
              color: "#040404",
              background: "#F6F7F8",
            },
          },
        },
        {
          name: "eye",
          style: {
            body: {
              color: "#000",
              background: "#ceeaba",
            },
          },
        },
        {
          name: "night",
          style: {
            body: {
              color: "#fff",
              background: "#000",
            },
          },
        },
        {
          name: "gold",
          style: {
            body: {
              color: "#000",
              background: "#F1DFCF",
            },
          },
        },
      ],
      defaultTheme: 0,
      // 图书是否是可用状态
      bookAvailable: false,
      navigation: {},
      bookMeta: {
        title: "",
        creator: "",
      },
    };
  },
  methods: {
    // 目录链接跳转
    jumpTo(href) {
      this.rendition.display(href);
      this.hideTitleAndMenuShow();
    },
    hideTitleAndMenuShow() {
      // 隐藏标题栏和菜单栏
      this.ifTitleAndMenuShow = false;
      // 隐藏菜单弹出的设置栏
      this.$refs.menuBar.hideSetting();
      // 隐藏目录
      this.$refs.menuBar.hideContent();
    },
    // 进度条的数值 (0-100)
    onProgressChange(progress) {
      const percentage = progress / 100;
      const location =
        percentage > 0 ? this.locations.cfiFromPercentage(percentage) : 0;
      this.rendition.display(location);
    },
    setTheme(index) {
      this.themes.select(this.themeList[index].name);
      this.defaultTheme = index;
    },
    registerTheme() {
      this.themeList.forEach((theme) => {
        this.themes.register(theme.name, theme.style);
      });
    },
    setFontSize(fontSize) {
      this.defaultFontSize = fontSize;
      if (this.themes) {
        this.themes.fontSize(fontSize + "px");
      }
    },
    toggleTitleAndMenu() {
      // this.ifTitleAndMenuShow = !this.ifTitleAndMenuShow;
      // if (!this.ifTitleAndMenuShow) {
      //   this.$refs.menuBar.hideSetting();
      // }
    },
    prevPage() {
      // this.ifTitleAndMenuShow = true;
      // this.$refs.menuBar.showSetting(2);
      if (this.rendition) {
        this.rendition.prev().then(() => {
          // 点击上一页,控制进度条变化
          if (this.locations) {
            const currentLocation = this.rendition.currentLocation();
            let progress = Math.ceil(
              this.locations.percentageFromCfi(currentLocation.start.cfi) * 100
            );
            this.$refs.menuBar.onProgressInput(progress);
          }
        });
      }
    },
    nextPage() {
      if (this.rendition) {
        // this.ifTitleAndMenuShow = true;
        // this.$refs.menuBar.showSetting(2);
        this.rendition.next().then(() => {
          console.log(this.rendition.location);
          // 点击上一页,控制进度条变化
          if (this.locations) {
            const currentLocation = this.rendition.currentLocation();
            let progress = Math.ceil(
              this.locations.percentageFromCfi(currentLocation.start.cfi) * 100
            );
            this.$refs.menuBar.onProgressInput(progress);
          }
        });
      }
    },
    // 电子书的解析和渲染
    showEpub() {
      let self = this;
      // 生成 Ebook
      this.book = new Epub(this.$route.query.bookUrl);
      // 生成 Rendtion
      this.rendition = this.book.renderTo("read", {
        width: window.innerWidth * 0.8,
        height: window.innerHeight - 60,
      });
      // 通过 Rendtion.display 渲染电子书
      this.rendition.display();
      // 获取 Theme 对象
      this.themes = this.rendition.themes;
      // 获取元数据
      this.book.loaded.metadata.then(function (result) {
        self.bookMeta.title = result.title;
        self.bookMeta.creator = result.creator;
      });
      // 设置默认字体
      this.setFontSize(this.defaultFontSize);
      // this.themes.register(name, styles)
      // this.themes.select(name)
      this.registerTheme();
      this.setTheme(this.defaultTheme);
      // 获取 Locations 对象
      // 通过 epubjs 的钩子函数来实现
      this.book.ready
        .then((chars) => {
          return this.book.locations.generate(chars);
        })
        .then(() => {
          this.navigation = this.book.navigation;
          this.locations = this.book.locations;
          this.bookAvailable = true;
        });
      // console.log(this.book.locations);
      // 获取上一次阅读位置
      this.rendition.display("epubcfi(/6/10!/4/2/2/1:0)");
      // 测试高亮
      // Illustration of how to get text from a saved cfiRange
      self.rendition.on("selected", function (cfiRange, contents) {
        self.rendition.annotations.highlight(cfiRange, {}, (e) => {
          console.log("highlight clicked", e.target);
        });
        contents.window.getSelection().removeAllRanges();
      });

      this.rendition.themes.default({
        "::selection": {
          background: "rgba(255,255,0, 0.3)",
        },
        ".epubjs-hl": {
          fill: "yellow",
          "fill-opacity": "0.3",
          "mix-blend-mode": "multiply",
        },
      });
      // 测试适用高亮
      self.rendition.annotations.highlight(
        "epubcfi(/6/10!/4/4,/1:18,/1:31)",
        {},
        (e) => {
          console.log("highlight clicked", e.target);
        }
      );
      // Illustration of how to get text from a saved cfiRange
      var highlights = document.getElementById("highlights");

      self.rendition.on("selected", function (cfiRange) {
        self.book.getRange(cfiRange).then(function (range) {
          var text;
          var li = document.createElement("li");
          var a = document.createElement("a");
          var remove = document.createElement("a");
          var textNode;

          if (range) {
            text = range.toString();
            textNode = document.createTextNode(text);

            a.textContent = cfiRange;
            a.href = "#" + cfiRange;
            a.onclick = function () {
              rendition.display(cfiRange);
            };

            remove.textContent = "remove";
            remove.href = "#" + cfiRange;
            remove.onclick = function () {
              rendition.annotations.remove(cfiRange);
              return false;
            };
            li.appendChild(a);
            li.appendChild(textNode);
            li.appendChild(remove);
            highlights.appendChild(li);
          }
        });
      });
    },
  },
  mounted() {
    this.showEpub();
  },
};
</script>

<style>
.title {
  color: #a6a6a6;
  display: flex;
  justify-content: center;
  align-items: center;
  min-height: 36px;
}
.title:hover {
  color: #535353;
}
.ebook {
  position: relative;
}
.read-wrapper {
  /* position: absolute; */
  /* top: 5%; */
  left: 0;
  display: flex;
  width: 100%;
  height: 100%;
}
.read-wrapper .read {
}
.mask {
  position: absolute;
  top: 5%;
  left: 0;
  z-index: 100;
  display: flex;
  width: 100%;
  height: 100%;
}
.read-wrapper .left {
  height: 100%;
  width: 50%;
  float: left;
  background: no-repeat center/100% url("~/assets/arrow_left.png");
}
.read-wrapper .left:hover {
  background-image: url("~/assets/arrow_left_hover.png");
}
.read-wrapper .center {
  height: 100%;
  flex: 1;
}
.read-wrapper .right {
  height: 100%;
  width: 50%;
  float: right;
  background: no-repeat center/100% url("~/assets/arrow_right.png");
}
.read-wrapper .right:hover {
  background-image: url("~/assets/arrow_right_hover.png");
}

.el-row {
  /* margin-bottom: 20px; */
  &:last-child {
    margin-bottom: 0;
  }
}
.el-col {
  border-radius: 4px;
}
.grid-content {
  border-radius: 4px;
  min-height: 36px;
}
</style>

<!-- <style scoped lang="sass">
@import "assets/styles/global"
</style> -->
