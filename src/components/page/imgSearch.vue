<template>
  <div @click="showSearchLi = false">
    <my-head></my-head>
    <mySpace></mySpace>
    <top-search v-if="topFixed"></top-search>
    <el-dialog :visible.sync="paperVisible" width="30%" :append-to-body="true">
      <div class="paper-list">
        <div class="left-title">
          试卷列表
          <div class="add-paper" title="点击创建试卷" @click="newPaper()">
            <i class="el-icon-plus"></i>
          </div>
        </div>
        <div class="left-main">
          <ul>
            <li class="paper-li"
                v-for="(value, index) in $store.state.paperList"
                @click="addQ(value.id)">
              {{value.title}}
              <div class="paper-li-set">
                <div class="paper-li-icon" title="添加到本试卷" @click="addQ(value.id)"><i class="el-icon-document"></i></div>
              </div>
            </li>
          </ul>
        </div>
      </div>
    </el-dialog>
    <el-dialog title="裁剪图片" :visible.sync="visible" width="60%" center :append-to-body="true">
      <vue-cropper ref='cropper'
                   :src="imageSrc"
                   :ready="cropImage"
                   :zoom="cropImage"
                   :cropmove="cropImage"
                   :autoCropArea = "0.99"
                   :background = 'false'
                   style="width:100%;height:100%;max-height: 60vh">
      </vue-cropper>
      <span slot="footer" class="dialog-footer">
        <el-button @click="rotateImage()">旋 转</el-button>
        <el-button type="primary" @click="sureCrop">确 定</el-button>
        <el-button @click="cancelCrop" type="warning">取 消</el-button>
      </span>
    </el-dialog>
    <el-dialog :visible.sync="imgVisible" width="70%" :append-to-body="true">
      <img style="max-height: 60vh; margin-left: 50%; transform: translateX(-50%)" :src="$store.state.cropImg">
    </el-dialog>
    <div class="query">{{nowQuery}}</div>
    <div class="home-main" :style="{minHeight: minHeight + 'px'}">
      <div class="main-middle">
        <div class="main-left">
          <div class="header-concern" @click.stop="showSearchLi = true">
            <div style="width: 100%; position: relative">
              <el-input v-model="$store.state.input_message"
                        @keyup.native.enter="searchMsg(0)"
                        @keyup.native="getEvent($event)"
                        @keydown.native.up="selectUp"
                        @keydown.native.down="selectDown"
                        placeholder="题干/知识点/中考试卷"></el-input>
              <div class="search-ul" v-if="showSearchLi">
                <ul>
                  <li class="search-li"
                      v-for="(value, index) in $store.state.myData"
                      :class="{selectback: index == nowLi}"
                      @mouseover="selectHover(index)"
                      @click="selectClick(index)"
                      :key="value">
                    {{value}}
                  </li>
                </ul>
              </div>
              <div v-if="!$store.state.input_message" class="crop-img">
                <img src="../../img/phone.png" width="28px" />
                <input class="crop-input" type="file" name="image" accept="image/*" @change="setImage"/>
              </div>
              <div v-if="$store.state.input_message" class="crop-img">
                <i class="el-icon-close" @click="$store.state.input_message = ''"></i>
              </div>
            </div>
            <div class="btn-primary search-btn" @click="searchMsg(0)">
              <i class="el-icon-search">搜题</i>
            </div>
            <!--<div class="btn-primary search-page" @click="searchMsg(0)">
              <i class="el-icon-document">组卷</i>
            </div>-->
          </div>
          <div class="block" v-if="$store.state.nowSub.length"
               v-loading="$store.state.history.loadingTwo"
               element-loading-text="拼命加载中"
               element-loading-spinner="el-icon-loading"
               element-loading-background="rgba(0, 0, 0, 0.1)"
          >
            <div>
              <img :src="$store.state.cropImg" @click="imgVisible = true" class="pre-img">
              <ul>
                <li class="ques" v-for="(value, index) in nowPageArr" :key="value.md5">
                  <div class="up">
                    <span class="TH">{{nowPage * 10 - 9 + index + '.  '}}</span>
                    <span class="QUE" v-html="value.question"></span>
                  </div>
                  <div class="low">
                    <div><el-button type="primary" size="mini" @click="showJX(index)">查看解析</el-button></div>
                    <div><el-button type="primary" @click="addPaper(value.md5)" size="mini">添加试题</el-button></div>
                    <div><el-button type="danger" size="mini" @click="similarSearch(value.md5)">相似推荐</el-button></div>
                  </div>
                </li>
              </ul>
              <el-pagination
                class="que-page"
                background
                layout="prev, pager, next"
                :page-size="10"
                :current-page.sync="nowPage"
                @current-change="nextPage"
                :total="$store.state.nowSub.length">
              </el-pagination>
            </div>
          </div>
        </div>
        <div class="main-right">
          <div class="right-fix">
            <div class="hot-search">热门搜索</div>
            <div class="hot-list">
              <ul>
                <li class="list" v-for="(value, index) in searchHot" @click="searchHotMsg(value)">
                  <img v-if="index < 3" src="../../img/fire.png" width="16px" style="transform: translateY(-3px);margin-right: 3px" alt="" />{{index + 1}}. {{value}}
                </li>
              </ul>
            </div>
          </div>
        </div>
      </div>
    </div>
    <answer></answer>
    <div v-show="noQue" :style="{minHeight: minHeight + 'px'}">
      <p style="position: absolute; top: 250px; left: 50%; transform: translateX(-50%)">暂没有与图片内容相关的题目</p>
    </div>
  </div>
