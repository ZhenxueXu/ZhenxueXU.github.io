---
layout: post
title: AirFlow定时调度执行Talend ETL任务
subtitle: Using Airflow to Manage Talend ETL Jobs
date: 2018-07-28T00:00:00.000Z
author: Xu Zhenxue
header-img: img/post-bg-universe.jpg
catalog: true
tags:
  - ETL
  - 任务调度
  - Airflow
comments:
  - author:
      type: github
      displayName: ZhenxueXu
      url: 'https://github.com/ZhenxueXu'
      picture: 'https://avatars0.githubusercontent.com/u/18114535?v=4&s=73'
    content: '&#x8BC4;&#x8BBA;&#x529F;&#x80FD;&#x6D4B;&#x8BD5;'
    date: 2018-07-28T17:04:51.148Z
  - author:
      type: github
      displayName: linkcheng
      url: 'https://github.com/linkcheng'
      picture: 'https://avatars0.githubusercontent.com/u/8966066?v=4&s=73'
    content: >
      &#x4F60;&#x597D;&#xFF0C;&#x8BF7;&#x6559;&#x4E00;&#x4E2A;&#x5173;&#x4E8E;
      airflow &#x4E2D;&#x4E00;&#x4E2A; DAG &#x4E2D;&#x5404;&#x4E2A; Operator
      &#x7684;&#x4F9D;&#x8D56;&#x5173;&#x7CFB;&#x95EE;&#x9898;&#x3002;


      default_args = {
          &apos;owner&apos;: &apos;python&apos;,
          &apos;depends_on_past&apos;: True,
          &apos;start_date&apos;: datetime(2019, 1, 17, 10, 45, 0),
          &apos;email&apos;: [&apos;example@qq.com&apos;],
          &apos;email_on_failure&apos;: True,
          &apos;email_on_retry&apos;: False,
          &apos;retries&apos;: 1,
          &apos;retry_delay&apos;: timedelta(minutes=5),
          &apos;queue&apos;: &apos;test4&apos;,
          # &apos;pool&apos;: &apos;backfill&apos;,
          # &apos;priority_weight&apos;: 10,
          # &apos;end_date&apos;: datetime(2020, 1, 1),
      }


      dag = DAG(&apos;sync_cashbill_new&apos;, default_args=default_args,
      schedule_interval=timedelta(minutes=15))


      start = BashOperator(
          task_id=&apos;activate_virtualenv&apos;,
          bash_command=&apos;source /data/virtualenv/test-bi-transfer/bin/activate&apos;,
          dag=dag)

      go_lib = BashOperator(
          task_id=&apos;go_lib_directory&apos;,
          bash_command=&apos;cd /data/code/bi-transfer/lib&apos;,
          dag=dag)

      sync = BashOperator(
          task_id=&apos;sync_CashBill&apos;,
          bash_command=&apos;sudo python2.7 mysql_sync.py update -t v2.CashBill&apos;,
          dag=dag)

      end = EmailOperator(
          task_id=&apos;send_email&apos;,
          to=&apos;zheng.long@shoufuyou.com&apos;,
          subject=&apos;mysql_sync done&apos;,
          html_content=&apos;mysql_sync success&apos;,
          dag=dag)

      start &gt;&gt; go_lib &gt;&gt; sync &gt;&gt; end


      &#x4EE5;&#x4E0A;&#x662F;&#x6211;&#x5B9A;&#x4E49;&#x7684;
      DAG&#xFF0C;&#x73B0;&#x5728;&#x6709;&#x4E2A;&#x95EE;&#x9898;&#x662F; sync
      &#x8FD9;&#x4E00;&#x6B65;&#x4E0D;&#x80FD;&#x6267;&#x884C;&#x6210;&#x529F;&#xFF0C;&#x539F;&#x56E0;&#x662F;&#x8FD9;&#x4E00;&#x6B65;&#x9700;&#x8981;&#x4F9D;&#x8D56;&#x524D;&#x4E24;&#x6B65;&#x6267;&#x884C;&#x6210;&#x529F;&#x5E76;&#x4E14;&#x9700;&#x8981;&#x4E4B;&#x524D;&#x7684;&#x7ED3;&#x679C;&#xFF0C;&#x5982;&#x6FC0;&#x6D3B;&#x4E00;&#x4E2A;python&#x865A;&#x62DF;&#x73AF;&#x5883;&#x4EE5;&#x53CA;&#x5207;&#x6362;&#x5230;&#x6307;&#x5B9A;&#x76EE;&#x5F55;&#x3002;&#x6211;&#x60F3;&#x8BF7;&#x6559;&#x4E00;&#x4E0B;&#x5982;&#x4F55;&#x53EF;&#x4EE5;&#x5728;&#x7B2C;&#x4E09;&#x6B65;&#x4F7F;&#x7528;&#x524D;&#x4E24;&#x6B65;&#x7684;&#x7ED3;&#x679C;&#xFF0C;&#x5C31;&#x662F;&#x4F7F;&#x7528;&#x6FC0;&#x6D3B;&#x7684;&#x865A;&#x62DF;&#x73AF;&#x5883;&#x4EE5;&#x53CA;&#x8FDB;&#x5165;&#x6307;&#x5B9A;&#x76EE;&#x5F55;&#xFF1F;


      &#x975E;&#x5E38;&#x8C22;&#x8C22;&#x3002;
    date: 2019-01-17T04:12:01.842Z

