<template>
  <div class="app-container">
    <el-form :model="queryParams" ref="queryForm" size="small" :inline="true" v-show="showSearch" label-width="150px">
      <el-form-item label="Origin Address" prop="operIp">
        <el-input
          v-model="queryParams.operIp"
          placeholder="Enter origin"
          clearable
          style="width: 240px;"
          @keyup.enter.native="handleQuery"
        />
      </el-form-item>
      <el-form-item label="Operation Menu" prop="title">
        <el-input
          v-model="queryParams.title"
          placeholder="Enter menu"
          clearable
          style="width: 240px;"
          @keyup.enter.native="handleQuery"
        />
      </el-form-item>
      <el-form-item label="Operation User" prop="operName">
        <el-input
          v-model="queryParams.operName"
          placeholder="Enter operation user"
          clearable
          style="width: 240px;"
          @keyup.enter.native="handleQuery"
        />
      </el-form-item>
      <el-form-item label="Type" prop="businessType">
        <el-select
          v-model="queryParams.businessType"
          placeholder="ActionType"
          clearable
          style="width: 240px"
        >
          <el-option
            v-for="dict in dict.type.sys_oper_type"
            :key="dict.value"
            :label="dict.label"
            :value="dict.value"
          />
        </el-select>
      </el-form-item>
      <el-form-item label="Status" prop="status">
        <el-select
          v-model="queryParams.status"
          placeholder="Status"
          clearable
          style="width: 240px"
        >
          <el-option
            v-for="dict in dict.type.sys_common_status"
            :key="dict.value"
            :label="dict.label"
            :value="dict.value"
          />
        </el-select>
      </el-form-item>
      <el-form-item label="Operation Date">
        <el-date-picker
          v-model="dateRange"
          style="width: 240px"
          value-format="yyyy-MM-dd HH:mm:ss"
          type="daterange"
          range-separator="-"
          start-placeholder="Start Date"
          end-placeholder="End Date"
          :default-time="['00:00:00', '23:59:59']"
        ></el-date-picker>
      </el-form-item>
      <el-form-item>
        <el-button type="primary" icon="el-icon-search" size="mini" @click="handleQuery">Search</el-button>
        <el-button icon="el-icon-refresh" size="mini" @click="resetQuery">Reset</el-button>
      </el-form-item>
    </el-form>

    <el-row :gutter="10" class="mb8">
      <el-col :span="1.5">
        <el-button
          type="danger"
          plain
          icon="el-icon-delete"
          size="mini"
          :disabled="multiple"
          @click="handleDelete"
          v-hasPermi="['monitor:operlog:remove']"
        >Delete</el-button>
      </el-col>
      <el-col :span="1.5">
        <el-button
          type="danger"
          plain
          icon="el-icon-delete"
          size="mini"
          @click="handleClean"
          v-hasPermi="['monitor:operlog:remove']"
        >Clear</el-button>
      </el-col>
      <el-col :span="1.5">
        <el-button
          type="warning"
          plain
          icon="el-icon-download"
          size="mini"
          @click="handleExport"
          v-hasPermi="['monitor:operlog:export']"
        >Export</el-button>
      </el-col>
      <right-toolbar :showSearch.sync="showSearch" @queryTable="getList"></right-toolbar>
    </el-row>

    <el-table ref="tables" v-loading="loading" :data="list" @selection-change="handleSelectionChange" :default-sort="defaultSort" @sort-change="handleSortChange">
      <el-table-column type="selection" width="50" align="center" />
      <el-table-column label="Number" align="center" prop="operId" />
      <el-table-column label="Menu" align="center" prop="title" :show-overflow-tooltip="true" />
      <el-table-column label="Type" align="center" prop="businessType">
        <template slot-scope="scope">
          <dict-tag :options="dict.type.sys_oper_type" :value="scope.row.businessType"/>
        </template>
      </el-table-column>
      <el-table-column label="User" align="center" prop="operName" width="110" :show-overflow-tooltip="true" sortable="custom" :sort-orders="['descending', 'ascending']" />
      <el-table-column label="Address" align="center" prop="operIp" width="130" :show-overflow-tooltip="true" />
      <el-table-column label="Location" align="center" prop="operLocation" :show-overflow-tooltip="true" />
      <el-table-column label="Status" align="center" prop="status">
        <template slot-scope="scope">
          <dict-tag :options="dict.type.sys_common_status" :value="scope.row.status"/>
        </template>
      </el-table-column>
      <el-table-column label="Date" align="center" prop="operTime" width="160" sortable="custom" :sort-orders="['descending', 'ascending']">
        <template slot-scope="scope">
          <span>{{ parseTime(scope.row.operTime) }}</span>
        </template>
      </el-table-column>
      <el-table-column label="Time Consumption" align="center" prop="costTime" width="110" :show-overflow-tooltip="true" sortable="custom" :sort-orders="['descending', 'ascending']">
        <template slot-scope="scope">
          <span>{{ scope.row.costTime }}milliseconds</span>
        </template>
      </el-table-column>
      <el-table-column label="Action" align="center" class-name="small-padding fixed-width">
        <template slot-scope="scope">
          <el-button
            size="mini"
            type="text"
            icon="el-icon-view"
            @click="handleView(scope.row,scope.index)"
            v-hasPermi="['monitor:operlog:query']"
          >Details</el-button>
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

    <!-- ActionLog Details -->
    <el-dialog title="ActionLog Details" :visible.sync="open" width="800px" append-to-body>
      <el-form ref="form" :model="form" label-width="100px" size="mini">
        <el-row>
          <el-col :span="12">
            <el-form-item label="Menu：">{{ form.title }} / {{ typeFormat(form) }}</el-form-item>
            <el-form-item
              label="Sign-in info："
            >{{ form.operName }} / {{ form.operIp }} / {{ form.operLocation }}</el-form-item>
          </el-col>
          <el-col :span="12">
            <el-form-item label="Request Origin：">{{ form.operUrl }}</el-form-item>
            <el-form-item label="Request Type：">{{ form.requestMethod }}</el-form-item>
          </el-col>
          <el-col :span="24">
            <el-form-item label="Request Method：">{{ form.method }}</el-form-item>
          </el-col>
          <el-col :span="24">
            <el-form-item label="Request Parameters：">{{ form.operParam }}</el-form-item>
          </el-col>
          <el-col :span="24">
            <el-form-item label="Return Parameters：">{{ form.jsonResult }}</el-form-item>
          </el-col>
          <el-col :span="8">
            <el-form-item label="ActionStatus：">
              <div v-if="form.status === 0">Pass</div>
              <div v-else-if="form.status === 1">Fail</div>
            </el-form-item>
          </el-col>
          <el-col :span="8">
            <el-form-item label="Time Consumption：">{{ form.costTime }}milliseconds</el-form-item>
          </el-col>
          <el-col :span="8">
            <el-form-item label="Date：">{{ parseTime(form.operTime) }}</el-form-item>
          </el-col>
          <el-col :span="24">
            <el-form-item label="Error Info：" v-if="form.status === 1">{{ form.errorMsg }}</el-form-item>
          </el-col>
        </el-row>
      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button @click="open = false">Close</el-button>
      </div>
    </el-dialog>
  </div>
