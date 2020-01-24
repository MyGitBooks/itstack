## 一、前言介绍
在日常开发中经常会用到定时任务，用来；库表扫描发送MQ、T+n账单结算、缓存数据更新、秒杀活动状态变更，等等。因为有了Spring的Schedule极大的方便了我们对这类场景的使用。那么，除了应用你还了解它多少呢；
1. 默认初始化多少个任务线程
2. JobStore有几种实现，你平时用的都是哪个
3. 一个定时任务的执行流程简述下

蒙圈了吧，是不感觉平时只是使用了，根本没关注过这些。有种冲动赶紧搜索答案吧！但只是知道答案是没有多少意义的，扛不住问不说，也不了解原理。所以，如果你想真的提升自己技能，还是要从根本搞定。

## 二、案例工程

为了更好的做源码分析，我们将平时用的定时任务服务单独抽离出来。工程下载，关注公众号：bugstack虫洞栈，回复：源码分析

```java
itstack-demo-code-schedule
└── src
    ├── main
    │   ├── java
    │   │   └── org.itstack.demo
    │   │       ├── DemoTask.java
    │   │       └── JobImpl.java   
    │   └── resources	
    │       ├── props	
    │       │   └── config.properties
    │       ├── spring
    │       │   └── spring-config-schedule-task.xml
    │       ├── logback.xml
    │       └── spring-config.xml
    └── test
         └── java
             └── org.itstack.demo.test
                 ├── ApiTest.java
                 ├── MyQuartz.java				
                 └── MyTask.java
```

## 三、环境配置

1. JDK 1.8
2. IDEA 2019.3.1
3. Spring 4.3.24.RELEASE
3. quartz 2.3.2 ｛不同版本略有代码差异｝

## 四、源码分析

```xml
<dependency>
    <groupId>org.quartz-scheduler</groupId>
    <artifactId>quartz</artifactId>
    <version>2.3.2</version>
</dependency>
```

依赖于Spring版本升级quartz选择2.3.2，同时如果你如本文案例中所示使用xml配置任务。那么会有如下更改；

> Spring 3.x/org.springframework.scheduling.quart.CronTriggerBean

```java
 <bean id="taskTrigger" class="org.springframework.scheduling.quartz.CronTriggerBean">
     <property name="jobDetail" ref="taskHandler"/>
     <property name="cronExpression" value="0/5 * * * * ?"/>
 </bean>
```

> Spring 4.x/org.springframework.scheduling.quartz.CronTriggerFactoryBean

```java
 <bean id="taskTrigger" class="org.springframework.scheduling.quartz.CronTriggerFactoryBean">
     <property name="jobDetail" ref="taskHandler"/>
     <property name="cronExpression" value="0/5 * * * * ?"/>
 </bean>
```

在正式分析前，可以看下quartz的默认配置，很多初始化动作都要从这里取得参数，同样你可以配置自己的配置文件。例如，当你的任务很多时，默认初始化的10个线程组不满足你的业务需求，就可以按需调整。

>quart.properties

```java
# Default Properties file for use by StdSchedulerFactory
# to create a Quartz Scheduler Instance, if a different
# properties file is not explicitly specified.
#

org.quartz.scheduler.instanceName: DefaultQuartzScheduler
org.quartz.scheduler.rmi.export: false
org.quartz.scheduler.rmi.proxy: false
org.quartz.scheduler.wrapJobExecutionInUserTransaction: false

org.quartz.threadPool.class: org.quartz.simpl.SimpleThreadPool
org.quartz.threadPool.threadCount: 10
org.quartz.threadPool.threadPriority: 5
org.quartz.threadPool.threadsInheritContextClassLoaderOfInitializingThread: true

org.quartz.jobStore.misfireThreshold: 60000

org.quartz.jobStore.class: org.quartz.simpl.RAMJobStore
```

### 1. 从一个简单案例开始

平时我们使用Schedule基本都是注解或者xml配置文件，但是为了可以更简单的分析代码，我们从一个简单的Demo入手，放到main函数中。

>DemoTask.java & 定义一个等待被执行的任务

```java
public class DemoTask {

    private Logger logger = LoggerFactory.getLogger(DemoTask.class);

    public void execute() throws Exception{
        logger.info("定时处理用户信息任务：0/5 * * * * ?");
    }

}
```

>MyTask.java & 测试类，将配置在xml中的代码抽离出来

