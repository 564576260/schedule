# schedule

## Project setup
```
npm install
```

### Compiles and hot-reloads for development
```
npm run serve
```

### Compiles and minifies for production
```
npm run build
```

### Lints and fixes files
```
npm run lint
```

### Customize configuration
See [Configuration Reference](https://cli.vuejs.org/config/).


## 功能简介

### 基本功能
```
1.简单的日程管理组件，少量参数可自定义（每日起止时间，每周需要的天数，卡牌背景色） 
2.可以进行日程的增删改功能，点击日志可以查看部分操作的记录  
```

### 运行方式
```
下载项目  > npm install > npm run serve
```

### 未完善部分
```
1.卡片展开收起的过渡动画已完成，但存在删除时的错误判定问题，暂时隐藏，如需查看请恢复schedule.vue文件456行.card选择器下被注释的transition属性  
2.过小的卡片信息显示不全，暂未做样式优化  
3.修改日程时间时有时可选时间列表错误的加载了所有时间，为待解决的BUG  
4.相邻卡片颜色不能相同的判断逻辑未完成，暂时注释掉，位置在schedule.vue文件239-257行  
5.修改即保存，标题与内容没有设置保存键，故日志中没有记录修改事件  
6.当前最小时间段为15分钟，暂不支持自定义  
7.新增日程时默认为15分钟，逻辑（或样式？）尚待优化  
```