<template>
  <div>
    <h1>home</h1>
    <div id="main" class="box"></div>
  </div>
</template>
<script>
import echarts from "echarts";
import { mapGetters, mapActions } from "vuex";
export default {
  props: [],
  components: {},
  data() {
    return {};
  },
  computed: {
    ...mapGetters({
      list: "cate/list",
    }),
  },
  watch: {
    list: {
      handler() {
        // 基于准备好的dom，初始化echarts实例
        var myChart = echarts.init(document.getElementById("main"));

        // 指定图表的配置项和数据
        var option = {
          title: {
            text: "商品分类",
          },
          tooltip: {},
          legend: {
            data: ["数量"],
          },
          xAxis: {
            // map返回一个新的数组
            data: this.list.map((item) => item.catename),
          },
          yAxis: {},
          series: [
            {
              name: "数量",
              type: "line",
              data: this.list.map((item) => item.children.length),
            },
          ],
        };

        // 使用刚指定的配置项和数据显示图表。
        myChart.setOption(option);
      },
      deep: true,
    },
  },
  methods: {
    ...mapActions({
      reqListAction: "cate/reqListAction",
    }),
  },
  mounted() {
    this.reqListAction();
  },
};
</script>
<style scoped>
.box {
  width: 80%;
  height: 500px;
  border: 1px solid black;
  margin: 20px auto;
}
</style>