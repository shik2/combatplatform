<template>
  <div class="pages-content">
    <el-row>
      <el-col :span="17">
        <div
          class="map-area"
          id="Map"
        >
        </div>
      </el-col>
      <el-col :span="3">
        <div class="list-panel">
          <div class="sailInfo-label">
            <h1>我方战舰</h1>
          </div>
          <div class="sailInfo-list">
            <el-table
              @row-click="selectShip"
              :row-class-name="tableRowClassName"
              :data="sailProList"
              v-loading="loading"
              element-loading-background="rgba(0, 0, 0, 0.5)"
              height="240"
            >
              <el-table-column
                prop="id"
                label="战舰编号"
                width="106"
                align="center"
              ></el-table-column>
              <el-table-column
                prop="type"
                label="种类"
                width="117"
                align="center"
              ></el-table-column>
            </el-table>
          </div>
          <div class="targetInfo-label">
            <h1>敌方信息</h1>
          </div>
          <div class="targetInfo-list">
            <keep-alive>
              <router-view
                v-bind:selTargetFP="this.selTarget"
                v-on:listenToChild="showStyleFromChild"
              ></router-view>
            </keep-alive>
          </div>
          <div class="switch-button">
            <div
              class="switch-sail"
              @click="routerPages(1)"
            >
              <img
                id="mapimg"
                style="margin-top:10px"
                src="static/image/ship.png"
              >
            </div>
            <div
              class="switch-sky"
              @click="routerPages(2)"
            >
              <img
                id="mapimg"
                style="margin-top:10px"
                src="static/image/plane.png"
              >
            </div>
            <div
              class="switch-missile"
              @click="routerPages(3)"
            >
              <img
                id="mapimg"
                style="margin-top:10px"
                src="static/image/rocket.png"
              >
            </div>
          </div>
        </div>
      </el-col>
      <el-col :span="4">
        <div class="detail-panel">
          <el-form
            ref="form"
            :model="form"
            label-width="90px"
            style="margin-top:10px"
          >
            <el-form-item label="目标编号">
              <el-input v-model="form.targetId"></el-input>
            </el-form-item>
            <el-form-item
              label="攻击距离"
              style="margin-top:50px"
            >
              <el-input v-model="form.distance"></el-input>
            </el-form-item>
            <el-form-item
              label="移动速度"
              style="margin-top:50px"
            >
              <el-input v-model="form.speed"></el-input>
            </el-form-item>
            <el-form-item
              label="目标类型"
              style="margin-top:50px"
            >
              <el-input v-model="form.targetType"></el-input>
            </el-form-item>
            <el-form-item
              label="威胁等级"
              v-if="seen"
              style="margin-top:50px"
            >
              <el-input v-model="form.threatRank"></el-input>
            </el-form-item>
          </el-form>
          <div class="attack-panel">
            <button
              type="button"
              class="confirm-btn"
              v-on:click="confirm()"
            >确认目标</button>
          </div>
        </div>
      </el-col>
    </el-row>
    <div class="bottom-panel">
      <div
        class="weaponInfo"
        v-if="!isFriend"
      >
        <el-form
          ref="form"
          :model="weaponInfo"
          label-width="100px"
        >
          <el-form-item
            label="武器类型："
            style="margin-top:40px;margin-left:100px;margin-right:40px"
          >
            <el-input v-model="weaponInfo.weaponType"></el-input>
          </el-form-item>
          <el-form-item
            label="武器数量："
            style="margin-top:50px;margin-left:100px;margin-right:40px"
          >
            <el-input v-model="weaponInfo.weaponNum"></el-input>
          </el-form-item>
        </el-form>
      </div>
      <div
        class="attackInfo"
        v-if="!isFriend"
      >
        <el-form
          ref="form"
          :model="attackInfo"
          label-width="100px"
        >
          <el-form-item
            label="攻击时间："
            style="margin-top:40px;margin-left:50px;margin-right:90px"
          >
            <el-input v-model="attackInfo.attackTime"></el-input>
          </el-form-item>
          <el-form-item
            label="攻击角度："
            style="margin-top:40px;margin-left:50px;margin-right:90px"
          >
            <el-input v-model="attackInfo.attackAngle"></el-input>
          </el-form-item>
        </el-form>
      </div>
      <div
        class="friendInfo"
        v-if="isFriend"
      >
        <p>目标为友军！</p>
      </div>
      <div class="attackButton">
        <button
          type="button"
          class="attack-btn"
          v-on:click="attack()"
        >{{buttonText}}</button>
      </div>
      <div class="resultInfo">
        <div class="result-label">
          战损报告
        </div>
        <div class="result-list">
          <el-table
            height="150"
            border
            :data="resultList"
            :row-class-name="tableRowClassDamage2"
          >
            <el-table-column
              prop="shipId"
              label="我方id"
              width="145"
              align="center"
            ></el-table-column>
            <el-table-column
              prop="targetId"
              label="目标id"
              width="145"
              align="center"
            ></el-table-column>
            <!-- <el-table-column
              prop="damageOur"
              label="我方损伤"
              width="110"
              align="center"
            ></el-table-column> -->
            <el-table-column
              prop="damageTg"
              label="战损比"
              width="150"
              align="center"
            ></el-table-column>
          </el-table>
        </div>
        <div style="float:right;margin-top:90px;margin-right:9px;">
          <button
            type="button"
            class="detail-btn"
            @click="detail()"
          >详细</button>
        </div>
      </div>
    </div>
    <el-dialog
      :visible.sync="detailDialogVisible"
      width="50%"
      :title="detailDialogTitle"
    >
      <el-table
        height="400"
        border
        :data="warList"
        :row-class-name="tableRowClassDamage"
      >
        <el-table-column
          prop="shipId"
          label="我方id"
          width="80"
          align="center"
        ></el-table-column>
        <el-table-column
          prop="targetId"
          label="目标id"
          width="80"
          align="center"
        ></el-table-column>
        <el-table-column
          prop="targetType"
          label="目标类型"
          width="109"
          align="center"
        ></el-table-column>
        <el-table-column
          prop="weaponType"
          label="武器类型"
          width="105"
          align="center"
        ></el-table-column>
        <el-table-column
          prop="weaponNum"
          label="武器数量"
          width="105"
          align="center"
        ></el-table-column>
        <el-table-column
          prop="attackTime"
          label="攻击时间"
          width="110"
          align="center"
        ></el-table-column>
        <el-table-column
          prop="attackAngle"
          label="攻击角度"
          width="110"
          align="center"
        ></el-table-column>
        <el-table-column
          prop="damageOur"
          label="我方损伤"
          width="110"
          align="center"
        ></el-table-column>
        <el-table-column
          prop="damageTg"
          label="敌方损伤"
          width="110"
          align="center"
        ></el-table-column>
      </el-table>
    </el-dialog>
  </div>
