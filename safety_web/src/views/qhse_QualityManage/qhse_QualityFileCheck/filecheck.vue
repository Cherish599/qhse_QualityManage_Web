<template>
  <div>
    <div class="page-title" style="width:100%">
        <el-breadcrumb separator="/">
            <el-breadcrumb-item><a @click="$router.go(-1)">QHSE文件审核</a></el-breadcrumb-item>
            <el-breadcrumb-item>详情查看</el-breadcrumb-item>
        </el-breadcrumb>
    </div>
    <div style="margin:15px 0px">
      <span style="margin-right:15px">请选择文件审核方式:</span>
          <el-radio v-model="checkType" label="树形审核">树形审核</el-radio> 
          <el-radio v-model="checkType" label="列表审核">列表审核</el-radio>     
    </div>
    <el-row style="padding:8px; border-top: 2px dashed #dddddd;text-align:center"></el-row>
    <div class="page-content">
      <el-row>
        <el-form label-width="80px"  :inline="true" :model="filterQuery">
          <el-form-item label="公司：">
            <el-input v-model="filterQuery.companyName" readonly></el-input>
          </el-form-item>
          <el-form-item label="年份：">
            <el-input v-model="filterQuery.year" readonly></el-input>
          </el-form-item>
          <el-form-item label="全要素：">
            <el-input v-model="allTotal" readonly  style="width:80px"></el-input>
          </el-form-item>
           <el-form-item label="已审核：">
            <el-input v-model="hasTotal" class="hasAudit" readonly  style="width:80px"></el-input>
          </el-form-item>
           <el-form-item label="未审核：">
            <el-input v-model="noTotal" class="noAudit" readonly  style="width:80px"></el-input>
          </el-form-item>
          &nbsp;&nbsp;&nbsp;
          <el-form-item>
            <el-button type="primary" v-show="finishAudit">审核完成</el-button>
          </el-form-item>
          <el-form-item>
            <el-button type="danger" @click="$router.go(-1)">返回前一页</el-button>
          </el-form-item>
        </el-form>
      </el-row>
      <el-row style="padding:10px; border-top: 2px dashed #dddddd;text-align:center" >
        <div v-show="checkType === '树形审核'">
        <el-table
          :data="treeList"
          style="width: 100%"
          ref="treeTable"
          row-key="id"
          :indent="30"
          max-height="510"
          highlight-current-row
          border
          @cell-click="handleCellClick"
          v-loading="loading"
          :tree-props="{children: 'childNode', hasChildren: 'hasChildren'}">
          <el-table-column prop="name" label="内容"></el-table-column>
          <el-table-column label="操作" width="200" align="center">
            <template slot-scope="scope">
              <el-button
                type="primary"
                size="mini"
                @click="goUpdateFile(scope.row)"
                v-if="scope.row.childNode.length === 0 && (!scope.row.fileCheckStatus || scope.row.fileCheckStatus === '未审核') && scope.row.status === '备案待查'"  
                icon="el-icon-edit"
              >开始审核</el-button>
              <el-button
              type="success"
              size="mini"
              v-if="scope.row.childNode.length === 0 && scope.row.fileCheckStatus === '通过' && scope.row.status === '备案待查'"
              @click="detaileFile(scope.row)" icon="el-icon-search"
              >
              查看详情  
              </el-button>
               <el-button
              type="danger"
              size="mini"
              v-if="scope.row.childNode.length === 0 && scope.row.fileCheckStatus === '不通过' && scope.row.status === '备案待查'"
              @click="detaileFile(scope.row)" icon="el-icon-search"
              >
              查看详情  
              </el-button>
               <span v-show="scope.row.childNode.length === 0  && scope.row.status !== '备案待查'">请完成要素证据批准</span>
            </template>
          </el-table-column>
        </el-table>
        </div>
        <div v-if="checkType === '列表审核'">
         <el-table
          :data="fileList.filter(data => !search || data.name.toLowerCase().includes(search.toLowerCase()))"
          ref="listTable"
          style="width: 100%"
          v-loading="loading"
          :indent="30"
          max-height="510"
          highlight-current-row
          border
          >
          <el-table-column prop="name" label="内容"></el-table-column>
          <el-table-column  width="200" align="center">
            <template slot="header" slot-scope="scope">
              <el-input
                :ref="scope.$index"
                v-model="search"
                size="mini"
                placeholder="输入关键字搜索"/>
            </template>
            <template slot-scope="scope">
              <el-button
                type="primary"
                size="mini"
                @click="goUpdateFile(scope.row)"
                v-if="scope.row.childNode.length === 0 && (!scope.row.fileCheckStatus || scope.row.fileCheckStatus === '未审核') && scope.row.status === '备案待查' "  
                icon="el-icon-edit"
              >开始审核</el-button>
              <el-button
              type="success"
              size="mini"
              v-if="scope.row.childNode.length === 0 && scope.row.fileCheckStatus === '通过' && scope.row.status === '备案待查'"
              @click="detaileFile(scope.row)" icon="el-icon-search"
              >
              查看详情  
              </el-button>
               <el-button
              type="danger"
              size="mini"
              v-if="scope.row.childNode.length === 0 && scope.row.fileCheckStatus === '不通过' && scope.row.status === '备案待查'"
              @click="detaileFile(scope.row)" icon="el-icon-search"
              >
              查看详情  
              </el-button>
               <span v-show="scope.row.childNode.length === 0  && scope.row.status !== '备案待查' && scope.row.totalCount !==0">请完成要素证据批准</span>
            </template>
          </el-table-column>
        </el-table>
        </div>
      </el-row>

      <!--文件审核的具体页面 -->
      <el-dialog title="文件审核" :visible.sync="dialogVisible" center width="55%" :close-on-click-modal='false'>
        <div v-loading='addLoading'>
        <el-form label-width="140px" :model="detailData" style="width:100%;" >
          <el-row>
            <el-col :span="12">
              <el-form-item label="打分项：" style="margin-bottom:1px">{{detailData.name}}</el-form-item>            
              <el-form-item label="审核方式：" style="margin-bottom:1px">{{detailData.auditMode}}</el-form-item>
              <el-form-item label="初始分数：" style="margin-bottom:1px">{{detailData.initialScore}}</el-form-item>
              <el-form-item label="计算公式：" style="margin-bottom:1px">{{detailData.formula}}</el-form-item>
              <el-form-item label="证据图片：" 
              style="margin-bottom:10px"
              >
              <span v-show="attachs.length === 0">无图片附件记录！</span>
             <div v-for="(item,index) in attachs" :key="index">
                                    <el-card :body-style="{ padding: '10px' }"
                                             style="width:100px;height:60px;text-align:center;float:left;margin:05px">
                                        <span v-if="!item">无图片文件记录！</span>
                                        <el-popover placement="right" title trigger="click" v-else>
                                            <div style="box-shadow: 0 0 2px 4px rgba(0,0,0,0.3);"
                                                 class="picPosition">
                                                <img :src="item" class="picSize"/>
                                            </div>
                                            <img slot="reference" :src="item" :alt="detailData.pictureFile"
                                                 style="width: 100%; height: 100%;"/>
                                        </el-popover>
                                    </el-card>
                                </div>
              </el-form-item>
              <el-form-item label="证据文件：" 
              style="margin-bottom:10px"
              >
               <span v-show="files.length === 0">无文件附件记录！</span>
                <div v-for="(item,index) in files" :key="index">
                  <!-- <a   style="max-width:600px;height:auto" >{{item + download[index]}}</a> -->
                    <span style="text-decoration: underline;color:blue;margin-right:8px">{{download[index]}}</span>
                    <el-button @click="previewFile(item)" type="text">预览</el-button>
                    <el-button  @click="downloadFile(item,download[index])" type="text">下载</el-button>
                </div>
              </el-form-item>
            </el-col>
            <el-col :span="12">
              <h3 style="margin-left:50px">
                    填写审核内容                
              </h3>
              <el-form :v-model="fileRecord">
              <el-form-item label='分数:' labelWidth='100px' style="width:70%">
                <el-input type="text" v-model.number="fileRecord.codeScore" :placeholder="inputPlace"></el-input>
              </el-form-item>
              <el-form-item label='通过状态:' labelWidth='100px' >
                  <el-switch
                          style="margin-right:10px"
                          v-model="filePass"
                          active-color="#13ce66"
                          inactive-color="#ff4949"
                          active-text="通过"
                          inactive-text="不通过">
                  </el-switch>
              </el-form-item>
               <el-form-item label='操作:' labelWidth='100px'  style="margin-bottom:30px;" v-if="fileRecord.pass === '不通过'">
               <el-radio v-model="reason" label="问题">录入问题</el-radio> 
               <el-radio v-model="reason" label="违章">录入违章</el-radio>            
               <el-radio v-model="reason" label="隐患">录入隐患</el-radio>
            </el-form-item>
              <el-form-item label='问题选择:' labelWidth='100px' v-show='reason === "问题" && fileRecord.pass === "不通过"'>
                <el-checkbox-group v-model="selectProblem" >
                  <el-checkbox v-for="(item,index) in problem" :key="index" :label="item">                   
                  </el-checkbox>
                  </el-checkbox-group>
                  <el-checkbox  v-model="elsePro">其他</el-checkbox>
                  <el-input
                    type="textarea"
                    v-if="elsePro"
                    :rows="2"
                    placeholder="请输入具体问题"
                    v-model="proTextarea">
                  </el-input>
              </el-form-item>
              </el-form>
              
            </el-col>
          </el-row>         
        </el-form>     
        
        <div style='margin-left:60%'>
            <el-button type="primary" @click="updataFileAudit">确定</el-button>
            <el-button @click="resetQuestion">取 消</el-button>
          </div>
          </div>
      </el-dialog>
      <!-- 查看文件审核详情 -->
      <el-dialog title="文件审核详情查看"  :visible.sync="detaildialogVisible" center  :close-on-click-modal='false' >
        <div v-loading='eviLoaind'>
        <el-form   label-width="140px" :model="detailData" style="width:100%;" >
          <el-row >
            <el-col :span="12">
              <el-form-item label="打分项：" style="margin-bottom:1px">{{detailData.name}}</el-form-item>            
              <el-form-item label="审核方式：" style="margin-bottom:1px">{{detailData.auditMode}}</el-form-item>
              <el-form-item label="初始分数：" style="margin-bottom:1px">{{detailData.initialScore}}</el-form-item>
              <el-form-item label="实际得分：" style="margin-bottom:1px"><el-button size='mini' type='primary'>{{detailData.codeScore}}</el-button></el-form-item>
              <el-form-item label="计算公式：" style="margin-bottom:1px">{{detailData.formula}}</el-form-item>
              <el-form-item label="审核状态：" style="margin-bottom:1px"><el-button size='mini' v-if="detailData.pass === '通过'" type='success'>{{detailData.pass}}</el-button>
              <el-button size='mini' v-if="detailData.pass === '不通过'" type='danger'>{{detailData.pass}}</el-button>
              </el-form-item> 
              <el-form-item label="审核时间：" v-show="detailData.auditTime"  style="margin-bottom:1px">{{detailData.auditTime}}</el-form-item> 
              
              <el-form-item label="操作：" style="margin-bottom:1px" v-if="detailData.noPassReason">
                <el-button v-if="detailData.noPassReason === '隐患'" @click="goDanger" size='mini' type="warning">补录隐患</el-button>
                <el-button v-if="detailData.noPassReason === '违章'" @click="goRegulation" size='mini' type="warning" >补录违章</el-button>
                <el-button v-if="detailData.noPassReason === '问题'" @click="goProblem" size='mini' type="warning" >补录问题</el-button>
                </el-form-item>      
            </el-col>
            <el-col :span="12" >
              <el-form-item label="不通过原因：" v-show="detailData.noPassReason"
              style="margin-bottom:5px"
              >存在{{detailData.noPassReason}}
               </el-form-item>
              <el-form-item label="证据图片：" 
              style="margin-bottom:10px"
              >
              <span v-show="attachs.length === 0">无图片文件记录！</span>
              <div v-for="(item,index) in attachs" :key="index">
                                    <el-card :body-style="{ padding: '10px' }"
                                             style="width:100px;height:60px;text-align:center;float:left;margin:05px">
                                        <span v-if="!item">无图片文件记录！</span>
                                        <el-popover placement="right" title trigger="click" v-else>
                                            <div style="box-shadow: 0 0 2px 4px rgba(0,0,0,0.3);"
                                                 class="picPosition">
                                                <img :src="item" class="picSize"/>
                                            </div>
                                            <img slot="reference" :src="item" :alt="detailData.pictureFile"
                                                 style="width: 100%; height: 100%;"/>
                                        </el-popover>
                                    </el-card>
                                </div>
              </el-form-item>
              <el-form-item label="证据文件：" 
              style="margin-bottom:10px"
              >
                <span v-show="files.length === 0">无文件附件记录！</span>
                <div v-for="(item,index) in files" :key="index">
                    <span style="text-decoration: underline;color:blue;margin-right:8px">{{download[index]}}</span>
                    <el-button @click="previewFile(item)" type="text">预览</el-button>
                    <el-button  @click="downloadFile(item)" type="text">下载</el-button>
                </div>
              </el-form-item>
            </el-col>
          </el-row>
        </el-form>       
          </div>
           <el-dialog title="问题补录"  :visible.sync="problemdialogVisible" center  append-to-body :close-on-click-modal='false'>
             <div v-loading='editQuestionLoadind'>
             <el-checkbox-group v-model="selectProblem" >
                  <el-checkbox v-for="(item,index) in problem" :key="index" :label="item">                   
                  </el-checkbox>
                  </el-checkbox-group>
                  <el-checkbox  v-model="elsePro">其他</el-checkbox>
                  <el-input
                    type="textarea"
                    v-if="elsePro"
                    :rows="2"
                    placeholder="请输入具体问题"
                    v-model="proTextarea">
                  </el-input>
                  <p style="margin-top;10px"></p>
                  <div slot="footer" class="dialog-footer" style="text-align:right;margin-top;10px">
                 <el-button type="danger"  @click="problemdialogVisible = false" >取消</el-button>
                 <el-button type="primary"  @click="confirmEdit" >确定</el-button>
            </div>
            </div>
           </el-dialog>
      </el-dialog>
             
          
        
    </div>
  </div>
