<template>
  <div class="app-container">
    <el-row :gutter="10">
      <el-col :span="8">
        <el-card style="height: calc(100vh - 125px)">
          <div slot="header">
            <span><i class="el-icon-collection"></i> Cache列表</span>
            <el-button
              style="float: right; padding: 3px 0"
              type="text"
              icon="el-icon-refresh-right"
              @click="refreshCacheNames()"
            ></el-button>
          </div>
          <el-table
            v-loading="loading"
            :data="cacheNames"
            :height="tableHeight"
            highlight-current-row
            @row-click="getCacheKeys"
            style="width: 100%"
          >
            <el-table-column
              label="序号"
              width="60"
              type="index"
            ></el-table-column>

            <el-table-column
              label="Cache Name "
              align="center"
              prop="cacheName"
              :show-overflow-tooltip="true"
              :formatter="nameFormatter"
            ></el-table-column>

            <el-table-column
              label="Description"
              align="center"
              prop="remark"
              :show-overflow-tooltip="true"
            />
            <el-table-column
              label="Action"
              width="60"
              align="center"
              class-name="small-padding fixed-width"
            >
              <template slot-scope="scope">
                <el-button
                  size="mini"
                  type="text"
                  icon="el-icon-delete"
                  @click="handleClearCacheName(scope.row)"
                ></el-button>
              </template>
            </el-table-column>
          </el-table>
        </el-card>
      </el-col>

      <el-col :span="8">
        <el-card style="height: calc(100vh - 125px)">
          <div slot="header">
            <span><i class="el-icon-key"></i> 键名列表</span>
            <el-button
              style="float: right; padding: 3px 0"
              type="text"
              icon="el-icon-refresh-right"
              @click="refreshCacheKeys()"
            ></el-button>
          </div>
          <el-table
            v-loading="subLoading"
            :data="cacheKeys"
            :height="tableHeight"
            highlight-current-row
            @row-click="handleCacheValue"
            style="width: 100%"
          >
            <el-table-column
              label="序号"
              width="60"
              type="index"
            ></el-table-column>
            <el-table-column
              label="Cache键名"
              align="center"
              :show-overflow-tooltip="true"
              :formatter="keyFormatter"
            >
            </el-table-column>
            <el-table-column
              label="Action"
              width="60"
              align="center"
              class-name="small-padding fixed-width"
            >
              <template slot-scope="scope">
                <el-button
                  size="mini"
                  type="text"
                  icon="el-icon-delete"
                  @click="handleClearCacheKey(scope.row)"
                ></el-button>
              </template>
            </el-table-column>
          </el-table>
        </el-card>
      </el-col>

      <el-col :span="8">
        <el-card :bordered="false" style="height: calc(100vh - 125px)">
          <div slot="header">
            <span><i class="el-icon-document"></i> Cache内容</span>
            <el-button
              style="float: right; padding: 3px 0"
              type="text"
              icon="el-icon-refresh-right"
              @click="handleClearCacheAll()"
              >清理全部</el-button
            >
          </div>
          <el-form :model="cacheForm">
            <el-row :gutter="32">
              <el-col :offset="1" :span="22">
                <el-form-item label="Cache Name :" prop="cacheName">
                  <el-input v-model="cacheForm.cacheName" :readOnly="true" />
                </el-form-item>
              </el-col>
              <el-col :offset="1" :span="22">
                <el-form-item label="Cache键名:" prop="cacheKey">
                  <el-input v-model="cacheForm.cacheKey" :readOnly="true" />
                </el-form-item>
              </el-col>
              <el-col :offset="1" :span="22">
                <el-form-item label="Cache内容:" prop="cacheValue">
                  <el-input
                    v-model="cacheForm.cacheValue"
                    type="textarea"
                    :rows="8"
                    :readOnly="true"
                  />
                </el-form-item>
              </el-col>
            </el-row>
          </el-form>
        </el-card>
      </el-col>
    </el-row>
  </div>
</template>

<script>
import { listCacheName, listCacheKey, getCacheValue, clearCacheName, clearCacheKey, clearCacheAll } from "@/api/monitor/cache";

export default {
  name: "CacheList",
  data() {
    return {
      cacheNames: [],
      cacheKeys: [],
      cacheForm: {},
      loading: true,
      subLoading: false,
      nowCacheName: "",
      tableHeight: window.innerHeight - 200
    };
  },
  created() {
    this.getCacheNames();
  },
  methods: {
    /** 查询Cache Name 列表 */
    getCacheNames() {
      this.loading = true;
      listCacheName().then(response => {
        this.cacheNames = response.data;
        this.loading = false;
      });
    },
    /** Flush Name 列表 */
    refreshCacheNames() {
      this.getCacheNames();
      this.$modal.msgSuccess("Flush列表成功");
    },
    /** 清理指定 Name Cache */
    handleClearCacheName(row) {
      clearCacheName(row.cacheName).then(response => {
        this.$modal.msgSuccess("清理Cache Name [" + row.cacheName + "]成功");
        this.getCacheKeys();
      });
    },
    /** 查询Cache键名列表 */
    getCacheKeys(row) {
      const cacheName = row !== undefined ? row.cacheName : this.nowCacheName;
      if (cacheName === "") {
        return;
      }
      this.subLoading = true;
      listCacheKey(cacheName).then(response => {
        this.cacheKeys = response.data;
        this.subLoading = false;
        this.nowCacheName = cacheName;
      });
    },
    /** Flush键名列表 */
    refreshCacheKeys() {
      this.getCacheKeys();
      this.$modal.msgSuccess("刷新键名列表成功");
    },
    /** 清理指定键名Cache */
    handleClearCacheKey(cacheKey) {
      clearCacheKey(cacheKey).then(response => {
        this.$modal.msgSuccess("清理Cache键名[" + cacheKey + "]成功");
        this.getCacheKeys();
      });
    },
    /** 列表前缀去除 */
    nameFormatter(row) {
      return row.cacheName.replace(":", "");
    },
    /** 键名前缀去除 */
    keyFormatter(cacheKey) {
      return cacheKey.replace(this.nowCacheName, "");
    },
    /** 查询Cache内容Details */
    handleCacheValue(cacheKey) {
      getCacheValue(this.nowCacheName, cacheKey).then(response => {
        this.cacheForm = response.data;
      });
    },
    /** 清理全部Cache */
    handleClearCacheAll() {
      clearCacheAll().then(response => {
        this.$modal.msgSuccess("清理全部Cache成功");
      });
    }
  },
};
</script>
