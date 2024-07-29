<template>
  <div>
    <!--地图视图-->
    <el-card class="box-card">
      <div class="echarts" ref="echarts"></div>
    </el-card>
    <!-- 弹窗 -->
    <el-dialog :title="dialogTitle" :visible.sync="dialogVisible" @close="closeDialog">
      <div>
        <p>著名建筑有：</p>
        <ul>
          <li v-for="item in dialogContent" :key="item.uri">
            <el-button type="text">{{ item.nameS }}</el-button>
          </li>
        </ul>
      </div>
      <span slot="footer" class="dialog-footer">
        <el-button @click="closeDialog">关闭</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
import axios from 'axios';
import * as echarts from 'echarts';
import 'echarts-gl';

export default {
  data() {
    return {
      dialogVisible: false,
      dialogTitle: '',
      dialogContent: []
    };
  },
  mounted() {
    this.initEcharts();
  },
  methods: {
    initEcharts() {
      // 获取数据
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
      /*获取地图数据*/
      myChart.showLoading();
      var mapFeatures = echarts.getMap(nameMap).geoJson.features;
      myChart.hideLoading();
      mapFeatures.forEach(function (v, index) {
        // 地区名称
        mapData.push({
          name: v.properties.name,
          value: Math.random() * 100
        });
      });
      var optionMap = {
        backgroundColor: '#cfdfef',
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
            color: "rgba(1, 16, 31, 0)",
            opacity: 0,
            borderColor: "rgba(1, 16, 31, 0)",
            borderWidth: 0
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
            }
          },
          light: {
            main: {
              color: '#20729b',
              intensity: 1,
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
            color: "rgba(1, 16, 31, .2)",
            opacity: 1,
            borderColor: "rgba(1,31,13,0.6)",
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
              color: "rgb(194,217,239)",
              opacity: 1
            }
          },
          zlevel: 1
        }]
      };
      myChart.setOption(optionMap);

      // 修改点击事件监听器中的请求
      myChart.on('click', params => {
        if (params.componentType === 'geo3D' || params.componentType === 'series') {
          axios.get(`http://localhost:8090/api/architecture?freetext=${params.name}`)
            .then(resp => {
              if (resp.data.result === "0") {
                this.dialogTitle = params.name;
                this.dialogContent = resp.data.data;
                this.dialogVisible = true;
              }
            })
            .catch(error => {
              console.error(error);
            });
        }
      });

    },
    closeDialog() {
      this.dialogVisible = false;
    }
  }
};
</script>

<style scoped>
.echarts {
  width: 100%;
  height: 100%;
  background: #0d1551;
  position: fixed;
}

.dialog-overlay {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background: rgba(0, 0, 0, 0.5);
  display: flex;
  justify-content: center;
  align-items: center;
}

.dialog-content {
  background: #fff;
  padding: 20px;
  border-radius: 5px;
  text-align: center;
}
</style>
