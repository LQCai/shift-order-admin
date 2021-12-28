<template>
  <div>
    <el-col :span="24">
      <basic-container>
        <el-collapse v-model="activeNames" @change="handleChange">
          <el-collapse-item title="帮助" name="1">
            <div>有问题请联系 孔维信</div>
            <div>微信：kwx43134</div>
            <div>电话：13853145123</div>
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

