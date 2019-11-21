<template>
    <div>
      <div v-if="reload" v-for="(child, index) in labellist" :key="index">
        <richtext v-if="child.tag==='div'" :class="child.class" :style="child.style">
          <template v-for="(item,y) in child.texts">
            <span v-if="item.tag==='text'" class="text" :key="y" :style="item.style">{{item.text}}</span>
            <image v-if="item.tag==='image'" :key="y" :src="item.src" resize="stretch" :style="{width: item.width, height: item.height}"></image>
            <image v-if="item.tag==='br'" src="data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAAGXRFWHRTb2Z0d2FyZQBBZG9iZSBJbWFnZVJlYWR5ccllPAAAAyJpVFh0WE1MOmNvbS5hZG9iZS54bXAAAAAAADw/eHBhY2tldCBiZWdpbj0i77u/IiBpZD0iVzVNME1wQ2VoaUh6cmVTek5UY3prYzlkIj8+IDx4OnhtcG1ldGEgeG1sbnM6eD0iYWRvYmU6bnM6bWV0YS8iIHg6eG1wdGs9IkFkb2JlIFhNUCBDb3JlIDUuMy1jMDExIDY2LjE0NTY2MSwgMjAxMi8wMi8wNi0xNDo1NjoyNyAgICAgICAgIj4gPHJkZjpSREYgeG1sbnM6cmRmPSJodHRwOi8vd3d3LnczLm9yZy8xOTk5LzAyLzIyLXJkZi1zeW50YXgtbnMjIj4gPHJkZjpEZXNjcmlwdGlvbiByZGY6YWJvdXQ9IiIgeG1sbnM6eG1wPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvIiB4bWxuczp4bXBNTT0iaHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wL21tLyIgeG1sbnM6c3RSZWY9Imh0dHA6Ly9ucy5hZG9iZS5jb20veGFwLzEuMC9zVHlwZS9SZXNvdXJjZVJlZiMiIHhtcDpDcmVhdG9yVG9vbD0iQWRvYmUgUGhvdG9zaG9wIENTNiAoV2luZG93cykiIHhtcE1NOkluc3RhbmNlSUQ9InhtcC5paWQ6RjRFQThBREQwQzA2MTFFQTlBQ0RCRkQ3NzJGNzU3MkIiIHhtcE1NOkRvY3VtZW50SUQ9InhtcC5kaWQ6RjRFQThBREUwQzA2MTFFQTlBQ0RCRkQ3NzJGNzU3MkIiPiA8eG1wTU06RGVyaXZlZEZyb20gc3RSZWY6aW5zdGFuY2VJRD0ieG1wLmlpZDpGNEVBOEFEQjBDMDYxMUVBOUFDREJGRDc3MkY3NTcyQiIgc3RSZWY6ZG9jdW1lbnRJRD0ieG1wLmRpZDpGNEVBOEFEQzBDMDYxMUVBOUFDREJGRDc3MkY3NTcyQiIvPiA8L3JkZjpEZXNjcmlwdGlvbj4gPC9yZGY6UkRGPiA8L3g6eG1wbWV0YT4gPD94cGFja2V0IGVuZD0iciI/PvzSZqoAAAAQSURBVHjaYvz//z8DQIABAAkBAv9pvJDwAAAAAElFTkSuQmCC" :key="y" class="br"></image>
          </template>
        </richtext>
      </div>
    </div>
