<template>
  <div class="hello">
    <!-- 导航 begin -->
    <van-nav-bar left-text="取消" right-text="确定" />
    <!-- 导航 end -->

    <!-- 地图搜索 begin -->
    <van-search
      v-model="searchValue"
      id="searchInput1"
      show-action
      placeholder="请输入搜索地址"
      @input="onSearch"
    />
    <!-- 地图搜索 end -->

    <!-- 搜索出来的内容 begin -->
    <!-- <van-list class="searchInfo">
      <van-cell
        class="searchInfo-cell"
        :title="item.name"
        :label="item.district"
        is-link
        v-for="(item, index) in searchResult"
        :key="item+index"
        @click="clickSearchResult(item)"
      >
        <van-icon slot="right-icon" style="line-height: inherit; " color="#67C23A" />
      </van-cell>
    </van-list>-->
    <!-- 搜索出来的内容 end -->

    <div id="container" class="map" tabindex="0"></div>

    <!-- 结果列表 begin -->

    <!-- <van-list class="searchResults">
      <van-cell
        :title="`${item.name}`"
        :label="`${item.address}`"
        is-link
        v-for="(item, index) in positionResult"
        :key="item+index"
      >
        <van-icon slot="right-icon" style="line-height: inherit; " color="#67C23A" />
      </van-cell>
    </van-list>-->
    <!-- 结果列表 end -->

    <div id="searchResults"></div>
  </div>
</template>

