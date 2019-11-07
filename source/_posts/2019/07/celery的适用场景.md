---
title: celery的适用场景
tags: python
categories: python后端
abbrlink: 84c8c02e
date: 2019-07-15 10:45:59
---

一、Celery 是什么?
---
1. Celery 是一个由 Python 编写的简单、灵活、可靠的用来处理大量信息的分布式系统,它同时提供操作和维护分布式系统所需的工具。

2. Celery 专注于实时任务处理，支持任务调度。

3. Celery 本身不是任务队列, 是管理分布式任务队列的工具. 它封装了操作常见任务队列的各种操作, 我们使用它可以快速进行任务队列的使用与管理.

二、主要特性：
---
1. 方便查询任务的进展情况，如执行结果，状态，消耗时间。

2. 可以利用功能齐全的后台管理进行任务的添加，删减等操作。

3. 可以进行并发操作。

4. 提供异常处理机制。

三、主要架构：
---
1. celery Beat:任务调度器，定时执行配置文件中的任务

2. celery Worker:任务执行的消费者，执行任务

3. Broker: 消息中间件，接收生产者产生的消息，存储队列并分发给任务方进行消费

4. Producer:生产者

5. Result Backend： 任务处理完后的结果存储

四、选择消息代理：
---

使用于生产环境的消息代理有 RabbitMQ 和 Redis, 官方推荐 RabbitMQ.





五、主要应用场景：
---
### 1.web应用
当触发事件需要较长时间处理完成，可以交给celery进行异步执行，执行后返回结果，这段时间不用等待，提高系统吞吐量和响应时间
```python
•    首先在项目虚拟环境里安装celery模块： 
o    pip install celery
　
•    然后在Django项目里新建一个python package，名字叫做celery_tasks，里面创建一个python文件tasks.py
o    首先确保你的电脑上已经安装有Redis数据库
o    在项目的虚拟环境中还需要 pip install redis
o    在tasks.py中的代码如下：其中第8-11行，是在使用不同的电脑运行celery的发起者、中间人、中间人时所需要用到的，这里我都是在一台电脑上运行，所以完全可以不写

 # 使用celery
 import time
 from celery import Celery
 from django.conf import settings
 from django.core.mail import send_mail
  
# 在任务处理者一端加这几句，如果使用的不是同一台电脑，django环境的初始化，一般启动项目的一端不需要加
import os
import django
os.environ.setdefault('DJANGO_SETTINGS_MODULE', 'dailyfresh.settings')
django.setup()

# 创建一个Celery类的实例对象
app = Celery('celery_tasks.tasks', broker='redis://127.0.0.1:6379/9')

# 定义任务函数
@app.task()
def send_register_active_mail(to_mail, username, token):
   """发送激活邮件"""
    # 组织邮件信息
     subject = '每日咨询欢迎信息'
     message = ''
     sender = settings.EMAIL_FROM
    receiver = [to_mail,]
    html_message = '<h1>{}，欢迎您成为每日咨询注册会员</h1>请点击下面链接激活您的用户<br><a href="http:127.0.0.1:8000/user/active/{}">http:127.0.0.1:8000/user/active/{}</a>'.format(
        username, token, token)
    send_mail(subject, message, sender, receiver, html_message=html_message)
     # 利用time模块模拟发了五秒钟邮件，原方法会页面加载5秒
    time.sleep(5)

o    修改原先视图函数里发送邮件的部分，修改的部分如下，原先发送邮件的代码被我们封装到了tasks.py文件中的send_register_active_mail函数中

# 导入上述创建的tasks.py文件中的异步发送邮件的方法
from celery_tasks.tasks import send_register_active_mail

# 发邮件
# subject = '每日咨询欢迎信息'
# message = ''
# sender = settings.EMAIL_FROM
# receiver = [email]
 # html_message =  '<h1>{}，欢迎您成为每日咨询注册会员</h1>请点击下面链接激活您的用户<br><a href="http:127.0.0.1:8000/user/active/{}">http:127.0.0.1:8000/user/active/{}</a>'.format(username, token, token)
# send_mail(subject, message, sender, receiver, html_message=html_message)
# 上面注释掉了原先发送邮件的代码，现在用刚导入的函数所替代
send_register_active_mail.delay(email, username, token)    

•    现在环境和代码都已经完成了，接下来在终端运行命
o    celery -A celery_tasks.tasks worker -l info -P eventlet
o    其中 -P enentlet 是为了防止 win10 运行出错而附加的
o    这样我们就启动了任务处理者
o    如果任务处理者和redis都在另一台计算机上，就需要把项目和环境拷贝过去，然后也是同样的命令启动，不过在不同的计算机上就一定不能少了tasks.py中的8-11行的内容，因为tasks.py文件中的操作用到了Django项目中的一些配置
o     
•    此时，在进行注册操作，虽然我们用 time.sleep函数模拟了邮件发送5s，但是当提交表单的时候发送邮件的操作已经是去异步处理了，页面几乎是没有太多延迟进行了刷新，这里就不再演示了，以下是在进行注册操作时，终端打印的信息
o    


```
###  2，后台定时任务处理，celery可以帮助我们在不同服务器进行定时任务管理。
#### 定时任务在Django中的配置与使用.

