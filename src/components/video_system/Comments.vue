<template>
  <div class="video-commit" v-loading="loading">
    <div style="width:99%">
      <el-divider></el-divider>


      <div style="display:flex;margin:0 20px;line-height:60px;height:60px;">
        <div>
          <el-avatar 
            style="height:60px;width:60px;"
          :src="myAvatar"></el-avatar>
        </div>
        <div style="margin-left:40px;    width: 80%;">
          <el-input show-word-limit maxlength="512" placeholder="请输入内容" v-model="commentContent" clearable></el-input>
        </div>
        <div style="margin-left:20px;">
          <el-button @click="comment(courseId,'comment')" type="primary">发表</el-button>
        </div>
      </div>
      
      <el-divider></el-divider>

      <el-tabs>
        <el-tab-pane :label="'热门评论（'+allReplies.length +'）'" name="first"></el-tab-pane>
      </el-tabs>

      <div style="text-align:left;margin-left:20px;">
        <div v-for="obj in comments" :key="obj.id">
          <div style="display:flex;height:40px;display:flex;justify-content:space-between">
            <div style="height:40px;display:flex;line-height: 40px;" >
              <el-avatar
                :src="QiniuyunUrl+obj.avatar"
              ></el-avatar>
               &nbsp;&nbsp;&nbsp;&nbsp;{{obj.nickName==null?obj.name:obj.nickName}}
           </div>
           
            <div   class="like"
             @click="liked(obj.id,'comment')">
             {{obj.likeNum}}</div>
            
          </div>
          <br />
          <div v-if="obj.repliedStatus=='ok'" style="margin-left:2em;font-size:18px;text-indent:2em">
             {{obj.content}}
          </div>
           <div v-if="obj.repliedStatus=='del'" style="color:grey;margin-left:2em;font-size:18px;text-indent:2em">
            <s>该评论被删除。</s>
          </div>

          <br />

          <div style="text-align:right;">
            <div style="padding:0px 10px;text-align:left;margin:0 auto;background:snow;width:80%;" v-for="(item,index) in sonReplies.replies" :key="index">
              <div v-if="sonReplies.parentId==obj.id">
              
              <div style="font-size:16px;display:flex;justify-content:space-between">
                <div >
                    <span style="color:dodgerblue">{{item.A.nickName?item.A.nickName:item.A.name}}</span>@
                    <span style="color:dodgerblue">{{item.B.nickName?item.B.nickName:item.B.name}}</span>
                    <span style="color:gainsboro"> {{"：“ "+item.B.content.substr(0,3)+"...”："}}</span>
               </div>
               <div >
                <div    class="like" @click="liked(item.A.id,'replied')">
                  {{item.A.likeNum}}</div>
              </div>

              
              </div>
               <div style="text-indent:2em;margin:5px auto;width:100%;background:whitesmoke">
               “ {{item.A.content}}  “
                <el-button size="mini" type="text" @click="showRepliedInputId=item.A.id">回复</el-button>
                </div>
                <el-input
                  style="width:500px;"
                  v-if="item.A.id==showRepliedInputId?true:false"
                  v-model="repliedContent"
                />
                <el-button
                  @click="comment(item.A.id,'replied')"
                  v-if="item.A.id==showRepliedInputId?true:false"
                >确定</el-button>
                <el-button
                  @click="showRepliedInputId=0"
                  v-if="item.A.id==showRepliedInputId?true:false"
                >取消</el-button>
              </div>


            </div>
            <br>
            <el-button v-if="obj.username==username" type="text" @click="delRepliedById(obj.id,obj)">删除</el-button>

            <el-button type="text" @click="diguiComments(obj.id,obj)">展开</el-button>

            <!-- <el-button type="mini"  @click="liked(obj.id,'comment')">赞 {{obj.likeNum}}</el-button> -->
           
          
            <el-button type="text"  @click="showRepliedInputId=obj.id">回复</el-button>
            <!-- {{obj.replyNum}} -->

          <br>

            <el-input
            maxlength="512"
              style="width:500px;"
              v-if="obj.id==showRepliedInputId?true:false"
              v-model="repliedContent"
            />
            <el-button
              @click="comment(obj.id,'replied')"
              v-if="obj.id==showRepliedInputId?true:false"
            >确定</el-button>
            <el-button @click="showRepliedInputId=0" v-if="obj.id==showRepliedInputId?true:false">
              取消</el-button>


          </div>

          <el-divider></el-divider>
        </div>
      </div>
    </div>
  </div>
</template>



