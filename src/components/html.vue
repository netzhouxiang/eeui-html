<template>
    <div>
      <div v-for="(child, index) in children" :key="index">
        <richtext v-if="child.tag==='div'" :class="child.class">
          <template v-for="(item,y) in child.texts">
            <span v-if="item.tag==='text'" class="text" :key="y" :style="item.style">{{item.text}}</span>
            <image v-if="item.tag==='image'" :key="y" @load="onImageLoad" :src="item.src" resize="stretch" 
            class="product-image" :style="{opacity:0,width: '2000px', height: '2000px'}"></image>
            <span v-if="item.tag==='br'" :key="y" class="br"></span>
          </template>
        </richtext>
      </div>
    </div>
</template>
<style>
.p{
  /* flex-direction: row;
  flex-wrap:wrap; */
}
.text{
  font-size: 28px;
}
.br {
  height: 20px;
}
.ql-align-center{
  justify-content: center;
}
.ql-align-right{
  justify-content: flex-end;
}
.ql-align-justify{
  justify-content: space-between;
}
</style>
<script>
  const eeui = app.requireModule('eeui');
  const modal = weex.requireModule('modal');
  const animation = weex.requireModule('animation');
  var {html2json} = require('../html2json/html2json');
  /*
    weex html解析
  */
  export default {
    props: {
      //内容
      content: {
        type: String,
        required: true
      },
      //图片最大宽度 自动等比
      imgWidth: {
        type: Number,
        required: true,
        default: 690
      }
    },
    mounted() {},
    computed: {
      children() {
        if(this.content){
          const json = html2json(this.content,this);
          return this.translateJSON(json)
        }
        return [];
      }
    },
    methods: {
      onImageLoad (event) {
        if (event.success) {
            eeui.getImageSize(event.target.attr.src, result=>{
              var _w=result.width;
              var _h=result.height;
              if(_w>this.imgWidth){
                var bi = this.imgWidth / _w;
                _h = result.height * bi;
                _w = this.imgWidth;
              }
              animation.transition(event.target,{
                styles:{
                  width: _w+"px",
                  height: _h+"px",
                  opacity:1
                },
                duration: 0,
                timingFunction:'ease',
                delay: 0,
                needLayout:true,
                }, function(){})
            })
          }
        },
        translateJSON(node) {
          const result = [];
          var _self=this;
          const translate = (_texts, node, styleList) => {
            var styleL = styleList;
            if(node.attr && node.attr.style){
              var css_arr = node.attr.style.split(";")
              css_arr.forEach(item=>{
                var css = item.split(":")
                var name=css[0];
                if(name=="background-color"){
                    name="backgroundColor";
                }
                styleL[name] = css[1];
              });
            }
            if (node.tag && node.tag === "img") {
              _texts.push({
                tag: "image",
                src: node.attr.src
              });
            } else if (node.tag && node.tag === "br") {
              _texts.push({
                tag: "br"
              });
            } else if (node.tag && node.tag === "u") {
              styleL['textDecoration'] = 'underline';
            } 
            else if (node.tag && node.tag === "em") {
              styleL['fontStyle'] = 'italic';
            } 
            else if (node.tag && node.tag === "s") {
              styleL['textDecoration'] = 'line-through';
            }  
            else if (node.tag && node.tag === "strong") {
              styleL.fontWeight = 'bold';
            } else if (!node.tag && node.node === "text") {
              const style = styleL
              _texts.push({
                tag: "text",
                style,
                text: node.text.replace(/&[a-z]{3,4};/g, "")
              });
            }
            if (node.child && node.child.length) {
              node.child.forEach(c => translate(_texts, c, styleL));
            }
          };
          //处理P标签
          node.child.forEach(item => {
            var _texts = [];
            if (item.child && item.child.length) {
              item.child.forEach(c => translate(_texts, c, {}));
            }
            var _class = ["p"];
            if(item.attr && item.attr.class){
              _class.push(item.attr.class);
            }
            result.push({
              tag: "div",
              class: _class,
              texts: _texts
            });
          });
          return result;
      }
    }
  }
</script>