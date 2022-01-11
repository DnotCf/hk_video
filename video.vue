<template>
  <div class="realtime-monitor">
    <!-- 用于装百度地图的容器 -->
    <Spin fix v-if="loading" class="global-lodaing">
      <Icon type="ios-loading" size="18" class="spin-icon-load"></Icon>
      <div>加载中...</div>
    </Spin>
    <div class="map-content realtime">
      <div class="map" id="map" v-show="!isTripDim"></div>
      <div class="map" id="cesiumMap" v-show="isTripDim"></div>
      <!-- 右上角功能区域 --begin -->
      <div class="functional">
        <div class="functional-content" v-show="showFunctional">
          <div
            class="functional-item waring-message-functional"
            @click="toogleWarnMessage"
            title="预警"
          >
            <Button type="text" ghost>
              <Badge
                :count="warningCountNumber"
                :class="showWarnMessageList ? 'active' : ''"
              >
                <i class="iconfont icon-yujing icon-blue"></i>
                <!-- <img src="@/assets/images/warn-home.png" alt="" /> -->
              </Badge>
            </Button>
          </div>
          <!-- <div
            class="functional-item"
            @click="toggleScreen"
            :class="showScreen ? 'active' : ''"
            title="筛选"
          >
            <Button type="text" ghost>
              <ColorIcons
                class="color-icon right"
                type="icon-guolv"
                v-show="!showScreen"
              ></ColorIcons>
              <Icon type=" iconfont icon-guolv" v-show="showScreen"></Icon>
            </Button>
          </div> -->
          <!-- <div class="functional-item" @click="toggleLayers" title="图层">
            <Button type="text" ghost>
              <i class="iconfont icontuceng icon-blue icon-blue-14"></i>
            </Button>
          </div> -->
          <!-- 2D/3D -->
          <div
            class="functional-item threed"
            :class="isTripDim ? 'active' : ''"
            :title="get3DButtonTitle"
          >
            <Button type="text" ghost @click.stop="toggleMapDim">
              <!-- <ColorIcons
                class="color-icon right"
                type="icon3D"
                :size="1.2"

              ></ColorIcons> -->
              <i
                class="iconfont icon-3D icon-blue icon-blue-14"
                v-show="!isTripDim"
              ></i>
              <i
                class="iconfont icon-2D icon-blue icon-blue-14"
                v-show="isTripDim"
              ></i>

              <!-- <ColorIcons
                class="color-icon right"
                type="icon2D"
                :size="1.2"
                v-show="isTripDim"
              ></ColorIcons> -->
              <!-- <img src="@/assets/images/3D.png" alt="" /> -->
            </Button>
          </div>
        </div>
      </div>
      <div
        class="threshold-value-functional"
        @click.stop="handleOpenModal(1)"
        title="设置阈值"
      >
        <div class="btn-box"></div>
      </div>
      <div
        class="artificial-warn-functional"
        @click.stop="handleOpenModal(2)"
        title="人工预警"
      >
        <div class="btn-box"></div>
      </div>
      <div class="message-list-content" v-show="showWarnMessageList">
        <div class="message-content i-standard-scrollbar">
          <template v-if="warningList.length > 0">
            <div
              class="card-item"
              v-for="(item, index) in warningList"
              :key="index"
              @click.stop="warningMessageItemClick(item, index)"
            >
              <Card :class="getWarnCardLevelClass(item.warningGrade)">
                <div class="item time">
                  <i class="iconfont icon-shijian"></i>
                  <span>{{ item.warningMessage.warnTime }}</span>
                </div>
                <div class="item project" :title="item.projectName">
                  <i class="iconfont icon-renwuweizhi"></i>
                  <span>{{ item.projectName }}</span>
                </div>
                <div class="item descrip">
                  <!-- <i class="iconfont iconshebeixinxi"></i> -->
                  <span>{{ item.warningMessage.warnContent }}</span>
                </div>
              </Card>
            </div>
          </template>
          <div class="nodata" v-else>
            <i class="iconfont icon-zanwushuju"></i>
            <div>暂无数据</div>
          </div>
        </div>
      </div>

      <div class="screen-box type-check" v-show="showScreen">
        <div class="screen-content">
          <Checkbox
            v-model="checkAll"
            :indeterminate="indeterminate"
            @click.prevent.native="handleCheckAll"
            v-show="monitorMethodList.length > 0"
            >全选</Checkbox
          >
          <CheckboxGroup v-model="checkAllGroup" @on-change="checkGroupChange">
            <Checkbox
              v-for="(item, index) in monitorMethodList"
              :label="item.value"
              :key="index"
              >{{ item.label }}</Checkbox
            >
          </CheckboxGroup>
        </div>
      </div>

      <div class="layers-box screen-box type-check" v-show="showLayer">
        <!-- <div class="screen-box-filter">
          <Button type="primary" class="reset-btn" @click.stop="resetTypeFilter">重置</Button>
        </div> -->
        <div class="screen-content">
          <CheckboxGroup v-model="checkAllLayer" @on-change="checkLayerChange">
            <Checkbox label="geo">地质图</Checkbox>
            <Checkbox label="hyp">水文图</Checkbox>
          </CheckboxGroup>
        </div>
      </div>

      <div class="warn-card" :class="getWarnCardClass">
        <div
          class="real-time-data-box"
          :style="{
            height: isRealOpen ? '47%' : '37px',
            width: isRealOpen ? '100%' : '50%',
          }"
        >
          <div class="title-box">
            <div class="title">
              <img src="@/assets/images/home-title.png" alt="" />
              <span>实时数据</span>
            </div>
            <div class="isOpen-box" @click="handleRealOpen">
              <Icon type="ios-arrow-forward" v-if="isRealOpen" />
              <Icon type="ios-arrow-back" v-else />
            </div>
          </div>
          <div class="list-box" v-if="isRealOpen">
            <div class="list-item list-header">
              <div class="point height-50">监测点位</div>
              <div class="item-common-3 height-50">行政区</div>
              <div class="item-common-2 height-50">状态</div>
              <div class="item-common item-common-2">
                <span>水位</span>
                <span>m</span>
              </div>
              <div class="item-common item-common-3">
                <span>流量</span>
                <span>m³/s</span>
              </div>
              <div class="item-common item-common-3">
                <span>降雨量</span>
                <span>mm</span>
              </div>
              <div class="item-common item-common-2">
                <span>水温</span>
                <span>℃</span>
              </div>
              <div class="item-common item-common-2">
                <span>pH</span>
                <span>—</span>
              </div>
              <div class="item-common item-common-3">
                <span>电导率</span>
                <span>μS/cm</span>
              </div>
              <div class="item-common item-common-2">
                <span>浊度</span>
                <span>NTU</span>
              </div>
              <div class="item-common item-common-3">
                <span>溶解氧</span>
                <span>mg/L</span>
              </div>
              <div class="item-common item-common-2">
                <span>氨氮</span>
                <span>mg/L</span>
              </div>
              <div class="item-common oxygen">
                <span>化学需氧量</span>
                <span>mg/L</span>
              </div>
              <div class="point height-50">水质类别</div>
            </div>
            <div class="list-content">
              <vue-seamless-scroll
                :data="monitoringPointList"
                class="seamless-warp"
                :class-option="defaultOption"
              >
                <ul class="ul-scoll">
                  <li v-if="!monitoringPointList.length" class="nodata">
                    暂无数据
                  </li>
                  <li
                    class="list-item height-38"
                    :class="temp.runStatus == 0 ? 'maintain' : ''"
                    v-for="(temp, index) in monitoringPointList"
                    :key="index"
                  >
                    <div class="point text-ellipsis" :title="temp.projectName">
                      {{ temp.projectName }}
                    </div>
                    <div class="item-common-3 text-ellipsis" :title="temp.area">
                      {{ temp.area }}
                    </div>
                    <div
                      class="item-common-2 text-ellipsis"
                      :class="temp.runStatus == 0 ? 'yellow' : ''"
                    >
                      {{ temp.runStatus == 0 ? "维护" : "正常" }}
                    </div>
                    <div
                      class="item-common-2 text-ellipsis"
                      :title="temp.z | keep"
                    >
                      {{ temp.z | keep }}
                    </div>
                    <div
                      class="item-common-3 text-ellipsis"
                      :title="temp.q | keep"
                    >
                      {{ temp.q | keep }}
                    </div>
                    <div
                      class="item-common-3 text-ellipsis"
                      :title="temp.drp | keep"
                    >
                      {{ temp.drp | keep }}
                    </div>
                    <div
                      class="item-common-2 text-ellipsis"
                      :title="temp.wt | keep"
                    >
                      {{ temp.wt | keep }}
                    </div>
                    <div
                      class="item-common-2 text-ellipsis"
                      :title="temp.ph | keep"
                    >
                      {{ temp.ph | keep }}
                    </div>
                    <div
                      class="item-common-3 text-ellipsis"
                      :title="temp.cond | keep"
                    >
                      {{ temp.cond | keep }}
                    </div>
                    <div
                      class="item-common-2 text-ellipsis"
                      :title="temp.turb | keep"
                    >
                      {{ temp.turb | keep }}
                    </div>
                    <div
                      class="item-common-3 text-ellipsis"
                      :title="temp.dox | keep"
                    >
                      {{ temp.dox | keep }}
                    </div>
                    <div
                      class="item-common-3 text-ellipsis"
                      :title="temp.nh3N | keep"
                    >
                      {{ temp.nh3N | keep }}
                    </div>
                    <div
                      class="oxygen text-ellipsis"
                      :title="temp.codcr | keep"
                    >
                      {{ temp.codcr | keep }}
                    </div>
                    <div class="point text-ellipsis" :title="temp.waterType">
                      {{ temp.waterType || '-' }}
                    </div>
                  </li>
                </ul></vue-seamless-scroll
              >
            </div>
          </div>
        </div>
        <div
          class="monitor-point-box"
          :style="{ height: isPointOpen ? `calc(53% - 5px)` : '40px' }"
        >
          <div class="title-box">
            <div class="title">
              <img src="@/assets/images/home-title.png" alt="" />
              <span>监测点位列表</span>
            </div>
            <div class="isOpen-box" @click="handlePointOpen">
              <Icon type="ios-arrow-forward" v-if="isPointOpen" />
              <Icon type="ios-arrow-back" v-else />
            </div>
          </div>
          <div class="tree-box" v-if="isPointOpen">
            <div class="search-type-input">
              <div class="input-search">
                <Input
                  v-model="searchValue"
                  placeholder="输入监测点位名称"
                  class="search-input"
                  clearable
                />
              </div>
              <div class="search-button">
                <Button
                  type="primary"
                  icon="ios-search"
                  @click.stop="filterNode"
                ></Button>
              </div>
            </div>
            <div class="sensor-content i-standard-scrollbar">
              <el-tree
                :data="projectList"
                node-key="label"
                default-expand-all
                :expand-on-click-node="false"
                :filter-node-method="filterNode"
                :props="defaultProps"
                ref="tree"
                :class="active ? 'active' : ''"
                :indent="8"
                empty-text="暂无数据"
                highlight-current
                @node-click="handlPointChange"
                :render-content="renderContent"
                check-on-click-node
              >
              </el-tree>
            </div>
          </div>
        </div>
      </div>
      <!-- 右上角功能区域 --end -->

      <!-- 点击具体项目以后弹出的项目信息框 begin -->
      <div class="NameModal" v-show="showPointModal">
        <div class="project-box">
          <div class="project-header">
            <div class="project-title">{{ monitorPointInfo.projectName }}</div>
            <div class="project-time">{{ monitorPointInfo.collectDate }}</div>
          </div>
          <div class="project-content">
            <div class="project-row">
              <div class="project-item-left">
                <span class="title">水位</span>
                <span class="price">{{ monitorPointInfo.z | keep }} m</span>
              </div>
              <div class="project-item-right">
                <span class="title">流量</span>
                <span class="price">{{ monitorPointInfo.q | keep }} m³/s</span>
              </div>
            </div>

            <div class="project-row">
              <div class="project-item-left">
                <span class="title">降雨量</span>
                <span class="price">{{ monitorPointInfo.drp | keep }} mm</span>
              </div>
              <div class="project-item-right">
                <span class="title">水温</span>
                <span class="price">{{ monitorPointInfo.wt | keep }} ℃</span>
              </div>
            </div>
            <div class="project-row">
              <div class="project-item-left">
                <span class="title">pH</span>
                <span class="price">{{ monitorPointInfo.ph | keep }} </span>
              </div>
              <div class="project-item-right">
                <span class="title">电导率</span>
                <span class="price"
                  >{{ monitorPointInfo.cond | keep }} μS/cm</span
                >
              </div>
            </div>
            <div class="project-row">
              <div class="project-item-left">
                <span class="title">浊度</span>
                <span class="price"
                  >{{ monitorPointInfo.turb | keep }} NTU</span
                >
              </div>
              <div class="project-item-right">
                <span class="title">溶解氧</span>
                <span class="price"
                  >{{ monitorPointInfo.dox | keep }} mg/L</span
                >
              </div>
            </div>
            <div class="project-row">
              <div class="project-item-left">
                <span class="title">氨氮</span>
                <span class="price"
                  >{{ monitorPointInfo.nh3N | keep }} mg/L</span
                >
              </div>
              <div class="project-item-right">
                <span class="title">化学需氧量</span>
                <span class="price"
                  >{{ monitorPointInfo.codcr | keep }} mg/L</span
                >
              </div>
            </div>
          </div>
        </div>
        <div class="close-area" @click="closePointModal">
          <img src="@/assets/images/close-red.png" class="close-btn" alt="" />
        </div>
      </div>
      <!-- 点击具体项目以后弹出的项目信息框 end -->
      <!-- 点击具体项目以后弹出的曲线图 begin -->
      <div class="diagramModal" v-show="showPointModal">
        <div class="tabs-box">
          <span
            class="tabs-item"
            v-for="(item, index) in curPointList"
            :key="index"
            :class="index == tabActive ? 'tab-active' : ''"
            @click="handleChangeTab(item, index)"
            >{{ item.contentName }}</span
          >
        </div>
        <div class="lineChart">
          <MonitorCurveV3
            ref="warnCurveRef"
            style="height: 100%"
          ></MonitorCurveV3>
        </div>
        <div class="close-area" @click="closePointModal">
          <img src="@/assets/images/close-red2.png" class="close-btn" alt="" />
        </div>
      </div>
      <!-- 点击具体项目以后弹出的曲线图 end -->
    </div>

    <div class="resize-modal-content">
      <template v-for="(modal, index) in chartsModalList">
        <ResizeModal
          v-model="modal.showModal"
          footer-hide
          draggable
          scrollable
          maxable
          minable
          class="geo-modal realtime-chart-modal"
          :key="modal.sensorId"
          @on-cancel="modalCancel(index)"
          :minWidth="700"
          :minHeight="320"
        >
          <div slot="header" class="modal-header">
            <div class="header-top">
              <span :title="modal.sensorName" class="sensor-name">{{
                modal.sensorName
              }}</span>
              <ColorIcons
                class="color-icon"
                type="icon-chuanganqi-lixian"
                v-if="modal.runStatus !== '1' && modal.runStatus !== 1"
              ></ColorIcons>
              <ColorIcons
                class="color-icon"
                type="icon-chuanganqi-zaixian"
                v-else
              ></ColorIcons>
            </div>
          </div>
          <div class="modal-content">
            <div class="more-btn">
              <Button type="info" @click="openSensorCruve(modal)"
                >查看更多</Button
              >
            </div>
            <div class="time-list">
              <Button
                v-for="(timetag, timeIndex) in modal.timeList"
                :key="timeIndex"
                @click.stop="chartTimeChange(index, timetag)"
                :class="timetag.isActive ? 'active' : ''"
              >
                {{ timetag.timeTitle }}
              </Button>
            </div>
            <div class="chart-content">
              <div class="chart-box" v-if="judeChartHasData(modal.chartOption)">
                <v-chart
                  ref="vueChartDom"
                  :options="modal.chartOption"
                  style="width: 100%; height: 100%"
                  :autoresize="true"
                />
              </div>
              <div class="no-chart-data" v-else>
                <img src="/images/no-data.png" alt="暂无数据" />
              </div>
            </div>
          </div>
        </ResizeModal>
      </template>
    </div>
    <sensor-modal ref="sensorModal"></sensor-modal>
    <data-analysis-form ref="dataAnalysisForm"></data-analysis-form>
    <artificial-modal ref="refNode2"></artificial-modal>
    <threshold-modal ref="refNode1"></threshold-modal>
  </div>
