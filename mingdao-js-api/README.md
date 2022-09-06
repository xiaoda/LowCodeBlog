# 前言
随着明道项目构建的不断深入，项目的复杂度与自定义要求越来越高，常规的工作流节点慢慢变得不够用，这时就需要 “代码块” 或 “发送API请求” 节点来对项目数据进行自定义处理。

目前明道支持 JS (v10.16.3) 和 Python (v3.7.5) 两种语言的代码块节点，结合公司前端技术组的现状，JS开发人员众多，所以JS脚本无疑是最佳选择。到目前为止，我们接到过公司IT部门及项目管理应用的JS代码需求，处理日期时间相关统计及数据格式转换等。

此外，考虑到JS函数代码的易用和可维护性，我们额外提供了函数API及文档（部署于腾讯云Serverless服务），方便在明道工作流中使用 “发送API请求” 节点调用。

# JS函数
每一个函数都是为了解决具体问题，因此普适性可能不强。如有类似代码需求，可考虑对函数代码进行改写，或联系我们前端技术组。

## 计算工作时间
获取两个日期时间之间的工作时长，可设置自定义上下班时间。
``` javascript
var DateTime1 = '2022-7-1 9:00:00'
var DateTime2 = '2022-7-1 11:00:00'
var SkipPeriod1 = {
  start: '9:00:00',
  duration: 3 * 60 * 60 // 单位：秒
}
var SkipPeriod2 = {
  start: '13:00:00',
  duration: 5 * 60 * 60 // 单位：秒
}

function getSkippedTimeStamp (SkipPeriod) {
  var date1 = DateTime1.split(' ')[0]
  var loopStartDateTime = date1 + ' ' + SkipPeriod.start
  var loopStartTimestamp = new Date(loopStartDateTime).getTime()
  var skippedTimestamp = 0
  for (i = 0; i < 999; i++) {
    if (loopStartTimestamp > timestamp2) break
    if (loopStartTimestamp > timestamp1) {
      var tempSkipped = SkipPeriod.duration * 1000
      var tempSkippedLimit = timestamp2 - loopStartTimestamp
      if (tempSkipped > tempSkippedLimit) tempSkipped = tempSkippedLimit
      skippedTimestamp += tempSkipped
    } else if (loopStartTimestamp + SkipPeriod.duration * 1000 > timestamp1) {
      var tempSkipped = loopStartTimestamp + SkipPeriod.duration * 1000 - timestamp1
      var tempSkippedLimit = timestamp2 - timestamp1
      if (tempSkipped > tempSkippedLimit) tempSkipped = tempSkippedLimit
      skippedTimestamp += tempSkipped
    }
    loopStartTimestamp += 24 * 60 * 60 * 1000
  }
  return skippedTimestamp
}

var timestamp1 = new Date(DateTime1).getTime()
var timestamp2 = new Date(DateTime2).getTime()
var workingTimestamp = getSkippedTimeStamp(SkipPeriod1) + getSkippedTimeStamp(SkipPeriod2)
var seconds = workingTimestamp / 1000
var minutes = seconds / 60
var hours = minutes / 60

console.log(hours)
```

## 获取某个月第一天及最后一天的日期
``` javascript
var Separator = '/'

function getFirstDateOfMonth(year, month) {
  var date = year + Separator + month + Separator + '1'
  return date
}

function getLastDateOfMonth(year, month) {
  year = Number(year)
  month = Number(month)
  var tempYear = year
  var tempMonth = month + 1
  if (tempMonth > 12) {
    tempYear += 1
    tempMonth = 1
  }
  var tempDateObj = new Date(tempYear + '/' + tempMonth + '/' + '1')
  var dateObj = new Date(Date.parse(tempDateObj) - 24 * 60 * 60 * 1000)
  var targetYear = dateObj.getFullYear()
  var targetMonth = dateObj.getMonth() + 1
  var targetDay = dateObj.getDate()
  var date = targetYear + Separator + targetMonth + Separator + targetDay
  return date
}

var firstDate = getFirstDateOfMonth('2022', '3')
var lastDate = getLastDateOfMonth('2022', '3')

console.log(firstDate, lastDate)
```

## 获取某个时间点对应一天中的部分
可自定义设置一天中多个部分的起始、结束时间

``` javascript
var Division = {
  600: '凌晨',
  1300: '上午',
  2400: '下午'
}

function getPartOfDay (dateTime) {
  var formatedTime = Number(dateTime.split(' ')[1].replace(/:/g, ''))
  var part
  for (threshold in Division) {
    if (formatedTime <= threshold) {
      part = Division[threshold]
      break
    }
  }
  return part
}

var dateTime = '2022/8/1 14:00'
console.log(getPartOfDay(dateTime))
```

## 将数组格式数据转换为JS对象
方便在明道工作流节点中直接获取

``` javascript
function arrayToObject (array, fields) {
  const object = {}
  const keyIndex = fields.indexOf('key')
  array.forEach(innerArray => {
    const key = innerArray[keyIndex]
    object[key] = {}
    innerArray.forEach((item, index) => {
      if (index === keyIndex) return
      object[key][fields[index]] = item
    })
  })
  return object
}
```

# API及文档
## 基本信息
项目地址：[Github](https://github.com/xiaoda/JSUtils)

文档地址：[Docs](https://service-m98cme2x-1252837186.sh.apigw.tencentcs.com/apidoc/)

后端框架：Node.js Express

部署服务：腾讯云 Serverless 函数服务

单元测试：包含

## 文档截图
![文档截图](https://xiaoda.github.io/data/mingdao/images/ApiDocs.png)

## 明道工作流使用截图
![明道工作流使用](https://xiaoda.github.io/data/mingdao/images/WorkFlow.png)

## 单元测试截图
![单元测试](https://xiaoda.github.io/data/mingdao/images/UnitTest.png)

# 写在最后
明道作为公司低（无）代码项目交付的重要工具，需要我们在功能及方案上不断探索。代码节点作为工作流中的一个重要功能，值得去做一定的技术积累。前端技术组以JS为主力开发语言，在为明道代码节点提供函数方法方面具备先天优势，在此我们欢迎所有明道开发同学向我们提出代码需求，帮助项目开发的同时也是在帮助低（无）代码技术积累。

***Please reach out to us. You are more than welcomed!***
