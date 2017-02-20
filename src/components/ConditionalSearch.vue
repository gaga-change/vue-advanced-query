<template>
  <!-- 筛选 -->
  <div class="filter">
    <!--不限 选项-->
    <transition name="outerPage">
      <template v-if="isShowOuter">
        <div class="filter-more docheight">
          <div class="top-header border-bottom fixed-top above-mask-header">
            <div class="top-back2 filter-back3">
              <a @click="hideOuter()">返回</a>
            </div>
            <div class="top-bt3 " v-text="outerName"></div>
          </div>
          <div class="filter-prof2 margin-top-2rem above-mask">
            <div class="filter-search border-bottom" v-if="outerName=='角色职业' ">
              <input class="f30" type="text" placeholder="首字母搜索" v-model="outerQuery">
            </div>
            <div class="prof2-list f30">
              <transition-group tag="ul" @before-enter="beforeEnter"
                                :appear="true"
                                @enter="enter" @leave="leave">
                <li class="border-bottom"
                    v-for="(item,index) in outerListFilter"
                    :key="item.id"
                    :data-index="index"
                    style="transition: all .5s; overflow: hidden"
                    @click="outerCheck(item)"
                    :class="{'active':item.check}">
                  <!--<p v-text="item.name"></p>-->
                  <em v-text="item.name"></em>
                  <span><i><input type="checkbox"></i></span>
                  <!--<div style="clear: both"></div>-->
                </li>
              </transition-group>
            </div>
            <div class="prof2-check f36">
              <a class="fcheckmore" @click="confirm()">确定</a>
            </div>
          </div>
          <div class="maskdiv3" @click="hideOuter()"></div>
        </div>

      </template>
    </transition>
    <!-- 显示筛选列表 -->
    <transition name="innerPage">
      <div class="filter-list docheight" v-if="!isShowOuter">
        <div class="filter-head">
          <div class="top-header border-bottom fixed-top">
            <div class="top-back2 filter-back">
              <a @click="back">返回</a>
            </div>
            <div class="top-bt3">
              精准筛选
            </div>
            <div class="top-right4 f32">
            </div>
          </div>
        </div>
        <div class="filter-body padding-b4 fw margin-top-2rem">
          <div class="filter01 fw " style="text-align: left" :class="{'margin-bottom': item.name!='角色职业'}"
               v-for="item in list">
            <h3 class="filter-title border-bottom f32"><b class="b-normal" v-text="item.name"></b>
              <span
                id="openprof"
                v-if="item.outer"
                @click="showOuter(item)">
                <b class="b-normal">{{item | outerLinkName}}</b>
                <i :class="{'remove-background-image': !item.outer.outerArrow}"></i>
                </span>
            </h3>
            <div class="fw filter-ul border-bottom" v-if="item.inner.length != 0">
              <ul>
                <li :class="{'active':inner.check}" :key="inner.name" v-for="(inner,index) in item.inner"
                    v-if="item.show || index < 6">
                     <span class="f30" @click="innerCheck(inner, item)">
                      <!--{{inner.check}}-->
                     <input :type="item.type" name="">
                       <b style="font-weight: normal" v-text="inner.name"></b>
                     </span>
                </li>
              </ul>
              <!--
                当index 大于6的时候，自动隐藏多余的条数，同时显示下拉按钮
              -->
              <!-- 下拉显示更多 -->
              <div v-if="item.inner.length > 6" class="fw downmore " @click="showResidue(item)"
                   :class="{'rotate-180': item.show}"></div>
              <!-- 缩回显示更多 -->
              <!--<div class="fw upnmore"></div>-->
            </div>
          </div>
        </div>
        <div class="filter-foot fixed-bottom f36 border-top ">
          <a><em class="whitesearch"></em>共精选出 <span>{{commodityNum | digitCommodityNum}}</span> 件商品</a>
        </div>
      </div>
    </transition>
  </div>
</template>