1. 首先,你需要在黑窗口下载以下扩展:
```python
　　pip install celery
　　pip install celery-with-redis
　　pip install django-celery
```
2. 在setting.py的结尾处，加入celery参数配置,这个celery可以提出来,单独做一个文件.
```python
import djcelery
djcelery.setup_loader()   #设置载入程序
BROKER_URL = 'redis://127.0.0.1:6379/6'    #代理人路由,它负责分发任务给worker去执行,我们这用Redis作为broker(代理人)
CELERY_IMPORTS = ('api.tasks')      #导入的文件
CELERY_TIMEZONE = TIME_ZONE       #时区
 #使用了django-celery默认的数据库调度模型,任务执行周期都被存在默认指定的orm数据库中
CELERYBEAT_SCHEDULER = 'djcelery.schedulers.DatabaseScheduler'  

在这里给他定义一个时间任务, 它会每隔一个固定时间周期去执行一次相应的task，比如隔1分钟，是周期性任务
from celery.schedules import crontab
CELERYBEAT_SCHEDULE = {
    #定时任务一：　每一分钟执行一次任务(del_redis_data)
    u'定义一个任务名为:del_redis_data': {
        "task": "app.tasks.del_redis_data",
        # "schedule": crontab(hour='*/1'),
        "schedule": crontab(minute=1),
        "args": (),
},
}

minute(分钟)  hour(小时,钟头)   dat_for_month(‘1’)(每月的第几天)
这些都可以套用,比如:
每月1号的6点启动
'schedule': crontab(hour=6, minute=0, day_of_month='1'),
```

3. 配置好settings之后,在你的文件目录下创建新文件夹task,可以在这里面定义你的任务,比如我定义了一个入库的任务,每分钟执行一次,这里他用装饰器@task(),可以看一下settings中定义的任务名为:del_redis_data,这里的函数也是del_redis_data,这里与settings最好一致
```python
@task
def del_redis_data():
    print('定时任务开始')
    conn = RedisSave()
    # 把属于今天的秒杀商品添加到redis

    # 查找当前时间
    now_time = datetime.now().strftime('%Y-%m-%d')
    activeList = Act.objects.filter(date=now_time).all()

    # 查询当前时间的场次
    for act in activeList:
        timelist = Time.objects.filter(act_id=act.id).all()  # 该活动下的所有场次
        alist = []
        for time in timelist:
            print(time.start.strftime('%H:%M:%S'))
            info = {}
            sklist = Sk.objects.filter(time=time, act_id=act.id).all()
            skList = SkSerializersModel(sklist, many=True).data
            mapping = {
                "start_time": time.start.strftime('%H:%M:%S'),
                "end_time": time.end.strftime('%H:%M:%S'),
                "content": skList,
                'now_time': now_time,
                'time1':'',
                'time2':'',

            }
            info = {
                'name': time.name,
                'course': mapping
            }
            alist.append(info)
        info = json.dumps(alist)
        print(info)
        conn.setex(act.title, info, 84600)

    print("定时任务结束")
    return HttpResponse('ok')
```


4. 当你写完这些,你就可以启动定时任务了,启动终端,切换到Django项目的根目录下，运行
```python
# 启动web服务
python manage.py runserver 

# 启动celery woker
python manage.py celery worker -l info 
 
# 启动beat, 执行定时任务
celery beat -A 项目名 -l info.
```