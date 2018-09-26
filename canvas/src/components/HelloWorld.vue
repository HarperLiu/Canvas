<template>
  <el-container id="container">
  <el-header id="header">
  <div class="toolbar">
    <el-menu
      :default-active="'download'"
      class="el-menu-demo"
      mode="horizontal"
      @select="handleSelect"
      background-color="#545c64"
      text-color="#fff"
      active-text-color="#ffd04b">
      <el-menu-item index="draw" >开始绘图</el-menu-item>
      <el-menu-item index="finish" >完成并识别</el-menu-item>
      <el-submenu index="file">
        <template slot="title">文件</template>
        <el-menu-item index="download">下载</el-menu-item>
        <el-menu-item index="upload">上传</el-menu-item>
      </el-submenu>
      <el-menu-item index="clear">清空画板</el-menu-item>
    </el-menu>
  </div>
  </el-header>
    <el-main id="main">
      <el-card id="card">
        <el-input v-model="filename" placeholder="请输入文件名" style="width: 150px;height: 40px;margin-bottom: 10px"></el-input>
        <canvas id="myCanvas" v-model="canvas" width="1100" height="500" style="background: rgba(106,114,122,0.2)" @click="drawLine"></canvas>
      </el-card>
    </el-main>
    <input @change="fileChange($event)" type="file" id="upload_file" multiple style="display: none"/>
  </el-container>
</template>



<script>
  import ElContainer from 'element-ui/packages/container/src/main'
  import ElHeader from 'element-ui/packages/header/src/main'
  import ElMain from 'element-ui/packages/main/src/main'
  import FileSaver from 'file-saver'