</template>

<script>
  import mySpace from '../common/mySpace.vue'
  import VueCropper from 'vue-cropperjs'
  import myHead from '../common/header.vue'
  import answer from '../common/anwer.vue'
  import topSearch from '../common/SearchTop.vue'
  import lrz from 'lrz'
  export default {
    components: {
      VueCropper,
      myHead,
      answer,
      mySpace,
      topSearch
    },
    data () {
      return {
        nowUnique: '',
        noQue: false,
        paperVisible: false,
        imgVisible: false,
        imageSrc: '',
        visible: false,
        minHeight: 0,
        minHeights: 0,
        topFixed: false,
        nowPage: 1,
        screenShow: true,
        screenChoiceOne: [true, false, false, false],
        screenChoiceTwo: [true, false, false, false, false, false],
        searchHot: [
          '三角形辅助线做法',
          '二次函数综合题',
          '一元一次方程的应用',
          '圆专题',
          '二次函数的动点问题',
          '初中数学新定义问题',
          '中考压轴题',
          '一元一次方程的基础测试',
          '一元一次方程的典型例题',
          '全等三角形培优经典题目'
        ],
        nowLi: -1,
        showSearchLi: true
      }
    },
    methods: {
      setImage (e) {
        const that = this
        const file = e.target.files[0]
        if (!file.type.includes('image/')) {
          return
        }
        const reader = new FileReader()
        reader.onload = (event) => {
          e.target.value = ''
          lrz(event.target.result, { width: 1080 }).then(function (rst) {
            that.visible = true
            that.imageSrc = rst.base64
            that.$refs.cropper && that.$refs.cropper.replace(rst.base64)
          }).catch(function (err) {
            console.log('压缩失败' + err)
          }).always(function () {
          })
        }
        reader.readAsDataURL(file)
      },
      cropImage () {
        this.$store.state.cropImg = this.$refs.cropper.getCroppedCanvas().toDataURL()
      },
      rotateImage () {
        this.$refs.cropper.rotate(90)
      },
      cancelCrop () {
        this.visible = false
      },
      sureCrop () {
        this.visible = false
        let num = Math.random() * 10000
        this.$router.push({
          path: '/imgSearch',
          query: {msg: this.$store.state.cropImg, page: 1, num: num}
        })
      },
      getEvent (ev) {
        if (ev.keyCode === 38 || ev.keyCode === 40) {
          return
        }
        this.getWordList(this.$store.state.input_message)
        this.showSearchLi = true
      },
      selectUp () {
        this.nowLi --
        if (this.nowLi === -1) {
          this.nowLi = this.$store.state.myData.length - 1
        }
        this.$store.state.input_message = this.$store.state.myData[this.nowLi]
      },
      selectDown () {
        this.nowLi ++
        if (this.nowLi === this.$store.state.myData.length) {
          this.nowLi = 0
        }
        this.$store.state.input_message = this.$store.state.myData[this.nowLi]
      },
      selectHover (index) {
        this.nowLi = index
      },
      selectClick (index) {
        this.$store.state.input_message = this.$store.state.myData[index]
        this.searchMsg(1)
        this.$store.state.myData = []
      },
      searchMsg (m) {
        let num = Math.random() * 10000
        this.$router.push({
          path: '/index',
          query: {
            servlet: 'wordSearch',
            msg: this.$store.state.input_message,
            page: 1,
            kind: '全部',
            nianji: this.$store.state.nianji,
            jiaocai: this.$store.state.jiaocai,
            way: m,
            num: num
          }
        })
      },
      /* choiceOne (num) {
        for (let i = 0; i < this.screenChoiceOne.length; i++) {
          this.screenChoiceOne[i] = false
        }
        this.$set(this.screenChoiceOne, num, true)
      },
      choiceTwo (num) {
        for (let i = 0; i < this.screenChoiceTwo.length; i++) {
          this.screenChoiceTwo[i] = false
        }
        this.$set(this.screenChoiceTwo, num, true)
      }, */
      showJX (x) {
        this.$store.state.myTest[0].question = this.$store.state.nowSub[x].question
        this.$store.state.myTest[0].kddp = this.$store.state.nowSub[x].comment
        this.$store.state.myTest[0].knowledge = this.$store.state.nowSub[x].knowledge
        this.$store.state.myTest[0].answer = this.$store.state.nowSub[x].answer
        this.$store.state.myTest[0].analysis = this.$store.state.nowSub[x].analysis
        this.$store.state.IFJX = true
      },
      addPaper (x) {
        this.nowUnique = x
        let sessionId = sessionStorage.getItem('sessionId')
        if (sessionId) {
          let url = this.$store.state.urls.url + 'paper/getList'
          let formData = new FormData()
          formData.append('sessionId', sessionId)
          this.$axios.post(url, formData, {
            headers: {
              'Content-Type': 'application/json;charset=utf-8'
            },
            withCredentials: true
          }).then((response) => {
            console.log(response.data)
            this.$store.state.paperList = response.data.data
            this.paperVisible = true
          }, (response) => {
            this.$message.error('请求服务端失败')
          })
        } else {
          this.signShows()
        }
      },
      addQ (pid) {
        this.paperVisible = false
        let que = this.nowUnique
        this.addQue(pid, que)
      },
      newPaper () {
        this.$prompt('请输入试卷名字', '提示', {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          inputPattern: /\S/,
          inputErrorMessage: '不能为空',
          inputValue: this.examName
        }).then(({ value }) => {
          this.createPaper(value)
        }).catch(() => {
          this.$message({
            type: 'info',
            message: '取消输入'
          })
        })
      },
      nextPage (val) {
        document.body.scrollTop = 0
        document.documentElement.scrollTop = 0
      },
      handleScroll () {
        let scrollTop = window.pageYOffset || document.documentElement.scrollTop || document.body.scrollTop
        if (scrollTop >= 200) {
          this.topFixed = true
        } else {
          this.topFixed = false
        }
      }
    },
    mounted () {
      window.addEventListener('scroll', this.handleScroll)
    },
    computed: {
      nowQuery: function () {
        return this.$route.query
      },
      nowPageArr: function () {
        let x = this.nowPage
        let arr = []
        for (let i = 1; i <= 10; i++) {
          let num = x * 10 - 11 + i
          if (this.$store.state.nowSub[num]) {
            arr.push(this.$store.state.nowSub[num])
          }
        }
        return arr
      }
    },
    watch: {
      nowQuery: function (val) {
        if (this.$route.path === '/imgSearch') {
          this.$store.state.cropImg = val.msg
          this.imgSearch()
        }
      }
    },
    created () {
      this.minHeight = document.documentElement.clientHeight - 251
      if (this.$route.query) {
        this.$store.state.cropImg = this.$route.query.msg
        this.imgSearch()
      }
    }
  }
