---
title: mysql同步数据到elasticSearch
date: 2024-08-08
authors: ["Mireux"]
categories: ["ElasticSearch", "MySQL"]
---

## 前言

> 这里只是简单描述从mysql同步数据到elasticSearch 举一反三
> 其实只要希望是从mysql进行数据同步 都可以通过这个方法

<!-- more -->

## 为什么需要同步数据

mysql存放的大都都是存放一些业务上需要的数据，在某些方面可能会导致各种问题
举个🌰  
现在大数据量时需要进行分库分表 这就会导致如果你只在mysql层面进行搜索会非常的困难
又或者可能某些数据需要多维度的进行查询 需要存入到ES 可能需要将一些数据存入到缓存Redis中... 

## 几种方式

1. 同步双写

> 优点: 实时性高 <br>

> 缺点: <br>
>
> 1. 代码高度耦合 代码复杂程度提成
> 2. 系统可用性同时受多个数据源可用性影响 系统可用性降低
> 3. 同步双写事务问题 性能较低
>
> > 这里顺便提一句 es官方文档声明 es是一个NRT(Near Realtime)搜索和分析引擎 这主要是跟es的底层 索引刷新机制有关

2. 异步双写
   由于同步双写会造成系统可用性降低，那很容易想到优化方案，就是同步改成异步<br>
   这里其实有很多方案，最常见的就是加入消息队列进行消费

> 优点: <br>
>
> 1. 代码耦合性低 业务代码无需关注数据同步
> 2. 性能高 避免了双写的事务问题 系统可用性较高
> 3. 拓展性强 基于消息中间件的消息订阅 非常容易进行多数据源和异构数据的拓展

> 缺点: <br>
>
> 1. 数据实时性相对同步写稍低 
> 2. 引入其他中间件 增加系统复杂度


3. 定时同步

定时任务同步的实现方式，是基于对MySQL表进行定时扫描读取数据后，写入到ElasticSearch中

> 优点: <br>
>
> 1. 实现简单 对原有代码无侵入性

> 缺点: <br>
>
> 1. 实时性取决与定时任务的周期 实时性难以保证
> 2. 当对实时性有较高要求时 对MySQL会造成较大的压力 可能导致系统整体可用性下降

4. binlog订阅同步

binlog是MySQL的逻辑日志 可以用来进行数据的同步和复制 MySQL主从同步就是基于binlog日志实现的 MySQL与ElasticSearch间的数据同步也可以基于binlog日志 该方案需要引入一个中间件工具 作用是伪装成MySQL从库 接收主库binlog 然后同步到的其他数据源 市面上常见中间件工具如Flink-CDC、Canal、Otter、DataX等等 本篇文章主要介绍Canal

> 优点: <br>
>
> 1. 对业务代码的入侵极少 开发过程中无感知
> 2. 数据实时性较高

> 缺点: <br>
>
> 1. 需要引入中间件 架构上增加了复杂度 且同时增加了维护成本

## canal配置

在MySQL的配置文件my.cnf中添加如下配置：

```shell
find / -name my.cnf # 查找到对应的配置文件位置
log-bin=mysql-bin # 开启二进制日志
binlog-format=ROW #使用row模式
server-id=1 #配置主数据库id，不能与从数据库重复
```

MySQL的主从复制分为三种模式：

Statement，基于语句的复制，在简单来说，就是在主库上执行的SQL，在从库上原样地执行一遍；

Row，基于行的复制，记录每次DML操作导致的数据行变化，并把行的变更同步到从库中；

Mixed，混合模式，根据执行的每一条具体SQL来选择Statement模式或者Rows模式。

使用Canal时建议使用Row模式。

> 这里可以选择新建一个用户或者直接使用root用户 这里偷懒就直接使用root

可以根据`show variables like 'binlog_format%'; `查看是否开启了binlog模式

从canal官网下载canal.deployer-1.1.8.tar.gz，解压后 在目录conf/example下面有一个文件instance.properties，然后修改里面的数据库相关配置信息，主要是以下几项（IP地址、端口号修改成实际的配置）:

```
# 数据库地址
canal.instance.master.address=MySQL服务器的IP地址:端口号
# 用户名/密码，数据库字符集
canal.instance.dbUsername=root
canal.instance.dbPassword=password 
canal.instance.connectionCharset = UTF-8
```

对于服务器的配置canal.properties，可以先不用修改。

接下来我们写一段示例代码，如果连接的MySQL中数据表数据发生了变化，那么在Java程序监听到这种变化并做相应的业务处理。首先引入Canal客户端的依赖:

```xml
 <dependency>
    <groupId>com.alibaba.otter</groupId>
    <artifactId>canal.client</artifactId>
    <version>1.1.7</version>
 </dependency>
<dependency>
    <groupId>com.alibaba.otter</groupId>
    <artifactId>canal.protocol</artifactId>
    <version>1.1.7</version>
</dependency>
```

具体实现代码如下：