</template>

<script>
import config from "@/config";
import ColorIcons from "_c/color-icons";
import VueBMap from "@/api/map/BMap.js";
import SearchTrieTree from "@/api/fuzzymatching/fuzzyMatching.js";
import Icons from "_c/icons";
import vueSeamlessScroll from "vue-seamless-scroll";

import SensorModal from "./sensor-modal/sensor-modal.vue";
import { getCenterPoint } from "@/libs/mapUtil.js";
import { getToken } from "@/libs/util";

import { mapActions, mapGetters } from "vuex";
import ArtificialModal from "./components/artificial-modal";
import ThresholdModal from "./components/threshold-modal";
import {
  getProjectList,
  getProjectPointList,
  getMonitorMode,
  getWarningList,
  getChartDatas,
  getAreaList,
  getMonitorObject,
  getRealTimeList,
} from "@/api/realtime/realtimeMapApi";

import DataAnalysisForm from "@/view/apps/manage/comprehensive/sensor/form/analysis/data-analysis-form.vue";
import ResizeModal from "@/components/modal";
import { getDataFromUrl } from "@/api/common";
import MonitorCurveV3 from "@/view/apps/realtime-monitor/data-analyse/monitor-curve/monitor-curve-v3.vue";
import eventBus from "@/eventbus/index.js";
import bus from "@/libs/bus";

