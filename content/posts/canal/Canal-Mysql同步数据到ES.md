---
title: "Canal MySql 数据增量/全量同步数据到 ES"
---

# Canal MySql 数据增量/全量同步数据到 ES

- [Canal](https://github.com/alibaba/canal) 项目地址

## Server 部署

- 官方文档[QuickStart](https://github.com/alibaba/canal/wiki/QuickStart)

## ClientAdapter 部署

- 官方文档[ClientAdapter](https://github.com/alibaba/canal/wiki/ClientAdapter)

### 同步数据到 ES

- 官方文档[Sync-ES](https://github.com/alibaba/canal/wiki/Sync-ES)

## 遇到问题

- SQL 需要使用别名, 没有自动转换

- 但表查询也需要使用别名,没有会空指针

- 日期格式, 需要使用 ES 标准格式

  ```json
  "createTime": {
          "type": "date",
          "format": "yyyy-MM-dd HH:mm:ss||strict_date_optional_time||epoch_millis"
        }
  ```

- ETL `etlCondition`参数使用 `{}`, 传参使用 `;` 分割
