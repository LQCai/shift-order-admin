<template>
  <div>
    <el-col :span="24">
      <basic-container>
        <el-collapse v-model="activeNames" @change="handleChange">
          <el-collapse-item title="拥有的核心功能" name="1">
            <div>1.开发中...</div>
          </el-collapse-item>
          <el-collapse-item title="软件定制开发合作" name="2">
            <div>1.接BladeX系列架构的定制服务</div>
            <div>2.接3个月以内工期的react、vue、springboot、springcloud、app、小程序等软件定制服务</div>
            <div>3.有意向请联系唯一指定微信:lqc0097</div>
          </el-collapse-item>
        </el-collapse>
      </basic-container>
    </el-col>
  </div>
</template>

<script>
  import {mapGetters} from "vuex";

  export default {
    name: "wel",
    data() {
      return {
        activeNames: ['1', '2', '3', '5'],
        logActiveNames: ['26']
      };
    },
    computed: {
      ...mapGetters(["userInfo"]),
    },
    created() {
    },
    methods: {
      handleChange(val) {
        console.log(val);
      }
    }
  };
</script>

<style>
  .el-divider--horizontal {
    margin: 12px 0 !important;
  }

  .el-font-size {
    font-size: 14px;
  }
</style>

