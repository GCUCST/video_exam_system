<template>
  <div class="footer">
      <div style="width:100%;height:100%;font-size:14px">版权所有@华广学习系统</div>

  </div>
</template>

<script>
export default {
  name: "Footer",
  data() {
    return {
      msg: "Footer" //没用
    };
  },
  methods: {}
};
</script>
<style scoped>
.footer {
  height: 30px;
  text-align: center;
  margin: 20px auto;
  width: 95%;
  padding: 30px;
  background: white;
 box-shadow: 0 2px 12px 0 rgba(0, 0, 0, 0.1);
  border-radius: 5px;
  background-color: white;
}
</style>