<script>
  /**
   *  样式问题：
   *      1. 不限条数大于 撑出页面时。 背景异常 -------------------------------------------------- √
   *      2. position：fixed 问题。如果是谷歌自己的异常。那忽略。可以选择查看他人代码时候有相同问题。-- 忽略
   *      3. 进入第二层，位置问题。默认位置覆盖掉了一条数据。 -------------------------------------- √
   *      4. 进入页面时，高度不是100%。 而是110%（猜）。 ------------------------------------------ 忽略
   *  动态效果：
   *      1. 首字母查询，动画效果  ------------------------------------ √
   *      2. 第一层列表载入动画效果 ----------------------------------- 忽略
   *      3. 第二层列表载入\退出动画效果 ------------------------------ √
   *      4. 下拉上拉动态效果
   *      5。精选出的条数改变的动画效果 -------------------------------- 忽略
   *  命名问题：
   *      不要用a,b,v,vv  -------------------------------------------------------------------- √
   *  注释问题：
   *      注释改改，或不写。  ------------------------------------------------------------------ 忽略
   *  数据问题：
   *      第一层数据：商品特色为动态载入。如果没有不显示该项目。   --------------------------------- √
   *      第二层数据：
   *          角色职业为动态载入。（点入才去获取）   --------------------------------------------- √
   *          提供首字母搜索功能 --------------------------------------------------------------- √
   *      显示刷选出的条数
   *          实现模拟显示条数        ---------------------------------------------------------- √
   *          商品数量无限大的时候 处理 --------------------------------------------------------- √
   *  样式问题(附加)：
   *      使用scss的文件导入时，图片路径失效。
   *      把图片放到公共区，路由都改为找公共区。
   */

  /**
   *  附加页，退出时，显示附加页勾选的内容。如果，大于两条，两条之后的忽略，加一个字“等”
   *  处理方法：
   *   方法一： 过滤器
   *      传入：当前的item
   *      输出：该怎么显示就怎么显示
   *
   *   箭头处理：
   *    方法一： 判断内容是否为“不限”两字
   *
   */


  import Vue from 'vue'
  var jsonData = {
    list: [
      {
        name: "角色职业",
        type: "radio",
        inner: [],
        outer: {
          name: "不限",
          list: []
        }
      },
      {
        name: "角色性别",
        type: "radio",
        inner: [
          {name: "不限性别"},
          {name: "男性角色"},
          {name: "女性角色"}
        ]
      },
      {
        name: "价格区间",
        type: "radio",
        inner: [
          {name: "不限"},
          {name: "50元以下"},
          {name: "50-100元"},
          {name: "100-500元"},
          {name: "500-1000元"},
          {name: "1000元以上"},
        ]
      },
      {
        name: "身份证绑定",
        type: "radio",
        inner: [
          {name: "不限"},
          {name: "未绑定"},
          {name: "已绑定"},
        ]
      },
      {
        name: "安心买",
        type: "radio",
        inner: [
          {name: "不限"},
          {name: "未开通"},
          {name: "已开通"},
        ]
      },
      {
        name: "QQ等级",
        type: "radio",
        inner: [
          {name: "不限"},
          {name: "0级"},
          {name: "1-10级"},
          {name: "12-20级"},
          {name: "21-40级"},
          {name: "40级以上"},
        ]
      },
      {
        name: "QQ好友",
        type: "radio",
        inner: [
          {name: "不限"},
          {name: "无友好"},
          {name: "有好友"},
        ]
      },
      {
        name: "更多服务",
        type: "radio",
        inner: [],
        outer: {
          name: "不限",
          outerArrow: "false",
          list: [
            {name: "免单购物", id: 1},
            {name: "无货赔付", id: 2},
            {name: "资料修改", id: 3},
            {name: "商品已投保", id: 4},
            {name: "找回包赔", id: 5},
            {name: "截图认证", id: 6},
            {name: "视频认证", id: 7},
            {name: "寄售交易", id: 8},
            {name: "担保交易", id: 9}
          ]
        }
      }
    ]
  };
  export default {
    data: function () {
      return {
        list: [],  //主数据
        isShowOuter: false,  //是否显示 “不限” 页面
        outerList: [],  // “不限” 页面里的数据
        outerQuery: "", //首字母筛选条件
        outerLink: {},
        outerName: "",
        commodityNum: 0,  //筛选出的商品条数
      }
    },
    /**
     * 监听被选中的数据
     */
    watch: {
      checked: {
        handler: function (val, oldVal) {
          console.log("-------被选中的条件---------", val);
          this.getCommodityNum(val);
        },
        deep: true
      }
    },
    computed: {
      /**
       * 过滤list数据，过滤出所有被选中的参数
       * @returns {{}}
       */
      checked: function () {
        var ret = {};
        /*循环父*/
        this.list.forEach(function (val) {
          ret[val.name] = [];
          /*如果有不限，循环不限*/
          if (val.outer) {
            val.outer.list.forEach(function (outerItem) {
              if (outerItem.check) {
                ret[val.name].push(outerItem.name);
              }
            })
          }
          /*循环子*/
          val.inner.forEach(function (innerItem) {
            if (innerItem.check) {
              ret[val.name].push(innerItem.name);
            }
          })
        });
        return ret;
      },
      /**
       * 过滤后的外部数据(过滤后的角色职业列表)
       */
      outerListFilter: function () {
        var self = this;
        return this.outerList.filter(function (item, index) {
          /**
           * 符合筛选条件
           *    已勾选  --- 不处理
           *    未勾选  --- 不处理
           * 不符合筛选条件
           *    已勾选  --- 处理
           *    未勾选  --- 不处理
           */
          if (item.name.toLocaleLowerCase().indexOf(self.outerQuery.toLocaleLowerCase()) == -1 && item.check) {
            console.log(item.name);
            item.check = false;
          }
          return item.name.toLocaleLowerCase().indexOf(self.outerQuery.toLocaleLowerCase()) !== -1;
        })
      }
    },
    /**
     * 组件加载后，获取主数据，赋值给list
     *    给每一项子数据，添加check属性，用于标记是否被选中
     * */
    mounted: function () {
      this.getCommodityFeature(function (commodityFeature) {
        var newJsonData = JSON.parse(JSON.stringify(jsonData));
        if (commodityFeature) {
          newJsonData.list.unshift(commodityFeature)
        }
        newJsonData.list.forEach(function (item) {
          item.show = false; //默认当直接子数据大于6条时，自动隐藏多余数据
          /*如果有“不限”， 给每个“不限”子数据添加check属性*/
          if (item.outer) {
            item.outer.list.forEach(function (outerItem, outerIndex, outer) {
              outer[outerIndex].check = false;
            })
          }
          item.inner.forEach(function (innerItem, innerIndex) {
            if (innerItem.name == "不限性别" || innerItem.name == "不限") {
              item.inner[innerIndex].check = true;
//                        console.log(a)
            } else {
              item.inner[innerIndex].check = false;

            }
          });
          self.list.push(item);
        });
      });
      var self = this;

//            console.log(self.list);

    },
    methods: {
      /**
       * 是否显示6条自后的数据，默认为false(自动隐藏多余6条自后的数据)。
       * */
      showResidue: function (item) {
        item.show = !item.show;
      },
      /**
       *  控制直接子数据的点击之后的选中状态
       * */
      innerCheck: function (innerItem, item) {
        /**
         *  1. false 点击之后
         *    1. 本身： check=true
         *    2. 同类：
         *        1. 单选： 同类所有 check=false
         *        2. 多选： 不变
         *  2. true 点击之后
         *    单选 不变
         *    多选 check = false
         *
         */
        if(innerItem.check && item.type != 'checkbox') return;
        if (innerItem.check && item.type == 'checkbox') {
          innerItem.check = false;
        } else {
          if (item.type != 'checkbox') {
            item.inner.forEach(function (v) {
              v.check = false;
            })
          }
          innerItem.check = true;
        }
      },
      /**
       *  控制是否显示额外“不限”数据，默认为false
       *    点击“不限”之后，改为true（显示）。同时改变“不限”里面的数据
       */
      showOuter: function (item) {
        /**
         *  如果是角色职业的不限，自动获取数据
         */
        var self = this;
        self.isShowOuter = true;
        self.outerName = item.name;
        if (item.name == "角色职业" && item.outer.list.length == 0) {
          Vue.set(item.outer, "list", []);
          this.getProfession(function (data) {
            var profession = [];
            if (data) {
              profession = data;
            } else {
              profession = [];
            }
            Vue.set(item.outer, "list", profession);
            self.outerList = item.outer.list.map(function (v) {
              return JSON.parse(JSON.stringify(v))
            });
            self.outerLink = item.outer.list;
          })
        } else {
          self.outerList = item.outer.list.map(function (v) {
            return JSON.parse(JSON.stringify(v))
          });
          self.outerLink = item.outer.list;
        }
        self.goTop();
      },
      /**
       * 隐藏“不限”页面
       */
      hideOuter: function () {
        this.isShowOuter = false;
        this.outerQuery = "";
        this.goTop();
      },
      /**
       * 点击“不限”页面的选项时。控制选中状态。默认“不限”页面里的数据是 多选
       */
      outerCheck: function (outerItem) {
        outerItem.check = !outerItem.check;
      },
      /**
       * “不限"页面，确认
       */
      confirm: function () {
        var self = this;
//                console.log(self.outerList);
        self.outerList.forEach(function (outerItem, outerIndex) {
          self.outerLink[outerIndex].check = outerItem.check;
        });
        this.hideOuter();
      },
      /** 获取商品特色
       */
      getCommodityFeature: function (callBack) {
        var self = this;
//                if(jsonData > 0) return;
        setTimeout(function () {
          var commodityFeature = {
            name: "商品特色",
            type: "checkbox",
            inner: [
              {name: "角色分离"},
              {name: "找回包赔"},
              {name: "未选阵营"},
              {name: "游戏币"},
              {name: "游戏币"},
              {name: "游戏币"},
              {name: "游戏币"},
              {name: "游戏币"}
            ]
          }
//                    commodityFeature = null;
          if (commodityFeature) {
//                        console.log("---------", self.list);
            callBack(commodityFeature);
          } else {
            callBack(null);
          }
        }, 100)
      },
      /**
       * 获取角色职业
       */
      getProfession: function (callBack) {
        var out = [
          {name: "XXX", check: false, id: 1},
          {name: "YY", check: false, id: 2},
          {name: "abc", check: false, id: 3},
          {name: "ddd", check: false, id: 4},
          {name: "gacda", check: false, id: 5},
          {name: "fdsafdsa", check: false, id: 6},
          {name: "fdsafdsa", check: false, id: 7},
          {name: "cd", check: false, id: 8},
          {name: "Xfds", check: false, id: 9},
          {name: "fsd", check: false, id: 10},
          {name: "fds", check: false, id: 11},
          {name: "fds", check: false, id: 12},
          {name: "fds", check: false, id: 13},
          {name: "fds", check: false, id: 14},
          {name: "Xfds", check: false, id: 15}
        ];
//                out = null;
        setTimeout(function () {
          callBack(out)
        }, 100)
      },
      /**
       * 获取当前筛选条件 选出的商品条数
       */
      getCommodityNum: function (filter) {
        var self = this;
        setTimeout(function () {
          self.commodityNum = Math.floor(Math.random() * 10000000000);
        }, 100)
      },
      /**
       * 返回路由上一层
       * */
      back: function () {
        this.$router.go(-1);
      },
      /**
       * 回到页面顶部
       */
      goTop: function () {
        var tep = 0;
        var heighs = document.documentElement.scrollTop || document.body.scrollTop;
        tep = heighs / 100;
        document.documentElement.scrollTop = 0;
        document.body.scrollTop = 0;
//        var inteval = setInterval(function () {
//          var heighs = document.documentElement.scrollTop || document.body.scrollTop;
//          if (heighs < 3) {
//            clearInterval(inteval);
//          }
//          if (document.documentElement.scrollTop) {
//
//            document.documentElement.scrollTop = document.documentElement.scrollTop - tep
//          } else {
//            document.body.scrollTop = document.body.scrollTop - tep;
//          }
//        }, 2);
      },
      beforeEnter: function (el) {
        el.style.opacity = 0;
        el.style.height = 0;
      },
      enter: function (el, done) {
        var delay = el.getAttribute('data-index') * 50;
        setTimeout(function () {
          el.style.opacity = 1;
          el.style.height = '2.125rem';
        }, delay);
      },
      leave: function (el, done) {
        var delay = el.getAttribute('data-index') * 50;
        setTimeout(function () {
          el.style.opacity = 0;
          el.style.height = 0;
        }, delay)
      }
    },
    filters: {
      digitCommodityNum: function (val) {
        /**
         * 大于 4位，如 40000，10000,99999
         *      除以 10000，然后四舍五入 ， 加 万
         * 大于 8位 如 1,0000,0000  除以1*10^8， 四舍五入 加亿
         */
        console.log("数据", val)
        if (String(val).length < 5) {
          return val;
        }
        if (String(val).length > 4 && String(val).length < 8) {
          return "约" + Math.round(val / 10000) + "万";
        }
        if (String(val).length > 8) {
          return "约" + Math.round(val / (10000 * 10000)) + "亿";
        }
      },
      outerLinkName: function (item) {
        /**
         *  数据量
         *    等于0 ，return item.outer.name
         *    大于0
         *      循环列表，判断 check是否选中。
         *      配置变量 name = []
         *      - 开始循环每一个数据 => val
         *      - 如果name的长度大于3，   结束
         *      - 如果选中，把当前数据名加到name列表中
         *      - 结束
         *    如果name长度为0 return item.outer.name
         *    如果name长度为1,或2  return name.join(",")
         *    如果name长度为3 return name.pop().join(",")
         *
         */
        if (!item.outer) return "";
        if (item.outer.list && item.outer.list == 0) {
          item.outer.outerArrow = true;
          return item.outer.name;
        }
        var name = [];
        item.outer.list.forEach(function (val, index) {
          if (name.length > 2) return;
          if (val.check) {
            name.push(val.name);
          }
        })
        if (name.length == 0) {
          item.outer.outerArrow = true;
          return item.outer.name;
        }
        item.outer.outerArrow = false;
        if (name.length < 3) {
          return name.join(", ")
        }
        name.pop();
        return name.join(", ") + "等";
      }
    }
  }
