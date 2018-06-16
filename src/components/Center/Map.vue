<template>
  <div>
    <div ref="chart" class="chart" :class="{mobile:!isPC}">
    </div>
    <div v-if="!isPC" class="tip-mobile">
      <span style="padding-left:5px;">{{nextTime}}</span>
      <!--下次更新: -->
      <span style="padding-right:5px;">技术支持：中国印钞造币总公司</span>
    </div>
  </div>
</template>

<script>
import echarts from "echarts";
import chinaJson from "./js/china.json";
// 以渐变来变化颜色
// import mapChart from './js/mapOption.js';

// 以名次变化地图颜色
import mapChart from "./js/mapOptionStep.js";
import convertProv from "./js/provData.js";

export default {
  data() {
    return {
      cityData: [],
      provData: 0
    };
  },
  computed: {
    nextTime() {
      return this.$store.state.nextTime;
    },
    chart() {
      return echarts.init(this.$refs.chart);
    },
    peopleCount: {
      get() {
        return this.$store.state.peopleCount;
      },
      set(val) {
        this.$store.commit("setPeopleCount", val);
      }
    },
    needRefresh: {
      get() {
        return this.$store.state.needRefresh;
      },
      set(val) {
        this.$store.commit("refresh", val);
      }
    },
    isPC() {
      return this.$store.state.isPC;
    }
  },
  watch: {
    needRefresh(val) {
      if (val) {
        this.getData();
      }
    },
    provData(data) {
      let sum = 0;
      data.forEach(item => {
        sum += parseInt(item.value);
      });
      this.peopleCount = sum;
    }
  },
  methods: {
    resizeChart() {
      this.chart.resize();
    },
    initEvent() {
      window.onresize = () => {
        this.resizeChart();
      };
      this.chart.on("click", params => {
        if (params.seriesIndex == 0) {
          let province = params.name;
          if (province == "香港" || province == "澳门") {
            province += "特别行政区";
          }
          this.$store.commit("setCurProvince", province);
        }
      });
    },
    getData() {
      let url =
        "http://cbpc540.applinzi.com/?s=/addon/Api/Api/getVoteCountByProv";
      this.$http
        .jsonp(url)
        .then(res => {
          this.provData = res.data
            .map(item => {
              item.name = convertProv(item.name);
              return item;
            })
            .filter(item => item.name);

          // console.log(option);

          // let passednum = 0;
          // this.provData.map(item => {
          //   passednum += parseInt(item.passed);
          // })
          // this.$store.commit('setPassed', passednum);

          if (!this.isPC) {
            var option = mapChart.refreshMain(this.provData);
            this.chart.setOption(option);
            this.needRefresh = false;
            return;
          }

          url = this.$baseurl + "?s=/addon/Api/Api/getVoteCountByCity";
          this.$http.jsonp(url).then(res => {
            let provData = res.data.map(item => {
              item.name = item.name.length == 0 ? "(未知)" : item.name;
              return item;
            });
            this.$store.commit("setTop20Cities", provData);
            let option = mapChart.refreshScatter(provData, this.provData);
            this.chart.setOption(option);
            this.needRefresh = false;
          });
        })
        .catch(e => console.log(e));
    },
    refreshChart() {
      this.chart.setOption(mapChart.init());
      this.getData();

      // 20秒更新一次数据
      // setInterval(() => {
      //   this.getData();
      // }, 60000);
    },
    init() {
      echarts.registerMap("china", chinaJson);
      this.refreshChart();
      this.initEvent();
    }
  },
  mounted() {
    this.init();
  }
};
</script>

<style scoped lang="less">
.chart {
  width: 100%;
  min-height: 100vh;
}

.mobile {
  min-height: 60vh;
  margin-top: -20%;
  margin-bottom: 40%;
}

.tip-mobile {
  color: #fff;
  font-size: 10pt;
  position: absolute;
  right: 10px;
  top: 53%;
  width: 100%;
  display: flex;
  justify-content: space-between;
  left: 0;
}
</style>
