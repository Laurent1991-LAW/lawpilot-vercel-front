<template>
  <el-form ref="basicInfoForm" :model="info" :rules="rules" label-width="150px">
    <el-row>
      <el-col :span="12">
        <el-form-item label="表 Name " prop="tableName">
          <el-input placeholder="Enter仓库 Name " v-model="info.tableName" />
        </el-form-item>
      </el-col>
      <el-col :span="12">
        <el-form-item label="表描述" prop="tableComment">
          <el-input placeholder="Enter" v-model="info.tableComment" />
        </el-form-item>
      </el-col>
      <el-col :span="12">
        <el-form-item label="实体类 Name " prop="className">
          <el-input placeholder="Enter" v-model="info.className" />
        </el-form-item>
      </el-col>
      <el-col :span="12">
        <el-form-item label="作者" prop="functionAuthor">
          <el-input placeholder="Enter" v-model="info.functionAuthor" />
        </el-form-item>
      </el-col>
      <el-col :span="24">
        <el-form-item label="Description" prop="remark">
          <el-input type="textarea" :rows="3" v-model="info.remark"></el-input>
        </el-form-item>
      </el-col>
    </el-row>
  </el-form>
</template>

<script>
export default {
  props: {
    info: {
      type: Object,
      default: null
    }
  },
  data() {
    return {
      rules: {
        tableName: [
          { required: true, message: "Enter表 Name ", trigger: "blur" }
        ],
        tableComment: [
          { required: true, message: "Enter表描述", trigger: "blur" }
        ],
        className: [
          { required: true, message: "Enter实体类 Name ", trigger: "blur" }
        ],
        functionAuthor: [
          { required: true, message: "Enter作者", trigger: "blur" }
        ]
      }
    };
  }
};
</script>
