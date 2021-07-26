---
layout: post
title:  "MATLAB 制作抖音同款含褶皱面料图"
author: slandarer
categories: [ MATLAB, 正交叠底, 扭曲置换, 抖音]
image: https://img-blog.csdnimg.cn/20210414212025565.jpg
image_external: true
featured: false
hidden: false
---

抖音同款第二弹


效果如下：！！！
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210414212025565.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3NsYW5kYXJlcg==,size_16,color_FFFFFF,t_70#pic_center)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210414212035358.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3NsYW5kYXJlcg==,size_16,color_FFFFFF,t_70#pic_center)
#### 步骤
##### 1.导入图片
我们需要导入一张褶皱图片(background.jpg)以及一张前景图片(foreground.jpg),将褶皱图片灰度化，将前景图调整至与褶皱图片相同大小：

```java
bkgPic=imread('background.jpg');
bkgPic=rgb2gray(bkgPic);
forePic=imread('foreground.jpg');
forePic=imresize(forePic,size(bkgPic));
```
原图在这里：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210414212331279.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3NsYW5kYXJlcg==,size_16,color_FFFFFF,t_70#pic_center)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210414212343537.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3NsYW5kYXJlcg==,size_16,color_FFFFFF,t_70#pic_center)

##### 2.图片扩张
因为我们要对前景图片进行拉伸，难免边角处缺一块，因此我们首先将边缘处颜色往外扩展几圈(13圈)

```java
exforePic=uint8(zeros(size(forePic)+[26,26,0]));
exforePic(14:end-13,14:end-13,1)=forePic(:,:,1);
exforePic(14:end-13,14:end-13,2)=forePic(:,:,2);
exforePic(14:end-13,14:end-13,3)=forePic(:,:,3);

for i=1:13
    exforePic(i,14:end-13,:)=forePic(1,:,:);
    exforePic(end+1-i,14:end-13,:)=forePic(end,:,:);
    exforePic(14:end-13,i,:)=forePic(:,1,:);
    exforePic(14:end-13,end+1-i,:)=forePic(:,end,:);
end
for i=1:3
    exforePic(1:13,1:13,i)=forePic(1,1,i);
    exforePic(end-13:end,end-13:end,i)=forePic(end,end,i);
    exforePic(end-13:end,1:13,i)=forePic(end,1,i);
    exforePic(1:13,end-13:end,i)=forePic(1,end,i);
end
```

扩展后图片(图片下侧明显一点)：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210414212724334.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3NsYW5kYXJlcg==,size_16,color_FFFFFF,t_70#pic_center)
##### 3.像素映射
原理借鉴ps扭曲置换的原理，亮度较大的像素(大于128)取右下角像素RGB值进行置换，亮度较小的像素(小于128)取左上角像素RGB值进行置换，由于
(255-128)/10=12.7
(0-128)/10=-12.8
各个像素点与替换像素点的距离不超过13，因此上一步共扩展了13圈。
同时因为各个像素分布为整数点位置，而位置差计算一般都不是整数，因此我们要对偏移距离向上向下取整，获得两个像素点RGB值，并对这两点数值进行线性插值即可

```java
newforePic=uint8(zeros(size(forePic)));
for i=1:size(bkgPic,1)
    for j=1:size(bkgPic,2)
        goffset=(double(bkgPic(i,j))-128)/10;
        offsetLim1=floor(goffset)+13;
        offsetLim2=ceil(goffset)+13;
        sep1=goffset-floor(goffset);
        sep2=ceil(goffset)-goffset;
        c1=double(exforePic(i+offsetLim1,j+offsetLim1,:));
        c2=double(exforePic(i+offsetLim2,j+offsetLim2,:));
        if sep1==0
            c=double(exforePic(i+offsetLim1,j+offsetLim1,:));
        else
            c=c2.*sep1+c1.*sep2;
        end
        newforePic(i,j,:)=c;
    end
end
```
像素值映射结果：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210414214048995.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3NsYW5kYXJlcg==,size_16,color_FFFFFF,t_70#pic_center)
##### 4.正片叠底
将两张图片叠加起来
公式:混合色×基色 / 255=结果色
由于正片叠底后所出图片较暗，这里我们选择除以220而不是255：

```java
newforePic=uint8((double(newforePic).*double(bkgPic))./220);
imwrite(newforePic,'result.jpg')
imshow(newforePic)
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/2021041421492222.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3NsYW5kYXJlcg==,size_16,color_FFFFFF,t_70#pic_center)

##### 5.完整代码

```java
function clothFold
bkgPic=imread('background.jpg');
bkgPic=rgb2gray(bkgPic);
forePic=imread('foreground.jpg');
forePic=imresize(forePic,size(bkgPic));

exforePic=uint8(zeros(size(forePic)+[26,26,0]));
exforePic(14:end-13,14:end-13,1)=forePic(:,:,1);
exforePic(14:end-13,14:end-13,2)=forePic(:,:,2);
exforePic(14:end-13,14:end-13,3)=forePic(:,:,3);

for i=1:13
    exforePic(i,14:end-13,:)=forePic(1,:,:);
    exforePic(end+1-i,14:end-13,:)=forePic(end,:,:);
    exforePic(14:end-13,i,:)=forePic(:,1,:);
    exforePic(14:end-13,end+1-i,:)=forePic(:,end,:);
end
for i=1:3
    exforePic(1:13,1:13,i)=forePic(1,1,i);
    exforePic(end-13:end,end-13:end,i)=forePic(end,end,i);
    exforePic(end-13:end,1:13,i)=forePic(end,1,i);
    exforePic(1:13,end-13:end,i)=forePic(1,end,i);
end
    
newforePic=uint8(zeros(size(forePic)));
for i=1:size(bkgPic,1)
    for j=1:size(bkgPic,2)
        goffset=(double(bkgPic(i,j))-128)/10;
        offsetLim1=floor(goffset)+13;
        offsetLim2=ceil(goffset)+13;
        sep1=goffset-floor(goffset);
        sep2=ceil(goffset)-goffset;
        c1=double(exforePic(i+offsetLim1,j+offsetLim1,:));
        c2=double(exforePic(i+offsetLim2,j+offsetLim2,:));
        if sep1==0
            c=double(exforePic(i+offsetLim1,j+offsetLim1,:));
        else
            c=c2.*sep1+c1.*sep2;
        end
        newforePic(i,j,:)=c;
    end
end

%grayForePic=rgb2gray(newforePic);
%rate=double(bkgPic)./double(grayForePic);

newforePic=uint8((double(newforePic).*double(bkgPic))./220);
imwrite(newforePic,'result.jpg')
imshow(newforePic)
end
```
