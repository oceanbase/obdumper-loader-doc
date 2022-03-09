# 文件格式
本篇旨在介绍 OBDUMBER 支持的文件格式以及使用注意事项。
## SQL 格式和 DDL 格式的区别
- DDL 格式：导出的文件名是 对象名-schema.sql ，文件中存放的都是对象创建语句。
- SQL 格式：导出的文件名是 对象名.sql ，文件中存放的全部是 INSERT SQL 语句。
## 数据文件的格式与文件后缀名的对应关系
- OBDUMPER 支持 CSV、SQL、和 DDL 共三种格式。用户使用 OBDUMPER 时，仅需指定导出的数据格式，无需关注导出的文件名与后缀名。
- 默认导出的 CSV 文件名是 表名.序号.csv，默认导出的 SQL 文件名是 表名.序号.sql，导出的 DDL 文件名是 表名-schema.sql。
- 当使用外部工具导出时，请注意 CSV、SQL 和 DDL 格式的说明，以及最终输出的文件名与后缀的格式。
##　指定 `--table` 选项时的注意事项
参数 `--table` 用于指定的表名。
指定多个表时使用逗号分隔，且要求表名与文件名大小写保持一致。
例如：
MySQL 模式下，如默认表名为小写，此时要求 `--table` 选项的值为小写表名，参数 `-f` 指定的目录下的文件名同样应为小写。
## 导出表结构或数据时的注意事项
- 导出表结构时，如表很多，请勿开启太多并发，建议 `--threads` 控制在 4 以内。太高的并发没有意义，反而会加重 sys 租户访问内部视图的负担，进而出现导出超时报错。同时在开启并发时，`--threads` 默认会根据 CPU 动态调整，同时可通过手动设置以控制导出对主机性能的影响。
- 导出大量的数据时，可能需占用较多的 OBDUMPER 内存，可编辑 bin/obdumper 脚本，修改 JVM 参数 `--Xms4G` 和 `--Xmx4G`，分别表示 JVM 初始堆内存和最大堆内存，建议两者均设置为可用物理内存的 60%。
  ```JavaScript
  vim bin/obdumper
  50 JAVA_OPTS="$JAVA_OPTS -server -Xms4G -Xmx4G -XX:MetaspaceSize=512M -XX:MaxMetaspaceSize=512M -Xss352K"
  ```
## 运行 OBLOADER/OBDUMPER 脚本时，需添加单 / 双引号的参数
建议用户在一些复杂的参数上加上引号，例如：密码或集群名。
- Windows 平台参数使用双引号，例如：`--table "*"`；
- 类 Linux 平台参数使用单引号，例如：`--table '*'`。
## 公有云与私有云的区别 
- 公私有云的 proxyro 权限区别：
   - 公有云模式（限制模式）有 proxyro 权限限制，只能导出表数据，以及表和视图的结构；
   - 私有云模式（非限制模式）无 proxyro 权限限制，可导出数据库中所有数据库对象的结构和数据。
- 公私有云的连接方式区别：
   - 公有云模式：通过 --public-cloud 选项连接。
   - 私有云模式：通过 --sys-password 选项, 或提供秘钥文件连接。
     > **说明**
     > 限制模式（公有云连接模式）会限制部分导入的功能、性能以及稳定性。同时，OceanBase 数据库 V2.2.30 及以上版本才支持服务端限流的能力，因此在使用限制模式时通过以下命令调整服务端限流：
     > ```SQL
     > alter system set freeze_trigger_percentage=50;
     > alter system set minor_merge_concurrence=64;
     > alter system set writing_throttling_trigger_percentage=80 t
     > ```