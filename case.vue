<template>
  <border :zoomIcon="zoomIcon" componentName="cases">
    <template #title>
      <div class="title-row">
        <span class="title" @click="clickTitle">排水工况</span>
        <div class="right">
          <el-radio-group v-model="radio" @change="changeType">
            <el-radio :label="1">污水</el-radio>
            <el-radio :label="2">雨水</el-radio>
          </el-radio-group>
        </div>
      </div>
    </template>
    <el-row type="flex" class="header" v-if="zoomIcon">
      <el-col :span="8">
        <el-dropdown>
          <div style="color:#fff;font-size:36px;" class="el-dropdown-link">
            <img src="../../../assets/images/screen/work/type1.png" class="icon" />
            排水报警
            <span :class="['type', {'type1': psNum != 0}]">{{psNum}}</span>
          </div>
          <el-dropdown-menu slot="dropdown">
            <el-dropdown-item><div @click="showList(1)">冒溢报警<span class="num">{{ mybjNum }}</span></div></el-dropdown-item>
            <el-dropdown-item><div @click="showList(2)">排水报警<span class="num">{{ psbjNum }}</span></div></el-dropdown-item>
            <el-dropdown-item><div @click="showList(3)">设备异常<span class="num">{{ sbbjNum }}</span></div></el-dropdown-item>
          </el-dropdown-menu>
        </el-dropdown>
      </el-col>
      <el-col :span="8">
        <img src="../../../assets/images/screen/work/type2.png" class="icon" />
        淤堵报警
        <span :class="['type', {'type2': ydNum != 0}]" @click="showList(4)">{{ydNum}}</span>
      </el-col>
      <el-col :span="8">
        <img src="../../../assets/images/screen/work/type4.png" class="icon" />
        风险点
        <span :class="['type', {'type4': fxdNum != 0}]" @click="showList(5)">{{fxdNum}}</span>
      </el-col>
    </el-row>
    <el-row type="flex" class="header" v-if="!zoomIcon">
      <el-col :span="5">
        <img src="../../../assets/images/screen/work/type1.png" class="icon" />
        排水报警
        <span :class="['type', {'type1': ydNum != 0}]" @click="showList(2)">{{psbjNum}}</span>
      </el-col>
      <el-col :span="5">
        <img src="../../../assets/images/screen/work/type3.png" class="icon" />
        冒溢报警
        <span :class="['type', {'type1': ydNum != 0}]" @click="showList(1)">{{mybjNum}}</span>
      </el-col>
      <el-col :span="5">
        <img src="../../../assets/images/screen/work/type5.png" class="icon" />
        设备报警
        <span :class="['type', {'type1': ydNum != 0}]" @click="showList(3)">{{sbbjNum}}</span>
      </el-col>
      <el-col :span="5">
        <img src="../../../assets/images/screen/work/type2.png" class="icon" />
        淤堵报警
        <span :class="['type', {'type2': ydNum != 0}]" @click="showList(4)">{{ydNum}}</span>
      </el-col>
      <el-col :span="4">
        <img src="../../../assets/images/screen/work/type4.png" class="icon" />
        风险点
        <span :class="['type', {'type4': fxdNum != 0}]" @click="showList(5)">{{fxdNum}}</span>
      </el-col>
    </el-row>
    <div class="psgk-img-box">
      <div class="switchAreaBtn" @click="changeAreaName">+</div>
      <div id="draw" ref="draw"></div>
      <div id="draw-inside"></div>
      <div id="draw-outside"></div>
      <div id="area-name">{{areaName}}</div>
      <div class="arrow-scroll">
        <span>
          <div id="arrow">
          </div>
        </span>
      </div>
      <div class="pipe_container">
        <div v-for="item in pipeList" :key="item.imei" :style="'height:' + item.divHeight + '%;' + 'top:' + (100 - item.divHeight) + '%'" :class="[{'item-hidden':!item.show},{'line-red':item.show && item.data_now > item.max},{'line-yellow':item.show && item.data_now < item.min},'item']">
          <template v-if="item.show">
            <div class="now">{{item.data_now}}</div>
            <div class="water_container">
              <div class="water" :style="'height:' + item.rotate3_height + 'px;'"></div>
              <div class="water_top1" :style="'bottom:' + (item.rotate3_height - 6) + 'px;'"></div>
            </div>
            <div id="rotate1" :style="'bottom:' + item.rotate1_height + 'px;'"></div>
            <div id="rotate2" :style="'bottom:' + item.rotate2_height + 'px;'"></div>
            <div id="rotate3" :style="'bottom:' + item.rotate3_height + 'px;'"></div>
          </template>
        </div>
      </div>
    </div>
    <div class="area_container">


    </div>
  </border>
