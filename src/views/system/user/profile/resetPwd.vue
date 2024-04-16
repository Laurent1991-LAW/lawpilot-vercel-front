<template>
  <el-form ref="form" :model="user" :rules="rules" label-width="130px">
    <el-form-item label="Old Password" prop="oldPassword">
      <el-input v-model="user.oldPassword" placeholder="Enter your old password" type="password" show-password/>
    </el-form-item>
    <el-form-item label="New Password" prop="newPassword">
      <el-input v-model="user.newPassword" placeholder="Enter your new password" type="password" show-password/>
    </el-form-item>
    <el-form-item label="New Password" prop="confirmPassword">
      <el-input v-model="user.confirmPassword" placeholder="Re-enter your new password" type="password" show-password/>
    </el-form-item>
    <el-form-item>
      <el-button type="primary" size="mini" @click="submit">Save</el-button>
      <el-button type="danger" size="mini" @click="close">Cancel</el-button>
    </el-form-item>
  </el-form>
</template>

<script>
import { updateUserPwd } from "@/api/system/user";

export default {
  data() {
    const equalToPassword = (rule, value, callback) => {
      if (this.user.newPassword !== value) {
        callback(new Error("Passwords do not match."));
      } else {
        callback();
      }
    };
    return {
      user: {
        oldPassword: undefined,
        newPassword: undefined,
        confirmPassword: undefined
      },
      // 表单校验
      rules: {
        oldPassword: [
          { required: true, message: "Old password can't be blank", trigger: "blur" }
        ],
        newPassword: [
          { required: true, message: "New password can't be blank", trigger: "blur" },
          { min: 6, max: 20, message: "length between 6 and 20 characters", trigger: "blur" },
          { pattern: /^[^<>"'|\\]+$/, message: "Invalid characters contained: < > \" ' \\\ |", trigger: "blur" }
        ],
        confirmPassword: [
          { required: true, message: "New password can't be blank", trigger: "blur" },
          { required: true, validator: equalToPassword, trigger: "blur" }
        ]
      }
    };
  },
  methods: {
    submit() {
      this.$refs["form"].validate(valid => {
        if (valid) {
          updateUserPwd(this.user.oldPassword, this.user.newPassword).then(response => {
            this.$modal.msgSuccess("Successfully saved");
          });
        }
      });
    },
    close() {
      this.$tab.closePage();
    }
  }
};
</script>
