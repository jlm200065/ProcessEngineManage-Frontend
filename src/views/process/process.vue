<template>
  <div>
    <!--面包屑导航区域-->
    <div class="board">
      <el-breadcrumb separator-class="el-icon-arrow-right">
        <el-breadcrumb-item :to="{ path: '/welcome' }">首页</el-breadcrumb-item>
        <el-breadcrumb-item>Process管理</el-breadcrumb-item>
        <el-breadcrumb-item>Process信息管理</el-breadcrumb-item>
      </el-breadcrumb>
    </div>

    <!--操作按钮区域-->
    <el-card class="box-card">
      <el-row :gutter="10">
        <el-col :span="3">
          <el-input placeholder="请输入名称" v-model="inputName" clearable />
        </el-col>
        <el-col :span="3">
          <el-button icon="el-icon-search" @click="getProcessList">搜索</el-button>
        </el-col>

        <el-col :span="3">
          <el-button type="primary" @click="combineSelectedProcesses">合并</el-button>
        </el-col>

        <el-col :span="3">
          <el-button type="primary" @click="showAddDialog">添加Process</el-button>
        </el-col>
        <el-col :span="3">
          <el-button type="danger" @click="multipleDelete">批量删除</el-button>
        </el-col>
        <el-col :span="3">
          <el-button type="success" @click="discoverSupplyChain">发现供应链</el-button>
        </el-col>
      </el-row>

      <el-table :data="processList" style="width: 100%" border stripe @selection-change="handleSelectionChange">
        <el-table-column type="selection" width="40"></el-table-column>
        <el-table-column prop="id" label="ID" width="300px"></el-table-column>
        <el-table-column prop="name" label="名称" width="200px"></el-table-column>
        <el-table-column prop="origin" label="上传者" width="150px"></el-table-column>
        <el-table-column prop="engineCategory" label="引擎类型" width="100px"></el-table-column>

        <el-table-column label="操作" width="420">
          <template slot-scope="scope">
            <el-button type="primary" icon="el-icon-edit" size="mini" @click="showEditDialog(scope.row.id)">编辑</el-button>
            <el-button type="danger" icon="el-icon-delete" size="mini" @click="deleteProcessById(scope.row.id)">删除</el-button>
            <el-button type="success" icon="el-icon-view" size="mini" @click="showViewDialog(scope.row)">查看</el-button>
            <el-button type="warning" icon="el-icon-upload" size="mini" @click="deployProcess(scope.row)">部署</el-button>
          </template>
        </el-table-column>
      </el-table>

      <!--分页区域-->
      <el-pagination
        @size-change="handleSizeChange"
        @current-change="handleCurrentChange"
        :current-page="queryInfo.pageNum"
        :page-sizes="[5, 10, 20]"
        :page-size="queryInfo.pageSize"
        layout="total, sizes, prev, pager, next, jumper"
        :total="total">
      </el-pagination>
    </el-card>

    <!--添加Process对话框-->
    <el-dialog title="添加Process" :visible.sync="addDialogVisible" width="60%" @close="addDialogClosed">
      <el-form :model="addForm" :rules="addFormRules" ref="addFormRef" label-width="150px">
        <el-form-item label="名称" prop="name">
          <el-input v-model="addForm.name"></el-input>
        </el-form-item>
        <el-form-item label="上传者" prop="origin">
          <el-input v-model="addForm.origin"></el-input>
        </el-form-item>
        <el-form-item label="引擎类型" prop="engineCategory">
          <el-select v-model="addForm.engineCategory" placeholder="请选择引擎类型">
            <el-option v-for="item in engineTypes" :key="item.value" :label="item.label" :value="item.value">
              <img :src="item.image" :alt="item.label" class="option-image" /> {{ item.label }}
            </el-option>
          </el-select>
        </el-form-item>
        <el-form-item label="BPMN文件" prop="bpmn">
          <el-upload
            class="upload-demo"
            action="#"
            :before-upload="beforeBpmnUpload"
            :on-change="handleChange"
            :on-remove="handleRemove"
            :file-list="fileList">
            <el-button size="small" type="primary">点击上传</el-button>
            <div slot="tip" class="el-upload__tip">只支持.bpmn或.xml扩展名的文件</div>
          </el-upload>
          <el-input v-model="addForm.bpmn" placeholder="BPMN文件内容" type="textarea" rows="10"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="addDialogVisible = false">取消</el-button>
        <el-button type="primary" @click="addProcess">确定</el-button>
      </span>
    </el-dialog>

    <!--修改Process对话框-->
    <el-dialog title="修改Process信息" :visible.sync="editDialogVisible" width="60%" @close="editDialogClosed">
      <el-form :model="editForm" :rules="editFormRules" ref="editFormRef" label-width="150px">
        <el-form-item label="名称" prop="name">
          <el-input v-model="editForm.name"></el-input>
        </el-form-item>
        <el-form-item label="上传者" prop="origin">
          <el-input v-model="editForm.origin"></el-input>
        </el-form-item>
        <el-form-item label="引擎类型" prop="engineCategory">
          <el-select v-model="editForm.engineCategory" placeholder="请选择引擎类型">
            <el-option v-for="item in engineTypes" :key="item.value" :label="item.label" :value="item.value">
              <img :src="item.image" :alt="item.label" class="option-image" /> {{ item.label }}
            </el-option>
          </el-select>
        </el-form-item>
        <el-form-item label="BPMN文件" prop="bpmn">
          <el-upload
            class="upload-demo"
            action="#"
            :before-upload="beforeBpmnUpload"
            :on-change="handleChange"
            :on-remove="handleRemove"
            :file-list="fileList">
            <el-button size="small" type="primary">点击上传</el-button>
            <div slot="tip" class="el-upload__tip">只支持.bpmn或.xml扩展名的文件</div>
          </el-upload>
          <el-input v-model="editForm.bpmn" placeholder="BPMN文件内容" type="textarea" rows="10"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="editDialogVisible = false">取消</el-button>
        <el-button type="primary" @click="editProcessInfo">确定</el-button>
      </span>
    </el-dialog>

    <el-dialog title="查看BPMN图" :visible.sync="viewDialogVisible" width="70%">
      <div id="bpmnCanvas" class="bpmn-container"></div>
      <span slot="footer" class="dialog-footer">
    <el-button @click="downloadBpmn">下载</el-button>
    <el-button @click="viewDialogVisible = false">关闭</el-button>
  </span>
    </el-dialog>

    <!--发现供应链对话框-->
    <el-dialog title="发现供应链" :visible.sync="discoverDialogVisible" width="60%">
      <el-row :gutter="20" v-for="(collaboration, index) in collaborations" :key="index">
        <el-col :span="24">
          <el-button style="width: 700px" type="success" @click="selectCollaboration(collaboration)" block>
            协作 {{ index + 1 }}
          </el-button>
        </el-col>
      </el-row>
      <span slot="footer" class="dialog-footer">
        <el-button @click="discoverDialogVisible = false">关闭</el-button>
      </span>
    </el-dialog>



    <!-- 部署结果展示 -->
    <el-dialog title="部署结果" :visible.sync="deployResultVisible" width="70%">
      <el-row :gutter="20">
        <el-col :span="24" v-for="(item, index) in deployResult" :key="index">
          <el-card @click.native="handleCardClick(item)">
            <h3>{{ item.name }}</h3>
            <p><strong>引擎类型: </strong>{{ item.engineType }}</p>
            <p><strong>URL: </strong>{{ item.url }}</p>
            <p><strong>消息队列: </strong>{{ item.pubSubName }} ({{ item.pubSubUrl }})</p>
            <div>
              <strong>主题: </strong>
              <el-tag
                v-for="(topic, index) in item.topics"
                :key="index"
                type="primary"
                size="small"
                style="margin-right: 5px;">
                {{ topic.topic }}
              </el-tag>
            </div>
          </el-card>
        </el-col>
      </el-row>
      <span slot="footer" class="dialog-footer">
        <el-button @click="deployResultVisible = false">关闭</el-button>
      </span>
    </el-dialog>

  </div>
