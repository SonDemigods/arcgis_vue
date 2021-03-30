<template>
  <div class="home">
    <div class="tool-btn">
      <div :class="{'switch-tool':true,'highlight':type}"
           @click="switchLayer({type:1, msg:'影像'})">
        <img src="../assets/img-c.png"
             alt=""
             width="100%">
      </div>
      <div :class="{'switch-tool':true, 'highlight':!type}"
           @click="switchLayer({type:2,msg:'矢量'})">
        <img src="../assets/vec_c.png"
             alt=""
             width="100%">
      </div>
    </div>

    <div ref="map"
         id="map"
         style="width: 100vw;height: 100%;"></div>
  </div>
</template>

<script>
import {
  loadCss,
  loadModules
} from 'esri-loader'
import tileInfo from '@/util/tileInfo'

export default {
  name: 'BaseMap',
  components: {},
  props: {
    // 打点
    point: {
      type: Array,
      default () {
        return []
      }
    },
    // 多边型
    polygon: {
      type: Array,
      default () {
        return []
      }
    }
  },
  data () {
    return {
      type: false,
      gisConstructor: {}, //gis 构造函数
      gisInstance: {}, // gis 实例
      layerID: {},
      layersInstance: {},
      gisModules: [
        'esri/Map',
        'esri/views/MapView',
        'esri/layers/WebTileLayer',
        'esri/layers/support/TileInfo',
        'esri/config',
        'esri/geometry/Point',
        'esri/geometry/Polygon',
        'esri/Graphic',
        'esri/widgets/ScaleBar',
        'dojo/domReady!',
        'esri/layers/GraphicsLayer'
      ]
    }
  },
  mounted () {
    this.init()
  },
  methods: {
    /**
     * @functionName init
     * @description 组件初始化
     * @author 张航
     * @date 2021-03-25 17:16:53
     * @version V1.0.0
     */
    init () {
      // 加载css;
      loadCss()
      // 加载模块
      loadModules(this.gisModules).then(this.loadMap)
    },
    /**
     * @functionName loadMap
     * @param {Object} args esri加载的模块
     * @description 加载地图及相关的自定义图层
     * @author 张航
     * @date 2021-03-25 17:18:52
     * @version V1.0.0
     */
    loadMap (args) {
      let container = this.$refs.map

      // 处理构造函数,绑定到gisConstructor,方便组件内其他地方调用
      for (let k in args) {
        let name = this.gisModules[k].split('/').pop()
        this.gisConstructor[name] = args[k]
      }
      // 初始化各种图层
      let cva_c = this.initYiledLayer('cva_c') //矢量注记
      let vec_c = this.initYiledLayer('vec_c') //矢量底图

      let cia_c = this.initYiledLayer('cia_c') //影像注记
      let img_c = this.initYiledLayer('img_c') //影像地图

      this.layersInstance.cva_c = cva_c
      this.layersInstance.vec_c = vec_c

      this.layersInstance.cia_c = cia_c
      this.layersInstance.img_c = img_c

      // 初始化地图
      let map = new this.gisConstructor.Map({
        spatialReference: {
          wkid: 4326
        },
        basemap: {
          baseLayers: [this.layersInstance.vec_c, this.layersInstance.cva_c]
        }
      })
      let view = new this.gisConstructor.MapView({
        container: container,
        spatialReference: {
          wkid: 4326
        },
        map: map,
        scale: 100,
        center: [117.204665, 39.136475]
      })

      this.view = view

      let applicationDiv = document.createElement('div')
      this.gisInstance.map = map
      this.gisInstance.mapView = view
      let full = new this.gisConstructor.ScaleBar({
        view: view,
        element: applicationDiv
      })

      view.ui.add(full, 'bottom-right')

      // 创建自定义图层
      const graphic = this.createGraphic()

      // 绘制多边型和点
      view.graphics.addMany(graphic)
    },
    /**
     * @functionName createGraphic
     * @return {Array} 自定义图层内容
     * @description 创建自定义图层内容
     * @author 张航
     * @date 2021-03-26 09:37:18
     * @version V1.0.0
     */
    createGraphic () {
      let res = []

      this.polygon.map(item => {
        const polygonGraphic = this.polygonGraphic(item)

        res.push(polygonGraphic)
      })

      this.point.map(item => {
        const pointGraphic = this.pointGraphic(item)

        res.push(pointGraphic)
      })

      console.log(res)
      return res
    },
    /**
     * @functionName switchLayer
     * @param {Object} para 切换的类型
     * @description 切换底图显示类型
     * @author 张航
     * @date 2021-03-25 17:19:52
     * @version V1.0.0
     */
    switchLayer (para) {
      if (para.type === 1) {
        this.type = true
        this.gisInstance.map.layers.add(this.layersInstance.img_c)
        this.gisInstance.map.layers.add(this.layersInstance.cia_c)

        this.gisInstance.map.layers.remove(this.layersInstance.vec_c)
        this.gisInstance.map.layers.remove(this.layersInstance.cva_c)
      } else {
        this.type = false
        this.gisInstance.map.layers.add(this.layersInstance.vec_c)
        this.gisInstance.map.layers.add(this.layersInstance.cva_c)

        this.gisInstance.map.layers.remove(this.layersInstance.img_c)
        this.gisInstance.map.layers.remove(this.layersInstance.cia_c)
      }
    },
    /**
     * @functionName initYiledLayer
     * @param {String} mapType 底图类型
     * @return {Object} 返回底图地址
     * @description 初始化底图服务
     * @author 张航
     * @date 2021-03-25 17:21:14
     * @version V1.0.0
     */
    initYiledLayer (mapType) {
      let result = this.gisConstructor.WebTileLayer(
        'http://{subDomain}.tianditu.com/DataServer?T=' +
        mapType +
        '&X={col}&Y={row}&L={level}' +
        '&tk=fa66ded203b504471b9f7ae00c69611f',
        {
          subDomains: ['t0', 't1', 't2', 't3', 't4', 't5', 't6', 't7'],
          tileInfo: tileInfo,
          spatialReference: {
            wkid: 4326
          }
        }
      )

      this.layerID[mapType] = result.id
      return result
    },
    /**
     * @functionName pointGraphic
     * @param {Object} para 经纬度信息
     * @return {Object} 绘制的点信息
     * @description 根据经纬度绘制点
     * @author 张航
     * @date 2021-03-25 17:12:37
     * @version V1.0.0
     */
    pointGraphic (para) {
      console.log(this.gisConstructor)
      let point = new this.gisConstructor.Point({
        longitude: para.lon,
        latitude: para.lat
      })
      let symbol = {
        type: 'simple-marker',
        color: [255, 0, 0, 0.5],
        size: 20,
        outline: {
          color: [255, 0, 0],
          width: 2
        }
      }

      let popupTemplate = {
        actions: [],
        title: '点',
        content: '<div>这是一个点</div>'
      }

      return new this.gisConstructor.Graphic({
        symbol,
        geometry: point,
        popupTemplate
      })
    },
    /**
     * @functionName polygonGraphic
     * @param {Array} rings 顶点坐标
     * @return {Object} 绘制的多边型信息
     * @description 根据顶点坐标绘制多边型
     * @author 张航
     * @date 2021-03-25 17:08:35
     * @version V1.0.0
     */
    polygonGraphic (rings) {
      let polygon = new this.gisConstructor.Polygon({
        rings
      })
      let symbol = {
        type: 'simple-fill',
        color: [255, 255, 0, 0.5],
        // size: 50,
        outline: {
          color: [255, 255, 0],
          width: 2
        }
      }

      let popupTemplate = {
        actions: [],
        title: '多边型',
        content: '<div>这是一个多边型</div>'
      }

      return new this.gisConstructor.Graphic({
        symbol,
        geometry: polygon,
        popupTemplate
      })
    }
  },
  beforeDestroy () {
    if (this.view) {
      this.view.container = null
    }
  }
}

</script>

<style lang = "less">
.home {
  height: 100vh;
  background: #cccccc;
  position: relative;
}

.tool-btn {
  position: absolute;
  top: 20px;
  right: 20px;
  z-index: 2;
  display: flex;
  width: 250px;
  justify-content: space-between;
  cursor: pointer;

  .switch-tool {
    width: 100px;
    height: 100px;
    border: 4px solid #fff;
    border-radius: 4px;
  }

  .highlight {
    border-color: #09f;
  }
}

.menu {
  position: absolute;
  left: 0;
  top: 200px;
  z-index: 2;

  .el-submenu__title i {
    color: #10d5f5;
  }
}
</style>
