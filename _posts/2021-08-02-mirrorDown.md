---
layout: post
title:  "MATLAB制作水波倒影特效"
author: slandarer
categories: [ MATLAB]
image: https://img-blog.csdnimg.cn/938dbfc75a5e42ea8641990dde8d2750.png
image_external: true
featured: false
hidden: false
---

**注：** 本文算法参考大佬 **grafx** 的这篇博客：
[图像处理算法之水面倒影特效](https://blog.csdn.net/grafx/article/details/54604251)

由于本文使用MATLAB复现，因此很多语法上会显得比较简洁，同时本博文对原大佬文章部分内容进行了改写，详见本文：

### 0效果展示
![在这里插入图片描述](https://img-blog.csdnimg.cn/938dbfc75a5e42ea8641990dde8d2750.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3NsYW5kYXJlcg==,size_16,color_FFFFFF,t_70#pic_center)

![在这里插入图片描述](https://img-blog.csdnimg.cn/59481ffaf0fb4f44b936c5a052215286.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3NsYW5kYXJlcg==,size_16,color_FFFFFF,t_70#pic_center)

![在这里插入图片描述](https://img-blog.csdnimg.cn/3aa4d7a11f1b443c9b4cd3d19b5abc76.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3NsYW5kYXJlcg==,size_16,color_FFFFFF,t_70#pic_center)

### 1图像翻转及白化
**导入图像：** 这部分其实没啥好说的：

```java
% 图片导入 
oriPic=imread('test.jpg');
[Row,Col,~]=size(oriPic);
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/d9976a87ff534a329f89a09183b2e90f.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3NsYW5kYXJlcg==,size_16,color_FFFFFF,t_70#pic_center)


**翻转及白化图像：**

翻转就是单纯的将行索引倒过来；
白化就是将当前像素的颜色按比例和白色取个带权均值，行索引越大白色权重也越大，图像也就越白。

```java
% 图片翻转及白化 ==========================================================
whiteMat=((1:Row)./Row./1.2)'*ones(1,Col); % 白化比例矩阵
flipPic=zeros(Row,Col,3);                  % 翻转后矩阵初始化
for i=1:3
    tempChannel=double(oriPic(:,:,i));     % 获得通道图
    tempChannel=tempChannel(end:-1:1,:);   % 翻转
    tempChannel=tempChannel.*(1-whiteMat)+255.*whiteMat; % 白化
    flipPic(:,:,i)=tempChannel;
    
end
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/1485b3b40bda4bc8b5318a772f6ec8b6.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3NsYW5kYXJlcg==,size_16,color_FFFFFF,t_70#pic_center)

当然如果我们将这一行：

```java
tempChannel=tempChannel.*(1-whiteMat)+255.*whiteMat;
```
改写为：

```java
tempChannel=tempChannel.*(1-whiteMat)+0.*whiteMat;
```
就变成了一个黑化的过程：

![在这里插入图片描述](https://img-blog.csdnimg.cn/8ff2129baa714ddca4d2318fd0a35c13.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3NsYW5kYXJlcg==,size_16,color_FFFFFF,t_70#pic_center)

当然你也可以尝试其他颜色，例如将整段改写为：

```java
Color=[255,0,0];
colorMat=((1:Row)./Row./1.2)'*ones(1,Col); % 比例矩阵
flipPic=zeros(Row,Col,3);                  % 翻转后矩阵初始化
for i=1:3
    tempChannel=double(oriPic(:,:,i));     % 获得通道图
    tempChannel=tempChannel(end:-1:1,:);   % 翻转
    tempChannel=tempChannel.*(1-colorMat)+Color(i).*colorMat; % 渐变
    flipPic(:,:,i)=tempChannel;
    
end
imshow(uint8(flipPic))
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/ab772a0b8f6c49b2a75b823da8ebe9de.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3NsYW5kYXJlcg==,size_16,color_FFFFFF,t_70#pic_center)

### 2波纹图像构造
**生成噪声并模糊**

```java
noiseMat=ones(Row,Col);
noiseMat=imnoise(noiseMat,'gaussian',0,5); % 噪声添加
gaussOpt=fspecial('gaussian',[3 3],1);
noiseMat=imfilter(noiseMat,gaussOpt);
```
噪声图：

![在这里插入图片描述](https://img-blog.csdnimg.cn/e5f75189d7864f9485264d5d0e4a2470.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3NsYW5kYXJlcg==,size_16,color_FFFFFF,t_70#pic_center)

模糊后噪声图：

![在这里插入图片描述](https://img-blog.csdnimg.cn/4fdc385ce5e042639fb8c028ffe2a9cc.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3NsYW5kYXJlcg==,size_16,color_FFFFFF,t_70#pic_center)

**浮雕特效**

实际上浮雕特效就是用以下类似形式的矩阵对图像进行卷积，卷积结果在加上RGB范围的均值，[0,1]区间就加0.5,[0,255]区间就加128： 
  $$
    \begin{bmatrix}
  1 & 0 & 0 \\
   0 & 0 & 0 \\
   0 & 0 & -1
  \end{bmatrix} 
 \begin{bmatrix}
  2 & 0 & 2 \\
   0 & 0 & 0 \\
   -2 & 0 & -2
  \end{bmatrix} 
 $$
数值和位置不重要，重要的是相对位置互为相反数，浮雕过程描述如下：

```java
H=[cos(pi+pi/4),0,cos(pi-pi/4);
   cos(pi+2*pi/4),0,cos(pi-2*pi/4);
   cos(pi+3*pi/4),0,cos(pi-3*pi/4)];
noiseMat=imfilter(noiseMat,H,'conv')+0.5;  
noiseMat=noiseMat.*255;
noiseMat(noiseMat<0)=0;
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/ede71040e7d245a198c1d4277d38a517.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3NsYW5kYXJlcg==,size_16,color_FFFFFF,t_70#pic_center)

**透视变换**
就是近大远小，这里为了方便起见只在横向方向上做了近大远小的拉伸，竖直方向进行了等比例拉伸，因而不是严格意义上的透视变换：
![在这里插入图片描述](https://img-blog.csdnimg.cn/224c17448f4b485cbc49ce64b9ec4574.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3NsYW5kYXJlcg==,size_16,color_FFFFFF,t_70#pic_center)

如图所示实际操作就是把左侧蓝色区域拉伸成右侧蓝色区域，并只选取红框内部分，代码如下：

```java
% 图像透视变换 ============================================================
exNoiseMat=zeros(Row,Col);
% 横向拉伸上下边倍数
K1=10;K2=4;
for i=1:Row
    for j=1:Col
        k=K2+i*(K1-K2)/Row;
        nJ=(j-(1+Col)/2)/k+(1+Col)/2;
        if floor(nJ)==ceil(nJ)
            nJ=round(nJ);
            exNoiseMat(i,j)=noiseMat(i,nJ);
        else
            nJ1=floor(nJ);nJ2=ceil(nJ);
            exNoiseMat(i,j)=noiseMat(i,nJ1)*(nJ2-nJ)+noiseMat(i,nJ2)*(nJ-nJ1);
        end
    end
end
% 竖向拉伸3倍并只取一部分
exNoiseMat=imresize(exNoiseMat,[3*Row,Col]);
exNoiseMat=exNoiseMat(end-Row+1:end,:);
exNoiseMat=uint8(exNoiseMat);
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/67e854315636465db9c3325a03b823da.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3NsYW5kYXJlcg==,size_16,color_FFFFFF,t_70#pic_center)

**注：** 如果原图像尺寸过大，水波就会过于密集，这时候可以适当调整放缩倍数或者将原图像重调大小到小一点的尺寸。

例如大波浪代码：

```java
% 图像透视变换 ============================================================
exNoiseMat=zeros(Row,Col);
K1=40;K2=10;
for i=1:Row
    for j=1:Col
        k=K2+i*(K1-K2)/Row;
        nJ=(j-(1+Col)/2)/k+(1+Col)/2;
        if floor(nJ)==ceil(nJ)
            nJ=round(nJ);
            exNoiseMat(i,j)=noiseMat(i,nJ);
        else
            nJ1=floor(nJ);nJ2=ceil(nJ);
            exNoiseMat(i,j)=noiseMat(i,nJ1)*(nJ2-nJ)+noiseMat(i,nJ2)*(nJ-nJ1);
        end
    end
end
exNoiseMat=imresize(exNoiseMat,[8*Row,Col]);
exNoiseMat=exNoiseMat(end-Row+1:end,:);
exNoiseMat=uint8(exNoiseMat);
```

小波浪和大波浪：

![在这里插入图片描述](https://img-blog.csdnimg.cn/c2bc017cb1414059820ec9ed6668dd38.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3NsYW5kYXJlcg==,size_16,color_FFFFFF,t_70#pic_center)
![在这里插入图片描述](https://img-blog.csdnimg.cn/0d80d08fd579413ea2191f662201418a.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3NsYW5kYXJlcg==,size_16,color_FFFFFF,t_70#pic_center)

### 3扭曲置换
这个。。。老朋友了，具体原理还是看这一篇叭：[MATLAB 制作抖音同款含褶皱面料图](https://slandarer.blog.csdn.net/article/details/115709803)

```java
% 扭曲置换 ================================================================
forePic=flipPic;
bkgPic=exNoiseMat;

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
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/9aabef2b9d5047b59323281f09c1ea59.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3NsYW5kYXJlcg==,size_16,color_FFFFFF,t_70#pic_center)
![在这里插入图片描述](https://img-blog.csdnimg.cn/e48b86ae039a4687ab04c0ed75b48c8f.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3NsYW5kYXJlcg==,size_16,color_FFFFFF,t_70#pic_center)

### 4图像拼接
就是把俩图像拼在一起，并把边缘模糊一下：

```java
% 图像拼接 ================================================================
resultPic(:,:,1)=[oriPic(:,:,1);newforePic(:,:,1)];
resultPic(:,:,2)=[oriPic(:,:,2);newforePic(:,:,2)];
resultPic(:,:,3)=[oriPic(:,:,3);newforePic(:,:,3)];
% imshow(resultPic)


% 边缘模糊 ================================================================
gaussOpt=fspecial('gaussian',[3 3],0.5);
gaussPic=imfilter(resultPic,gaussOpt);
resultPic(Row-1:Row+2,:,1)=gaussPic(Row-1:Row+2,:,1);
resultPic(Row-1:Row+2,:,2)=gaussPic(Row-1:Row+2,:,2);
resultPic(Row-1:Row+2,:,3)=gaussPic(Row-1:Row+2,:,3);
imshow(resultPic)
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/11a367eec737425193ead50eaca0016d.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3NsYW5kYXJlcg==,size_16,color_FFFFFF,t_70#pic_center)

### 5完整代码

```java
function mirrorDown
% @author slandarer

% 图片导入 
oriPic=imread('test.jpg');
[Row,Col,~]=size(oriPic);

% 图片翻转及白化 ==========================================================
whiteMat=((1:Row)./Row./1.2)'*ones(1,Col); % 白化比例矩阵
flipPic=zeros(Row,Col,3);                  % 翻转后矩阵初始化
for i=1:3
    tempChannel=double(oriPic(:,:,i));     % 获得通道图
    tempChannel=tempChannel(end:-1:1,:);   % 翻转
    tempChannel=tempChannel.*(1-whiteMat)+255.*whiteMat; % 白化
    flipPic(:,:,i)=tempChannel;
    
end
% imshow(uint8(flipPic))


% 噪声图构造(高斯噪声及高斯模糊)===========================================
noiseMat=ones(Row,Col);
noiseMat=imnoise(noiseMat,'gaussian',0,5); % 噪声添加
gaussOpt=fspecial('gaussian',[3 3],1);
noiseMat=imfilter(noiseMat,gaussOpt);
imshow(noiseMat)

H=[cos(pi+pi/4),0,cos(pi-pi/4);
   cos(pi+2*pi/4),0,cos(pi-2*pi/4);
   cos(pi+3*pi/4),0,cos(pi-3*pi/4)];
noiseMat=imfilter(noiseMat,H,'conv')+0.5;  
noiseMat=noiseMat.*255;
noiseMat(noiseMat<0)=0;
% imshow(uint8(noiseMat))


% 图像透视变换 ============================================================
exNoiseMat=zeros(Row,Col);
K1=10;K2=4;
for i=1:Row
    for j=1:Col
        k=K2+i*(K1-K2)/Row;
        nJ=(j-(1+Col)/2)/k+(1+Col)/2;
        if floor(nJ)==ceil(nJ)
            nJ=round(nJ);
            exNoiseMat(i,j)=noiseMat(i,nJ);
        else
            nJ1=floor(nJ);nJ2=ceil(nJ);
            exNoiseMat(i,j)=noiseMat(i,nJ1)*(nJ2-nJ)+noiseMat(i,nJ2)*(nJ-nJ1);
        end
    end
end
exNoiseMat=imresize(exNoiseMat,[6*Row,Col]);
exNoiseMat=exNoiseMat(end-Row+1:end,:);
exNoiseMat=uint8(exNoiseMat);
% imshow(exNoiseMat)


% 扭曲置换 ================================================================
forePic=flipPic;
bkgPic=exNoiseMat;

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
% imshow(newforePic)


% 图像拼接 ================================================================
resultPic(:,:,1)=[oriPic(:,:,1);newforePic(:,:,1)];
resultPic(:,:,2)=[oriPic(:,:,2);newforePic(:,:,2)];
resultPic(:,:,3)=[oriPic(:,:,3);newforePic(:,:,3)];
% imshow(resultPic)


% 边缘模糊 ================================================================
gaussOpt=fspecial('gaussian',[3 3],0.5);
gaussPic=imfilter(resultPic,gaussOpt);
resultPic(Row-1:Row+2,:,1)=gaussPic(Row-1:Row+2,:,1);
resultPic(Row-1:Row+2,:,2)=gaussPic(Row-1:Row+2,:,2);
resultPic(Row-1:Row+2,:,3)=gaussPic(Row-1:Row+2,:,3);
imshow(resultPic)


end
```
### 6其他效果展示
![在这里插入图片描述](https://img-blog.csdnimg.cn/aeaa18abe4ab42d68987646f352d48c7.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3NsYW5kYXJlcg==,size_16,color_FFFFFF,t_70#pic_center)
![在这里插入图片描述](https://img-blog.csdnimg.cn/73c29049c5f647f0ad6231cec906a1b8.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3NsYW5kYXJlcg==,size_16,color_FFFFFF,t_70#pic_center)
![在这里插入图片描述](https://img-blog.csdnimg.cn/66921ff006ec45f988836c66f130df66.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3NsYW5kYXJlcg==,size_16,color_FFFFFF,t_70#pic_center)
![在这里插入图片描述](https://img-blog.csdnimg.cn/9a3a7996dd894c9c9a32beb5e5cd0375.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3NsYW5kYXJlcg==,size_16,color_FFFFFF,t_70#pic_center)
