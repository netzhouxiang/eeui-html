<template>
  <div>
    <div v-if="reload" v-for="(child, index) in labellist" :key="index">
      <richtext v-if="child.tag==='div'" :class="child.class" :style="child.style">
        <template v-for="(item,y) in child.texts">
          <span v-if="item.tag==='text'" class="text" :key="y" :style="item.style">{{item.text}}</span>
          <image v-if="item.tag==='image'" :key="y" :src="item.src" resize="stretch"
            :style="{width: item.width, height: item.height}"></image>
          <image v-if="item.tag==='br'" src="data:image/png;" :key="y" class="br"></image>
        </template>
      </richtext>
    </div>
  </div>
</template>
<style>
  /*行样式，需要什么自行添加*/
  .div {
    /* margin-bottom: 30px; */
  }

  /*文字大小*/
  .text {
    font-size: 28px;
  }

  /*richtext不支持换行，改用image标签处理*/
  .br {
    height: 20px;
    width: 690px;
  }

  /*编辑器 对齐样式*/
  .ql-align-center {
    text-align: center;
  }

  .ql-align-right {
    text-align: right;
  }

  .ql-align-justify {
    text-align: justify;
  }
</style>
<script>
  const eeui = app.requireModule('eeui');
  var { html2json } = require('../html2json/html2json');
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
        imgNum: 0,
        showImgNum: 0,
        reload: false,
        labellist: [],
      };
    },
    watch: {
      //获取数据进行解析绑定，获取原图片大小处理
      content() {
        if (this.content) {
          this.reload = false;
          this.labellist = [];
          this.imgNum = 0;
          const json = html2json(this.content);
          this.translateJSON(json);
          this.showImgNum = 0;
          this.labellist.forEach((item, x) => {
            item.texts.forEach((image, y) => {
              if (image.tag == "image") {
                this.getImageLoad(image, x, y);
              }
            })
          });
        }
      },
      //如果监听到值改变
      labellist() {
        //表示没有需要处理的图片，直接展示HTML
        if (this.imgNum == 0) {
          this.reload = true;
          return;
        }
        //表示有需要处理的图片，判定是否处理完毕
        if (this.showImgNum > 0 && this.showImgNum == this.imgNum) {
          this.reload = true;
        }
      }
    },
    methods: {
      /*
      * 图片自适应
      */
      getImageLoad(image, x, y) {
        if (image.width && image.height) {
          return;
        }
        eeui.getImageSize(image.src, result => {
          this.showImgNum++;
          if (result.status === 'success') {
            var _w = result.width;
            var _h = result.height;
            if (_w > this.imgWidth) {
              var bi = this.imgWidth / _w;
              _h = result.height * bi;
              _w = this.imgWidth;
            }
            if(this.labellist[x] && this.labellist[x].texts[y]){
              this.labellist[x].texts[y].width = _w + 'px';
              this.labellist[x].texts[y].height = _h + 'px';
              this.labellist = JSON.parse(JSON.stringify(this.labellist));
            }
          }
        });
      },
      /*
      * css处理
      */
      translateCSS(node, styleL) {
        if (node.attr && node.attr.style) {
          //循环处理
          var css_arr = node.attr.style.split(";")
          css_arr.forEach(item => {
            var css = item.split(":")
            var name = css[0];
            //更换css名为 richtext 下子节点支持的名字；如果是其他 比如：font-size需要支持，则需要更改为fontSize
            if (name == "background-color") {
              name = "backgroundColor";
            }
            if (name == "font-style") {
              name = "fontStyle";
            }
            if (name == "text-decoration") {
              name = "textDecoration";
            }
            if (name == "font-weight") {
              name = "fontWeight";
            }
            if (name == "line-height") {
              name = "lineHeight";
            }
            if (name == "font-size") {
              name = "fontSize";
            }
            styleL[name] = css[1];
          });
        }
      },
      /*
      * html解析，可自行扩展
      */
      translateJSON(node) {
        var _labellist = [];
        const translate = (_texts, node, styleList) => {
          //继承上级样式
          var styleL = styleList;
          this.translateCSS(node, styleL);
          //处理img标签
          if (node.tag && node.tag === "img") {
            if (!node.attr.width || !node.attr.height) {
              this.imgNum++;
            }
            _texts.push({
              tag: "image",
              width: (node.attr.width ? node.attr.width + 'px' : 0),
              height: (node.attr.height ? node.attr.height + 'px' : 0),
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
          if (item.attr && item.attr.class) {
            _class.push(item.attr.class);
          }
          var styleL = {};
          this.translateCSS(item, styleL);
          _labellist.push({
            tag: "div",
            style: styleL,
            class: _class,
            texts: _texts
          });
        });
        this.labellist = _labellist;
      }
    }
  }
</script>