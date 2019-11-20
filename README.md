# eeui-html
eeui 解析html标签组件

当前支持 em u s strong p img标签；支持背景色字体颜色，对齐方向； 在组件内可自行无下限扩展

实现思路：通过html2json转换接口返回的HTML为JSON，循环JSON处理想要的标签，转化为对应数据，进行循环绑定；  
参考链接：https://blog.csdn.net/codingandroid/article/details/77451389

# 使用方法：
1.下载压缩包,解压后，复制src里文件夹到eeui/src目录  
2.然后在项目引入后，调用即可，如下：  
```javascript
    import html from "../../components/html.vue";
    <html :content="v.content"></html>
```

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
# html.vue 组件内样式，分别对应vue-quill-editor编辑器 （toolbar：align） 的对齐方向，可自行调整：
```css
.ql-align-center{
  justify-content: center;
}
.ql-align-right{
  justify-content: flex-end;
}
.ql-align-justify{
  justify-content: space-between;
}
```

# vue-quill-editor.vue  
是通过element-ui vue-quill-editor 插件编辑文章的示例文件  
使用vue-quill-editor，可以参考一下;

# 效果
  
后台：  
![image](https://raw.githubusercontent.com/netzhouxiang/eeui-html/master/1.jpg)  
  
eeui app：  
![image](https://raw.githubusercontent.com/netzhouxiang/eeui-html/master/2.jpg)  