</template>

<script>
import border from "../components/border";
import customTable from "../components/customTable";
import axios from 'axios';
import {getYDList} from '@/api/screen/index'
export default {
  name:'cases',
  props: {
    zoomIcon: {
      type: Boolean,
      default: true
    }
  },
  components: {
    border,
    customTable,
  },
  data() {
    return {
      radio:1,
      psNum:0,
      ydNum:0,
      fxdNum:0,
      mybjNum: 0,
      psbjNum: 0,
      sbbjNum: 0,
      curArea:'颛桥',
      areaName:'W1',
      index:0,
      WAreaList:['W1','W2','W3','W4'],
      YAreaList:['Y1','Y2','Y3','Y4','Y5','Y6'],
      pipeList:[]

    }
  },
   created () {
    this.getProblemsList()
    this.getMainPipe()
    this.getYDList()
  },
  methods:{
    clickTitle() {
      // 恢复地图初始化（关闭街镇聚焦及街镇管道窨井图层）
      this.$EventBus.$emit('handleEvent', {
        method: "doBackMapInit",
        data: [false]
      });
      // 关闭属性描述窗口
      this.$EventBus.$emit("descClose");
      // 关闭设备信息弹窗
      this.$EventBus.$emit('closeDeviceDialog')
      // 关闭table弹窗
      this.$EventBus.$emit("showWorkDialog", {visiable:false, type: ''});
    },
    async getYDList () {
      const {data} = await getYDList()
      this.ydNum = data.length  
    },
    changeAreaName () {
      let length = this.radio == 1 ? this.WAreaList.length : this.YAreaList.length
      this.index = this.index == (length - 1) ?  0 : this.index + 1
      this.areaName = this.radio == 1  ? this.WAreaList[this.index] : this.YAreaList[this.index]
      this.getMainPipe()
    },


    getMainPipe()
    {
      axios.post("http://49.232.124.52:1005/main_pipe",{
        area: this.areaName
      }).then((Response)=>{
        this.pipeList = Response.data.data

        // 获取最高的柱子高度
      let maxHeight = Response.data.data.map(item => {return item.pipeheigh})
                          .sort((a,b) => a - b)
                          .reverse()[0]
      
      let unitPixelpercent = 50 / maxHeight

      let drawHeight = this.$refs.draw.clientHeight // draw高度
      let insideHeight = drawHeight / 2 // draw一半高度
      
      this.pipeList.forEach(item => {
        // 计算每根柱子的高度
        item.divHeight = item.pipeheigh ?  item.pipeheigh * unitPixelpercent + 50 : 100 // %
        let time = new Date(item.receive_time)
        let now = new Date()
        item.show = (now - time) / 1000 / 60 < 40
        // 计算三根警戒线的位置
        item.rotate1_height = item.max * (insideHeight  / maxHeight ) + insideHeight
        item.rotate2_height = item.min * (insideHeight / maxHeight ) + insideHeight
        item.rotate3_height = item.data_now * (insideHeight / maxHeight ) + insideHeight                   

      })
      })
    },

    
    getProblemsList()
    {
      axios.post("http://49.232.124.52:1005/problems",{
      }).then((Response)=>{
        let res = Response.data.data;
        let {cznum,devices, mynum, pw} = res
        this.psbjNum = cznum
        this.sbbjNum = devices.length
        this.mybjNum = mynum
        this.psNum = this.psbjNum + this.sbbjNum + this.mybjNum
        // this.ydNum = level1.length
        this.fxdNum = pw.length
      })
    },



    showList(type){
      // 恢复地图初始化（关闭街镇聚焦及街镇管道窨井图层）
      this.$EventBus.$emit('handleEvent', {
        method: "doBackMapInit",
        data: [false]
      });
      // 关闭属性描述窗口
      this.$EventBus.$emit("descClose");
      this.$EventBus.$emit("showWorkDialog", {visiable:true, type});
    },
    changeType (val) {
      this.index = 0
      this.areaName = val == 1 ? this.WAreaList[this.index] : this.YAreaList[this.index]
   
   
      this.getMainPipe()
    },
    toDetail (id) {
      // if (id != '04') return
      let url = this.radio == 1 ? window.maintainURL : window.mainYSURL
      window.location.href = url + '?area='+ this.areaName
    },
  }
}
</script>

