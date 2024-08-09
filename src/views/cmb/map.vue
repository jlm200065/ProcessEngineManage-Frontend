<template>
  <div>
    <!-- 显示当前鼠标放置的地区名称 -->
    <div class="region-name" v-if="currentRegionName">
      {{ currentRegionName }}
    </div>
    <!-- 地图视图 -->
    <el-card class="box-card">
      <div class="echarts" ref="echarts"></div>
    </el-card>

    <!-- AI助手图标 -->
    <AIAssistant @show-dialog="showAIDialog" />

    <!-- AI助手对话框 -->
    <AIAssistantDialog :visible.sync="aiDialogVisible" ref="aiDialog" />
  </div>
</template>

<script>
import axios from 'axios';
import * as echarts from 'echarts';
import 'echarts-gl';
import AIAssistant from './AIAssistant.vue';
import AIAssistantDialog from './AIAssistantDialog.vue';

export default {
  components: {
    AIAssistant,
    AIAssistantDialog,
  },
  data() {
    return {
      currentRegionName: '',
      aiDialogVisible: false, // AI Assistant dialog visibility
    };
  },
  mounted() {
    this.initEcharts();
  },
  methods: {
    initEcharts() {
      axios.get('https://jlm-1321383016.cos.ap-shanghai.myqcloud.com/map/data.json')
        .then(resp => {
          this.renderEcharts(resp.data);
        })
        .catch(error => {
          console.error(error);
        });
    },
    renderEcharts(data) {
      var chartDom = this.$refs.echarts;
      var myChart = echarts.init(chartDom);
      var nameMap = '地图数据';
      var mapData = [];
      echarts.registerMap(nameMap, data);
      myChart.showLoading();
      var mapFeatures = echarts.getMap(nameMap).geoJson.features;
      myChart.hideLoading();
      mapFeatures.forEach(function (v, index) {
        mapData.push({
          name: v.properties.name,
          value: Math.random() * 100
        });
      });
      var optionMap = {
        backgroundColor: 'rgba(0,0,0,0)',
        toolbox: {
          show: true,
          feature: {
            saveAsImage: {
              show: true,
              excludeComponents: ['toolbox'],
              pixelRatio: 1
            }
          }
        },
        geo3D: {
          map: nameMap,
          itemStyle: {
            color: "rgba(0, 101, 163, 0.7)",
            opacity: 0.8,
            borderColor: "rgba(255, 255, 255, 1)",
            borderWidth: 1
          },
          roam: true,
          label: {
            show: false,
            textStyle: {
              color: '#fff',
              fontSize: 16,
              opacity: 1,
              backgroundColor: 'rgba(0,0,0,0)'
            }
          },
          emphasis: {
            label: {
              show: true,
              textStyle: {
                color: '#ffeb3b',
                fontSize: 16,
                backgroundColor: 'rgba(0,23,11,0)'
              }
            },
            itemStyle: {
              color: "#ff7f50",
              opacity: 1
            }
          },
          light: {
            main: {
              color: '#fff',
              intensity: 1.2,
              alpha: 55,
              beta: 10
            },
            ambient: {
              intensity: 0.3
            }
          }
        },
        series: [{
          type: 'map3D',
          map: nameMap,
          itemStyle: {
            color: "rgba(0, 101, 163, 0.7)",
            opacity: 0.8,
            borderColor: "rgba(255, 255, 255, 1)",
            borderWidth: 1
          },
          label: {
            show: false
          },
          emphasis: {
            label: {
              show: true
            },
            itemStyle: {
              color: "#ff7f50",
              opacity: 1
            }
          },
          zlevel: 1
        }]
      };
      myChart.setOption(optionMap);

      myChart.on('click', params => {
        if (params.componentType === 'geo3D' || params.componentType === 'series') {
          this.sendMessageAndShowDialog(params.name);
        }
      });

      myChart.on('mousemove', params => {
        if (params.componentType === 'geo3D' || params.componentType === 'series') {
          this.currentRegionName = params.name;
        } else {
          this.currentRegionName = '';
        }
      });
    },
    sendMessageAndShowDialog(regionName) {
      this.aiDialogVisible = true;
      this.$refs.aiDialog.sendMessage(`上海"${regionName}"附近有招商银行网点吗？`);
    },
    showAIDialog() {
      this.aiDialogVisible = true;
    },
  }
};
</script>

<style scoped>
.echarts {
  width: 100%;
  height: 100%;
  background: url('../../assets/img/招商银行背景图.jpg') no-repeat center center;
  background-size: cover;
  position: fixed;
}

.echarts::before {
  content: '';
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background-color: rgba(0, 0, 0, 0.6); /* 调整透明度 */
  pointer-events: none;
}

.region-name {
  position: fixed;
  top: 10px;
  left: 10px;
  background: rgba(0, 0, 0, 0.5);
  color: #fff;
  padding: 20px 40px; /* 调整 padding 使其更大 */
  border-radius: 5px;
  z-index: 1000;
  font-size: 30px; /* 调整字体大小 */
}
</style>
