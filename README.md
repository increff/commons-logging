# commons-logging
commons-logging provides a custom log4j appender to manage log files
## Overview
Logging has following challenges

1. Disk becomes fully consumed by logs. This results in server stalling, and loss of logs.
2. We primarily use **Log4j**
   1. Log4j RollingFileAppender renames older files 
      1. So this can fool Greylog's Sidecar tool into thinking that new log files have been created. So Sidecar may send duplicate logs to Greylog 
      2. It does not name files as per date/time

   2. Log4j DailyRollingFileAppender only renames files as per date/time. It does not take care of disk utilization
3. There is no combination of the above which does this in a meaningful manner.

Thus, we have created our own Log4j appender in the commons-logging library.

## Usage
### log4j.properties
Add the following to the log4j.properties file in the project
```properties
#Worked with 2.17 version
log4j.rootLogger=INFO, file

#Specify our class to be used as the appender
log4j.appender.file=com.increff.commons.logging.SmartFileAppender

#Maximum file size which triggers rolling - 10MB
log4j.appender.file.MaxFileSize=10000000

#Maximum total log file disk usage - 1GB
log4j.appender.file.MaxDiskUsage=1000000000

#Log file name
log4j.appender.file.File=app.log
log4j.appender.file.BufferedIO=true
log4j.appender.file.layout=org.apache.log4j.PatternLayout  
log4j.appender.file.layout.ConversionPattern=%d [%t] %-5p (%F:%L) - %m%n
```

### pom.xml
To use `commons-logging`, include the following in the project's pom.xml
```xml
<dependency>
    <groupId>com.increff.commons</groupId>
    <artifactId>commons-logging</artifactId>
    <version>1.7</version>
</dependency>
```
## Behavior
1. The actual log files are named as `app.<yyyy-MM-dd-HH-mm>.log` For example, `app.2020-04-27-14-01.log`
2. The log file rotation happens when either of the following conditions is satisfied
   1. Current log file size exceeds `MaxFileSize`
   2. The day has changed
3. If too many logs are getting generated, then the file will get rotated only when the minute changes
4. When the total disk consumed by all log files exceeds `MaxDiskUsage`, then the oldest files are deleted to bring the usage under `MaxDiskUsage`

## Parameters
| Name | Required | Default | Example | Purpose |
|----|----|-----|-----|-----|
|File|	yes|	NA|	app.log|	Name of the log file. Actual file is named as `<yyyy-MM-dd-HH-mm>.log`|
|BufferedIO|	no|	false|	true|	Whether to use buffering or not. This parameter is recommended to improve performance.|
|BufferSize|	no|	8*1024 (8Kb)|16000|	bytes to buffer before flushing. This parameter can be left to default.|
|MaxFileSize|	no|	10000000 (10MB)|50000000 (50 MB)|Disk usage by current log file, before a new log file is created. This parameter may not behonored perfectly if logs generated per minute exceed this limit. **Value under 1MB will be ignored.**|
|MaxDiskUsage|	no|	10000000000 (10 GB)|50000000000 (50 GB)|Disk usage by all log files, before oldest files are deleted.Value under 50MB will be ignored|

