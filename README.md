# eeui-html
解析eeui html标签

当前支持 em u s strong p img标签；支持背景色字体颜色，对齐方向； 在组件内可自行扩展

实现思路：通过html2json转换接口返回的HTML为JSON，循环JSON处理想要的标签，转化为对应数据，进行循环绑定；  
参考链接：https://blog.csdn.net/codingandroid/article/details/77451389

# 使用方法：
1.下载压缩包,解压到eeui/src目录  
2.然后在项目引入后，调用即可，如下：  
```java
    import html from "../../components/html.vue";
    <html :content="v.content"></html>
```

# 如果是使用VUE开发后台，建议使用配合vue-quill-editor编辑器使用，配置:
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

# html.vue 组件内样式，分别对应vue-quill-editor编辑器 （toolbar：align） 的对齐方向，可自行调整：
.ql-align-center{
  justify-content: center;
}
.ql-align-right{
  justify-content: flex-end;
}
.ql-align-justify{
  justify-content: space-between;
}

