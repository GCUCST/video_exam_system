<template>
  <div class="personal-center">
       <el-tabs tab-position="up">
    <el-tab-pane label="我的信息">
      <v-MyInfo></v-MyInfo>
    </el-tab-pane>
    <el-tab-pane label="我的头像">
      <v-MyAvatar></v-MyAvatar>  
     </el-tab-pane>
    <el-tab-pane label="账户安全">
      <v-AccountSafe></v-AccountSafe>
    </el-tab-pane>
  </el-tabs>
  </div>
</template>


<script>
import MyInfo from "@/components/public/personal_center/MyInfo.vue"
import AccountSafe from "@/components/public/personal_center/AccountSafe.vue"

import MyAvatar from "@/components/public/personal_center/MyAvatar.vue"
export default {
  name: "PersonalCenter",
  data() {
    return {
      msg: "个人中心"
   
    };
  },
  components:{
    "v-MyInfo":MyInfo,
    "v-MyAvatar":MyAvatar,
    "v-AccountSafe":AccountSafe
  }
};
</script>

<style scoped>
.personal-center{
  margin: 20px auto;
  padding: 30px;
  background: white;
  width: 95%;
  box-shadow: 0 2px 12px 0 rgba(0, 0, 0, 0.1);
  border-radius: 5px;
}
</style>
