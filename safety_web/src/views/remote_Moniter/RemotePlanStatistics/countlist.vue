<template>
	<div>
		<el-breadcrumb separator="/" style="margin-bottom: 20px;">
			<el-breadcrumb-item :to="{ path: '/remote_Moniter/RemotePlanStatistics' }">远程计划统计</el-breadcrumb-item>
			<el-breadcrumb-item>远程计划统计表</el-breadcrumb-item>
		</el-breadcrumb>
		<div class="page-content">
			<el-row>
				<el-form :inline="true">
					<el-form-item style="float:right">
						<el-button type="warning" icon="el-icon-download " @click="output()">导出</el-button>
					</el-form-item>
				</el-form>
			</el-row>
			<!-- 计划列表 -->
			<el-row style="padding:10px; border-top: 2px dashed #dddddd;text-align:center">
				<el-table :data="listData" style="width: 100%;text-align:center" ref="treeTable" :indent="30" max-height="560"
				 highlight-current-row border>
					<el-table-column type="index" label="序号" width="60" align="center" show-overflow-tooltip></el-table-column>
					<el-table-column prop="sumDate" label="日期" width="120" align="center" show-overflow-tooltip> </el-table-column>
					<el-table-column prop="companyName" label="基层单位" width="200" align="center" show-overflow-tooltip> </el-table-column>
					<el-table-column prop="workNum" label="开工项目数量" width="120" align="center" show-overflow-tooltip> </el-table-column>
					<el-table-column prop="dayReportNum" label="日报数量" align="center" show-overflow-tooltip> </el-table-column>
					<el-table-column prop="recordDeviceNum" label="配备记录仪数量" width="150" align="center" show-overflow-tooltip>
					</el-table-column>
					<el-table-column prop="outStockNum" 	 label="出库数量" width="100" align="center" show-overflow-tooltip>
					</el-table-column>
					<el-table-column prop="powerOnNum" label="开机使用数量" width="150" align="center" show-overflow-tooltip> </el-table-column>
					<el-table-column prop="backNum" label="备用数量" width="100" align="center" show-overflow-tooltip> </el-table-column>
					<el-table-column prop="coverageRate" label="覆盖率" width="100" align="center" show-overflow-tooltip>
					</el-table-column>
					<el-table-column prop="availableRate" label="利用率" width="100" align="center" show-overflow-tooltip>
					</el-table-column>
					<el-table-column prop="useRate" label="使用率" width="100" align="center" show-overflow-tooltip> </el-table-column>
					<el-table-column label="操作" align="center" width="180" show-overflow-tooltip fixed="right">
						<template slot-scope="scope">
							<el-button type="primary" size="mini" icon="el-icon-edit" @click="handelcelchange(scope.row)">编辑</el-button>
							<el-button type="danger" size="mini" icon="el-icon-delete" @click="handelcelDelete(scope.row)">删除</el-button>
						</template>
					</el-table-column>
				</el-table>
			</el-row>
			<el-dialog title="新增计划" :visible.sync="tochange" width="700px" :close-on-click-modal="false">
				<el-form label-width="120px" style="width:100%;">
					<el-row>
						<el-col :span="24">
							<el-form-item label="日期:" prop="sumDate">
								{{resData.sumDate}}
							</el-form-item>
							<el-form-item label="基层单位:" prop="companyName">
								{{resData.companyName}}
							</el-form-item>
							<el-form-item label="开工项目数量:" prop="workNum">
								<el-input type="text" label="开工项目数量:" class="resizeNone" v-model="resData.workNum" placeholder="请输入内容"></el-input>
							</el-form-item>
							<el-form-item label="日报数量:" prop="dayReportNum">
								<el-input type="text" label="日报数量: " class="resizeNone" v-model="resData.dayReportNum" placeholder="请输入内容"></el-input>
							</el-form-item>
							<!-- <el-form-item label="配备记录仪数量:" prop="recordDeviceNum">
								<el-input type="text" label="配备记录仪数量:" class="resizeNone" v-model="resData.recordDeviceNum" placeholder="请输入内容"></el-input>
							</el-form-item>
							<el-form-item label="出库数量:" prop="outStockNum" >
								<el-input type="text" label="出库数量:" class="resizeNone" v-model="resData.outStockNum" placeholder="请输入内容"></el-input>
							</el-form-item>
							<el-form-item label="开机使用数量:" prop="powerOnNum" >
								<el-input type="text" label="开机使用数量:" class="resizeNone" v-model="resData.powerOnNum" placeholder="请输入内容"></el-input>
							</el-form-item>
							<el-form-item label="备用数量:" prop="backNum">
								<el-input type="text" label="备用数量:" class="resizeNone" v-model="resData.backNum" placeholder="请输入内容"></el-input>
							</el-form-item> -->
						</el-col>
					</el-row>
				</el-form>
				<span slot="footer" class="dialog-footer">
					<el-button type="" @click="tochange=false" icon="el-icon-refresh-left">取 消</el-button>
					<el-button type="primary" @click="changeres" icon="el-icon-check">确 定</el-button>
				</span>
			</el-dialog>
		</div>
	</div>
