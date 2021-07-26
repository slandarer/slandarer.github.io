---
layout: post
title:  "毕业季，用MATLAB制作一款环形相册吧 !"
author: slandarer
categories: [ MATLAB]
image: https://img-blog.csdnimg.cn/20210601200627288.png
image_external: true
featured: false
hidden: false
---

本文主要使用MATLAB的image对象构造图片，并通过一系列交并运算得出透明区域范围，构造弧型图片，再将图片放置在相应位置即可

### 0.运行效果
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210601200627288.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3NsYW5kYXJlcg==,size_16,color_FFFFFF,t_70#pic_center)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210601200722529.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3NsYW5kYXJlcg==,size_16,color_FFFFFF,t_70#pic_center)
### 完整步骤
#### 1.图片准备及导入
要制作一款相册足够的图片量是必不可少的，不然整个相册只有一张图来回重复多没意思呀，因此我们需要一个文件夹专门放图片，为了方便导入，这里全部都是jpg格式：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210601201637990.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3NsYW5kYXJlcg==,size_16,color_FFFFFF,t_70#pic_center)
**图片导入代码：**

```java
path='.\album\';%文件夹路径
files=dir(fullfile(path,'*.jpg')); 
picNum=size(files,1);

%遍历路径下每一幅图像
for i=1:picNum
   fileName = strcat(path,files(i).name); 
   img = imread(fileName);
   imgSet.(['p',num2str(i)])=img;
end
```
#### 2.为每张图片制作遮罩层

```java
for i=1:length(BlockNum)%圆环层数
    blockNum=BlockNum(i);%每一层图片数量
    Rrange=R(i,:);%每次一层半径范围
    for j=1:blockNum
        tempBoard=ones(2401,2401)==1;
        tempBoard=tempBoard&(disMesh>Rrange(1))&(disMesh<Rrange(2));
        tempBoard=tempBoard&(thetaMesh>((j-1)*2*pi/blockNum))&(thetaMesh<(j*2*pi/blockNum));
    end
end
```

