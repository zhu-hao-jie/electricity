<template>
  <div class="app-container">
    <div class="filter-container">
      <el-input
        v-model="listQuery.name"
        placeholder="试验名称"
        style="width: 200px"
        class="filter-item"
        @keyup.enter.native="handleFilter"
      />
      <el-input
        v-model="listQuery.id"
        placeholder="任务ID"
        style="width: 200px"
        class="filter-item"
        @keyup.enter.native="handleFilter"
      />
      <!-- <el-input
        v-model="listQuery.device_no"
        placeholder="设备ID"
        style="width: 200px"
        class="filter-item"
        @keyup.enter.native="handleFilter"
      /> -->
      <!-- <el-select
        v-model="listQuery."
        placeholder="ID"
        clearable
        style="width: 90px"
        class="filter-item"
      >
        <el-option
          v-for="item in importanceOptions"
          :key="item"
          :label="item"
          :value="item"
        />
      </el-select> -->
      <el-select
        v-model="listQuery.status"
        placeholder="处理状态"
        clearable
        class="filter-item"
        style="width: 130px"
        @change="handleFilter"
      >
        <el-option
          v-for="item in queryOptions.statusOptions"
          :key="item"
          :label="item"
          :value="item"
        />
      </el-select>
      <el-select
        v-model="listQuery.district"
        placeholder="检测机构"
        clearable
        style="width: 140px"
        class="filter-item"
        @change="handleFilter"
      >
        <el-option
          v-for="item in queryOptions.districtOptions"
          :key="item"
          :label="item"
          :value="item"
        />
      </el-select>
      <el-button
        v-waves
        class="filter-item"
        type="primary"
        icon="el-icon-search"
        @click="handleFilter"
      >
        搜索
      </el-button>
      <el-button
        class="filter-item"
        style="margin-left: 10px"
        type="success"
        icon="el-icon-plus"
        @click="addNodedialogVisible = true"
      >
        添加
      </el-button>
      <el-button
        v-waves
        :loading="downloadLoading"
        class="filter-item"
        type="warning"
        icon="el-icon-edit"
        @click="handleDownload"
      >
        编辑
      </el-button>
    </div>

    <el-table
      v-loading="listLoading"
      :data="list"
      element-loading-text="Loading"
      border
      fit
      highlight-current-row
    >
      <el-table-column align="center" label="ID" width="95">
        <template slot-scope="scope">
          {{ scope.$index + 1 }}
        </template>
      </el-table-column>
      <el-table-column label="机构">
        <template slot-scope="scope">
          {{ scope.row.district_id == 1 ? "省中心（电科院）" : "" }}
        </template>
      </el-table-column>
      <el-table-column label="工位名称">
        <template slot-scope="scope">
          {{ scope.row.station }}
        </template>
      </el-table-column>
      <el-table-column label="物资类别">
        <template slot-scope="scope">
          {{ scope.row.type }}
        </template>
      </el-table-column>
      <el-table-column label="试验名称">
        <template slot-scope="scope">
          {{ scope.row.experiment_name }}
        </template>
      </el-table-column>

      <el-table-column
        class-name="status-col"
        label="状态"
        width="110"
        align="center"
      >
        <template slot-scope="scope">
          <el-tag :type="scope.row.status | statusFilter">{{
            scope.row.status
          }}</el-tag>
        </template>
      </el-table-column>

      <el-table-column label="问题类型">
        <template slot-scope="scope">
          {{ scope.row.description }}
        </template>
      </el-table-column>
      <el-table-column
        align="center"
        prop="created_at"
        label="发生时间"
        width="200"
      >
        <template slot-scope="scope">
          <i class="el-icon-time" />
          <span>{{ scope.row.updatetime }}</span>
        </template>
      </el-table-column>
      <el-table-column label="上链信息">
        <el-button
          :v-if="show"
          type="primary"
          size="mini"
          @click="dialogVisible = true"
        >
          查看详情
        </el-button>
      </el-table-column>
    </el-table>

    <el-dialog
      title="上链信息"
      :visible.sync="dialogVisible"
      width="30%"
      :before-close="handleClose"
    >
      <el-timeline :reverse="reverse">
        <el-timeline-item
          v-for="(activity, index) in activities"
          :key="index"
          :timestamp="activity.timestamp"
        >
          <p>交易ID：{{ activity.transaction_id }}</p>
          <p>数据哈希：{{ activity.data_hash }}</p>
          <p>状态变更：{{ activity.status }}</p>
        </el-timeline-item>
      </el-timeline>

      <span slot="footer" class="dialog-footer">
        <el-button @click="dialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="dialogVisible = false"
          >确 定</el-button
        >
      </span>
    </el-dialog>
  </div>
</template>

<script>
import { getList } from "@/api/table";

export default {
  filters: {
    statusFilter(status) {
      const statusMap = {
        published: "success",
        draft: "gray",
        deleted: "danger",
      };
      return statusMap[status];
    },
  },
  data() {
    return {
      listQuery: {},
      activities: [
        {
          transaction_id:
            "0fa5db699d9f8aed7c75b478b2876a37bdbd0a5b819f4b9b52c6c8d63a1d2eb3",
          data_hash:
            "	06120358e39a1ee7dd29dcc77f48cc4444741df07c0381e24f4a240b600cf179",
          status: "待解决",
          timestamp: "2022-12-10T10:47:02.653Z",
        },
        {
          transaction_id: "adlfgkakerhgaasdgaehjgfkytlkg",
          data_hash: "alskjdfhkaweghajrtyjtryjrtwg",
          status: "已排查",
          timestamp: "2018-04-15 11:56 ***",
        },
      ],
      dialogVisible: false,
      list: [
        {
          district_id: 1, // 机构
          experiment_name: "耐压试验", // 试验名称
          status: "待解决",
          type: "低压开关柜",
          description: "操作不规范",
          updatetime: "2022-11-11 19:13",
          station: "A1",
          transaction_id: "asifhwoigaewjiofgasg",
        },
        {
          district_id: 1, // 机构
          experiment_name: "耐压试验", // 试验名称
          status: "待解决",
          type: "低压开关柜",
          description: "操作不规范",
          updatetime: "2022-11-11 19:13",
          station: "A1",
          transaction_id: "asifhwoigaewjiofgasg",
        },
      ],
      listLoading: false,
      listQuery: {
        status: "",
        type: "",
        district: "",
      },
      queryOptions: {
        districtOptions: [
          "苏北分中心",
          "苏南分中心",
          "苏中分中心",
          "省中心（电科院）",
        ],
        statusOptions: ["已解决", "待解决"],
        type: ["低压开关柜", "电力电缆"],
      },
    };
  },
  methods: {
    handleClose(done) {
      this.$confirm("确认关闭？")
        .then((_) => {
          done();
        })
        .catch((_) => {});
    },
  },
  created() {
    // this.fetchData();
  },
  methods: {
    fetchData() {
      this.listLoading = true;
      getList().then((response) => {
        this.list = response.data.items;
        this.listLoading = false;
      });
    },
  },
};
</script>
