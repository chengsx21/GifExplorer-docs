# 需求实现

## 模块分析

采用模块化设计，将系统分为 Django 后端，Elastic Search 模块，Next.js 前端，Scrapy 爬虫四部分，最终得到如下架构：

![整体架构](images/整体架构.png)

## 功能分析

在本章节中，我们将通过泳道图展示各模块如何通信以满足用户需求：

### 用户管理相关

![用户相关](./images/泳道图-用户相关.svg)

### Gif 搜索相关

![Gif 搜索相关](./images/泳道图-搜索相关.svg)

### Gif 任务相关

![Gif 任务相关](./images/泳道图-任务相关.svg)

### Gif 爬取相关

![Gif 爬取相关](./images/泳道图-爬虫相关.svg)
