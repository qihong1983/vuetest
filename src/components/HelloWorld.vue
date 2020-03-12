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
      finished: false
    };
  },

  mounted() {
    this.initMap();
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
      // var that = this;
      // window.AMap.plugin("AMap.Autocomplete", function() {
      //   // 实例化Autocomplete
      //   var autoOptions = {
      //     // input 为绑定输入提示功能的input的DOM ID
      //     input: "searchInput1",
      //     output: "searchResults",
      //     outPutDirAuto: false
      //   };
      //   var autoComplete = new window.AMap.Autocomplete(autoOptions);
      //   // 无需再手动执行search方法，autoComplete会根据传入input对应的DOM动态触发search
      //   autoComplete.search(val, function(status, result) {
      //     // 搜索成功时，result即是对应的匹配数据
      //     console.log(result, "提示result");
      //     that.searchResult = result.tips;
      //   });
      // });
      // Toast(val);
    },
    changeSearch(val) {
      console.log(val);
    },
    initMap(val) {
      var that = this;
      window.AMapUI.loadUI(
        ["misc/PositionPicker", "misc/PoiPicker", "overlay/SimpleMarker"],
        function(PositionPicker, PoiPicker, MarkerList) {
          // var map = new window.AMap.Map("container", {
          //   zoom: 15,
          //   resizeEnable: true,
          //   scrollWheel: false
          //   // center: [116.40725, 39.904527]
          // });

          var map = new window.AMap.Map("container", {
            resizeEnable: true
          });

          // 39.9041790000,116.4073870000
          //         lng: 113.253185
          // lat: 22.528704
          console.log(PoiPicker, "PoiPicker");
          var poiPicker = new PoiPicker({
            input: "searchInput1",
            autocompleteOptions: {
              output: "searchResults",
              outPutDirAuto: false
            },
            placeSearchOptions: {
              map: map,
              pageSize: 10
            },
            searchResultsContainer: "searchResults"
          });

          // var poiPicker = new PoiPicker({
          //   placeSearchOptions: {
          //     map: map,
          //     pageSize: 10
          //   },
          //   searchResultsContainer: "searchResults"
          // });

          var markerList = new MarkerList({
            //关联的map对象
            map: map
          });
          console.log(markerList);

          poiPicker.on("poiPicked", function(poiResult) {
            console.log(poiResult, "poiResult");
            poiPicker.hideSearchResults();
            var source = poiResult.source,
              poi = poiResult.item;
            console.log(source, "source");
            console.log(poi, "poi");
            if (source !== "search") {
              //suggest来源的，同样调用搜索
              poiPicker.searchByKeyword(poi.name);
            } else {
              // poiPicker.searchByKeyword("北京市");
            }
          });
          poiPicker.onCityReady(function() {
            console.log(val, "valvalvalvalvalval");

            poiPicker.searchByKeyword("北京市");
            // if (!val) {
            //   poiPicker.searchByKeyword("北京市");
            //   console.log("北京");
            // } else {
            //   console.log(val.name);
            //   poiPicker.searchByKeyword(val);
            // }
          });

          console.log(PositionPicker, "PositionPicker");
          console.log(map, "map");

          var positionPicker = new PositionPicker({
            mode: "dragMap",
            map: map
          });

          console.log(positionPicker, "positionPicker");
          positionPicker.on("success", function(positionResult) {
            console.log(positionResult);
            console.log(that, "thatthatthatthat");

            // that.positionResult = positionResult.regeocode.pois;

            var pois = positionResult.regeocode.pois;

            var list = [];

            pois.map(v => {
              list.push({
                name: v.name,
                address: v.address,
                location: `${v.location.lng},${v.location.lat}`
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
          positionPicker.on("fail", function(positionResult) {
            console.log(positionResult, "positionResult");
          });
          positionPicker.setMode("dragMap");
          positionPicker.start(map.getBounds().getSouthWest());
          positionPicker.start();
          map.panBy(0, 1);

          map.addControl(
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