</template>

<script>
import BpmnViewer from 'bpmn-js/lib/NavigatedViewer';
import BpmnModeler from 'bpmn-js/lib/Modeler'
import xmlStr from '@/assets/xmlStr'
export default {
  data() {
    return {
      queryInfo: {
        name: '',
        pageNum: 1,
        pageSize: 10
      },
      total: 0,
      addDialogVisible: false,
      addForm: {
        name: '',
        origin: '',
        engineCategory: '',
        bpmn: ''
      },
      viewDialogVisible: false,
      addFormRules: {
        name: [{ required: true, message: '请输入名称', trigger: 'blur' }],
        origin: [{ required: true, message: '请输入上传者', trigger: 'blur' }],
        engineCategory: [{ required: true, message: '请输入引擎类型', trigger: 'blur' }],
        bpmn: [{ required: true, message: '请上传BPMN文件', trigger: 'change' }]
      },
      editDialogVisible: false,
      editForm: {},
      editFormRules: {
        name: [{ required: true, message: '请输入名称', trigger: 'blur' }],
        origin: [{ required: true, message: '请输入上传者', trigger: 'blur' }],
        engineCategory: [{ required: true, message: '请输入引擎类型', trigger: 'blur' }],
        bpmn: [{ required: true, message: '请上传BPMN文件', trigger: 'change' }]
      },
      bpmnViewerVisible: false,
      inputName: '',
      processList: [],
      fileList: [],
      multipleSelection: [],
      engineTypes: [
        {value: 'camunda7', label: 'Camunda 7', image: require('@/assets/img/camunda.png')},
        {value: 'activiti7', label: 'Activiti 7', image: require('@/assets/img/activiti.png')},
        {value: 'flowable6', label: 'Flowable 6', image: require('@/assets/img/flowable.png')}
      ],
      viewer: null,
      bpmnModeler: null,
      bpmnXml: '',
      discoverDialogVisible: false,
      collaborations: [],
      deployResultVisible: false,
      deployResult: []
    };
  },
  created() {
    this.getProcessList();
  },
  methods: {
    downloadBpmn() {
      const element = document.createElement('a');
      const file = new Blob([this.bpmnXml], {type: 'text/xml'});
      element.href = URL.createObjectURL(file);
      element.download = "combinedProcess.bpmn";
      document.body.appendChild(element); // Required for this to work in FireFox
      element.click();
      document.body.removeChild(element);
    },

    combineSelectedProcesses() {
      const bpmnStrings = this.multipleSelection.map(item => item.bpmn);
      axios.post('/process/getCombine', bpmnStrings)
        .then(response => {
          this.bpmnXml = response.data;
          this.showViewDialog({ bpmn: this.bpmnXml }); // Reuse the viewing dialog
        })
        .catch(error => {
          console.error('Error combining processes:', error);
          this.$message.error('合并过程中出现错误');
        });
    },
    handleCardClick(item) {
      this.$message.success(`部署成功: ${item.name}`);
    },

    getProcessList() {
      this.queryInfo.name = this.inputName;
      axios.post('/process/all', this.queryInfo).then(resp => {
        this.processList = resp.data.data;
        this.total = resp.data.total;
        this.queryInfo.pageSize = resp.data.pageSize;
        this.queryInfo.pageNum = resp.data.pageNum;
      }).catch(error => {
        console.error(error);
      });
    },
    handleSizeChange(newSize) {
      this.queryInfo.pageSize = newSize;
      this.getProcessList();
    },
    handleCurrentChange(newPage) {
      this.queryInfo.pageNum = newPage;
      this.getProcessList();
    },
    showAddDialog() {
      this.addDialogVisible = true;
    },
    addDialogClosed() {
      this.$refs.addFormRef.resetFields();
    },
    addProcess() {
      this.$refs.addFormRef.validate(valid => {
        if (!valid) return;
        axios.post('/process/add', this.addForm).then(() => {
          this.getProcessList();
          this.addDialogVisible = false;
          this.fileList = [];
        }).catch(error => {
          console.error(error);
        });
      });
    },
    showEditDialog(id) {
      axios.get(`/process/${id}`).then(resp => {
        this.editForm = resp.data.data;
        this.editDialogVisible = true;
      }).catch(error => {
        console.error(error);
      });
    },
    editDialogClosed() {
      this.$refs.editFormRef.resetFields();
    },
    editProcessInfo() {
      this.$refs.editFormRef.validate(valid => {
        if (!valid) return;
        axios.put('/process/update', this.editForm).then(() => {
          this.getProcessList();
          this.editDialogVisible = false;
          this.fileList = [];
        }).catch(error => {
          console.error(error);
        });
      });
    },
    handleSelectionChange(val) {
      this.multipleSelection = val;
    },
    multipleDelete() {
      const ids = this.multipleSelection.map(item => item.id);
      axios.delete('/process/delete', {data: ids}).then(() => {
        this.getProcessList();
      }).catch(error => {
        console.error(error);
      });
    },
    deleteProcessById(id) {
      axios.delete(`/process/delete/${id}`).then(() => {
        this.getProcessList();
      }).catch(error => {
        console.error(error);
      });
    },
    beforeBpmnUpload(file) {
      const isBPMN = file.name.match(/\.(bpmn|xml)$/i);
      if (!isBPMN) {
        this.$message.error('您只能上传BPMN或XML文件!');
        return false;
      }
      // 调用 handleBpmnUpload 方法
      this.handleBpmnUpload(file);
      return false; // 防止自动上传
    },
    handleChange(file, fileList) {
      // 将文件添加到 fileList 中
      this.fileList = fileList;
    },
    handleRemove() {
      this.addForm.bpmn = ''; // 清空BPMN数据
    },
    handleBpmnUpload(file) {
      console.log("file object: ", file); // 确认文件对象
      const reader = new FileReader();
      reader.onload = (e) => {
        this.addForm.bpmn = e.target.result;
        this.editForm.bpmn = e.target.result;
        console.log("e.target.result: ", e.target.result); // 确认文件对象
      };
      reader.readAsText(file); // 直接读取 file
      return false; // 防止自动上传
    },
    viewBpmnDiagram(bpmnContent) {
      console.log('BPMN Content:', bpmnContent);
      if (!this.bpmnModeler) {
        console.log('纳尼123？!');
        this.bpmnModeler = new BpmnViewer({
          container: '#bpmnCanvas'
        });
      }
      console.log('纳尼456？!', xmlStr);
      try {
        this.bpmnModeler.importXML(bpmnContent, (err) => {
          if (!err) {
            console.log('BPMN Diagram rendered successfully!');
            this.bpmnModeler.get('canvas').zoom('fit-viewport');
          } else {
            console.error('Error rendering BPMN Diagram:', err);
          }
        });
      } catch (error) {
        console.error('Exception while importing BPMN:', error);
      }

    },
    showViewDialog(row) {
      this.viewDialogVisible = true;
      // 延迟展示 BPMN 图，确保对话框和容器已经渲染完成
      this.$nextTick(() => {
        this.viewBpmnDiagram(row.bpmn);

      });
    },
    discoverSupplyChain() {
      axios.get('http://localhost:8182/process/getAllCombination')
        .then(response => {
          console.log("discoverSupplyChain", response)
          this.collaborations = response.data.data;
          this.discoverDialogVisible = true;
        })
        .catch(error => {
          console.error('Error fetching supply chain:', error);
          this.$message.error('发现供应链过程中出现错误');
        });
    },
    selectCollaboration(collaboration) {
      const participants = collaboration.map(item => item.participant);
      axios.post('http://localhost:8182/process/getCombineProcess', participants)
        .then(response => {
          console.log("selectCollaboration", response)
          this.processList = response.data.data;
          this.discoverDialogVisible = false;
        })
        .catch(error => {
          console.error('Error combining processes:', error);
          this.$message.error('组合协作过程中出现错误');
        });
    },
    deployProcess(row) {
      const payload = {
        engineType: row.engineCategory,
        url: '',
        pageNum: 1,
        pageSize: 100
      };
      axios.post('http://localhost:8182/engine/all', payload)
        .then(response => {
          this.deployResult = response.data.data.list;
          this.deployResultVisible = true;
        })
        .catch(error => {
          console.error('Error during deployment:', error);
          this.$message.error('部署过程中出现错误');
        });
    }
  }
}
</script>

<style scoped>
.row {
  white-space: nowrap;
  margin-top: 10px;
  padding: 0 10px;
  text-align: center;
  display: flex;
  justify-content: space-between;
}

.option-image {
  width: 20px;
  height: 20px;
  vertical-align: middle;
  margin-right: 5px;
}

.bpmn-container {
  width: 90%;
  height: 500px;
  border: 1px solid #ccc;
}

.dialog-footer .el-button {
  margin: 0 10px;
}
</style>
