<template>
  <div class="search-form">
    <!-- 头部tab切换 -->
    <el-row type="flex" class="search-tab">
      <span
        v-for="(item, index) in tabs"
        :key="index"
        @click="handleSearchTab(item, index)"
        :class="{active: index === currentTab}"
      >
        <i :class="item.icon"></i>
        {{item.name}}
      </span>
    </el-row>

    <el-form class="search-form-content" ref="form" label-width="80px">
      <el-form-item label="出发城市">
        <el-autocomplete
          :fetch-suggestions="queryDepartSearch"
          placeholder="请搜索出发城市"
          @select="handleDepartSelect"
          @blur="defaultDepartChoice"
          class="el-autocomplete"
          v-model="userdata.departCity"
        ></el-autocomplete>
      </el-form-item>
      <el-form-item label="到达城市">
        <el-autocomplete
          :fetch-suggestions="queryDestSearch"
          placeholder="请搜索到达城市"
          @select="handleDestSelect"
          @blur="defaultDestChoice"
          class="el-autocomplete"
          v-model="userdata.destCity"
        ></el-autocomplete>
      </el-form-item>
      <el-form-item label="出发时间">
        <!-- change 用户确认选择日期时触发 -->
        <el-date-picker
          type="date"
          placeholder="请选择日期"
          style="width: 100%;"
          @change="handleDate"
          v-model="userdata.departData"
          format="yyyy - MM - dd "
        ></el-date-picker>
      </el-form-item>
      <el-form-item label>
        <el-button style="width:100%;" type="primary" icon="el-icon-search" @click="handleSubmit">搜索</el-button>
      </el-form-item>
      <div class="reverse">
        <span @click="handleReverse">换</span>
      </div>
    </el-form>
  </div>
</template>

<script>
import moment from "moment";
export default {
  data() {
    return {
      tabs: [
        { icon: "iconfont icondancheng", name: "单程" },
        { icon: "iconfont iconshuangxiang", name: "往返" }
      ],
      currentTab: 0,
      userdata: {
        departCity: "",
        departCode: "",
        destCity: "",
        destCode: "",
        departData: ""
      },
      departCityList: [],
      destCityList: []
    };
  },
  methods: {
    // tab切换时触发
    handleSearchTab(item, index) {},

    // 出发城市输入框获得焦点时触发
    // value 是选中的值，cb是回调函数，接收要展示的列表
    queryDepartSearch(value, cb) {
      if (!this.userdata.departCity) {
        cb([]);
      } else {
        this.$store
          .dispatch("post/getPlace", this.userdata.departCity)
          .then(res => {
            this.departCityList = res;
            cb(res);
          });
      }
    },

    // 目标城市输入框获得焦点时触发
    // value 是选中的值，cb是回调函数，接收要展示的列表
    queryDestSearch(value, cb) {
      if (!this.userdata.destCity) {
        cb([]);
      } 
      else {
        this.$store
          .dispatch("post/getPlace", this.userdata.destCity)
          .then(res => {
            this.destCityList = res;
            cb(res);
          });
      }
    },

    // 出发城市下拉选择时触发
    handleDepartSelect(item) {
      this.userdata.departCity = item.value;
      this.userdata.departCode = item.sort;
    },

    // 目标城市下拉选择时触发
    handleDestSelect(item) {
      this.userdata.destCity = item.value;
      this.userdata.destCode = item.sort;
    },
    // 默认的的出发城市
    defaultDepartChoice() {
      if(!this.userdata.departCity){
        return
      }
      if (!this.departCityList[0]) {
        return;
      }
      this.userdata.departCity = this.departCityList[0].value;
      this.userdata.departCode = this.departCityList[0].sort;
    },
    //默认的到达城市
    defaultDestChoice() {
      if(!this.userdata.destCity){
        return
      }
      if (!this.destCityList[0]) {
        return;
      }
      this.userdata.destCity = this.destCityList[0].value;
      this.userdata.destCode = this.destCityList[0].sort;
    },
    // 确认选择日期时触发
    handleDate() {
      this.userdata.departData = moment(this.userdata.departData).format(
        "YYYY-MM-DD"
      );
    },

    // 触发和目标城市切换时触发
    handleReverse() {
      let { departCity, departCode, destCity, destCode } = this.userdata;
      this.userdata.departCity = destCity;
      this.userdata.departCode = destCode;
      this.userdata.destCity = departCity;
      this.userdata.destCode = departCode;
    },

    // 提交表单是触发
    handleSubmit() {
      //减数据存储到本地制作历史查询功能
      // this.$store.commit("air/setHistory", []);
      this.$store.commit("air/setHistory", this.userdata);
      this.$router.push({
        path: "/air/flights",
        query: this.userdata
      });
    }
  },
  mounted() {}
};
</script>

<style scoped lang="less">
.search-form {
  border: 1px #ddd solid;
  border-top: none;
  width: 360px;
  height: 350px;
  box-sizing: border-box;
}

.search-tab {
  span {
    display: block;
    flex: 1;
    text-align: center;
    height: 48px;
    line-height: 42px;
    box-sizing: border-box;
    border-top: 3px #eee solid;
    background: #eee;
  }

  .active {
    border-top-color: orange;
    background: #fff;
  }

  i {
    margin-right: 5px;
    font-size: 18px;

    &:first-child {
      font-size: 16px;
    }
  }
}

.search-form-content {
  padding: 15px 50px 15px 15px;
  position: relative;

  .el-autocomplete {
    width: 100%;
  }
}

.reverse {
  position: absolute;
  top: 35px;
  right: 15px;

  &:after,
  &:before {
    display: block;
    content: "";
    position: absolute;
    left: -35px;
    width: 25px;
    height: 1px;
    background: #ccc;
  }

  &:after {
    top: 0;
  }

  &:before {
    top: 60px;
  }

  span {
    display: block;
    position: absolute;
    top: 20px;
    right: 0;
    font-size: 12px;
    background: #999;
    color: #fff;
    width: 20px;
    height: 20px;
    line-height: 18px;
    text-align: center;
    border-radius: 2px;
    cursor: pointer;

    &:after,
    &:before {
      display: block;
      content: "";
      position: absolute;
      left: 10px;
      width: 1px;
      height: 20px;
      background: #ccc;
    }

    &:after {
      top: -20px;
    }

    &:before {
      top: 20px;
    }
  }
}
</style>
