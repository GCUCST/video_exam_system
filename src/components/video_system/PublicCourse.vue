<template>
  <div class="video-course" >
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/' }">主页</el-breadcrumb-item>
      <el-breadcrumb-item>{{course?course.title:'加载中...'}}</el-breadcrumb-item>
      <el-breadcrumb-item>{{getVideoTitleByVideoId(videoId)}}</el-breadcrumb-item>
    </el-breadcrumb>
    <div style="width:99%">
      <div>
        <div
          style="width: 10%;
    text-align: center;
    position: absolute;
    right: 50px;
    top: 28px;"
        >
          <el-avatar :src="course?QiniuyunUrl+course.cover:''"></el-avatar>
        </div>
      </div>

      <el-divider></el-divider>

      <div style="width:100%" v-loading="loading_course">
        <div style="width:100%;display:flex; justify-content:space-between">
          <div style="width:100%">
            <div style="width:100%;text-align:center">
              <video
                :class="[video_full?'video-big':'video-small']"
                controls="controls"
                id="qnv"
                :src="videoSrc"
                type="video/mp4"
              >您的浏览器不支持播放该视频！</video>

              <br />
              <div
                style="display: flex;justify-content: space-between;margin: 0 auto;"
                :style="{'width': (video_full ? '100%':'85%')}"
              >
                <span>
                  <div
                    class="like"
                    @click="liked(course.courseId,'course')"
                  > 
                  {{course?course.likeNum:0}}
                  </div>
                </span>
                <el-button @click="video_full=!video_full" circle icon="el-icon-full-screen" ></el-button>
              </div>
            </div>
          </div>

          <transition name="custom-classes-transition" enter-active-class="animated bounceInRight">
            <div style="width:35%" v-show="!video_full">
              <el-tabs style="height:100%" type="border-card" stretch>
                <el-tab-pane  style="height: 470px;overflow:auto"  label="章节">
                  <el-tree
                    node-key="id"
                    default-expand-all
                    :data="course?JSON.parse(course.catalogData):null"
                    :props="defaultProps"
                    @node-click="handleNodeClick"
                  ></el-tree>
                </el-tab-pane>
              </el-tabs>
            </div>
          </transition>
        </div>
      </div>
      <v-Comments></v-Comments>
    </div>
  </div>
</template>



<script>
import axios from "axios";
import Comments from "./Comments";
import UrlConfig from "../../config/UrlConfig.js";
export default {
  name: "VideoCourse",
  components: {
    "v-Comments": Comments
  },
  data() {
    return {
      loading_course:true,
      QiniuyunUrl: UrlConfig.getQiniuyunUrl(), //七牛云地址
      courseId: null, //第一次进来课程id
      videoId: null, //第一次进来当前视频id
      videoSrc: null, //视频地址
      video_full: false, //全屏模式
      course: null, //当前的课程
      defaultProps: {
        children: "children",
        label: "label"
      }, //目录菜单
      catalogData: null, //当前目录数据
      minCatalog: []
    };
  },
  methods: {
    //点赞
    liked(targetId, targetType) {
      var parmas = new URLSearchParams();
      parmas.append("targetId", targetId);
      parmas.append("targetType", targetType);
      var that = this;
      axios.post("/comm/addLiked", parmas).then(function(response) {
        console.log(response.data.object);
        if(!response.data.object){
        if (targetType == "course") that.course.likeNum++;
        if (targetType == "comment") {
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
    },

    //视频跳转
    jumpVideo(videoId) {
      this.$router.push({
        name: "PublicCourse",
        query: { courseId: this.courseId, videoId: videoId }
      });
      // location.reload();
      this.findVideoSrc(this.catalogData,videoId)
    },

     findVideoSrc(data,videoId) {
       var that = this
      for (var i = 0; i < data.length; i++) {
        if (data[i].level == 2 || data[i].level == 1) {
          this.findVideoSrc(data[i].children,videoId);
        } else if (data[i].level == 3) {
          if(videoId == data[i].id)
          {
            console.log("太难了吧")
            that.videoSrc = that.QiniuyunUrl+data[i].url;
            that.videoId =videoId;
          }
        }
      }
    },

    //获取小标题
    getVideoTitleByVideoId(videoId) {
      var title = null;
      this.minCatalog.forEach(element => {
        if (element.videoId == videoId) {
          title = element.label;
        }
      });
      return title;
    },

    //第一次进来，获取目录中所有的视频
    findVideoIdUrl(data) {
      for (var i = 0; i < data.length; i++) {
        if (data[i].level == 2 || data[i].level == 1) {
          this.findVideoIdUrl(data[i].children);
        } else if (data[i].level == 3) {
          this.minCatalog.push({
            label: data[i].label,
            videoId: data[i].id,
            url: data[i].url
          });
        }
      }
    },

    setNowVideoSrc(videoId) {
      if (!videoId) {
        this.$router.push({
          name: "PublicCourse",
              query: {
                courseId: this.courseId,
                videoId: this.minCatalog[0].videoId
              }
            });
       this.videoId = this.minCatalog[0].videoId
       this.videoSrc = this.QiniuyunUrl + this.minCatalog[0].url;
      }
      else{
         this.findVideoSrc(this.catalogData,videoId)
      }

    },

    //点击了目录菜单
    handleNodeClick(data) {
      if (data.level == 3) {
        this.jumpVideo(data.id);
      }
    },

    //请求课程资源和目录
    reqCourseRes(courseId, videoId) {

      var parmas = new URLSearchParams();
      parmas.append("courseId", courseId);
      var that = this;
      axios
        .post("/comm/getCourseByCourseId", parmas)
        .then(function(response) {
          //设置课程
          that.course = response.data.object;
          console.log("获取课程资源完成。");
          //设置目录
          that.catalogData = JSON.parse(response.data.object.catalogData);
          console.log("获取课程目录完成。");
          //取出所有视频
          that.findVideoIdUrl(that.catalogData);
          that.loading_course = false;
          that.setNowVideoSrc(that.videoId);
        })
        .catch(function(error) {
          console.log("获取课程失败。");
        });
        
    }
  },

  mounted() {
    this.courseId = this.$route.query.courseId;
    this.videoId = this.$route.query.videoId;
    this.reqCourseRes(this.courseId);
  }
};
</script>

<style scoped>
.video-course {
  min-height: 600px;
  margin: 0px auto;
  padding: 20px;
  background: white;
  width: 100%;
  box-shadow: 0 2px 12px 0 rgba(0, 0, 0, 0.1);
  border-radius: 5px;
}
.video-small {
  width: 85%;
  text-align: center;
  object-fit: fill;
  height: 547px;
}
.video-big {
  margin: 0 auto;
  width: 100%;
  height: 750px;
  text-align: center;
  object-fit: fill;
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

video {
  transition: all 300ms ease-in-out 50ms;
  -o-transition: all 300ms ease-out 50ms; /*兼容parsto内核*/
  -moz-transition: all 300ms ease-in-out 50ms; /*兼容gecko内核*/
  -webkit-transition: all 300ms ease-in-out 50ms; /*兼容webkit内核*/
}
</style>
