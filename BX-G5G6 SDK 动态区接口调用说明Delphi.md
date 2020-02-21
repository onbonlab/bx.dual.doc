#  BX-G5/G6 SDK 动态区接口调用说明Delphi

## 文档适用者

通过SDK向仰邦控制卡发送显示内容的软件开发者；

本文档主要描述5代控制卡和6代控制卡的动态区的SDK接口使用；



## 动态区接口的使用

### 6代卡动态区接口使用步骤

#### 初始化SDK库

linux下不需要此步骤；windows平台需要；只需要初始化一次；

#### 配置参数

根据不同的接口函数参数要求，配置函数需要传递的参数值；

控制卡IP、端口；这2个函数每个动态区接口都需要；

对于发送信息的动态区接口还包括如下的参数要配置，这个参数有的是定义在一个结构体中；

- 屏幕类型：单色、双色、三色；
- 配置动态区参数：显示区的位置、大小、内容等

- 配置动态区内显示内容的属性：显示效果（移动速度等）、字体名称和大小等

#### 调用动态区接口函数

- 一次向一个动态区域发送多条要显示的文本或图片信息

  dynamicArea_AddAreaInfos_6G

  如果每次向相同ID的动态区中发送信息，则动态区只会显示最近一次发送的内容；

- 删除动态区

  dynamicArea_DelArea_6G

#### 释放SDK库

与初始化SDK库一一对应；linux下不需要；



### 6代卡动态区接口代码实例

#### 一次发送多条要显示的文本或图片信息

以一次向一个动态区域发送多条要显示的文本或图片信息的动态区接口dynamicArea_AddAreaInfos_6G为例，包括如下步骤：

##### 初始化SDK库

初始化SDK库函数声明：

```
  // 初始化动态库
  InitSdk: function(): integer; stdcall;
  // 释放动态库
  ReleaseSdk: procedure(); stdcall;
```

从c++ dll加载初始化函数：

```
  gbxconfigDll := LoadLibraryEx(BXCONFIGDLL, 0, LOAD_WITH_ALTERED_SEARCH_PATH);
  if gbxconfigDll < 32 then
    Exit; // 如果Dll无法加载则退出
  InitSdk := GetProcAddress(gbxconfigDll, 'bxDual_InitSdk');
```

执行初始化SDK库：

```
var
  err: Integer;
begin
  //初始化动态库
  err := InitSdk();
end;
```

##### 配置参数

- ###### 声明结构：字体信息

```

```

###### 声明函数传递参数数据结构

调用动态区接口dynamicArea_AddAreaInfos_6G时作为参数传递；

```
type
  DynamicAreaBaseInfo_6G = packed record

    nType: byte; // nType=1:文本； nType=2:图片；

    // PageStyle begin---------------
    DisplayMode: byte;  //显示模式：上移、左移、右移
    ClearMode: byte;	//清除模式
    Speed: byte;		//移动速度
    StayTime: word;		//停留时间
    RepeatTime: byte;	//重复次数
    // PageStyle End.

    // 文本显示内容和字体格式
    oFont: EQfontData;	//字体信息结构
    fontName: pbyte;	//字体名称
    strAreaTxtContent: pbyte;	//要显示的文本内容

    // 图片路径 begin---------
    filePath: pbyte;			//要显示的图片路径

  end;
```

###### 声明二维数组结构体

```
TDynamicAreaBaseInfo_6G = Array of DynamicAreaBaseInfo_6G;
MTDynamicAreaBaseInfo_6G = Array of TDynamicAreaBaseInfo_6G;
```



###### 动态区发送函数中用到的参数变量声明

```
var
  nRet: Integer;
  nPort: LongWord;
  nAreaID: byte;
  pIP: pansichar;
  eColor: E_ScreenColor_G56;
  oPicInfo, oPicInfo2, oPicInfo3: DynamicAreaBaseInfo_6G;

  stDynamicAreaBaseInfo_6G: MTDynamicAreaBaseInfo_6G;
  filestrA, filestrB, filestrC: ansistring;
  ip, filefont, strAreaTxtContent: ansistring;
  cnst_InfoCount: byte;
  RunMode, RelateAllPro, ImmePlay: byte;
  Timeout, RelateProNum, uAreaX, uAreaY, uWidth, uHeight: word;
  oFrame: EQareaframeHeader;
  RelateProSerial: word;
  oFont, oFont2: EQfontData;
  i: Integer;

```

参数变量赋值：

此处向一个动态区发送2个要显示的图片文件

