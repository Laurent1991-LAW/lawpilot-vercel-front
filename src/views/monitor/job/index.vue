<template>
  <div class="app-container">
    <el-form :model="queryParams" ref="queryForm" size="small" :inline="true" v-show="showSearch" label-width="130px">
      <el-form-item label="Task Name " prop="jobName">
        <el-input
          v-model="queryParams.jobName"
          placeholder="Enter task name "
          clearable
          @keyup.enter.native="handleQuery"
        />
      </el-form-item>
      <el-form-item label="Task Group" prop="jobGroup">
        <el-select v-model="queryParams.jobGroup" placeholder="Choose task group" clearable>
          <el-option
            v-for="dict in dict.type.sys_job_group"
            :key="dict.value"
            :label="dict.label"
            :value="dict.value"
          />
        </el-select>
      </el-form-item>
      <el-form-item label="TaskStatus" prop="status">
        <el-select v-model="queryParams.status" placeholder="Choose status" clearable>
          <el-option
            v-for="dict in dict.type.sys_job_status"
            :key="dict.value"
            :label="dict.label"
            :value="dict.value"
          />
        </el-select>
      </el-form-item>
      <el-form-item>
        <el-button type="primary" icon="el-icon-search" size="mini" @click="handleQuery">Search</el-button>
        <el-button icon="el-icon-refresh" size="mini" @click="resetQuery">Reset</el-button>
      </el-form-item>
    </el-form>

    <el-row :gutter="10" class="mb8">
      <el-col :span="1.5">
        <el-button
          type="primary"
          plain
          icon="el-icon-plus"
          size="mini"
          @click="handleAdd"
          v-hasPermi="['monitor:job:add']"
        >Add</el-button>
      </el-col>
      <el-col :span="1.5">
        <el-button
          type="success"
          plain
          icon="el-icon-edit"
          size="mini"
          :disabled="single"
          @click="handleUpdate"
          v-hasPermi="['monitor:job:edit']"
        >Edit</el-button>
      </el-col>
      <el-col :span="1.5">
        <el-button
          type="danger"
          plain
          icon="el-icon-delete"
          size="mini"
          :disabled="multiple"
          @click="handleDelete"
          v-hasPermi="['monitor:job:remove']"
        >Delete</el-button>
      </el-col>
      <el-col :span="1.5">
        <el-button
          type="warning"
          plain
          icon="el-icon-download"
          size="mini"
          @click="handleExport"
          v-hasPermi="['monitor:job:export']"
        >Export</el-button>
      </el-col>
      <el-col :span="1.5">
        <el-button
          type="info"
          plain
          icon="el-icon-s-operation"
          size="mini"
          @click="handleJobLog"
          v-hasPermi="['monitor:job:query']"
        >Log </el-button>
      </el-col>
      <right-toolbar :showSearch.sync="showSearch" @queryTable="getList"></right-toolbar>
    </el-row>

    <el-table v-loading="loading" :data="jobList" @selection-change="handleSelectionChange">
      <el-table-column type="selection" width="55" align="center" />
      <el-table-column label="Number" width="100" align="center" prop="jobId" />
      <el-table-column label="Task Name " align="center" prop="jobName" :show-overflow-tooltip="true" />
      <el-table-column label="Task Group" align="center" prop="jobGroup">
        <template slot-scope="scope">
          <dict-tag :options="dict.type.sys_job_group" :value="scope.row.jobGroup"/>
        </template>
      </el-table-column>
      <el-table-column label="Target String" align="center" prop="invokeTarget" :show-overflow-tooltip="true" />
      <el-table-column label="cron" align="center" prop="cronExpression" :show-overflow-tooltip="true" />
      <el-table-column label="Status" align="center">
        <template slot-scope="scope">
          <el-switch
            v-model="scope.row.status"
            active-value="0"
            inactive-value="1"
            @change="handleStatusChange(scope.row)"
          ></el-switch>
        </template>
      </el-table-column>
      <el-table-column label="Action" align="center" class-name="small-padding fixed-width">
        <template slot-scope="scope">
          <el-button
            size="mini"
            type="text"
            icon="el-icon-edit"
            @click="handleUpdate(scope.row)"
            v-hasPermi="['monitor:job:edit']"
          >Edit</el-button>
          <el-button
            size="mini"
            type="text"
            icon="el-icon-delete"
            @click="handleDelete(scope.row)"
            v-hasPermi="['monitor:job:remove']"
          >Delete</el-button>
          <el-dropdown size="mini" @command="(command) => handleCommand(command, scope.row)" v-hasPermi="['monitor:job:changeStatus', 'monitor:job:query']">
            <el-button size="mini" type="text" icon="el-icon-d-arrow-right">More</el-button>
            <el-dropdown-menu slot="dropdown">
              <el-dropdown-item command="handleRun" icon="el-icon-caret-right"
                v-hasPermi="['monitor:job:changeStatus']">Execute Once</el-dropdown-item>
              <el-dropdown-item command="handleView" icon="el-icon-view"
                v-hasPermi="['monitor:job:query']">Task Details</el-dropdown-item>
              <el-dropdown-item command="handleJobLog" icon="el-icon-s-operation"
                v-hasPermi="['monitor:job:query']">Invocation Log </el-dropdown-item>
            </el-dropdown-menu>
          </el-dropdown>
        </template>
      </el-table-column>
    </el-table>

    <pagination
      v-show="total>0"
      :total="total"
      :page.sync="queryParams.pageNum"
      :limit.sync="queryParams.pageSize"
      @pagination="getList"
    />

    <!-- 添加或Edit定时Task对话框 -->
    <el-dialog :title="title" :visible.sync="open" width="800px" append-to-body>
      <el-form ref="form" :model="form" :rules="rules" label-width="120px">
        <el-row>
          <el-col :span="12">
            <el-form-item label="Task Name" prop="jobName">
              <el-input v-model="form.jobName" placeholder="EnterTask Name " />
            </el-form-item>
          </el-col>
          <el-col :span="12">
            <el-form-item label="Group" prop="jobGroup">
              <el-select v-model="form.jobGroup" placeholder="Please choose group">
                <el-option
                  v-for="dict in dict.type.sys_job_group"
                  :key="dict.value"
                  :label="dict.label"
                  :value="dict.value"
                ></el-option>
              </el-select>
            </el-form-item>
          </el-col>
          <el-col :span="24">
            <el-form-item prop="invokeTarget">
              <span slot="label">
                Invocation Method
                <el-tooltip placement="top">
                  <div slot="content">
                    Bean invocation exemple：ryTask.ryParams('ry')
                    <br />Class invocation exemple：com.ruoyi.quartz.task.RyTask.ryParams('ry')
                    <br />Parameters：support String, boolean, Long, Float, Integer
                  </div>
                  <i class="el-icon-question"></i>
                </el-tooltip>
              </span>
              <el-input v-model="form.invokeTarget" placeholder="Enter invocation target string" />
            </el-form-item>
          </el-col>
          <el-col :span="24">
            <el-form-item label="cron" prop="cronExpression">
              <el-input v-model="form.cronExpression" placeholder="Enter cron">
                <template slot="append">
                  <el-button type="primary" @click="handleShowCron">
                    Cron Generator
                    <i class="el-icon-time el-icon--right"></i>
                  </el-button>
                </template>
              </el-input>
            </el-form-item>
          </el-col>
          <el-col :span="24" v-if="form.jobId !== undefined">
            <el-form-item label="Status">
              <el-radio-group v-model="form.status">
                <el-radio
                  v-for="dict in dict.type.sys_job_status"
                  :key="dict.value"
                  :label="dict.value"
                >{{dict.label}}</el-radio>
              </el-radio-group>
            </el-form-item>
          </el-col>
          <el-col :span="12">
            <el-form-item label="Frequency" prop="misfirePolicy">
              <el-radio-group v-model="form.misfirePolicy" size="small">
                <el-radio-button label="1">At once</el-radio-button>
                <el-radio-button label="2">Only Once</el-radio-button>
                <el-radio-button label="3">Discard</el-radio-button>
              </el-radio-group>
            </el-form-item>
          </el-col>
          <el-col :span="12">
            <el-form-item label="Parallelism" prop="concurrent">
              <el-radio-group v-model="form.concurrent" size="small">
                <el-radio-button label="0">Allowed</el-radio-button>
                <el-radio-button label="1">Forbidden</el-radio-button>
              </el-radio-group>
            </el-form-item>
          </el-col>
        </el-row>
      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button type="primary" @click="submitForm">Submit</el-button>
        <el-button @click="cancel">Cancel</el-button>
      </div>
    </el-dialog>

    <el-dialog title="Cron Generator" :visible.sync="openCron" append-to-body destroy-on-close class="scrollbar">
      <crontab @hide="openCron=false" @fill="crontabFill" :expression="expression"></crontab>
    </el-dialog>

    <!-- TaskLog Details -->
    <el-dialog title="TaskDetails" :visible.sync="openView" width="700px" append-to-body>
      <el-form ref="form" :model="form" label-width="120px" size="mini">
        <el-row>
          <el-col :span="12">
            <el-form-item label="Number：">{{ form.jobId }}</el-form-item>
            <el-form-item label="Task Name ：">{{ form.jobName }}</el-form-item>
          </el-col>
          <el-col :span="12">
            <el-form-item label="Group：">{{ jobGroupFormat(form) }}</el-form-item>
            <el-form-item label="Creation Date：">{{ form.createTime }}</el-form-item>
          </el-col>
          <el-col :span="12">
            <el-form-item label="cron：">{{ form.cronExpression }}</el-form-item>
          </el-col>
          <el-col :span="12">
            <el-form-item label="Next time：">{{ parseTime(form.nextValidTime) }}</el-form-item>
          </el-col>
          <el-col :span="24">
            <el-form-item label="Target：">{{ form.invokeTarget }}</el-form-item>
          </el-col>
          <el-col :span="12">
            <el-form-item label="TaskStatus：">
              <div v-if="form.status == 0">Enabled</div>
              <div v-else-if="form.status == 1">Paused</div>
            </el-form-item>
          </el-col>
          <el-col :span="12">
            <el-form-item label="Parallelism：">
              <div v-if="form.concurrent == 0">Allowed</div>
              <div v-else-if="form.concurrent == 1">Forbidden</div>
            </el-form-item>
          </el-col>
          <el-col :span="12">
            <el-form-item label="Frequency：">
              <div v-if="form.misfirePolicy == 0">Default</div>
              <div v-else-if="form.misfirePolicy == 1">At once</div>
              <div v-else-if="form.misfirePolicy == 2">Only once</div>
              <div v-else-if="form.misfirePolicy == 3">Discard</div>
            </el-form-item>
          </el-col>
        </el-row>
      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button @click="openView = false">Close</el-button>
      </div>
    </el-dialog>
  </div>
