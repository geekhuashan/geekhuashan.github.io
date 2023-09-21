+++
title = '利用 Google Sheet自动创建Google Calendar来动态管理项目' 
date = 2023-09-19T16:20:16+08:00
lastmod = 2023-09-19T16:20:16+08:00
author = ["geekhuashan"]
draft = false
categories = ["life", "tech"]
tags = ["life", "tech"]
description = ""
weight = 1
hidemeta = false
+++


# 起因
由于我在项目管理时没有找到合适的工具，我的需求比较特殊，所以想利用Google Sheet、Google Calendar和Google App Script来自己写一个。实际上并不是我亲自写的，而是通过ChatGPT让它帮我写的。
这个工具的主要功能是将项目中非常细小的单元精确地显示在我的日历上。
由于我有很多样品需要跟踪其随时间变化后的稳定性，所以需要一个能够准确记录这些样品时间点并提醒我的工具。同时，这个工具还需要能够在我使用的Outlook上显示出来，这样我就可以在工作时看到项目进度了。

# 功能
通过Google Sheet公式，我可以填入制备样品的时间，并计算出样品需要观察的时间节点（如1天、3天、7天、14天、21天、28天）。然后通过Google App Script将这些时间节点及相关任务添加到我的Google Calendar中，并通过Outlook同步至我的电脑上。这样，在工作电脑上就能看到哪些样品需要跟踪了。
另外，由于经常根据TransREM负荷进行调整和改动，每天凌晨1点定时启动App Script获取当日信息并填入我的Google Calendar项目中。这样第二天早上就能在工作电脑上看到项目进展情况了。

# 实现步骤
## 1. 创建Google Sheet
根据自己的需求来创建表格来跟踪项目进度，这里我创建了一个表格来跟踪样品的制备进度。表格的内容如下：

![TransREM manangment](/tech/TransREM%20management.png)
 
## 2. Google app script code
从google sheet 顶部的 extension - Apps script 进入到google app script的编辑界面，然后将下面的代码复制进去，然后保存，然后运行，然后授权，然后就可以看到自己的日历中有了对应的事件了。

```javascript
function addComplexEventsToCalendar() {
  // 获取活动表格
  var sheet = SpreadsheetApp.getActiveSheet();
  
  // 读取C3:G100的日期数据
  var dateData = sheet.getRange('C3:H73').getValues();
  
  // 读取A列的内容
  var aColumn = sheet.getRange('A3:A73').getValues();
  
  // 读取第2行的内容
  var secondRow = sheet.getRange('C2:H2').getValues()[0];
  
  // 使用Calendar ID获取特定日历
  var calendar = CalendarApp.getCalendarById('cebe7a75ebba32d2898d1cd61aa32617cd40c21e81dff4e0e34ac1fbed1e687d@group.calendar.google.com');
  
  // 获取今天的日期
  var today = new Date();
  today.setHours(0, 0, 0, 0);
  
  // 遍历日期数据
  for (var i = 0; i < dateData.length; i++) {
    for (var j = 0; j < dateData[i].length; j++) {
      // 只处理非空日期
      if (dateData[i][j]) {
        var eventDate = new Date(dateData[i][j]);
        eventDate.setHours(0, 0, 0, 0);
        
        // 只处理当天的事件
        if (eventDate.getTime() === today.getTime()) {
          var eventTitle = aColumn[i][0] + ' ' + secondRow[j]; // 使用A列和第2行组合作为事件名称
          
          // 设置开始和结束时间
          var startTime = new Date(eventDate);
          startTime.setHours(9, 0, 0);
          
          var endTime = new Date(eventDate);
          endTime.setHours(11, 0, 0);
          
          // 创建事件
          calendar.createEvent(eventTitle, startTime, endTime);
        }
      }
    }
  }
}
```
## 3. 设置定时任务
在google app script的编辑界面中，点击左侧的时钟图标，然后设置定时任务，这里我设置的是每天凌晨1点执行一次，这样就能同步当天的工作任务到自己的Google Calendar中了。
![google calendar](/tech/google_calendar_from_google_sheet.png)

## 4. 设置Outlook同步
在Google Calendar中，点击的设置，然后选择对应的日历，然后选择Public address in iCal format，然后在Outlook中添加日历，然后选择From Internet，然后将复制的地址粘贴进去，然后就可以看到自己的Google Calendar中的事件了。

![outlook_from_google_calendar](/tech/outlook_from_google_calendar.png)

# 总结
这个工具的好处是可以将项目中非常细小的单元精确地显示在我的日历上，这样我就可以在工作时看到项目进度了。另外，由于经常根据TransREM负荷进行调整和改动，每天凌晨1点定时启动App Script获取当日信息并填入我的Google Calendar项目中。这样第二天早上就能在工作电脑上看到项目进展情况了。

自己找遍了整个项目管理工具，没有很方便的利用公式能计算时间的，说的就是你notion，notion的公式实在难用。日期的输入也得一个一个输入，非常的麻烦。

google sheet 与 google calendar 的联用实在太符合我的预期了。

还有一点就是在没有chatgpt的时候，我是不会写google app script的，现在有了chatgpt或者LLM的加持，我就可以写出这样的代码了，这样的代码对于我来说，是非常有用的，可以帮助我提高效率，让我更加专注于自己的工作。

话说，我这边文章就是在vscode里面，利用 Github copilot写的，它会自动给我补齐内容，让我很快的能够完成这篇文章，这样的效率提升，是非常的惊人的。