```
begin
  nPort := 5005;
  nAreaID := 0;
  ip := ansistring(Edit1.Text);
  pIP := pansichar(ip);
  // 图片路径 begin---------
  filestrA := exepath + '1.png';
  filestrB := exepath + '2.png';
  filestrC := exepath + '3.png';
  filefont := 'simsun';
  strAreaTxtContent := '1234567890';
  // 删除所有动态区
  nRet := dynamicArea_DelArea_6G(pIP, 5005, $00FF);
  if nRet = 0 then
    FLogs.info('删除动态区域成功')
  else
    FLogs.info('删除动态区域失败：' + inttostr(nRet));
  eColor := E_ScreenColor_G56.eSCREEN_COLOR_SINGLE; // eSCREEN_COLOR_THREE;
  // New(oPicInfo);
  oPicInfo.nType := 2;
  oPicInfo.DisplayMode := 2;
  // DisplayMode = 0x04;		//显示方式:  0x00 –随机显示 0x01 –静止显示 0x02 –快速打出 0x03 –向左移动 0x04 –向左连移 0x05 –向上移动 0x06 –向上连移 0x07 –闪烁 ......
  // 0x25 –向右移动  0x26 –向右连移  0x27 –向下移动  0x28 –向下连移
  oPicInfo.ClearMode := 0; // 退出方式/清屏方式: 每一页的退出方式；
  oPicInfo.Speed := 1; // 速度等级
  oPicInfo.StayTime := 100; // 停留时间，单位为 10ms
  oPicInfo.RepeatTime := 1;
  oPicInfo.oFont.arrMode := Integer(eMULTILINE);
  oPicInfo.oFont.fontSize := 10;
  oPicInfo.oFont.Color := Integer(E_Color_G56.eRED);
  oPicInfo.oFont.fontBold := 0;
  oPicInfo.oFont.fontItalic := 0;
  oPicInfo.oFont.tdirection := Integer(pNORMAL);
  oPicInfo.oFont.txtSpace := 0;
  oPicInfo.oFont.Halign := 1;
  oPicInfo.oFont.Valign := 2;
  oPicInfo.fontName := pansichar(filefont);
  oPicInfo.strAreaTxtContent := pansichar(strAreaTxtContent);
  oPicInfo.filepath := pansichar(filestrA);

  oPicInfo2.nType := 2;
  oPicInfo2.DisplayMode := 2;
  oPicInfo2.ClearMode := 0; // 退出方式/清屏方式: 每一页的退出方式；
  oPicInfo2.Speed := 1; // 速度等级
  oPicInfo2.StayTime := 200; // 停留时间，单位为 10ms
  oPicInfo2.RepeatTime := 1;
  oPicInfo2.oFont.arrMode := Integer(eMULTILINE);
  oPicInfo2.oFont.fontSize := 10;
  oPicInfo2.oFont.Color := Integer(E_Color_G56.eRED);
  oPicInfo2.oFont.fontBold := 0;
  oPicInfo2.oFont.fontItalic := 0;
  oPicInfo2.oFont.tdirection := Integer(pNORMAL);
  oPicInfo2.oFont.txtSpace := 0;
  oPicInfo2.oFont.Halign := 1;
  oPicInfo2.oFont.Valign := 2;
  oPicInfo2.fontName := pansichar(filefont);
  oPicInfo2.strAreaTxtContent := pansichar(strAreaTxtContent);
  oPicInfo2.filepath := pansichar(filestrB);
```

定义并对二维数组结构体赋值

```
  stDynamicAreaBaseInfo_6G: MTDynamicAreaBaseInfo_6G;
  cnst_InfoCount := 2; // 单区域内有多少个数据单元内容
  SetLength(stDynamicAreaBaseInfo_6G, cnst_InfoCount);
  SetLength(stDynamicAreaBaseInfo_6G[0], 1);
  SetLength(stDynamicAreaBaseInfo_6G[1], 1);
  stDynamicAreaBaseInfo_6G[0][0] := &oPicInfo;
  stDynamicAreaBaseInfo_6G[1][0] := &oPicInfo2;
```



##### 调用发送动态区接口

```
  RunMode := 0;
  (* 动态区运行模式 ： 默认值 = 0 x00 0 — 动态区数据循环显示 。 
  1 — 动态区数据显示完成后静止显示最后一页数据 。 
  2 — 动态区数据循环显示 ， 超过设定时间后数据仍未更新时不再显示 
  3 — 动态区数据循环显示 ， 超过设定时间后数据仍未更新时显示Logo 信息, Logo 信息即为动态区域的最后一页信息 
  4 — 动态区数据顺序显示 ， 显示完最后一页后就不再显示 *)
  
  Timeout := 3; 
  // Timeout 2 动态区数据超时时间，单位为秒
  
  RelateAllPro := 1;
  //RelateAllPro 1 当该字节为 1 时，所有异步节目播放时都允许播放该动态区域；为 0 时，由接下来的规则来决定
  
  RelateProNum := 0;
  //动态区域关联了多少个异步节目一旦关联了某个异步节目，则当该异步节目播放时允许播放该动态区域，否则，不允许播放该动态区域；以下的节目编号个数根据 RelateProNum 的值来确定，当该值为 0 时不发送；
  
  RelateProSerial := 0;
  
  ImmePlay := 0;    
  //是否立即播放：该字节为 0 时，该动态区域与异步节目一起播放；
  该字节为 1 时，异步节目停止播放，仅播放该动态区域该字节; 
  为 2 时，暂存该动态区域，当播放完节目编号最高的异步节目后播放该动态区域注意：
  当该字节为 0 时，RelateAllPro 到RelateProSerialN-1 的参数才有效，否则无效当该参数为 1 或 2 时，由于不与异步节目同时播放，为控制该动态区域能及时结束，可选择RunMode 参数为 2 或 4，当然也
  
  uAreaX := 0;
  uAreaY := 0;
  uWidth := 168;
  uHeight := 64;

  oFrame.AreaFFlag := 0; // 边框默认无边框

  nRet := dynamicArea_AddAreaInfos_6G(pIP, 5005, Integer(eColor), nAreaID,
    RunMode, Timeout, RelateAllPro, RelateProNum, @RelateProSerial, ImmePlay,
    uAreaX, uAreaY, uWidth, uHeight, oFrame, cnst_InfoCount,
    stDynamicAreaBaseInfo_6G);
```



##### 释放SDK库

```
var
  err: Integer;
begin
  //初始化动态库
  err := ReleaseSdk();
end;
```



#### 动态区的删除

删除所有动态区：

```c++
  // 删除所有动态区
  nRet := dynamicArea_DelArea_6G(pIP, 5005, $00FF);
```