</template>
<style>
/*行样式，需要什么自行添加*/
.div{
  /* margin-bottom: 50px; */
}
/*文字大小*/
.text{
  font-size: 28px;
}
/*richtext不支持换行，改用image标签处理*/
.br {
  height: 20px;
  width: 690px;
}
/*编辑器 对齐样式*/
.ql-align-center{
  text-align: center;
}
.ql-align-right{
  text-align: right;
}
.ql-align-justify{
  text-align: justify;
}
</style>
<script>
  const eeui = app.requireModule('eeui');
  var {html2json} = require('../html2json/html2json');
  /*
    eeui html解析 (weex)
  */
  export default {
    props: {
      //内容
      content: {
        type: String,
        required: true
      },
      //图片最大宽度 自动等比 px
      imgWidth: {
        type: Number,
        required: true,
        default: 690
      }
    },
    data() {
        return {
          //是否显示，用于销毁组件
          reload:true,
          //标签结构
          labellist: [],
        };
    },
    watch:{
      //获取数据进行解析绑定，获取原图片大小处理
      content(){
        if(this.content){
          const json = html2json(this.content);
          //循环处理结构和图片，因为异步获取，无法判定什么时候完成，则先加载结构
          this.translateJSON(json);
          //后处理图片，监听更新界面
          this.labellist.forEach((item,x)=>{
            item.texts.forEach((image,y)=>{
              if(image.tag == "image"){
                this.getImageLoad(image,x,y);
              }
            })
          });
        }
      },
      //如果监听到值改变，则销毁数据重新绑定
      labellist(){
        this.reload = false;
        //weex不支持$nexttick 先延时搞一搞
        setTimeout(()=>{
          this.reload = true;
        },10)
      }
    },
    methods: {
      /*
      * 图片自适应
      */
      getImageLoad (image,x,y) {
            if(image.width){
              return;
            }
            eeui.getImageSize(image.src, result=>{
              var _w=result.width;
              var _h=result.height;
              if(_w>this.imgWidth){
                var bi = this.imgWidth / _w;
                _h = result.height * bi;
                _w = this.imgWidth;
              }
              this.labellist[x].texts[y].width = _w +'px';
              this.labellist[x].texts[y].height= _h +'px';
              this.labellist = JSON.parse(JSON.stringify(this.labellist));
            });
        },
        /*
        * html解析，可自行扩展
        */
        translateJSON(node) {
          this.labellist =  [];
          var _self=this;
          const translate = (_texts, node, styleList) => {
            //继承上级样式
            var styleL = styleList;
            //处理样式
            if(node.attr && node.attr.style){
              //循环处理
              var css_arr = node.attr.style.split(";")
              css_arr.forEach(item=>{
                var css = item.split(":")
                var name=css[0];
                //更换css名为 richtext 下子节点支持的名字
                if(name=="background-color"){
                    name="backgroundColor";
                }
                styleL[name] = css[1];
              });
            }
            //处理img标签
            if (node.tag && node.tag === "img") {
              _texts.push({
                tag: "image",
                width:(node.attr.width?node.attr.width+'px':0),
                height:(node.attr.height?node.attr.height+'px':0),
                src: node.attr.src
              });
            } else if (node.tag && node.tag === "br") {
              //处理br标签
              _texts.push({
                tag: "br"
              });
            } else if (node.tag && node.tag === "u") {
              //处理下划线
              styleL['textDecoration'] = 'underline';
            } 
            else if (node.tag && node.tag === "em") {
              //处理斜体
              styleL['fontStyle'] = 'italic';
            } 
            else if (node.tag && node.tag === "s") {
              //处理中间线
              styleL['textDecoration'] = 'line-through';
            }  
            else if (node.tag && node.tag === "strong") {
              //处理加粗
              styleL.fontWeight = 'bold';
            } else if (!node.tag && node.node === "text") {
              //处理所有文本
              const style = styleL
              _texts.push({
                tag: "text",
                style,
                text: node.text.replace(/&[a-z]{3,4};/g, "")
              });
            }
            //循环处理
            if (node.child && node.child.length) {
              node.child.forEach(c => translate(_texts, c, styleL));
            }
          };
          //处理一级标签
          node.child.forEach(item => {
            var _texts = [];
            if (item.child && item.child.length) {
              item.child.forEach(c => translate(_texts, c, {}));
            }
            //默认加载div样式
            var _class = ["div"];
            if(item.attr && item.attr.class){
              _class.push(item.attr.class);
            }
            var styleL = {};
            //处理样式
            if(item.attr && item.attr.style){
              //循环处理
              var css_arr = item.attr.style.split(";")
              css_arr.forEach(item=>{
                var css = item.split(":")
                var name=css[0];
                //更换css名为 richtext 下子节点支持的名字
                if(name=="background-color"){
                    name="backgroundColor";
                }
                styleL[name] = css[1];
              });
            }
            _self.labellist.push({
              tag: "div",
              style: styleL,
              class: _class,
              texts: _texts
            });
          });
      }
    }
  }
</script>