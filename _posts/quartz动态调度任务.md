### 好处:
- quartz号称能够同时对上万个任务进行调度，拥有丰富的功能特性，包括任务调度、任务持久化、可集群化、插件等
- ​
###



# Cron Expressions

### cron的表达式被用来配置CronTrigger实例。 cron的表达式是字符串，实际上是由七子表达式，描述个别细节的时间表。这些子表达式是分开的空白

#### 每一个字段都有一套可以指定有效值

- Seconds (秒) ：可以用数字0－59 表示

- Minutes(分)  ：可以用数字0－59 表示

- Hours(时)  ：可以用数字0-23表示

- Day-of-Month(天) ：数字1-31,但要注意一些特别的月份

- Month(月) ：数字0-11 或用字符串 “JAN, FEB, MAR, APR, MAY, JUN, JUL, AUG, SEP, OCT, NOV and DEC” 表示

- Day-of-Week(每周)：数字1-7表示（1 ＝ 星期日）或用字符口串“SUN, MON, TUE, WED, THU, FRI and SAT”表示

- “/”：为特别单位，表示为“每”如“0/15”表示每隔15分钟执行一次,“0”表示为从“0”分开始, “3/20”表示表示每隔20分钟执行一次，“3”表示从第3分钟开始执行

- “?”：表示每月的某一天，或第周的某一天

- “L”：用于每月，或每周，表示为每月的最后一天，或每个月的最后星期几如“6L”表示“每月的最后一个星期五”

- “W”：表示为最近工作日，如“15W”放在每月（day-of-month）字段上表示为“到本月15日最近的工作日”

- ““#”：是用来指定“的”每月第n个工作日,例 在每周（day-of-week）这个字段中内容为"6#3" or "FRI#3" 则表示“每月第三个星期五”


### Cron表达式的格式：秒 分 时 日 月 周 年(可选)。
- Day-of-Month(天) Day-of-Week(每周) 二者选其一 另外一个未被选的默认为"?"

  - “?”字符：表示不确定的值

  - “,”字符：指定数个值

  - “-”字符：指定一个值的范围

  - “/”字符：指定一个值的增加幅度。n/m表示从n开始，每次增加m

  - “L”字符：用在日表示一个月中的最后一天，用在周表示该月最后一个星期X

  - “W”字符：指定离给定日期最近的工作日(周一到周五)

  - “#”字符：表示该月第几个周X。6#3表示该月第3个周五

  - “*” 代表整个时间段


### Cron表达式范例：
- 常用范例

   - 每秒钟执行:		* * * * * ? *
    - 每分钟执行 	0 * * * * ? *
     - 每小时执行 0 0 * * * ? *
   - 每天一点执行(日报<下一天初跑上一天>):0 0 1 * * ? *
    - 周一一点执行(周报<下周一跑上一周>): 0 0 1 ? * 2 *
    - 月初一点执行(月报<下月初跑上一月>):0 0 1 1 * ? *
   - 季末一点执行(季报<下季度第一天跑上季度>):0 0 1 1 1,4,7,11 ? *
    - 每年一点执行(年报<1月一号跑上一年>):0 0 1 1 1 ? *
   - 每隔5秒执行一次：*/5 * * * * ?
   - 每隔1分钟执行一次：0 */1 * * * ?
   - 每天23点执行一次：0 0 23 * * ?
   - 每天凌晨1点执行一次：0 0 1 * * ?
   - 每月1号凌晨1点执行一次：0 0 1 1 * ?
   - 每月最后一天23点执行一次：0 0 23 L * ?
   - 每周星期天凌晨1点实行一次：0 0 1 ? * L
   - 在26分、29分、33分执行一次：0 26,29,33 * * * ?
   - 每天的0点、13点、18点、21点都执行一次：0 0 0,13,18,21 * * ?

### quartz-SchedulerFactory 
最常用:StdSchedulerFactory
```
private static final String SIMPLE_THREAD_POOL = "org.quartz.simpl.SimpleThreadPool";
private static final String THREAD_COUNT = "org.quartz.threadPool.threadCount";
private static final String THREAD_COUNT_VALUE = "10";
static {
    factory  = new StdSchedulerFactory();
    Properties props = new Properties();
    props.put(StdSchedulerFactory.PROP_THREAD_POOL_CLASS, SIMPLE_THREAD_POOL);
    props.put(THREAD_COUNT, THREAD_COUNT_VALUE);
    try {
        factory.initialize(props);
        log.info("SchedulerFactory init start ...");
    } catch (SchedulerException e) {
        log.error("SchedulerFactory init failed ...", e);
    }
}
```

### quartz-Shceduler
```
Scheduler scheduler = null;
try {
    log.info("SchedulerFactory get default scheduler start ...");
    scheduler = factory.getScheduler();
    ListenerManager listenerManager = scheduler.getListenerManager();
    listenerManager.addJobListener(new QuartzJobListener());
    listenerManager.addSchedulerListener(new QuartzSchedulerListener());
    listenerManager.addTriggerListener(new QuartzTriggerListener());
    log.info("SchedulerFactory getScheduler successfully ...");
} catch (SchedulerException e) {
    log.error("SchedulerFactory start failed ...", e);
}
```