</template>
<script>
import ExportJsonExcel from "js-export-excel";
import '@riophae/vue-treeselect/dist/vue-treeselect.css'
import { getStatisticsInfoByDate } from "../../../services/remote";//查询细节
import { deleteSumData } from "../../../services/remote";
import { updateMesData } from "../../../services/remotenew";
export default {
   name:'',
   data(){
       return{
           listData:[],
           resData:'',
           ifchange:false,
           tochange:false,
           resData:[]
       }
   },
   methods:{
     changeres(){
       updateMesData(this.resData).then(res=>{
         this.$message.success('修改成功');
         this.tochange=false;
          getStatisticsInfoByDate(data).then(res=>{
           this.listData=res.data; })
       })
     },
       handelcelchange(data){
         this.tochange=true;
         this.resData=data
       },
       handleCancel(){//返回
         this.$router.go(-1)
       },
       output(data){
          var data={date:''};
       data.date=this.$route.params.time
       getStatisticsInfoByDate(data).then(res=>{
           this.listData=res.data; 
           this.downloadData=[];
          // this.resData=[{"monitorPlanDetailID":'125105',"deviceNo":'adasd',"myNo":'adasddddd',"projectName":'mock测试',"charger":'adasda',"tel":'asdasdasd'}]
          
						//将树形数据转换为table型数据
						this.parseTreeToTable(this.listData)
						var option = {};
						//下载文件名
						option.fileName =this.$route.params.time+'统计表';
						//设置数据来源和数据格式
						option.datas = [{
							sheetData: this.downloadData,
							sheetHeader: ["日期","基层单位", "开工项目数量", "日报数量", "配备记录仪数量", "出库数量", "开机使用数量","备用数量","覆盖率","利用率","使用率"]
            }];
						// //导出
						var toExcel = new ExportJsonExcel(option);
						toExcel.saveExcel();
           })
         
       },
       parseTreeToTable(node) {//转换格式
        //初始化下载数据项对象
        this.downloadData=[]
				//遍历当前节点，装填数据
				for (var i = 0; i < node.length; i++) {
					//如果当前节点存在，装填数据
					if (node[i]) {
           this.downloadDataItem = {}
						this.downloadDataItem.sumDate = node[i].sumDate
						this.downloadDataItem.companyName = node[i].companyName
						this.downloadDataItem.workNum = node[i].workNum
						this.downloadDataItem.dayReportNum = node[i].dayReportNum
						this.downloadDataItem.recordDeviceNum = node[i].recordDeviceNum
            this.downloadDataItem.outStockNum = node[i].outStockNum
            this.downloadDataItem.powerOnNum = node[i].powerOnNum
            this.downloadDataItem.backNum = node[i].backNum
            this.downloadDataItem.coverageRate = node[i].coverageRate
            this.downloadDataItem.availableRate = node[i].availableRate
            this.downloadDataItem.useRate = node[i].useRate
            this.downloadData[i]=this.downloadDataItem
            
            console.log(this.downloadDataItem)
            console.log(this.downloadData[i])
            console.log(this.downloadData)
					}
				}
			},
       handelcelDelete(data){//删除
         this.resData=data;
         var data1={sumDataId:data.mesCheckSumDataID};
         deleteSumData(data1).then(res=>{
           this.$message.success('删除成功',res)
           var data={date:''};
           data.date=this.$route.params.time
           getStatisticsInfoByDate(data).then(res=>{
           this.listData=res.data; })
         }).catch(err=>{
           this.$message.error('删除失败',err)
         })
       },
   },
   mounted(){
       console.log('细节页面报错',JSON.stringify(this.$route.params.time))
    //    this.id={"planId":this.$route.params.monitorPlanID}
       var data={date:''};
       data.date=this.$route.params.time
       getStatisticsInfoByDate(data).then(res=>{
           this.listData=res.data; })
   }
}
</script>
<style lang='scss'>

</style>