```java
public class MyTask {

    public static void main(String[] args) throws Exception {

        DemoTask demoTask = new DemoTask();

        // 定义了；执行的内容
        MethodInvokingJobDetailFactoryBean methodInvokingJobDetailFactoryBean = new MethodInvokingJobDetailFactoryBean();
        methodInvokingJobDetailFactoryBean.setTargetObject(demoTask);
        methodInvokingJobDetailFactoryBean.setTargetMethod("execute");
        methodInvokingJobDetailFactoryBean.setConcurrent(true);
        methodInvokingJobDetailFactoryBean.setName("demoTask");
        methodInvokingJobDetailFactoryBean.afterPropertiesSet();

        // 定义了；执行的计划
        CronTriggerFactoryBean cronTriggerFactoryBean = new CronTriggerFactoryBean();
        cronTriggerFactoryBean.setJobDetail(methodInvokingJobDetailFactoryBean.getObject());
        cronTriggerFactoryBean.setCronExpression("0/5 * * * * ?");
        cronTriggerFactoryBean.setName("demoTask");
        cronTriggerFactoryBean.afterPropertiesSet();

        // 实现了；执行的功能
        SchedulerFactoryBean schedulerFactoryBean = new SchedulerFactoryBean();
        schedulerFactoryBean.setTriggers(cronTriggerFactoryBean.getObject());
        schedulerFactoryBean.setAutoStartup(true);
        schedulerFactoryBean.afterPropertiesSet();

        schedulerFactoryBean.start();

        // 暂停住
        System.in.read();

    }

}
```

**如果一切顺利，那么会有如下结果：**

```java
2020-01-01 10:47:16.369 [main] INFO  org.quartz.impl.StdSchedulerFactory[1220] - Using default implementation for ThreadExecutor
2020-01-01 10:47:16.421 [main] INFO  org.quartz.core.SchedulerSignalerImpl[61] - Initialized Scheduler Signaller of type: class org.quartz.core.SchedulerSignalerImpl
2020-01-01 10:47:16.422 [main] INFO  org.quartz.core.QuartzScheduler[229] - Quartz Scheduler v.2.3.2 created.
2020-01-01 10:47:16.423 [main] INFO  org.quartz.simpl.RAMJobStore[155] - RAMJobStore initialized.
2020-01-01 10:47:16.424 [main] INFO  org.quartz.core.QuartzScheduler[294] - Scheduler meta-data: Quartz Scheduler (v2.3.2) 'QuartzScheduler' with instanceId 'NON_CLUSTERED'
  Scheduler class: 'org.quartz.core.QuartzScheduler' - running locally.
  NOT STARTED.
  Currently in standby mode.
  Number of jobs executed: 0
  Using thread pool 'org.quartz.simpl.SimpleThreadPool' - with 10 threads.
  Using job-store 'org.quartz.simpl.RAMJobStore' - which does not support persistence. and is not clustered.

2020-01-01 10:47:16.424 [main] INFO  org.quartz.impl.StdSchedulerFactory[1374] - Quartz scheduler 'QuartzScheduler' initialized from an externally provided properties instance.
2020-01-01 10:47:16.424 [main] INFO  org.quartz.impl.StdSchedulerFactory[1378] - Quartz scheduler version: 2.3.2
2020-01-01 10:47:16.426 [main] INFO  org.quartz.core.QuartzScheduler[2293] - JobFactory set to: org.springframework.scheduling.quartz.AdaptableJobFactory@3e9b1010
2020-01-01 10:47:16.651 [main] INFO  org.quartz.core.QuartzScheduler[547] - Scheduler QuartzScheduler_$_NON_CLUSTERED started.
一月 04, 2020 10:47:16 上午 org.springframework.scheduling.quartz.SchedulerFactoryBean startScheduler
信息: Starting Quartz Scheduler now
2020-01-01 10:47:20.321 [QuartzScheduler_Worker-1] INFO  org.itstack.demo.DemoTask[11] - 定时处理用户信息任务：0/5 * * * * ?
2020-01-01 10:47:25.001 [QuartzScheduler_Worker-2] INFO  org.itstack.demo.DemoTask[11] - 定时处理用户信息任务：0/5 * * * * ?
2020-01-01 10:47:30.000 [QuartzScheduler_Worker-3] INFO  org.itstack.demo.DemoTask[11] - 定时处理用户信息任务：0/5 * * * * ?
2020-01-01 10:47:35.001 [QuartzScheduler_Worker-4] INFO  org.itstack.demo.DemoTask[11] - 定时处理用户信息任务：0/5 * * * * ?
2020-01-01 10:47:40.000 [QuartzScheduler_Worker-5] INFO  org.itstack.demo.DemoTask[11] - 定时处理用户信息任务：0/5 * * * * ?

Process finished with exit code -1
```