</script>

<style scoped>
  .home-main{
    width: 100%;
    position: relative;
    top: 50px;
    z-index: 9;
    background-color: #fbfbfb;
  }
  .main-middle{
    min-width: 1200px;
    display: flex;
    flex-direction: row;
    position: relative;
    margin-left: 50%;
    transform: translateX(-50%);
  }
  .main-left{
    width: 848px;
    margin: 20px 20px 20px 0;
  }
  .right-fix{
    width: 23%;
    height: 350px;
    background-color: #fff;
    border-radius: 5px;
    box-sizing: border-box;
    border: 1px solid #DCDFE6;
    position: fixed;
  }
  .paperList-li{
    height: 40px;
    cursor: pointer;
    line-height: 40px;
  }
  .paperList-li:hover {
    background-color: #F2F6FC;
  }
  .screen{
  }
  .screen-title{
    display: flex;
    flex-direction: row;
  }
  .screen-name{
    margin: 10px 3px 10px 0;
    cursor: pointer;
    color: #333;
  }
  .screen-name:hover{
    text-decoration: underline;
  }
  .screen-edition{
    margin: 10px 3px 10px 0;
    cursor: pointer;
    color: #409EFF;
    font-weight: bold;
  }
  .screen-menu{
    margin-top: 8px;
  }
  .screen-menu img{
    width: 32px;
    height: 25px;
  }
  .screen-window{
    border-radius: 5px;
    box-sizing: border-box;
    border: 1px solid #DCDFE6;
    padding: 10px;
    background-color: #fff;
  }
  .screen-list{
    margin: 10px 0 10px 20px;
    display: flex;
    flex-direction: row;
    color: #333;
  }
  .screen-ul{
    font-size: 14px;
    font-weight: 600;
    font-family: "PingFang SC";
    letter-spacing: 1px;
    margin-right: 20px;
  }
  .screen-li{
    font-size: 14px;
    letter-spacing: 1px;
    margin-right: 20px;
    border-radius: 3px;
    padding: 2px 3px;
    cursor: pointer;
  }
  .choice{
    background-color: #409EFF;
    color: #fff;
  }
  .hot-search{
    height: 35px;
    line-height: 35px;
    color: #333;
    font-size: 17px;
    font-family: 黑体;
    width: 90%;
    margin: 0 auto 5px;
    box-sizing: border-box;
    border-bottom: 1px solid #DCDFE6;
  }
  .list{
    letter-spacing: 1px;
    width: 88%;
    margin: 0 auto;
    color: #333;
    height: 30px;
    line-height: 30px;
    font-size: 14px;
    cursor: pointer;
  }
  .list:hover{
    text-decoration: underline;
    color: #409EFF;
  }
  .pre-img{
    max-width: 96%;
    max-height: 200px;
    border: 1px solid #eee;
    border-radius: 5px;
    box-sizing: border-box;
    margin-left: 50%;
    transform: translateX(-50%);
  }
  .header-concern {
    display: flex;
    flex-direction: row;
    position: relative;
  }
  .block{
  }
  .ques{
    position: relative;
    clear: both;
    margin: 10px 0;
    box-sizing: border-box;
    border-radius: 5px;
    background-color: #fff;
    border: 1px solid #DCDFE6;
    word-wrap: break-word;
    letter-spacing: 1px;
  }
  .up{
    line-height: 25px;
    font-size: 0.875rem;
    padding: 20px 20px 20px 20px;
  }
  .low{
    background-color: #f4f4f4;
    height: 40px;
    position: relative;
    bottom: 0;
    display: flex;
    justify-content: flex-end;
    line-height: 40px;
    box-sizing: border-box;
    border-top: 1px solid #dadada;
    padding: 0 20px;
    color: #666666;
    border-radius: 0 0 10px 10px;
    font-size: 0.8125rem;
  }
  .low div{
    margin-left: 15px;
    cursor: pointer;
  }
  .que-page{
    text-align: center;
    margin: 50px 0;
    font-size: 16px;
  }
  .topfix {
    position: fixed !important;
    top: 60px;
    z-index: 999;
    background-color: #fff;
  }
  .query {
    color:#F2F6FC;
    width: 100%;
    background-color: #F2F6FC;
    text-align: center;
    position: fixed;
    top: 50px;
    height: 0;
    opacity: 0;
  }
</style>
