---
layout: post
title:  "嘘！我有特殊的绕线画技巧，教你用MATLAB绘制另一种绕线画"
author: slandarer
categories: [ MATLAB, 绕线画]
image: https://img-blog.csdnimg.cn/20210526203453396.png
image_external: true
featured: false
hidden: false
---

算法来自linify.me网站和郑越升改进算法

**效果：**
<img src="https://img-blog.csdnimg.cn/20210526200512639.png#pic_center" width="70%">

**过程图效果：**
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210526201127700.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3NsYW5kYXJlcg==,size_16,color_FFFFFF,t_70#pic_center)

### 1.过程描述
**linify.me网站算法：**

 1. 随机寻找一个亮度值低于阈值（80）的像素点。
 2. 找出所有由200个点里任意两个点连成的，并且离该像素点最近的线。
 3. 在这些线里面，计算每条线上经过的像素的平均亮度值，画出最低亮度值的那条线。
 4. 被画出来的那条线经过的所有像素，将亮度值设为最大（255），即不参与下次循环。
 5. 重复以上过程，直到画出 lineNum 条线。

**郑越升改进算法：**

 1. 在圆周上200点里，以两个点为一组，随机抽取80个组合。
 2. 抽取的每一组的两点可以定义一条线段，计算每一组的线段经过的像素亮度平均值，找出平均像素亮度值最低的一组，并对该组两点进行连线。
 3. 连线经过的像素亮度值提高50。
 4. 重复以上过程，直到画出 lineNum 条线。