<style lang="scss" scoped>
  .title-row {
    display: flex;
    justify-content: center;
    position: relative;
    font-size: 26px;
    margin-bottom: 10px;

    .right{
      position: absolute;
      right: 20px;

      .pipe-btn {
        padding: 5px 8px;
        margin-right: 10px;
        cursor: pointer;
        border-radius: 5px;
        background-color: rgba(0,175,255,0.3);
      }

      ::v-deep .el-radio-group {
        .el-radio {
          font-size: 32px;
          display: inline-flex;
          align-items: center;

          .el-radio__label {
            color: #c6e4ff;
            font-size: 32px;
          }

          .el-radio__input {
            /*background-color: transparent;*/
            width: 20px;
            height: 20px;
            border-width: 2px;

            .el-radio__inner {
              width: 20px;
              height: 20px;
              background-color: rgba(57, 158, 252, 0.21);
              border-color: #399efc;
              top: -5px;


              &:after {
                width: 10px;
                height: 10px;
              }
            }

            &.is-checked .el-radio__inner::after{
              background-color: rgba(255, 216, 0, 0.92);
            }
          }

        }

      }

    }

  }
  .header {
    font-size: 32px;
    .el-col {
      display: flex;
      align-items: center;
      justify-content: center;
    }
    .icon {
      height: 32px !important;
      vertical-align:top;
      margin-right: 7px;
    }
    [class*='type'] {
      font-weight: bolder;
      font-size: 40px;
      letter-spacing: 2px;
      font-family: REEJI-Sharp, serif;
      margin-left: 7px;
      cursor: pointer;
      vertical-align:middle;
    }
    .type{
      color: #42ef93;
    }
    .type1{
      color: #ee2222;
    }
    .type2{
      // margin-left:12px;
      color: #ee2222;
    }
    .type3{
      margin-left:12px;
      color: #ffd32a;
    }
    .type4{
      color: #ee2222;
    }
  }
  .area_container{
    width: 100%;
    display: flex;
    justify-content: space-between;
    .item{
      font-size: 30px;
      color: #fff;
      .item_data{
        width: 50px;
        height: 50px;
        text-align: center;
        line-height: 50px;
        border-radius: 6px;
        background: #2EBEFA;
        cursor: pointer;
      }
      .item_name{
        margin-top: 10px;
        width: 50px;
        // line-height: 36px;
        text-align: center;
      }
      .cur_name{
        color: #fede05;
      }
    }
  }
  .psgk-img-box{
    position: relative;
    height: 300px;
    margin-top: 40px;
  }
  #draw {
	position: absolute;
	top: 0%;
	left: 5%;
	width: 90%;
	height: 95%;
	z-index: 2;
}
.switchAreaBtn{
  font-size: 70px;
  position: absolute;
  cursor: pointer;
}
#draw-inside{
	position: absolute;
	bottom: 15%;
	left: 5%;
	width: 90%;
	height: 35%;
	z-index: 1;
	box-shadow: 0px 0px 8px 1px rgba(239, 239, 239, 0.6);
}
#draw-outside{
	position: absolute;
	bottom: 15%;
	left: 5%;
	width: 90%;
	height: 35%;
	/* background-color: #121D3D; */
    background: url("../../../assets/images/screen/work/h.png") center no-repeat;
	// background-image: url(../resource/index/h.png);
	box-sizing: border-box;
	background-size: 100% 100%;
	z-index: 9;

}