</template>

<script>
import CurrentUser from '../../../store/CurrentUser'
import {noPassReasonFileAudit,addProblemDescription,querryQhseElement,updateCheckstatus,addFileaduitrecord,queryRecordId,getStatus} from "../../../services/qhse_Filecheck"; // 文件审核相关
import {querryQHSEproblemDiscription} from '../../../services/qhse_QualityStandard'
import { show_elementReviewer,downloadElementFile } from"../../../services/qhse_EvidenceCheck"//显示要素证据信息
export default {
  data() {
    return {
      editQuestionLoadind:false,
      noProblem: '暂无',
      search: '',
      checkType: '树形审核',
      inputPlace: '',
      filterQuery: {
        year: "",
        companyCode: null,
        status:''
      },
      // 获取单位年度审核表
      querryTree: {
        companyCode: '',
        year: ''
      },
      // 更新文件审核状态
      updateCheckForm: {
        qhseCompanyYearManagerSysElementTableID: '',
        code: '',
        fileCheckStatus:''
      },
      addformVisible: false,
      // 添加文件审核记录
      fileRecord: {
        fileAuditId: '',
        code: '',
        codeScore: '',
        pass: '通过',
        additor: '',
        auditTime: ''
      },
      // 控制问题
      questionLoading: false,
      // 获取审核状态
      statusForm: {
        fileAuditId: '',
        code: ''
      },
      // 不通过但是不录入问题的原因
      noPassReasonForm:{
         fileAuditId:'',
         code:'',
         noPassReason:''
      },
      // 问题描述
      addQuestionForm: {
        qHSE_FileAudit_ID: '',
        qHSE_FileAuditRecord_ID: '',
        code: '',
        problemDescription: '',
        companyCode: '',
        companyName: '',
        auditTime: '',
        auditor:'',
        itemName:'',
        problemSource:'文件审核'
      },
      nowcode: null,
      // 控制弹出页面
      detaildialogVisible: false,
      noinnerVisible: false,
      innerVisible: false,
      status:'',
      disabled1:false,
      dialogFormVisible: false,
      loading: true,
      dialogVisible: false,
      // 控制表格数据
      detailData: {},
      treeData: [],
      initData:[],
      reason:'问题',
      editdata: '',
      nodialogVisible: false,
      nopass: {},
      fileList: [],
      treeList: [],
      // 问题列表
      problemForm: {},
      problem: [],
      selectProblem: [],
      elsePro: false,
      proTextarea: '',
      // 控制证据图片的具体录入
      nodeData: '',
      attachs: [],
      files: [],
      eviLoaind: false,
      addLoading: false,
      download:[],
      filePass:true,
      // 管理审核数量
      hasTotal: 0,
      noTotal: 0,
      allTotal: 0,
      finishAudit: false,
      problemdialogVisible: false,
      addMorePro:{
        qHSE_FileAudit_ID: '', //
        qHSE_FileAuditRecord_ID: '',//
        code: '', //
        problemDescription: '',
        companyCode: '', //
        companyName: '', //
        auditTime: '', //
        auditor:'', //
        itemName:'',//
        problemSource:'文件审核'
      },
      // 跳转到隐患违章页面所需要的数据
      goHidden:{code:'',qHSE_FileAudit_ID:'',qHSE_FileAuditRecord_ID:''}
    };
  },
  watch: {
    filePass(val){
        if(val){
           this.fileRecord.pass = '通过'
        }else{
           this.fileRecord.pass = '不通过'
        }
    }
  },
  methods: {
    previewFile(data){
      console.log(data)
      window.open('http://39.98.173.131:8012/onlinePreview?url='+data)
    },
    downloadFile(data,filename){
                const elink = document.createElement("a"); // 创建的标签
                elink.download =  filename;
                elink.style.display = "none";
                elink.href = data; // 创建url
                document.body.appendChild(elink); // 把 创建的标签追加到body里
                elink.click();
                URL.revokeObjectURL(elink.href); // 释放URL 对象
                document.body.removeChild(elink); // 移除标签
    },
    confirmEdit(){
       let _this = this
       let str = ''
       _this.addMorePro.problemDescription = _this.proTextarea
       _this.selectProblem.forEach(item => {
          str = str + item + ' '
       })
       _this.addMorePro.problemDescription = `${_this.proTextarea}-${str}`
       if(_this.addMorePro.problemDescription === '-'){
         _this.$message.warning('请填写具体问题！')
         return
       }
       _this.editQuestionLoadind = true;
       // 先获取到id之后再进行添加问题
       console.log(_this.addMorePro)
       addProblemDescription(_this.addMorePro)
       .then(() => {     
         _this.editQuestionLoadind = false
         _this.problemdialogVisible = false
         this.$message.success("添加成功")
       }).catch(err => {
        this.$message.error(err)
        _this.editQuestionLoadind = false
      })
    },
    goProblem(){
      let _this = this
        _this.problemdialogVisible = true
        _this.problem = []
        _this.proTextarea = ''
         querryQHSEproblemDiscription({code:_this.addMorePro.code}).then(res => {
        if(res.data.length !== 0){
        res.data.forEach(item => {
          if(!item.description.includes('其他')){
          _this.problem.push(item.description)
          }
        })
        }
        this.problemdialogVisible = true
      })     
    },
    // 获取当前时间
    getTime() {
      let nowDate = new Date()
      let month  = Number(nowDate.getMonth() + 1)
      if(month>=10) {
        month = String(month)
      } else {
        month = String('0' + month)
      }
      this.fileRecord.auditTime = `${nowDate.getFullYear()}-${month}-${nowDate.getDate()}`
      this.addQuestionForm.auditTime = this.fileRecord.auditTime
      this.addMorePro.auditTime = this.fileRecord.auditTime
    },
    // 筛选树形图叶子结点信息
    async deepTree (treedata) {
            let _this = this
            
            treedata.forEach(item => {
              // 叶子节点全部进入
                if (item.childNode.length === 0) {
                     _this.allTotal++
                     
                   if (item.status === '备案待查' && (item.fileCheckStatus === '通过' || item.fileCheckStatus === '不通过')){
                      _this.fileList.push(item)
                      _this.hasTotal++
                      return
                   } else if (item.status === '备案待查' && !item.fileCheckStatus ) {
                      _this.fileList.push(item)                     
                   } else{
                     _this.fileList.push(item)     
                   }                   
                } else {
                    _this.deepTree(item.childNode)
                }
            })
            return _this.fileList
      },
      // 填充基础表单
    loadFilterParams() {
      let _this = this
      _this.$route.params.data && localStorage.setItem('data',JSON.stringify(_this.$route.params.data));
      const initData = JSON.parse(localStorage.getItem('data'));
      _this.goHidden.qHSE_FileAudit_ID = initData.fileAuditId
      const user = CurrentUser.get()
      initData.additor = user.employeeName
      _this.filterQuery.year = initData.year
      _this.filterQuery.companyCode = initData.companyCode
      _this.filterQuery.companyName = initData.companyName
      _this.querryTree.year = initData.year
      // 隐患违章表单
      _this.nopass.fileAuditId = initData.fileAuditId
      _this.nopass.companyCode = initData.companyCode
      // 查询单位年度审核表表表单
      _this.querryTree.companyCode = initData.companyCode
      // 新增审核记录表单
      _this.addQuestionForm.qHSE_FileAudit_ID = initData.fileAuditId
      _this.addMorePro.qHSE_FileAudit_ID = initData.fileAuditId
      _this.fileRecord.fileAuditId = initData.fileAuditId
      _this.fileRecord.additor = initData.additor
      // 获取审核状态表单
      _this.statusForm.fileAuditId = initData.fileAuditId
      // 不添加原因表单
       _this.noPassReasonForm.fileAuditId = initData.fileAuditId
      
      // 添加问题表单
      _this.addQuestionForm.companyCode = initData.companyCode
      _this.addQuestionForm.companyName = initData.companyName
      _this.addQuestionForm.auditor = initData.additor
      _this.addMorePro.companyCode = initData.companyCode
      _this.addMorePro.companyName = initData.companyName
      _this.addMorePro.auditor = initData.additor
      
    },
    handleCellClick(row, cell, column) {
      if (row.code.length < 10) {
        let els = column.getElementsByClassName("el-icon-arrow-right");
        if (els.length != 0) {
          els[0].click();
        }
      }
    },
    editFileRecord(data) {
       console.log(data)
    },
    cancelInput() {
      this.$confirm('确认不录入隐患违章吗？','提示',{
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      })
      .then(()=>{
        this.noinnerVisible = false
       })
       
    },
    getEvidence(data) {
        let _this = this 
        _this.files = []
        _this.download = []
        _this.attachs = []
        // 获取证据
        show_elementReviewer({qHSE_CompanyYearManagerSysElement_ID:data.id}).then(res => {
        this.nodeData=res.data;
        console.log(res.data)
        let attach = this.nodeData.attach;//获取地址字符串
         if(attach!=null){
          let arr=attach.split(";");
          for(let i=0,j=0,k=0;i<arr.length-1;i++)
                {
                  //j代表图片数量，k代表文件数量
                  let startIndex = arr[i].lastIndexOf(".");
                  let houzhui=arr[i].substring(startIndex+1, arr[i].length).toLowerCase();//获取到链接后缀
                  if(houzhui=='jpg'||houzhui=='png'){
                  this.attachs[j]=arr[i];            
                  j++;
                  }
                  else{
                    let len = arr[i].length
                    if(arr[i][len-1]!=="无"){
                    this.files[k]=arr[i];
                    downloadElementFile(this.files[k].substring(49,this.files[k].length))
                    .then(res=>{
                      if(res.data)
                      this.download.push(res.data)
                    })
                    .catch(err=>{
                      console.log('报错',err)
                    })
                    k++;
                    }
                  }
                }
        }
      _this.detailData.status = data.status 
      _this.detailData.auditMode = data.auditMode
       _this.detailData.code = data.code
      _this.detailData.initialScore = data.initialScore
      _this.inputPlace = `请输入0-${data.initialScore}之间的分数`
      _this.detailData.formula = data.formula
      // 添加问题描述来源
      
      _this.detailData.name = data.name
      // 状态数据
      /* _this.detailData.codeScore = data.codeScore
      _this.detailData.pass = data.pass */
      _this.detailData.problemDescription = data.problemDescription
      // 获取信息表单    
      _this.statusForm.code = data.code
      return  getStatus(_this.statusForm)
      }).then(res => {
        _this.detailData.codeScore = ''
        _this.detailData.pass = ''
        if(res.data.length > 0){
          console.log(res.data)
        _this.detailData.codeScore = res.data[0].codeScore
        _this.detailData.auditTime = res.data[0].auditTime
        _this.detailData.pass = res.data[0].pass
        _this.addMorePro.qHSE_FileAuditRecord_ID = res.data[0].qHSE_FileAudit_RecordID
        _this.detailData.noPassReason = res.data[0].noPassReason
        _this.detaildialogVisible = true
        }
         _this.eviLoaind = false
         _this.addLoading = false
      })
      .catch(err => {
        _this.$message.error(err)
         _this.dialogVisible = false
          _this.detaildialogVisible = false
      })
    },
    // 获取已经审核完成的记录的详细信息
    detaileFile(data) {
      let _this = this
       _this.nodeData = ''
      _this.attachs = []
      _this.files = []
      _this.problem = []
      _this.selectProblem = []
      // 获取证据图片 
      _this.eviLoaind = true
      _this.addMorePro.code = data.code
      _this.addMorePro.itemName = data.name
      _this.detaildialogVisible = true
      _this.getEvidence(data)  
      //  获取分数与状态填充
      
    },
    // 重置证据
    resetEvidence () {
       this.detaildialogVisible = false
    },
    goRegulation() {
      let localData = {}
      let _this  = this
      localData.code = _this.detailData.code
      localData.qHSE_FileAudit_ID = _this.fileRecord.fileAuditId
      _this.eviLoaind = true
      queryRecordId({fileAuditId:_this.fileRecord.fileAuditId,code:_this.detailData.code}).then(res => {  
        localData.qHSE_FileAuditRecord_ID = res.data[0].qHSE_FileAudit_RecordID
        localStorage.setItem('sourcedata',JSON.stringify(localData))
         this.$router.push({
            path: '/hidden_danger/illegal_entry',
            params: {
            data: this.editdata
            }
            })
      }).catch(err => {
        this.$message.error(err)
      })
      
    },
    goDanger() {
     let localData = {}
      let _this  = this
      localData.code = _this.detailData.code
      localData.qHSE_FileAudit_ID = _this.fileRecord.fileAuditId
      _this.eviLoaind = true
      queryRecordId({fileAuditId:_this.fileRecord.fileAuditId,code:_this.detailData.code}).then(res => {  
        localData.qHSE_FileAuditRecord_ID = res.data[0].qHSE_FileAudit_RecordID
        localStorage.setItem('sourcedata',JSON.stringify(localData))
          this.$router.push({
            path: '/hidden_danger/input',
            params: {
            data: this.editdata
            }
            })
      }).catch(err => {
        this.$message.error(err)
      })
     
    },
    // 填充文件审核页面
    goUpdateFile(data){
      let _this = this
      _this.dialogVisible = true
      _this.editdata = data
      _this.addLoading  = true
      _this.fileRecord.codeScore = ''
      _this.addQuestionForm.itemName = data.name
      _this.nodeData = ''
      _this.attachs = []
      _this.files = []
      _this.problem = []
      _this.selectProblem = []
      _this.getEvidence(data)
      // 填充文件审核详情页面
      _this.fileRecord.code = data.code
      _this.updateCheckForm.code = data.code
      _this.nopass.code = data.code
      querryQHSEproblemDiscription({code:data.code}).then(res => {
        if(res.data.length !== 0){
        res.data.forEach(item => {
          if(!item.description.includes('其他')){
          _this.problem.push(item.description)
          }
        })
        }
      })     
    },
   // 再次验证数据
    handleGetInitialData() {
      let _this = this
      _this.loading = true;
      _this.hasTotal = 0
      _this.noTotal = 0
      _this.allTotal = 0
      // 获取单位年度审核表
     
      querryQhseElement(_this.querryTree).then(res => {
        _this.treeData = res.data;
        _this.treeList = res.data;
         console.log(res.data)
        _this.updateCheckForm.qhseCompanyYearManagerSysElementTableID = res.data[0].tableID
        if(res.data.length === 0){
          _this.$message.warning('请检查要素证据审批是否完成！')
          _this.$router.go(-1)
        }
         _this.fileList = []
        return _this.deepTree(_this.treeData)
      })  
      .then(() => {
        _this.noTotal = _this.allTotal - _this.hasTotal
        if(_this.hasTotal  === _this.allTotal){
           _this.finishAudit = true
        }
        _this.loading = false
        })
      .catch(err => {
          _this.$message.error(err.message);
          _this.$message.warning('请检查要素证据审批是否完成！')
          _this.$router.go(-1)
        });   
    },
    // 添加问题
    addQuestion () { 
       let _this = this
       let str = ''
       _this.addQuestionForm.code = _this.fileRecord.code
       _this.addQuestionForm.problemDescription = _this.proTextarea
       _this.selectProblem.forEach(item => {
          str = str + item + ' '
       })
       _this.addQuestionForm.problemDescription = `${_this.proTextarea} ${str}`
       // 先获取到id之后再进行添加问题
       _this.noPassReasonForm.code = _this.addQuestionForm.code
       localStorage.setItem('sourcedata',JSON.stringify(_this.goHidden))
        addProblemDescription(_this.addQuestionForm)
       .then(() => {     
         _this.questionLoading = false
       }).catch(err => {
        this.$message.error(err)
        _this.noinnerVisible = false
      })
    },
    // 不录入时候的原因
    updataFileAudit() {
      // 上传文件审核记录
      let _this = this
      if(_this.fileRecord.codeScore === '' || _this.fileRecord.pass === '' ) {
        _this.$message.error('请填写正确的审核内容')
        return
        } else if (_this.proTextarea === '' &&  _this.selectProblem.length === 0 && _this.fileRecord.pass === '不通过' && _this.reason === 
        '问题') {
          _this.$message.error('请填写具体问题内容')
          return
        } else if (_this.fileRecord.codeScore > Number(_this.detailData.initialScore) || _this.fileRecord.codeScore < 0) {
          _this.$message.error('请填写合法的分数')
          return
        }
      _this.addLoading = true
      _this.status = _this.fileRecord.pass
      _this.updateCheckForm.fileCheckStatus = _this.fileRecord.pass
      // 再添加文件审核记录
      addFileaduitrecord(_this.fileRecord).then((res) => {
         _this.addQuestionForm.qHSE_FileAuditRecord_ID  = res.key
         _this.goHidden.qHSE_FileAuditRecord_ID = res.key
        _this.goHidden.code = _this.fileRecord.code
              // 先更新状态
        _this.noPassReasonForm.code = _this.fileRecord.code
        _this.noPassReasonForm.noPassReason = _this.reason
        return noPassReasonFileAudit(this.noPassReasonForm)
        })
       .then(() =>{
         return updateCheckstatus(_this.updateCheckForm)
       })
      .then(() => {
        this.noPassReasonForm.noPassReason = ''
        // 这里应该返回文件审核记录的recordID
        if(_this.status === '不通过') {
          if(_this.reason === '问题'){
            _this.addQuestion()   
          } else{
            _this.noScore()
          }
           
        }  
        _this.$message.success('添加成功！')
        _this.addLoading = false
        _this.dialogVisible = false
        // 重置添加状态
        _this.fileRecord.code = ''
        _this.fileRecord.codeScore = ''
        _this.problemList = []
        _this.proTextarea = ''
        _this.elsePro = false
        _this.handleGetInitialData()
      })
      .catch(err => {
        _this.addLoading = false
        _this.$message.error(err)
      })

    },
    resetQuestion () {
        this.dialogVisible = false
    },
    checkNopass () {
      this.noScore()
    },
    noScore() {
      // 不通过时开始录入隐患违章
      let rowdata = this.editdata
      let _this = this
      // 新增不成功
      
      if (_this.reason === '问题') _this.noinnerVisible = false
      else if (_this.reason === '隐患'){
           _this.$router.push({
            path: '/hidden_danger/input',
            params: {
            data: rowdata
            }
          })
      } else {
            _this.$router.push({
            path: '/hidden_danger/illegal_entry',
            params: {
            data: rowdata
            }
          })
      }
      _this.dialogVisible = false
    }
  },
  mounted() {
    this.loadFilterParams();
    this.handleGetInitialData();
    this.getTime()
  }
};
</script>

<style lang="scss">
.hasAudit{
  .el-input__inner{
    color: green;
  }
}
.noAudit{
  .el-input__inner{
    color: red;
  }
}
    .picSize {
        max-width: 1400px;
        max-height: 750px;
        vertical-align: bottom;
    }

    .picPosition {
        position: fixed;
        left: 50%;
        top: 50%;
        transform: translate(-50%, -50%);
    }
</style>