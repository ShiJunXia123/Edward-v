# Edward-v

> 一个 vue 组件，用于裁剪并上传图片，支持自定义裁剪长宽比例，固定裁剪后上传图片像素，没有额外插件依赖的轻量级插件
> ![image](https://user-images.githubusercontent.com/26054388/113959964-4e1c0280-9856-11eb-8a7b-19f8709fa23c.png)



## 安装

```bash
# 首先安装插件
$ npm install edward-v

# run all tests
npm test
```

## 使用

```
<picture-cropper
    infoText="<div>
        <div>
          最多1个文件，最大2M，支持png,jpg,jpeg格式。推荐用透明底色PNG格式。查看上传规范请点击“上传规范说明”。
        </div>
      </div>"
    :cropBoxSize="{ width: 220, height: 330 }"
    :croppedBoxSize="{ width: 80, height: 80, borderRadius: '100%' }"
    :limitSize=2
    action="http://localhost:8080/#/"
    saveBtnInfo="保存头像"
    :handleSuccess="handleSuccess"
    :proportion="16/9"/>

```

###### action: 上传 url

###### infoText: 组件的描述字段

###### cropBoxSize: 被截图片框大小 默认{ width: 164, height: 210}

###### croppedBoxSize: 预览图大小 默认{ width: 54, height: 54}

###### clipDivSize: 截图矩形初始大小 默认{ width: 100, height: 100}

###### limitSize: 图片大小限制

###### fixedCropWidth: 无论截图框大小，自定义最终转化成图片的像素宽

###### fixedCropHeight 无论截图框大小，自定义最终转化成图片的像素高

###### saveBtnInfo 保存按钮文案

###### handleSuccess 保存成功后执行函数

###### proportion 截图框长宽比例默认是 1


