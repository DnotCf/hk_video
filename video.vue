<template>
  <div class="video-list">
    <div class="left-tree" :class="showLeft ? '' : 'hide'">
      <Input
        v-model="filterStr"
        placeholder="搜索"
        clearable
        style="width: 200px; display: table; margin: 0 auto; top: 15px"
        @on-change="onFilter"
      >
      </Input>
      <div class="left-switch-content">
        <Button
          class="left-switch-btn show"
          v-if="showLeft"
          @click.stop="switchLeftShow"
          title="收起"
        >
          <ColorIcons type="ceneirong-shouqi" :size="1"></ColorIcons>
        </Button>
        <Button
          class="left-switch-btn hide"
          v-else
          @click.stop="switchLeftShow"
          title="展开"
        >
          <ColorIcons type="ceneirong-zhankai" :size="1"></ColorIcons>
        </Button>
      </div>

      <div class="left-tree-content">
        <Tree
          :data="projectVideoList"
          show-checkbox
          v-if="showTree"
          ref="videolisttree"
          @on-select-change="onTreeSelectChange"
          @on-check-change="onTreeCheckChange"
          :empty-text="loadingText"
        ></Tree>
      </div>
    </div>
    <!-- 视频播放组件 -->
    <div class="right-video-area">
      <div class="top-filter">
        <div class="top-filter-content">
<!--          <Button  class="folter-button" :class="videoSplitNum===5?'active':''" @click.stop="switchVideoSplit(5)">
            <span class="icon-item"></span>
            <span class="icon-item"></span>
            <span class="icon-item"></span>
            <span class="icon-item"></span>
            <span class="icon-item"></span>
          </Button>
          <Button class="folter-button" :class="videoSplitNum===6?'active':''" @click.stop="switchVideoSplit(6)">
            <span class="icon-item"></span>
            <span class="icon-item"></span>
            <span class="icon-item"></span>
            <span class="icon-item"></span>
          </Button>
          <Button class="folter-button" :class="videoSplitNum===8?'active':''" @click.stop="switchVideoSplit(8)">
            <span class="icon-item"></span>
            <span class="icon-item"></span>
            <span class="icon-item"></span>
          </Button>-->
          <Button class="folter-button" :class="videoSplitNum===12?'active':''" @click.stop="switchVideoSplit(12)">
            <span class="icon-item"></span>
            <span class="icon-item"></span>
          </Button>
        </div>
      </div>
      <div class="bottom-video-area">
        <div class="video-area-content">
          <Row class="video-area-row">
            <Col :span="videoSplitNum" class="video-area-col">
                <div class="video-item">
                  <div class="video-item-content">
                    <template >
<!--                      <div class="video-title">''</div>-->
                      <div class="video-close">
                        <Button class="close-btn" @click.stop="closeVideoClick(item)" title="关闭">
                          <ColorIcons type="guanbi" class="close-icon"></ColorIcons>
                        </Button>
                      </div>
                      <div class="video-control-area" :class="showFullScreen?'full-screen':''">
                        <Button class="video-control up" @mousedown.native="mouseDownPTZControl(1)" @mouseup.native="mouseUpPTZControl(1)">上</Button>
                        <Button class="video-control down" @mousedown.native="mouseDownPTZControl(2)" @mouseup.native="mouseUpPTZControl(2)">下</Button>
                        <Button class="video-control left" @mousedown.native="mouseDownPTZControl(3)" @mouseup.native="mouseUpPTZControl(3)">左</Button>
                        <Button class="video-control right" @mousedown.native="mouseDownPTZControl(4)" @mouseup.native="mouseUpPTZControl(4)">右</Button>
                        <Button class="video-control bite320" @mousedown.native="mouseDownPTZControl(5)" @mouseup.native="mouseUpPTZControl(5)">左上</Button>
                        <Button class="video-control bite320" @mousedown.native="mouseDownPTZControl(6)" @mouseup.native="mouseUpPTZControl(6)">左下</Button>
                        <Button class="video-control bite320" @mousedown.native="mouseDownPTZControl(7)" @mouseup.native="mouseUpPTZControl(7)">右上</Button>
                        <Button class="video-control bite320" @mousedown.native="mouseDownPTZControl(8)" @mouseup.native="mouseUpPTZControl(8)">右下</Button>
                        <Button class="video-control bite320" @click="mouseDownPTZControl(9)">自动</Button>
                        <Button class="video-control bite320" @mousedown.native="mouseDownPTZControl(10)" @mouseup.native="mouseUpPTZControl(10)">放大</Button>
                        <Button class="video-control bite320" @mousedown.native="mouseDownPTZControl(11)" @mouseup.native="mouseUpPTZControl(11)">缩小</Button>
                        <Button class="video-control bite320" @click="clickCapturePic()">截图</Button>
                        <Button class="video-control bite640" @click="clickStartRecord('realplay')">开始录像</Button>
                        <Button class="video-control bite640" @click="clickStopRecord('realplay')">结束录像</Button>
                        <Button class="video-control bite320" @click="clickStartRealPlay()">播放</Button>
                        <Button class="video-control bite320" @click="clickStopRealPlay()">停止</Button>
                        <Button class="video-control bite320" @click="clickFullScreen()">全屏</Button>
                      </div>
                      <div id="divPlugin" style="width: 100%;height: 100%"></div>
                    </template>
                  </div>
                </div>
            </Col>
          </Row>
        </div>

