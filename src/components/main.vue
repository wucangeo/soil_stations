<template>
  <div id="main">
    <div id="mapDiv">
      <!-- 地图底图切换 -->
      <div id="map-switch">
        <div @click="base_map_switch()" v-bind:class="{'vector-base': vector_base, 'image-base': !vector_base}"></div>
      </div>
      <!-- 用户头像 -->
      <div id="user-logo">
        <div v-if="user_name&&!show_data" class="user-in" @click="data_panel_switcher">
          <span class="user-in-logo"></span>
          <span class="user-name">{{user_name}}</span>
        </div>
        <div v-if="user_name&&show_data" class="user-in" @click="data_panel_switcher">
          <span class="user-name" style="float:none !important;">{{user_name}}</span>
          <span class="user-in-logo" style="float:right !important;"></span>
        </div>
        <div v-if="!user_name" class="user-out"></div>
      </div>
      <!-- 功能区面板 -->
      <div id="data-panel" v-show="show_data"></div>
    </div>
  </div>
</template>
<script>
export default {
  name: "main-vue",
  data() {
    return {
      vector_base: true,
      image_base: null,
      user_name: "赵文武",
      show_data: false,
      data_info: [[116.417854, 39.921988, "地址：北京市东城区王府井大街88号乐天银泰百货八层"],
      [116.406605, 39.921585, "地址：北京市东城区东华门大街"],
      [116.412222, 39.912345, "地址：北京市东城区正义路甲5号"]
      ]
    }
  },
  mounted() {
    var zoom = CONFIG.init_zoom;
    var x = CONFIG.init_center.x, y = CONFIG.init_center.y;
    map = new T.Map('mapDiv', {
      // projection: 'EPSG:4326'
    });
    map.centerAndZoom(new T.LngLat(x, y), zoom);
    //创建缩放平移控件对象
    var control = new T.Control.Zoom();
    //添加缩放平移控件
    map.addControl(control);
    //允许鼠标滚轮缩放地图
    map.enableScrollWheelZoom();
    //创建比例尺控件对象
    var scale = new T.Control.Scale();
    //添加比例尺控件
    map.addControl(scale);

    this.fetchStations();
  },
  methods: {
    // 地图底图切换
    base_map_switch() {
      this.vector_base = !this.vector_base;
      if (this.vector_base) {
        map.removeLayer(this.image_base);
      } else {
        var imageURL = "http://t0.tianditu.cn/img_w/wmts?" +
          "SERVICE=WMTS&REQUEST=GetTile&VERSION=1.0.0&LAYER=img&STYLE=default&TILEMATRIXSET=w&FORMAT=tiles" +
          "&TILEMATRIX={z}&TILEROW={y}&TILECOL={x}";
        //创建自定义图层对象
        this.image_base = new T.TileLayer(imageURL, { minZoom: 1, maxZoom: 18 });
        //将图层增加到地图上
        map.addLayer(this.image_base);
      }
    },
    //是否显示数据面板切换
    data_panel_switcher() {
      this.show_data = !this.show_data;
    },
    //获取目前所有站点
    fetchStations() {
      //...
      this.initPops();
    },
    //初始化地图标记点
    initPops() {
      for (var i = 0; i < this.data_info.length; i++) {
        var marker = new T.Marker(new T.LngLat(this.data_info[i][0], this.data_info[i][1]));  // 创建标注
        var content = this.data_info[i][2];
        map.addOverLay(marker);               // 将标注添加到地图中
        addClickHandler(content, marker);
      }
      function addClickHandler(content, marker) {
        marker.addEventListener("click", function(e) {
          openInfo(content, e)
        }
        );
      }
      function openInfo(content, e) {
        var point = e.lnglat;
        marker = new T.Marker(point);// 创建标注
        var markerInfoWin = new T.InfoWindow(content, { offset: new T.Point(0, -30) }); // 创建信息窗口对象
        map.openInfoWindow(markerInfoWin, point); //开启信息窗口
      }
    },

  }
}
</script>

<style scoped>
#allmap {
  width: 100%;
  height: 100%;
  overflow: hidden;
  margin: 0;
  font-family: "微软雅黑";
}

body,
html {
  width: 100%;
  height: 100%;
  margin: 0;
  font-family: "Microsoft YaHei"
}

#mapDiv {
  /* width: 300px;
  height: 400px */
  position: absolute;
  top: 0px;
  bottom: 0px;
  left: 0px;
  right: 0px;
  /*z-index:10;*/
  overflow: hidden;
}

#map-switch {
  width: 70px;
  height: 65px;
  position: absolute;
  bottom: 65px;
  left: 10px;
  z-index: 1000;
}

input,
b,
p {
  margin-left: 5px;
  font-size: 14px
}

.vector-base,
.image-base {
  width: 100%;
  height: 100%;
  cursor: pointer;
}

.vector-base {
  background-image: url("../assets/image_base.jpg")
}

.image-base {
  background-image: url("../assets/vector_base.jpg")
}

.user-in,
.user-out {
  position: absolute;
  top: 20px;
  right: 20px;
  z-index: 1020;
  overflow: hidden;
  cursor: pointer;
}

.user-in {
  height: 40px;
  width: auto;
  font-size: 16px;
  background-color: #3385ff;
  color: #fff;
  /* padding: 0 15px; */
  line-height: 40px;
  border-radius: 20px;
}

.user-in-logo,
.user-name {
  display: inline-block;
}

.user-name {
  float: right;
  padding: 0 8px;
}

.user-out,
.user-in-logo {
  height: 40px;
  width: 40px;
  background: #fff;
  background-size: cover;
  border-radius: 50%;
  background-repeat: no-repeat;
  background-position: center center;
  background-image: url("../assets/user.png");
}

#data-panel {
  position: absolute;
  top: 70px;
  right: 10px;
  bottom: 10px;
  width: 300px;
  /* height: 100%; */
  background-color: #0ef;
  z-index: 1030;
}
</style>