export default {
  name: "realtimemonitor",
  components: {
    MonitorCurveV3,
    Icons,
    ColorIcons,
    ResizeModal,
    SensorModal,
    DataAnalysisForm,
    vueSeamlessScroll,
    ArtificialModal,
    ThresholdModal,
  },
  computed: {
    ...mapGetters(["mapCenter", "mapLevel", "geoJsonId"]),
    getWarnCardClass() {
      let className = "";
      if (this.showWarnCard) {
        className = "show";
      } else {
        className = "hide";
      }
      return className;
    },
    get3DButtonTitle() {
      if (this.isTripDim) {
        return "切换为2D";
      } else {
        return "切换为3D";
      }
    },
    areaCodeParent() {
      return this.$store.state.user.company.areaCode;
    },
    areaNameParent() {
      return this.$store.state.user.company.name;
    },
    orgUnitId() {
      return this.$store.state.user.company.id;
    },
    userId() {
      return this.$store.state.user.userId;
    },
  },
  watch: {
    searchValue(val) {
      this.$refs.tree.filter(val);
    },
  },
  data() {
    return {
      ponintList: [], //监测点电位数据
      active: false,
      defaultProps: {
        children: "children",
        label: "label",
      },
      htmlData: "",
      isTripDim: false,
      loading: false,
      Map: null,
      BBMap: null,
      mapZoom: 11,
      showFunctional: true,
      projectList: [], // 项目列表集合
      sensorList: [],
      projectMarker: [], // 项目Marker点集合
      countyMarker: [], // 乡镇名称marker集合
      clickMarkerProject: null, // 当前点击的是哪个对象
      clickMarkerProjectHaseWave: false, // 是否有水波纹
      clickMarkerList: [],
      clickProjectObject: {}, // 当前点击的项目的对象
      searchValue: "", // 搜索关键字
      showScreen: false, // 控制是否显示筛选框
      showWarnMessageList: false,
      showLayer: false, // 控制图层
      projectName: "", // 监测项目名称
      projectLocation: "", // 监测项目位置
      projectId: "", // 监测项目id
      projectIds: [], // 所有的监测项目的集合
      curPointList: [],
      showPointModal: false, // 控制监测点弹窗
      markerClusterer: null, // 聚合点对象
      waveList: [], // 水波纹合集
      checkAll: false, // 筛选功能-是否全选
      monitorObjectCheckAll: false,
      indeterminate: false, // 筛选功能-设置 indeterminate 状态，只负责样式控制
      indeterminateMonitorObject: false,
      checkAllGroup: [], // 筛选功能，选项列表
      checkAllLayer: [], // 勾选的图层
      checkAllMonitorObjectGroup: [], //
      checkAllListMethod: [], // 监测方式选择框列表
      checkAllListObject: [],
      monitorMode: "1",
      monitorObject: "",
      monitorMethodList: [],
      monitorObjectList: [],
      showWarnCard: true, // 是否显示右侧警情卡片
      curLabel1: "",
      curLabel2: "",
      curLabel3: "",
      areaCode: "",
      viewScreenList: [],
      projectNum: 0,
      proNum: 0,
      sensorNum: 0,
      badSensorNum: 0,
      warningCountNumber: 0, // 待办警情数量
      warnBlick: true,
      abnormalSensorList: [], // 异常传感器数据列表
      chartsModalList: [], // 统计图表弹窗
      warningList: [
        // {
        //   projectName: "闸前监测点",
        //   warningMessage: {
        //     warnTime: "2021-03-17 16:58:55",
        //     warnContent: "监测到水位异常，一直下雨，雨量增大",
        //   },
        // },
        // {
        //   warnTime: "2021-03-17 16:58:55",
        //   projectName: "闸前监测点",
        //   warnContent: "监测到水位异常，一直下雨，雨量增大",
        // },
        // {
        //   warnTime: "2021-03-17 16:58:55",
        //   projectName: "闸前监测点",
        //   warnContent: "监测到水位异常，一直下雨，雨量增大",
        // },
        // {
        //   warnTime: "2021-03-17 16:58:55",
        //   projectName: "闸前监测点",
        //   warnContent: "监测到水位异常，一直下雨，雨量增大",
        // },
      ], // 警情列表
      wmsLayer: "", //水务图层
      layers: "http://localhost:8080/geoserver/nachong/wms",
      curPointId: "",
      sensorTabLeft: 0,
      analyseProjectId: null,
      analyseProjectType: null,
      analyseBack: false,
      areaTreeData: [],
      timer: null,
      isRealOpen: true, //是否打开实时数据
      isPointOpen: true, //是否打开监测点列表
      // 默认设置
      defaultOption: {
        step: 0.2, // 数值越大速度滚动越快
        limitMoveNum: 2, // 开始无缝滚动的数据量 this.dataList.length
        hoverStop: true, // 是否开启鼠标悬停stop
        direction: 1, // 0向下 1向上 2向左 3向右
        openWatch: true, // 开启数据实时监控刷新dom
        singleHeight: 0, // 单步运动停止的高度(默认值0是无缝不停止的滚动) direction => 0/1
        singleWidth: 0, // 单步运动停止的宽度(默认值0是无缝不停止的滚动) direction => 2/3
        waitTime: 1000,
        autoPlay: false,
      },
      monitoringPointList: [], //监测点列表数据
      monitorPointInfo: {}, //监测点信息
      tabActive: 0, //当前tab选中
      // 曲线图tab数据
      tabList: [
        {
          label: "水位",
          value: 0,
          unit: "m",
        },
        {
          label: "流量",
          value: 1,
          unit: "m³/s",
        },
        {
          label: "降雨量",
          value: 2,
          unit: "mm",
        },
        {
          label: "水温",
          value: 3,
          unit: "℃",
        },
        {
          label: "pH",
          value: 4,
          unit: "",
        },
        {
          label: "电导率",
          value: 5,
          unit: "μS/cm",
        },
        {
          label: "浊度",
          value: 6,
          unit: "NTU",
        },
        {
          label: "溶解氧",
          value: 7,
          unit: "mg/L",
        },
        {
          label: "氨氮",
          value: 8,
          unit: "mg/L",
        },
        {
          label: "化学需氧量",
          value: 9,
          unit: "mg/L",
        },
      ],
      rowData: {},
      currentId: "",
    };
  },
  methods: {
    ...mapActions(["getNewWarnList"]),

    // 阈值和人工预警弹窗
    handleOpenModal(val) {
      this.$refs["refNode" + val].show();
    },

    // 切换tab页
    handleChangeTab(item, index) {
      this.tabActive = index;
      console.log(this.projectId);
      let contentNo = item.contentCode;
      let contentName = item.contentName;
      let sensorId = item.sensorList[0].sensorId;
      let projectId = this.projectId;
      let defaultConditions = {};
      this.$nextTick(() => {
        this.$refs.warnCurveRef.load(
          {
            contentNo,
            contentName,
            projectId,
            sensorId,
          },
          defaultConditions
        );
      });
    },
    // 勾选当前监测点时
    handlPointChange(data, node) {
      this.active = true;
      if (data.sensorId) {
        this.active = false;
        return;
      } else if (data.title) {
        this.showPointModal = false;
        this.removeMapMaker();
        return;
      }
      this.rowData = data;
      if (!data.id) {
        this.showPointModal = false;
        return;
      }
      this.getDetailProject(data.id);
      this.clickPointListItem(data);

      this.showPointModal = true;
      this.tabActive = 0;
    },

    /**
     * 关闭监测点弹窗
     */
    closePointModal() {
      this.showPointModal = false;
      this.removeMapMaker();
    },
    // 关闭弹窗移除标记点
    removeMapMaker() {

      this.clickMarkerList.forEach((item) => {
        this.Map.removeOverLay(item);
      });
      // TODO linx 移除三维地图的标点
      let dataSources = window.viewer.dataSources.getByName("pointClickDraw");
      if (dataSources.length > 0) {
        for (let i = 0; i < dataSources.length; i++) {
          window.viewer.dataSources.remove(dataSources[i]);
        }
      }
      // // TODO ycz 三维地图恢复默认定位视角
      // let center = this.getCenter();
      // // 将三维球定位到阆中
      // window.viewer.camera.flyTo({
      //   destination: Cesium.Cartesian3.fromDegrees(
      //     center[0] + 0.1,
      //     center[1] - 0.7,
      //     80000
      //   ),
      //   orientation: {
      //     heading: Cesium.Math.toRadians(348.4202942851978),
      //     pitch: Cesium.Math.toRadians(-45.74026687972041),
      //     roll: Cesium.Math.toRadians(0),
      //   },
      //   complete: function callback() {
      //     // 定位完成之后的回调函数
      //   },
      // });
      // 把之前取消的点加回去--ycz
      this.Map.addOverLay(this.clickMarkerProject);
      // if (this.clickMarkerProjectHaseWave) {
      //   this.displaySingleWave(this.clickMarkerProject, true);
      // }
      this.Map.setZoom(11);
    },
    // 获取实时数据
    getRealTimeList() {
      getRealTimeList()
        .then((res) => {
          console.log("获取实时数据", res.data);
          if (!res) return;
          this.monitoringPointList = res.data;
        })
        .catch((err) => {});
    },
    // 渲染监测点列表树
    renderContent(h, { node, data }) {
      if (data.runStatus) {
        return h(
          "span",
          {
            style: {
              display: "inline-block",
              "font-size": "14px",
            },
          },
          [
            h(
              "span",
              {
                style: {
                  "padding-left": "5px",
                },
              },
              [h("span", data.label)]
            ),
            h(
              "span",
              {
                style: {
                  display: "inline-block",
                  "padding-left": "10px",
                },
              },
              [
                h("img", {
                  attrs: {
                    src:
                      data.runStatus == 1
                        ? "/images/on-line.png"
                        : "/images/off-line.png",
                  },
                }),
              ]
            ),
          ]
        );
      } else {
        return h("span", [
          h(
            "span",
            {
              style: {
                display: "inline-block",
                "padding-left": "5px",

                "font-size": data.title ? "16px" : "14px",
              },
            },
            [h("span", data.label)]
          ),
        ]);
      }
    },
    //实时数据是否展开
    handleRealOpen() {
      this.isRealOpen = !this.isRealOpen;
    },
    //监测点位列表是否展开
    handlePointOpen() {
      this.isPointOpen = !this.isPointOpen;
    },
    // 查看某个监测点信息
    getDetailProject(id) {
      getRealTimeList(id)
        .then((res) => {
          this.monitorPointInfo = {};
          if (!res) return;
          console.log("查看某个监测点信息", res.data);
          this.monitorPointInfo = res.data;
        })
        .catch((err) => {});
    },
    ...mapActions([
      "setRealtimeProjectId",
      "setRealtimeProjectName",
      "setRealtimeProjectType",
    ]),
    getCenter() {
      let pointCenterLon = this.mapCenter[0] || "";
      let pointCenterLat = this.mapCenter[1] || "";
      if (!pointCenterLon) {
        pointCenterLon = config.tdt.center.lon;
      }
      if (!pointCenterLat) {
        pointCenterLat = config.tdt.center.lat;
      }
      return [pointCenterLon, pointCenterLat];
    },
    /*
     * 初始化地图函数
     */
    initMap() {
      this.Map = new T.Map("map", {
        //projection:"EPSG:4326"
      });
      this.Map.setMinZoom(3);
      this.Map.setMaxZoom(30);
      var center = this.getCenter();
      var point = new T.LngLat(center[0], center[1]);

      // this.Map.centerAndZoom(point, this.mapZoom || this.mapLevel) // 设置地图默认中心点以及默认级别
      this.Map.centerAndZoom(point, this.mapZoom);
      // 地图类型切换控件
      this.Map.addControl(
        new T.Control.MapType({
          position: T_ANCHOR_TOP_RIGHT,
        })
      );
      // 初始化地图平移缩放控件
      this.Map.addControl(
        new T.Control.Zoom({
          position: T_ANCHOR_TOP_RIGHT,
        })
      );

      setTimeout(() => {
        // 默认设置类型为卫星地图
        this.Map.setMapType(TMAP_SATELLITE_MAP);
        this.Map.checkResize();
      }, 500);

      this.Map.addEventListener("maptypechange", () => {
        this.checkLayerChange();
      });

      // 地图全局事件监听
      this.mapEventListen();
      //
      this.getCityBdary();
      // 定义阆中的地质、水文图
      /*var imageURL =
        "http://plat-gis.clzytech.com:8091/geoserver/gwc/service/wmts/rest/langzhong:geo_map/raster/EPSG:900913/EPSG:900913:{z}/{y}/{x}?format=image/png";
      window.geoLayer = new T.TileLayer(imageURL, {
        minZoom: 1,
        maxZoom: 18,
        errorTileUrl: "/images/map/blank.png",
      });
      imageURL =
        "http://plat-gis.clzytech.com:8091/geoserver/gwc/service/wmts/rest/langzhong:hyp_map/raster/EPSG:900913/EPSG:900913:{z}/{y}/{x}?format=image/png";
      window.hypLayer = new T.TileLayer(imageURL, {
        minZoom: 1,
        maxZoom: 18,
        errorTileUrl: "/images/map/blank.png",
      });*/

      // linx 三维地图初始化完成后才
      this.initTripMap();
      // 获取项目列表
      this.getProjectList();
    },
    getWMS(url, config) {
      if (this.wmsLayer) {
        this.Map.removeLayer(wmsLayer);
      }
      this.wmsLayer = new T.TileLayer.WMS(url, config);
      //
      // var layers=this.Map.getLayers()
      // console.log(layers)
      // layers.push(this.wmsLayer)
      this.Map.addLayer(this.wmsLayer);
    },

    getWMTS(url, config) {
      var wmtsLayer = new T.TileLayer({
        opacity: 0.7, //图层透明度
        source: new ol.source.WMTS({
          url: "http://localhost:8080/geoserver/gwc/service/wmts", //WMTS服务基地址
          matrixSet: "EPSG:4326", //投影坐标系设置矩阵
          format: "image/png", //图片格式
          projection: projection, //数据的投影坐标系
          //瓦片网格对象
          tileGrid: tileGrid,
          style: "default",
          wrapX: true,
        }),
      });
      this.Map.addLayer(wmtsLayer);
    },

    addWmsLayer(layers, url, zindex) {
      var config = {
        version: "1.1.1", //请求服务的版本
        layers: layers,
        transparent: true, //输出图像背景是否透明
        styles: "", //每个请求图层的用","分隔的描述样式
        format: "image/png", //输出图像的类型
        zIndex: zindex,
      };
      this.getWMS(url, config);
      // this.getWMTS(url, config);
    },
    gotoHome() {
      // 设置home位置
      var center = this.getCenter();
      try {
        var point = new T.LngLat(center[0], center[1] + 0.05);
        this.Map.centerAndZoom(point, this.mapZoom); // 设置地图默认中心点以及默认级别
      } catch (e) {}

      var homeCameraView = {
        destination: Cesium.Cartesian3.fromDegrees(
          center[0] + 0.1,
          center[1] - 0.7,
          80000
        ),
        orientation: {
          heading: Cesium.Math.toRadians(348.4202942851978),
          pitch: Cesium.Math.toRadians(-45.74026687972041),
          roll: Cesium.Math.toRadians(0),
        },
        complete: function callback() {
          // 定位完成之后的回调函数
        },
      };
      if (window.viewer) {
        window.viewer.camera.flyTo(homeCameraView);
      }
    },
    // 初始化三维地图
    initTripMap() {
      if (window.viewer) {
        var div = document.getElementById("cesiumMap");
        div.appendChild(window.viewer._element);
        return;
      }
      var token = config.tdt.tk;
      // 服务域名
      var tdtUrl = "https://t{s}.tianditu.gov.cn/";
      // 服务负载子域
      var subdomains = ["0", "1", "2", "3", "4", "5", "6", "7"];

      // cesium 初始化
      window.viewer = new Cesium.Map("cesiumMap", {
        animation: false,
        timeline: false,
        homeButton: false,
        navigation: false,
        infoBox: false,
        fullscreenButton: false,
        scene3DOnly: true,
        navigationHelpButton: true,
        selectionIndicator: true,
      });
      window.viewer.scene.globe.depthTestAgainstTerrain = true;
      // 去除版权信息
      window.viewer._cesiumWidget._creditContainer.style.display = "none";
      let provider = new Cesium.CesiumTerrainProvider({
        url: "https://www.supermapol.com/realspace/services/3D-stk_terrain/rest/realspace/datas/info/data/path",
        // 地形服务源自SuperMap iServer发布时需设置isSct为true
        requestVertexNormals: true,
        requestWaterMask: false,
        isSct: true,
      });
      window.viewer.terrainProvider = provider;
      // 叠加影像服务
      var imgMap = new Cesium.UrlTemplateImageryProvider({
        url: tdtUrl + "DataServer?T=img_w&x={x}&y={y}&l={z}&tk=" + token,
        subdomains: subdomains,
        tilingScheme: new Cesium.WebMercatorTilingScheme(),
        maximumLevel: 18,
      });
      window.viewer.imageryLayers.addImageryProvider(imgMap);

      // 嘉陵江图层
      var wgs84 = new Cesium.GeographicTilingScheme();
      var jialingjiang = new Cesium.WebMapServiceImageryProvider({
        // 这里是你的 geoserver服务点击查看图层的 url
        url: "http://server.clzytech.com:8861/geoserver/nanchong/wms",
        // 这里是自定义的图层名称
        layers: "nanchong:jilingjiang",
        parameters: {
          service: "WMS",
          format: "image/png",
          transparent: true,
          tileMatrixSetID: "EPSG:4326",
          tilingScheme: wgs84,
        },
      });
      // 图层添加
      window.viewer.imageryLayers.addImageryProvider(jialingjiang);
      //window.viewer.raiseToTop(window.viewer.imageryLayers.addImageryProvider(jialingjiang))

      // // 叠加影像服务
      // let geoMapProvider = new Cesium.UrlTemplateImageryProvider({
      //   url:
      //     "http://plat-gis.clzytech.com:8091/geoserver/gwc/service/wmts/rest/langzhong:geo_map/raster/EPSG:900913/EPSG:900913:{z}/{y}/{x}?format=image/png",
      //   tilingScheme: new Cesium.WebMercatorTilingScheme(),
      //   maximumLevel: 18,
      // });
      // window.geoMap = viewer.imageryLayers.addImageryProvider(geoMapProvider);
      // window.geoMap.show = false;
      //
      // let hypMapProvider = new Cesium.UrlTemplateImageryProvider({
      //   url:
      //     "http://plat-gis.clzytech.com:8091/geoserver/gwc/service/wmts/rest/langzhong:hyp_map/raster/EPSG:900913/EPSG:900913:{z}/{y}/{x}?format=image/png",
      //   tilingScheme: new Cesium.WebMercatorTilingScheme(),
      //   maximumLevel: 18,
      // });
      // window.hypMap = viewer.imageryLayers.addImageryProvider(hypMapProvider);
      // window.hypMap.show = false;

      // 设置home位置
      let centerTemp = this.getCenter();
      let lon = parseFloat(centerTemp[0]);
      let lat = parseFloat(centerTemp[1]);
      let center = [lon, lat];
      console.log(center);
      Cesium.Camera.DEFAULT_VIEW_RECTANGLE = Cesium.Rectangle.fromDegrees(
        center[0],
        center[1],
        center[0],
        center[1]
      );
      var homeCameraView = {
        destination: Cesium.Cartesian3.fromDegrees(
          center[0] + 0.1,
          center[1] - 0.7,
          80000
        ),
        orientation: {
          heading: Cesium.Math.toRadians(348.4202942851978),
          pitch: Cesium.Math.toRadians(-45.74026687972041),
          roll: Cesium.Math.toRadians(0),
        },
        complete: function callback() {
          // 定位完成之后的回调函数
        },
      };
      // 将三维球定位到阆中
      window.viewer.scene.camera.setView(homeCameraView);
      window.viewer.camera.flyTo(homeCameraView);

      // region 设置相机不要到地下 是定义的viewer对象的全局变量
      let canvas = viewer.canvas;
      let camera = viewer.camera;
      let scene = viewer.scene;
      scene.screenSpaceCameraController.minimumZoomDistance = 100; // 距离地形的距离？这个值可以多测试几个值，，我这不太好描述
      viewer.clock.onTick.addEventListener(function () {
        setMinCamera();
      });
      var setMinCamera = function () {
        if (camera.pitch > 0) {
          scene.screenSpaceCameraController.enableTilt = false;
        }
      };
      var startMousePosition;
      var mousePosition;
      var hitHandler = new Cesium.ScreenSpaceEventHandler(canvas);
      hitHandler.setInputAction(function (movement) {
        mousePosition = startMousePosition = Cesium.Cartesian3.clone(
          movement.position
        );
        hitHandler.setInputAction(function (movement) {
          mousePosition = movement.endPosition;
          var y = mousePosition.y - startMousePosition.y;
          if (y > 0) {
            scene.screenSpaceCameraController.enableTilt = true;
          }
        }, Cesium.ScreenSpaceEventType.MOUSE_MOVE);
      }, Cesium.ScreenSpaceEventType.MIDDLE_DOWN);
      // endregion

      this.getCityBdaryForTripDim();

      let handler = new Cesium.ScreenSpaceEventHandler(viewer.scene.canvas);
      var _this = this;
      // console.log("scene", scene);

      handler.setInputAction(function (movement) {
        var scene = viewer.scene;

        // 关闭如果打开的有框
        _this.closeMessageList();
        _this.closeScreen();

        if (scene.mode !== Cesium.SceneMode.MORPHING) {
          var pickedObject = scene.pick(movement.position);
          if (
            scene.pickPositionSupported &&
            Cesium.defined(pickedObject) &&
            pickedObject.id
          ) {
            var cartesian = viewer.scene.pickPosition(movement.position);
            // 判断当前点击的点的类型
            let type = "";
            try {
              type = pickedObject.id.pointData.type || "";
            } catch (e) {
              // 报错说明不是我们自己添加的点
            }
            if (type === "project") {
              // 找到当前点击的是哪个点，二维地图的地方要用--ycz
              let clickmaer = null;

              for (let i = 0; i < _this.projectMarker.length; i++) {
                let pointData = _this.projectMarker[i].pointData;
                let id = pointData.id || "";
                if (id && id === pickedObject.id.pointData.id) {
                  clickmaer = _this.projectMarker[i];
                  break;
                }
              }
              if (clickmaer) {
                _this.clickMarkerProject = clickmaer;
                if (clickmaer.waveItem) {
                  _this.clickMarkerProjectHaseWave = true;
                } else {
                  _this.clickMarkerProjectHaseWave = false;
                }
                _this.clickProjectMarker(pickedObject.id.pointData, clickmaer);
              } else {
                _this.clickProjectMarker(pickedObject.id.pointData);
              }
              // 选中左侧监测站列表--ycz
              _this.setProjectListActive(pickedObject.id.pointData.id, true);
            } else if (type === "sensor") {
              _this.clickPointMarker(pickedObject.id.pointData);
            }
          }
        }
      }, Cesium.ScreenSpaceEventType.LEFT_CLICK);

      // 叠加三维地名服务
      // var wtfs = new Cesium.GeoWTFS({
      //   viewer,
      //   subdomains:subdomains,
      //   metadata:{
      //     boundBox: {
      //       minX: -180,
      //       minY: -90,
      //       maxX: 180,
      //       maxY: 90
      //     },
      //     minLevel: 1,
      //     maxLevel: 20
      //   },
      //   aotuCollide: true, //是否开启避让
      //   collisionPadding: [115, 100, 80, 50], //开启避让时，标注碰撞增加内边距，上、右、下、左
      //   serverFirstStyle: true, //服务端样式优先
      //   labelGraphics: {
      //     font:"28px sans-serif",
      //     fontSize: 28,
      //     fillColor:Cesium.Color.WHITE,
      //     scale: 0.5,
      //     outlineColor:Cesium.Color.BLACK,
      //     outlineWidth: 5,
      //     style:Cesium.LabelStyle.FILL_AND_OUTLINE,
      //     showBackground:false,
      //     backgroundColor:Cesium.Color.RED,
      //     backgroundPadding:new Cesium.Cartesian2(10, 10),
      //     horizontalOrigin:Cesium.HorizontalOrigin.MIDDLE,
      //     verticalOrigin:Cesium.VerticalOrigin.TOP,
      //     eyeOffset:Cesium.Cartesian3.ZERO,
      //     pixelOffset:new Cesium.Cartesian2(0, 8)
      //   },
      //   billboardGraphics: {
      //     horizontalOrigin:Cesium.HorizontalOrigin.CENTER,
      //     verticalOrigin:Cesium.VerticalOrigin.CENTER,
      //     eyeOffset:Cesium.Cartesian3.ZERO,
      //     pixelOffset:Cesium.Cartesian2.ZERO,
      //     alignedAxis:Cesium.Cartesian3.ZERO,
      //     color:Cesium.Color.WHITE,
      //     rotation:0,
      //     scale:1,
      //     width:18,
      //     height:18
      //   }
      // });
      // //三维地名服务，使用wtfs服务
      // wtfs.getTileUrl = function(){
      //   return tdtUrl + 'mapservice/GetTiles?lxys={z},{x},{y}&tk='+ token;
      // }
      /*
             wtfs.initTDT([{"x":6,"y":1,"level":2,"boundBox":{"minX":90,"minY":0,"maxX":135,"maxY":45}},{"x":7,"y":1,"level":2,"boundBox":{"minX":135,"minY":0,"maxX":180,"maxY":45}},{"x":6,"y":0,"level":2,"boundBox":{"minX":90,"minY":45,"maxX":135,"maxY":90}},{"x":7,"y":0,"level":2,"boundBox":{"minX":135,"minY":45,"maxX":180,"maxY":90}},{"x":5,"y":1,"level":2,"boundBox":{"minX":45,"minY":0,"maxX":90,"maxY":45}},{"x":4,"y":1,"level":2,"boundBox":{"minX":0,"minY":0,"maxX":45,"maxY":45}},{"x":5,"y":0,"level":2,"boundBox":{"minX":45,"minY":45,"maxX":90,"maxY":90}},{"x":4,"y":0,"level":2,"boundBox":{"minX":0,"minY":45,"maxX":45,"maxY":90}},{"x":6,"y":2,"level":2,"boundBox":{"minX":90,"minY":-45,"maxX":135,"maxY":0}},{"x":6,"y":3,"level":2,"boundBox":{"minX":90,"minY":-90,"maxX":135,"maxY":-45}},{"x":7,"y":2,"level":2,"boundBox":{"minX":135,"minY":-45,"maxX":180,"maxY":0}},{"x":5,"y":2,"level":2,"boundBox":{"minX":45,"minY":-45,"maxX":90,"maxY":0}},{"x":4,"y":2,"level":2,"boundBox":{"minX":0,"minY":-45,"maxX":45,"maxY":0}},{"x":3,"y":1,"level":2,"boundBox":{"minX":-45,"minY":0,"maxX":0,"maxY":45}},{"x":3,"y":0,"level":2,"boundBox":{"minX":-45,"minY":45,"maxX":0,"maxY":90}},{"x":2,"y":0,"level":2,"boundBox":{"minX":-90,"minY":45,"maxX":-45,"maxY":90}},{"x":0,"y":1,"level":2,"boundBox":{"minX":-180,"minY":0,"maxX":-135,"maxY":45}},{"x":1,"y":0,"level":2,"boundBox":{"minX":-135,"minY":45,"maxX":-90,"maxY":90}},{"x":0,"y":0,"level":2,"boundBox":{"minX":-180,"minY":45,"maxX":-135,"maxY":90}}]);
            */
    },
    getCityBdaryForTripDim() {
      const url =
        window.baseURL +
        "sys/file/download?id=" +
        this.geoJsonId +
        "&token=" +
        getToken();
      getDataFromUrl(url)
        .then((res) => {
          for (var i = 0; i < res.features.length; i++) {
            let geoPromise = Cesium.GeoJsonDataSource.load(res.features[i], {
              stroke: new Cesium.Color(0.407, 0.815, 0.972, 1),
              fill: new Cesium.Color(1, 1, 1, 0.3),
              strokeWidth: 3,
              clampToGround: true,
            });
            geoPromise.then(function (dataSource) {
              dataSource.name = "area" + i;
              window.viewer.dataSources.add(dataSource);
            });
          }
        })
        .catch((err) => {
          console.error(err);
        });
    },
    getCityBdary() {
      if (!this.geoJsonId) {
        return;
      }
      const url =
        window.baseURL +
        "sys/file/download?id=" +
        this.geoJsonId +
        "&token=" +
        getToken();
      getDataFromUrl(url)
        .then((res) => {
          let coords = [];
          for (let i = 0; i < res.features.length; i++) {
            let points = [];
            if (res.features[i].geometry.type === "MultiPolygon") {
              const areaDatas = res.features[i].geometry.coordinates.forEach(
                (polygon) => {
                  points = [];
                  polygon.forEach((lonlats) => {
                    lonlats.map((lonlat) => {
                      points.push(new T.LngLat(lonlat[0], lonlat[1]));
                      return [lonlat[0], lonlat[1]];
                    });
                  });
                  coords.push(points);
                }
              );
            } else {
              points = [];
              const areaDatas = res.features[i].geometry.coordinates.forEach(
                (lonlats) => {
                  lonlats.map((lonlat) => {
                    points.push(new T.LngLat(lonlat[0], lonlat[1]));
                    return [lonlat[1], lonlat[0]];
                  });
                }
              );
              coords.push([points]);
            }

            // 创建面对象
            // let polygon = new T.Polygon(points, {
            //   color: '#68d0f8',
            //   weight: 2,
            //   opacity: 0.8,
            //   fillColor: '#FFFFFF',
            //   fillOpacity: 0.2
            // })
            // // 向地图上添加面
            // this.Map.addOverLay(polygon)
            let center =
              res.features[i].properties.Center ||
              res.features[i].properties.center;
            if (!center) {
              center = getCenterPoint(res.features[i].geometry.coordinates);
            }
            let label = new T.Label({
              text:
                res.features[i].properties.Name ||
                res.features[i].properties.name,
              position: new T.LngLat(center[0], center[1]),
              offset: new T.Point(-42, 0),
              notClear: true,
            });
            label.setBackgroundColor("transparent");
            label.setBorderLine(0);
            label.setFontColor("WHITE");
            this.countyMarker.push(label);
            // 创建地图文本对象
            this.Map.addOverLay(label);
          }

          // 添加遮罩
          const coordsMask = [
            [
              [
                [
                  new T.LngLat(180, 90),
                  new T.LngLat(180, -90),
                  new T.LngLat(-180, -90),
                  new T.LngLat(-180, 90),
                ],
              ],
            ],
            ...coords,
          ];
          const ploygonMask = new T.Polygon(coordsMask, {
            color: "#3399CC",
            weight: 2,
            opacity: 0.8,
            fillColor: "rgba(0,0,0,0.6)",
            fillOpacity: 1,
          });
          this.Map.addOverLay(ploygonMask);
        })
        .catch((err) => {});
    },
    addFeature(map, features, { formatter, type, group, dataType }) {
      return new Promise((resolve, reject) => {
        group = group || "defaultFeatures";
        dataType = dataType || "Array";
        type = type || "marker";
        if (dataType === "Array") {
          if (type === "polygon") {
            let geoJson = {
              type: "Feature",
              geometry: {
                type: "Polygon",
                coordinates: [formatter ? formatter(features) : features],
              },
            };
            let geoPromise = Cesium.GeoJsonDataSource.load(geoJson, {
              fill: new Cesium.Color(0.32, 0.14, 0.95, 0.2),
              stroke: new Cesium.Color(0.28, 0.23, 0.86, 1),
              strokeWidth: 10,
              clampToGround: true,
            });
            geoPromise.then(function (dataSource) {
              dataSource.name = group;
              map.dataSources.add(dataSource);
            });
          } else {
            let dataSource = new Cesium.CustomDataSource(group);
            features.map((item) => {
              let entity = null;
              if (type === "marker") {
                entity = this.buildMarker(
                  map,
                  formatter ? formatter(item) : item
                );
              }
              if (entity instanceof Promise) {
                entity
                  .then((res) => {
                    dataSource.entities.add(res);
                  })
                  .catch((e) => {
                    console.error(e);
                  });
              } else {
                dataSource.entities.add(entity);
              }
            });
            map.dataSources.add(dataSource);
            resolve({
              cesiumEntity: dataSource,
            });
          }
        } else {
        }
      });
    },
    buildMarker(map, ccMapFeature) {
      return new Promise((resolve, reject) => {
        // 转成弧度坐标
        let position = Cesium.Cartographic.fromDegrees(
          ccMapFeature.longitude,
          ccMapFeature.latitude
        );
        var promise = Cesium.sampleTerrain(map.terrainProvider, 10, [position]);
        promise.then((updatedPositions) => {
          // positions[0].height and positions[1].height have been updated.
          // updatedPositions is just a reference to positions.
          // 弧度再转成笛卡尔3坐标
          let ellipsoid = map.scene.globe.ellipsoid;
          let hepos = updatedPositions[0];
          if (isNaN(hepos.height)) {
            hepos.height = 0;
          }
          let pos = ellipsoid.cartographicToCartesian(hepos);
          let icon = {
            image: "",
          };
          if (typeof ccMapFeature.icon === "string") {
            icon.image = ccMapFeature.icon;
          } else {
            icon = ccMapFeature.icon;
          }
          resolve({
            id: ccMapFeature.id || new Date().getTime(),
            name: ccMapFeature.name || "",
            position: pos,
            billboard: {
              image: icon.image || "/images/map/point.png",
              show: true, // default
              width: icon.width, // default: undefined
              height: icon.height, // default: undefined
              scaleByDistance: new Cesium.NearFarScalar(1.5e2, 1.0, 1.5e7, 0.5),
              // heightReference: Cesium.HeightReference.RELATIVE_TO_GROUND, // 坐标里面计算了高度进去，所以基于平面
              horizontalOrigin: Cesium.HorizontalOrigin.CENTER,
              verticalOrigin: Cesium.VerticalOrigin.BOTTOM,
              // pixelOffset: this.pixelOffset // 偏移
            },
            infoWindow: ccMapFeature.infoWindow,
            pointData: ccMapFeature.data,
          });
        });
      });
    },
    /**
     * 初始化聚合点
     */
    initClusterer() {
      let theBMap = VueBMap;
      let that = this;
      theBMap.initTextIconOverlay().then(() => {
        theBMap.initMarkerClusterer().then(() => {
          // 实例化聚合点工具
          that.markerClusterer = new BMapLib.MarkerClusterer(that.Map, {
            markers: that.projectMarker,
            maxZoom: 17,
          });
        });
      });
    },
    /*
     * 地图全局事件监听
     */
    mapEventListen() {
      let that = this;
      // 监听地图缩放结束事件，如果级别大于14才显示标题label，否则不显示
      this.Map.addEventListener("zoomend", function (e) {
        console.log("地图全局事件监听");
        var zoom = this.getZoom();
        if (zoom >= 14) {
          that.projectMarker.map((item, index) => {
            if (that.clickMarkerProject) {
              let nowClickId = that.clickMarkerProject.pointData.id || "";
              if (item.pointData.id !== nowClickId) {
                let label = item.labelData;
                // 添加标题
                that.Map.addOverLay(label);
                that.displaySingleWave(item, true);
              }
            } else {
              let label = item.labelData;
              // 添加标题
              that.Map.addOverLay(label);
              that.displaySingleWave(item, true);
            }
          });
          that.countyMarker.forEach((item) => {
            item.setOpacity(1);
          });
        } else {
          that.projectMarker.map((item, index) => {
            let label = item.labelData;
            // 清除标题
            that.Map.removeOverLay(label);
            that.displaySingleWave(item, true);
          });
          that.countyMarker.forEach((item) => {
            item.setOpacity(0);
          });
        }

        // 控制乡镇一级的label的显示
        // console.log(zoom)
        if (zoom >= 10) {
          that.countyMarker.forEach((item) => {
            item.setOpacity(1);
          });
        } else {
          that.countyMarker.forEach((item) => {
            item.setOpacity(0);
          });
        }
      });
      this.Map.addEventListener("click", function (e) {
        that.closeScreen();
        that.closeMessageList();
      });
    },
    /*
     * 获取监测项目列表
     */
    getProjectList(noLoading = true) {
      // 根据不同业务设置loading是否打开
      this.loading = !!noLoading;
      this.projectList = [];
      getProjectList({
        orgUnitId: this.orgUnitId,
        area: this.areaCode,
        monitorMode: this.monitorMode,
        monitorObject: this.monitorObject,
        projectName: this.searchValue,
      })
        .then((req) => {
          console.log("获取监测项目列表", req.data);
          // 处理数据，如果存在异常则提示
          // console.log(req);
          if (req.code === 200) {
            const data = req.data || {};
            this.clearAllMarker();
            this.setData(req.data);
            this.parseProjectListForMap(this.ponintList);
          } else {
            this.$Message.error(req.msg || "获取监测站列表失败!");
          }
          this.loading = false;
        })
        .catch((err) => {
          this.loading = false;
          console.error(err);
          this.$Message.error("服务器异常，请稍后再试!");
        });
    },
    // 过滤监测数据
    filterNode(value, data) {
      console.log(value, data);
      if (!value) return true;
      return data.label.indexOf(value) !== -1;
    },
    // 监测数据字段处理
    setData(list) {
      this.ponintList = [];
      this.projectList = list.map((item) => {
        item.children = item.projectList;
        delete item.projectList;
        item.label = item.area;
        item.title = item.area;
        item.children.map((temp) => {
          this.ponintList.push(temp);
          temp.children = temp.sensorList;
          delete item.sensorList;
          temp.label = temp.projectName;
          temp.children.map((element) => {
            element.label = element.sensorName;
          });
        });
        return item;
      });
    },
    /*
     * 将监测项目的点绘制到地图上
     */
    parseProjectListForMap(projectList) {
      if (this.showPointModal) {
        this.showPointModal = false;
      }
      let _this = this;
      this.projectMarker = [];
      this.projectIds = [];
      // 用来调整可视区域的
      let locations = [];
      for (var i = 0; i < projectList.length; i++) {
        this.projectIds.push(projectList[i].id);
        let tempProjectObject = projectList[i];
        // var point = new BMap.Point(projectList[i].lon, projectList[i].lat)
        let point = new T.LngLat(projectList[i].lon, projectList[i].lat);
        // 判断不同类型监测项目显示不同的图标 (是否完结、项目类型、是否有异常传感器)
        let modeName = "";
        if (this.rowData.id) {
          modeName = "active";
        } else {
          modeName = "normal";
        }
        console.log("图片图片图片图片", modeName, projectList[i]);

        let iconX = 22;
        let iconY = 31;
        let myIcon = new T.Icon({
          iconUrl: "/images/real-time/" + modeName + ".png",

          iconSize: new T.Point(iconX, iconY),
          // iconAnchor: new T.Point(22, 31),
        });
        // 创建标注对象并添加到地图
        let marker = new T.Marker(point, {
          icon: myIcon,
        });
        locations.push(marker.getLngLat());
        // 添加监测项目简称label
        let label = new T.Label({
          text: projectList[i].shortName,
          position: point,
          offset: new T.Point(-60, -30),
        });

        // 保存数据
        marker.pointData = tempProjectObject;
        marker.labelData = label;
        // 添加标注
        this.Map.addOverLay(marker);
        // 加载到三维里面
        this.addFeature(
          window.viewer,
          [
            {
              longitude: point.getLng(),
              latitude: point.getLat(),
              icon: {
                image: "/images/real-time/" + modeName + ".png",
                width: iconX,
                height: iconY,
              },
              data: tempProjectObject,
            },
          ],
          {}
        );
        // 为当前点添加事件(钦州市青年水闸屏蔽图标点击事件)
        marker.addEventListener("click", function (data) {
          _this.clickMarkerProject = this;
          if (this.waveItem) {
            _this.clickMarkerProjectHaseWave = true;
          } else {
            _this.clickMarkerProjectHaseWave = false;
          }
          _this.clickProjectMarker(this.pointData, this);
          _this.setProjectListActive(this.pointData.id, true);
        });

        // 为当前点添加鼠标浮动事件
        marker.addEventListener("mouseover", function (data) {
          if (_this.Map.getZoom() < 14) {
            let label = data.target.labelData;
            // 添加标题
            _this.Map.addOverLay(label);
          }
        });
        // 为当前点添加鼠标离开事件
        marker.addEventListener("mouseout", function (data) {
          if (_this.Map.getZoom() < 14) {
            let label = data.target.labelData;
            // 添加标题
            _this.Map.removeOverLay(label);
          }
        });
        this.projectMarker.push(marker);

        // 有新警情的项目-添加水波纹
        // this.waveList = [];
        // if (projectList[i].notEndWarn === 0) {
        //   var waveItem = null;
        //   for (var j = 0, len = this.waveList.length; j < len; j++) {
        //     if (!this.waveList[j].hasMarker) {
        //       waveItem = this.waveList[j];
        //       break;
        //     }
        //   }
        //   if (!waveItem) {
        //     waveItem = this.addWaveToMarker(this.Map);
        //   }
        //   marker.waveItem = waveItem;
        //   // 绑定标记
        //   waveItem.hasMarker = true;
        //   this.displaySingleWave(marker, true);
        // }
      }
      if (locations.length > 0) {
        // alert('asd');
        let zone = this.Map.getViewport(locations);
        this.Map.centerAndZoom(zone.center, this.mapZoom);
        this.$nextTick(() => {
          this.mapZoom = zone.zoom;
        });
      }
    },
    returnProjectListItem(id) {
      for (var i = 0; i < this.projectList.length; i++) {
        if (id === this.projectList[i].id) {
          return this.projectList[i];
          break;
        }
      }
    },
    // 清空所有marker
    clearAllMarker(id) {
      let allOverlay = this.Map.getOverlays();
      if (id) {
        for (let i = 0; i < allOverlay.length; i++) {
          if (
            (allOverlay[i].getType() === 1 || allOverlay[i].getType() === 2) &&
            allOverlay[i].pointData.id === id
          ) {
            this.Map.removeOverLay(allOverlay[i]);
          }
        }
      } else {
        for (let i = 0; i < allOverlay.length; i++) {
          if (allOverlay[i].getType() === 1 || allOverlay[i].getType() === 2) {
            if (!allOverlay[i].options.notClear) {
              this.Map.removeOverLay(allOverlay[i]);
            }
          }
        }
        // 清除聚合点
        if (this.markerClusterer) {
          this.markerClusterer.clearMarkers(this.projectMarker);
        }
        this.projectMarker.map((item, index) => {
          this.displaySingleWave(item, false);
        });
      }
      // linx 清理三维的图标
      this.clearAllMarkerForTripDim();
    },
    clearAllMarkerForTripDim() {
      let dataSources = window.viewer.dataSources.getByName("defaultFeatures");
      if (dataSources.length > 0) {
        for (let i = 0; i < dataSources.length; i++) {
          window.viewer.dataSources.remove(dataSources[i]);
        }
      }
    },
    /*
     * 为地图添加水波纹
     */
    addWaveToMarker(mp) {
      var waveWapper = document.createElement("div"),
        waveChild1 = document.createElement("div"),
        waveChild2 = document.createElement("div");

      waveWapper.className = "wave-wapper";
      waveChild1.className = "wave-item wave-item-inside";
      waveChild2.className = "wave-item wave-item-outside";
      waveWapper.appendChild(waveChild1);
      waveWapper.appendChild(waveChild2);

      mp.getPanes().markerPane.appendChild(waveWapper);
      this.waveList.push(waveWapper);

      return waveWapper;
    },
    displaySingleWave(marker, isShow) {
      var cssValue = isShow ? "block" : "none";
      if (marker && marker.waveItem) {
        if (isShow) {
          var pixel = this.Map.lngLatToLayerPoint(marker.getLngLat());
          marker.waveItem.style.left = pixel.x - 35 + "px";
          marker.waveItem.style.top = pixel.y - 20 + "px";
          marker.waveItem.className = "wave-wapper levelOne";
          marker.waveItem.style.display = cssValue;
        } else {
          marker.waveItem.remove();
        }
      }
    },
    /*
     * 获取监测项目 - 测点列表
     */
    getProjectPointList() {
      this.loading = true;
      getProjectPointList(this.projectId)
        .then((req) => {
          debugger
          console.log(req.data, " 获取监测项目 - 测点列表");
          if (!req.data.length){
            this.showPointModal=false
            this.loading = false
            this.projectSensorList = [];
            this.curPointList = [];
            return
          }
          // 处理数据，如果存在异常则提示
          if (req.code === 200) {
            req.data = req.data || [];

            this.parseProjectPointListForMap(this.projectId, req.data);
            this.handleChangeTab(this.curPointList[0], 0);
          } else {
            this.$Message.error(req.msg || "获取监测站失败!");
          }

          this.loading = false;
        })
        .catch((err) => {
          this.loading = false;
          this.$Message.error("服务器异常，请稍后再试!");
        });
    },
    /*
     * 将监测项目-测点绘制到地图上
     */
    parseProjectPointListForMap(projectId, pointList) {
      this.loading = true;
      this.curPointList = pointList;
      this.projectSensorList = [];
      let _this = this;
      this.projectSensorList.push(this.rowData || this.monitorPointInfo);
      if (!this.rowData.id && !this.monitorPointInfo.projectId) return;
      // 判断不同状态的传感器显示不同的图标
      let modeName = "";
      if (this.rowData.id || this.monitorPointInfo.projectId) {
        modeName = "active";
      } else {
        modeName = "normal";
      }
      let point = new T.LngLat(
        this.monitorPointInfo.lon || this.rowData.lon,
        this.monitorPointInfo.lat || this.rowData.lat
      );
      let myIcon = new T.Icon({
        iconUrl: "/images/real-time/" + modeName + ".png",
        iconSize: new T.Point(27, 38),
      });

      // 创建标注对象并添加到地图
      let marker = new T.Marker(point, {
        icon: myIcon,
      });
      // 添加监测项目简称label
      let label = new T.Label({
        text: this.rowData.title,
        position: point,
        offset: new T.Point(-50, -40),
      });
      // 保存数据
      marker.labelData = label;
      marker.pointData = this.rowData;

      // 添加标注
      this.Map.addOverLay(marker);
      this.addFeature(
        window.viewer,
        [
          {
            longitude: point.getLng(),
            latitude: point.getLat(),
            icon: {
              image: "/images/real-time/" + modeName + ".png",
              width: 27,
              height: 38,
            },
            data: this.rowData,
          },
        ],
        {
          group: "pointClickDraw",
        }
      );

      marker.addEventListener("mouseover", function (data) {
        let label = data.target.pointData.label;
        // 添加标题
        _this.Map.addOverLay(label);
      });
      marker.addEventListener("mouseout", function (data) {
        let label = data.target.pointData.label;
        console.log(data.target.pointData.label, "mouseover");
        // 添加标题
        _this.Map.removeOverLay(label);
      });
      this.clickMarkerList.push(marker);
      this.loading = false;
    },

    returnSensorListItem(id) {
      for (let i = 0; i < this.projectSensorList.length; i++) {
        if (id === this.projectSensorList[i].pointId) {
          return this.projectSensorList[i];
          break;
        }
      }
    },

    /**
     * 点击监测项目触发事件
     */
    clickProjectMarker(data, marker) {
      console.log(data, "点击监测项目触发事件点击监测项目触发事件");
      this.clickProjectObject = data;
      this.projectId = data.id;
      this.projectIds = [];
      this.projectIds.push(data.id);
      this.htmlData = data.content;
      if (marker) {
        if (marker.labelData) {
          this.Map.removeOverLay(marker.labelData);
        }
        this.Map.removeOverLay(marker);
      }

      this.clickMarkerList.forEach((item) => {
        if (item.labelData) {
          this.Map.removeOverLay(item.labelData);
        }
        this.Map.removeOverLay(item);
      });
      this.clickMarkerList = [];

      // 三维地图上的之前打开过的标记删除
      let dataSources = window.viewer.dataSources.getByName("pointClickDraw");
      if (dataSources.length > 0) {
        for (let i = 0; i < dataSources.length; i++) {
          window.viewer.dataSources.remove(dataSources[i]);
        }
      }
      // 获取监测点数据
      this.getDetailProject(data.id);
      // // 重新定位中心点
      let point = new T.LngLat(data.lon, data.lat);
      // // 绘制监测项目区域
      this.drawPolygon(point, data.locationArea);
      // 查询测点标记测点坐标
      this.getProjectPointList();

      this.showPointModal = true;
    },
    /**
     * 点击监测项目-测点触发事件
     */
    clickPointMarker(data) {
      let hasAlredyOpen = false;
      this.loading = true;
      for (let i = 0; i < this.chartsModalList.length; i++) {
        if (this.chartsModalList[i].sensorId === data.sensorId) {
          hasAlredyOpen = true;
          break;
        }
      }
      if (hasAlredyOpen) {
        this.loading = false;
        return;
      }
      let tempObject = {};
      Object.assign(tempObject, data);
      tempObject.showModal = true;
      tempObject.chartOption = {};

      tempObject.timeList = [];
      let date = new Date();
      let lastDate = new Date(date.getTime() - 24 * 60 * 60 * 1000 * 2);
      let todayStartDay =
        lastDate.getFullYear() +
        "-" +
        (lastDate.getMonth() + 1) +
        "-" +
        lastDate.getDate() +
        " 00:00:00";
      let todayEndDay =
        date.getFullYear() +
        "-" +
        (date.getMonth() + 1) +
        "-" +
        date.getDate() +
        " 24:00:00";
      tempObject.timeList.push({
        timeTitle: "近三天",
        isActive: true,
        timeArray: [todayStartDay, todayEndDay],
      });

      let weekDate = new Date(date.getTime() - 24 * 60 * 60 * 1000 * 7);
      let weekDateStartDay =
        weekDate.getFullYear() +
        "-" +
        (weekDate.getMonth() + 1) +
        "-" +
        weekDate.getDate() +
        " 00:00:00";
      let weekDateEndDay =
        weekDate.getFullYear() +
        "-" +
        (weekDate.getMonth() + 1) +
        "-" +
        weekDate.getDate() +
        " 24:00:00";
      tempObject.timeList.push({
        timeTitle: "近一周",
        isActive: false,
        timeArray: [weekDateStartDay, todayEndDay],
      });

      let monthDate = new Date(date.getTime() - 24 * 60 * 60 * 1000 * 30);
      let monthDateStartDay =
        monthDate.getFullYear() +
        "-" +
        (monthDate.getMonth() + 1) +
        "-" +
        monthDate.getDate() +
        " 00:00:00";
      let monthDateEndDay =
        monthDate.getFullYear() +
        "-" +
        (monthDate.getMonth() + 1) +
        "-" +
        monthDate.getDate() +
        " 24:00:00";
      tempObject.timeList.push({
        timeTitle: "近一个月",
        isActive: false,
        timeArray: [monthDateStartDay, todayEndDay],
      });
      this.getSensorChartData(tempObject, tempObject.timeList[0].timeArray);
    },
    // 点击了视频图标
    videoClicked(video) {
      let tempId = encodeURIComponent(video.id);
      const { href } = this.$router.resolve({
        name: "viewVideo",
        query: {
          id: tempId,
          type: 1,
        },
      });
      window.open(href);
    },
    /*
     * 筛选
     */
    toggleScreen() {
      this.closeMessageList();
      if (this.showScreen) {
        this.showScreen = false;
      } else {
        this.showScreen = true;
      }
    },
    toggleLayers() {
      if (this.showLayer) {
        this.showLayer = false;
      } else {
        this.showLayer = true;
      }
    },
    toggleMapDim() {
      if (this.isTripDim) {
        this.isTripDim = false;
      } else {
        if (!window.viewer) {
          this.initTripMap();
        }
        this.isTripDim = true;
      }
    },
    toogleWarnMessage() {
      this.closeScreen();
      this.showWarnMessageList = !this.showWarnMessageList;
    },

    closeScreen() {
      this.showScreen = false; // 关闭筛选框
      this.showLayer = false;
    },
    closeMessageList() {
      this.showWarnMessageList = false;
    },
    clickPointListItem(data) {
      // console.log("MMMMMMMMMMMMMMMMMMMMMMMMMMMM");
      // console.log(data);
      debugger
      let clickmaer = null;
      for (let i = 0; i < this.projectMarker.length; i++) {
        let pointData = this.projectMarker[i].pointData;
        let id = pointData.id || "";
        if (id && id === data.id) {
          clickmaer = this.projectMarker[i];
          break;
        }
      }
      if (clickmaer) {
        this.clickMarkerProject = clickmaer;
        if (clickmaer.waveItem) {
          this.clickMarkerProjectHaseWave = true;
        } else {
          this.clickMarkerProjectHaseWave = false;
        }
        this.clickProjectMarker(data, clickmaer);
      } else {
        this.clickProjectMarker(data);
      }
      this.setProjectListActive(data.id, false);
    },
    setProjectListActive(projectId, scroll) {
      debugger
      this.projectList.forEach((item) => {
        if (item.id === projectId) {
          item.isActive = true;
          // 左侧也定位到相关区域
          if (scroll) {
            let offseTop = this.$refs[item.id][0].offsetTop || 0;
            let dom = this.$refs.dangerPointListContent;
            dom.scrollTop = offseTop - 100;
          }
        } else {
          item.isActive = false;
        }
      });
    },
    /**
     * 画监测项目区域
     */
    drawPolygon(point, data) {
      let areaData = null;
      let pointArr = [];
      try {
        areaData = JSON.parse(data);
      } catch (err) {
        console.log(err);
      }
      if (areaData && areaData.length > 0) {
        for (let i = 0; i < areaData.length; i++) {
          let areaPoint = new T.LngLat(areaData[i].lng, areaData[i].lat);
          pointArr.push(areaPoint);
        }
        // 创建多边形
        // let polygon = new T.Polygon(pointArr, {
        //   color: "#0042A1",
        //   weight: 2,
        //   opacity: 1,
        //   lineStyle: "solid",
        // });
        // this.clickMarkerList.push(polygon);
        // this.Map.addOverLay(polygon); // 增加多边形区域

        // region linx 三维地图绘制区域
        this.addFeature(window.viewer, pointArr, {
          formatter: function (items) {
            let arry = [];
            items.forEach((lnglat) => {
              arry.push([lnglat.getLng(), lnglat.getLat()]);
            });
            return arry;
          },
          type: "polygon",
          group: "pointClickDraw",
        });
        // endregion
      }
      if (pointArr.length > 0) {
        debugger
        let optimalZoom = this.Map.getViewport(pointArr).zoom;
        if (optimalZoom > 18) {
          optimalZoom = 18;
        }
        this.Map.centerAndZoom(point, optimalZoom);
      } else {
        this.Map.centerAndZoom(point, this.mapZoom);
      }

      // region linx 三维地图定位和绘制区域
      window.viewer.camera.flyTo({
        destination: Cesium.Cartesian3.fromDegrees(
          point.getLng(),
          point.getLat() - 0.015,
          2000
        ),
        orientation: {
          heading: Cesium.Math.toRadians(0),
          pitch: Cesium.Math.toRadians(-45),
          roll: Cesium.Math.toRadians(0),
        },
      });
      // endregion
    },
    /**
     * 清空地图上的marker点
     */
    clearAll() {
      // 清除聚合点
      if (this.markerClusterer) {
        this.markerClusterer.clearMarkers(this.projectMarker);
      }
      this.projectMarker.map((item, index) => {
        this.displaySingleWave(item, false);
      });
    },
    /**
     * 跳转到监测项目详情
     */
    toProjectDetail() {
      this.$router.push({
        path: "/realtimeManager/realtime-monitorproject",
      });
    },

    /**
     * 初始化行政区域
     */
    addDistrict(BMap, bdaryName) {
      let that = this;
      var bdary = new BMap.Boundary();
      bdary.get(bdaryName, function (rs) {
        // 获取行政区域
        var count = rs.boundaries.length; // 行政区域的点有多少个
        if (count === 0) {
          alert("未能获取当前输入行政区域");
          return;
        }
        var pointArray = [];
        for (var i = 0; i < count; i++) {
          var ply = new BMap.Polygon(rs.boundaries[i], {
            // 建立多边形覆盖物
            strokeWeight: 2,
            strokeColor: "#ff0000",
          });
          that.Map.addOverLay(ply); // 添加覆盖物
          pointArray = pointArray.concat(ply.getPath());
        }
        that.Map.setViewport(pointArray); // 调整视野
      });
    },
    /**
     * 条件筛选---全选/取消全选
     */
    handleCheckAll() {
      if (this.indeterminate) {
        this.checkAll = false;
      } else {
        this.checkAll = !this.checkAll;
      }
      this.indeterminate = false;
      if (this.checkAll) {
        this.checkAllGroup = this.checkAllListMethod;
        this.monitorMode = "";
        let count = this.checkAllGroup.length - 1;
        this.checkAllGroup.map((item, index) => {
          this.monitorMode += item;
          if (index < count) {
            this.monitorMode += "|";
          }
        });
        this.getProjectList();
      } else {
        this.checkAllGroup = [];
        this.monitorMode = "";
        // 清空所有marker
        this.clearAllMarker();
        // 判断如果是已经打开过详情框那么就把详情框关闭
        if (this.showPointModal) {
          this.showPointModal = false;
        }
        this.projectList = [];
      }
    },
    handleCheckAllMonitorObject() {
      if (this.indeterminateMonitorObject) {
        this.monitorObjectCheckAll = false;
      } else {
        this.monitorObjectCheckAll = !this.monitorObjectCheckAll;
      }
      this.indeterminateMonitorObject = false;
      if (this.monitorObjectCheckAll) {
        this.checkAllMonitorObjectGroup = this.checkAllListObject;
        this.monitorObject = "";
        let count = this.checkAllMonitorObjectGroup.length - 1;
        this.checkAllMonitorObjectGroup.map((item, index) => {
          this.monitorObject += item;
          if (index < count) {
            this.monitorObject += "|";
          }
        });
      } else {
        this.checkAllMonitorObjectGroup = [];
        this.monitorObject = "";
      }
    },
    /**
     * 条件筛选---选择或取消某一项时触发
     */
    checkGroupChange(value) {
      if (value.length > 0) {
        // 先清空所有marker
        this.clearAllMarker();
        if (value.length === this.checkAllListMethod.length) {
          this.checkAll = true;
          this.indeterminate = true;
        }
        this.monitorMode = "";
        let count = value.length - 1;
        value.map((item, index) => {
          this.monitorMode += item;
          if (index < count) {
            this.monitorMode += "|";
          }
        });
        this.getProjectList();
      } else {
        this.checkAllGroup = [];
        this.checkAll = false;
        this.indeterminate = false;
        this.clearAllMarker();
        this.projectList = [];
      }
    },

    /**
     * 图层选择变化
     */
    checkLayerChange(value) {
      this.Map.removeLayer(window.geoLayer);
      this.Map.removeLayer(window.hypLayer);
      // var layers=this.Map.getLayers()
      // console.log(layers)
      // layers.push(this.wmsLayer)
      window.geoMap.show = false;
      window.hypMap.show = false;
      this.checkAllLayer.forEach((item) => {
        this.Map.addLayer(window[item + "Layer"]);
        window[item + "Map"].show = true;
      });
      // this.Map.addLayer(this.wmsLayer)
    },

    checkGroupMonitorObjetChange(value) {
      if (value.length > 0) {
        if (value.length === this.checkAllListObject.length) {
          this.monitorObjectCheckAll = true;
          this.indeterminateMonitorObject = true;
        }
        this.monitorObject = "";
        let count = value.length - 1;
        value.map((item, index) => {
          this.monitorObject += item;
          if (index < count) {
            this.monitorObject += "|";
          }
        });
      } else {
        this.checkAllMonitorObjectGroup = [];
        this.monitorObjectCheckAll = false;
        this.indeterminateMonitorObject = false;
      }
    },
    drawAreaBorder(areaName) {
      if (areaName) {
        this.addDistrict(this.BBMap, areaName);
      }
    },
    resetTypeFilter() {
      this.checkAllGroup = this.checkAllListMethod.filter(
        (item) => item === "1"
      );
      this.checkAll = false;
      this.indeterminate = false;

      this.monitorMode = "1";
      this.monitorObject = "";
      this.getProjectList();
    },
    getMonitorMethodList() {
      getMonitorMode()
        .then((req) => {
          if (req.code === 200) {
            let data = req.data || [];
            this.monitorMethodList = [];
            this.checkAllGroup = [];
            this.checkAllListMethod = [];
            data.map((item) => {
              this.checkAllListMethod.push(item.value);
              if (item.value === "1") {
                this.checkAllGroup.push(item.value);
              }
              this.checkAll = false;
            });
            this.monitorMethodList = data;
          } else {
            this.$Message.error(req.msg || "获取监测方式失败");
          }
        })
        .catch((err) => {
          console.error(err);
          this.$Message.error("服务器异常，请稍后重试");
        });
    },
    dealAbnormalSensorData(data) {
      data.map((item) => {
        item.sensorList = item.sensorList || [];
        item.sensorList.map((itemSensor) => {
          this.abnormalSensorList.push({
            projectId: item.projectId,
            projectName: item.projectName,
            sensorName: itemSensor.sensorName,
            exceptionTime: itemSensor.exceptionTime,
            id: itemSensor.sensorId,
          });
        });
      });
    },
    abnormalSensorClick(data) {
      this.$refs.AbnormalSensorForm.viewData(data);
    },
    warningItemClick(data) {
      this.$router.push({
        path: "/alarmmanager/watchhall",
        name: "watchhallManager",
        params: {
          projectId: data.id,
        },
      });
    },
    videoItemClick(data) {
      // 点击了左侧监测站中的某个摄像头
    },
    getWarningList() {
      if (this.userId) {
        this.getNewWarnList({})
          .then((req) => {
            // console.log(req.data, "预警列表");
            if (req.code === 200) {
              let data = req.data || [];
              this.warningCountNumber = data.length;

              if (this.warningCountNumber && this.warningCountNumber > 0) {
                this.blinkWarnIcon();
              } else {
                this.warnBlick = true;
              }
              this.warningList = [];
              this.warningList = data;
              // setTimeout(() => {
              //   this.getWarningList();
              // }, 5000);
            } else {
              // setTimeout(() => {
              //   this.getWarningList();
              // }, 5000);
            }
          })
          .catch((err) => {
            // setTimeout(() => {
            //   this.getWarningList();
            // }, 5000);
            console.error(err);
          });
      }
    },
    blinkWarnIcon() {
      if (!this.timer) {
        this.timer = setInterval(() => {
          this.warnBlick = !this.warnBlick;
        }, 5000);
      }
    },
    getWarnCardLevelClass(warnType) {
      let warnClass = "levelOne";
      switch (warnType) {
        case "1":
          warnClass = "levelOne";
          break;
        case "2":
          warnClass = "levelTwo";
          break;
        case "3":
          warnClass = "levelThree";
          break;
        case "4":
          warnClass = "levelFour";
          break;
        default:
          warnClass = "levelOne";
          break;
      }
      return warnClass;
    },
    shoWarnCardClick() {
      this.showWarnCard = !this.showWarnCard;
    },
    handlescroll(e) {
      var type = e.type;
      let delta = 0;
      if (type === "DOMMouseScroll" || type === "mousewheel") {
        delta = e.wheelDelta ? e.wheelDelta : -(e.detail || 0) * 40;
      }
      this.handleScroll(delta);
    },
    handleScroll(offset) {
      const outerWidth = this.$refs.scrollOuter.offsetWidth;
      const bodyWidth = this.$refs.scrollBody.offsetWidth;
      if (offset > 0) {
        this.tagBodyLeft = Math.min(0, this.tagBodyLeft + offset);
      } else {
        if (outerWidth < bodyWidth) {
          if (this.tagBodyLeft < -(bodyWidth - outerWidth)) {
            this.tagBodyLeft = this.tagBodyLeft;
          } else {
            this.tagBodyLeft = Math.max(
              this.tagBodyLeft + offset,
              outerWidth - bodyWidth
            );
          }
        } else {
          this.tagBodyLeft = 0;
        }
      }
    },
    clickPeojectSensorItem(data) {
      this.curPointId = data.pointId;
      this.curContentCode = data.contentCode;
      this.curPointNo = data.pointNo;
      this.curContentName = data.contentName;
      this.tsensorItemCanClick = false;
      let pointIdsTemp = [];
      pointIdsTemp.push(data.pointId);
      this.projectSensorList.map((item, index) => {
        if (
          item.pointId === data.pointId &&
          item.contentCode === data.contentCode
        ) {
          this.$set(this.projectSensorList[index], "active", true);
        } else {
          this.$set(this.projectSensorList[index], "active", false);
        }
      });
      let senorChartSearch = {
        projectId: this.projectId,
        contentNo: data.contentCode,
        pointIds: pointIdsTemp,
        results: ["COLLECT_DATA"],
        startTime: null,
        endTime: null,
      };
      this.isShowSensorChart = true;
      this.getSensorChartData(senorChartSearch);
    },
    caculScrollLeft(val) {
      let aIndex = 0;
      if (val) {
        for (let i = 0; i < this.projectSensorList.length; i++) {
          if (this.projectSensorList[i].active) {
            aIndex = i;
            break;
          }
        }
        const outerWidth = this.$refs.scrollOuter.offsetWidth;
        const bodyWidth = this.$refs.scrollBody.offsetWidth;
        if (outerWidth < bodyWidth) {
          let itemArr = document.getElementsByClassName("scroll-item");
          var widthTotal = 0;
          var itemIndex = 0;
          for (var key in itemArr) {
            itemIndex++;
            if (itemIndex - 1 < aIndex) {
              if (itemArr[key].style) {
                let widthNum = Number(
                  window.getComputedStyle(itemArr[key]).width.split("px")[0]
                );
                widthTotal += widthNum;
              }
            } else {
              break;
            }
          }
          this.tagBodyLeft = Math.max(-widthTotal, outerWidth - bodyWidth); // 727
        } else {
          this.tagBodyLeft = 0;
        }
      }
    },
    toggleSensorChart() {
      this.isShowSensorChart = !this.isShowSensorChart;
    },
    /**
     * 显示测点分布图
     **/
    showPointDistribution() {
      this.$refs.showPointDistribution.viewData(
        this.projectId,
        this.measuringPointPic
      );
    },
    clickShow3DModal() {
      this.$Message.info("正在开发中...");
    },
    getSensorChartData(dataObject, timeArray) {
      // 先初始化图表
      var chart = window.CHARTS_LOADER[dataObject.contentCode];
      let options = chart.getOptions();
      let echarts = [];
      options.forEach((item) => {
        if (
          item["chartNo"] === "profile" &&
          !this.query.pointIds &&
          this.query.pointIds.length === 0
        ) {
          // 传感器维度不展示刨面图
        } else {
          echarts.push(item);
        }
      });
      for (let i = 0; i < echarts.length; i++) {
        let results = ["collectData"];
        if (dataObject.contentCode === "JY") {
          results = ["hourRain"];
        }
        getChartDatas({
          chartNo: echarts[i].chartNo,
          sensors: [
            {
              sensorId: dataObject.sensorId,
              contentNo: dataObject.contentCode,
              resultList: results,
              projectId: dataObject.projectId,
            },
          ],
          startDate: timeArray[0],
          endDate: timeArray[1],
        })
          .then((res) => {
            if (res.code === 200) {
              let data = res.data;
              this.chartDataProcessing(chart, dataObject, echarts[i], data);
              this.loading = false;
            } else {
              this.chartsModalList.push(dataObject);
              this.loading = false;
            }
          })
          .catch((err) => {
            this.chartsModalList.push(dataObject);
            this.loading = false;
          });
      }
    },
    changeSensorChartData(dataObject, timeArray) {
      this.loading = true;
      let results = ["collectData"];
      if (dataObject.contentCode === "JY") {
        results = ["hourRain"];
      }
      getChartDatas({
        chartNo: dataObject.chartOption.chartNo,
        sensors: [
          {
            sensorId: dataObject.sensorId,
            contentNo: dataObject.contentCode,
            resultList: results,
            projectId: dataObject.projectId,
          },
        ],
        startDate: timeArray[0],
        endDate: timeArray[1],
      })
        .then((res) => {
          if (res.code === 200) {
            let data = res.data;
            dataObject.chartOption.series.forEach((item) => {
              item.data = [];
            });
            dataObject.chartOption.xAxis = {};
            dataObject.chartOption.yAxis = {};

            let resultOptionList = [
              {
                code: "collectData",
                label: "采集值",
              },
            ];
            let yAriesName =
              "采集值(" + this.getUnitName(dataObject.contentCode) + ")";
            if (dataObject.contentCode === "JY") {
              resultOptionList = [
                {
                  code: "hourRain",
                  label: "雨强",
                },
              ];
              yAriesName = "雨强(mm)";
            }
            dataObject.chart.formatSeries(
              dataObject.chartOption,
              data,
              resultOptionList
            );
            dataObject.chartOption.grid = {
              containLabel: true,
              left: 50,
              right: 60,
              top: 30,
              bottom: 30,
            };
            dataObject.chartOption.yAxis.name = yAriesName;
            if (dataObject.chartOption.legend) {
              dataObject.chartOption.legend.show = false;
            }
          } else {
            if (dataObject.chartOption.legend) {
              dataObject.chartOption.legend.show = false;
            }
            dataObject.chartOption.series.forEach((item) => {
              item.data = [];
            });
            dataObject.chartOption.xAxis = {};
            dataObject.chartOption.yAxis = {};
          }
          this.loading = false;
        })
        .catch((err) => {
          if (dataObject.chartOption.legend) {
            dataObject.chartOption.legend.show = false;
          }
          dataObject.chartOption.series.forEach((item) => {
            item.data = [];
          });
          dataObject.chartOption.xAxis = {};
          dataObject.chartOption.yAxis = {};
          this.loading = false;
        });
    },
    getUnitName(unit) {
      let unitName = "mm";
      switch (unit) {
        case "QJ":
          unitName = "°";
          break;
        case "TL":
          unitName = "KPa";
          break;
        case "ZL":
          unitName = "kN";
          break;
        case "WJ":
          unitName = "kN";
          break;
        case "JSD":
          unitName = "g";
          break;
        case "DW":
          unitName = "m";
          break;
        case "KY":
          unitName = "KPa";
          break;
        case "SY":
          unitName = "KPa";
          break;
        case "TH":
          unitName = "％";
          break;
        case "WD":
          unitName = "°C";
          break;
        default:
          unitName = "mm";
          break;
      }
      return unitName;
    },
    chartDataProcessing(chart, sensorData, option, data) {
      option.grid = {
        containLabel: true,
        left: 50,
        right: 60,
        top: 30,
        bottom: 30,
      };
      let resultOptionList = [
        {
          code: "collectData",
          label: "采集值",
        },
      ];
      let yAriesName =
        "采集值(" + this.getUnitName(sensorData.contentCode) + ")";
      if (sensorData.contentCode === "JY") {
        resultOptionList = [
          {
            code: "hourRain",
            label: "雨强",
          },
        ];
        yAriesName = "雨强(mm)";
      }
      chart.formatSeries(option, data, resultOptionList);
      option.yAxis.name = yAriesName;
      sensorData.chartOption = option;
      sensorData.chart = chart;
      sensorData.chartOption.legend.show = false;
      this.chartsModalList.push(sensorData);
    },
    chartTimeChange(index, timeData) {
      let dataObject = this.chartsModalList[index];
      this.changeSensorChartData(dataObject, timeData.timeArray);
      dataObject.timeList.forEach((item) => {
        item.isActive = false;
      });
      timeData.isActive = true;
    },
    modalCancel(index) {
      this.$nextTick(() => {
        this.chartsModalList.splice(index, 1);
      });
    },
    // 打开自动预警弹窗回显数据
    warningMessageItemClick(data, index) {
      this.$notice().show(data, index);
      bus.$on("message", (e) => {
        console.log(e, "message");

        let current = this.warningList.findIndex((item) => item.id == e);
        console.log(current, "current");
        if (current >= 0) {
          this.warningList.splice(current, 1);
          this.warningCountNumber = this.warningList.length;

          if (this.warningCountNumber && this.warningCountNumber > 0) {
            this.blinkWarnIcon();
          } else {
            this.warnBlick = true;
          }
        }
      });
    },
    judeChartHasData(options) {
      if (options.xAxis.data && options.xAxis.data.length > 0) {
        if (options.series && options.series.length > 0) {
          let hasData = false;
          for (let i = 0; i < options.series.length; i++) {
            if (options.series[i].data && options.series[i].data.length > 0) {
              hasData = true;
              break;
            }
          }
          return hasData;
        } else if (options.series.data && options.series.data.length > 0) {
          return true;
        } else {
          return false;
        }
      } else {
        return false;
      }
    },
    clickBadSensorNum() {
      this.$refs.sensorModal.showSensorModal();
    },
    areaTreeCheckChange(selectedNodes, node) {
      this.areaCode = "";
      this.dealAreaTreeResult(selectedNodes);
    },
    dealAreaTreeResult(nodes) {
      const length = nodes.length;
      nodes.forEach((item, index) => {
        if (item.areaCode !== this.areaCodeParent) {
          this.areaCode += item.areaCode;
          if (index < length - 1) {
            this.areaCode += "|";
          }
        }
      });
    },
    getAreaListData() {
      this.areaTreeData = [];
      getAreaList(this.areaCodeParent)
        .then((res) => {
          if (res.code === 200) {
            const data = res.data || [];
            if (data.length > 0) {
              this.areaTreeData.push({
                areaCode: this.areaCodeParent,
                title: this.areaNameParent,
                expand: true,
                children: [],
              });
              data.forEach((item) => {
                this.areaTreeData[0].children.push({
                  areaCode: item.value,
                  level: item.level,
                  title: item.label,
                });
              });
            }
          }
        })
        .catch((err) => {});
    },
    getMonitorObject() {
      getMonitorObject()
        .then((res) => {
          if (res.code === 200) {
            let data = res.data || [];
            this.monitorObjectList = [];
            this.checkAllMonitorObjectGroup = [];
            this.checkAllListObject = [];
            this.monitorObjectCheckAll = false;
            data.map((item) => {
              this.checkAllListObject.push(item.value);
            });
            this.monitorObjectList = data;
          } else {
          }
        })
        .catch((err) => {});
    },
    resetAreaFilter() {
      // 重置灾害类型
      this.monitorObject = "";
      this.checkAllMonitorObjectGroup = [];
      this.monitorObjectCheckAll = false;
      this.indeterminateMonitorObjects = false;
      // 重置地区选择
      this.areaCode = "";
      this.resetTreeData(this.areaTreeData);
      // 重置输入条件
      this.searchValue = "";
      this.getProjectList();
    },
    resetTreeData(data) {
      data.forEach((item) => {
        if (item.checked) {
          item.checked = false;
        }
        if (item.selected) {
          item.selected = false;
        }
        if (item.children) {
          this.resetTreeData(item.children);
        }
      });
    },
    openSensorCruve(sensor) {
      this.$refs.dataAnalysisForm.showSensor(
        sensor.contentCode,
        sensor.sensorId
      );
    },
  },
  mounted() {
    // this.$notice().show();
    this.initMap(); // 初始化地图
    this.addWmsLayer(
      "nanchong:jilingjiang",
      "http://server.clzytech.com:8861/geoserver/nanchong/wms",
      10
    );
    this.getMonitorMethodList(); // 获取监测方式
    this.getWarningList(); // 获取警情列表
    // var timer1 = setInterval(() => {
    //   this.getWarningList();
    // }, 10000);
    // this.$once("hook:beforeDestroy", () => {
    //   clearInterval(timer1);
    // });
    this.getAreaListData(); // 获取行政区域
    this.getMonitorObject(); // 获取灾害类型
    // 获取实时数据
    this.getRealTimeList();
    var timer2 = setInterval(() => {
      this.getRealTimeList();
    }, 600000);
    this.$once("hook:beforeDestroy", () => {
      clearInterval(timer2);
    });
    if (this.$route.params.projectId) {
      this.analyseProjectId = this.$route.params.projectId;
      this.analyseProjectType = this.$route.params.projectType;
      this.curPointId = this.$route.params.curPointId;
      this.sensorTabLeft = this.$route.params.sensorTabLeft;
    }
    this.defaultOption.autoPlay =
      this.monitoringPointList.length > 7 ? true : false;
  },
  beforeRouteLeave(to, from, next) {
    this.chartsModalList.forEach((item) => {
      item.showModal = false;
    });
    this.$nextTick(() => {
      this.chartsModalList = [];
    });
    // this.countyMarker = [];
    next();
  },
  beforeDestroy() {
    window.viewer.destory();
    window.viewer = null;
    this.chartsModalList.forEach((item) => {
      item.showModal = false;
    });
    this.$nextTick(() => {
      this.chartsModalList = [];
    });
    this.countyMarker = [];
    if (this.timer) {
      clearInterval(this.timer);
    }
    eventBus.$off("new-message-coming");
  },
  filters: {
    keep(val) {
      if (val) {
        return val.toFixed(2);
      } else {
        return val == 0 ? val : "-";
      }
    },
  },
  created() {},
};
</script>

<style lang="less">
@import "./realtime-monitor.less";
</style>
