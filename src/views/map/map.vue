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
    <!-- 年份进度条 -->
    <YearSlider v-model="selectedYear" />

    <!-- 建筑弹窗 -->
    <el-dialog :title="dialogTitle" :visible.sync="dialogVisible" @close="closeDialog">
      <div>
        <p>著名建筑有：</p>
        <ul>
          <li v-for="item in dialogContent" :key="item.uri">
            <el-button class="item-button" type="text" @click="showDetail(item.uri, item)">{{ item.nameS }}</el-button>
          </li>
        </ul>
        <p>相关事件：</p>
        <ul>
          <li v-for="event in eventList" :key="event.uri">
            <el-button
              class="item-button"
              type="text"
              @click="showEventDetail(event.uri)"
              style="position: relative; z-index: 10;"
            >
              {{ event.title }} ({{ event.begin }} - {{ event.end }})
            </el-button>
          </li>
        </ul>
      </div>
      <span slot="footer" class="dialog-footer">
        <el-button @click="closeDialog">关闭</el-button>
      </span>
    </el-dialog>

    <!-- 建筑详细信息弹窗 -->
    <el-dialog :title="detailDialogTitle" :visible.sync="detailDialogVisible" @close="closeDetailDialog">
      <div v-if="detailContent">
        <div class="detail-item">
          <span class="label">建筑名称:</span>
          <span class="value">{{ detailContent.nameS }}</span>
        </div>
        <div class="detail-item">
          <span class="label">地址:</span>
          <span class="value">{{ detailContent.address }}</span>
        </div>
        <div class="detail-item">
          <span class="label">类型:</span>
          <span class="value">{{ detailContent.type }}</span>
        </div>
        <div class="detail-item">
          <span class="label">描述:</span>
          <span class="value">{{ currentItem.des }}</span>
        </div>
        <div class="detail-item">
          <span class="label">地区:</span>
          <span class="value">{{ currentItem.placeValue }}</span>
        </div>
      </div>
      <span slot="footer" class="dialog-footer">
        <el-button @click="closeDetailDialog">关闭</el-button>
      </span>
    </el-dialog>

    <!-- 事件详细信息弹窗 -->
    <EventDetailDialog
      :visible.sync="eventDetailDialogVisible"
      :title="eventDetailDialogTitle"
      :content="eventDetailContent"
    />
  </div>
</template>

<script>
import axios from 'axios';
import * as echarts from 'echarts';
import 'echarts-gl';
import YearSlider from './YearSlider.vue';
import EventDetailDialog from './EventDetailDialog.vue';

export default {
  components: {
    YearSlider,
    EventDetailDialog
  },
  data() {
    return {
      selectedYear: 1800,
      dialogVisible: false,
      dialogTitle: '',
      dialogContent: [],
      eventList: [],
      detailDialogVisible: false,
      detailDialogTitle: '',
      detailContent: null,
      currentItem: null,
      currentRegionName: '',
      eventDetailDialogVisible: false,
      eventDetailDialogTitle: '',
      eventDetailContent: null
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
          axios.get(`http://localhost:8090/api/architecture?freetext=${params.name}`)
            .then(resp => {
              if (resp.data.result === "0") {
                this.dialogTitle = params.name;
                this.dialogContent = resp.data.data;
                this.dialogVisible = true;
                this.fetchEvents(params.name);
              }
            })
            .catch(error => {
              console.error(error);
            });
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
    fetchEvents(regionName) {
      this.eventList = [];
      axios.get(`http://localhost:8090/api/eventListByText?eventFreeText=${regionName}`)
        .then(resp => {
          if (resp.data.result === "0") {
            this.eventList = resp.data.data;
          }
        })
        .catch(error => {
          console.error(error);
        });

      axios.get(`http://localhost:8090/api/eventListByDate?eventDate=${this.selectedYear}`)
        .then(resp => {
          if (resp.data.result === "0") {
            this.eventList = this.eventList.concat(resp.data.data);
          }
        })
        .catch(error => {
          console.error(error);
        });
    },
    showEventDetail(uri) {
      axios.get(`http://localhost:8090/api/eventDetail?uri=${uri}`)
        .then(resp => {
          if (resp.data.result === "0") {
            this.eventDetailDialogTitle = "事件详细信息";
            this.eventDetailContent = resp.data.data[0];
            this.eventDetailDialogVisible = true;
          }
        })
        .catch(error => {
          console.error(error);
        });
    },
    showDetail(uri, item) {
      axios.get(`http://localhost:8090/architectureDetail?uri=${uri}`)
        .then(resp => {
          if (resp.data.result === "0") {
            this.detailDialogTitle = "详细信息";
            this.detailContent = resp.data.data[0];
            this.detailDialogVisible = true;
            this.currentItem = item;
          }
        })
        .catch(error => {
          console.error(error);
        });
    },
    closeDialog() {
      this.dialogVisible = false;
    },
    closeDetailDialog() {
      this.detailDialogVisible = false;
    }
  }
};
</script>

<style scoped>
.echarts {
  width: 100%;
  height: 100%;
  background: url('../../assets/img/shangtu-bg.png') no-repeat center center;
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

.item-button {
  border: 1px solid #007bff;
  color: #007bff;
  padding: 5px 10px;
  margin: 5px 0;
  display: inline-block;
  cursor: pointer;
  position: relative;
  z-index: 10; /* 确保按钮位于上层 */
}

.item-button:hover {
  background-color: #007bff;
  color: #fff;
}

.detail-item {
  margin: 10px 0;
  padding: 10px;
  border: 1px solid #ddd;
  border-radius: 5px;
  background-color: #f9f9f9;
}

.label {
  font-weight: bold;
  color: #333;
}

.value {
  color: #555;
}
</style>
