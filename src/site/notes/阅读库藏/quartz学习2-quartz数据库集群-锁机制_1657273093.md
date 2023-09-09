---
{"title":"quartz学习2：quartz数据库集群-锁机制","url":"https://blog.csdn.net/wudiyong22/article/details/77435956","clipped_at":"2022-07-08 17:38:13","tags":["无"],"dg-publish":true,"permalink":"/阅读库藏/quartz学习2-quartz数据库集群-锁机制_1657273093/","dgPassFrontmatter":true}
---


# quartz学习2：quartz数据库集群-锁机制

一、quartz数据库锁

其中，QRTZ\_LOCKS就是Quartz集群实现同步机制的行锁表，其表结构如下：  

1.  \-\-QRTZ\_LOCKS表结构  
    
2.  CREATE TABLE \`QRTZ\_LOCKS\` (  
    
3.    \`LOCK\_NAME\` varchar(40) NOT NULL,  
    
4.     PRIMARY KEY (\`LOCK\_NAME\`)  
    
5.  ) ENGINE\=InnoDB DEFAULT CHARSET\=utf8;  
    
6.    
    
7.  \-\-QRTZ\_LOCKS记录  
    
8.  +\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-+   
    
9.  | LOCK\_NAME |  
    
10.  +\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-+   
    
11.  | CALENDAR\_ACCESS |  
    
12.  | JOB\_ACCESS |  
    
13.  | MISFIRE\_ACCESS |  
    
14.  | STATE\_ACCESS |  
    
15.  | TRIGGER\_ACCESS |  
    
16.  +\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-+

注：此表结构在2.2版本有新增字段，这里暂时不考虑。  
可以看出QRTZ\_LOCKS中有5条记录，代表5把锁，分别用于实现多个Quartz Node对Job、Trigger、Calendar访问的同步控制。    
关于行锁的机制：  
1、mysql >  set autocommit=0;    //先把mysql设置为不自动提交。  
2、 select \* from es\_locks where lock\_name = 'TRIGGER\_ACCESS' for update ;     //线程一通过for update 可以把这行锁住  
3、 select \* from es\_locks where lock\_name = 'TRIGGER\_ACCESS' for update ;     //线程二通过for update 无法获得锁，线程等待。  
4、commit;        //线程一通过commit 释放锁  
5、 //线程二可以访问到数据，线程不再等待。  
  
所以，通过这个机制，一次只能有一个线程来操作 加锁 -   操作  - 释放锁。  如果  操作  的时间过长的话，会带来集群间的主线程等待。  

数据库行锁是一种悲观锁，锁表时其它线程无法查询。  
  
源码中关于数据库集群加锁的方法有如下几种：  
1、executeInNonManagedTXLock方法的含义是自己管理事务，不让容器管理事务的加锁方法。  

1.  executeInNonManagedTXLock(  
    
2.              String lockName,  
    
3.              TransactionCallback<T\> txCallback , final TransactionValidator<T\> txValidator )

三个参数lockName的值是上面所说的 TRIGGER\_ACCESS，表示要加锁的类型。  
txCallback是加锁后再回调的方法。  
txValidator是验证方法，一般为null  
函数先执行加锁，再回调要操作的方法，然后再解锁。  
看一下源码：  

1.  if (lockName !\= null) {  
    
2.                  // If we aren't using db locks, then delay getting DB connection   
    
3.                  // until after acquiring the lock since it isn't needed.  
    
4.                  if (getLockHandler().requiresConnection()) {  
    
5.                      conn \= getNonManagedTXConnection();  
    
6.                  }  
    
7.                    
    
8.                  transOwner \= getLockHandler().obtainLock(conn, lockName);  
    
9.              }  
    
10.                
    
11.              if (conn \=\= null) {  
    
12.                  conn \= getNonManagedTXConnection();  
    
13.              }  
    
14.                
    
15.              final T result \= txCallback.execute(conn);  
    
16.              try {  
    
17.                  commitConnection(conn);  
    
18.              } catch (JobPersistenceException e) {  
    
19.                  rollbackConnection(conn);  
    
20.                  if (txValidator \=\= null || !retryExecuteInNonManagedTXLock(lockName, new TransactionCallback<Boolean\>() {  
    
21.                      @Override  
    
22.                      public Boolean execute(Connection conn) throws JobPersistenceException {  
    
23.                          return txValidator.validate(conn, result);  
    
24.                      }  
    
25.                  })) {  
    
26.                      throw e;  
    
27.                  }  
    
28.              }  
    
29.    
    
30.              Long sigTime \= clearAndGetSignalSchedulingChangeOnTxCompletion();  
    
31.              if(sigTime !\= null && sigTime \>\= 0) {  
    
32.                  signalSchedulingChangeImmediately(sigTime);  
    
33.              }  
    
34.                
    
35.              return result;  
    
36.          } catch (JobPersistenceException e) {  
    
37.              rollbackConnection(conn);
38.      throw e;
39.    } catch (RuntimeException e) {  
                rollbackConnection(conn);  
                throw new JobPersistenceException("Unexpected runtime exception: "  
                        + e.getMessage(), e);  
            } finally {  
                try {  
                    releaseLock(lockName, transOwner);  
                } finally {  
                    cleanupConnection(conn);  
                }  
            }  
      
            
    

  
2、如果不是通过这种回调方法的加锁，一般是：  
getLockHandler().obtainLock  
执行  
commitConnection(conn)  
releaseLock  
cleanupConnection  
  
  
二、源码分析锁  

目前代码中行锁只用到了 STATE\_ACCESS 和 TRIGGER\_ACCESS 这两种。  
  

1、TRIGGER\_ACCESS  
先了解一篇文章，通过源码来分析quartz是如何通过加锁来实现集群环境，触发器状态的一致性。   
[http://www.360doc.com/content/14/0926/08/15077656\_412418636.shtml  
](http://www.360doc.com/content/14/0926/08/15077656_412418636.shtml)可以看到触发器的操作主要用主线程StdScheduleThread来完成，不管是获取需要触发的30S内的触发器，还是触发过程。select和update触发器表时  
都会先加锁，后解锁。如果数据库资源竞争比较大的话，锁会影响整个性能。可以考虑将任务信息放在分布式内存，如redis上进行处理。数据库只是定时从redis上load数据下来做统计。  
参考：[quartz详解2：quartz由浅入深](http://blog.itpub.net/11627468/viewspace-1763498/)     查看第四章第1，2节  
实现都在JobStoreSupport类   

<table cellpadding="2" cellspacing="0" align="left" border="1" style="color:rgb(51,51,51); font-family:Arial; font-size:14px"><tbody><tr><td>加锁类型</td><td>加锁方法</td><td>底层数据库操作</td><td>备注</td></tr><tr><td rowspan="3">executeInNonManagedTXLock</td><td>acquireNextTrigger</td><td>selectTriggerToAcquire<br>selectTrigger<br>selectJobDetail<br>insertFiredTrigger</td><td>查询需要点火的trigger<br>选择需要执行的trigger加入到fired_trigger表</td></tr><tr><td>for执行&nbsp;triggerFired</td><td>selectJobDetail<br>selectCalendar<br>updateFiredTrigger<br>triggerExists&nbsp;updateTrigger</td><td>点火trigger<br>修改trigger状态为可执行状态。</td></tr><tr><td>recoverJobs</td><td>updateTriggerStatesFromOtherStates<br>hasMisfiredTriggersInState&nbsp;doUpdateOfMisfiredTrigger<br>selectTriggersForRecoveringJobs<br>selectTriggersInState<br>deleteFiredTriggers</td><td>非集群环境下重新执行<br>failed与misfired的trigger</td></tr><tr><td rowspan="2">retryExecuteInNonManagedTXLock</td><td>releaseAcquiredTrigger</td><td>updateTriggerStateFromOtherState<br>deleteFiredTrigger</td><td>异常情况下重新释放trigger到初使状态。</td></tr><tr><td>triggeredJobComplete</td><td>selectTriggerStatus<br>removeTrigger &nbsp;&nbsp;updateTriggerState<br>deleteFiredTrigger</td><td>触发JOB任务完成后的处理。</td></tr><tr><td rowspan="2">obtainLock</td><td>recoverMisfiredJobs</td><td>hasMisfiredTriggersInState&nbsp;doUpdateOfMisfiredTrigger</td><td>重新执行misfired的trigger<br>可以在启动时执行，也可以由misfired线程定期执行。</td></tr><tr><td>clusterRecover</td><td>selectInstancesFiredTriggerRecords<br>updateTriggerStatesForJobFromOtherState<br>storeTrigger<br>deleteFiredTriggers<br>selectFiredTriggerRecords<br>removeTrigger<br>deleteSchedulerState</td><td>集群有结点faied，让JOB能重新执行。</td></tr><tr><td rowspan="21">executeInLock<br>数据库集群里等同于<br>executeInNonManagedTXLock</td><td>storeJobAndTrigger</td><td>updateJobDetail&nbsp;insertJobDetail<br>triggerExists<br>selectJobDetail<br>updateTrigger&nbsp;insertTrigger</td><td>保存JOB和TRIGGER配置</td></tr><tr><td>storeJob</td><td>&nbsp;</td><td>保存JOB</td></tr><tr><td>removeJob</td><td>&nbsp;</td><td>删除JOB</td></tr><tr><td>removeJobs</td><td>&nbsp;</td><td>批量删除JOB</td></tr><tr><td>removeTriggers</td><td>&nbsp;</td><td>批量删除triggers</td></tr><tr><td>storeJobsAndTriggers</td><td>&nbsp;</td><td>保存JOB和多个trigger配置</td></tr><tr><td>removeTrigger</td><td>&nbsp;</td><td>删除trigger</td></tr><tr><td>replaceTrigger</td><td>&nbsp;</td><td>替换trigger</td></tr><tr><td>storeCalendar</td><td>&nbsp;</td><td>保存定时日期</td></tr><tr><td>removeCalendar</td><td>&nbsp;</td><td>删除定时日期</td></tr><tr><td>clearAllSchedulingData</td><td>&nbsp;</td><td>清除所有定时数据</td></tr><tr><td>pauseTrigger</td><td>&nbsp;</td><td>停止触发器</td></tr><tr><td>pauseJob</td><td>&nbsp;</td><td>停止任务</td></tr><tr><td>pauseJobs</td><td>&nbsp;</td><td>批量停止任务</td></tr><tr><td>resumeTrigger</td><td>&nbsp;</td><td>恢复触发器</td></tr><tr><td>resumeJob</td><td>&nbsp;</td><td>恢复任务</td></tr><tr><td>resumeJobs</td><td>&nbsp;</td><td>批量恢复任务</td></tr><tr><td>pauseTriggers</td><td>&nbsp;</td><td>批量停止触发器</td></tr><tr><td>resumeTriggers</td><td>&nbsp;</td><td>批量恢复触发器</td></tr><tr><td>pauseAll</td><td>&nbsp;</td><td>停止所有</td></tr><tr><td>resumeAll</td><td>&nbsp;</td><td>恢复所有</td></tr></tbody></table>

  
  
  
  
  
  
  
  
  
  
  
  
  
\---  
  
  
  
2、STATE\_TRIGGER  
实现都在JobStoreSupport类   

<table cellpadding="2" cellspacing="0" align="left" border="1" style="color:rgb(51,51,51); font-family:Arial; font-size:14px"><tbody><tr><td>加锁类型</td><td>加锁方法</td><td>底层数据库操作</td><td>备注</td></tr><tr><td rowspan="3">obtainLock</td><td>doCheckin</td><td>clusterCheckIn</td><td>判断集群状态<br>先用LOCK_STATE_ACCESS锁集群状态<br>再用LOCK_TRIGGER_ACCESS恢复集群运行</td></tr><tr><td>&nbsp;</td><td>&nbsp;</td><td>&nbsp;</td></tr></tbody></table>