```java
package com.ruoyi.canal;

import com.alibaba.otter.canal.client.CanalConnector;
import com.alibaba.otter.canal.client.CanalConnectors;
import com.alibaba.otter.canal.protocol.CanalEntry;
import com.alibaba.otter.canal.protocol.Message;
import com.ruoyi.config.CanalConfig;
import com.ruoyi.service.impl.esService;
import lombok.extern.slf4j.Slf4j;
import org.springframework.stereotype.Component;

import java.net.InetSocketAddress;
import java.util.HashMap;
import java.util.List;
import java.util.Map;

import static com.ruoyi.common.constant.SedinnbConstants.TABLE_NAME;

/**
 * @author lianghaijie
 */
@Component
@Slf4j
public class CanalClient {

    private final static int BATCH_SIZE = 1000;

    private final EsService esService;

    // 获取配置文件中的canal 配置
    private final CanalConfig config;

    public CanalClient(esService esService, CanalConfig config) {
        this.esService = esService;
        this.config = config;
    }

    public void run() {
        // 创建链接
        CanalConnector connector = CanalConnectors.
                newSingleConnector(new InetSocketAddress(config.getHost(), config.getPort()),
                        config.getDestination(),
                        config.getUserName(),
                        config.getPassword());
        try {
            //打开连接
            connector.connect();
            //订阅数据库表,全部表
            connector.subscribe("*");
            //回滚到未进行ack的地方，下次fetch的时候，可以从最后一个没有ack的地方开始拿
            connector.rollback();
            while (true) {
                // 获取指定数量的数据
                Message message = connector.getWithoutAck(BATCH_SIZE);
                //获取批量ID
                long batchId = message.getId();
                //获取批量的数量
                int size = message.getEntries().size();
                //如果没有数据
                if (batchId == -1 || size == 0) {
                    try {
                        //线程休眠2秒
                        Thread.sleep(2000);
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                } else {
                    //如果有数据,处理数据
                    printEntry(message.getEntries());
                }
                //进行 batch id 的确认。确认之后，小于等于此 batchId 的 Message 都会被确认。
                connector.ack(batchId);
            }
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            connector.disconnect();
        }
    }

    /**
     * 打印canal server解析binlog获得的实体类信息
     */
    private void printEntry(List<CanalEntry.Entry> entryList) {
        for (CanalEntry.Entry entry : entryList) {
            if (entry.getEntryType() == CanalEntry.EntryType.TRANSACTIONBEGIN || entry.getEntryType() == CanalEntry.EntryType.TRANSACTIONEND) {
                //开启/关闭事务的实体类型，跳过
                continue;
            }
            //RowChange对象，包含了一行数据变化的所有特征
            //比如isDdl 是否是ddl变更操作 sql 具体的ddl sql beforeColumns afterColumns 变更前后的数据字段等等
            CanalEntry.RowChange rowChange;
            try {
                rowChange = CanalEntry.RowChange.parseFrom(entry.getStoreValue());
            } catch (Exception e) {
                throw new RuntimeException("ERROR # parser of eromanga-event has an error , data:" + entry, e);
            }
            //获取操作类型：insert/update/delete类型
            CanalEntry.EventType eventType = rowChange.getEventType();
            //打印Header信息
            System.out.printf("================》; binlog[%s:%s] , name[%s,%s] , eventType : %s%n",
                    entry.getHeader().getLogfileName(), entry.getHeader().getLogfileOffset(),
                    entry.getHeader().getSchemaName(), entry.getHeader().getTableName(),
                    eventType);
            //判断是否是DDL语句
            if (rowChange.getIsDdl()) {
                // 后续可以通知管理员 表级别的变动
                esService.clearCache(rowChange.getDdlSchemaName());
                System.out.println("================》;isDdl: true,sql:" + rowChange.getSql());
            }
            //获取RowChange对象里的每一行数据，打印出来
            for (CanalEntry.RowData rowData : rowChange.getRowDatasList()) {
                if (eventType == CanalEntry.EventType.DELETE) {
                    //如果是删除语句
                    printColumn(rowData.getBeforeColumnsList());
                    changeToEs(entry.getHeader().getTableName(), rowData.getBeforeColumnsList(), CanalEntry.EventType.DELETE);
                } else if (eventType == CanalEntry.EventType.INSERT) {
                    //如果是新增语句
                    printColumn(rowData.getAfterColumnsList());
                    changeToEs(entry.getHeader().getTableName(), rowData.getAfterColumnsList(), CanalEntry.EventType.INSERT);
                } else {
                    //如果是更新的语句
                    //变更前的数据
                    System.out.println("------->; before");
                    printColumn(rowData.getBeforeColumnsList());
                    //变更后的数据
                    System.out.println("------->; after");
                    printColumn(rowData.getAfterColumnsList());
                    changeToEs(entry.getHeader().getTableName(), rowData.getAfterColumnsList(), CanalEntry.EventType.UPDATE);
                }
            }
        }
    }

    private void changeToEs(String tableName, List<CanalEntry.Column> afterColumnsList, CanalEntry.EventType operation) { 
        Map<String, String> params = new HashMap<>();
        switch (operation) {
            case INSERT:
                esService.addToEs(params);
                break;
            case UPDATE:
                params.put(TABLE_NAME, tableName);
                esService.updateToEs(tableName, id, params);
                break;
            case DELETE:
                esService.deleteToEs(tableName, id);
                break;
            default:
                break;

        }
    }

    private static void printColumn(List<CanalEntry.Column> columns) {
        for (CanalEntry.Column column : columns) {
            System.out.println(column.getName() + " : " + column.getValue() + "    update=" + column.getUpdated());
        }
    }
}

```

## 