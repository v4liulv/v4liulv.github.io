---
title: "PowerJob容器开发集成DataX及任务配置"
subtitle: "「PowerJob容器V1.0.1」- DataX3"
layout: post
author: "LiuL"
header-style: text
tags:
  - PowerJob
  - DataX
  - 数字贵阳
---

# 1. DataX开发说明

DataX二次开发主要是插件扩展开发（集成支持TBDS腾讯大数据套件）和PowerJob调度平台集成DataX开发。

DataX插件扩展开发相关内容请参考[DataX插件开发宝典](https://github.com/alibaba/DataX/blob/master/dataxPluginDev.md)和[DataX集成支持TBDS腾讯大数据套件插件开发](https://liulv.work/2020/08/14/datax-bigdata-tbds/)。

本文档主要是介绍PowerJob调度平台集成DataX开发及如何在PowerJob调度平台配置DataX JOB。

# 2. Java容器集成DataX实现

Java集成DataX开发主要包含：

1. 本地编译打包部署DataX的Maven本地私服
2. PowerJob集成DataX开发

## 2.1. 本地编译打包部署DataX的Maven本地私服

DataX没有提供远程的Maven库，如果需要Java集成Datax开发，需要Maven本地库拥有DataX的Core模块依赖，实现步骤：

* **step1:** 通过git clone DataX项目[源码](https://gitlab.tcfuture.tech/szgy/caelum-datax)

```powershell
git clone https://gitlab.tcfuture.tech/szgy/caelum-datax.git
```

* **step2:** IDEA开发工具打开DataX工程

* step3: IDEA新建Maven编译Run Conigurations, 如下图

![1597312511(1)](https://liulv.work/images/img/1597312511(1).jpg)

* **step4:** 执行编译打包

* **step5:** 通过命令`Maven package`打包发布到本地Maven库

* **step6:**检查本地Maven库是否存在DataX依赖`com\alibaba\datax`, 如下图证明已经编译打包到本地Maven库

![20200813181302](https://liulv.work/images/img/20200813181302.png)

## 2.2. PowerJob集成DataX开发

### 2.2.1. PowerJob容器模块添加DataX依赖

Java集成Data需要根据上一章节[Java集成前提条件](#21-本地编译打包部署datax的maven本地私服)的本地Maven库添加datax-core模块依赖，如下：

```xml
<dependency>
    <groupId>com.alibaba.datax</groupId>
    <artifactId>datax-core</artifactId>
    <version>0.0.1-SNAPSHOT</version>
</dependency>
```

### 2.2.2. DataX二次开发改造

DataX执行都是运行结果都是后台打印，需要改造执行后可获取DataX执行JOB后结果信息。DataX集成调用主要是通过DataX的入口类`com.alibaba.datax.core.Engine`的`entry`方法调用，可在Engine类改造获取执行结果信息。

Engine添加静态的Communication变量

```java
private static Communication result;
```

在容器启动前添加Start通讯信息，为后面作为记录转换开始时间和耗时使用。

```java
 //启动前设置通讯信息
Communication startCommunication = new Communication();
startCommunication.setTimestamp(System.currentTimeMillis());     
```

在容器启动后添加End通讯信息，并整合启动前的通讯信息和启动后的通讯信息并赋值给Communication变量

```java
 //设置启动后通讯信息
Communication endCommunication = container.getContainerCommunicator().collect();
endCommunication.setTimestamp(System.currentTimeMillis());
//整合返回通讯信息
result = getReportCommunication(endCommunication, startCommunication, 1);

//整合返回通讯信息
result = getReportCommunication(endCommunication, startCommunication, 1);
```

Engine的`entry`方法中暂定解析job目录文件方式

```java
Configuration configuration = ConfigParser.parseFile(jobPath);
```

Engine添加获取通讯信息静态方法

```java
public static Communication getResult(){
        return result == null ? new Communication(): result;
}
```

最终代码如下：

```java
package com.alibaba.datax.core;

import com.alibaba.datax.common.element.ColumnCast;
import com.alibaba.datax.common.exception.DataXException;
import com.alibaba.datax.common.spi.ErrorCode;
import com.alibaba.datax.common.statistics.PerfTrace;
import com.alibaba.datax.common.statistics.VMInfo;
import com.alibaba.datax.common.util.Configuration;
import com.alibaba.datax.core.job.JobContainer;
import com.alibaba.datax.core.statistics.communication.Communication;
import com.alibaba.datax.core.taskgroup.TaskGroupContainer;
import com.alibaba.datax.core.util.ConfigParser;
import com.alibaba.datax.core.util.ConfigurationValidate;
import com.alibaba.datax.core.util.ExceptionTracker;
import com.alibaba.datax.core.util.FrameworkErrorCode;
import com.alibaba.datax.core.util.container.CoreConstant;
import com.alibaba.datax.core.util.container.LoadUtil;
import org.apache.commons.cli.BasicParser;
import org.apache.commons.cli.CommandLine;
import org.apache.commons.cli.Options;
import org.apache.commons.lang3.StringUtils;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import java.util.Arrays;
import java.util.List;
import java.util.Set;
import java.util.regex.Matcher;
import java.util.regex.Pattern;

import static com.alibaba.datax.core.statistics.communication.CommunicationTool.getReportCommunication;

/**
 * Engine是DataX入口类，该类负责初始化Job或者Task的运行容器，并运行插件的Job或者Task逻辑
 */
public class Engine {
    private static final Logger LOG = LoggerFactory.getLogger(Engine.class);

    private static String RUNTIME_MODE;

    private static Communication result;

    /* check job model (job/task) first */
    public void start(Configuration allConf) {

        // 绑定column转换信息
        //对StringCast、DateCast、BytesCast三个实体类初始化
        ColumnCast.bind(allConf);

        /**
         * 初始化PluginLoader，可以获取各种插件配置
         */
        LoadUtil.bind(allConf);

        boolean isJob = !("taskGroup".equalsIgnoreCase(allConf
                .getString(CoreConstant.DATAX_CORE_CONTAINER_MODEL)));
        //JobContainer会在schedule后再行进行设置和调整值
        int channelNumber =0;
        AbstractContainer container;
        long instanceId;
        int taskGroupId = -1;
        if (isJob) {
            allConf.set(CoreConstant.DATAX_CORE_CONTAINER_JOB_MODE, RUNTIME_MODE);
            container = new JobContainer(allConf);
            instanceId = allConf.getLong(
                    CoreConstant.DATAX_CORE_CONTAINER_JOB_ID, 0);

        } else {
            container = new TaskGroupContainer(allConf);
            instanceId = allConf.getLong(
                    CoreConstant.DATAX_CORE_CONTAINER_JOB_ID);
            taskGroupId = allConf.getInt(
                    CoreConstant.DATAX_CORE_CONTAINER_TASKGROUP_ID);
            channelNumber = allConf.getInt(
                    CoreConstant.DATAX_CORE_CONTAINER_TASKGROUP_CHANNEL);
        }

        //缺省打开perfTrace
        boolean traceEnable = allConf.getBool(CoreConstant.DATAX_CORE_CONTAINER_TRACE_ENABLE, true);
        boolean perfReportEnable = allConf.getBool(CoreConstant.DATAX_CORE_REPORT_DATAX_PERFLOG, true);

        //standlone模式的datax shell任务不进行汇报
        if(instanceId == -1){
            perfReportEnable = false;
        }

        int priority = 0;
        try {
            priority = Integer.parseInt(System.getenv("SKYNET_PRIORITY"));
        }catch (NumberFormatException e){
            LOG.warn("prioriy set to 0, because NumberFormatException, the value is: "+System.getProperty("PROIORY"));
        }

        Configuration jobInfoConfig = allConf.getConfiguration(CoreConstant.DATAX_JOB_JOBINFO);
        //初始化PerfTrace
        PerfTrace perfTrace = PerfTrace.getInstance(isJob, instanceId, taskGroupId, priority, traceEnable);
        perfTrace.setJobInfo(jobInfoConfig,perfReportEnable,channelNumber);
        //启动前设置通讯信息
        Communication startCommunication = new Communication();
        startCommunication.setTimestamp(System.currentTimeMillis());
        //启动
        container.start();
        //设置启动后通讯信息
        Communication endCommunication = container.getContainerCommunicator().collect();
        endCommunication.setTimestamp(System.currentTimeMillis());
        //整合返回通讯信息
        result = getReportCommunication(endCommunication, startCommunication, 1);
    }


    // 注意屏蔽敏感信息
    public static String filterJobConfiguration(final Configuration configuration) {
        Configuration jobConfWithSetting = configuration.getConfiguration("job").clone();

        Configuration jobContent = jobConfWithSetting.getConfiguration("content");

        filterSensitiveConfiguration(jobContent);

        jobConfWithSetting.set("content",jobContent);

        return jobConfWithSetting.beautify();
    }

    public static Configuration filterSensitiveConfiguration(Configuration configuration){
        Set<String> keys = configuration.getKeys();
        for (final String key : keys) {
            boolean isSensitive = StringUtils.endsWithIgnoreCase(key, "password")
                    || StringUtils.endsWithIgnoreCase(key, "accessKey");
            if (isSensitive && configuration.get(key) instanceof String) {
                configuration.set(key, configuration.getString(key).replaceAll(".", "*"));
            }
        }
        return configuration;
    }

    public static void entry(final String[] args) throws Throwable {
        Options options = new Options();
        options.addOption("job", true, "Job config.");
        options.addOption("jobid", true, "Job unique id.");
        options.addOption("mode", true, "Job runtime mode.");

        BasicParser parser = new BasicParser();
        //对象值初始化
        CommandLine cl = parser.parse(options, args);

        String jobPath = cl.getOptionValue("job");

        // 如果用户没有明确指定jobid, 则 datax.py 会指定 jobid 默认值为-1
        String jobIdString = cl.getOptionValue("jobid");
        RUNTIME_MODE = cl.getOptionValue("mode");

        LOG.info("job path：{}, jobId：{}, mode：{}", jobPath, jobIdString, RUNTIME_MODE);

        Configuration configuration = ConfigParser.parseFile(jobPath);

        long jobId;
        if (!"-1".equalsIgnoreCase(jobIdString)) {
            jobId = Long.parseLong(jobIdString);
        } else {
            // only for dsc & ds & datax 3 update
            String dscJobUrlPatternString = "/instance/(\\d{1,})/config.xml";
            String dsJobUrlPatternString = "/inner/job/(\\d{1,})/config";
            String dsTaskGroupUrlPatternString = "/inner/job/(\\d{1,})/taskGroup/";
            List<String> patternStringList = Arrays.asList(dscJobUrlPatternString,
                    dsJobUrlPatternString, dsTaskGroupUrlPatternString);
            jobId = parseJobIdFromUrl(patternStringList, jobPath);
        }

        boolean isStandAloneMode = "standalone".equalsIgnoreCase(RUNTIME_MODE);
        if (!isStandAloneMode && jobId == -1) {
            // 如果不是 standalone 模式，那么 jobId 一定不能为-1
            throw DataXException.asDataXException(FrameworkErrorCode.CONFIG_ERROR, "非 standalone 模式必须在 URL 中提供有效的 jobId.");
        }
        configuration.set(CoreConstant.DATAX_CORE_CONTAINER_JOB_ID, jobId);

        //打印vmInfo
        VMInfo vmInfo = VMInfo.getVmInfo();
        if (vmInfo != null) {
            LOG.info(vmInfo.toString());
        }

        LOG.info("\n" + Engine.filterJobConfiguration(configuration) + "\n");

        LOG.debug(configuration.toJSON());

        ConfigurationValidate.doValidate(configuration);
        Engine engine = new Engine();
        engine.start(configuration);
    }

    /**
     * -1 表示未能解析到 jobId
     *
     *  only for dsc & ds & datax 3 update
     */
    private static long parseJobIdFromUrl(List<String> patternStringList, String url) {
        long result = -1;
        for (String patternString : patternStringList) {
            result = doParseJobIdFromUrl(patternString, url);
            if (result != -1) {
                return result;
            }
        }
        return result;
    }

    private static long doParseJobIdFromUrl(String patternString, String url) {
        Pattern pattern = Pattern.compile(patternString);
        Matcher matcher = pattern.matcher(url);
        if (matcher.find()) {
            return Long.parseLong(matcher.group(1));
        }

        return -1;
    }

    public static void main(String[] args) throws Exception {
        int exitCode = 0;
        try {
            Engine.entry(args);
        } catch (Throwable e) {
            exitCode = 1;
            LOG.error("\n\n经DataX智能分析,该任务最可能的错误原因是:\n" + ExceptionTracker.trace(e));

            if (e instanceof DataXException) {
                DataXException tempException = (DataXException) e;
                ErrorCode errorCode = tempException.getErrorCode();
                if (errorCode instanceof FrameworkErrorCode) {
                    FrameworkErrorCode tempErrorCode = (FrameworkErrorCode) errorCode;
                    exitCode = tempErrorCode.toExitValue();
                }
            }

            System.exit(exitCode);
        }
        System.exit(exitCode);
    }

    public static Communication getResult(){
        return result == null ? new Communication(): result;
    }

}
```

**注意：**调整DataX代码需要重新打包到本地Maven库

### 2.2.3. ProJob容器模块集成DataX

先通过普通执行器执行DataX作业，后续在考虑调整其他执行器执行。开发执行器需实现BasicProcessor，并复写process方法。

根据配置读取DATAX_HOME,并设置系统环境变量

```java
Properties properties = getProperties(log);
String dataxHome = properties.getProperty("DATAX_HOME");

System.setProperty("datax.home", dataxHome);
```

组装DataX执行需要的参数

```java
 String jobParams = context.getJobParams();
String[] datxArgs = {"-job", jobParams, "-mode", "standalone", "-jobid", "-1"};
```

具体任务执行方法和日志调整

```java
 try {
            log.info("Datax job start");
            log.info("...");
            long startTimeStamp = System.currentTimeMillis();
            Engine.entry(datxArgs);
            long endTimeStamp = System.currentTimeMillis();
            log.info("Datax job success");
            Communication communication = Engine.getResult();
            Map<String, Number> result = communication.getCounter();
            log.info(String.format(
                    "\n" + "%-26s: %-18s\n" + "%-26s: %-18s\n" + "%-26s: %19s\n"
                            + "%-26s: %19s\n" + "%-26s: %19s\n" + "%-26s: %19s\n"
                            + "%-26s: %19s\n",
                    "任务启动时刻", CommonUtils.formatTime(startTimeStamp),
                    "任务结束时刻", CommonUtils.formatTime(endTimeStamp),
                    "任务总计耗时", (endTimeStamp - startTimeStamp) / 1000 + "s",
                    "任务平均流量",
                    StrUtil.stringify((Long) result.get(CommunicationTool.BYTE_SPEED)) + "/s",
                    "记录写入速度", result.get(CommunicationTool.RECORD_SPEED) + " 条/s",
                    "读出记录总数", String.valueOf(CommunicationTool.getTotalReadRecords(communication)),
                    "读写失败总数", String.valueOf(CommunicationTool.getTotalErrorRecords(communication))
                    ));
        } catch (Throwable e) {
            log.error("Execute datax job {}, Exception info : \n {}", jobParams,
                    ExceptionUtils.stackTrace2String(e));
            return new ProcessResult(false, "Failed");
        }

        // 返回结果，该结果会被持久化到数据库，在前端页面直接查看，极为方便
        return new ProcessResult(true, "Successful");
```

执行器全部代码

```java
package com.tcfuture.kfcfans.containers;

import com.alibaba.datax.common.util.StrUtil;
import com.alibaba.datax.core.Engine;
import com.alibaba.datax.core.statistics.communication.Communication;
import com.alibaba.datax.core.statistics.communication.CommunicationTool;
import com.github.kfcfans.powerjob.common.ContainerConstant;
import com.github.kfcfans.powerjob.common.utils.CommonUtils;
import com.github.kfcfans.powerjob.common.utils.ExceptionUtils;
import com.github.kfcfans.powerjob.worker.core.processor.ProcessResult;
import com.github.kfcfans.powerjob.worker.core.processor.TaskContext;
import com.github.kfcfans.powerjob.worker.core.processor.sdk.BasicProcessor;
import com.github.kfcfans.powerjob.worker.log.OmsLogger;
import org.apache.commons.beanutils.PropertyUtils;
import org.springframework.stereotype.Component;

import java.io.InputStream;
import java.net.URL;
import java.util.Map;
import java.util.Objects;
import java.util.Properties;

/**
 * @author liulv
 *
 * Datax执行器
 *
 * 参数job目录地址，如：/job/mysql2mysql.json
 */
@Component
public class ContainerDataXProcessor implements BasicProcessor {

    /**
     * @param context 任务上下文，可通过 jobParams 和 instanceParams 分别获取控制台参数和OpenAPI传递的任务实例参数
     *
     * @return ProcessResult
     */
    @Override
    public ProcessResult process(TaskContext context){
        // 在线日志功能，可以直接在控制台查看任务日志，非常便捷
        OmsLogger log = context.getOmsLogger();
        String jobParams = context.getJobParams();
        log.info("ContainerDataXProcessor开始流程, 当前JOB参数 {}.", jobParams);

        //任务打印
        Properties properties = getProperties(log);
        String dataxHome = properties.getProperty("DATAX_HOME");

        // 进行实际处理...
        System.setProperty("datax.home", dataxHome);
        //job/mysql2mysql.json
        String[] datxArgs = {"-job", jobParams, "-mode", "standalone", "-jobid", "-1"};
        try {
            log.info("Datax job start");
            log.info("...");
            long startTimeStamp = System.currentTimeMillis();
            Engine.entry(datxArgs);
            long endTimeStamp = System.currentTimeMillis();
            log.info("Datax job success");
            Communication communication = Engine.getResult();
            Map<String, Number> result = communication.getCounter();
            log.info(String.format(
                    "\n" + "%-26s: %-18s\n" + "%-26s: %-18s\n" + "%-26s: %19s\n"
                            + "%-26s: %19s\n" + "%-26s: %19s\n" + "%-26s: %19s\n"
                            + "%-26s: %19s\n",
                    "任务启动时刻", CommonUtils.formatTime(startTimeStamp),
                    "任务结束时刻", CommonUtils.formatTime(endTimeStamp),
                    "任务总计耗时", (endTimeStamp - startTimeStamp) / 1000 + "s",
                    "任务平均流量",
                    StrUtil.stringify((Long) result.get(CommunicationTool.BYTE_SPEED)) + "/s",
                    "记录写入速度", result.get(CommunicationTool.RECORD_SPEED) + " 条/s",
                    "读出记录总数", String.valueOf(CommunicationTool.getTotalReadRecords(communication)),
                    "读写失败总数", String.valueOf(CommunicationTool.getTotalErrorRecords(communication))
                    ));
        } catch (Throwable e) {
            log.error("Execute datax job {}, Exception info : \n {}", jobParams,
                    ExceptionUtils.stackTrace2String(e));
            return new ProcessResult(false, "Failed");
        }

        // 返回结果，该结果会被持久化到数据库，在前端页面直接查看，极为方便
        return new ProcessResult(true, "Successful");
    }

    /**
     * 加载配置文件
     *
     * @param log OmsLogger
     * @return Properties
     */
    private static Properties getProperties(OmsLogger log) {
        final Properties properties = new Properties();
        URL propertiesURL =PropertyUtils.class.getClassLoader().getResource(
                ContainerConstant.CONTAINER_PROPERTIES_FILE_NAME);
        Objects.requireNonNull(propertiesURL);
        try (InputStream is = propertiesURL.openStream()) {
            properties.load(is);
        }catch (Exception e) {
            log.error(ExceptionUtils.stackTrace2String(e));
        }
        return properties;
    }
}
```

# 3. 配置ProwerJob支持DataX

配置ProwerJob支持DataX需要下面步骤

* **step 1:** 编译打包后目录在DataX工程下`target/datax/datax`，将此目录压缩上传到ProwerJob的Worker服务器下，如上传到`/data/datax/datax`目录下，如下图

![20200814000027](https://liulv.work/images/img/20200814000027.png)

* **step 2:** 根据step 1的datax目录，修改ProwerJob容器模块配置DATAX_HOME配置`/data/datax/datax`

![20200814000352](https://liulv.work/images/img/20200814000352.png)

* **step 3:** 编译打包ProwerJob，更新新调整的容器JAR包，直接在容器运维在编辑添加上传新打包的容器JAR包并部署

打包后容器JAR

![20200814000616](https://liulv.work/images/img/20200814000616.png)

容器运维模块点击编辑

![20200814000823](https://liulv.work/images/img/20200814000823.png)

点击click on Upload

![20200814000721](https://liulv.work/images/img/20200814000721.png)

上传容器JAR包，等待上传完成，保存

![20200814001052](https://liulv.work/images/img/20200814001052.png)

部署容器，在容器运维界面点击“部署”

![20200814001142](https://liulv.work/images/img/20200814001142.png)

出现下面提示信息，则证明部署成功

![20200814001223](https://liulv.work/images/img/20200814001223.png)


# 4. ProwerJob配置DataX 作业

## 4.1. 配置DataXJob的JSON文件

配置DataX作业Job，详细更多Job配置可参考[DataX Job配置说明](https://liulv.work/2020/08/14/datax-job-configuration-instruction/)，这里示例配置从数据源mysql同步到mysql，Job json如下:

```json
{
    "job": {
        "content": [
            {
                "reader": {
                    "name": "mysqlreader",
                    "parameter": {
                        "column": ["id","actual_trigger_time","app_id","expected_trigger_time","finished_time","gmt_create","gmt_modified","instance_id","instance_params","job_id","last_report_time","result","running_times","status","task_tracker_address","type","wf_instance_id"],
                        "connection": [
                            {
                                "jdbcUrl": ["jdbc:mysql://127.0.0.1:3306/powerjob_dev?useUnicode=true&characterEncoding=UTF-8"],
                                "table": ["instance_info"]
                            }
                        ],
                        "password": "123456",
                        "username": "root",
                        "where": ""
                    }
                },
                "writer": {
                    "name": "mysqlwriter",
                    "parameter": {
                        "column": ["id","actual_trigger_time","app_id","expected_trigger_time","finished_time","gmt_create","gmt_modified","instance_id","instance_params","job_id","last_report_time","result","running_times","status","task_tracker_address","type","wf_instance_id"],
                        "connection": [
                            {
                                "jdbcUrl": "jdbc:mysql://127.0.0.1:3306/datax_test?useUnicode=true&characterEncoding=UTF-8",
                                "table": ["instance_info"]
                            }
                        ],
                        "password": "123456",
                        "preSql": [],
                        "session": [],
                        "username": "root",
                        "writeMode": "update"
                    }
                }
            }
        ],
        "setting": {
            "speed": {
                "channel": "1"
            }
        }
    }
}
```

保持json文件文件`mysql2mysql.json`文件，并上传到PowerJob服务器的`/data/datax/datax/job`目录下， 如下图：

![20200813231619](https://liulv.work/images/img/20200813231619.png)

**特别注意：**后续不会使用json文件方式，需要在PowerJob端开发前端可配置方式作为参数传递Java容器执行方式。

## 4.2. 任务管理-新建任务

任务配置参考如下：

![20200814001509](https://liulv.work/images/img/20200814001509.png)

![20200814001528](https://liulv.work/images/img/20200814001528.png)

核心配置主要在两个配置：

1. 任务参数：`/data/datax/datax/job/mysql2mysql.json`，配置为上一步服务器上的[配置DataXJob的JSON文件](#配置DataXJob的JSON文件)中的上传服务器的json文件目录。

2. 执行配置：单机运行 Java容器 1#com.tcfuture.kfcfans.containers.ContainerDataXProcessor

**注意：**配置请按照实际情况调整

# 5. DataX任务实例运行情况

ProWerJob任务运行可根据任务名，查看任务运行情况。

![20200814002041](https://liulv.work/images/img/20200814002041.png)

查看详情

![20200814002105](https://liulv.work/images/img/20200814002105.png)

查看日志

![20200814002128](https://liulv.work/images/img/20200814002128.png)

# 6. 检查数据同步结果

通过json中配置的Writer中mysql的库表，查询数据是否同步成功，比如同步到本地数据库表datax_test.instance_info，开始时候是空表，打开表已经有数据表示同步数据成功，可按照实际情况进行数据一致性和数据准确检查。

![20200814002606](https://liulv.work/images/img/20200814002606.png)


END !!!