<script>
import axios from "axios";
import UrlConfig from "../../config/UrlConfig.js";
export default {
  name: "VideoCourse",
  data() {
    return {
      username:JSON.parse(localStorage.getItem('userInfo')).account,
      loading:true,
      QiniuyunUrl: UrlConfig.getQiniuyunUrl(), //七牛云地址
      myAvatar: null,//我的头像
      courseId: null, //第一次进来课程id

      repliedContent: "",//回复内容
      showRepliedInputId: null,
      showSonReplies: true,

      course: null, //当前的课程
      commentContent: "", //输入框内容
      comments: [], //所有一级评论
      allReplies: [],//所有评论和回复
      sonReplies: { parentId: null, replies: [] }  //一条评论下的所有回复
    };
  },
  methods: {
  //递归出一条评论下的所有回复 
    diguiComments(id, obj) {
      //关闭
      if(this.sonReplies.parentId==id){
          this.sonReplies.parentId = false;
          return;
      }
      //继续
      if (id != null) {
        this.sonReplies.parentId = id;
        this.sonReplies.replies = [];
      }
      this.allReplies.forEach(element => {
        //避免死循环
        if (element.targetId == obj.id&&element.targetType!='comment') {
          this.diguiComments(null, element);
          this.sonReplies.replies.push({ A: element, B: obj });
        }
      });
    },

  //处理解析所有一级评论出来
    dealComments(comments) {
      var commentsJSON = []; //评论
      var repliesJSON = []; //回复
      comments.forEach(element => {
        if (element.targetType == "comment") {
          commentsJSON.push(element);
        } else {
          repliesJSON.push(element);
        }
      });
      this.comments = commentsJSON;
      console.log("评论获取完毕。")
      this.loading = false;

    },
    //获取所有评论和回复
    getAllComments(courseId) {
      var parmas = new URLSearchParams();
      parmas.append("courseId", courseId);
      var that = this;
      axios
        .post("/comm/getAllRepliedByCourseId", parmas)
        .then(function(response) {
          that.allReplies = response.data.object;
          that.dealComments(that.allReplies);
        });
    },

    delRepliedById(id,obj){
      var parmas = new URLSearchParams();
      parmas.append("id", id);
      var that = this;
      axios.post("/comm/delRepliedById", parmas).then(function(response) {
        if(response.data.object==1){
           that.$message({
          message: '删除成功。',
          type: 'success'
        });
        obj.repliedStatus='del'
        }
      });

    },
    //评论
    comment(targetId, targetType) {
      var  content = targetType == "comment" ? this.commentContent : this.repliedContent
      if( content==null||content==""){
         alert("内容不允许为空哦~")
         return ;
      }
      var parmas = new URLSearchParams();
      parmas.append("targetId", targetId);
      parmas.append("targetType", targetType);
      parmas.append("courseId", this.courseId);
      parmas.append(
        "content",
         content
      );
      var that = this;
      axios.post("/comm/addReplied", parmas).then(function(response) {
        console.log(response);
        that.$message({
          message: '发送成功',
          type: 'success'
        });
        that.getAllComments(that.courseId);
        that.commentContent = null;
        that.repliedContent = null;
        that.showRepliedInputId=0;
           that.sonReplies.parentId = false;
      });
    },

    liked(targetId, targetType) {
      console.log(targetId);
      console.log(targetType);
      var parmas = new URLSearchParams();
      parmas.append("targetId", targetId);
      parmas.append("targetType", targetType);
      var that = this;
      axios.post("/comm/addLiked", parmas).then(function(response) {
        console.log(response.data.object);
        if(!response.data.object){
        if (targetType == "course") that.course.likeNum++;
        if (targetType == "comment"||targetType == "replied") {
          for (var i = 0; i < that.allReplies.length; i++) {
            if (that.allReplies[i].id == targetId) {
              that.allReplies[i].likeNum++;
            }
          }
        }
        }
        else{
            that.$message('已经点过赞了哦');
        }

      });
    }
  },

  mounted() {
    this.courseId = this.$route.query.courseId;
    //根据courseId获取其所有评论回复
    this.getAllComments(this.courseId);
    this.myAvatar =
      UrlConfig.getQiniuyunUrl() +
      JSON.parse(localStorage.getItem("userInfo")).info.avatar;
  }
};
</script>

<style scoped>
.video-commit {
  min-height: 600px;
  margin: 0px auto;
  padding: 20px;
  /* background: white;
  width: 100%;
  box-shadow: 0 2px 12px 0 rgba(0, 0, 0, 0.1);
  border-radius: 5px; */
}


.like{
      display: inline-block;
      margin-left: 15px;
      padding-left: 30px;
      color: #E96565;
      line-height: 40px;
      cursor: pointer;
      font-size: 15px;
      background: url("../../assets/icons/like.png") no-repeat left center;
}
.like:hover{
      display: inline-block;
      margin-left: 15px;
      cursor: pointer;
      line-height: 40px;
      padding-left: 30px;
      color: #E96565;
      font-size: 15px;
      background: url("../../assets/icons/like-full.png") no-repeat left center;
}
</style>