### quartz-Job
- ```implements Job```:实现job接口,并重写execute方法
```
public void execute(JobExecutionContext context) throws JobExecutionException

```
- JobExecutionContext作为quartz运行时上下文
```
JobDetail jobDetail = context.getJobDetail();
//jobKey
JobKey jobKey = jobDetail.getKey();
//detail name
String name = jobKey.getName();
//detail group
String group = jobKey.getGroup();
info("execute group[{}]  task [{}] ",group,name);
// 获取参数封装到map中传递
JobDataMap jobDataMap = jobDetail.getJobDataMap();
//获取用户的参数信息
String  params = (String)jobDataMap.get(PARAMS);
context.setResult(result.getSendStatus());
```
### quartz-JobDetail
```
Scheduler scheduler = schedulerFactory.getScheduler();
log.info("addJob method executing...");
JobDetail jobDetail = newJob(RocketMqJob.class)
        .withIdentity(quartzJob.getJobName(), quartzJob.getJobGroup())
        .withDescription(quartzJob.getDescription())
        .usingJobData(PARAMS,quartzJob.getParams())
        .build();
```
### quartz-Trigger
注:注意处理表达式错误的场景
```
try {
    Trigger trigger = newTrigger()
            .withIdentity(quartzJob.getTriggerName(), quartzJob.getTriggerGroup())
            .withSchedule(CronScheduleBuilder.cronSchedule(quartzJob.getCronExpression())
            ).build();
    scheduler.scheduleJob(jobDetail,trigger);
}catch (SchedulerException e) {
    log.error("scheduler add job failed ...",e);
    throw new Exception(e);
}
```

### quartz动态管理Job
- 新增JOB: ```scheduler.scheduleJob(jobDetail,trigger);```
- 删除JOB: ```scheduler.resumeJob(jobKey);```
- 暂停JOB: ```scheduler.pauseJob(jobKey);```
- 恢复JOB: ```scheduler.resumeJob(jobKey);```
- 全部暂停: ```scheduler.pauseAll();```
- 全部恢复: ```scheduler.resumeAll();```
- 启动scheduler: ```scheduler.start();```
- 检查Trigger: ```scheduler.checkExists(triggerKey);```
- 修改JOB:  
``` 
TriggerKey triggerKey = new TriggerKey(triggerName, triggerGroup);
if (!checkTrigger(triggerKey)) {
    return false;
}
boolean opt = true;
Scheduler scheduler = schedulerFactory.getScheduler();
try {

    CronTrigger cronTrigger =  (CronTrigger)scheduler.getTrigger(triggerKey);
    if (cronTrigger != null && !cronTrigger.getCronExpression().equals(cronExpression)) {
        CronScheduleBuilder scheduleBuilder = CronScheduleBuilder.cronSchedule(cronExpression);
        // 按新的cronExpression表达式重新构建trigger
        cronTrigger = cronTrigger.getTriggerBuilder().withIdentity(triggerKey).withSchedule(scheduleBuilder).build();
        // 按新的trigger重新设置job执行
        cronTrigger.getJobDataMap().put(PARAMS,params);
        scheduler.rescheduleJob(triggerKey, cronTrigger);
    }
    if (!scheduler.isStarted()) {
        scheduler.start();
    }
    log.info("updateTrigger method executed successfully ...");
} catch (SchedulerException e) {
    log.error("updateTrigger method executed fail ...");
    throw new Exception(e);
}
```

注:Trigger的操作和JOB类似,只需把对应的jobKey替换成triggerKey;

### quartz-Listener

- JobListener:

方式一:```extends JobListenerSupport```

方式二:```implements JobListener```

```
- JOB执行前准备
 public void jobToBeExecuted(JobExecutionContext context)
- JOB将要执行时
public void jobExecutionVetoed(JobExecutionContext context)
- JOB执行成功后
public void jobWasExecuted(JobExecutionContext context, JobExecutionException jobException)
```


- TriggerListener

方式一:```extends TriggerListenerSupport```

方式二:```implements TriggerListener```

```
 /**
     * 任务本该打火(触发)而错过打火(触发)
     * @Title
     * @Description
     * @param [trigger]
     * @return void
     * @date  2017/12/4 下午4:09
     * @author tush
     */
    @Override
    public void triggerMisfired(Trigger trigger) {
        super.triggerMisfired(trigger);
        log.info("{}:任务错过触发...");
        final String triggerName = trigger.getKey().getName();
        final String triggerGroup = trigger.getKey().getGroup();
        //获取任务详情
        JobInfoRepository jobInfoRepository = (JobInfoRepository)SpringContextHolder.getBean(JobInfoRepository.class);
        final JobInfoBO jobInfoBO = jobInfoRepository.selectJobByTrigger(triggerName, triggerGroup);
        final String jobName = jobInfoBO.getJobName();
        final String jobGroup = jobInfoBO.getJobGroup();
        String topicName = jobGroup;
        JobRouteRepository jobRouteRepository = (JobRouteRepository) SpringContextHolder.getBean(JobRouteRepository.class);
        final JobRouteBO jobRouteBO = jobRouteRepository.selectJobRoutByTopicName(topicName);
        final String sysName = jobRouteBO.getSysName();
        //记录错过触发的任务流水信息
        JobJrnRepository jobJrnRepository = (JobJrnRepository)SpringContextHolder.getBean(JobJrnRepository.class);
        JobJrnBO jobJrnBO =  JobJrnBO.builder()
                .jobGroup(triggerGroup)
                .jobName(triggerName)
                .triggerGroup(triggerGroup)
                .triggerName(triggerName)
                .topicName(topicName)
                .sysName(sysName)
                .runStatus(MISS_FIRED)
                .runMessage(MISS_FIRED_MESSAGE)
                .build();
        log.info("任务错过触发登记流水");
        jobJrnRepository.insertJobJrn(jobJrnBO);
    }
```

- SchedulerListener

方式一:```extends SchedulerListenerSupport```

方式二:```implements SchedulerListener```

```
@Override
public void jobDeleted(JobKey jobKey) {
    log.info("QuartzSchedulerListener jobDeleted method successfully :{} has been deleted",jobKey);
}
```