</template>

<script>
var map;
var selMk1; //我方选中
var selMk2; //敌方选中

import addMapStyle from "@/style/mapStyle.json";
import {
  getGuardInfo,
  getTargets,
  getSelTarget,
  getSelShip,
  getWeaponInfo,
  getAttackInfo,
  getResult,
  getWarInfo
} from "@/api/api.js";
export default {
  name: "Information",
  data() {
    return {
      parentMsg: "",
      seen: false,
      isFriend: false,
      loading: false,
      selShip: "",
      selTarget: "",
      targetList: [],
      sailProList: [],
      weaponInfo: {
        weaponType: "",
        weaponNum: ""
      },
      attackInfo: {
        attackTime: "",
        attackAngle: ""
      },
      buttonText: "攻击",
      form: {
        targetId: "",
        distance: "",
        speed: "",
        targetType: "",
        threatRank: ""
      },
      resultList: [],
      detailDialogTitle: "",
      detailDialogVisible: false,
      warList: []
    };
  },
  methods: {
    routerPages(val) {
      if (val == 1) {
        this.$router.push("/targetSail");
      }
      if (val == 2) {
        this.$router.push("/targetSky");
      }
      if (val == 3) {
        this.$router.push("/targetMissile");
      }
    },
    initMap() {
      var _this = this;
      // 获取我方战舰信息，填入表格 TODO
      getGuardInfo().then(data => {
        //console.log(data);
        this.sailProList = [];
        data.forEach(element => {
          let obj = {};
          obj.id = element.shipId;
          obj.type = element.type;
          obj.longitude = element.longitude;
          obj.latitude = element.latitude;
          this.sailProList.push(obj);
        });
        // 获取敌方信息，填入表格 TODO
        getTargets().then(data => {
          this.targetList = [];
          
          data.forEach(element => {
            let obj = {};
            obj.id = element.id;
            obj.type = element.type;
            obj.distance = element.distance;
            obj.speed = element.speed;
            obj.threatRank = element.threatRank;
            obj.elevation = element.elevation;
            obj.longitude = element.longitude;
            obj.latitude = element.latitude;
            this.targetList.push(obj);
          });
          
          map = new BMap.Map("Map"); //创建实例
          var point = new BMap.Point(113.0,10.7);
          /* map.setMapStyle({
            styleJson: addMapStyle
          }); */
          //地图描绘敌军目标
          map.centerAndZoom(point, 9); //初始化并且设立级别
          map.enableScrollWheelZoom(true); //激活滚轮操

          
          /* for (var i in this.targetList) {
            var pt = new BMap.Point(
              this.targetList[i].longitude,
              this.targetList[i].latitude,
              
            );
            //console.log("lon:"+this.targetList[i].longitude+" "+"lat:"+this.targetList[i].latitude);
          } */
          //TODO 如果经纬度null，不画
          addMarker1(this.sailProList, this.targetList);
          addMarker2(this.targetList, this.sailProList);
            //绘制我方战舰
          function addMarker1(ships, targets, selShipMap) {
            for (var i = 0; i < ships.length; i++) {
              var pt = new BMap.Point(ships[i].longitude, ships[i].latitude);
              var myIcon = new BMap.Icon(
                "../../static/image/ships.png",
                new BMap.Size(16, 16)
              );
              var marker = new BMap.Marker(pt, { icon: myIcon }); // 创建标注
              map.addOverlay(marker);
              (function() {
                var selShipInMap = ships[i];
                marker.addEventListener("click", function() {
                  selInMap(this, selShipInMap, targets);
                });
              })();
            }
          }
          
          //绘制敌方战舰
          function addMarker2(targets, ships) {
            
            
            targets.forEach((obj)=>{
                if(obj.longitude==null||obj.latitude==null){
                    var id = obj.id;
                    setTimeout(() => {
                        _this.$notify({
                            title:"错误",
                            message:"目标 "+id+" 经纬度丢失，无法在地图上正常显示！",
                            duration:5000,
                            type:"error",
                            position:"top-left"
                        });
                    }, 1000);
                }
            })
            for (var i = 0; i < targets.length; i++) {
                
                if(targets[i].longitude==null||targets[i].latitude==null){
                    continue;
                }
                
              var pt = new BMap.Point(
                targets[i].longitude,
                targets[i].latitude
              );
              console.log(targets[i].id)
              if (targets[i].type == "小型舰" || targets[i].type == "大型舰"||targets[i].type == "友军" ) {
                  
                var myIconShip = new BMap.Icon(
                  "../../static/image/tgShip.png",
                  new BMap.Size(16, 16)
                );
                var marker = new BMap.Marker(pt, {
                  icon: myIconShip
                }); // 创建标注
                map.addOverlay(marker);
              }

              if (targets[i].type == "空军") {
                var myIconPlane = new BMap.Icon(
                  "../../static/image/tgPlane.png",
                  new BMap.Size(16, 16)
                );
                var marker = new BMap.Marker(pt, {
                  icon: myIconPlane
                }); // 创建标注
                map.addOverlay(marker);
              }

              if (targets[i].type == "导弹"||targets[i].type == "炮弹") {
                var myIconRocket = new BMap.Icon(
                  "../../static/image/tgMissile.png",
                  new BMap.Size(16, 16)
                );

                var marker = new BMap.Marker(pt, {
                  icon: myIconRocket
                }); // 创建标注
                map.addOverlay(marker);
              }

              (function() {
                var selTargetInMap = targets[i];
                marker.addEventListener("click", function() {
                  selInMap2(this, selTargetInMap, ships);
                });
              })();
            }
            
          }


            // 地图点击我方战舰，圈出对方战舰
          function selInMap(thisMarker, selShipInMap, targets) {

            
            _this.selShip = selShipInMap;
            var ps = thisMarker.getPosition();
            if (selMk1) {
              map.removeOverlay(selMk1);
            }
            var selIcon = new BMap.Icon(
              "../../static/image/o.png",
              new BMap.Size(32, 32)
            );
            var point = new BMap.Point(
              selShipInMap.longitude,
              selShipInMap.latitude
            );
            selMk1 = new BMap.Marker(point, {
              icon: selIcon
            });
            map.addOverlay(selMk1);
            getSelTarget(selShipInMap.id).then(data => {
              for (var j in targets) {
                if (targets[j].id == data) {
                  var selTargetInMap = targets[j];
                  _this.selTarget = selTargetInMap;
                  var sContent =
                    '<ul style="margin:0 0 5px 0;padding:0.2em 0">' +
                    '<li style="line-height: 26px;font-size: 15px;">' +
                    '<span style="width: 100px;display: inline-block;">战舰id：</span>' +
                    selShipInMap.id +
                    "</li>" +
                    '<li style="line-height: 26px;font-size: 15px;">' +
                    '<span style="width: 100px;display: inline-block;">战舰类型：</span>' +
                    selShipInMap.type +
                    "</li>" +
                    '<li style="line-height: 26px;font-size: 15px;">' +
                    '<span style="width: 100px;display: inline-block;">攻击目标：</span>' +
                    selTargetInMap.id +
                    "</li>" +
                    "</ul>";
                  var infoWindow = new BMap.InfoWindow(sContent); // 创建信息窗口对象
                  thisMarker.openInfoWindow(infoWindow); //图片加载完毕重绘infowindow
                  console.log(selTargetInMap.id);
                  break;
                }
              }
              var selIcon = new BMap.Icon(
                "../../static/image/o.png",
                new BMap.Size(32, 32)
              );
              var point2 = new BMap.Point(
                selTargetInMap.longitude,
                selTargetInMap.latitude
              );
              //判断地图上是否圈出
              if (selMk2) {
                map.removeOverlay(selMk2);
              }
              selMk2 = new BMap.Marker(
                point2, //起始点的经纬度
                {
                  icon: selIcon
                }
              );
              map.addOverlay(selMk2);
            });
          }

          // 点击敌方战舰，圈出我方战舰
          function selInMap2(thisMarker, selTargetInMap, ships) {
            console.log("选择的敌方id：" + selTargetInMap.id);
            _this.selTarget = selTargetInMap;
            var ps = thisMarker.getPosition();
            if (selMk2) {
              map.removeOverlay(selMk2);
            }
            var selIcon = new BMap.Icon(
              "../../static/image/o.png",
              new BMap.Size(32, 32)
            );
            var point = new BMap.Point(
              selTargetInMap.longitude,
              selTargetInMap.latitude
            );
            selMk2 = new BMap.Marker(point, {
              icon: selIcon
            });
            map.addOverlay(selMk2);
            getSelShip(selTargetInMap.id).then(data => {
              for (var j in ships) {
                if (ships[j].id == data) {
                  var selShipInMap = ships[j];
                  _this.selShip = selShipInMap;

                  var selIcon = new BMap.Icon(
                    "../../static/image/o.png",
                    new BMap.Size(32, 32)
                  );
                  var point1 = new BMap.Point(
                    selShipInMap.longitude,
                    selShipInMap.latitude
                  );
                  //判断地图上是否圈出
                  if (selMk1) {
                    map.removeOverlay(selMk1);
                  }
                  selMk1 = new BMap.Marker(
                    point1, //起始点的经纬度
                    {
                      icon: selIcon
                    }
                  );
                  map.addOverlay(selMk1);
                  var sContent =
                    '<ul style="margin:0 0 5px 0;padding:0.2em 0">' +
                    '<li style="line-height: 26px;font-size: 15px;">' +
                    '<span style="width: 100px;display: inline-block;">目标id：</span>' +
                    selTargetInMap.id +
                    "</li>" +
                    '<li style="line-height: 26px;font-size: 15px;">' +
                    '<span style="width: 100px;display: inline-block;">目标类型：</span>' +
                    selTargetInMap.type +
                    "</li>" +
                    '<li style="line-height: 26px;font-size: 15px;">' +
                    '<span style="width: 100px;display: inline-block;">应对战舰：</span>' +
                    selShipInMap.id +
                    "</li>" +
                    "</ul>";
                  var infoWindow = new BMap.InfoWindow(sContent); // 创建信息窗口对象
                  thisMarker.openInfoWindow(infoWindow); //图片加载完毕重绘infowindow
                  break;
                }
              }
            });
          }
        });
      });
    },

    confirm() {
        let _this = this;
      if(this.selTarget.threatRank!=null){
            var rank = parseInt(this.selTarget.threatRank);
            if(rank<0||rank>5){
                this.$notify({
                    title:"错误",
                    message:"目标 "+this.selTarget.id+" 威胁等级异常，无法进行下一步攻击决策！",
                    duration:5000,
                    type:"error",
                    position:"top-left"
                });
                return;
            }
      }
      if (this.selTarget.type == "友军") {
        this.isFriend = true;
        this.buttonText = "撤退";
      } else {
        this.isFriend = false;
        this.buttonText = "攻击";
        getWeaponInfo(this.selShip.id).then(data => {
            
           let weaponRepo = ["舰艇","机炮","近程导弹","中程导弹","远程导弹"];
           let type = parseInt(data.weaponType);
           if(type>=0&&type<=4){
               this.weaponInfo.weaponType = weaponRepo[type]
           }else{
               this.weaponInfo.weaponType = "未知类型";
               setTimeout(() => {
                    this.$notify({
                        title:"错误",
                        message:"武器类型异常！",
                        duration:5000,
                        type:"error",
                        position:"top-left"
                    });
                }, 500);
           }
           
           let num = parseInt(data.weaponNum);
           if(num>=0&&num<=112){
               this.weaponInfo.weaponNum = num;
           }else{
               this.weaponInfo.weaponNum = num;
               setTimeout(() => {
                    this.$notify({
                        title:"错误",
                        message:"武器数量超出规定范围！",
                        duration:5000,
                        type:"error",
                        position:"top-left"
                    });
                }, 500);
           }
        });
        getAttackInfo(this.selShip.id).then(data => {
            
            let time = parseInt(data.attackTime);
            if(time>0){
                this.attackInfo.attackTime = _this.timeFormat(time);
            }else{
                this.attackInfo.attackTime = time;
                setTimeout(() => {
                    this.$notify({
                        title:"错误",
                        message:"攻击时间异常！",
                        duration:5000,
                        type:"error",
                        position:"top-left"
                    });
                }, 500);
            }

            let angle = parseInt(data.attackAngle);
            if(angle>=0&&angle<=360){
                this.attackInfo.attackAngle = angle;
            }else{
                this.attackInfo.attackAngle = angle;
                setTimeout(() => {
                    this.$notify({
                        title:"错误",
                        message:"攻击角度超出规定范围！",
                        duration:5000,
                        type:"error",
                        position:"top-left"
                    });
                }, 500);
            }


        });
      }
    },
    timeFormat(n) {
        let ns;
        if(n.toString().length<13){
            ns = n*1000
        }else{
            ns = n
        }
        console.log(ns)
        let date = new Date(ns) // 时间戳为10位需乘1000，为13位则不用
        let Y = date.getFullYear() // 年
        let M = (date.getMonth() + 1 < 10 ? '0' + (date.getMonth() + 1) : date.getMonth() + 1) // 月
        let D = date.getDate() < 10 ? '0' + date.getDate() + '' : date.getDate() + '' // 日

        let h = date.getHours() < 10 ? '0' + date.getHours() : date.getHours() // 时
        let m = date.getMinutes() < 10 ? '0' + date.getMinutes() : date.getMinutes() // 分
        let s = date.getSeconds() < 10 ? '0' + date.getSeconds() : date.getSeconds() // 秒

        // 一个函数只能有一个return，以下仅做示例
        return Y + '-' + M + '-' + D + ' ' + h + ':' + m + ':' + s// yyyy-mm-dd hh:mm:ss
    },
    attack() {
       if(this.selTarget.type != "友军"&&(this.weaponInfo.weaponType==""||this.weaponInfo.weaponNum==""||
        this.attackInfo.attackTime==""||this.attackAngle=="")){
            this.$notify({
                title:"错误",
                message:"请先确认目标！",
                duration:5000,
                type:"error",
                position:"top-left"
            });
            return;
        }
      let errorLog = "无法进行攻击，原因: ";
       let hasError = false;
      if(this.weaponInfo.weaponType=="未知类型"){
          errorLog += "未知的武器类型！"
          hasError = true;
      }
      if(this.weaponInfo.weaponNum<0||this.weaponInfo.weaponNum>112){
          errorLog += "武器数量超出规定范围！"
          hasError = true;
      }
      if(Number.isInteger(this.attackInfo.attackTime)){
          errorLog += "攻击时间异常！"
          hasError = true;
      }
      if(this.attackInfo.attackAngle<0||this.attackInfo.attackAngle>360){
          errorLog += "攻击角度超出规定范围！"
          hasError = true;
      }
        if(hasError){
            this.$notify({
                title:"错误",
                message:errorLog,
                duration:5000,
                type:"error",
                position:"top-left"
            });
            return;
        }
      

      var startLong = this.selShip.longitude;
      var startLat = this.selShip.latitude;
      var endLong = this.selTarget.longitude;
      var endLat = this.selTarget.latitude;
      //如果是友军则画撤退箭头，并且标注友军
      if (this.selTarget.type == "友军") {
        var endLong2 = startLong - (endLong - startLong) / 8;
        var endLat2 = startLat - (endLat - startLat) / 8;
        var polyline = new BMap.Polyline(
          [
            new BMap.Point(startLong, startLat),
            new BMap.Point(endLong2, endLat2)
          ],
          { strokeColor: "yellow", strokeWeight: 3, strokeOpacity: 0.5 }
        );
        console.log(startLong + "" + startLat);
        console.log(endLong2 + "" + endLat2);
        map.addOverlay(polyline);
        addArrow(polyline, 10, Math.PI / 7);

        var ourPt = new BMap.Point(endLong, endLat);
        var ourIcon = new BMap.Icon(
          "../../static/image/oOur.png",
          new BMap.Size(32, 32),
          {
            imageOffset: new BMap.Size(0, 0)
          }
        );
        var ourMk = new BMap.Marker(ourPt, {
          icon: ourIcon
        }); // 创建标注
        map.addOverlay(ourMk);

        function addArrow(polyline, length, angleValue) {
          //绘制箭头的函数
          var linePoint = polyline.getPath(); //线的坐标串
          var arrowCount = linePoint.length;
          for (var i = 1; i < arrowCount; i++) {
            //在拐点处绘制箭头
            var pixelStart = map.pointToPixel(linePoint[i - 1]);
            var pixelEnd = map.pointToPixel(linePoint[i]);
            var angle = angleValue; //箭头和主线的夹角
            var r = length; // r/Math.sin(angle)代表箭头长度
            var delta = 0; //主线斜率，垂直时无斜率
            var param = 0; //代码简洁考虑
            var pixelTemX, pixelTemY; //临时点坐标
            var pixelX, pixelY, pixelX1, pixelY1; //箭头两个点
            if (pixelEnd.x - pixelStart.x == 0) {
              //斜率不存在时
              pixelTemX = pixelEnd.x;
              if (pixelEnd.y > pixelStart.y) {
                pixelTemY = pixelEnd.y - r;
              } else {
                pixelTemY = pixelEnd.y + r;
              }
              //已知直角三角形两个点坐标及其中一个角，求另外一个点坐标算法
              pixelX = pixelTemX - r * Math.tan(angle);
              pixelX1 = pixelTemX + r * Math.tan(angle);
              pixelY = pixelY1 = pixelTemY;
            } //斜率存在时
            else {
              delta = (pixelEnd.y - pixelStart.y) / (pixelEnd.x - pixelStart.x);
              param = Math.sqrt(delta * delta + 1);

              if (pixelEnd.x - pixelStart.x < 0) {
                //第二、三象限
                pixelTemX = pixelEnd.x + r / param;
                pixelTemY = pixelEnd.y + (delta * r) / param;
              } //第一、四象限
              else {
                pixelTemX = pixelEnd.x - r / param;
                pixelTemY = pixelEnd.y - (delta * r) / param;
              }
              //已知直角三角形两个点坐标及其中一个角，求另外一个点坐标算法
              pixelX = pixelTemX + (Math.tan(angle) * r * delta) / param;
              pixelY = pixelTemY - (Math.tan(angle) * r) / param;

              pixelX1 = pixelTemX - (Math.tan(angle) * r * delta) / param;
              pixelY1 = pixelTemY + (Math.tan(angle) * r) / param;
            }

            var pointArrow = map.pixelToPoint(new BMap.Pixel(pixelX, pixelY));
            var pointArrow1 = map.pixelToPoint(
              new BMap.Pixel(pixelX1, pixelY1)
            );
            var Arrow = new BMap.Polyline(
              [pointArrow, linePoint[i], pointArrow1],
              { strokeColor: "yellow", strokeWeight: 3, strokeOpacity: 0.5 }
            );
            map.addOverlay(Arrow);
          }
        }
        setTimeout(() => {
            getResult(this.selShip.id).then(data => {
                let damageTg = parseFloat(parseFloat(data.damageTg).toFixed(4));
                data.damageTg = damageTg;
                if(damageTg<0||damageTg>1){
                    this.$notify({
                        title:"错误",
                        message:"损伤数值异常！",
                        duration:5000,
                        type:"error",
                        position:"top-left"
                    });
                } 
                this.resultList.push(data);
                
            });
        }, 3500);
        
      }
      //是敌军攻击动画
      else {
        var longStep = (endLong - startLong) / 600;
        var latSetp = (endLat - startLat) / 600;
        var nextLong = startLong + longStep;
        var nextLat = startLat + latSetp;
        var count = 0; //设置执行次数
        var interval = setInterval(goWay, 2);
        var attackMk;
        var myIcon = new BMap.Icon(
          "../../static/image/daodan.png",
          new BMap.Size(32, 32),
          {
            imageOffset: new BMap.Size(0, 0)
          }
        );
        setTimeout(() => {
            getResult(this.selShip.id).then(data => {
                let damageTg = parseFloat(parseFloat(data.damageTg).toFixed(4));
                data.damageTg = damageTg;
                if(damageTg<0||damageTg>1){
                    this.$notify({
                        title:"错误",
                        message:"损伤数值异常！",
                        duration:5000,
                        type:"error",
                        position:"top-left"
                    });
                } 
                this.resultList.push(data);
            });
        }, 3500);
      }

      function goWay() {
        startLong = nextLong;
        startLat = nextLat;
        nextLong = getRound(nextLong);
        nextLat = getRound1(nextLat);

        drawIcon(startLong, startLat, nextLong, nextLat);
        count++;
        if (count == 599) {
          var destoryPt = new BMap.Point(endLong, endLat);
          var destoryIcon = new BMap.Icon(
            "../../static/image/x.png",
            new BMap.Size(32, 32)
          );
          var destoryMk = new BMap.Marker(destoryPt, {
            icon: destoryIcon
          }); // 创建标注
          map.removeOverlay(attackMk);
          map.addOverlay(destoryMk);

          clearInterval(interval); //关闭定时器
        }
      }
      function getRound(temp) {
        return temp + longStep;
      }
      function getRound1(temp) {
        return temp + latSetp;
      }

      function drawGreenLine(startLong, startLat, nextLong, nextLat) {
        var polyline = new BMap.Polyline(
          [
            new BMap.Point(startLong, startLat), //起始点的经纬度
            new BMap.Point(nextLong, nextLat) //终止点的经纬度
          ],
          {
            strokeColor: "yellow", //设置颜色
            strokeWeight: 1, //宽度
            strokeOpacity: 1
          }
        ); //透明度
        map.addOverlay(polyline);
      }

      function drawIcon(startLong, startLat, nextLong, nextLat) {
        if (attackMk) {
          map.removeOverlay(attackMk);
        }
        attackMk = new BMap.Marker(
          new BMap.Point(nextLong, nextLat), //起始点的经纬度
          {
            icon: myIcon
          }
        );
        map.addOverlay(attackMk);
        drawGreenLine(startLong, startLat, nextLong, nextLat);
      }
    },

    detail() {

        // getSelTarget(2);

      this.detailDialogTitle = "详细作战报告";
      this.detailDialogVisible = true;
      getWarInfo().then(data => {
        this.warList = [];
        data.forEach(element => {
          let obj = {};
          obj.shipId = element.shipId;
          obj.targetId = element.targetId;
          obj.targetType = element.targetType;
          obj.weaponType = element.weaponType;
          obj.weaponNum = element.weaponNum;
          obj.attackTime = element.attackTime;
          obj.attackAngle = element.attackAngle;
          obj.damageOur = element.damageOur;
          obj.damageTg = element.damageTg;
          this.warList.push(obj);
        });
      });
    },

    changeSelectID(selTarget){
        this.selTarget =selTarget;
    },
    //我方战舰表选择战舰
    selectShip(row, event, column) {
      //获取选择的己方战舰selShip
      var _this = this;
      for (var i in this.sailProList) {
        if (this.sailProList[i].id == row.id) {
          this.selShip = this.sailProList[i];
          //获取与选择的己方战舰匹配的处理目标
          getSelTarget(this.selShip.id).then(data => {
            for (var j in this.targetList) {
              if (this.targetList[j].id == data) {
                this.selTarget = this.targetList[j];
                break;
              }
            }
            
            
            
            var selIcon = new BMap.Icon(
              "../../static/image/o.png",
              new BMap.Size(32, 32)
            );
            var point1 = new BMap.Point(
              this.selShip.longitude,
              this.selShip.latitude
            );
            var point2 = new BMap.Point(
              this.selTarget.longitude,
              this.selTarget.latitude
            );
            //判断地图上是否圈出
            if (selMk1) {
              map.removeOverlay(selMk1);
            }
            if (selMk2) {
              map.removeOverlay(selMk2);
            }
            selMk1 = new BMap.Marker(
              point1, //起始点的经纬度
              {
                icon: selIcon
              }
            );
            selMk2 = new BMap.Marker(
              point2, //起始点的经纬度
              {
                icon: selIcon
              }
            );
            map.addOverlay(selMk1);
            map.addOverlay(selMk2);
          });
        }
      }
    },

    //从子组件表传递的目标信息显示在右边
    showStyleFromChild: function(data) {
      this.selTarget = data;
      var _this = this;
      
      this.weaponInfo = {weaponType:"",weaponNum:""};
      this.attackInfo = {attackTime:"",attackAngle:""};  
      getSelShip(this.selTarget.id).then(data => {
        for (var j in this.sailProList) {
          if (this.sailProList[j].id == data) {
            this.selShip = this.sailProList[j];
            var selIcon = new BMap.Icon(
              "../../static/image/o.png",
              new BMap.Size(32, 32)
            );
            var point1 = new BMap.Point(
              this.selShip.longitude,
              this.selShip.latitude
            );
            var point2 = new BMap.Point(
              this.selTarget.longitude,
              this.selTarget.latitude
            );
            //判断地图上是否圈出
            if (selMk1) {
              map.removeOverlay(selMk1);
            }
            if (selMk2) {
              map.removeOverlay(selMk2);
            }
            selMk1 = new BMap.Marker(
              point1, //起始点的经纬度
              {
                icon: selIcon
              }
            );
            selMk2 = new BMap.Marker(
              point2, //起始点的经纬度
              {
                icon: selIcon
              }
            );
            map.addOverlay(selMk1);
            map.addOverlay(selMk2);
            break;
          }
        }
      });
        if(this.selTarget.threatRank!=null){
            var rank = parseInt(this.selTarget.threatRank);
            if(rank<0||rank>5){
                _this.$message({
                    showClose:"true",
                    message:"目标 "+this.selTarget.id+" 威胁等级异常！",
                    duration:10000,
                    type:"error"
                });
            }
        }
        if(this.selTarget.longitude==null||this.selTarget.latitude==null){
            _this.$message({
                showClose:"true",
                message:"目标 "+this.selTarget.id+" 经纬度丢失，无法在地图上正常显示！",
                duration:10000,
                type:"error"
            });
        }


      if (this.selTarget.type === "小型舰" || this.selTarget.type === "大型舰" || this.selTarget.type == "友军") {
        this.$router.push("/targetSail");
        this.seen = false;
      }
      if (this.selTarget.type === "空军") {
        this.$router.push("/targetSky");
        this.seen = false;
      }
      if (this.selTarget.type === "导弹" || this.selTarget.type === "炮弹") {
        this.$router.push("/targetMissile");
        this.seen = true;
      }
      this.form.targetId = this.selTarget.id;
      this.form.distance = this.selTarget.distance;
      this.form.speed = this.selTarget.speed;
      this.form.targetType = this.selTarget.type;
      this.form.threatRank = this.selTarget.threatRank;
    },

    //我方表格选中样式
    tableRowClassName({ row, rowIndex }) {
      if (row.id === this.selShip.id) {
        return "select-row1";
      }
      return "";
    },
    tableRowClassDamage({ row, rowIndex }) {
     
        if (row.damageTg >1 ||row.damageTg <0) {
            return "select-row";
        }   
        return "";
    
    },
    tableRowClassDamage2({ row, rowIndex }) {
     
       
        return "select-row";
    
    },



  },
  mounted() {
    map = null;
    this.initMap();
  }
};
</script>

<style>
.result-list > .el-table th {
  background: #424e62;
  padding: 5px 0;
  font-size: 15px;
}

.result-list > .el-table td {
  background: #424e62;
  padding: 5px 0;
  font-size: 16px;
}

.result-list .el-table th > .cell {
  position: relative;
  word-wrap: normal;
  text-overflow: ellipsis;
  vertical-align: middle;
  width: 100%;
  box-sizing: border-box;
  background-color: #424e62;
}

.result-list {
  margin-left: 10px;
  line-height: 23px;
  width: 442px;
  border: 1px solid #a09d9d;
  float: left;
}

.cell-grey{
    background: #f2f2f2;
}

</style>





