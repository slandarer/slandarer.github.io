---
layout: post
title:  "使用MATLAB制作3D动态旋转相册"
author: slandarer
categories: [ MATLAB, 3D ]
image: https://img-blog.csdnimg.cn/20210717160948559.jpg
image_external: true
featured: false
hidden: false
---

看到很多HTML制作的这款相册，手痒，尝试用MATLAB实现一下，考虑了多个方案，最终采用了最流畅的一版。

### 效果
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210717160917749.gif#pic_center)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210717160948559.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3NsYW5kYXJlcg==,size_16,color_FFFFFF,t_70#pic_center)
### 教程部分
#### 1 图片导入与大小重设
需要有一个名为album的文件夹和当前m文件在同一文件夹，另外ablum文件夹内至少要有一张jpg格式图片
```java
path='.\album\';%文件夹名称
files=dir(fullfile(path,'*.jpg')); 
picNum=size(files,1);

%遍历路径下每一幅图像
for i=1:picNum
   fileName=strcat(path,files(i).name); 
   img=imread(fileName);
   img=imresize(img,[120,120]);
   imgSet{i}=img;
end
```
我们注意到，这里用了一次imresize将突破变为120x120大小，这里重设大小有三个作用：

 - 将不是方形的图片变为方形
 - 将图像设置固定大小，方便构造网格放置图片
 - 120x120的大小大约是能让图片表示清晰为前提下最小的大小，图片太大的话运行会卡，太小的话不清晰
#### 2 fig axes设置

```java
% fig axes设置
fig=figure('units','pixels','position',[50 50 600 600],...
                       'Numbertitle','off','resize','off',...
                       'name','album3d','menubar','none');
ax=axes('parent',fig,'position',[-0.5 -0.5 2 2],...
   'XLim', [-6 6],...
   'YLim', [-6 6],...
   'ZLim', [-6 6],...
   'Visible','on',...
   'XTick',[], ...
   'YTick',[],...
   'Color',[0 0 0],...
   'DataAspectRatioMode','manual',...
   'CameraPositionMode','manual');
hold(ax,'on')
```
大部分设置大家都能看懂，这里讲解一下一些比较少见的设置：

##### 2.1 为什么 axes的'position'属性不设置[0 0 1 1]?