</template>

<script>
import { list, delOperlog, cleanOperlog } from "@/api/monitor/operlog";

export default {
  name: "Operlog",
  dicts: ['sys_oper_type', 'sys_common_status'],
  data() {
    return {
      // 遮罩层
      loading: true,
      // 选中数组
      ids: [],
      // 非多个禁用
      multiple: true,
      // 显示Search条件
      showSearch: true,
      // 总条数
      total: 0,
      // 表格数据
      list: [],
      // 是否显示弹出层
      open: false,
      // 日期范围
      dateRange: [],
      // 默认Order
      defaultSort: {prop: 'operTime', order: 'descending'},
      // 表单参数
      form: {},
      // 查询参数
      queryParams: {
        pageNum: 1,
        pageSize: 10,
        operIp: undefined,
        title: undefined,
        operName: undefined,
        businessType: undefined,
        status: undefined
      }
    };
  },
  created() {
    this.getList();
  },
  methods: {
    /** 查询登录Log  */
    getList() {
      this.loading = true;
      list(this.addDateRange(this.queryParams, this.dateRange)).then( response => {
          this.list = response.rows;
          this.total = response.total;
          this.loading = false;
        }
      );
    },
    // ActionLog Type字典翻译
    typeFormat(row, column) {
      return this.selectDictLabel(this.dict.type.sys_oper_type, row.businessType);
    },
    /** Search按钮Action */
    handleQuery() {
      this.queryParams.pageNum = 1;
      this.getList();
    },
    /** Reset按钮Action */
    resetQuery() {
      this.dateRange = [];
      this.resetForm("queryForm");
      this.queryParams.pageNum = 1;
      this.$refs.tables.sort(this.defaultSort.prop, this.defaultSort.order)
    },
    /** 多选框选中数据 */
    handleSelectionChange(selection) {
      this.ids = selection.map(item => item.operId)
      this.multiple = !selection.length
    },
    /** Order触发事件 */
    handleSortChange(column, prop, order) {
      this.queryParams.orderByColumn = column.prop;
      this.queryParams.isAsc = column.order;
      this.getList();
    },
    /** Details按钮Action */
    handleView(row) {
      this.open = true;
      this.form = row;
    },
    /** Delete按钮Action */
    handleDelete(row) {
      const operIds = row.operId || this.ids;
      this.$modal.confirm('Confirm delete Log Number "' + operIds + '"?').then(function() {
        return delOperlog(operIds);
      }).then(() => {
        this.getList();
        this.$modal.msgSuccess("Successfully deleted");
      }).catch(() => {});
    },
    /** Clear按钮Action */
    handleClean() {
      this.$modal.confirm('Confirm clear all logs info ?').then(function() {
        return cleanOperlog();
      }).then(() => {
        this.getList();
        this.$modal.msgSuccess("Successfully Clear");
      }).catch(() => {});
    },
    /** Export按钮Action */
    handleExport() {
      this.download('monitor/operlog/export', {
        ...this.queryParams
      }, `operlog_${new Date().getTime()}.xlsx`)
    }
  }
};
</script>