注：算法中各个数值都可以进行改变(比如想绘制的更细节可以在圆周上取300个点，算法部分来自于推送：[Processing绕线作画](https://mp.weixin.qq.com/s?__biz=MzA4NjM4MTcyNA==&mid=2650153069&idx=1&sn=a90bebbfef3c848d4b03c29867eb2cd7&chksm=87cb059eb0bc8c8835b7b4c4f05b83543aa3d85f20cdd3ad70dc00d6148c751609ee8f22e8bd&mpshare=1&scene=23&srcid=0526194n7RFlqTJcRhdp2TFb&sharer_sharetime=1622031205494&sharer_shareid=160e3ea2dc81aae4386ebd2d68333bae#rd)
### 2.如何连线经过的点
#### 2.1 点到直线的距离
我们已知两个钉子的位置，求点到两钉子构成直线距离，首先就要做出一个垂直于直线方向的单位向量，下图V，之后用向量CB与V做点乘即可，计算出如果点到直线距离小于1，点就在线上(0.5-1均可)
<img src="https://img-blog.csdnimg.cn/2021052620225057.png#pic_center" width="70%">

#### 2.2 选择椭圆内部的点
我们简单的取个交集即可
第二张图是我们计算出直线经过的点，图一是提前计算好的椭圆内点。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210526202419512.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3NsYW5kYXJlcg==,size_16,color_FFFFFF,t_70#pic_center)

### 3.plot函数如何绘制透明线
一般颜色参数是0-1范围的1x3数组，我们可以给他扩充为1x4，第四个参数就是透明度，例如：

> plot(X,Y,'Color',[0,0,0,0.5])

### 4.绕线部分完整代码
图片大小500x500左右效果比较好，图片最好边缘明显，灰度差距大。

```java
function wireDarw
oriPic=imread('test.jpg');
nailNum=300;%钉子数量
randNum=80;%一次采样数量
lineNum=2000;%线条数量



[rows,cols,nChannels]=size(oriPic);
if nChannels>1
    oriPic=rgb2gray(oriPic);
end

ratio=[1260,560]./[cols,rows];
fig=figure('units','pixels',...
        'position',[20 60 min(ratio)*(cols+1) min(ratio)*(rows+1)],...
        'Color',[1 1 1]);
ax=axes('Units','pixels',...
        'parent',fig,...  
        'Color',[1 1 1],...
        'Position',[0 0 min(ratio)*(cols+1) min(ratio)*(rows+1)],...
        'XLim',[0 cols+1],...
        'YLim',[0 rows+1],...
        'XColor','none',...
        'YColor','none');
hold on
ax.YDir='normal';



midX=(1+cols)/2;
midY=(1+rows)/2;
t=linspace(0,2*pi,nailNum+1);
t(end)=[];t=t(:);

nailPos=[(midX-0.5).*(cos(t)+1),(midY-0.5).*(sin(t)+1)]+0.5;
scatter(nailPos(:,1),nailPos(:,2),5,'filled','CData',[0 0 0])

degreeMat=oriPic(end:-1:1,:);
[XMesh,YMesh]=meshgrid((1:cols)-midX,(1:rows)-midY);
normXMesh=XMesh./(midX-0.5);
normYMesh=YMesh./(midY-0.5);
l2Mesh=normXMesh.^2+normYMesh.^2;
outCircleMesh=l2Mesh>1;
degreeMat=double(degreeMat);


[XSet,YSet]=meshgrid(1:cols,1:rows);

for i=1:lineNum
    randiNail=randi([1,nailNum],[randNum,2]);
    randiNail(randiNail(:,1)==randiNail(:,2),:)=[];
    lumSet=inf.*ones(1,size(randiNail,1));
    for j=1:size(randiNail,1)
        pnt1=nailPos(randiNail(j,1),:);
        pnt2=nailPos(randiNail(j,2),:);
        v=[pnt2(2)-pnt1(2),pnt1(1)-pnt2(1)];v=v./norm(v);
        onLine=abs((XSet-pnt1(1)).*v(1)+(YSet-pnt1(2)).*v(2))<1;
        onLine=onLine&(~outCircleMesh);
        if sum(sum(onLine))>10
            lumMean=mean(degreeMat(onLine));
            lumSet(j)=lumMean;
        else
            lumSet(j)=inf;
        end
    end
    [~,index]=sort(lumSet);
    S_pnt1=nailPos(randiNail(index(1),1),:);
    S_pnt2=nailPos(randiNail(index(1),2),:);
    S_v=[S_pnt2(2)-S_pnt1(2),S_pnt1(1)-S_pnt2(1)];S_v=S_v./norm(S_v);
    S_onLine=abs((XSet-S_pnt1(1)).*S_v(1)+(YSet-S_pnt1(2)).*S_v(2))<0.6;
    S_onLine=S_onLine&(~outCircleMesh);
    degreeMat(S_onLine)=degreeMat(S_onLine)+50;
    plot([S_pnt1(1),S_pnt2(1)],[S_pnt1(2),S_pnt2(2)],'Color',[0.2,0.2,0.2,0.4],'LineWidth',0.3)
    disp(i)
    pause(0.001)
end

end
```
夹带私货hiahiahia : 
<img src="https://img-blog.csdnimg.cn/20210526202902692.png#pic_center" width="70%">

<img src="https://img-blog.csdnimg.cn/20210526203004630.png#pic_center" width="70%">
<img src="https://img-blog.csdnimg.cn/20210526203255534.png#pic_center" width="70%">




### 5.绕线部分存在问题
若图片灰度对比度不大，绘制出图片会很模糊，例如我的头像：
<img src="https://img-blog.csdnimg.cn/20210526203158590.png#pic_center" width="80%">
<img src="https://img-blog.csdnimg.cn/20210526203401239.png#pic_center" width="70%">

### 6.颜色渲染

```java
oriPic=imread('test.jpg');%原图片
linePic=imread('untitled1.png');%绕线画

linePic=rgb2gray(linePic);
oriPic=imresize(oriPic,size(linePic));


newPic=255-(255-double(linePic)).*(255-double(oriPic))./255;
figure
imshow(uint8(newPic))
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210526203453396.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3NsYW5kYXJlcg==,size_16,color_FFFFFF,t_70#pic_center)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210526203627517.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3NsYW5kYXJlcg==,size_16,color_FFFFFF,t_70#pic_center)

---
有好多人向我反映，图像渲染点乘不正确的问题，是由于较早的版本还不支持三维数组与二维数组的点乘，若是较早版本请使用如下代码，即将三色通道分别处理后再合成：

```java
oriPic=imread('test.jpg');%原图
linePic=imread('untitled1.png');%绕线图

linePic=rgb2gray(linePic);
oriPic=imresize(oriPic,size(linePic));

oriR=oriPic(:,:,1);
oriG=oriPic(:,:,2);
oriB=oriPic(:,:,3);
newPic(:,:,1)=255-(255-double(linePic)).*(255-double(oriR))./255;
newPic(:,:,2)=255-(255-double(linePic)).*(255-double(oriG))./255;
newPic(:,:,3)=255-(255-double(linePic)).*(255-double(oriB))./255;
newPic=uint8(newPic);
figure
imshow(newPic)
```