export default {
  mounted(){
    this.drawLine()
  },
  watch: {
    imgList: {
      handler (oldValue, newValue) {
        this.draw()
      },
    }
  },
  components: {
    ElHeader,
    ElContainer,
    ElMain
  },
  name: 'HelloWorld',
  data () {
    return {
      counter:0, //记录一次绘画的笔画数
      isDrawing: false,
      imgList: [],
      shape:{
        pointList:[],
        label:"",
      }, //shape对象存储当前正在画的图形信息
      shapeList:[], //存储这张图的全部图形信息，数组内为shape对象
      filename:"",
    }
  },
  methods:{
    drawLine () {
        var canvas = document.getElementById('myCanvas')
        var ctx = canvas.getContext('2d')
        var _this = this
        ctx.lineWidth = 1
        ctx.strokeStyle = 'rgba(16,140,238,1)'
        var tempPointList = []
        canvas.onmousedown = function (ev) {
          if(_this.isDrawing) {
            var x = ev.offsetX
            var y = ev.offsetY
            tempPointList.push({x: x, y: y})
            ctx.moveTo(x, y)
          }
          canvas.onmousemove = function (ev) {
            if(_this.isDrawing) {
              var targetX = ev.offsetX
              var targetY = ev.offsetY
              tempPointList.push({x: targetX, y: targetY})
              ctx.lineTo(targetX, targetY)
              ctx.stroke()
            }
          }
          canvas.onmouseup = function (ev) {
            if(_this.isDrawing) {
              canvas.onmousemove = null
              canvas.onmouseup = null
              _this.counter++;
              _this.shape.pointList.push(tempPointList)
              tempPointList = []
            }
          }
        }
    },

    handleSelect (key) {
      switch (key) {
        case 'draw': {
          this.isDrawing = true;
          this.counter = 0;
          break
        }
        case 'finish': {
          //获取标注结果并把它显示在屏幕上
          var recognizeRes = this.recognize(this.counter);
          this.shape.label = recognizeRes;
          this.showResult(recognizeRes)
          this.writeLabel(recognizeRes)
          //把当前图形存入图形数组
          var shape = this.shape;
          this.shapeList.push(shape);
          this.shape = {
            pointList:[],
            label:"",
          }
          //清空数据
          this.counter = 0;
          this.isDrawing = false;
          break
        }
        case 'download': {
          this.downloadRes();
          break
        }
        case 'upload': {
          this.uploadImage();
          break
        }
        case 'clear': {
          this.clearCanvas();
          break
        }
      }
    },

    recognize(counter){
      switch(counter){
        case 1:{
          return "圆形";
          break;
        }
        case 2:{
          return "三角形";
          break;
        }
        case 3:{
          return "矩形";
          break;
        }
        case 4:{
          return "正方形";
          break;
        }
        default:{
          return "未知图形"
        }
      }
    },

    showResult(result) {
      const h = this.$createElement;
      this.$message({
        message: h('p', null, [
          h('span', null, '您所绘的图形是： '),
          h('i', { style: 'color: teal' },result)
        ])
      });
    },

    writeLabel(label){
      var canvas = document.getElementById("myCanvas");
      var ctx = canvas.getContext("2d");
      //设置字体样式
      ctx.font = "20px Microsoft Yahei";
      //设置字体填充颜色
      ctx.fillStyle = "#ffc653";
      //从第一个坐标附近开始绘制文字
      if(this.shape.pointList!==[]) {
        var textPosX = this.shape.pointList[0][0].x;
        var textPosY = this.shape.pointList[0][0].y;
        ctx.fillText(label, textPosX, textPosY);
      }
    },

    clearCanvas() {
      var canvas = document.getElementById("myCanvas");
      canvas.height = canvas.height;
    },

    downloadRes(){
      var FileSaver = require('file-saver');
      var canvas = document.getElementById("myCanvas");
      var date = new Date();
      var time = date.getTime();
      var filename = this.filename;
      if(this.filename.length==0){
        filename = time;
      }
      canvas.toBlob(function(blob) {
        FileSaver.saveAs(blob,filename +".png");
      });

      var blob = new Blob([this.formatShapeList()], {type: "text/plain;charset=utf-8"});
      FileSaver.saveAs(blob, filename +".txt");
      this.filename = ""
    },

    uploadImage(){
      this.imgList = [];
      this.fileClick();
    },

    fileClick() {
      document.getElementById('upload_file').click()
    },

    fileChange(el) {
      if (!el.target.files[0].size) return;
      this.fileList(el.target);
      el.target.value = ''
    },

    fileList(fileList) {
      let files = fileList.files;
      for (let i = 0; i < files.length; i++) {
        this.fileAdd(files[i]);
      }
    },

    fileAdd(file) {
      let reader = new FileReader();
      let image = new Image();
      let _this=this;
      reader.readAsDataURL(file);
      reader.onload = function () {
        file.src = this.result;
        image.onload = function () {
          let width = image.width;
          let height = image.height;
          file.width = width;
          file.height = height;
          _this.imgList.push({
            file
          });
          console.log(_this.imgList);
        };
        image.src = file.src;
      }
    },
    draw (){
      this.clearCanvas()
      var cvs = document.getElementById('myCanvas')
      // 创建image对象
      var imgObj = new Image()
      var img = this.imgList[0].file.src
      imgObj.src = img
      // 待图片加载完后，将其显示在canvas上
      imgObj.onload = function () {
        var ctx = cvs.getContext('2d')
        ctx.drawImage(this, 0, 0, 1100, 500)
      }
    },

    formatShapeList(){
      var data = "";
      for(var i=0;i<this.shapeList.length;i++){
        data = data +"形状：" +this.shapeList[i].label+"\n"
        data = data + "坐标："
        for(var j=0; j<this.shapeList[i].pointList.length;j++) {
          for(var k=0;k<this.shapeList[i].pointList[j].length;k++) {
            data = data + "("+this.shapeList[i].pointList[j][k].x+","+this.shapeList[i].pointList[j][k].y+")"
          }
          data = data + "\n";
        }
        data = data + "\n"
      }
      return data;
    },


  },
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
#container{
  position: absolute;
  width: 99%;
  height: 98%;
}
  #main{
  }
    #card{
      position: relative;
      width: 98%;
      height: 90%;
      left: 1%;
    }

</style>
