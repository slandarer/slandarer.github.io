---
layout: post
title:  "MATLAB竟能制作如此方便的划词翻译工具"
author: slandarer
categories: [ MATLAB, 划词翻译]
image: https://img-blog.csdnimg.cn/01a7bcbe7e204e6db431ce5e4984638c.jpg
image_external: true
featured: false
hidden: false
---



> 我点开程序一看，程序第一行就写着import，
却歪歪斜斜的每行上都是着MATLAB几个大字。
我横竖睡不着，仔细看了半夜，
才从字缝里看出字来，满页都写着
⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀——Java

其实用的java的部分也不是很多，而且用的都是MATLAB自带的java包，主体还是MATLAB，这篇文章主要是手把手教你如何用MATLAB+java+app designer制作一款划词翻译工具：
### -1注
2021a版本MATLAB相对于之前版本有些变更，已在文中注明


---
### 0使用效果
**效果图片：**
![在这里插入图片描述](https://img-blog.csdnimg.cn/67d88e1f2d0f4277a3eda08f0fd4c7a0.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3NsYW5kYXJlcg==,size_16,color_FFFFFF,t_70#pic_center)


如下图所示，只要复制新内容(任何复制方式均可，包括直接ctrl+c的方式)，就会有一个翻译框蹦出来，显示句子的原文和翻译：

**效果动图：**

![在这里插入图片描述](https://img-blog.csdnimg.cn/dce4e0ef3669425b87d81274723b47f0.gif#pic_center)


即使最小化或者关闭翻译窗口，再次 [**划词**] 依旧会蹦出窗口


---
### 1如何获得剪切板内容？
这一部分是用的java的库，代码如下：

```java
import java.awt.Toolkit
import java.awt.datatransfer.DataFlavor

clip=Toolkit.getDefaultToolkit().getSystemClipboard();
clipTf=clip.getContents([]);
clipContent=clipTf.getTransferData(DataFlavor.stringFlavor)
```
---
### 2如何获得鼠标在全屏位置
这步是为了让翻译内容显示在鼠标附近，这步可以用java来写也可以用matlab来写：

**Java版本:**

```java
import java.awt.MouseInfo;
mousepoint=MouseInfo.getPointerInfo().getLocation();
```
**MATLAB版本:**

```java
root=get(0);
mousepoint=root.PointerLocation;
```
2021-8-5日改：
若MATLAB版本为2021a
则使用：

```java
root=get(0);
mousepoint=root.PointerLocation;
mousepoint=mousepoint./root.ScreenSize(3:4);
mousepoint=mousepoint.*[1280,720];
```

---
### 3如何翻译整段文字
参考大佬 **肆拾伍** 的博文：[matlab 使用有道翻译API （推荐！）](https://blog.csdn.net/qq_43157190/article/details/89400624)
过程描述一下即为：

```java
clipContent='剪切板里的内容'
website=['http://fanyi.youdao.com/openapi.do?keyfrom=cxvsdffd33&key=1310976914&type=data&doctype=xml&version=1.1&q=',clipContent,'&only=translate"'];
webContent=webread(website);
trans_begin=regexpi(webContent,'<paragraph><![CDATA[');
trans_end=regexpi(webContent,']></paragraph>');
transContent=webContent(trans_begin+20:trans_end-2);
```
也就是把链接中一部分改为想要翻译的文本，通过webread获取返回的结果，再通过正则表达将翻译部分切割出来。因为是用的有道德API因此程序需要连网使用。
___
### 4如何在点击关闭按钮时隐藏窗口而不是删除
假设我们创建了一个窗口：

```java
% figure窗口构建
transFig=uifigure('units','pixels');
transFig.Position=[10,65,300,200];
transFig.NumberTitle='off';
transFig.MenuBar='none';
transFig.Name='translation tool | by slandarer';
transFig.Color=[1,1,1];
transFig.Resize='off';
transFig.Visible='on';
```
我们可以为其创建 **CloseRequestFcn** 回调函数，并在其中只隐藏窗口而不删除窗口：

```java
% 通过设置回调函数，让点击右上角关闭时窗口隐藏而不是被删除
set(transFig,'CloseRequestFcn',@closeFig)
function closeFig(~,~)
    transFig.Visible='off';
end
```
2021-8-5日改：
若MATLAB版本为2021a
为了在新版本方便关闭程序：
我们将while循环的参数设为runflag，并编写如下代码：

```java
% 上下文菜单
ContextMenu=uicontextmenu(transFig);
Menu=uimenu(ContextMenu);
Menu.Text='关闭划词翻译工具';
set(Menu,'MenuSelectedFcn',@closeFig2)
function closeFig2(~,~)
    runflag=false;
    delete(transFig)
    clc;
end
transFig.ContextMenu=ContextMenu;
```
这样右键工具时就会出现关闭工具的选项：

![在这里插入图片描述](https://img-blog.csdnimg.cn/01a7bcbe7e204e6db431ce5e4984638c.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3NsYW5kYXJlcg==,size_16,color_FFFFFF,t_70#pic_center)



___
### 5如何让复制后窗口显示在最上方
transFig是我们之前创建的figure，我们将其窗口状态更改为全屏，这时候他就会位于最上方且全屏，此时再将窗口状态更改为正常状态，则窗口会在保持在最上方的同时退出全屏：
```java
transFig.WindowState='fullscreen';
transFig.WindowState='normal';
```
2021-8-5日改：
若MATLAB版本为2021a
则使用：

```java
transFig.Visible='off';
transFig.Visible='on';
```



### 6完整代码

```java
function copyTrans
import java.awt.Toolkit
import java.awt.datatransfer.DataFlavor
import java.awt.datatransfer.StringSelection


% 获取剪切板文本
% clip=Toolkit.getDefaultToolkit().getSystemClipboard();
% clipTf=clip.getContents([]);
% clipContent=clipTf.getTransferData(DataFlavor.stringFlavor)
clip=Toolkit.getDefaultToolkit().getSystemClipboard();

% 获取鼠标位置
% ---------------------------------------------------
% 方法一：
% import java.awt.MouseInfo;
% mousepoint=MouseInfo.getPointerInfo().getLocation();
% ---------------------------------------------------
% 方法二
% root=get(0);
% mousepoint=root.PointerLocation;

% 旧的剪切板内容：若是剪切板内容与之前不同则进行翻译过程
lastContent='translation tool made by slandarer';
tText=StringSelection(lastContent);
clip.setContents(tText,[])

warning off

% figure窗口构建
transFig=uifigure('units','pixels');
transFig.Position=[10,65,300,200];
transFig.NumberTitle='off';
transFig.MenuBar='none';
transFig.Name='translation tool | by slandarer';
transFig.Color=[1,1,1];
transFig.Resize='off';
transFig.Visible='on';

% 通过设置回调函数，让点击右上角关闭时窗口隐藏而不是被删除
set(transFig,'CloseRequestFcn',@closeFig)
function closeFig(~,~)
    transFig.Visible='off';
end

% 文本标签创建
oriTextAreaLabel=uilabel(transFig);
oriTextAreaLabel.FontSize=16;
oriTextAreaLabel.FontColor=[0.35,0.39,0.19];
oriTextAreaLabel.Position=[10 175 150 20];
oriTextAreaLabel.FontWeight='bold';
oriTextAreaLabel.Text=' 原文(Original text)';

transTextAreaLabel=uilabel(transFig);
transTextAreaLabel.FontSize=16;
transTextAreaLabel.FontColor=[0.35,0.39,0.19];
transTextAreaLabel.Position=[10 80 150 20];
transTextAreaLabel.FontWeight='bold';
transTextAreaLabel.Text=' 翻译(Translation)';
% 文本框创建
oriTextArea=uitextarea(transFig);
oriTextArea.FontSize=15;
oriTextArea.FontColor=[0.4 0.4 0.4];
oriTextArea.Position=[10 105 280 60];
oriTextArea.Value='translation tool made by slandarer';

transTextArea=uitextarea(transFig);
transTextArea.FontSize=15;
transTextArea.FontColor=[0.4 0.4 0.4];
transTextArea.Position=[10 10 280 60];
transTextArea.Value='翻译工具由slandarer';


while 1
    pause(0.5)
    clipTf=clip.getContents([]);
    clipContent=clipTf.getTransferData(DataFlavor.stringFlavor);
    if ~strcmp(lastContent,clipContent)
        % 通过接口获取翻译
        website=['http://fanyi.youdao.com/openapi.do?keyfrom=cxvsdffd33&key=1310976914&type=data&doctype=xml&version=1.1&q=',...
            clipContent,'&only=translate"'];
        webContent=webread(website);
        trans_begin=regexpi(webContent,'<paragraph><![CDATA[');
        trans_end=regexpi(webContent,']></paragraph>');
        transContent=webContent(trans_begin+20:trans_end-2);
        
        % 命令行展示
        disp(' ')
        disp('翻译：')
        disp(transContent)
        
        % 更改工具窗口的文本
        oriTextArea.Value=clipContent;
        transTextArea.Value=transContent; 
        
        % 将窗口显示在其他窗口上面
        transFig.Visible='on';
        transFig.WindowState='fullscreen';
        transFig.WindowState='normal';

        % 根据鼠标位置更改窗口位置
        root=get(0);
        mousepoint=root.PointerLocation;
        transFig.Position=[mousepoint(1),mousepoint(2)-200,300,200];
        
        lastContent=clipContent;
    end
end
end
```
若版本为2021a及以后，则使用：

```java
function copyTrans
import java.awt.Toolkit
import java.awt.datatransfer.DataFlavor
import java.awt.datatransfer.StringSelection
import java.awt.MouseInfo;

% 获取剪切板文本
% clip=Toolkit.getDefaultToolkit().getSystemClipboard();
% clipTf=clip.getContents([]);
% clipContent=clipTf.getTransferData(DataFlavor.stringFlavor)
clip=Toolkit.getDefaultToolkit().getSystemClipboard();

% 获取鼠标位置
% ---------------------------------------------------
% 方法一：
% import java.awt.MouseInfo;
% mousepoint=MouseInfo.getPointerInfo().getLocation();
% ---------------------------------------------------
% 方法二
% root=get(0);
% mousepoint=root.PointerLocation;

% 旧的剪切板内容：若是剪切板内容与之前不同则进行翻译过程
lastContent='translation tool made by slandarer';
tText=StringSelection(lastContent);
clip.setContents(tText,[])

warning off

% figure窗口构建
transFig=uifigure('units','pixels');
transFig.Position=[10,65,300,200];
transFig.NumberTitle='off';
transFig.MenuBar='none';
transFig.Name='translation tool | by slandarer';
transFig.Color=[1,1,1];
transFig.Resize='off';
transFig.Visible='on';

runflag=true;
% 通过设置回调函数，让点击右上角关闭时窗口隐藏而不是被删除
set(transFig,'CloseRequestFcn',@closeFig)
function closeFig(~,~)
    transFig.Visible='off';
end

% 文本标签创建
oriTextAreaLabel=uilabel(transFig);
oriTextAreaLabel.FontSize=16;
oriTextAreaLabel.FontColor=[0.35,0.39,0.19];
oriTextAreaLabel.Position=[10 175 150 20];
oriTextAreaLabel.FontWeight='bold';
oriTextAreaLabel.Text=' 原文(Original text)';

transTextAreaLabel=uilabel(transFig);
transTextAreaLabel.FontSize=16;
transTextAreaLabel.FontColor=[0.35,0.39,0.19];
transTextAreaLabel.Position=[10 80 150 20];
transTextAreaLabel.FontWeight='bold';
transTextAreaLabel.Text=' 翻译(Translation)';
% 文本框创建
oriTextArea=uitextarea(transFig);
oriTextArea.FontSize=15;
oriTextArea.FontColor=[0.4 0.4 0.4];
oriTextArea.Position=[10 105 280 60];
oriTextArea.Value='translation tool made by slandarer';

transTextArea=uitextarea(transFig);
transTextArea.FontSize=15;
transTextArea.FontColor=[0.4 0.4 0.4];
transTextArea.Position=[10 10 280 60];
transTextArea.Value='翻译工具由slandarer';

% 上下文菜单
ContextMenu=uicontextmenu(transFig);
Menu=uimenu(ContextMenu);
Menu.Text='关闭划词翻译工具';
set(Menu,'MenuSelectedFcn',@closeFig2)
function closeFig2(~,~)
    runflag=false;
    delete(transFig)
    clc;
end
transFig.ContextMenu=ContextMenu;

while runflag
    pause(0.5)
    clipTf=clip.getContents([]);
    clipContent=clipTf.getTransferData(DataFlavor.stringFlavor);
    if ~strcmp(lastContent,clipContent)
        % 通过接口获取翻译
        website=['http://fanyi.youdao.com/openapi.do?keyfrom=cxvsdffd33&key=1310976914&type=data&doctype=xml&version=1.1&q=',...
            clipContent,'&only=translate"'];
        webContent=webread(website);
        trans_begin=regexpi(webContent,'<paragraph><![CDATA[');
        trans_end=regexpi(webContent,']></paragraph>');
        transContent=webContent(trans_begin+20:trans_end-2);
        
        % 命令行展示
        disp(' ')
        disp('翻译：')
        disp(transContent)
        
        % 更改工具窗口的文本
        oriTextArea.Value=clipContent;
        transTextArea.Value=transContent; 
        
        % 将窗口显示在其他窗口上面
        transFig.Visible='off';
        transFig.Visible='on';
        %transFig.WindowState='fullscreen';
        %transFig.WindowState='normal';
        
        % 根据鼠标位置更改窗口位置
        
        root=get(0);
        mousepoint=root.PointerLocation;
        mousepoint=mousepoint./root.ScreenSize(3:4);
        mousepoint=mousepoint.*[1280,720];
        % root=get(0);
        % mousepoint=root.PointerLocation
        transFig.Position=[mousepoint(1),mousepoint(2)-200,300,200];
        
        lastContent=clipContent;
    end
end
end
```