### 2. 定义执行内容(MethodInvokingJobDetailFactoryBean)

```java
// 定义了；执行的内容
MethodInvokingJobDetailFactoryBean methodInvokingJobDetailFactoryBean = new MethodInvokingJobDetailFactoryBean();
methodInvokingJobDetailFactoryBean.setTargetObject(demoTask);
methodInvokingJobDetailFactoryBean.setTargetMethod("execute");
methodInvokingJobDetailFactoryBean.setConcurrent(true);
methodInvokingJobDetailFactoryBean.setName("demoTask");
methodInvokingJobDetailFactoryBean.afterPropertiesSet();
```

这块内容主要将我们的任务体(即待执行任务DemoTask)交给MethodInvokingJobDetailFactoryBean管理，首先设置必要信息；
- targetObject：目标对象bean，也就是demoTask
- targetMethod：目标方法name，也就是execute
- concurrent：是否并行执行，非并行执行任务，如果上一个任务没有执行完，下一刻不会执行
- name：xml配置非必传，源码中可以获取beanName

最后我们通过手动调用 afterPropertiesSet() 来模拟初始化。如果我们的类是交给 Spring 管理的，那么在实现了 InitializingBean 接口的类，在类配置信息加载后会自动执行 afterPropertiesSet() 。一般实现了 InitializingBean 接口的类，同时也会去实现 FactoryBean<T> 接口，因为这个接口实现后就可以通过 T getObject() 获取自己自定义初始化的类。这也常常用在一些框架开发中。
 
>MethodInvokingJobDetailFactoryBean.afterPropertiesSet()

```java
public void afterPropertiesSet() throws ClassNotFoundException, NoSuchMethodException {
	prepare();
	// Use specific name if given, else fall back to bean name.
	String name = (this.name != null ? this.name : this.beanName);
	// Consider the concurrent flag to choose between stateful and stateless job.
	Class<?> jobClass = (this.concurrent ? MethodInvokingJob.class : StatefulMethodInvokingJob.class);
	// Build JobDetail instance.
	JobDetailImpl jdi = new JobDetailImpl();
	jdi.setName(name);
	jdi.setGroup(this.group);
	jdi.setJobClass((Class) jobClass);
	jdi.setDurability(true);
	jdi.getJobDataMap().put("methodInvoker", this);
	this.jobDetail = jdi;
	
	postProcessJobDetail(this.jobDetail);
}
```

- **源码168行：** 根据是否并行执行选择任务类，这两个类都是MethodInvokingJobDetailFactoryBean的内部类，非并行执行的StatefulMethodInvokingJob只是继承MethodInvokingJob添加了标记注解。
- **源码171行：** 创建JobDetailImpl，添加任务明细信息，注意这类的jdi.setJobClass((Class) jobClass)实际就是MethodInvokingJob。MethodInvokingJob也是我们最终要反射调用执行的内容。
- **源码177行：** 初始化任务后赋值给this.jobDetail = jdi，也就是最终的类对象

	>MethodInvokingJobDetailFactoryBean.getObject()

	```java
	@Override
	public JobDetail getObject() {
		return this.jobDetail;
	}
	```

