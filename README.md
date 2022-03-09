PowerBI的一些功能的备注，全靠搜索，有空加点关键字上去



[TOC]



# DAX新建表

```
Dax.新建表UnitTable = 
SELECTCOLUMNS(    //选择某张表，按上表头
    
//以下是新建表，但没有表头
    {
        ("K",1000),
        ("w",10000),
        ("M",1000000)
    },
//新建表

//按上表头
    "UnitName",[Value1],
    "UnitValue",[Value2]

)
```



# 日期表

以下生成表格挑个就好，内容再照着抄抄改成自己希望的格式，但其实由于实际情况的多变，很多情况下用excel自己做一张日期表似乎更好

## 第一种，直接指定日期

```
日期表 = ADDCOLUMNS (
CALENDAR (DATE(2018,1,1), DATE(2019,12,31)),
"年度", YEAR ( [Date] ),
"月份", FORMAT ( [Date], "MM" ),
"周",WEEKNUM([Date],2),
"年月", FORMAT ( [Date], "YYYY/MM" ),
"星期", WEEKDAY ( [Date],2 ) & "-" & FORMAT ( [Date], "dddd" ),
"季度", "Q" & FORMAT ( [Date], "Q" ),
"年份季度", FORMAT ( [Date], "YYYY" ) & "/Q" & FORMAT ( [Date], "Q" ))
```



## 第二种，用事实表中的日期，自动化程度高

```
日期表 =
GENERATE(
CALENDAR(
    MIN('事实表'[日期]),//注意：这里需要替换成你自己的数据
    MAX('事实表'[日期])//注意：这里需要替换成你自己的数据
),
VAR DA=[Date]
VAR YEAR=YEAR(DA)
VAR QUARTER="季度"&FORMAT(DA,"Q")
VAR MONTE=FORMAT(DA,"MM")&"月"
VAR DAY=DAY(DA)
VAR WEEKID=WEEKDAY(DA,2)
RETURN
ROW(
    "年度",YEAR,
    "季度",QUARTER,
    "月份",MONTE,
    "日",DAY,
    "年度季度",YEAR&QUARTER,
    "年度月份",YEAR&MONTE,
    "星期",WEEKID
)
)
```



# MOM、YTD

```
去年同期=CALCULATE([销售金额],SAMEPERIODLASTYEAR('日期表'[Date]))
```

```
去年同期=CALCULATE([销售金额],DATEADD('日期表'[Date],-1,YEAR))
```

```
月环比=CALCULATE([销售金额],DATEADD('日期表'[Date],-1,MONTH))
```

YTD年度累计

```
YTD = TOTALYTD([销售金额],'日期表'[Date])
```

YTD计算逻辑

```
YTD = 
CALCULATE(
[销售金额],
FILTER(
ALL('日期表'[Date]),
'日期表'[Date]<=MAX('日期表'[Date]) && YEAR('日期表'[Date])=YEAR(MAX('日期表'[Date]))
)
)
```

从指定日期开始年度累计，举例为从4月开始累计

```
FYTD = TOTALYTD([销售金额],'日期表'[Date],"3-31")
```



# 本地图像的使用

1. 图像先导入数据源

2. 直接导入并编辑

3. 保留Content列
4. 添加自定义列
5. 输入公式："data:image/jpeg;base64,"&Binary.ToText([Content])

该列就成为了图像URL了，到时候直接当图片用就可以了

仅适用于小图像，一般都是几十KB这样子，太大的会被裁剪，毕竟不用加载网络图片了



# 评分加相应数量的星星

使用Emoji图像

```
好评 = REPT("⭐",AVERAGE('门店评分'[评分]))
```

使用uincode代码作为图像

```
好评 = REPT(UNICHAR(9733),AVERAGE('门店评分'[评分]))
```

这个是模仿最高分5分，得分四分就是4颗实心星星，1颗空心星星

```
好评 = REPT(UNICHAR(9733),AVERAGE('门店评分'[评分]))&REPT(UNICHAR(9734),5-AVERAGE('门店评分'[评分]))
```



# SWITCH函数、指标更换

## 第一种，硬来，麻烦是麻烦，但清晰容易改

```
同期 = 
SWITCH(TRUE(),
SELECTEDVALUE('指标'[项目])="销售额",[销售额-同期],
SELECTEDVALUE('指标'[项目])="销售量",[销售量-同期],
SELECTEDVALUE('指标'[项目])="毛利额",[毛利额-同期],
SELECTEDVALUE('指标'[项目])="来客数",[来客数-同期],
SELECTEDVALUE('指标'[项目])="毛利率",DIVIDE([毛利额-同期],[销售额-同期]),
SELECTEDVALUE('指标'[项目])="客单价",DIVIDE([销售额-同期],[来客数-同期]),
SELECTEDVALUE('指标'[项目])="客单量",DIVIDE([销售额-同期],[销售量-同期]),
[销售额-同期])
```



换个思路，用参数，也可以参考，这个其实更方便
---------------------

```
同期 =

VAR UserCode = SELECTEDVALUE('指标'[项目])

VAR UserResult= 
	SWITCH(  TRUE() ,
	UserCode = "Sales" , [KPI.订单销售额]，
	UserCode = "Volume" , [KPI.销售数量]，
	UserCode = "Profit%" , [KPI.利润率]，
	BLANK()
	）
RETURN UserResult
```



# 中国地图#形状地图

下载地址