#rotate1 {
	height: 3%;
	width: 200%;
	position: absolute;
  left: 50%;
  transform: translateX(-50%);
	// top: 22%;
	border-top: 8px dashed rgba(213, 79, 69, 0.8);
	pointer-events: none;
	z-index: 10;
}

#rotate2 {
	height: 3%;
  width: 200%;
	position: absolute;
  left: 50%;
  transform: translateX(-50%);
	border-bottom: 8px dashed rgba(213, 213, 103, 0.8);
	// top: 35%;
	pointer-events: none;
	z-index: 10;
}
#rotate3 {
	height: 3%;
  width: 200%;
	position: absolute;
  left: 50%;
  transform: translateX(-50%);
	// top: 49%;
	border-bottom: 8px dashed rgba(47, 136, 213, 0.8);
	pointer-events: none;
	z-index: 10;
}

#area-name{
	position: absolute;
	bottom: 18%;
	height: 20%;
	right: 5%;
	width: 20%;
	text-align: center;
	font-size: 32px;
	font-weight: bold;
	color: yellow;
	z-index: 12;
	
}


.arrow-scroll {
	overflow: hidden;
	white-space: nowrap;
	position: absolute;
	bottom: 24%;
	width: 60%;
	height: 15%;
	left: 5%;
	z-index: 5;
	z-index: 12;
}

.arrow-scroll span {
	display: inline-block;
	width: 100%;
	height: 100%;
	/* height: 50px; */
	animation: arrow-scroll 3s infinite linear;

}

.arrow-scroll span:before {
	padding-right: 10px;
}
@keyframes arrow-scroll {
	0% {
		transform: translateX(100%);
	}

	100% {
		transform: translateX(-100%);
	}
}
.item-hidden{
  background: #878787;
}
.line-red{
		box-shadow: 0px 0px 15px 4px rgba(255, 11, 47, 1.0) !important;
	}
	.line-yellow{
		box-shadow: 0px 0px 12px 4px rgba(255, 215, 12, 1.0) !important;
	}

#arrow {
    background: url("../../../assets/images/screen/work/arrow.png") center no-repeat;
	// background-image: url(../resource/index/arrow.png);
	background-repeat: no-repeat;
	background-position: center;
	background-size: 100% 100%;
	height: 100%;
	width: 30%;
}
.pipe_container{
  width: 90%;
  height: 95%;
  margin: 0 auto;
  display: flex;
  justify-content: space-around;
  .item{
    width: 60px;
    height: 100%;
    box-shadow: 0px 0px 8px 1px rgba(239, 239, 239, 0.6);
    box-sizing: border-box;
    position: relative;
    // overflow: hidden;
    .now{
      position: absolute;
      top: -25px;
      left: 50%;
      transform: translateX(-50%);
      font-size: 24px;
      width: 80px;
      text-align: center;
    }
    .water_container{
      position: relative;
      height: 100%;
      width: 100%;
      overflow: hidden;
    }
    .water{
      position: absolute;
      // height: 100%;
      width: 100%;
      bottom:0;
      background: url("../../../assets/images/screen/work/z.png") center no-repeat;
      background-size: 100% 100%;
    }
    .water_top1{
      position: absolute;
      bottom:0;
      height: 20px;
      width: 100%;
      background: url("../../../assets/images/screen/work/波浪3.png") center no-repeat;
      background-size: 100% 100%;
    }
  }
}
.el-dropdown-menu{
  // width:200px;
  padding:15px 0;
  text-align:center;
  background:rgb(1, 21, 58);
  border:2px solid rgba(57, 158, 252, 0.53);
  .el-dropdown-menu__item{
    font-size:36px;
    line-height:70px;
    padding: 0 25px;
    color:#fff;
    color:#fff;
    &:hover{
      background:rgb(7, 45, 115);
  // background:rgb(5, 43, 113);
    }
    .num{
      font-family: REEJI-Sharp, serif;
      color: #ee2222;
      font-size:40px;
      font-weight: 600;
      vertical-align: middle;
      margin-left:20px;
    }
  }
}
</style>