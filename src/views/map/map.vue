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

    <!-- 留声机图标 -->
    <Gramophone @show-video="showVideo" />

    <!-- 电视机样式的视频弹窗 -->
    <TVFrame :visible.sync="videoVisible" />

    <!-- 建筑弹窗 -->
    <ArchitectureDialog
      :dialogTitle="dialogTitle"
      :dialogVisible.sync="dialogVisible"
      :dialogContent="dialogContent"
      :eventList="eventList"
      @show-detail="showDetail"
      @show-event-detail="showEventDetail"
    />

    <!-- 建筑详细信息弹窗 -->
    <ArchitectureDetailDialog
      :detailDialogTitle="detailDialogTitle"
      :detailDialogVisible.sync="detailDialogVisible"
      :detailContent="detailContent"
      :currentItem="currentItem"
    />

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
import ArchitectureDialog from './ArchitectureDialog.vue';
import ArchitectureDetailDialog from './ArchitectureDetailDialog.vue';
import EventDetailDialog from './EventDetailDialog.vue';
import Gramophone from './Gramophone.vue';
import TVFrame from './TVFrame.vue';

export default {
  components: {
    YearSlider,
    ArchitectureDialog,
    ArchitectureDetailDialog,
    EventDetailDialog,
    Gramophone,
    TVFrame
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
      eventDetailContent: null,
      videoVisible: false
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
    showVideo() {
      this.videoVisible = true;
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
</style>
