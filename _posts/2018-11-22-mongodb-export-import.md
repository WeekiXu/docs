---
title: MongoDB导入导出
categories:
 - java
tags:
 - MongoDB
---

## 导出csv
`mongoexport -d "estate" -c "mall" --csv -f "id","所在城市","name","url","商业楼层","连锁项目","上市企业","项目类型","开业时间","开发商","项目状态","商业建筑面积","项目地址","招商状态","项目简介","loc_bd" -o mall.csv`