<!--        <div class="left">
          <fieldset class="preview">
            <legend>预览</legend>
            <table cellpadding="0" cellspacing="3" border="0">
              <tr>
                <Button type="primary" :loading="loginLoading" @click="onLogin">登录</Button>
                <Button type="primary" :loading="startPlayLoading" @click="clickStartRealPlay">开始预览</Button>
                <Button type="primary" @click="clickStopRealPlay">停止预览</Button>
                <Button type="primary" @click="onLogout">退出</Button>
                <Button type="primary" @click="clickCapturePic()">抓图</Button>
                <Button type="primary" @click="clickStartRecord('realplay');">开始录像</Button>
                <Button type="primary" @click="clickStopRecord('realplay');">停止录像</Button>
                <Button type="primary" @click="clickFullScreen();" >全屏</Button>
                <Button type="primary" @click="clickEnableEZoom()" >启用电子放大</Button>
                <Button type="primary" @click="clickDisableEZoom();">禁用电子放大</Button>
                <Button type="primary" @click="clickEnable3DZoom();">启用3D放大</Button>
                <Button type="primary" @click="clickDisable3DZoom();">禁用3D放大</Button>
              </tr>
            </table>
          </fieldset>
          <fieldset class="ptz">
            <legend>云台控制</legend>
            <table cellpadding="0" cellspacing="3" border="0" class="left">
              <tr>
                <td>
                  <input type="button" class="btn" value="左上" @mousedown="mouseDownPTZControl(5);" @mouseup="mouseUpPTZControl();"/>
                  <input type="button" class="btn" value="上" @mousedown="mouseDownPTZControl(1);" @mouseup="mouseUpPTZControl();" />
                  <input type="button" class="btn" value="右上" @mousedown="mouseDownPTZControl(7);" @mouseup="mouseUpPTZControl();" />
                </td>
              </tr>
              <tr>
                <td>
                  <input type="button" class="btn" value="左" @mousedown="mouseDownPTZControl(3);" @mouseup="mouseUpPTZControl();" />
                  <input type="button" class="btn" value="自动" @onclick="mouseDownPTZControl(9);" />
                  <input type="button" class="btn" value="右" @mousedown="mouseDownPTZControl(4);" @mouseup="mouseUpPTZControl();" />
                </td>
              </tr>
              <tr>
                <td>
                  <input type="button" class="btn" value="左下" @mousedown="mouseDownPTZControl(6);" @mouseup="mouseUpPTZControl();" />
                  <input type="button" class="btn" value="下" @mousedown="mouseDownPTZControl(2);" @mouseup="mouseUpPTZControl();" />
                  <input type="button" class="btn" value="右下" @mousedown="mouseDownPTZControl(8);" @mouseup="mouseUpPTZControl();" />
                </td>
              </tr>
            </table>
          </fieldset>
&lt;!&ndash;          <fieldset class="playback">
            <legend>回放</legend>
            <table width="100%" cellpadding="0" cellspacing="3" border="0">
              <tr>
                <td class="tt">码流类型</td>
                <td>
                  <select id="record_streamtype" class="sel">
                    <option value="1">主码流</option>
                    <option value="2">子码流</option>
                  </select>
                </td>
              </tr>
              <tr>
                <td class="tt">开始时间</td>
                <td>
                  <input id="starttime" type="text" class="txt" v-model="startTime" />（时间格式：2021-11-11 12:34:56）
                </td>
              </tr>
              <tr>
                <td class="tt">结束时间</td>
                <td>
                  <input id="endtime" type="text" class="txt" v-model="endTime" />
                  <input type="button" class="btn" value="搜索" @click="clickRecordSearch(0);" />
                </td>
              </tr>
              <tr>
                <td class="tt">按时间下载开始时间</td>
                <td>
                  <input id="downloadstarttime" type="text" class="txt" v-model="startTime" />（时间格式：2021-11-11 12:34:56）
                </td>
              </tr>
              <tr>
                <td class="tt">按时间下载结束时间</td>
                <td>
                  <input id="downloadendtime" type="text" class="txt" v-model="endTime" />
                  <input type="button" class="btn" value="下载" @click="clickStartDownloadRecordByTime();" />
                </td>
              </tr>
              <tr>
                <td colspan="2">
                  <div id="searchdiv" class="searchdiv">
                    <table id="searchlist" class="searchlist" cellpadding="0" cellspacing="0" border="0"></table>
                  </div>
                </td>
              </tr>
              <tr>
                <td colspan="2">
                  <input type="button" class="btn2" value="开始回放" @click="clickStartPlayback(1);" />
                  <input type="button" class="btn2" value="停止回放" @click="clickStopPlayback();" />
                  <input id="btnReverse" type="button" class="btn" value="倒放" @click="clickReversePlayback();" />
                  <input type="button" class="btn" value="单帧" @click="clickFrame();" />
                  <input id="transstream" type="checkbox" class="vtop" />&nbsp;启用转码码流
                </td>
              </tr>
              <tr>
                <td colspan="2">
                  <input type="button" class="btn" value="暂停" @click="clickPause();" />
                  <input type="button" class="btn" value="恢复" @click="clickResume();" />
                  <input type="button" class="btn" value="慢放" @click="clickPlaySlow();" />
                  <input type="button" class="btn" value="快放" @click="clickPlayFast();" />
                </td>
              </tr>
            </table>
          </fieldset>&ndash;&gt;
        </div>-->
      </div>
    </div>
  </div>
</template>

<script>
import ColorIcons from "_c/color-icons";
import liveVideo from "../../../../public/LiveVideo/video.vue";
import { getProjectList } from "@/api/realtime/realtimeMapApi";