</script>

<style scoped>
  @import "../css/public.css";
  @import "../css/list.css";

  .remove-background-image {
    width: 0 !important;
    background-image: none !important;
  }

  .filter {
    display: block;
  }

  .rotate-180 {
    transform: rotate(180deg);
  }

  .b-normal {
    font-weight: normal;
  }

  .margin-top-2rem {
    margin-top: 2rem;
  }

  .filter-more {

    display: block !important;
  }

  .filter-more .above-mask {
    /*z-index: 52 !important;*/
    position: relative;
  }

  .maskdiv3 {
    width: 10%;
  }

  .filter-more .above-mask-header {
    z-index: 102 !important;
  }

  .filter-more .prof2-list {
    background-color: white;
  }

  /* outerPage 动画效果*/

  .outerPage-enter-active {
    animation: fadeInDown .5s;
  }

  .outerPage-leave-active {
    animation: fadeOutRight .5s;
  }

  .innerPage-enter-active {
    animation: fadeIn .5s;
  }

  .innerPage-leave-active {
    animation: fadeOut .5s;
  }

  .maxSex-enter-active {
    animation: fadeIn .5s;
  }

  /* animate.css含有的 部分动画 */

  @-webkit-keyframes fadeInRight {
    from {
      opacity: 0;
      -webkit-transform: translate3d(100%, 0, 0);
      transform: translate3d(100%, 0, 0);
    }

    to {
      opacity: 1;
      -webkit-transform: none;
      transform: none;
    }
  }

  @keyframes fadeInRight {
    from {
      opacity: 0;
      -webkit-transform: translate3d(100%, 0, 0);
      transform: translate3d(100%, 0, 0);
    }

    to {
      opacity: 1;
      -webkit-transform: none;
      transform: none;
    }
  }

  .fadeInRight {
    -webkit-animation-name: fadeInRight;
    animation-name: fadeInRight;
  }

  @-webkit-keyframes fadeOutRight {
    from {
      opacity: 1;
    }

    to {
      opacity: 0;
      -webkit-transform: translate3d(100%, 0, 0);
      transform: translate3d(100%, 0, 0);
    }
  }

  @keyframes fadeOutRight {
    from {
      opacity: 1;
    }

    to {
      opacity: 0;
      -webkit-transform: translate3d(100%, 0, 0);
      transform: translate3d(100%, 0, 0);
    }
  }

  .fadeOutRight {
    -webkit-animation-name: fadeOutRight;
    animation-name: fadeOutRight;
  }

  @-webkit-keyframes fadeIn {
    from {
      opacity: 0;
    }

    to {
      opacity: 1;
    }
  }

  @keyframes fadeIn {
    from {
      opacity: 0;
    }

    to {
      opacity: 1;
    }
  }

  .fadeIn {
    -webkit-animation-name: fadeIn;
    animation-name: fadeIn;
  }

  @-webkit-keyframes fadeOut {
    from {
      opacity: 1;
    }

    to {
      opacity: 0;
    }
  }

  @keyframes fadeOut {
    from {
      opacity: 1;
    }

    to {
      opacity: 0;
    }
  }

  .fadeOut {
    -webkit-animation-name: fadeOut;
    animation-name: fadeOut;
  }

  @-webkit-keyframes fadeInDown {
    from {
      opacity: 0;
      -webkit-transform: translate3d(0, -100%, 0);
      transform: translate3d(0, -100%, 0);
    }

    to {
      opacity: 1;
      -webkit-transform: none;
      transform: none;
    }
  }

  @keyframes fadeInDown {
    from {
      opacity: 0;
      -webkit-transform: translate3d(0, -100%, 0);
      transform: translate3d(0, -100%, 0);
    }

    to {
      opacity: 1;
      -webkit-transform: none;
      transform: none;
    }
  }

  .fadeInDown {
    -webkit-animation-name: fadeInDown;
    animation-name: fadeInDown;
  }

</style>