<script>
import { Button, NavBar, Search, Toast, List, Cell, Icon } from "vant";
export default {
  name: "HelloWorld",
  data() {
    return {
      //搜索的
      searchResult: [],
      //选中的
      positionResult: [],
      //搜索的key
      searchValue: "",
      locat: "",
      locats: "",
      longitudes: 0, //经度
      latitudes: 0, //维度
      addressBox: [],
      //搜索出来的列表
      list: [],
      loading: false,
      finished: false,
      map: null,
      positionPicker: null
    };
  },

  mounted() {
    // lng: 116.413628
    // lat: 39.905581
    // this.map = new window.AMap.Map("container", {
    //   resizeEnable: true,
    //   zoom: 11,
    //   center: [116.407387, 39.904179]
    // });
    // center: [116.407387, 39.904179]
    // var that = this;
    // var gps = [116.407387, 39.904179];
    // window.AMap.convertFrom(gps, "gps", function(status, result) {
    //   if (result.info === "ok") {
    //     var lnglats = result.locations; // Array.<LngLat>
    //     console.log(lnglats, "lnglats");
    //   }
    // });
    // this.initMap();

    this.init();
  },
  components: {
    [Button.name]: Button,
    [NavBar.name]: NavBar,
    [Search.name]: Search,
    [Toast.name]: Toast,
    [List.name]: List,
    [Cell.name]: Cell,
    [Icon.name]: Icon
  },
  methods: {
    async init() {
      //初始化
      this.map = await new window.AMap.Map("container", {
        resizeEnable: true,
        zoom: 11,
        center: new window.AMap.LngLat(116.296642, 40.050837)
      });

      //       lng: 116.296642
      // lat: 40.050837

      await this.initMap();
    },
    /**
     * 点击搜索
     */
    clickSearchResult(val) {
      console.log(val, "3333333");

      console.log(this, "thisthis");

      this.searchResult = [];

      this.searchValue = "";

      console.log(val, "4444");

      this.initMap(val);
    },
    /**
     * 列表
     */

    onLoad() {
      // 异步更新数据
      // setTimeout 仅做示例，真实场景中一般为 ajax 请求
      setTimeout(() => {
        for (let i = 0; i < 10; i++) {
          this.list.push(this.list.length + 1);
        }

        console.log(this.list, "this.list");
        // 加载状态结束
        this.loading = false;

        // 数据全部加载完成
        if (this.list.length >= 40) {
          this.finished = true;
        }
      }, 1000);
    },

    onSearch(val) {
      console.log(val);
      var that = this;
      window.AMap.plugin("AMap.Autocomplete", function() {
        // 实例化Autocomplete
        var autoOptions = {
          // input 为绑定输入提示功能的input的DOM ID
          input: "searchInput1",
          city: val
        };
        var autoComplete = new window.AMap.Autocomplete(autoOptions);
        // 无需再手动执行search方法，autoComplete会根据传入input对应的DOM动态触发search
        autoComplete.search(val, function(status, result) {
          // 搜索成功时，result即是对应的匹配数据
          console.log(result, "提示result");
          that.searchResult = result.tips;
        });

        autoComplete.on("select", function(obj) {
          console.log(obj, "####");

          // that.map.panTo([116.405467, 39.907761]);
          console.log(that.map.getBounds().getSouthWest());
          console.log(that.map.getBounds().getSouthWest().lng);
          console.log(that.map.getBounds().getSouthWest().lat);

          that.positionPicker.start(obj.poi.location);
          // that.map.panBy(obj.poi.location.lng, obj.poi.location.lat);
        });
      });

      // autoComplete.select(id, name, adcode, district, location, type);

      // Toast(val);
    },
    changeSearch(val) {
      console.log(val);
    },
    initMap(val) {
      console.log(val);
      var that = this;

      // window.AMap.service(["AMap.PlaceSearch"], function() {
      //   //构造地点查询类
      //   var placeSearch = new window.AMap.PlaceSearch({
      //     pageSize: 10, // 单页显示结果条数
      //     pageIndex: 1, // 页码
      //     // city: "010", // 兴趣点城市
      //     citylimit: false, //是否强制限制在设置的城市内搜索
      //     map: that.map, // 展现结果的地图实例
      //     // panel: "searchResults", // 结果列表将在此容器中进行展示。
      //     autoFitView: false // 是否自动调整地图视野使绘制的 Marker点都处于视口的可见范围
      //   });
      //   //关键字查询
      //   placeSearch.search("北京市");

      //   placeSearch.on("complete", SearchResult => {
      //     console.log(SearchResult, "SearchResult");
      //   });
      // });

      window.AMapUI.loadUI(
        ["misc/PositionPicker", "misc/PoiPicker", "overlay/SimpleMarker"],
        function(PositionPicker, PoiPicker, MarkerList) {
          console.log(PoiPicker);
          console.log(MarkerList);

          console.log(PositionPicker, "PositionPicker");
          console.log(that.map, "map");

          that.positionPicker = new PositionPicker({
            mode: "dragMap",
            map: that.map
          });

          /**
           * 这个是可用的 begin
           */
          // window.AMap.service(["AMap.PlaceSearch"], function() {
          //   //构造地点查询类
          //   var placeSearch = new window.AMap.PlaceSearch({
          //     pageSize: 10, // 单页显示结果条数
          //     pageIndex: 1, // 页码
          //     // city: "010", // 兴趣点城市
          //     citylimit: false, //是否强制限制在设置的城市内搜索
          //     map: that.map, // 展现结果的地图实例
          //     // panel: "searchResults", // 结果列表将在此容器中进行展示。
          //     autoFitView: false // 是否自动调整地图视野使绘制的 Marker点都处于视口的可见范围
          //   });
          //   //关键字查询
          //   placeSearch.search("北京市");

          //   placeSearch.on("complete", SearchResult => {
          //     console.log(SearchResult, "SearchResult");
          //     // that.positionPicker.start(that.map.getBounds().getSouthWest());

          //     that.positionPicker.start(SearchResult.poiList.pois[0].location);
          //   });
          // });
          // 这个是可用的 end

          console.log(that.positionPicker, "positionPicker");
          that.positionPicker.on("success", function(positionResult) {
            console.log(positionResult);
            console.log(that, "thatthatthatthat");

            // that.positionResult = positionResult.regeocode.pois;

            var pois = positionResult.regeocode.pois;

            var list = [];

            pois.map(v => {
              list.push({
                name: v.name,
                address: v.address,
                location: v.location
              });
            });

            // list.unshift({
            //   name: "xxx",
            //   address: "yyyy",
            //   location: `zzzz`
            // });

            console.log(list, "list");

            that.positionResult = list;

            //
          });
          that.positionPicker.on("fail", function(positionResult) {
            console.log(positionResult, "positionResult");
          });
          that.positionPicker.setMode("dragMap");

          console.log(that.map.getBounds().getSouthWest(), "*****");

          // var that = this;
          // var gps = [116.407387, 39.904179];
          // window.AMap.convertFrom(gps, "gps", function(status, result) {
          //   if (result.info === "ok") {
          //     var lnglats = result.locations; // Array.<LngLat>

          //     console.log(lnglats, "lnglats");
          //     that.positionPicker.start(that.map.getBounds().getSouthWest());
          //   }
          // });

          // console.log(
          //   new window.AMap.LngLat(116.407387, 39.904179),
          //   "jingweidu"
          // );

          // that.positionPicker.start(that.map.getBounds().getSouthWest());
          // that.positionPicker.start(
          //   new window.AMap.LngLat(116.407387, 39.904179)
          // );
          that.positionPicker.start();
          that.map.panBy(0, 1);
          // that.map.panBy(116.407387, 39.904179);

          // 116.407387,39.904179

          that.map.addControl(
            new window.AMap.ToolBar({
              liteStyle: true
            })
          );
        }
      );
    }
  },
  props: {
    msg: String
  }
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped lang="less">
h3 {
  margin: 40px 0 0;
}
ul {
  list-style-type: none;
  padding: 0;
}
li {
  display: inline-block;
  margin: 0 10px;
}
a {
  color: #42b983;
}

.map {
  height: 300px;
  width: 100vw;
  float: left;
  // background: red;
}

.customButton {
  background: #e56045;
}

#searchResults {
  // display: none !important;
  position: absolute;
  z-index: 999999999999;
  display: none;
}

#searchResults .auto-item {
  background: red !important;
}

.auto-item {
  background: red !important;
}

.searchInfo {
  position: absolute;
  width: 100vw;
  z-index: 100000;
  display: none;
}

// .searchInfo-cell {
//   background: #e5e5e5;
// }

.searchResults {
  width: 100vw;
  height: 300px;
}

.amap-ui-poi-picker-sugg-container {
  // display: none !important;
  background: red;
}

.amap-ui-poi-picker-search-results-container,
.amap-ui-poi-picker-sugg-container {
  background: red;
}
</style>