---

> 滚动阅读全文  

## AirFlow调度平台简介

airflow 是一个编排、调度和监控工作流的平台，由Airbnb开源，现在在Apache Software Foundation 孵化。airflow将工作流编排为tasks组成的有向无环图（DAGs），调度器在一组workers上按照指定的依赖关系执行tasks。同时，airflow提供了丰富的命令行工具和简单易用的用户界面以便用户查看和操作，并且airflow提供了监控和报警系统

## AirFlow基础概念  
  
 Airflow主要是将工作流的相关信息定义到一个Python文件中，airflow根据文件中的定义信息执行工作流，在Airflow pipeline定义中，主要涉及两个类： <code color='red' >DAG</code>，<code >Operator</code>

<code color='red' style='background-color:#EAEDED;'>DAG</code> : 有向无环图，它将定义的任务按照依赖关系组织起来   
<code color='red' style='background-color:#EAEDED;'>Operator</code>：用来描述每个任务具体做的事，airflow内置了很多operator，如<code color='red' style='background-color:#EAEDED;'>BashOperator</code> 执行一个bash 命令，<code color='red' style='background-color:#EAEDED;'>PythonOperator</code> 调用任意的Python 函数，<code color='red' style='background-color:#EAEDED;'>EmailOperator</code> 用于发送邮件，<code color='red' style='background-color:#EAEDED;'>HTTPOperator</code> 用于发送HTTP请求， <code color='red' style='background-color:#EAEDED;'>SqlOperator</code> 用于执行SQL命令…同时，用户可以自定义Operator，这给用户提供了极大的便利性。  
通过DAG和Operator结合起来就可以构建复杂的工作流了  

## Talend简介

Talend是一个开源的ELT任务构建工具，可以通过简单拖拽的方式设计复杂的ETL任务并自动生成Java代码，设计完成后可以通过构建导出ETL任务java源码和可直接执行jar包及执行jar包的shell和bat文件。

接下来，我们通过Talend 桌面软件构建一个简单的ELT任务，然后通过airflow UI执行这个ETL任务

## Airflow调度ETL任务实例

### 1.通过Talend 设计、构建ETL任务

打开Talend设计如下简单的ETL任务，将一个CSV文件导入到数据库中：
![image](http://zhenxuexu.github.io/img/etl.png)

然后，将这个ETL任务构建并导出，目录结构如下：

![image](http://zhenxuexu.github.io/img/job-dir.png)

我们使用在linux环境中执行shell文件的方式执行ETL任务。

### 2.定义Airflow pipeline
我们在airflow 指定的dags目录下新建python文件来定义airflow pipeline

代码如下:


```
import airflow
from datetime import datetime
from airflow.operators.bash_operator import BashOperator
from airflow.models import DAG
from airflow.utils.email import send_email

#shell命令执行.sh文件

job_commd = "/home/airflow/jobs/AirFlowTestJob_0.1/AirFlowTestJob/AirFlowTestJob_run.sh '/home/airflow/jobs/AirFlowTestJob_0.1/AirFlowTestJob/'"

#dag中的一些参数设置
default_args = {
    'owner': 'Airflow',
    'depends_on_past': False,
    'start_date': datetime(2018, 7, 23),    
    'email': ['VALID_EMAIL_ID'],
    'email_on_failure': True,
    'email_on_success': True,
    'retries':3,
}

dag = DAG('csv_to_data', default_args=default_args)

#定义ETL任务,多个任务时定义多个Operator
t1 = BashOperator(
    task_id='csv_to_database',
    bash_command=job_commd,
    dag=dag
)
```

### 3.调度执行ETL任务
完成以上工作，我们启动airflow webserver服务器后,便可以在airflow UI界面看到我们定义的ETl任务<font color='red' style='background-color:#EAEDED;'>csv_to_data</font>，可以进行相应操作。

![image](http://zhenxuexu.github.io/img/airflow.png)
