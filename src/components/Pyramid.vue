<!--
 * @Description:
 * @Date: 2021-10-09 10:56:46
-->
<template>
  <div ref="canvas-warpper" id="canvas-warpper">
    <div id="canvas-tooltip"></div>
  </div>
</template>

<script>
export default {
  name: 'Pyramid',
  props: {
    options: {
      type: Object,
      default: () => {
        return {
          title: '',
          // 主体离边框距离
          distance: [0, 0],
          // 主体偏移值 (x,y)
          offset: [0, 0],
          // 排序(max , min)优先
          sort: '',
          // 颜色
          color: ['#80FFA5', '#00DDFF', '#37A2FF', '#FF0087', '#FFBF00'],
          // 格式化字体输出
          fontFormatter: () => {
            return 'default'
          },
          // 鼠标点击事件
          lMouseClick: false,
          // 鼠标移动事件
          lMouseMove: false,
          // tooltip信息配置
          tooltip: {
            show: true, // 是否显示
            fontColor: '#000', //  字体内部颜色
            fontSize: 14, // 字体大小
            backgroundColor: '#fff', // tooltip背景
            formatter: null, // 回调方法
            z: 999999 // tooltip z-index层级
          },
          // 样式
          infoStyle: {
            stroke: false, // 是否描边
            strokeColor: '#fff', //描边颜色
            size: null, // 字体大小
            color: null, //颜色
            highlightedColor: '#fff', // 高亮颜色
            setLineDash: [0, 0], // 虚线值
            width: -10, // 设置多少 就会在基础上加上设置的值
            offset: [0, 0], // 字体x,y的偏移度
            dotSize: 4 //点大小
          }
        }
      }
    },
    // 渲染数据
    data: {
      type: Array,
      default: () => {
        return []
      }
    }
  },
  watch: {
    data: {
      deep: true,
      handler(newValue) {
        this.init()
      }
    }
  },
  computed: {
    integration() {
      return {
        title: this.options.title ? this.options.title : '',
        // 主体离边框距离
        distance: this.options.distance ? this.options.distance : [0, 0],
        // 主体偏移值 (x,y)
        offset: this.options.offset ? this.options.offset : [0, 0],
        // 排序(max , min)优先
        sort: this.options.sort ? this.options.sort : '',
        // 鼠标点击事件
        lMouseClick: this.options.lMouseClick ? this.options.lMouseClick : false,
        // 鼠标移动事件
        lMouseMove: this.options.lMouseMove ? this.options.lMouseMove : false,
        // 颜色
        color: this.options.color ? this.options.color : ['#80FFA5', '#00DDFF', '#37A2FF', '#FF0087', '#FFBF00'],
        // 格式化字体输出
        fontFormatter: this.options.fontFormatter
          ? this.options.fontFormatter
          : () => {
              return 'default'
            },
        // tooltip显示
        tooltip: {
          show: this.options.tooltip ? (this.options.tooltip.show ? this.options.tooltip.show : true) : true, // 是否显示
          fontColor: this.options.tooltip
            ? this.options.tooltip.fontColor
              ? this.options.tooltip.fontColor
              : '#000'
            : '#000', //  字体内部颜色
          fontSize: this.options.tooltip ? (this.options.tooltip.fontSize ? this.options.tooltip.fontSize : 14) : 14, // 字体大小
          backgroundColor: this.options.tooltip
            ? this.options.tooltip.backgroundColor
              ? this.options.tooltip.backgroundColor
              : '#fff'
            : '#fff', // tooltip背景
          formatter: this.options.tooltip
            ? this.options.tooltip.formatter
              ? this.options.tooltip.formatter
              : null
            : null, // 返回方法
          z: this.options.tooltip ? (this.options.tooltip.z ? this.options.tooltip.z : 999999) : 999999 // tooltip z-index层级
        },
        // 样式
        infoStyle: {
          stroke: this.options.infoStyle
            ? this.options.infoStyle.stroke
              ? this.options.infoStyle.stroke
              : false
            : false, //是否描边
          strokeColor: this.options.infoStyle
            ? this.options.infoStyle.strokeColor
              ? this.options.infoStyle.strokeColor
              : '#fff'
            : '#fff', // 描边颜色
          size: this.options.infoStyle ? (this.options.infoStyle.size ? this.options.infoStyle.size : null) : null, // 字体大小
          color: this.options.infoStyle ? (this.options.infoStyle.color ? this.options.infoStyle.color : null) : null, //颜色
          width: this.options.infoStyle
            ? this.options.infoStyle.width || this.options.infoStyle.width !== 0
              ? this.options.infoStyle.width
              : -10
            : -10, // 设置多少 就会在基础上加上设置的值
          offset: this.options.infoStyle
            ? this.options.infoStyle.offset
              ? this.options.infoStyle.offset
              : [0, 0]
            : [0, 0], // 字体x,y的偏移度
          setLineDash: this.options.infoStyle
            ? this.options.infoStyle.setLineDash
              ? this.options.infoStyle.setLineDash
              : [0, 0]
            : [0, 0], //虚线值
          highlightedColor: this.options.infoStyle
            ? this.options.infoStyle.highlightedColor
              ? this.options.infoStyle.highlightedColor
              : '#fff'
            : '#fff', //高亮颜色
          dotSize: this.options.infoStyle
            ? this.options.infoStyle.dotSize || this.options.infoStyle.dotSize !== 0
              ? this.options.infoStyle.dotSize
              : 4
            : 4 //点大小
        }
      }
    }
  },
  data() {
    return {
      // canvas 主体
      canvas: null,
      // 图像渲染内容
      ctx: null,
      // 画布高度
      canvasHeight: 0,
      // 画布宽度
      canvasWidth: 0,
      // 画布中心点 [x,y]
      canvasCenter: [0, 0],
      // 金字塔四个点位置
      point: {
        top: [0, 0],
        left: [0, 0],
        right: [0, 0],
        bottom: [0, 0],
        shadow: [0, 0]
      },
      // 数据信息
      dataInfo: [],
      // 金字塔顶端角度信息
      topAngle: {
        LTB: 0,
        RTB: 0
      },
      // tooltip 模板
      tooltipDiv: `<div  style="margin: 0px 0 0; line-height: 1;border-color: $[backgroundColor]$ ;background-color: $[backgroundColor]$;color: $[fontColor]$;
    border-width: 1px;border-radius: 4px;padding: 10px;pointer-events: none;box-shadow: rgb(0 0 0 / 20%) 1px 2px 10px;border-style: solid;white-space: nowrap;">
        <div style="margin: 0px 0 0; line-height: 1">
          <div style="font-size: $[fontSize]$px; color: $[fontColor]$; font-weight: 400; line-height: 1"> $[title]$ </div>
          <div style="margin: 10px 0 0; line-height: 1">
            <div style="margin: 0px 0 0; line-height: 1">
              <div style="margin: 0px 0 0; line-height: 1">
                <span
                  style="
                    display: inline-block;
                    margin-right: 4px;
                    border-radius: 10px;
                    width: 10px;
                    height: 10px;
                    background-color: $[color]$;
                  "
                ></span>
                <span style="font-size: $[fontSize]$px; color: $[fontColor]$; font-weight: 400; margin-left: 2px">$[name]$</span>
                <span style="float: right; margin-left: 20px; font-size: $[fontSize]$px; color: $[fontColor]$; font-weight: 900">$[val]$</span>
                <div style="clear: both"></div>
              </div>
              <div style="clear: both"></div>
            </div>
            <div style="clear: both"></div>
          </div>
          <div style="clear: both"></div>
        </div>
        <div style="clear: both"></div>
      </div>`
    }
  },
  async created() {},
  mounted() {
    this.init()
  },
  methods: {
    init() {
      if (!this.data || !this.data.length) return false
      // 数据总量
      let totalData = 0
      this.data.forEach(element => {
        if (isNaN(element.value)) {
          throw '"data" 中的数据类型错误，请检查 "value" 的数据类型'
        }
        totalData = totalData + Number(element.value)
      })
      this.dataInfo = this.data.map(item => {
        const accounted = (item.value / totalData) * 100
        return { ...item, accounted, title: this.integration.title }
      })
      if (this.integration.sort === 'max') {
        this.dataInfo.sort((a, b) => {
          return a.value - b.value
        })
      } else if (this.integration.sort === 'min') {
        this.dataInfo.sort((a, b) => {
          return b.value - a.value
        })
      }
      this.initCanvasBaseInfo()
      this.paintDataInfo()
      this.paintingText()
      this.paintingBody()
      this.eventRegistered()
    },
    /**
     * @description: 初始化canvas基本信息
     * @param {*}
     * @return {*}
     * @author: 舒
     */
    initCanvasBaseInfo() {
      let el = document.getElementById('canvas-warpper')
      // 创建canvas元素
      this.canvas = document.createElement('canvas')
      // 把canvas元素节点添加在el元素下
      el.appendChild(this.canvas)
      this.canvasWidth = el.offsetWidth
      this.canvasHeight = el.offsetHeight
      // 将canvas元素设置与父元素同宽
      this.canvas.setAttribute('width', this.canvasWidth)
      // 将canvas元素设置与父元素同高
      this.canvas.setAttribute('height', this.canvasHeight)
      this.canvasCenter = [
        Math.round((this.canvasWidth - this.integration.distance[0] * 2) / 2) + this.integration.distance[0],
        Math.round((this.canvasHeight - this.integration.distance[1] * 2) / 2) + this.integration.distance[1]
      ]
      if (this.canvas.getContext) {
        this.ctx = this.canvas.getContext('2d')
        // 金字塔基本点位置
        this.point.top = [this.canvasCenter[0] - this.canvasWidth / 13, this.integration.distance[1]]
        this.point.left = [
          this.integration.distance[0] * 1.5,
          this.canvasHeight - this.integration.distance[1] - this.canvasHeight / 5
        ]
        this.point.right = [
          this.canvasWidth - this.integration.distance[0] * 1.9,
          this.canvasHeight - this.integration.distance[1] - this.canvasHeight / 5
        ]
        this.point.bottom = [
          this.canvasCenter[0] - this.canvasWidth / 13,
          this.canvasHeight - this.integration.distance[1]
        ]
        this.point.shadow = [
          this.integration.distance[0] - this.canvasCenter[0] / 5,
          this.canvasHeight / 1.2 - this.integration.distance[1]
        ]
        for (const key in this.point) {
          this.point[key][0] = this.point[key][0] + this.integration.offset[0]
          this.point[key][1] = this.point[key][1] + this.integration.offset[1]
        }
      } else {
        throw 'canvas下未找到 getContext方法'
      }
      this.topAngle.LTB = this.angle(this.point.top, this.point.left, this.point.bottom)
      this.topAngle.RTB = this.angle(this.point.top, this.point.right, this.point.bottom)
      // 计算各数据点位置
      this.calculationPointPosition(this.dataInfo)
    },
    // ======================================事件==========================================
    /**
     * @description: 鼠标事件注册
     * @param {*}
     * @return {*}
     * @author: 舒
     */
    eventRegistered() {
      const canvasWarpper = document.getElementById('canvas-warpper')
      //注册事件
      canvasWarpper.addEventListener('mouseup', this.doMouseUp, false)
      if (this.integration.lMouseClick) {
        canvasWarpper.addEventListener('mousedown', this.doMouseDown, false)
      }
      if (this.integration.lMouseMove) {
        canvasWarpper.addEventListener('mousemove', this.doMouseMove, false)
      }
      // //注册事件
      // this.canvas.addEventListener('mousedown', this.doMouseDown, false)
      // this.canvas.addEventListener('mouseup', this.doMouseUp, false)
      // this.canvas.addEventListener('mousemove', this.doMouseMove, false)
    },
    /**
     * @description: 鼠标按下
     * @param {*} e
     * @return {*}
     * @author: 舒
     */
    // eslint-disable-next-line no-unused-vars
    doMouseDown(e) {
      if (this.integration.lMouseClick) {
        const x = e.pageX
        const y = e.pageY
        if (this.determineDataMouse(this.getLocation(x, y))) {
          this.$emit('pyramidClick', this.determineDataMouse(this.getLocation(x, y)))
        }
      }
    },
    /**
     * @description: 鼠标弹起
     * @param {*} e
     * @return {*}
     * @author: 舒
     */
    // eslint-disable-next-line no-unused-vars
    doMouseUp(e) {},
    /**
     * @description: 鼠标移动
     * @param {*} e
     * @return {*}
     * @author: 舒
     */
    // eslint-disable-next-line no-unused-vars
    doMouseMove(e) {
      const x = e.pageX
      const y = e.pageY
      this.highlightCurrentRegion(this.determineDataMouse(this.getLocation(x, y)))
      if (this.integration.tooltip.show) {
        this.showTooltip(this.determineDataMouse(this.getLocation(x, y)), this.getLocation(x, y))
      }
    },

    /**
     *  @description 判断一个点是否在多边形内部
     *  @param points 多边形坐标集合
     *  @param testPoint 测试点坐标
     *  @author: 舒
     *  返回true为真，false为假
     */
    insidePolygon(points, testPoint) {
      const x = testPoint[0],
        y = testPoint[1]
      let inside = false
      for (let i = 0, j = points.length - 1; i < points.length; j = i++) {
        const xi = points[i][0],
          yi = points[i][1]
        const xj = points[j][0],
          yj = points[j][1]

        const intersect = yi > y !== yj > y && x < ((xj - xi) * (y - yi)) / (yj - yi) + xi
        if (intersect) inside = !inside
      }
      return inside
    },
    /**
     * @description: 获取当前鼠标坐标
     * @param {*}
     * @return {*}
     * @author: 舒
     */
    getLocation(x, y) {
      const bbox = this.canvas.getBoundingClientRect()
      return [(x - bbox.left) * (this.canvas.width / bbox.width), (y - bbox.top) * (this.canvas.height / bbox.height)]
    },
    // ======================================算法==========================================

    /**
     * @description: 根据A点旋转指定角度后B点的坐标位置
     * @param {*} ptSrc 圆上某点(初始点);
     * @param {*} ptRotationCenter 圆心点
     * @param {*} angle 旋转角度°  -- [angle * M_PI / 180]:将角度换算为弧度
     * 【注意】angle 逆时针为正，顺时针为负
     * @return {*}
     * @author: 舒
     */
    rotatePoint(ptSrc, ptRotationCenter, angle) {
      const a = ptRotationCenter[0]
      const b = ptRotationCenter[1]
      const x0 = ptSrc[0]
      const y0 = ptSrc[1]
      const rx = a + (x0 - a) * Math.cos((angle * Math.PI) / 180) - (y0 - b) * Math.sin((angle * Math.PI) / 180)
      const ry = b + (x0 - a) * Math.sin((angle * Math.PI) / 180) + (y0 - b) * Math.cos((angle * Math.PI) / 180)
      const point = [rx, ry]
      return point
    },

    /**
     * @description: 求3点之间角度
     * @return {*} 点 a 的角度
     * @author: 舒
     */
    angle(a, b, c) {
      const A = { X: a[0], Y: a[1] }
      const B = { X: b[0], Y: b[1] }
      const C = { X: c[0], Y: c[1] }
      const AB = Math.sqrt(Math.pow(A.X - B.X, 2) + Math.pow(A.Y - B.Y, 2))
      const AC = Math.sqrt(Math.pow(A.X - C.X, 2) + Math.pow(A.Y - C.Y, 2))
      const BC = Math.sqrt(Math.pow(B.X - C.X, 2) + Math.pow(B.Y - C.Y, 2))
      const cosA = (Math.pow(AB, 2) + Math.pow(AC, 2) - Math.pow(BC, 2)) / (2 * AB * AC)
      const angleA = Math.round((Math.acos(cosA) * 180) / Math.PI)
      return angleA
    },
    /**
     * @description: 计算两点之间距离
     * @return {*}
     * @author: 舒
     */
    getDistanceBetweenTwoPoints(a, b) {
      const A = a[0] - b[0]
      const B = a[1] - b[1]
      const result = Math.sqrt(Math.pow(A, 2) + Math.pow(B, 2))
      return result
    },
    /**
     * @description: 计算数据的点位置
     * @param {*} val 点占比
     * @return {*}
     * @author: 舒
     */
    calculationPointPosition(val) {
      const LP = this.rotatePoint(this.point.left, this.point.top, this.topAngle.LTB * -1)
      const RP = this.rotatePoint(this.point.right, this.point.top, this.topAngle.RTB)
      let temporary = {
        left: [
          [0, 0],
          [0, 0],
          [0, 0]
        ],
        right: [
          [0, 0],
          [0, 0],
          [0, 0]
        ],
        middle: [
          [0, 0],
          [0, 0],
          [0, 0]
        ]
      }
      const dataInfo = val.map((item, index) => {
        if (index === 0) {
          for (const key in temporary) {
            if (key === 'left') {
              // 垂直后点的位置
              // 垂直后点点距离
              const vertical = [
                this.point.top[0],
                (LP[1] - this.point.top[1]) * (item.accounted / 100) + this.point.top[1]
              ]
              // 还原后点的位置
              temporary.left = [this.point.top, this.rotatePoint(vertical, this.point.top, this.topAngle.LTB), vertical]
            } else if (key === 'right') {
              // 垂直后点点距离
              const vertical = [
                this.point.top[0],
                (RP[1] - this.point.top[1]) * (item.accounted / 100) + this.point.top[1]
              ]
              // 还原后点的位置
              temporary.right = [
                this.point.top,
                this.rotatePoint(vertical, this.point.top, this.topAngle.RTB * -1),
                vertical
              ]
            } else if (key === 'middle') {
              // 垂直后点点距离
              temporary.middle = [
                this.point.top,
                [
                  this.point.top[0],
                  (this.point.bottom[1] - this.point.top[1]) * (item.accounted / 100) + this.point.top[1]
                ],
                [
                  this.point.top[0],
                  (this.point.bottom[1] - this.point.top[1]) * (item.accounted / 100) + this.point.top[1]
                ]
              ]
            }
          }
        } else {
          for (const key in temporary) {
            const vertical = JSON.parse(JSON.stringify(temporary[key][2]))
            if (key === 'left') {
              // 垂直后点点距离
              const vertical1 = [this.point.top[0], vertical[1] + (LP[1] - this.point.top[1]) * (item.accounted / 100)]
              // 还原后点的位置
              temporary.left = [
                this.point.top,
                this.rotatePoint(vertical1, this.point.top, this.topAngle.LTB),
                vertical1
              ]
            } else if (key === 'right') {
              // 垂直后点点距离
              const vertical1 = [this.point.top[0], vertical[1] + (RP[1] - this.point.top[1]) * (item.accounted / 100)]
              // 还原后点的位置
              temporary.right = [
                this.point.top,
                this.rotatePoint(vertical1, this.point.top, this.topAngle.RTB * -1),
                vertical1
              ]
            } else if (key === 'middle') {
              temporary.middle = [
                this.point.top,
                [this.point.top[0], (this.point.bottom[1] - this.point.top[1]) * (item.accounted / 100) + vertical[1]],
                [this.point.top[0], (this.point.bottom[1] - this.point.top[1]) * (item.accounted / 100) + vertical[1]]
              ]
            }
          }
        }

        return { ...item, temporary: JSON.parse(JSON.stringify(temporary)) }
      })
      this.dataInfo = dataInfo
    },
    /**
     * @description: 判断鼠标在哪层位置上
     * @param {*}
     * @return {*}
     * @author: 舒
     */
    determineDataMouse(mouseLocation) {
      let req = false
      for (let index = 0; index < this.dataInfo.length; index++) {
        if (this.insidePolygon(this.dataInfo[index].drawingPoint, mouseLocation)) {
          return (req = { l: index + 1, obj: this.dataInfo[index] })
        }
      }
      return req
    },
    // ======================================绘图==========================================
    /**
     * @description: 绘画主体
     * @param {*}
     * @return {*}
     * @author: 舒
     */
    paintingBody() {
      // // 右半边金字塔
      // this.ctx.fillStyle = "white";
      // this.ctx.beginPath();
      // this.ctx.moveTo(...this.point.top);
      // this.ctx.lineTo(...this.point.bottom);
      // this.ctx.lineTo(...this.point.right);
      // this.ctx.fill();
      // // 左半边金字塔
      // this.ctx.beginPath();
      // this.ctx.moveTo(...this.point.top);
      // this.ctx.lineTo(...this.point.bottom);
      // this.ctx.lineTo(...this.point.left);
      // this.ctx.fill();
      // 左半边金字塔阴影
      this.ctx.fillStyle = 'rgba(120,120,120,.15)'
      this.ctx.beginPath()
      this.ctx.moveTo(...this.point.top)
      this.ctx.lineTo(...this.point.bottom)
      this.ctx.lineTo(...this.point.left)
      this.ctx.fill()
      // 金字塔影子
      // this.ctx.beginPath()
      // this.ctx.moveTo(...this.point.left)
      // this.ctx.lineTo(...this.point.bottom)
      // this.ctx.lineTo(...this.point.shadow)
      // this.ctx.shadowColor = 'rgba(110,110,110,.3)'
      // // 将阴影向右移动15px，向上移动10px
      // this.ctx.shadowOffsetX = 20
      // this.ctx.shadowOffsetY = 20
      // this.ctx.shadowBlur = 15
      // this.ctx.fillStyle = 'rgba(110,110,110,.3)'

      this.ctx.fill()
    },
    /**
     * @description: 数据图层绘画
     * @param {*}
     * @return {*}
     * @author: 舒
     */
    paintDataInfo() {
      // let data = JSON.parse(JSON.stringify(this.dataInfo))
      // data.reverse()
      var index = -1
      this.dataInfo = this.dataInfo.map(item => {
        index++
        if (this.integration.color.length === index) {
          index = 0
        }
        return { ...item, color: this.integration.color[index] }
      })
      this.dataInfo = this.dataInfo.map((item, index) => {
        let drawingPoint = []
        this.ctx.fillStyle = item.color
        this.ctx.beginPath()
        let point1, point2, point3, point4, point5, point6
        if (index === 0) {
          ;[point1, point2, point3, point4, point5, point6] = [
            item.temporary.left[0],
            item.temporary.left[1],
            item.temporary.middle[1],
            item.temporary.right[1],
            item.temporary.right[0],
            item.temporary.middle[0]
          ]
        } else {
          ;[point1, point2, point3, point4, point5, point6] = [
            this.dataInfo[index - 1].temporary.left[1],
            item.temporary.left[1],
            item.temporary.middle[1],
            item.temporary.right[1],
            this.dataInfo[index - 1].temporary.right[1],
            this.dataInfo[index - 1].temporary.middle[1]
          ]
        }
        this.ctx.moveTo(...point1)
        this.ctx.lineTo(...point2)
        this.ctx.lineTo(...point3)
        this.ctx.lineTo(...point4)
        this.ctx.lineTo(...point5)
        this.ctx.lineTo(...point6)
        drawingPoint = [point1, point2, point3, point4, point5, point6]
        if (this.integration.infoStyle.stroke) {
          this.ctx.shadowOffsetX = 0
          this.ctx.shadowOffsetY = 0
          this.ctx.shadowBlur = 2
          this.ctx.shadowColor = this.integration.infoStyle.strokeColor
        }
        this.ctx.fill()
        return { ...item, drawingPoint }
      })
    },
    /**
     * @description: 绘画字体
     * 此方法请在 paintDataInfo() 执行后使用
     * @param {*}
     * @return {*}
     * @author: 舒
     */
    paintingText(lData) {
      this.ctx.shadowColor = 'rgba(90,90,90,0)'
      const color = this.integration.infoStyle.color ? this.integration.infoStyle.color : '#fff'
      const width = this.integration.infoStyle.width ? this.integration.infoStyle.width : 0
      const dotSize = this.integration.infoStyle.dotSize ? this.integration.infoStyle.dotSize : 4
      const offset = this.integration.infoStyle.offset ? this.integration.infoStyle.offset : [0, 0]
      let text = ''
      this.ctx.strokeStyle = color
      this.ctx.fillStyle = color
      this.dataInfo.forEach((item, index) => {
        if (item.drawingPoint) {
          let line = [
            [0, 0],
            [0, 0]
          ]
          this.ctx.font = `normal lighter ${
            this.integration.infoStyle.size ? this.integration.infoStyle.size : 14
          }px sans-serif `

          this.ctx.beginPath()
          if (lData && index + 1 === lData.l) {
            line = [
              [
                lData.obj.drawingPoint[2][0],
                (lData.obj.drawingPoint[2][1] - lData.obj.drawingPoint[5][1]) / 2 + lData.obj.drawingPoint[5][1]
              ],
              [
                lData.obj.drawingPoint[2][0] + lData.obj.drawingPoint[2][0] / 2 + width,
                (lData.obj.drawingPoint[2][1] - lData.obj.drawingPoint[5][1]) / 2 + lData.obj.drawingPoint[5][1]
              ]
            ]

            this.ctx.font = `normal lighter ${
              this.integration.infoStyle.size ? this.integration.infoStyle.size + 2 : 16
            }px sans-serif `
            text =
              this.integration.fontFormatter(item) !== 'default'
                ? this.integration.fontFormatter(item)
                : lData.obj.value + ' ---- ' + lData.obj.name
            this.ctx.setLineDash([0, 0])
            this.ctx.strokeText(
              text,
              line[1][0] + offset[0],
              line[1][1] + (this.integration.infoStyle.size ? this.integration.infoStyle.size + 2 : 14) / 3 + offset[1]
            )
          } else {
            line = [
              [
                item.drawingPoint[2][0],
                (item.drawingPoint[2][1] - item.drawingPoint[5][1]) / 2 + item.drawingPoint[5][1]
              ],
              [
                item.drawingPoint[2][0] + item.drawingPoint[2][0] / 2 + width,
                (item.drawingPoint[2][1] - item.drawingPoint[5][1]) / 2 + item.drawingPoint[5][1]
              ]
            ]
            text =
              this.integration.fontFormatter(item) !== 'default'
                ? this.integration.fontFormatter(item)
                : item.value + ' ----- ' + item.name
            this.ctx.setLineDash([0, 0])
            this.ctx.strokeText(
              text,
              line[1][0] + offset[0],
              line[1][1] + (this.integration.infoStyle.size ? this.integration.infoStyle.size + 2 : 16) / 3 + offset[1]
            )
          }
          this.ctx.setLineDash(this.integration.infoStyle.setLineDash)
          this.ctx.moveTo(...line[0])
          this.ctx.lineTo(...line[1])
          this.ctx.stroke()
          this.ctx.arc(...line[0], dotSize, 0, 360, false)
          this.ctx.fill() //画实心圆
        } else {
          throw '未找到 drawingPoint 属性'
        }
      })
    },
    /**
     * @description: 显示tooltip位置
     * @param {*} lData 当前层级
     * @param {*} coordinates 鼠标位置
     * @return {*}
     * @author: 舒
     */
    showTooltip(lData, coordinates) {
      let canvasWarpper = document.getElementById('canvas-warpper')
      let canvasTooltip = document.getElementById('canvas-tooltip')
      if (lData) {
        canvasTooltip.style.zIndex = this.integration.tooltip.z
        canvasTooltip.style.transition =
          ' opacity 0.2s cubic-bezier(0.23, 1, 0.32, 1) 0s, visibility 0.2s cubic-bezier(0.23, 1, 0.32, 1) 0s,transform 0.15s'
        let html = JSON.parse(JSON.stringify(this.tooltipDiv))
        if (this.integration.tooltip.formatter) {
          html = this.integration.tooltip.formatter(lData)
        } else {
          const searchVal = [
            ['$[title]$', lData.obj.title],
            ['$[name]$', lData.obj.name],
            ['$[val]$', lData.obj.value],
            ['$[color]$', lData.obj.color],
            ['$[fontSize]$', this.integration.tooltip.fontSize],
            ['$[backgroundColor]$', this.integration.tooltip.backgroundColor],
            ['$[fontColor]$', this.integration.tooltip.fontColor]
          ]
          searchVal.forEach(el => {
            html = html.replaceAll(...el)
          })
        }
        canvasTooltip.innerHTML = html
        canvasWarpper.style.cursor = 'pointer'
        canvasTooltip.style.visibility = 'visible'
        canvasTooltip.style.opacity = 1
        let [x, y] = coordinates
        x = x + 20
        y = y + 20
        // 画布高度
        // canvasHeight: 0,
        // 画布宽度
        // canvasWidth: 0,
        // 判断是否超出框架内容
        if (x + canvasTooltip.clientWidth > this.canvasWidth) {
          x = x - canvasTooltip.clientWidth - 40
        }
        if (y + canvasTooltip.clientHeight > this.canvasHeight) {
          y = y - canvasTooltip.clientHeight - 40
        }
        canvasTooltip.style.transform = `translate3d(${x}px, ${y}px, 0px)`
      } else {
        canvasWarpper.style.cursor = 'default'
        canvasTooltip.style.visibility = 'hidden'
        canvasTooltip.style.opacity = 0
      }
    },
    /**
     * @description: 高亮某一层级
     * @param {*} lData 层级数据
     * @return {*}
     * @author: 舒
     */
    highlightCurrentRegion(lData) {
      // const width = this.canvas.width;
      // this.canvas.width = width;

      this.ctx.clearRect(0, 0, this.canvas.width, this.canvas.height)
      if (!lData) {
        this.paintDataInfo()
        this.ctx.shadowColor = 'rgba(90,90,90,0)'
        this.paintingBody()
        this.paintingText()
        return
      }
      this.paintDataInfo()
      this.ctx.shadowColor = 'rgba(90,90,90,0)'
      this.paintingBody()
      this.ctx.fillStyle = lData.obj.color
      //  this.ctx.scale(1.05, 1.05)
      this.ctx.beginPath()
      this.ctx.moveTo(lData.obj.drawingPoint[0][0], lData.obj.drawingPoint[0][1])
      this.ctx.lineTo(lData.obj.drawingPoint[1][0], lData.obj.drawingPoint[1][1])
      this.ctx.lineTo(lData.obj.drawingPoint[2][0], lData.obj.drawingPoint[2][1])
      this.ctx.lineTo(lData.obj.drawingPoint[3][0], lData.obj.drawingPoint[3][1])
      this.ctx.lineTo(lData.obj.drawingPoint[4][0], lData.obj.drawingPoint[4][1])
      this.ctx.lineTo(lData.obj.drawingPoint[5][0], lData.obj.drawingPoint[5][1])
      this.ctx.shadowOffsetX = 0
      this.ctx.shadowOffsetY = 0
      this.ctx.shadowBlur = 10
      this.ctx.shadowColor = this.integration.infoStyle.highlightedColor
      this.ctx.fill()
      // 阴影绘制
      this.ctx.beginPath()
      this.ctx.moveTo(lData.obj.drawingPoint[0][0], lData.obj.drawingPoint[0][1])
      this.ctx.lineTo(lData.obj.drawingPoint[1][0], lData.obj.drawingPoint[1][1])
      this.ctx.lineTo(lData.obj.drawingPoint[2][0], lData.obj.drawingPoint[2][1])
      this.ctx.lineTo(lData.obj.drawingPoint[5][0], lData.obj.drawingPoint[5][1])
      this.ctx.fillStyle = 'rgba(120,120,120,.15)'
      this.ctx.fill()
      this.paintingText(lData)
    }
  }
}
</script>

<style lang="scss" scoped>
#canvas-warpper {
  position: relative;
  width: 100%;
  height: 100%;
  #canvas-tooltip {
    position: absolute;
    display: block;
    z-index: 9999999;
    transition: opacity 0.2s cubic-bezier(0.23, 1, 0.32, 1) 0s, visibility 0.2s cubic-bezier(0.23, 1, 0.32, 1) 0s;
    font: 14px / 21px sans-serif;
    top: 0px;
    left: 0px;
    transform: translate3d(0px, 0px, 0px);
    visibility: hidden;
    opacity: 0;
  }
}
</style>