就是依靠两个判定条件来叠加来构造扇形结构：

 - 离中心点半径处于[r,R]范围内
 - 与x轴正半轴夹角处于[theta1,theta2]之间
 
 这是一个取交集的过程，图片描述大概是下面这个样子：
 ![在这里插入图片描述](https://img-blog.csdnimg.cn/2021060120223472.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3NsYW5kYXJlcg==,size_16,color_FFFFFF,t_70#pic_center)
 假设我们已构建好Xmesh，Ymesh矩阵
 
```java
[XMesh,YMesh]=meshgrid(-1200:1:1200,-1200:1:1200);
```

 
 **距离矩阵：**
 那么距离矩阵disMesh可以这样构造：

```java
disMesh=sqrt(XMesh.^2+YMesh.^2);
```
**theta角矩阵：**
我们首先肯定能想到atan2,一个四象限反正弦函数，他的映射关系是这样的：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210601202724734.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3NsYW5kYXJlcg==,size_16,color_FFFFFF,t_70#pic_center)

是从z值范围为-pi到pi，且是以x轴负半轴为0度角的，这里我们将其z值增加pi且将坐标轴翻转，就能得到theta角矩阵：

```java
thetaMesh=atan2(YMesh,XMesh)+pi;
thetaMesh=thetaMesh(:,end:-1:1);
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210601203533364.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3NsYW5kYXJlcg==,size_16,color_FFFFFF,t_70#pic_center)
更改后便是从x轴正半轴开始，映射范围为[0,2*pi].

#### 3.调整每张图大小
我们找到每个蒙版x,y的范围将其裁剪出来：

<img src="https://img-blog.csdnimg.cn/20210601204513825.png#pic_center" width="50%">

然后按照比例将原图大小变换至至少有一个边长与蒙版相等，另一边长长于蒙版，然后截取图片中心部分，代码如下：

```java
for i=1:length(BlockNum)%圆环层数
    blockNum=BlockNum(i);%每一层图片数量
    Rrange=R(i,:);%每一层半径范围
    for j=1:blockNum
        tempBoard=ones(2401,2401)==1;
        tempBoard=tempBoard&(disMesh>Rrange(1))&(disMesh<Rrange(2));
        tempBoard=tempBoard&(thetaMesh>((j-1)*2*pi/blockNum))&(thetaMesh<(j*2*pi/blockNum));
        TrueX=find(sum(tempBoard,1)>0);
        TrueY=find(sum(tempBoard,2)>0);
        tempMask=tempBoard(min(TrueY):max(TrueY),min(TrueX):max(TrueX));
        x1=YMesh(min(TrueX),min(TrueY));
        y1=XMesh(min(TrueX),min(TrueY));
        x2=YMesh(max(TrueX),max(TrueY));
        y2=XMesh(max(TrueX),max(TrueY));
        xdiff=x2-x1;
        ydiff=y2-y1;
        
        pic=imgSet.(['p',num2str(tempPic)]);
        [rows,cols,~]=size(pic);
        ratio=[ydiff+1,xdiff+1]./[rows,cols];
        newsize=ceil([rows,cols].*max(ratio));
        offset=floor((newsize-[ydiff+1,xdiff+1])./2);
        pic=imresize(pic,newsize);
        pic=pic((1:ydiff+1)+offset(1),(1:xdiff+1)+offset(2),:);
    end
end
```
#### 3.绘图及绘图参数详解
**基本参数：**

> BlockNum=[7,11];%每层扇形数量
R=[300,670;%第一层半径范围
   670,1090];%第二层半径范围
lineColor=[0.98,0.98,0.98];线颜色
lineWidth=2;%线粗细

关于线的属性之后再说

绘图就直接是用image函数，这个没啥好说的，如果文件夹图片不多我们会采用取余的方式循环画之前的图：

```java
tempPic=1;
for i=1:length(BlockNum)
    blockNum=BlockNum(i);
    for j=1:blockNum
        tempPic=tempPic+1;
        tempPic=mod(tempPic-1,picNum)+1;
    end
end
```
我们发现直接绘图的话边缘锯齿化比较严重：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210601205739565.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3NsYW5kYXJlcg==,size_16,color_FFFFFF,t_70#pic_center)
我们很容易想到画线来遮丑：

```java
t=0:0.001:(2*pi+0.001);
for i=1:length(BlockNum)
    blockNum=BlockNum(i);
    Rrange=R(i,:);
    for j=1:blockNum
        plot(cos(j*2*pi/blockNum).*Rrange,sin(j*2*pi/blockNum).*Rrange,'Color',lineColor,'LineWidth',lineWidth)
    end
    plot(cos(t).*Rrange(1),sin(t).*Rrange(1),'Color',lineColor,'LineWidth',lineWidth)
    plot(cos(t).*Rrange(2),sin(t).*Rrange(2),'Color',lineColor,'LineWidth',lineWidth)
end
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210601211023681.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3NsYW5kYXJlcg==,size_16,color_FFFFFF,t_70#pic_center)

当然也可以画黑线：
只需要lineColor=[0,0,0]或者lineColor='k'即可
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210601205904370.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3NsYW5kYXJlcg==,size_16,color_FFFFFF,t_70#pic_center)
#### 4.完整代码

```java
function ringAlbum
BlockNum=[7,11];
R=[300,670;
   670,1090];
lineColor=[0.98,0.98,0.98];
lineWidth=2;

path='.\album\';%文件夹名称
files=dir(fullfile(path,'*.jpg')); 
picNum=size(files,1);

%遍历路径下每一幅图像
for i=1:picNum
   fileName = strcat(path,files(i).name); 
   img = imread(fileName);
   imgSet.(['p',num2str(i)])=img;
end

fig=figure('units','pixels',...
        'position',[20 60 560 560],...
        'Color',[1 1 1]);
ax=axes('Units','pixels',...
        'parent',fig,...  
        'Color',[1 1 1],...
        'Position',[0 0 560,560],...
        'XLim',[-1200,1200],...
        'YLim',[-1200,1200],...
        'XColor','none',...
        'YColor','none');
hold(ax,'on')
ax.YDir='reverse';
ax.XDir='normal';



[XMesh,YMesh]=meshgrid(-1200:1:1200,-1200:1:1200);
disMesh=sqrt(XMesh.^2+YMesh.^2);
thetaMesh=atan2(YMesh,XMesh)+pi;
thetaMesh=thetaMesh(:,end:-1:1);

tempPic=1;
t=0:0.001:(2*pi+0.001);
for i=1:length(BlockNum)
    blockNum=BlockNum(i);
    Rrange=R(i,:);
    for j=1:blockNum
        tempBoard=ones(2401,2401)==1;
        tempBoard=tempBoard&(disMesh>Rrange(1))&(disMesh<Rrange(2));
        tempBoard=tempBoard&(thetaMesh>((j-1)*2*pi/blockNum))&(thetaMesh<(j*2*pi/blockNum));
        TrueX=find(sum(tempBoard,1)>0);
        TrueY=find(sum(tempBoard,2)>0);
        tempMask=tempBoard(min(TrueY):max(TrueY),min(TrueX):max(TrueX));
        x1=YMesh(min(TrueX),min(TrueY));
        y1=XMesh(min(TrueX),min(TrueY));
        x2=YMesh(max(TrueX),max(TrueY));
        y2=XMesh(max(TrueX),max(TrueY));
        xdiff=x2-x1;
        ydiff=y2-y1;
        
        pic=imgSet.(['p',num2str(tempPic)]);
        [rows,cols,~]=size(pic);
        ratio=[ydiff+1,xdiff+1]./[rows,cols];
        newsize=ceil([rows,cols].*max(ratio));
        offset=floor((newsize-[ydiff+1,xdiff+1])./2);
        pic=imresize(pic,newsize);
        pic=pic((1:ydiff+1)+offset(1),(1:xdiff+1)+offset(2),:);
        
        image(ax,[x1,x2],[y1,y2],pic,'alphaData',tempMask);
        tempPic=tempPic+1;
        tempPic=mod(tempPic-1,picNum)+1;
    end
    for j=1:blockNum
        plot(cos(j*2*pi/blockNum).*Rrange,sin(j*2*pi/blockNum).*Rrange,'Color',lineColor,'LineWidth',lineWidth)
    end
    plot(cos(t).*Rrange(1),sin(t).*Rrange(1),'Color',lineColor,'LineWidth',lineWidth)
    plot(cos(t).*Rrange(2),sin(t).*Rrange(2),'Color',lineColor,'LineWidth',lineWidth)
end

end
```
