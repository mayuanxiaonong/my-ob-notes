---
{"dg-publish":true,"permalink":"/Homepage/"}
---


```dataview
list from "02 - Diary" sort file.mtime desc limit (5)
```

今天是`=dateformat(date(today),"DD")`，`=date(today).year` 年已经过去了 `=(date(today)-date(date(today).year + "-01-01")).days` 天

`=dv.pages("Omnivore")`

今年你总共在`$=dv.pages('"02 - Diary"').file.length`天中留下了你的足迹，继续加油！

