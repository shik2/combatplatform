<template>
    <el-table @row-click="selectTarget" :row-class-name="tableRowClassName" :data="targetList" v-loading="loading" element-loading-background="rgba(0, 0, 0, 0.5)" height="240">
        <el-table-column prop="id" label="目标编号" width="106" align="center"></el-table-column>
        <el-table-column prop="type" label="种类" width="117" align="center"></el-table-column>
    </el-table>
</template>

<script>
import { getTgMissile } from "@/api/api.js";
export default {
    name: "TargetMissile",
    props: ["selShip", "selTargetFP"],
    data: function() {
        return {
            loading: false,
            targetList: [],
            selTarget:""
        };
    },
    watch: {
        selTargetFP: function(newVal, oldVal) {
           // console.log("eeeeee");
            this.selTarget = newVal;
            this.$emit("listenToChild", this.selTarget); //选择的敌方id传回父组件，右表对应的信息也修改
        }
    },
    methods: {
        setTableData() {
            this.loading = true;
            getTgMissile().then(data => {
                this.targetList = [];
                console.log("swtich")
                console.log(data);
                var _this = this;
                data.forEach(element => {
                    let obj = {};
                    obj.id = element.id;
                    obj.type = element.type;
                    obj.elevation = element.elevation;
                    obj.distance = element.distance;
                    obj.speed = element.speed;
                    obj.threatRank = element.threatRank;
                    
                    if(parseInt(obj.threatRank)>5||parseInt(obj.threatRank)<0){
                        
                        setTimeout(() => {
                            _this.$notify({
                                title:"错误",
                                message:"目标 "+obj.id+" 威胁等级异常！",
                                duration:5000,
                                type:"error",
                                position:"top-left"
                            });
                        }, 1000);
                        
                    }
                    obj.longitude = element.longitude;
                    obj.latitude = element.latitude;
                    this.targetList.push(obj);
                });
                this.loading = false;
                

            });
            
        },
        selectTarget(row, event, column) {
            for (var i in this.targetList) {
                if (this.targetList[i].id == row.id) {
                    this.selTarget = this.targetList[i];
                    break;
                }
            }
            
            this.$emit("listenToChild", this.selTarget);
            //this.selTargetFP(this.selTarget);
        },
        tableRowClassName({ row, rowIndex }) {
            if (row.id === this.selTarget.id) {
                return "select-row";
            }
            return "";
        }
    },
    mounted() {
        this.setTableData();
    }
};
</script>

<style>
</style>