</template>

<script>
import { listJob, getJob, delJob, addJob, updateJob, runJob, changeJobStatus } from "@/api/monitor/job";
import Crontab from '@/components/Crontab'

export default {
  components: { Crontab },
  name: "Job",
  dicts: ['sys_job_group', 'sys_job_status'],
  data() {
    return {
      // 遮罩层
      loading: true,
      // 选中数组
      ids: [],
      // 非单个禁用
      single: true,
      // 非多个禁用
      multiple: true,
      // 显示Search条件
      showSearch: true,
      // 总条数
      total: 0,
      // 定时Task表格数据
      jobList: [],
      // 弹出层Title
      title: "",
      // 是否显示弹出层
      open: false,
      // 是否显示Details弹出层
      openView: false,
      // 是否显示Cron表达式弹出层
      openCron: false,
      // 传入的表达式
      expression: "",
      // 查询参数
      queryParams: {
        pageNum: 1,
        pageSize: 10,
        jobName: undefined,
        jobGroup: undefined,
        status: undefined
      },
      // 表单参数
      form: {},
      // 表单校验
      rules: {
        jobName: [
          { required: true, message: "Task Name can't be blank", trigger: "blur" }
        ],
        invokeTarget: [
          { required: true, message: "Target can't be blank", trigger: "blur" }
        ],
        cronExpression: [
          { required: true, message: "Cron can't be blank", trigger: "blur" }
        ]
      }
    };
  },
  created() {
    this.getList();
  },
  methods: {
    /** 查询定时Task列表 */
    getList() {
      this.loading = true;
      listJob(this.queryParams).then(response => {
        this.jobList = response.rows;
        this.total = response.total;
        this.loading = false;
      });
    },
    // Task组名字典翻译
    jobGroupFormat(row, column) {
      return this.selectDictLabel(this.dict.type.sys_job_group, row.jobGroup);
    },
    // 取消按钮
    cancel() {
      this.open = false;
      this.reset();
    },
    // 表单Reset
    reset() {
      this.form = {
        jobId: undefined,
        jobName: undefined,
        jobGroup: undefined,
        invokeTarget: undefined,
        cronExpression: undefined,
        misfirePolicy: 1,
        concurrent: 1,
        status: "0"
      };
      this.resetForm("form");
    },
    /** Search按钮Action */
    handleQuery() {
      this.queryParams.pageNum = 1;
      this.getList();
    },
    /** Reset按钮Action */
    resetQuery() {
      this.resetForm("queryForm");
      this.handleQuery();
    },
    // 多选框选中数据
    handleSelectionChange(selection) {
      this.ids = selection.map(item => item.jobId);
      this.single = selection.length != 1;
      this.multiple = !selection.length;
    },
    // MoreAction触发
    handleCommand(command, row) {
      switch (command) {
        case "handleRun":
          this.handleRun(row);
          break;
        case "handleView":
          this.handleView(row);
          break;
        case "handleJobLog":
          this.handleJobLog(row);
          break;
        default:
          break;
      }
    },
    // TaskStatusEdit
    handleStatusChange(row) {
      let text = row.status === "0" ? "enable" : "disable";
      this.$modal.confirm('Confirm to "' + text + '""' + row.jobName + '" Task?').then(function() {
        return changeJobStatus(row.jobId, row.status);
      }).then(() => {
        this.$modal.msgSuccess(text + " succeed");
      }).catch(function() {
        row.status = row.status === "0" ? "1" : "0";
      });
    },
    /* 立即执行一次 */
    handleRun(row) {
      this.$modal.confirm('Confirm to execute for one time "' + row.jobName + '" Task?').then(function() {
        return runJob(row.jobId, row.jobGroup);
      }).then(() => {
        this.$modal.msgSuccess("Successfully Executed");
      }).catch(() => {});
    },
    /** TaskDetails信息 */
    handleView(row) {
      getJob(row.jobId).then(response => {
        this.form = response.data;
        this.openView = true;
      });
    },
    /** cron表达式按钮Action */
    handleShowCron() {
      this.expression = this.form.cronExpression;
      this.openCron = true;
    },
    /** 确定后回传值 */
    crontabFill(value) {
      this.form.cronExpression = value;
    },
    /** TaskLog 列表查询 */
    handleJobLog(row) {
      const jobId = row.jobId || 0;
      this.$router.push('/monitor/job-log/index/' + jobId)
    },
    /** Add按钮Action */
    handleAdd() {
      this.reset();
      this.open = true;
      this.title = "Add Task";
    },
    /** Edit按钮Action */
    handleUpdate(row) {
      this.reset();
      const jobId = row.jobId || this.ids;
      getJob(jobId).then(response => {
        this.form = response.data;
        this.open = true;
        this.title = "Edit Task";
      });
    },
    /** 提交按钮 */
    submitForm: function() {
      this.$refs["form"].validate(valid => {
        if (valid) {
          if (this.form.jobId != undefined) {
            updateJob(this.form).then(response => {
              this.$modal.msgSuccess("Successfully edited");
              this.open = false;
              this.getList();
            });
          } else {
            addJob(this.form).then(response => {
              this.$modal.msgSuccess("Successfully added");
              this.open = false;
              this.getList();
            });
          }
        }
      });
    },
    /** Delete按钮Action */
    handleDelete(row) {
      const jobIds = row.jobId || this.ids;
      this.$modal.confirm('Confirm delete Task "' + jobIds + '"?').then(function() {
        return delJob(jobIds);
      }).then(() => {
        this.getList();
        this.$modal.msgSuccess("Successfully deleted");
      }).catch(() => {});
    },
    /** Export按钮Action */
    handleExport() {
      this.download('monitor/job/export', {
        ...this.queryParams
      }, `job_${new Date().getTime()}.xlsx`)
    }
  }
};
</script>