- **源码：220行：** 获取对象时返回 this.jobDetail，这也就解释了为什么 MethodInvokingJobDetailFactoryBean 初始化后直接赋值给了一个 JobDetail ；

	![微信公众号：bugstack虫洞栈 & Schedule.xml](https://fuzhengwei.github.io/assets/images/pic-content/2019/11/itstack-demo-code-schedule-01.png)


### 3. 定义执行计划(CronTriggerFactoryBeann)

```java
// 定义了；执行的计划
CronTriggerFactoryBean cronTriggerFactoryBean = new CronTriggerFactoryBean();
cronTriggerFactoryBean.setJobDetail(methodInvokingJobDetailFactoryBean.getObject());
cronTriggerFactoryBean.setCronExpression("0/5 * * * * ?");
cronTriggerFactoryBean.setName("demoTask");
cronTriggerFactoryBean.afterPropertiesSet();
```

这一块主要定义任务的执行计划，并将任务执行内容交给 CronTriggerFactoryBean 管理，同时设置必要信息；
- jobDetail：设置任务体，xml 中可以直接将对象赋值，硬编码中设置执行的 JobDetail 对象信息。也就是我们上面设置的 JobDetailImpl ，通过 getObject() 获取出来。
- cronExpression：计划表达式；秒、分、时、日、月、周、年

>CronTriggerFactoryBean.afterPropertiesSet()

```java
@Override
public void afterPropertiesSet() throws ParseException {
    
	// ... 校验属性信息
	
	CronTriggerImpl cti = new CronTriggerImpl();
	cti.setName(this.name);
	cti.setGroup(this.group);
	if (this.jobDetail != null) {
		cti.setJobKey(this.jobDetail.getKey());
	}
	cti.setJobDataMap(this.jobDataMap);
	cti.setStartTime(this.startTime);
	cti.setCronExpression(this.cronExpression);
	cti.setTimeZone(this.timeZone);
	cti.setCalendarName(this.calendarName);
	cti.setPriority(this.priority);
	cti.setMisfireInstruction(this.misfireInstruction);
	cti.setDescription(this.description);
	this.cronTrigger = cti;
}
```

- **源码237行：** 创建触发器 CronTriggerImpl 并设置相关属性信息
- **源码245行：** 生成执行计划类 cti.setCronExpression(this.cronExpression);
	
	```java
	public void setCronExpression(String cronExpression) throws ParseException {
		TimeZone origTz = getTimeZone();
		this.cronEx = new CronExpression(cronExpression);
		this.cronEx.setTimeZone(origTz);
	}
	```
	
	>CronExpression.java & 解析Cron表达式
	
	```java
	protected void buildExpression(String expression) throws ParseException {
		expressionParsed = true;
		try {
			
			// ... 初始化 TreeSet xxx = new TreeSet<Integer>();
			
			int exprOn = SECOND;
			StringTokenizer exprsTok = new StringTokenizer(expression, " \t",
					false);
					
			while (exprsTok.hasMoreTokens() && exprOn <= YEAR) {
				String expr = exprsTok.nextToken().trim();
				
				// ... 校验DAY_OF_MONTH和DAY_OF_WEEK字段的特殊字符
				
				StringTokenizer vTok = new StringTokenizer(expr, ",");
				while (vTok.hasMoreTokens()) {
					String v = vTok.nextToken();
					storeExpressionVals(0, v, exprOn);
				}
				exprOn++;
			}
			
			// ... 校验DAY_OF_MONTH和DAY_OF_WEEK字段的特殊字符
			
		} catch (ParseException pe) {
			throw pe;
		} catch (Exception e) {
			throw new ParseException("Illegal cron expression format ("
					+ e.toString() + ")", 0);
		}
	}
	```
	
	- Cron表达式有7个字段，CronExpression 把7个字段解析为7个 TreeSet 对象。
	- 填充TreeSet对象值的时候，表达式都会转换为起始值、结束值和增量的计算模式，然后计算出匹配的值放进TreeSet对象
	
>CronTriggerFactoryBean.getObject()

```java
@Override
public CronTrigger getObject() {
	return this.cronTrigger;
}
```

- **源码257行：** 获取对象时返回 this.cronTrigger ，也就是 CronTriggerImpl 对象

### 4. 调度执行计划(SchedulerFactoryBean)

```java
// 调度了；执行的计划(scheduler)
SchedulerFactoryBean schedulerFactoryBean = new SchedulerFactoryBean();
schedulerFactoryBean.setTriggers(cronTriggerFactoryBean.getObject());
schedulerFactoryBean.setAutoStartup(true);
schedulerFactoryBean.afterPropertiesSet();

schedulerFactoryBean.start();
```

这一部分如名字一样调度工厂，相当于一个指挥官，可以从全局做调度，比如监听哪些trigger已经ready、分配线程等等，同样也需要设置必要的属性信息；
- triggers：按需可以设置多个触发器，本文设置了一个 cronTriggerFactoryBean.getObject() 也就是 CronTriggerImpl 对象
- autoStartup：默认是否自动启动任务，默认值为true

这个过程较长包括：调度工厂、线程池、注册任务等等，整体核心加载流程如下；

![微信公众号：bugstack虫洞栈 & 调度工程初始化流程](https://fuzhengwei.github.io/assets/images/pic-content/2019/11/itstack-demo-code-schedule-02.png)

- 整个加载过程较长，抽取部分核心代码块进行分析，其中包括的类；
	- StdScheduler
	- StdSchedulerFactory
	- SimpleThreadPool
	- QuartzScheduler
	- QuartzSchedulerThread
	- RAMJobStore
	- CronTriggerImpl
	- CronExpression

>SchedulerFactoryBean.afterPropertiesSet()

```java
public void afterPropertiesSet() throws Exception {
	if (this.dataSource == null && this.nonTransactionalDataSource != null) {
		this.dataSource = this.nonTransactionalDataSource;
	}
	if (this.applicationContext != null && this.resourceLoader == null) {
		this.resourceLoader = this.applicationContext;
	}
	// Initialize the Scheduler instance...
	this.scheduler = prepareScheduler(prepareSchedulerFactory());
	try {
		registerListeners();
		registerJobsAndTriggers();
	}
	catch (Exception ex) {
		try {
			this.scheduler.shutdown(true);
		}
		catch (Exception ex2) {
			logger.debug("Scheduler shutdown exception after registration failure", ex2);
		}
		throw ex;
	}
}
```

- **源码474行：** 为调度器做准备工作 prepareScheduler(prepareSchedulerFactory()) ，依次执行如下；
	1. SchedulerFactoryBean.prepareScheduler(SchedulerFactory schedulerFactory)
	2. SchedulerFactoryBean.createScheduler(schedulerFactory, this.schedulerName);
	3. SchedulerFactoryBean.createScheduler(SchedulerFactory schedulerFactory, String schedulerName)
	4. Scheduler newScheduler = schedulerFactory.getScheduler();
	5. StdSchedulerFactory.getScheduler();
	6. sched = instantiate(); **包括一系列核心操作；**
	
	```java
	1）初始化threadPool(线程池)：开发者可以通过org.quartz.threadPool.class配置指定使用哪个线程池类，比如SimpleThreadPool。
	2）初始化jobStore(任务存储方式)：开发者可以通过org.quartz.jobStore.class配置指定使用哪个任务存储类，比如RAMJobStore。
	3）初始化dataSource(数据源)：开发者可以通过org.quartz.dataSource配置指定数据源详情，比如哪个数据库、账号、密码等。
	4）初始化其他配置：包括SchedulerPlugins、JobListeners、TriggerListeners等；
    5）初始化threadExecutor(线程执行器)：默认为DefaultThreadExecutor；
    6）创建工作线程：根据配置创建N个工作thread，执行start()启动thread，并将N个thread顺序add进threadPool实例的空闲线程列表availWorkers中；
    7）创建调度器线程：创建QuartzSchedulerThread实例，并通过threadExecutor.execute(实例)启动调度器线程；
    8）创建调度器：创建StdScheduler实例，将上面所有配置和引用组合进实例中，并将实例存入调度器池中
    ```
		
- **源码477行：** 调用父类 SchedulerAccessor.registerJobsAndTriggers() 注册任务和触发器
	
	```java
	for (Trigger trigger : this.triggers) {
		addTriggerToScheduler(trigger);
	}
	```
	
>SchedulerAccessor.addTriggerToScheduler() & SchedulerAccessor 是SchedulerFactoryBean的父类

```java
private boolean addTriggerToScheduler(Trigger trigger) throws SchedulerException {
	boolean triggerExists = (getScheduler().getTrigger(trigger.getKey()) != null);
	if (triggerExists && !this.overwriteExistingJobs) {
		return false;
	}
	// Check if the Trigger is aware of an associated JobDetail.
	JobDetail jobDetail = (JobDetail) trigger.getJobDataMap().remove("jobDetail");
	if (triggerExists) {
		if (jobDetail != null && !this.jobDetails.contains(jobDetail) && addJobToScheduler(jobDetail)) {
			this.jobDetails.add(jobDetail);
		}
		try {
			getScheduler().rescheduleJob(trigger.getKey(), trigger);
		}
		catch (ObjectAlreadyExistsException ex) {
			if (logger.isDebugEnabled()) {
				logger.debug("Unexpectedly encountered existing trigger on rescheduling, assumably due to " +
						"cluster race condition: " + ex.getMessage() + " - can safely be ignored");
			}
		}
	}
	else {
		try {
			if (jobDetail != null && !this.jobDetails.contains(jobDetail) &&
					(this.overwriteExistingJobs || getScheduler().getJobDetail(jobDetail.getKey()) == null)) {
				getScheduler().scheduleJob(jobDetail, trigger);
				this.jobDetails.add(jobDetail);
			}
			else {
				getScheduler().scheduleJob(trigger);
			}
		}
		catch (ObjectAlreadyExistsException ex) {
			if (logger.isDebugEnabled()) {
				logger.debug("Unexpectedly encountered existing trigger on job scheduling, assumably due to " +
						"cluster race condition: " + ex.getMessage() + " - can safely be ignored");
			}
			if (this.overwriteExistingJobs) {
				getScheduler().rescheduleJob(trigger.getKey(), trigger);
			}
		}
	}
	return true;
}
```

- **源码299行：** addJobToScheduler(jobDetail) 一直会调用到 RAMJobStore	进行存放任务信息到 HashMap<JobKey, JobWrapper>(100)

	```java
	public void storeJob(JobDetail newJob,
        boolean replaceExisting) throws ObjectAlreadyExistsException {
		JobWrapper jw = new JobWrapper((JobDetail)newJob.clone());
		boolean repl = false;
		synchronized (lock) {
			if (jobsByKey.get(jw.key) != null) {
				if (!replaceExisting) {
					throw new ObjectAlreadyExistsException(newJob);
				}
				repl = true;
			}
			if (!repl) {
				// get job group
				HashMap<JobKey, JobWrapper> grpMap = jobsByGroup.get(newJob.getKey().getGroup());
				if (grpMap == null) {
					grpMap = new HashMap<JobKey, JobWrapper>(100);
					jobsByGroup.put(newJob.getKey().getGroup(), grpMap);
				}
				// add to jobs by group
				grpMap.put(newJob.getKey(), jw);
				// add to jobs by FQN map
				jobsByKey.put(jw.key, jw);
			} else {
				// update job detail
				JobWrapper orig = jobsByKey.get(jw.key);
				orig.jobDetail = jw.jobDetail; // already cloned
			}
		}
	}
	```

- 初始化线程组；
	- prepareScheduler
	- createScheduler
	- schedulerFactory
	- StdSchedulerFactory.getScheduler()
	- getScheduler()->instantiate()
	- **源码1323行：**  tp.initialize();
	
	>SimpleThreadPool.initialize() & 这里的count是默认配置中的数量，可以更改
	
	```java
	 // create the worker threads and start them
	 Iterator<WorkerThread> workerThreads = createWorkerThreads(count).iterator();
	 while(workerThreads.hasNext()) {
		 WorkerThread wt = workerThreads.next();
		 wt.start();
		 availWorkers.add(wt);
	 }
	```
	
### 5. 启动定时任务

案例中使用硬编码方式调用 schedulerFactoryBean.start() 启动线程服务。线程的协作通过Object sigLock来实现，关于sigLock.wait()方法都在QuartzSchedulerThread的run方法里面，所以sigLock唤醒的是只有线程QuartzSchedulerThread。核心流程如下；

![微信公众号：bugstack虫洞栈 & 调度启动流程](https://fuzhengwei.github.io/assets/images/pic-content/2019/11/itstack-demo-code-schedule-03.png)

这个启动过程中，核心的代码类，如下；
- StdScheduler
- QuartzScheduler
- QuartzSchedulerThread
- ThreadPool
- RAMJobStore
- CronTriggerImpl
- JobRunShellFactory

>QuartzScheduler.start() & 启动

```java
public void start() throws SchedulerException {

    if (shuttingDown|| closed) {
        throw new SchedulerException(
                "The Scheduler cannot be restarted after shutdown() has been called.");
    }
	
    // QTZ-212 : calling new schedulerStarting() method on the listeners
    // right after entering start()
    notifySchedulerListenersStarting();
    
	if (initialStart == null) {
        initialStart = new Date();
        this.resources.getJobStore().schedulerStarted();            
        startPlugins();
    } else {
        resources.getJobStore().schedulerResumed();
    }
	
    // 唤醒线程
	schedThread.togglePause(false);
	
    getLog().info(
            "Scheduler " + resources.getUniqueIdentifier() + " started.");
    
    notifySchedulerListenersStarted();
}
```

>QuartzSchedulerThread.run() & 执行过程

```java
@Override
public void run() {
    int acquiresFailed = 0;
	
	// 只有调用了halt()方法，才会退出这个死循环
    while (!halted.get()) {
        try {
			
			// 一、如果是暂停状态，则循环超时等待1000毫秒

            // wait a bit, if reading from job store is consistently failing (e.g. DB is down or restarting)..
           
		    // 阻塞直到有空闲的线程可用并返回可用的数量
            int availThreadCount = qsRsrcs.getThreadPool().blockForAvailableThreads();
            if(availThreadCount > 0) {
			
                List<OperableTrigger> triggers;
                long now = System.currentTimeMillis();
                clearSignaledSchedulingChange();
                
				try {
					// 二、获取acquire状态的Trigger列表，也就是即将执行的任务
                    triggers = qsRsrcs.getJobStore().acquireNextTriggers(
                            now + idleWaitTime, Math.min(availThreadCount, qsRsrcs.getMaxBat
                    acquiresFailed = 0;
                    if (log.isDebugEnabled())
                        log.debug("batch acquisition of " + (triggers == null ? 0 : triggers
                } catch(){//...}
				
                if (triggers != null && !triggers.isEmpty()) {
                    
					// 三：获取List第一个Trigger的下次触发时刻
					long triggerTime = triggers.get(0).getNextFireTime().getTime();
                    
					// 四：获取任务触发集合
					List<TriggerFiredResult> res = qsRsrcs.getJobStore().triggersFired(triggers);
					
					// 五：设置Triggers为'executing'状态
					qsRsrcs.getJobStore().releaseAcquiredTrigger(triggers.get(i));
                    
					// 六：创建JobRunShell
					qsRsrcs.getJobRunShellFactory().createJobRunShell(bndle);
					
					// 七：执行Job
					qsRsrcs.getThreadPool().runInThread(shell)
					
                    continue; // while (!halted)
                }
            } else { // if(availThreadCount > 0)
                // should never happen, if threadPool.blockForAvailableThreads() follows con
                continue; // while (!halted)
            }
			
            
        } catch(RuntimeException re) {
            getLog().error("Runtime error occurred in main trigger firing loop.", re);
        }
    }
    
    qs = null;
    qsRsrcs = null;
}
```

- **源码391行：** 创建JobRunShell，JobRunShell实例在initialize()方法就会把包含业务逻辑类的JobDetailImpl设置为它的成员属性，为后面执行业务逻辑代码做准备。执行业务逻辑代码在runInThread(shell)方法里面。
	
	>QuartzSchedulerThread.run() & 部分代码
	
	```java
	JobRunShell shell = null;
	try {
		shell = qsRsrcs.getJobRunShellFactory().createJobRunShell(bndle);
		shell.initialize(qs);
	} catch (SchedulerException se) {
		qsRsrcs.getJobStore().triggeredJobComplete(triggers.get(i), bndle.getJobDetail(), CompletedExecutionInstruction.SET_ALL_JOB_TRIGGERS_ERROR);
		continue;
	}
	```

- **源码398行：** qsRsrcs.getThreadPool().runInThread(shell)
	
	>SimpleThreadPool.runInThread
	
	```java
	// 保存所有WorkerThread的集合
	private List<WorkerThread> workers;
	// 空闲的WorkerThread集合
	private LinkedList<WorkerThread> availWorkers = new LinkedList<WorkerThread>();
	// 任务的WorkerThread集合
	private LinkedList<WorkerThread> busyWorkers = new LinkedList<WorkerThread>();

	/**
	 * 维护workers、availWorkers和busyWorkers三个列表数据
	 * 有任务需要一个线程出来执行：availWorkers.removeFirst();busyWorkers.add()
	 * 然后调用WorkThread.run(runnable)方法
	 */
	public boolean runInThread(Runnable runnable) {
		if (runnable == null) {
			return false;
		}

		synchronized (nextRunnableLock) {

			handoffPending = true;

			// Wait until a worker thread is available
			while ((availWorkers.size() < 1) && !isShutdown) {
				try {
					nextRunnableLock.wait(500);
				} catch (InterruptedException ignore) {
				}
			}

			if (!isShutdown) {
				WorkerThread wt = (WorkerThread)availWorkers.removeFirst();
				busyWorkers.add(wt);
				wt.run(runnable);
			} else {
				// If the thread pool is going down, execute the Runnable
				// within a new additional worker thread (no thread from the pool).
				
				WorkerThread wt = new WorkerThread(this, threadGroup,
						"WorkerThread-LastJob", prio, isMakeThreadsDaemons(), runnable);
				busyWorkers.add(wt);
				workers.add(wt);
				wt.start();
			}
			nextRunnableLock.notifyAll();
			handoffPending = false;
		}

		return true;
	}
	```

- **源码428行：** WorkerThread ，是一个内部类，主要是赋值并唤醒lock对象的等待线程队列
	
	>WorkerThread.run(Runnable newRunnable)
	
	```java
	public void run(Runnable newRunnable) {
		synchronized(lock) {
			if(runnable != null) {
				throw new IllegalStateException("Already running a Runnable!");
			}
			runnable = newRunnable;
			lock.notifyAll();
		}
	}
	```

- **源码561行：** WorkerThread 的run方法，方法执行lock.notifyAll()后，对应的WorkerThread就会来到run()方法。到这！接近曙光了！终于来到了执行业务的execute()方法的倒数第二步，runnable对象是一个JobRunShell对象，下面在看JobRunShell.run()方法。

	>WorkerThread.run()
	
	```java
	@Override
	public void run() {
		boolean ran = false;
		
		while (run.get()) {
			try {
				synchronized(lock) {
					while (runnable == null && run.get()) {
						lock.wait(500);
					}
					if (runnable != null) {
						ran = true;
						// 启动真正执行的内容，runnable就是JobRunShell
						runnable.run();
					}
				}
			} cache(){//...}
		}
		//if (log.isDebugEnabled())
		try {
			getLog().debug("WorkerThread is shut down.");
		} catch(Exception e) {
			// ignore to help with a tomcat glitch
		}
	}
	```
	
>JobRunShell.run() & 从上面WorkerThread.run()，调用到这里执行

```java
public void run() {
    qs.addInternalSchedulerListener(this);

    try {
        OperableTrigger trigger = (OperableTrigger) jec.getTrigger();
        JobDetail jobDetail = jec.getJobDetail();

        do {
            // ...

            long startTime = System.currentTimeMillis();
            long endTime = startTime;

            // execute the job
            try {
                log.debug("Calling execute on job " + jobDetail.getKey());
                
				// 执行业务代码，也就是我们的task
				job.execute(jec);
                
				endTime = System.currentTimeMillis();
            } catch (JobExecutionException jee) {
                endTime = System.currentTimeMillis();
                jobExEx = jee;
                getLog().info("Job " + jobDetail.getKey() +
                        " threw a JobExecutionException: ", jobExEx);
            } catch (Throwable e) {
                endTime = System.currentTimeMillis();
                getLog().error("Job " + jobDetail.getKey() +
                        " threw an unhandled Exception: ", e);
                SchedulerException se = new SchedulerException(
                        "Job threw an unhandled exception.", e);
                qs.notifySchedulerListenersError("Job ("
                        + jec.getJobDetail().getKey()
                        + " threw an exception.", se);
                jobExEx = new JobExecutionException(se, false);
            }

            jec.setJobRunTime(endTime - startTime);

            // 其他代码
        } while (true);

    } finally {
        qs.removeInternalSchedulerListener(this);
    }
}
```

>QuartzJobBean.execte() & 继续往下走

```java
public final void execute(JobExecutionContext context) throws JobExecutionException {
	try {
		BeanWrapper bw = PropertyAccessorFactory.forBeanPropertyAccess(this);
		MutablePropertyValues pvs = new MutablePropertyValues();
		pvs.addPropertyValues(context.getScheduler().getContext());
		pvs.addPropertyValues(context.getMergedJobDataMap());
		bw.setPropertyValues(pvs, true);
	}
	catch (SchedulerException ex) {
		throw new JobExecutionException(ex);
	}
	executeInternal(context);
}
```

>MethodInvokingJobDetailFactoryBean->MethodInvokingJob.executeInternal(JobExecutionContext context)

```java
protected void executeInternal(JobExecutionContext context) throws JobExecutionException {
	try {
		// 反射执行业务代码
		context.setResult(this.methodInvoker.invoke());
	}
	catch (InvocationTargetException ex) {
		if (ex.getTargetException() instanceof JobExecutionException) {
			// -> JobExecutionException, to be logged at info level by Quartz
			throw (JobExecutionException) ex.getTargetException();
		}
		else {
			// -> "unhandled exception", to be logged at error level by Quartz
			throw new JobMethodInvocationFailedException(this.methodInvoker, ex.getTargetException());
		}
	}
	catch (Exception ex) {
		// -> "unhandled exception", to be logged at error level by Quartz
		throw new JobMethodInvocationFailedException(this.methodInvoker, ex);
	}
}
```

## 五、综上总结

- quartz，即石英的意思，隐喻如石英钟般对时间的准确把握。
- 源码分析是一个很快乐的过程，这个快乐是分析完才能获得的快乐。纵横交互的背后是面向对象的高度解耦，对线程精彩的使用，将任务执行做成计划单，简直是一个超级棒的作品。
- 对于quartz.properties，简单场景下，开发者不用自定义配置，使用quartz默认配置即可，但在要求较高的使用场景中还是要自定义配置，比如通过org.quartz.threadPool.threadCount设置足够的线程数可提高多job场景下的运行性能。
- quartz 对任务处理高度解耦，job与trigger解藕，将任务本身和任务执行策略解藕，这样可以方便实现N个任务和M个执行策略自由组合。
- scheduler单独分离出来，相当于一个指挥官，可以从全局做调度，比如监听哪些trigger已经ready、分配线程等等。
- 外部链接：
	- [http://www.quartz-scheduler.org](http://www.quartz-scheduler.org)
    - [quartz-2.1.x/configuration](http://www.quartz-scheduler.org/documentation/quartz-2.1.x/configuration/)