---
layout: post
title:  "MATLAB 制作抖音同款 立体人物文字海报"
author: slandarer
categories: [ MATLAB, 立体文字, 抖音, 正交叠底, 扭曲置换]
image: https://img-blog.csdnimg.cn/20210415143241815.jpg
image_external: true
featured: false
hidden: false
---

抖音同款第三弹

效果如下：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210415143241815.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3NsYW5kYXJlcg==,size_16,color_FFFFFF,t_70#pic_center)

#### 步骤
##### 1.导入图片并制作文字图
原图在这里：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210415143426883.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3NsYW5kYXJlcg==,size_16,color_FFFFFF,t_70#pic_center)

原理就是创建一个隐藏的fig窗口，画完图后存储为图片，再调节至与原本图片相同大小
代码：

```java
string='you are very welcome to follow my CSDN blog';
lineNum=40;%文本行数
fontSize=13;%字号
dislocation=1;%文本偏移
fontName='Helvetica';%字体
fontWeight='normal';%bold/normal是否粗体


bkgPic=imread('test.jpg');
[m,n,k]=size(bkgPic);
if k~=1
    bkgPic=rgb2gray(bkgPic);
end


if length(string)<100
    newString=[];
    repeatTimes=ceil(100/length(string));
    for i=1:repeatTimes
        newString=[newString,'  ',string];
    end
end
string=newString;


fig=figure('units','pixels',...
        'position',[20 20 n m],...
        'Numbertitle','off',...
        'Color',[0 0 0],...
        'resize','off',...
        'visible','off',...
         'menubar','none');
ax=axes('Units','pixels',...
        'parent',fig,...  
        'Color',[0 0 0],...
        'Position',[0 0 n m],...
        'XLim',[0 n],...
        'YLim',[0 m],...
        'XColor','none',...
        'YColor','none');

sep=m/lineNum;
i=0;
for h=sep/2:sep:m
    modNum=mod(i*dislocation,length(string));
    tempStr=[string((1+modNum):end),string(1:modNum)];
    text(ax,0,h,tempStr,'Color',[1 1 1],'FontSize',fontSize,...
        'FontWeight',fontWeight,'FontName',fontName);
    i=i+1;
end

saveas(fig,'text.png');
textPic=imread('text.png');
pause(0.5)
delete('text.png')
clc;close all

figure(1)
textPic=255-textPic;
forePic=imresize(textPic,size(bkgPic));
```

文字图制作结果：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210415143553785.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3NsYW5kYXJlcg==,size_16,color_FFFFFF,t_70#pic_center)

dislocation=1;%文本偏移
这个参数可调成别的数值，例如0的时候就意味着不偏移：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210415143703452.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3NsYW5kYXJlcg==,size_16,color_FFFFFF,t_70#pic_center)
##### 2.背景图高斯模糊

```java
bkgPic=imgaussfilt(bkgPic,3);
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210415143951783.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3NsYW5kYXJlcg==,size_16,color_FFFFFF,t_70#pic_center)
##### 3.文字图像素映射
原理和上一篇一样，可以瞅一眼：
[MATLAB 制作抖音同款含褶皱面料图](https://blog.csdn.net/slandarer/article/details/115709803)
代码：

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
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210415144331768.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3NsYW5kYXJlcg==,size_16,color_FFFFFF,t_70#pic_center)
##### 4.正交叠底
原理依旧和上一篇一样。。。
代码：

```java
newforePic=uint8((double(newforePic).*double(bkgPic))./220);
imwrite(newforePic,'result.jpg')
imshow(newforePic)
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210415143241815.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3NsYW5kYXJlcg==,size_16,color_FFFFFF,t_70#pic_center)
##### 5.完整代码

```java
function characterText
string='you are very welcome to follow my CSDN blog';
lineNum=40;%文本行数
fontSize=13;%字号
dislocation=1;%文本偏移
fontName='Helvetica';%字体
fontWeight='normal';%bold/normal是否粗体


bkgPic=imread('test.jpg');
[m,n,k]=size(bkgPic);
if k~=1
    bkgPic=rgb2gray(bkgPic);
end


if length(string)<100
    newString=[];
    repeatTimes=ceil(100/length(string));
    for i=1:repeatTimes
        newString=[newString,'  ',string];
    end
end
string=newString;


fig=figure('units','pixels',...
        'position',[20 20 n m],...
        'Numbertitle','off',...
        'Color',[0 0 0],...
        'resize','off',...
        'visible','off',...
         'menubar','none');
ax=axes('Units','pixels',...
        'parent',fig,...  
        'Color',[0 0 0],...
        'Position',[0 0 n m],...
        'XLim',[0 n],...
        'YLim',[0 m],...
        'XColor','none',...
        'YColor','none');

sep=m/lineNum;
i=0;
for h=sep/2:sep:m
    modNum=mod(i*dislocation,length(string));
    tempStr=[string((1+modNum):end),string(1:modNum)];
    text(ax,0,h,tempStr,'Color',[1 1 1],'FontSize',fontSize,...
        'FontWeight',fontWeight,'FontName',fontName);
    i=i+1;
end

saveas(fig,'text.png');
textPic=imread('text.png');
pause(0.5)
delete('text.png')
clc;close all

figure(1)
textPic=255-textPic;
forePic=imresize(textPic,size(bkgPic));

bkgPic=imgaussfilt(bkgPic,3);

%bkgPic
%imshow(bkgPic)

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

newforePic=uint8((double(newforePic).*double(bkgPic))./220);
imwrite(newforePic,'result.jpg')
imshow(newforePic)

end
```
##### 5.其他成品展示
注：图片需选择深色背景图片以避免过度迁移，同时在选择其它图片时应对代码最前面字体，字号，文本行数进行调整，若图片尺寸过小，需要将高斯模糊的第二个参数调小(sigma)

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210415145544977.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3NsYW5kYXJlcg==,size_16,color_FFFFFF,t_70#pic_center)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210415145713438.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3NsYW5kYXJlcg==,size_16,color_FFFFFF,t_70#pic_center)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210415145828824.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3NsYW5kYXJlcg==,size_16,color_FFFFFF,t_70#pic_center)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210415150859226.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3NsYW5kYXJlcg==,size_16,color_FFFFFF,t_70#pic_center)
