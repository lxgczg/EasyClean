# EasyClean
数据库自动清理空洞率工具，写的第一个作品，纪念一下。

# 本人CSDN博客
[阳光九叶草LXGZXJ](https://blog.csdn.net/qq_45111959?type=blog)

# 一、环境信息
名称|值
--|--
CPU|Intel(R) Core(TM) i5-1035G1 CPU @ 1.00GHz
操作系统|CentOS Linux release 7.9.2009 (Core)
内存|3G
逻辑核数|2
Gbase8a版本|8.6.2-R43.34.27468a27
EasyClean版本|V1.3

# 二、简述
工作和兴趣相结合的产物，既能更好的完成工作，也能看看自己的学习情况如何，无论如何，大家一起加油。

# 三、升级点
序号|功能|备注
--|--|--
1|提高清理空洞率的效率|将检索表空洞率的逻辑放到了子进程。
2|优化清理空洞率逻辑|

# 四、支持功能
序号|功能|备注
--|--|--
1|Gbase8a空洞率清理|

# 五、空洞率
大家可以参考之前的博客《[南大通用数据库-Gbase-8a-学习-33-空洞率查询与解决方法](https://blog.csdn.net/qq_45111959/article/details/129923944)》。

# 六、工具流程图
![清理空洞率工具流程图](https://github.com/lxgczg/EasyClean/blob/main/Photo/%E6%B8%85%E7%90%86%E7%A9%BA%E6%B4%9E%E7%8E%87%E5%B7%A5%E5%85%B7%E6%B5%81%E7%A8%8B%E5%9B%BE.PNG)

## 1、流程描述
（1）管理者进程检查传入参数是否正确。

（2）管理者进程获取环境变量和创建消息队列。

（3）管理者进程创建多个执行者进程。

（4）管理者进程向消息队列发送消息，当消息队列中的任务到达100件时，获取消息队列状态，管理者进程休眠5s，继续获取消息队列中的任务数。

（5）执行者进程检查传入参数是否正确。

（6）执行者连接数据库和获取环境变量、连接消息队列。

（7）执行者从消息队列中接收消息。

（8）执行者进程查看此表是否满足清理空洞率的标准。

（9）执行者进程操作数据库清理空洞率。

（10）管理者进程发送完所有清理的表，向消息队列发送完成任务消息。

（11）执行者进程接收到完成任务消息，清理申请的资源。

（12）管理者进程回收所有执行者进程的PCB资源。

（13）管理者进程关闭消息队列。

（14）管理者进程清理申请的资源。

## 2、注意点
如果大家遇到某些大表清理空洞率较慢，想终止此表的清理，推荐方法如下：

（1）方法一
登录Gbase8a，使用kill命令强杀SQL，需要杀三次相同的SQL，因为工具有错误重试机制。

（2）方法二
操作系统层杀死执行SQL的执行者进程，这样会影响清理空洞率的效率，个人更推荐方法一。

不建议大家杀死管理者进程，这样会导致执行者进程的PCB资源无法回收，变成僵尸进程，虽然操作系统的init进程会接管僵尸进程和释放其资源，但毕竟不由我们程序自己控制，不保险。还有可能导致消息队列没有清理，出现任务残留，影响下一次程序运行。

# 七、清理空洞率流程图
复制表的表名是在原表名基础上加上_COPY_TAB。
![清理空洞率流程图](https://github.com/lxgczg/EasyClean/blob/main/Photo/%E6%B8%85%E7%90%86%E7%A9%BA%E6%B4%9E%E7%8E%87%E6%B5%81%E7%A8%8B%E5%9B%BE.PNG)

# 八、安装包下载地址
[GITHUB-EasyClean-releases版本下载地址](https://github.com/lxgczg/EasyClean/releases/tag/EasyClean-V1.3)

# 九、参数介绍
## 1、命令模板
```
[gbase@czg2 Exec]$ ./Manager 'DbHost;DbUser;DbPwd;DbName;DbPort;DbCharset;ChdProcessNum;TargetDb;VoidRate;'
```

## 2、命令样例
```
[gbase@czg2 Exec]$ ./Manager '192.168.142.12;czg;qwer1234;gbase;5258;utf8;3;zxj;0;'
```

## 3、参数表格
序号|参数|备注
--|--|--
1	|DbHost	|连接源端数据库IP。
2	|DbUser	|连接源端数据库用户。
3	|DbPwd	|连接源端数据库用户密码。
4	|DbName	|连接源端数据库。
5	|DbPort	|连接源端数据库端口号。
6	|DbCharset	|连接源端数据库的字符集。
7	|ChdProcessNum	|启动的子进程数。
8	|TargetDb	|需要清理空洞率的数据库。
9	|VoidRate	|空洞率到达此值时进行清理，0-100。

# 十、安装步骤
## 1、配置环境变量
```
/home/gbase/.bashrc中添加如下

export CLEAN_VOID_RATE_TOOL_HOME=/home/gbase/EasyClean/
export LD_LIBRARY_PATH=$CLEAN_VOID_RATE_TOOL_HOME/Libs:$LD_LIBRARY_PATH
```

## 2、生效环境变量
```
source /home/gbase/.bashrc
```

## 3、检验动态链接是否正常
```
[gbase@czg2 Exec]$ ldd Manager 
        linux-vdso.so.1 =>  (0x00007ffffdda3000)
        libPublic.so => /opt/Developer/ComputerLanguageStudy/C/DataStructureTestSrc/PublicFunction/Cmake/Libs/libPublic.so (0x00007f6d9a6c3000)
        libLog.so => /opt/Developer/ComputerLanguageStudy/C/DataStructureTestSrc/PublicFunction/Cmake/Libs/libLog.so (0x00007f6d9a4be000)
        libSqQueue.so => /opt/Developer/ComputerLanguageStudy/C/DataStructureTestSrc/PublicFunction/Cmake/Libs/libSqQueue.so (0x00007f6d9a2b8000)
        libPthread.so => /opt/Developer/ComputerLanguageStudy/C/DataStructureTestSrc/PublicFunction/Cmake/Libs/libPthread.so (0x00007f6d9a0ab000)
        libFileOperate.so => /opt/Developer/ComputerLanguageStudy/C/DataStructureTestSrc/PublicFunction/Cmake/Libs/libFileOperate.so (0x00007f6d99ea5000)
        libDataConvertion.so => /opt/Developer/ComputerLanguageStudy/C/DataStructureTestSrc/PublicFunction/Cmake/Libs/libDataConvertion.so (0x00007f6d99ca1000)
        libProcess.so => /opt/Developer/ComputerLanguageStudy/C/DataStructureTestSrc/PublicFunction/Cmake/Libs/libProcess.so (0x00007f6d99a98000)
        libGbase8aDb.so => /opt/Developer/ComputerLanguageStudy/C/DataStructureTestSrc/PublicFunction/Cmake/Libs/libGbase8aDb.so (0x00007f6d99891000)
        libgbase.so.16 => /opt/Developer/ComputerLanguageStudy/C/DataStructureTestSrc/PublicFunction/Gbase8aDb/Libs/x86_64_linux/libgbase.so.16 (0x00007f6d993d1000)
        libHashTable.so => /opt/Developer/ComputerLanguageStudy/C/DataStructureTestSrc/PublicFunction/Cmake/Libs/libHashTable.so (0x00007f6d991cd000)
        libc.so.6 => /lib64/libc.so.6 (0x00007f6d98dff000)
        libpthread.so.0 => /lib64/libpthread.so.0 (0x00007f6d98be3000)
        libdl.so.2 => /lib64/libdl.so.2 (0x00007f6d989df000)
        libm.so.6 => /lib64/libm.so.6 (0x00007f6d986dd000)
        /lib64/ld-linux-x86-64.so.2 (0x00007f6d9a8c6000)

[gbase@czg2 Exec]$ ldd Executor 
        linux-vdso.so.1 =>  (0x00007fffee9f6000)
        libPublic.so => /opt/Developer/ComputerLanguageStudy/C/DataStructureTestSrc/PublicFunction/Cmake/Libs/libPublic.so (0x00007fd71cd14000)
        libLog.so => /opt/Developer/ComputerLanguageStudy/C/DataStructureTestSrc/PublicFunction/Cmake/Libs/libLog.so (0x00007fd71cb0f000)
        libSqQueue.so => /opt/Developer/ComputerLanguageStudy/C/DataStructureTestSrc/PublicFunction/Cmake/Libs/libSqQueue.so (0x00007fd71c909000)
        libPthread.so => /opt/Developer/ComputerLanguageStudy/C/DataStructureTestSrc/PublicFunction/Cmake/Libs/libPthread.so (0x00007fd71c6fc000)
        libFileOperate.so => /opt/Developer/ComputerLanguageStudy/C/DataStructureTestSrc/PublicFunction/Cmake/Libs/libFileOperate.so (0x00007fd71c4f6000)
        libDataConvertion.so => /opt/Developer/ComputerLanguageStudy/C/DataStructureTestSrc/PublicFunction/Cmake/Libs/libDataConvertion.so (0x00007fd71c2f2000)
        libProcess.so => /opt/Developer/ComputerLanguageStudy/C/DataStructureTestSrc/PublicFunction/Cmake/Libs/libProcess.so (0x00007fd71c0e9000)
        libGbase8aDb.so => /opt/Developer/ComputerLanguageStudy/C/DataStructureTestSrc/PublicFunction/Cmake/Libs/libGbase8aDb.so (0x00007fd71bee2000)
        libgbase.so.16 => /opt/Developer/ComputerLanguageStudy/C/DataStructureTestSrc/PublicFunction/Gbase8aDb/Libs/x86_64_linux/libgbase.so.16 (0x00007fd71ba22000)
        libHashTable.so => /opt/Developer/ComputerLanguageStudy/C/DataStructureTestSrc/PublicFunction/Cmake/Libs/libHashTable.so (0x00007fd71b81e000)
        libc.so.6 => /lib64/libc.so.6 (0x00007fd71b450000)
        libpthread.so.0 => /lib64/libpthread.so.0 (0x00007fd71b234000)
        libdl.so.2 => /lib64/libdl.so.2 (0x00007fd71b030000)
        libm.so.6 => /lib64/libm.so.6 (0x00007fd71ad2e000)
        /lib64/ld-linux-x86-64.so.2 (0x00007fd71cf17000)
```
如果有动态库没有找到，就要看看环境变量是否配置正确或是否生效。

# 十一、运行效果
```
[gbase@czg2 Exec]$ ./Manager '192.168.142.12;czg;qwer1234;gbase;5258;utf8;3;bd_db_apblc;0;' > CleanVoid.log &

2024-09-18 18:22:17-P[105057]-T[105057]-[Info ]-EasyClean-V1.3-Executor.
2024-09-18 18:22:18-P[105057]-T[105057]-[Info ]-G8aVoidRateCleanTb : OK, DbName : bd_db_apblc, TabName : d_mds_ind_lvl_count_rule.
2024-09-18 18:22:18-P[105057]-T[105057]-[Info ]-G8aVoidRateCleanTb : OK, DbName : bd_db_apblc, TabName : d_strategic_new_industry.
2024-09-18 18:22:22-P[105057]-T[105057]-[Info ]-G8aVoidRateCleanTb : OK, DbName : bd_db_apblc, TabName : gbase8a_i2b_config_table_2024_06_24_bak.
2024-09-18 18:22:27-P[105057]-T[105057]-[Info ]-G8aVoidRateCleanTb : OK, DbName : bd_db_apblc, TabName : gbase8a_i2b_config_table_2024_07_02_bak.
2024-09-18 18:22:31-P[105057]-T[105057]-[Info ]-G8aVoidRateCleanTb : OK, DbName : bd_db_apblc, TabName : sg_t_loadconfig_incr_odm_2024_06_24_bak.
2024-09-18 18:22:17-P[105058]-T[105058]-[Info ]-EasyClean-V1.3-Executor.
2024-09-18 18:22:18-P[105058]-T[105058]-[Info ]-G8aVoidRateCleanTb : OK, DbName : bd_db_apblc, TabName : d_mds_agent_cate.
2024-09-18 18:22:22-P[105058]-T[105058]-[Info ]-G8aVoidRateCleanTb : OK, DbName : bd_db_apblc, TabName : gbase8a_i2b_config_table_2024_06_20_bak.
2024-09-18 18:22:27-P[105058]-T[105058]-[Info ]-G8aVoidRateCleanTb : OK, DbName : bd_db_apblc, TabName : gbase8a_i2b_config_table_2024_06_25_bak.
2024-09-18 18:22:31-P[105058]-T[105058]-[Info ]-G8aVoidRateCleanTb : OK, DbName : bd_db_apblc, TabName : sg_t_loadconfig_incr_20230801.
2024-09-18 18:22:17-P[105059]-T[105059]-[Info ]-EasyClean-V1.3-Executor.
2024-09-18 18:22:18-P[105059]-T[105059]-[Info ]-G8aVoidRateCleanTb : OK, DbName : bd_db_apblc, TabName : d_mds_ind_coding_new.
2024-09-18 18:22:22-P[105059]-T[105059]-[Info ]-G8aVoidRateCleanTb : OK, DbName : bd_db_apblc, TabName : gbase8a_i2b_config_table.
2024-09-18 18:22:27-P[105059]-T[105059]-[Info ]-G8aVoidRateCleanTb : OK, DbName : bd_db_apblc, TabName : gbase8a_i2b_config_table_2024_06_27_bak.
2024-09-18 18:22:30-P[105059]-T[105059]-[Info ]-G8aVoidRateCleanTb : OK, DbName : bd_db_apblc, TabName : sg_t_loadconfig_incr_odm.
2024-09-18 18:22:31-P[105059]-T[105059]-[Info ]-G8aVoidRateCleanTb : OK, DbName : bd_db_apblc, TabName : sg_t_loadconfig_incr_odm_2024_06_25_bak.
2024-09-18 18:22:17-P[105053]-T[105053]-[Info ]-EasyClean-V1.3-Manager.
```

​
