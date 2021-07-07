<template>
  <div>
    <div ref="olMap" id="map" class="ol-map"/>
    <div id="popup" class="ol-popup">
      <a href="#" id="popup-closer" class="ol-popup-closer"></a>
      <div id="popup-content" class="popup-content"></div>
    </div>

  </div>
</template>

<script>
import 'ol/ol.css'
import TileLayer from 'ol/layer/Tile'
import VectorLayer from 'ol/layer/Vector'
import VectorSource from 'ol/source/Vector'
import XYZ from 'ol/source/XYZ'
import {Map, View, Feature} from 'ol'
import Overlay from 'ol/Overlay'
import {
  Style,
  Stroke,
  Fill,
  Icon,
  Text
} from 'ol/style'
import {Point} from 'ol/geom'
import {Cluster} from 'ol/source'
import {fromLonLat} from 'ol/proj'
// import {toStringHDMS} from 'ol/coordinate'

export default {
  name: 'defineMap',
  data () {
    return {
      imgUrl: require('../../src/assets/pointer.png'),
      map: null,
      pointLayer: null,
      diffLayer: null,
      overlay: null,
      clusterData:null,
    }
  },
  mounted () {
    this.init()
    this.diffLayer = new VectorLayer({
      source: new VectorSource()
    })
    this.clusterData = {
      点A: {center: {lng: 119.99082431579592, lat: 30.277877393725625}},
      点B: {center: {lng: 119.92396220947268, lat: 30.256453639392525}}
    }
    let points = [
      {name: '点A', value: 1},
      {name: '点B', value: 1}
    ]
    this.addCluster(this.clusterData, points, true)
    this.addPopup()
    this.addPoint(this.diffLayer, [119.97846324707033, 30.274838322154054])
  },
  methods: {
    init () {
      const tileLayer = new TileLayer({
        source: new XYZ({
          url: `tiles/{z}/{x}/{y}.png`
        })
      })
      // 初始化地图
      this.map = null
      this.map = new Map({
        layers: [tileLayer],
        view: new View({
          center: fromLonLat([119.97846324707033, 30.274838322154054]), // 地图中心点
          zoom: 14, // 缩放级别
          minZoom: 0, // 最小缩放级别
          maxZoom: 18, // 最大缩放级别
          constrainResolution: true// 因为存在非整数的缩放级别，所以设置该参数为true来让每次缩放结束后自动缩放到距离最近的一个整数级别，这个必须要设置，当缩放在非整数级别时地图会糊
        }),
        target: this.$refs.olMap// DOM容器
      })
    },
    addCluster (clusterData, points) {
      let source = new VectorSource()
      let clusterSource = new Cluster({
        distance: parseInt(20, 10),
        source: source
      })
      this.pointLayer = new VectorLayer({
        source: clusterSource
        // style: this.clusterStyle
      })
      this.pointLayer.setStyle(this.clusterStyle())
      this.map.addLayer(this.pointLayer)
      for (const key in clusterData) {
        points.forEach(e => {
          if (e.name === key) {
            let point = fromLonLat([
              clusterData[key].center.lng,
              clusterData[key].center.lat
            ])
            var f = new Feature({
              geometry: new Point(point),
              data: {
                path: 'homepage'
              }
            })
            f.set('name', e.name)
            f.set('value', e.value)
            source.addFeature(f)
          }
        })
      }
    },
    // 添加弹窗
    addPopup () {
      var container = document.getElementById('popup')
      var closer = document.getElementById("popup-closer");
      var content = document.getElementById('popup-content')

      this.overlay = new Overlay({
        element: container,
        autoPan: true,
        autoPanAnimation: {
          duration: 250
        }
      })

      this.map.addOverlay(this.overlay)
      this.map.on('click', function (evt) {
        //查询当前点击的地方是否存在要素(图标)
        var feature = this.map.forEachFeatureAtPixel(evt.pixel, function (feature) { return feature; });
        console.log(feature)
        if (feature) {
          content.innerHTML = ''; //清空popup的内容容器
          // 点击尺 （这里是尺(米)，并不是经纬度）;
          content.innerHTML = `
                <p>A<p>
                <p>状态：正常</p>安装人员：lisa`
          this.overlay.setPosition(evt.coordinate) // 把 overlay 显示到指定的 x,y坐标
        }
      }.bind(this))
      this.map.on('pointermove', function (e) {
        var pixel = this.map.getEventPixel(e.originalEvent);
        var hit = this.map.hasFeatureAtPixel(pixel);
        this.map.getTargetElement().style.cursor = hit ? 'pointer' : '';
      }.bind(this));

      closer.onclick = function() {
        this.overlay.setPosition(undefined);
        closer.blur();
        return false;
      }.bind(this);
    },
    clusterStyle () {
      return () => {
        var style = new Style({
          image: new Icon({
            src: this.imgUrl,
            // offset: [-15, 0], // 偏移量
            anchor: [1, 1]
          }),
          text: new Text({
            text: '我是一个点',
            fill: new Fill({
              color: '#FFF'
            }),
            font: '12px Calibri,sans-serif',
            stroke: new Stroke({
              color: 'red',
              width: 5
            })
          })
        })
        return style
      }
    },
    // 添加点位
    addPoint (pointLayer, condiation) {
      // 添加图层
      this.map.addLayer(pointLayer)
      // 循环添加feature
      // 创建feature，一个feature就是一个点坐标信息
      let feature = new Feature({
        geometry: new Point(
            // 经纬度需要转换一下
            fromLonLat(condiation)
        ),
        data: {
          path: 'homepage'
        }
      })
      feature.set('marker', () => {
        console.log('1')
      })
      this.map.on('singleClick', (e) => {
        console.log(e)
      })
      feature.setStyle(new Style({
        // 设置图片效果
        image: new Icon({
          src: this.imgUrl,
          // offset: [-15, 0], // 偏移量
          anchor: [1, 1]
        }),
        name: '我是一个点',
        text: new Text({
          textAlign: 'right',
          textBaseline: 'middle',
          text: 'xxx'
        })
      }))
      let featuresArr = []
      featuresArr.push(feature)
      // 批量添加feature
      pointLayer.getSource().addFeatures(featuresArr)
    },
  }
}
</script>

<style scoped>
.ol-map {
  width: 100%;
  font-size: 14px;
  height: calc(100vh - 96px);
}

.ol-popup {
  position: absolute;
  background-color: white;
  -webkit-filter: drop-shadow(0 1px 4px rgba(0, 0, 0, 0.2));
  filter: drop-shadow(0 1px 4px rgba(0, 0, 0, 0.2));
  padding: 15px;
  border-radius: 10px;
  border: 1px solid #cccccc;
  bottom: 12px;
  left: -50px;
}

.ol-popup:after,
.ol-popup:before {
  top: 100%;
  border: solid transparent;
  content: " ";
  height: 0;
  width: 0;
  position: absolute;
  pointer-events: none;
}

.ol-popup:after {
  border-top-color: white;
  border-width: 10px;
  left: 48px;
  margin-left: -10px;
}

.ol-popup:before {
  border-top-color: #cccccc;
  border-width: 11px;
  left: 48px;
  margin-left: -11px;
}

.ol-popup-closer {
  text-decoration: none;
  position: absolute;
  top: 2px;
  right: 8px;
}

.popup-content {
  width: 400px;
}

.ol-popup-closer:after {
  content: "✖";
}

</style>