export default {
  name: "viewScreenListManager",
  components: {
    ColorIcons,
    liveVideo,
  },
  props: {},
  data() {
    return {
      closeID: "",
      watchVideo: false,
      loading: false,
      singleLoading: false,
      loadingText: "数据加载中...",
      videoSplitNum: 12,
      showLeft: true,
      projectVideoList: [],
      allProjectVideoList: [],
      videoList: [],
      videoObjects: {},
      showTree: true,
      showFullScreen: false,
      filterStr: "",
      hkvInfo: {
        ip: '221.10.75.3',//海康威视摄像头/硬盘录像机的ip地址
        port: '61101',//海康威视摄像头/硬盘录像机的端口
        username: 'admin',//海康威视摄像头/硬盘录像机的用户名
        password: 'Nctt123456!',//海康威视摄像头/硬盘录像机的密码
        channels: [],//海康威视摄像头/硬盘录像机的通道
      },
      mySelectWnd: 0,//当前选中的窗口
      g_bPTZAuto: false,
      iProtocol: 1,
      loginLoading: false,
      startPlayLoading: false,
      iStreamType: 1,
      bZeroChannel: false,
      iRtspPort: 0,
      g_iWndIndex:0,
      g_szRecordType:'',//记录类型
      g_iSearchTimes:0, //搜索录像
      transstream:false, //是否转码
      startTime:new Date().format("yyyy-MM-dd 00:mm:ss"),
      endTime:new Date().format("yyyy-MM-dd 23:mm:ss"),
      g_iDownloadID:'',
      g_tDownloadProcess:'',

    };
  },
  methods: {
    getProjectList() {
      // this.projectVideoList = [];
      this.loading = true;
      this.singleLoading = false;
      this.loadingText = "数据加载中...";
      getProjectList({
        orgUnitId: "",
        area: "",
        monitorMode: "1",
        monitorObject: "",
      })
        .then((req) => {
          if (req.code === 200) {
            // console.log("req11111",req.data[0],req.data[0].area)
            // this.allProjectVideoList = req.data[0].projectList || [];
            req.data.forEach((s) => {
              // let obj = {
              //   expand: true,
              //   title: s.area,
              //   children: [],
              // };
              // let newarr = [];
              s.projectList.forEach((s2) => {
                let data={
                  title: s2.shortName,
                  shortName:s2.shortName,
                  expand: true,
                  checked: false,
                  text: s2.vidoeUrl ? (s2.vidoeUrl[0]?s2.vidoeUrl[0]:""):""
                }
                this.projectVideoList.push(data);
                this.allProjectVideoList.push(data)
              });
              if (this.projectVideoList && this.projectVideoList.length > 0) {
                this.projectVideoList[0].checked=true;
              }
              // obj.children = newarr;
              // console.log("1111".obj);
              // this.projectVideoList.push(obj);
            });
            // this.onFilter(); //修改的值记得取消注释内容
          } else {
            this.$Message.error({
              content: req.msg || "获取视频列表失败，请稍后再试！",
              duration: 3,
              background: true,
            });
          }
          this.loading = false;
        })
        .catch((err) => {
          this.loading = false;
          this.$Message.error({
            content: "获取视频列表失败，请稍后再试！",
            duration: 3,
            background: true,
          });
        });
    },
    switchVideoSplit(split) {
      this.videoSplitNum = split || 12;
    },

    switchLeftShow() {
      this.showLeft = !this.showLeft;
    },
    onTreeSelectChange(nodes, node) {
      this.$set(node, "expand", !node.expand);
    },
    // 分割符
    onTreeCheckChange(nodes, node) {
      if (node.checked){
        this.clickStartRealPlay()
      }else {
        this.clickStopRealPlay()
      }

      console.log("node111", nodes, node,this.projectVideoList);
     /* if (node.children && node.children.length > 0) {
        // 项目
        const videoListArray = node.children || [];
        if (node.checked) {
          // 选中
          if (videoListArray.length > 0) {
            this.loading = true;
            this.singleLoading = true;
            this.loadingText = "数据加载中1...";
          }
          videoListArray.forEach((item, videoIndex) => {
            const macId = item.nodeKey;
            this.videoList.push(macId);
            console.log("输出item", item);
          });
        } else {
          videoListArray.forEach((item) => {
            const macId = item.nodeKey;
            const index = this.videoList.indexOf(macId);
            this.videoList.splice(index, 1);
          });
        }
      } else {

        if (!node.text) {
          this.$Message.warning({
            content: node.shortName + "没有绑定摄像头！",
            duration: 3,
            closable:true
          });
          return;
        }
        // 摄像头
        if (node.checked) {
          this.loading = true;
          this.singleLoading = true;
          this.loadingText = "数据加载中2...";
          const macId = node.nodeKey;
          this.videoList.push(macId);

          this.$nextTick(() => {
            this.loading = false;
            this.singleLoading = false;
          });
          console.log("11233",node.text)

          this.videoObjects["url" + macId] =node.text;
          // let a="videoBackCkPlayer" + macId
          // this.$refs[a].addplay(url);
          console.log(321, this.videoObjects);
        } else {

          this.watchVideo = true;
          const macId = node.nodeKey;
          this.closeID="videoBackCkPlayer"+macId
          const index = this.videoList.indexOf(macId);
          this.videoList.splice(index, 1);
        }
      }*/
    },
    // 分割符
    closeVideo(macid) {
      this.watchVideo = true;
      return new Promise((resolve, reject) => {
        closePush(macid)
          .then((res) => {
            resolve(true);
          })
          .catch((err) => {
            reject(err);
          });
      });
    },
    closeVideoClick(value){
      //    console.log("关闭内容", value);
      this.watchVideo = true;
      const macId = value;
      this.closeID="videoBackCkPlayer"+value
      for (let i = 0; i < this.projectVideoList.length; i++) {
        if(value===this.projectVideoList[i].nodeKey){
          this.projectVideoList[i].checked=false
          //  console.log("删除选中",value,this.projectVideoList[i])
        }
      }
      //console.log("删除选中",this.projectVideoList)
      const index = this.videoList.indexOf(macId);
      this.videoList.splice(index, 1);
    },
    arrFilter(arr, exp, str) {
      let result = [];
      arr.map((item) => {
        let viewScreenList = item.viewScreenList || [];
        if (str) {
          viewScreenList = viewScreenList.filter(
            (item) => item.name.indexOf(this.filterStr) >= 0
          );
        }
        if (viewScreenList.length > 0) {
          let childrenArray = [];
          viewScreenList.map((video) => {
            childrenArray.push({
              title: video.name,
              macId: video.macId,
              dtu: video.dtu,
              id: video.id,
              checked: false,
            });
          });
          result.push({
            //title: item.projectName,
            title: item.shortName,
            id: item.id,
            children: childrenArray,
            checked: false,
            expand: exp ? true : false,
          });
        }
      });
      return result;
    },
    onFilter() {
      console.log("filter", this.filterStr);
      this.loadingText = "暂无数据";
      // let data = this.projectVideoList;
      let data = this.allProjectVideoList;
      if (this.filterStr) {
        let filterMaster = data.filter(
          (item) => item.shortName.indexOf(this.filterStr) >= 0
        );
        this.projectVideoList=filterMaster
        // console.log(JSON.stringify(filterMaster));
        // this.projectVideoList = this.projectVideoList.concat(
        //   this.arrFilter(filterMaster, true)
        // );
        // let filterItem = data.filter(
        //   (item) => item.shortName.indexOf(this.filterStr) < 0
        // );
        // console.log(JSON.stringify(filterItem));
        // this.projectVideoList = this.projectVideoList.concat(
        //   this.arrFilter(filterItem, this.filterStr, true)
        // );
      } else {
        this.projectVideoList = data;
      }
    },
    onLogin() {
      var that = this;
      that.loginLoading = true;
      debugger
      // 登录设备
      WebVideoCtrl.I_Login(that.hkvInfo.ip, that.iProtocol, that.hkvInfo.port, that.hkvInfo.username, that.hkvInfo.password, {
        async: false,
        success: function (xmlDoc) {
          // console.log('xmlDoc2', xmlDoc);//不能删除
          //TODO 获取通道信息
          that.getChannelInfo();
          that.getDevicePort(that.hkvInfo.ip + "_" + that.hkvInfo.port);
          that.loginLoading = false;
          // that.$Message.success("视频登录成功");
        },
        error: function (status, xmlDoc) {
          console.log(status, xmlDoc)
          that.loginLoading = false;
          that.$Message.error("视频登录接入失败");
        }
      });
    },
    onLogout() {
      this.hkvInfo.channels = [];
      var szDeviceIdentify = this.hkvInfo.ip + "_" + this.hkvInfo.port;
      var iRet = WebVideoCtrl.I_Logout(szDeviceIdentify);
      if (0 == iRet) {
        // this.$Message.success('退出成功');
      } else {
        // this.$Message.error('退出失败');
      }
    },
    clickStartRealPlay() {
      // 开始预览
      var that = this;
      that.startPlayLoading = true;
      var szDeviceIdentify = that.hkvInfo.ip + "_" + that.hkvInfo.port;

      var j = that.hkvInfo.channels.length > 4 ? 4 : that.hkvInfo.channels.length;
      debugger
      for (var i = 0; i < j; i++) {
        that.startRealPlay(szDeviceIdentify, i, that.hkvInfo.channels[i])
        // setTimeout(that.startRealPlay(szDeviceIdentify, i, that.hkvInfo.channels[i]), 200);
        break;
      }
      that.startPlayLoading = false;
    },
    videoInitPlugin: function () {
      var iRet = WebVideoCtrl.I_CheckPluginInstall();
      if (iRet === -1) {
        alert('您还未安装过插件，双击开发包目录里的WebComponentsKit.exe安装');
        return;
      }
      this.initPlugin()
    },
    initPlugin: function () {
      var that = this;

      WebVideoCtrl.I_InitPlugin(800, 800, {
        bWndFull: true,//是否支持单窗口双击全屏，默I_CheckPluginInstall
        iPackageType: 2,
        iWndowType: 1,
        // bNoPlugin: true,
        CbSelWnd: function (xmlStr) { //自己新增的方法
          that.g_iWndIndex = parseInt($(xmlDoc).find("SelectWnd").eq(0).text(), 10);
          var szInfo = "当前选择的窗口编号：" + g_iWndIndex;
          console.log(szInfo);
        },
        cbInitPluginComplete: function () {
          debugger
          WebVideoCtrl.I_InsertOBJECTPlugin("divPlugin");
          that.onLogin()
          // 检查插件是否最新
          if (WebVideoCtrl.I_CheckPluginVersion() === -1) {
            alert("检测到新的插件版本，双击开发包目录里的WebComponentsKit.exe升级！");
            return;
          }
        }
      });
    },
    getDevicePort: function (szDeviceIdentify) {
      var oPort = WebVideoCtrl.I_GetDevicePort(szDeviceIdentify);
      this.iRtspPort = oPort.iRtspPort;
    },
    startRealPlay: function (szDeviceIdentify, iWndIndex, iChannelID) {
      var that = this;
      WebVideoCtrl.I_StartRealPlay(szDeviceIdentify, {
        iRtspPort: that.iRtspPort,
        iWndIndex: iWndIndex,
        iStreamType: 1,
        iChannelID: iChannelID,
        bZeroChannel: that.bZeroChannel,
        success: function () {
          // that.$Message.success("开始预览");
        },
        error: function (status, xmlDoc2) {
          console.log(xmlDoc2)//不能删除
          that.$Message.error("预览失败");
          if (status === 403) {
            console.log("szInfo 设备不支持Websocket取流！");
          } else {
            console.log("视频预览失败 ", status, xmlDoc2);
          }
        }
      });
    },
    clickStopRealPlay: function () {
      var j = this.hkvInfo.channels.length > 4 ? 4 : this.hkvInfo.channels.length;
      for (var i = 0; i < j; i++) {
        setTimeout(this.stopRealPlay(i), 1000);
      }
    },
    stopRealPlay: function (iWndIndex) {
      var that = this;
      WebVideoCtrl.I_Stop({
        iWndIndex: iWndIndex,
        success: function () {
          // that.$Message.success("停止预览");
        },
        error: function () {
          // that.$Message.error("停止预览失败");
        }
      });
    },
    // 获取通道，实际上可以根据自己的项目，获取数字通道，模拟通道，零通道中的一个或多个，不用全部获取（提高效率）
    getChannelInfo: function () {
      var that = this;
      var szDeviceIdentify = this.hkvInfo.ip + "_" + this.hkvInfo.port;
      // debugger
      // 数字通道
      that.hkvInfo.channels = [];
      WebVideoCtrl.I_GetDigitalChannelInfo(szDeviceIdentify, {
          async: false,
          success: function (xmlStr) {
            var oChannels = $(xmlStr).find("InputProxyChannelStatus");
            $.each(oChannels, function (i) {
              var id = $(this).find("id").eq(0).text();
              that.hkvInfo.channels.push(id);
            });
            that.clickStartRealPlay()
          },
          error: function (status, xmlDoc) {
            console.log("获取数字通道失败");
          }
        }
      );
      // 模拟通道
      WebVideoCtrl.I_GetAnalogChannelInfo(szDeviceIdentify, {
        async: false,
        success: function (xmlStr) {
          console.log(xmlStr)
          var oChannels = $(xmlStr).find("VideoInputChannel");
          $.each(oChannels, function (i) {
            var id = $(this).find("id").eq(0).text();
            that.hkvInfo.channels.push(id);
          });
          that.clickStartRealPlay()
        },

        error: function (status, xmlDoc) {
          console.log("模拟通道error",xmlDoc);
        }
      });
    },
    mouseDownPTZControl: function (iPTZIndex) {
      console.log("云台控制")
      var that=this
      var oWndInfo = WebVideoCtrl.I_GetWindowStatus(this.mySelectWnd);
      var iPTZSpeed=4
      if (oWndInfo !== null) {
        if (iPTZIndex === 9 && this.g_bPTZAuto) {
          iPTZSpeed = 0;
        } else {
          this.g_bPTZAuto = false;
        }
        console.log(iPTZSpeed)
        WebVideoCtrl.I_PTZControl(iPTZIndex, false, {
          iPTZSpeed: iPTZSpeed,
          success: function (xmlStr) {
            console.log("I_PTZControl", xmlStr);
            if (iPTZIndex === 9) {
              if (that.g_bPTZAuto) {
                that.$Message.success(" 停止自动预览");
              }else {
                that.$Message.success(" 开启自动预览");
              }
            } else {
              console.log(oWndInfo.szDeviceIdentify + " 开启云台成功！mouseDown");
            }
            if (iPTZIndex === 9) {
              that.g_bPTZAuto = !that.g_bPTZAuto;
            }
          },
          error: function (status, xmlDoc) {
            console.log(oWndInfo.szDeviceIdentify + " 开启云台失败！mouseDown", status, xmlDoc);
          }
        });
      }
    },
    mouseUpPTZControl: function (iPTZIndex) {
      var oWndInfo = WebVideoCtrl.I_GetWindowStatus(this.mySelectWnd);
      if (oWndInfo !== null) {
        WebVideoCtrl.I_PTZControl(iPTZIndex, true, {
          success: function (xmlStr) {
            console.log(oWndInfo.szDeviceIdentify + " 停止云台成功！mouseUp", xmlStr)
          },
          error: function (status, xmlDoc) {
            console.log(oWndInfo.szDeviceIdentify + " 停止云台失败！mouseUp", status, xmlDoc);
          }
        });
      }
    },
    clickCapturePic(){
      console.log("截图")
      var that = this
      var oWndInfo = WebVideoCtrl.I_GetWindowStatus(this.g_iWndIndex),
        szInfo = "";

      if (oWndInfo != null) {
        var xmlDoc = WebVideoCtrl.I_GetLocalCfg();
        var szCaptureFileFormat = "0";
        if (xmlDoc != null) {
          szCaptureFileFormat = $(xmlDoc).find("CaptureFileFormat").eq(0).text();
        }

        var szChannelID = this.hkvInfo.channels[this.g_iWndIndex];
        var szPicName = oWndInfo.szDeviceIdentify + "_" + szChannelID + "_" + new Date().getTime();

        szPicName += ("0" === szCaptureFileFormat) ? ".jpg": ".bmp";

        WebVideoCtrl.I2_CapturePic(szPicName, {
          bDateDir: true  //是否生成日期文件
        }).then(function(){
          szInfo = "截图成功！";
          that.$Message.success( szInfo);
        },function(){
          szInfo = "截图失败！";
          that.$Message.error(oWndInfo.szDeviceIdentify + " " + szInfo);
        });
      }
    },
    clickStartRecord(szType){
      console.log("开始录像")
      var that = this;
      var oWndInfo = WebVideoCtrl.I_GetWindowStatus(this.g_iWndIndex),
        szInfo = "";

      this.g_szRecordType = szType;

      if (oWndInfo != null) {
        var szChannelID = this.hkvInfo.channels[this.g_iWndIndex],
          szFileName = oWndInfo.szDeviceIdentify + "_" + szChannelID + "_" + new Date().getTime();

        WebVideoCtrl.I_StartRecord(szFileName, {
          bDateDir: true, //是否生成日期文件
          success: function () {
            if ('realplay' === szType) {
              szInfo = "开始录像成功";
            } else if ('playback' === szType) {
              szInfo = "开始剪辑成功";
            }
            that.$Message.success(szInfo);
          },
          error: function () {
            if ('realplay' === szType) {
              szInfo = "开始录像失败";
            } else if ('playback' === szType) {
              szInfo = "开始剪辑失败";
            }
            that.$Message.error(szInfo);
          }
        });
      }
    },
    clickStopRecord(szType, iWndIndex){
      console.log("结束录像")
      var that = this;
      if ("undefined" === typeof iWndIndex) {
        iWndIndex = this.g_iWndIndex;
      }
      var oWndInfo = WebVideoCtrl.I_GetWindowStatus(iWndIndex),
        szInfo = "";

      if (oWndInfo != null) {
        WebVideoCtrl.I_StopRecord({
          success: function () {
            if ('realplay' === szType) {
              szInfo = "停止录像成功";
            } else if ('playback' === szType) {
              szInfo = "停止剪辑成功！";
            }

            that.$Message.success(szInfo);
          },
          error: function () {
            if ('realplay' === szType) {
              szInfo = "停止录像失败";
            } else if ('playback' === szType) {
              szInfo = "停止剪辑失败！";
            }
            that.$Message.error(szInfo);
          }
        });
      }
    },
    clickRecordSearch(iType){
      console.log("搜索录像")
      var that=this
      var szDeviceIdentify = that.hkvInfo.ip + "_" + that.hkvInfo.port,
        iStreamType = 1,
        bZeroChannel = that.g_iWndIndex != 0 ? true : false,
        iChannelID = that.hkvInfo.channels[this.g_iWndIndex],
        szStartTime = that.startTime,
        szEndTime = that.endTime;

      if (Date.parse(szEndTime.replace(/-/g, "/")) - Date.parse(szStartTime.replace(/-/g, "/")) < 0) {
        alert("开始时间大于结束时间");
        return;
      }
      if (null == szDeviceIdentify) {
        return;
      }

      if (bZeroChannel) {// 零通道不支持录像搜索
        return;
      }
      var g_iSearchTimes = that.g_iSearchTimes
      if (0 == iType) {// 首次搜索
        g_iSearchTimes = 0;
      }

      WebVideoCtrl.I_RecordSearch(szDeviceIdentify, iChannelID, szStartTime, szEndTime, {
        iStreamType: iStreamType,
        iSearchPos: g_iSearchTimes * 40,
        success: function (xmlDoc) {
          if ("MORE" === $(xmlDoc).find("responseStatusStrg").eq(0).text()) {

            for(var i = 0, nLen = $(xmlDoc).find("searchMatchItem").length; i < nLen; i++) {
              var szPlaybackURI = $(xmlDoc).find("playbackURI").eq(i).text();
              if(szPlaybackURI.indexOf("name=") < 0) {
                break;
              }
              var szStartTime = $(xmlDoc).find("startTime").eq(i).text();
              var szEndTime = $(xmlDoc).find("endTime").eq(i).text();
              var szFileName = szPlaybackURI.substring(szPlaybackURI.indexOf("name=") + 5, szPlaybackURI.indexOf("&size="));

              var objTr = $("#searchlist").get(0).insertRow(-1);
              var objTd = objTr.insertCell(0);
              objTd.id = "downloadTd" + i;
              objTd.innerHTML = g_iSearchTimes * 40 + (i + 1);
              objTd = objTr.insertCell(1);
              objTd.width = "30%";
              objTd.innerHTML = szFileName;
              objTd = objTr.insertCell(2);
              objTd.width = "30%";
              objTd.innerHTML = (szStartTime.replace("T", " ")).replace("Z", "");
              objTd = objTr.insertCell(3);
              objTd.width = "30%";
              objTd.innerHTML = (szEndTime.replace("T", " ")).replace("Z", "");
              objTd = objTr.insertCell(4);
              objTd.width = "10%";
              objTd.innerHTML = "<a onclick='clickStartDownloadRecord(" + (i + g_iSearchTimes * 40) + ");'>下载</a>";
              $("#downloadTd" + (i + g_iSearchTimes * 40)).data("fileName", szFileName);
              $("#downloadTd" + (i + g_iSearchTimes * 40)).data("playbackURI", szPlaybackURI);
            }

            g_iSearchTimes++;
            clickRecordSearch(1);// 继续搜索
          } else if ("OK" === $(xmlDoc).find("responseStatusStrg").eq(0).text()) {
            var iLength = $(xmlDoc).find("searchMatchItem").length;
            for(var i = 0; i < iLength; i++) {
              var szPlaybackURI = $(xmlDoc).find("playbackURI").eq(i).text();
              if(szPlaybackURI.indexOf("name=") < 0) {
                break;
              }
              var szStartTime = $(xmlDoc).find("startTime").eq(i).text();
              var szEndTime = $(xmlDoc).find("endTime").eq(i).text();
              var szFileName = szPlaybackURI.substring(szPlaybackURI.indexOf("name=") + 5, szPlaybackURI.indexOf("&size="));

              var objTr = $("#searchlist").get(0).insertRow(-1);
              var objTd = objTr.insertCell(0);
              objTd.id = "downloadTd" + i;
              objTd.innerHTML = g_iSearchTimes * 40 + (i + 1);
              objTd = objTr.insertCell(1);
              objTd.width = "30%";
              objTd.innerHTML = szFileName;
              objTd = objTr.insertCell(2);
              objTd.width = "30%";
              objTd.innerHTML = (szStartTime.replace("T", " ")).replace("Z", "");
              objTd = objTr.insertCell(3);
              objTd.width = "30%";
              objTd.innerHTML = (szEndTime.replace("T", " ")).replace("Z", "");
              objTd = objTr.insertCell(4);
              objTd.width = "10%";
              objTd.innerHTML = "<a  onclick='clickStartDownloadRecord(" + (i + g_iSearchTimes * 40) + ");'>下载</a>";
              $("#downloadTd" + (i + g_iSearchTimes * 40)).data("fileName", szFileName);
              $("#downloadTd" + (i + g_iSearchTimes * 40)).data("playbackURI", szPlaybackURI);
            }
            that.$Message.success(szDeviceIdentify + " 搜索录像文件成功！");
          } else if("NO MATCHES" === $(xmlDoc).find("responseStatusStrg").eq(0).text()) {
            setTimeout(function() {
              that.$Message.error(szDeviceIdentify + " 没有录像文件！");
            }, 50);
          }
        },
        error: function (status, xmlDoc) {
          that.$Message.error(szDeviceIdentify + " 搜索录像文件失败！", status, xmlDoc);
        }
      });
    },
    clickStartDownloadRecordByTime(){
      console.log("下载")
      var that=this
      var szDeviceIdentify = that.hkvInfo.ip + "_" + that.hkvInfo.port,
        szChannelID = that.hkvInfo.channels[this.g_iWndIndex],
        szStartTime = that.startTime,
        szFileName = $("#downloadTd0").data("fileName"),
        szPlaybackURI = $("#downloadTd0").data("playbackURI"),
        szEndTime = that.endTime;

      if (null == szDeviceIdentify) {
        return;
      }
      if (Date.parse(szEndTime.replace(/-/g, "/")) - Date.parse(szStartTime.replace(/-/g, "/")) < 0) {
        alert("开始时间大于结束时间");
        return;
      }
      this.g_iDownloadID = WebVideoCtrl.I_StartDownloadRecordByTime(szDeviceIdentify, szPlaybackURI, szFileName, szStartTime,szEndTime,{
        bDateDir: true  //是否生成日期文件
      });

      if (g_iDownloadID < 0) {
        var iErrorValue = WebVideoCtrl.I_GetLastError();
        if (34 == iErrorValue) {
          that.$Message.success(szDeviceIdentify + " 已下载！");
        } else if (33 == iErrorValue) {
          that.$Message.error(szDeviceIdentify + " 空间不足！");
        } else {
          that.$Message.error(szDeviceIdentify + " 下载失败！");
        }
      } else {
        $("<div id='downProcess' class='freeze'></div>").appendTo("body");
        this.g_tDownloadProcess = setInterval("downProcess(" + 0 + ")", 1000);
      }
    },
    clickStartDownloadRecord(i) {
      var that=this
      var szDeviceIdentify = that.hkvInfo.ip + "_" + that.hkvInfo.port,
        szChannelID = that.hkvInfo.channels[this.g_iWndIndex],
        szFileName = $("#downloadTd" + i).data("fileName"),
        szPlaybackURI = $("#downloadTd" + i).data("playbackURI");

      if (null == szDeviceIdentify) {
        return;
      }

      this.g_iDownloadID = WebVideoCtrl.I_StartDownloadRecord(szDeviceIdentify, szPlaybackURI, szFileName, {
        bDateDir: true  //是否生成日期文件
      });

      if (g_iDownloadID < 0) {
        var iErrorValue = WebVideoCtrl.I_GetLastError();
        if (34 == iErrorValue) {
          that.$Message.success(szDeviceIdentify + " 已下载！");
        } else if (33 == iErrorValue) {
          that.$Message.success(szDeviceIdentify + " 空间不足！");
        } else {
          that.$Message.success(szDeviceIdentify + " 下载失败！");
        }
      } else {
        $("<div id='downProcess' class='freeze'></div>").appendTo("body");
        this.g_tDownloadProcess = setInterval("downProcess(" + i + ")", 1000);
      }
    },
    downProcess() {
      var iStatus = WebVideoCtrl.I_GetDownloadStatus(this.g_iDownloadID);
      if (0 == iStatus) {
        $("#downProcess").css({
          width: $("#searchlist").width() + "px",
          height: "100px",
          lineHeight: "100px",
          left: $("#searchdiv").offset().left + "px",
          top: $("#searchdiv").offset().top + "px"
        });
        var iProcess = WebVideoCtrl.I_GetDownloadProgress(g_iDownloadID);
        if (iProcess < 0) {
          clearInterval(g_tDownloadProcess);
          this.g_tDownloadProcess = 0;
          this.g_iDownloadID = -1;
        } else if (iProcess < 100) {
          $("#downProcess").text(iProcess + "%");
        } else {
          $("#downProcess").text("100%");
          setTimeout(function () {
            $("#downProcess").remove();
          }, 1000);

          WebVideoCtrl.I_StopDownloadRecord(this.g_iDownloadID);

          that.$Message.success("录像下载完成！");
          clearInterval(this.g_tDownloadProcess);
          this.g_tDownloadProcess = 0;
          this.g_iDownloadID = -1;
        }
      } else {
        WebVideoCtrl.I_StopDownloadRecord(this.g_iDownloadID);

        clearInterval(this.g_tDownloadProcess);
        this.g_tDownloadProcess = 0;
        this.g_iDownloadID = -1;
      }
    },
    clickStartPlayback(streamType){
      console.log("开始回放")
      if (!streamType){
        streamType =1
      }
      var that=this
      var oWndInfo = WebVideoCtrl.I_GetWindowStatus(this.g_iWndIndex),
        szDeviceIdentify = that.hkvInfo.ip + "_" + that.hkvInfo.port,
        iRtspPort = that.hkvInfo.port,
        iStreamType = streamType,
        bZeroChannel = that.g_iWndIndex === 0 ? true : false,
        iChannelID = that.hkvInfo.channels[this.g_iWndIndex],
        szStartTime = that.startTime,
        szEndTime = that.endTime,
        szInfo = "",
        bChecked = that.transstream,
        iRet = -1;
      console.log(szStartTime,szEndTime)
      if (null == szDeviceIdentify) {
        return;
      }

      // if (bZeroChannel) {// 零通道不支持回放
      //   return;
      // }
      var startPlayback = function () {

        if (bChecked) {// 启用转码回放
          var oTransCodeParam = {
            TransFrameRate: "14",// 0：全帧率，5：1，6：2，7：4，8：6，9：8，10：10，11：12，12：16，14：15，15：18，13：20，16：22
            TransResolution: "1",// 255：Auto，3：4CIF，2：QCIF，1：CIF
            TransBitrate: "19"// 2：32K，3：48K，4：64K，5：80K，6：96K，7：128K，8：160K，9：192K，10：224K，11：256K，12：320K，13：384K，14：448K，15：512K，16：640K，17：768K，18：896K，19：1024K，20：1280K，21：1536K，22：1792K，23：2048K，24：3072K，25：4096K，26：8192K
          };
          WebVideoCtrl.I_StartPlayback(szDeviceIdentify, {
            iRtspPort: iRtspPort,
            iStreamType: iStreamType,
            iChannelID: iChannelID,
            szStartTime: szStartTime,
            szEndTime: szEndTime,
            oTransCodeParam: oTransCodeParam,
            success: function () {
              szInfo = "开始回放成功！";
              that.$Message.success(szDeviceIdentify + " " + szInfo);
            },
            error: function (status, xmlDoc) {
              if (403 === status) {
                szInfo = "设备不支持Websocket取流！";
              } else {
                szInfo = "开始回放失败！";
              }
              that.$Message.error(szDeviceIdentify + " " + szInfo);
            }
          });
        } else {
          WebVideoCtrl.I_StartPlayback(szDeviceIdentify, {
            iRtspPort: iRtspPort,
            iStreamType: iStreamType,
            iChannelID: iChannelID,
            szStartTime: szStartTime,
            szEndTime: szEndTime,
            success: function () {
              szInfo = "开始回放成功！";
              that.$Message.success(szDeviceIdentify + " " + szInfo);
            },
            error: function (status, xmlDoc) {
              if (403 === status) {
                szInfo = "设备不支持Websocket取流！";
              } else {
                szInfo = "开始回放失败！";
              }
              that.$Message.error(szDeviceIdentify + " " + szInfo);
            }
          });
        }
      };

      if (oWndInfo != null) {// 已经在播放了，先停止
        WebVideoCtrl.I_Stop({
          success: function () {
            startPlayback();
          }
        });
      } else {
        startPlayback();
      }
    },
    clickStopPlayback(){
      console.log("停止回放")
      let that=this
      var oWndInfo = WebVideoCtrl.I_GetWindowStatus(this.g_iWndIndex),
        szInfo = "";

      if (oWndInfo != null) {
        WebVideoCtrl.I_Stop({
          success: function () {
            szInfo = "停止回放成功！";
            that.$Message.success(oWndInfo.szDeviceIdentify + " " + szInfo);
          },
          error: function () {
            szInfo = "停止回放失败！";
            that.$Message.error(oWndInfo.szDeviceIdentify + " " + szInfo);
          }
        });
      }
    },
    clickReversePlayback(){
      console.log("倒放")
      var that=this
      var oWndInfo = WebVideoCtrl.I_GetWindowStatus(this.g_iWndIndex),
        szDeviceIdentify = that.hkvInfo.ip + "_" + that.hkvInfo.port,
        iRtspPort = that.hkvInfo.port,
        iStreamType = streamType,
        bZeroChannel = that.g_iWndIndex === 0 ? true : false,
        iChannelID = that.hkvInfo.channels[this.g_iWndIndex],
        szStartTime = that.startTime,
        szEndTime = that.endTime,
        szInfo = "";

      if (null == szDeviceIdentify) {
        return;
      }

      if (bZeroChannel) {// 零通道不支持倒放
        return;
      }

      var reversePlayback = function () {
        var iRet = WebVideoCtrl.I_ReversePlayback(szDeviceIdentify, {
          iRtspPort: iRtspPort,
          iStreamType: iStreamType,
          iChannelID: iChannelID,
          szStartTime: szStartTime,
          szEndTime: szEndTime
        });

        if (0 == iRet) {
          szInfo = "开始倒放成功！";
        } else {
          szInfo = "开始倒放失败！";
        }
        that.$Message.success(szDeviceIdentify + " " + szInfo);
      };

      if (oWndInfo != null) {// 已经在播放了，先停止
        WebVideoCtrl.I_Stop({
          success: function () {
            reversePlayback();
          }
        });
      } else {
        reversePlayback();
      }
    },
    clickPause(){
      console.log("暂停")
      let that=this
      var oWndInfo = WebVideoCtrl.I_GetWindowStatus(this.g_iWndIndex),
        szInfo = "";

      if (oWndInfo != null) {
        WebVideoCtrl.I_Pause({
          success: function () {
            szInfo = "暂停成功！";
            that.$Message.success(oWndInfo.szDeviceIdentify + " " + szInfo);
          },
          error: function () {
            szInfo = "暂停失败！";
            that.$Message.error(oWndInfo.szDeviceIdentify + " " + szInfo);
          }
        });
      }
    },
    clickResume(){
      console.log("恢复")
      let that=this
      var oWndInfo = WebVideoCtrl.I_GetWindowStatus(this.g_iWndIndex),
        szInfo = "";

      if (oWndInfo != null) {
        WebVideoCtrl.I_Resume({
          success: function () {
            szInfo = "恢复成功！";
            that.$Message.success(oWndInfo.szDeviceIdentify + " " + szInfo);
          },
          error: function () {
            szInfo = "恢复失败！";
            that.$Message.error(oWndInfo.szDeviceIdentify + " " + szInfo);
          }
        });
      }
    },
    clickPlaySlow(){
      console.log("慢放")
      let that=this
      var oWndInfo = WebVideoCtrl.I_GetWindowStatus(this.g_iWndIndex),
        szInfo = "";

      if (oWndInfo != null) {
        WebVideoCtrl.I_PlaySlow({
          success: function () {
            szInfo = "慢放成功！";
            that.$Message.success(oWndInfo.szDeviceIdentify + " " + szInfo);
          },
          error: function () {
            szInfo = "慢放失败！";
            that.$Message.error(oWndInfo.szDeviceIdentify + " " + szInfo);
          }
        });
      }
    },
    clickPlayFast(){
      console.log("快放")
      let that=this
      var oWndInfo = WebVideoCtrl.I_GetWindowStatus(this.g_iWndIndex),
        szInfo = "";

      if (oWndInfo != null) {
        WebVideoCtrl.I_PlayFast({
          success: function () {
            szInfo = "快放成功！";
            that.$Message.success(oWndInfo.szDeviceIdentify + " " + szInfo);
          },
          error: function () {
            szInfo = "快放失败！";
            that.$Message.error(oWndInfo.szDeviceIdentify + " " + szInfo);
          }
        });
      }
    },
    clickFullScreen(){
      WebVideoCtrl.I_FullScreen(true);
    },

    isFullscreen() {
      return Boolean(document.fullscreenElement
        || document.msFullscreenElement
        || document.mozFullScreenElement
        || document.webkitFullscreenElement || false);
    },
    clickEnableEZoom() {
      var that = this;
      var oWndInfo = WebVideoCtrl.I_GetWindowStatus(this.g_iWndIndex),
        szInfo = "";

      if (oWndInfo != null) {
        var iRet = WebVideoCtrl.I_EnableEZoom();
        if (0 == iRet) {
          szInfo = "启用电子放大成功";
        } else {
          szInfo = "启用电子放大失败";
        }
        that.$Message.success(szInfo);
      }
    },
    clickDisableEZoom() {
      var that = this;
      var oWndInfo = WebVideoCtrl.I_GetWindowStatus(this.g_iWndIndex),
        szInfo = "";

      if (oWndInfo != null) {
        var iRet = WebVideoCtrl.I_DisableEZoom();
        if (0 == iRet) {
          szInfo = "禁用电子放大成功";
        } else {
          szInfo = "禁用电子放大失败";
        }
        that.$Message.success(szInfo);
      }
    },
    clickEnable3DZoom() {
      var that = this;
      var oWndInfo = WebVideoCtrl.I_GetWindowStatus(this.g_iWndIndex),
        szInfo = "";

      if (oWndInfo != null) {
        var iRet = WebVideoCtrl.I_Enable3DZoom();
        if (0 == iRet) {
          szInfo = "启用3D放大成功！";
        } else {
          szInfo = "启用3D放大失败！";
        }
        that.$Message.success(oWndInfo.szDeviceIdentify + " " + szInfo);
      }
    },
    clickDisable3DZoom() {
      var that = this;
      var oWndInfo = WebVideoCtrl.I_GetWindowStatus(this.g_iWndIndex),
        szInfo = "";

      if (oWndInfo != null) {
        var iRet = WebVideoCtrl.I_Disable3DZoom();
        if (0 == iRet) {
          szInfo = "禁用3D放大成功！";
        } else {
          szInfo = "禁用3D放大失败！";
        }
        that.$Message.success(oWndInfo.szDeviceIdentify + " " + szInfo);
      }
    },
    clickFrame() {
      var that = this;
      var oWndInfo = WebVideoCtrl.I_GetWindowStatus(this.g_iWndIndex),
        szInfo = "";

      if (oWndInfo != null) {
        WebVideoCtrl.I_Frame({
          success: function () {
            szInfo = "单帧播放成功！";
            that.$Message.success(oWndInfo.szDeviceIdentify + " " + szInfo);
          },
          error: function () {
            szInfo = "单帧播放失败！";
            that.$Message.error(oWndInfo.szDeviceIdentify + " " + szInfo);
          }
        });
      }
    }
  },
  mounted() {
    this.videoList = [];
    window.addEventListener('resize', () => {
      this.showFullScreen = this.isFullscreen();
      console.log(this.showFullScreen)
    });
    this.getProjectList();
    this.videoInitPlugin(); // 初始化video界面
  },
  beforeDestroy() {},
  destroyed: function () {
    this.clickStopRealPlay();
    this.onLogout();
  },
};
</script>

<style lang="less">
@import "./video-list.less";
</style>