> 因为是3D坐标轴，设置为[0 0 1 1]后旋转起来效果是这样的，所以我们axes要设置的比figure大一圈：
> 
> ![在这里插入图片描述](https://img-blog.csdnimg.cn/20210717162114141.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3NsYW5kYXJlcg==,size_16,color_FFFFFF,t_70#pic_center)

##### 2.2 为什么要设置CameraPositionMode这一奇怪的属性？

> 因为我们后期要频繁改变CameraPosition这一属性，而CameraPositionMode设置为manual可以让视角完全按照CameraPosition的数值来调整，至于为什么要调整视角呢？
,
当然是因为如果对图像位置数据进行处理数据量会贼大，因此我们不妨直接转动axes视角而非转动图片。

#### 3 绘制图形句柄
就是绘制小型立方体，中型立方体和大型立方体，其中鼠标移动到中型立方体中心时中型立方体变成大型立方体，这个可以靠设置图形对象的XData,YData,ZData数值来改变

##### 3.1 构造网格
由于surf曲面图可以将图像贴在上面，还可以设置透明度，我们决定用surf函数来绘制，要贴图首先要将曲面绘制出来，就要先构造曲面网格：

```java
% 用于绘制图片的网格
[XMesh,YMesh]=meshgrid(linspace(-1,1,120),linspace(-1,1,120));
ZMesh=ones(120,120);
```
##### 3.2 绘制小型立方体

```java
% 绘制图片立方体
surfPic(1)=surf(XMesh,YMesh,ZMesh,'CData',imgSet{mod(0,picNum)+1},'EdgeColor','none','FaceColor','interp');
surfPic(2)=surf(XMesh,YMesh(end:-1:1,:),-ZMesh,'CData',imgSet{mod(1,picNum)+1},'EdgeColor','none','FaceColor','interp');
surfPic(3)=surf(ZMesh,XMesh,YMesh(end:-1:1,:),'CData',imgSet{mod(2,picNum)+1},'EdgeColor','none','FaceColor','interp');
surfPic(4)=surf(XMesh,ZMesh,YMesh(end:-1:1,:),'CData',imgSet{mod(3,picNum)+1},'EdgeColor','none','FaceColor','interp');
surfPic(5)=surf(-ZMesh,XMesh,YMesh(end:-1:1,:),'CData',imgSet{mod(4,picNum)+1},'EdgeColor','none','FaceColor','interp');
surfPic(6)=surf(XMesh,-ZMesh,YMesh(end:-1:1,:),'CData',imgSet{mod(5,picNum)+1},'EdgeColor','none','FaceColor','interp');
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210717163212486.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3NsYW5kYXJlcg==,size_16,color_FFFFFF,t_70#pic_center)
##### 3.3 绘制中型立方体
有了小型立方体，中型的绘制起来就简单了起来，甚至可以用一个for循环解决，只需要循环提取小型立方体的XData,YData,ZData数据后乘以1.5绘制图像，并设置透明度即可：

```java
% 依靠小立方体数据绘制中等立方体
for i=1:6
    surfPicA(i)=surf(surfPic(i).XData.*1.5,surfPic(i).YData.*1.5,surfPic(i).ZData.*1.5,...
        'CData',surfPic(i).CData,'EdgeColor','none','FaceColor','interp','FaceAlpha',0.7);  
end
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210717163428689.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3NsYW5kYXJlcg==,size_16,color_FFFFFF,t_70#pic_center)
##### 3.4 大型立方体参数设置
大型立方体参数设置时就没那么简单，如果直接乘以2.5，图片与图片之间会没有缝隙，因此我们XData,YData,ZData数据虽然都要变大，但是要乘以不一样的数值，而且各个方向上乘的数值不同，因此我们可以事先设立一个矩阵，用来存储其参数：

```java
% 用来调整放大比例的矩阵
resizeMat=[2 2 2.5;2 2 2.5;2.5 2 2;
           2 2.5 2;2.5 2 2;2 2.5 2];
```
想直接画大型正方形可以试试如下代码：

```java
% 最大图片绘制       
% for i=1:6
%     surfPicB(i)=surf(surfPic(i).XData.*resizeMat(i,1),...
%                      surfPic(i).YData.*resizeMat(i,2),...
%                      surfPic(i).ZData.*resizeMat(i,3),...
%                      'CData',surfPic(i).CData,'EdgeColor',...
%                      'none','FaceColor','interp','FaceAlpha',0.7);  
% end    
```
#### 4 立方体旋转
我们只需要设置一个timer函数不断调整CameraPosition即可：

```java
fps=40;theta=0;
rotateTimer=timer('ExecutionMode', 'FixedRate', 'Period',1/fps, 'TimerFcn', @rotateCube);
start(rotateTimer)

    function rotateCube(~,~)
        theta=theta+0.02;
        ax.CameraPosition=[cos(theta)*5*sqrt(2),sin(theta)*5*sqrt(2),5];
    end
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210717164952585.gif#pic_center)
#### 5 获取鼠标与中心点的距离
本来想直接在timer调用的函数里写get(fig,'CurrentPoint');来获得鼠标当前位置的，但发现这样写只有鼠标点击窗口才会有反应，并不是鼠标移动就会有反应，因此我们再构造一个WindowButtonMotionFcn回调，**！！！这一部分代码要写在上一步代码的前面！！！**

```java
lastDis=300;
preDis=300;
set(fig,'WindowButtonMotionFcn',@move2center)    
    function move2center(~,~)
        xy=get(fig,'CurrentPoint');
        preDis=sqrt(sum((xy-[300,300]).^2));
    end
```

preDis就是鼠标到图片中心的位置，我为什么要设置一个lastDis呢，因为每次移动鼠标都更新图像实在太卡了，因此我们要加一个判定，当且仅当以下两种情况更新图片大小

 - 之前鼠标距离中心>=150，现在<150
 - 之前鼠标距离中心<150，现在>=150
#### 6 鼠标移动到fig中心时更新图片
将之前的rotateCube函数改成这样：
```java
function rotateCube(~,~)
        theta=theta+0.02;
        ax.CameraPosition=[cos(theta)*5*sqrt(2),sin(theta)*5*sqrt(2),5];
        if (~all([preDis lastDis]<150))&&any([preDis lastDis]<150)
            for ii=1:6
                if preDis<150
                    surfPicA(ii).XData=surfPic(ii).XData.*resizeMat(ii,1);
                    surfPicA(ii).YData=surfPic(ii).YData.*resizeMat(ii,2);
                    surfPicA(ii).ZData=surfPic(ii).ZData.*resizeMat(ii,3);
                else
                    surfPicA(ii).XData=surfPic(ii).XData.*1.5;
                    surfPicA(ii).YData=surfPic(ii).YData.*1.5;
                    surfPicA(ii).ZData=surfPic(ii).ZData.*1.5;
                end
            end
        end
        lastDis=preDis;
    end
```
其中：

> (~all([preDis lastDis]<150))&&any([preDis lastDis]<150)

是用来判断上一次鼠标位置和当前鼠标位置是否只有一个距离中心<150

另：

for 循环中使用else来判断应该绘制大图片还是中等图片
### 完整代码

```java
function album3d
path='.\album\';%文件夹名称
files=dir(fullfile(path,'*.jpg')); 
picNum=size(files,1);

%遍历路径下每一幅图像
for i=1:picNum
   fileName=strcat(path,files(i).name); 
   img=imread(fileName);
   img=imresize(img,[120,120]);
   imgSet{i}=img;
end

% fig axes设置
fig=figure('units','pixels','position',[50 50 600 600],...
                       'Numbertitle','off','resize','off',...
                       'name','album3d','menubar','none');
ax=axes('parent',fig,'position',[-0.5 -0.5 2 2],...
   'XLim', [-6 6],...
   'YLim', [-6 6],...
   'ZLim', [-6 6],...
   'Visible','on',...
   'XTick',[], ...
   'YTick',[],...
   'Color',[0 0 0],...
   'DataAspectRatioMode','manual',...
   'CameraPositionMode','manual');
hold(ax,'on')
ax.CameraPosition=[5 5 5];

% 用于绘制图片的网格
[XMesh,YMesh]=meshgrid(linspace(-1,1,120),linspace(-1,1,120));
ZMesh=ones(120,120);

% 绘制图片立方体
surfPic(1)=surf(XMesh,YMesh,ZMesh,'CData',imgSet{mod(0,picNum)+1},'EdgeColor','none','FaceColor','interp');
surfPic(2)=surf(XMesh,YMesh(end:-1:1,:),-ZMesh,'CData',imgSet{mod(1,picNum)+1},'EdgeColor','none','FaceColor','interp');
surfPic(3)=surf(ZMesh,XMesh,YMesh(end:-1:1,:),'CData',imgSet{mod(2,picNum)+1},'EdgeColor','none','FaceColor','interp');
surfPic(4)=surf(XMesh,ZMesh,YMesh(end:-1:1,:),'CData',imgSet{mod(3,picNum)+1},'EdgeColor','none','FaceColor','interp');
surfPic(5)=surf(-ZMesh,XMesh,YMesh(end:-1:1,:),'CData',imgSet{mod(4,picNum)+1},'EdgeColor','none','FaceColor','interp');
surfPic(6)=surf(XMesh,-ZMesh,YMesh(end:-1:1,:),'CData',imgSet{mod(5,picNum)+1},'EdgeColor','none','FaceColor','interp');

% 依靠小立方体数据绘制中等立方体
for i=1:6
    surfPicA(i)=surf(surfPic(i).XData.*1.5,surfPic(i).YData.*1.5,surfPic(i).ZData.*1.5,...
        'CData',surfPic(i).CData,'EdgeColor','none','FaceColor','interp','FaceAlpha',0.7);  
end

% 用来调整放大比例的矩阵
resizeMat=[2 2 2.5;2 2 2.5;2.5 2 2;
           2 2.5 2;2.5 2 2;2 2.5 2];

% 最大图片绘制       
% for i=1:6
%     surfPicB(i)=surf(surfPic(i).XData.*resizeMat(i,1),...
%                      surfPic(i).YData.*resizeMat(i,2),...
%                      surfPic(i).ZData.*resizeMat(i,3),...
%                      'CData',surfPic(i).CData,'EdgeColor',...
%                      'none','FaceColor','interp','FaceAlpha',0.7);  
% end     


lastDis=300;
preDis=300;
set(fig,'WindowButtonMotionFcn',@move2center)    
    function move2center(~,~)
        xy=get(fig,'CurrentPoint');
        preDis=sqrt(sum((xy-[300,300]).^2));
    end



fps=40;theta=0;
rotateTimer=timer('ExecutionMode', 'FixedRate', 'Period',1/fps, 'TimerFcn', @rotateCube);
start(rotateTimer)



    function rotateCube(~,~)
        theta=theta+0.02;
        ax.CameraPosition=[cos(theta)*5*sqrt(2),sin(theta)*5*sqrt(2),5];
        if (~all([preDis lastDis]<150))&&any([preDis lastDis]<150)
            for ii=1:6
                if preDis<150
                    surfPicA(ii).XData=surfPic(ii).XData.*resizeMat(ii,1);
                    surfPicA(ii).YData=surfPic(ii).YData.*resizeMat(ii,2);
                    surfPicA(ii).ZData=surfPic(ii).ZData.*resizeMat(ii,3);
                else
                    surfPicA(ii).XData=surfPic(ii).XData.*1.5;
                    surfPicA(ii).YData=surfPic(ii).YData.*1.5;
                    surfPicA(ii).ZData=surfPic(ii).ZData.*1.5;
                end
            end
        end
        lastDis=preDis;
    end




% 弃用方案：太卡
% set(fig,'WindowButtonMotionFcn',@move2center)    
%     function move2center(~,~)
%         xy=get(fig,'CurrentPoint');
%         dis=sum((xy-[300,300]).^2);
%         for ii=1:6
%             if dis<200
%                 surfPicA(ii).XData=surfPic(ii).XData.*resizeMat(ii,1);
%                 surfPicA(ii).YData=surfPic(ii).YData.*resizeMat(ii,2);
%                 surfPicA(ii).ZData=surfPic(ii).ZData.*resizeMat(ii,3);
%             else
%                 surfPicA(ii).XData=surfPic(ii).XData;
%                 surfPicA(ii).YData=surfPic(ii).YData;
%                 surfPicA(ii).ZData=surfPic(ii).ZData;
%             end    
%         end
%         
%         
%         
%     end

end
```