<template>
  <div id="app">
    <div class="test">
      <pyramid :options="ops" :data="datas" />
    </div>
  </div>
</template>

<script>
import Pyramid from "@/components/Pyramid";
export default {
  name: "App",
  components: {
    Pyramid,
  },
  data() {
    return {
        ops: {
      title: "企业培育",
      // 主体离边框距离
      distance: [75, 10],
      // 主体偏移值 (x,y)
      offset: [-70, 2],
      // 排序(max , min)优先
      sort: "max",
      // 颜色
      color: [
        "rgba(104, 236, 234, 1.000)",
        "rgba(84, 155, 180, 1.000)",
        "rgba(59, 94, 185, 1.000)",
        "rgba(31, 63, 144, 1.000)",
        "rgba(13, 34, 103, 1.000)",
      ],
      // 格式化字体输出
      fontFormatter: (val) => {
        return ` ${val.name}   ${val.value}%`;
      },
      tooltip: {
        formatter: (val) => {
          return `<div  style="margin: 0px 0 0; color:#000; line-height: 1;border-color: #fff ;background-color: #fff;;
        border-width: 1px;border-radius: 4px;padding: 10px;pointer-events: none;box-shadow: rgb(0 0 0 / 20%) 1px 2px 10px;border-style: solid;white-space: nowrap;">
            <div style="margin: 0px 0 0; line-height: 1">
              <div style="font-size: ${14}px; font-weight: 400; line-height: 1">${
            val.obj.title
          } </div>
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
                        background-color:  ${val.obj.color};
                      "
                    ></span>
                    <span style="font-size: ${14}px; color: $[fontColor]$; font-weight: 400; margin-left: 2px"> ${
            val.obj.name
          }</span>
                    <span style="float: right; margin-left: 20px; font-size: ${14}px; color: $[fontColor]$; font-weight: 900"> ${
            val.obj.value
          } %</span>
                    <div style="clear: both"></div>
                  </div>
                  <div style="clear: both"></div>
                </div>
                <div style="clear: both"></div>
              </div>
              <div style="clear: both"></div>
            </div>
            <div style="clear: both"></div>
          </div>`;
        },
      },
      // 样式
      infoStyle: {
        stroke: true,
        color: "rgba(189, 192, 214, 1.000)",
        setLineDash: [4, 4], // 虚线值
        size: 12, // 字体大小
        width: 120, // 设置多少 就会在基础上加上设置的值
        dotSize: 2, //点大小
      },
    },
    datas: [
      { name: "示例1", value: 5 },
      { name: "示例2", value: 10 },
      { name: "示例3", value: 20 },
      { name: "示例4", value: 30 },
      { name: "示例5", value: 35 },
    ],
    }
  },
  mounted() {},
  methods: {
  
  },
};
</script>

<style lang="scss">
#app,
body {
  background-color: rgb(72, 72, 72);
  height: 100%;
  .test {
    width: 800px;
    height: 600px;
  }
}
</style>
