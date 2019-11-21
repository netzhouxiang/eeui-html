# eeui-html
eeui 解析html标签组件

1.支持 br div span em u s strong p img等常用标签；  
2.支持背景色字体颜色，对齐方向等；  
3.支持图片自适应；  
4.支持当下手机页面基本格式展示； 

目前组件不可能做到100%完全适用，所以有需要调整的地方，自行修改即可； 

适用于weex所有项目，更改获取图片宽高方法即可；  

# 使用方法：
1.下载压缩包,解压后，复制src里文件夹到eeui/src目录  
2.然后在项目引入后，调用即可，如下：  
```javascript
    import html from "../../components/html.vue";
    components: { html },
    <html :content="v.content"></html>
```

# html.vue 组件内样式，分别对应vue-quill-editor编辑器 （toolbar：align） 的对齐方向，可自行调整：
```css
.ql-align-center{
  text-align: center;
}
.ql-align-right{
  text-align: right;
}
.ql-align-justify{
  text-align: justify;
}
```

# 效果
  
后台：  
![image](https://raw.githubusercontent.com/netzhouxiang/eeui-html/master/1.jpg)  
  
eeui app：  
![image](https://raw.githubusercontent.com/netzhouxiang/eeui-html/master/2.jpg)  




# 如果是使用VUE开发后台，建议使用配合vue-quill-editor编辑器使用，配置:

```javascript
    editorOption: {
      modules: {
        toolbar: [
          ["bold", "italic", "underline", "strike"],
          [{ color: [] }, { background: [] }],
          [{ align: [] }],
          ["image"],
          ["clean"]
        ]
      }
    },
```

# vue-quill-editor.vue  
是通过element-ui vue-quill-editor 插件编辑文章的示例文件   
使用vue-quill-editor，可以参考一下;   
