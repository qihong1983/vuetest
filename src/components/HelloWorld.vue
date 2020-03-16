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
    <van-list class="searchInfo">
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
    </van-list>
    <!-- 搜索出来的内容 end -->

    <div id="container" class="map" tabindex="0"></div>

    <!-- 结果列表 begin -->

    <van-list class="searchResults">
      <van-cell
        :title="index == 0 ? '\b[位置]\b ' + item.name:item.name "
        :label="`${item.address}`"
        is-link
        v-for="(item, index) in positionResult"
        :key="item+index"
        @click="clickOkResult(item)"
      >
        <van-icon
          slot="right-icon"
          :name="item.selected? 'success' : ''"
          style="line-height: inherit; "
          color="#67C23A"
        />
      </van-cell>
    </van-list>
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
      positionPicker: null,
      clickSelected: true //判断就下拉点选，还是底部详情列表点选
    };
  },

  mounted() {
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
        zoom: 16,
        center: new window.AMap.LngLat(116.296642, 40.050837)
      });

      this.map.on("dragging", async () => {
        this.clickSelected = true;
      });
      //       lng: 116.296642
      // lat: 40.050837
      var that = this;

      await this.initMap();

      await this.map.plugin("AMap.Geolocation", async function() {
        var geolocation = new window.AMap.Geolocation({
          enableHighAccuracy: true, //是否使用高精度定位，默认:true
          timeout: 10000, //超过10秒后停止定位，默认：无穷大
          maximumAge: 0, //定位结果缓存0毫秒，默认：0
          convert: true, //自动偏移坐标，偏移后的坐标为高德坐标，默认：true
          showButton: true, //显示定位按钮，默认：true
          buttonPosition: "LB", //定位按钮停靠位置，默认：'LB'，左下角
          buttonOffset: new window.AMap.Pixel(10, 20), //定位按钮与设置的停靠位置的偏移量，默认：Pixel(10, 20)
          showMarker: true, //定位成功后在定位到的位置显示点标记，默认：true
          showCircle: true, //定位成功后用圆圈表示定位精度范围，默认：true
          panToLocation: true, //定位成功后将定位到的位置作为地图中心点，默认：true
          zoomToAccuracy: true //定位成功后调整地图视野范围使定位位置及精度范围视野内可见，默认：false
        });

        await that.map.addControl(geolocation);

        await window.AMap.event.addListener(geolocation, "complete", e => {
          console.log(e, "eeee");
          that.clickSelected = true;
          that.positionPicker.start(e.position);
        });
      });
    },
    /**
     * 点击搜索
     */
    clickSearchResult(val) {
      this.searchResult = [];

      this.searchValue = "";
      this.clickSelected = true;

      this.positionPicker.start(val.location);
    },

    clickOkResult(val) {
      console.log(val, "选中位置");
      this.clickSelected = false;
      var list = this.positionResult;

      list.map(v => {
        // console.log(v, k);

        if (val.name == v.name) {
          v.selected = true;
        } else {
          v.selected = false;
        }
      });
      this.positionResult = list;

      this.positionPicker.start(val.location);
      // this.positionPicker.setMode("dragMarker");

      //
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

        // 加载状态结束
        this.loading = false;

        // 数据全部加载完成
        if (this.list.length >= 40) {
          this.finished = true;
        }
      }, 1000);
    },

    onSearch(val) {
      var that = this;
      window.AMap.plugin("AMap.Autocomplete", function() {
        // 实例化Autocomplete
        var autoOptions = {
          // input 为绑定输入提示功能的input的DOM ID
          // input: "searchInput1",
          city: val
        };
        var autoComplete = new window.AMap.Autocomplete(autoOptions);
        // 无需再手动执行search方法，autoComplete会根据传入input对应的DOM动态触发search
        autoComplete.search(val, function(status, result) {
          // 搜索成功时，result即是对应的匹配数据

          if (result.info && result.info == "OK") {
            var list = [];
            result.tips.map(v => {
              if (v.location != "") {
                list.push(v);
              }
            });

            that.searchResult = list;
          } else {
            that.searchResult = [];
          }
        });

        autoComplete.on("select", function(obj) {
          that.positionPicker.start(obj.poi.location);
        });
      });
    },
    changeSearch(val) {
      console.log(val);
    },
    initMap(val) {
      console.log(val);
      var that = this;

      window.AMapUI.loadUI(
        ["misc/PositionPicker", "misc/PoiPicker", "overlay/SimpleMarker"],
        function(PositionPicker, PoiPicker, MarkerList) {
          console.log(PoiPicker);
          console.log(MarkerList);

          console.log(PositionPicker, "PositionPicker");

          that.positionPicker = new PositionPicker({
            mode: "dragMap",
            map: that.map
          });

          that.positionPicker.on("success", function(positionResult) {
            console.log(positionResult, "positionResult");
            if (!that.clickSelected) {
              return;
            }
            var pois = positionResult.regeocode.pois;

            var list = [];

            pois.map((v, k) => {
              if (k == 0) {
                list.push({
                  name: v.name,
                  address: v.address,
                  location: v.location,
                  selected: true
                });
              } else {
                list.push({
                  name: v.name,
                  address: v.address,
                  location: v.location,
                  selected: false
                });
              }
            });

            console.log(list, "list");

            that.positionResult = list;

            //
          });
          that.positionPicker.on("fail", function(positionResult) {
            console.log(positionResult, "positionResult");
          });

          that.positionPicker.setMode("dragMap");

          console.log(that.map.getBounds().getSouthWest(), "*****");

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
  // display: none;
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
