# WyzLearningNotebook   

Hey，欢迎来到Wyz的笔记本！:smirk:
---
   
@2018/5/9

**指标**

指标：用于衡量事物发展程度的单位或方法，它还有个IT上常用的名字，也就是度量。例如：人口数、GDP、收入、用户数、利润率、留存率、覆盖率等。  
很多公司都有自己的KPI指标体系，就是通过几个关键指标来衡量公司业务运营情况的好坏。

指标需要经过加和、平均等汇总计算方式得到，并且是需要在一定的前提条件进行汇总计算，如时间、地点、范围，也就是我们常说的统计口径与范围。

指标可以分为绝对数指标和相对数指标，绝对数指标反映的是规模大小的指标，如人口数、GDP、收入、用户数，而相对数指标主要用来反映质量好坏的  
指标，如利润率、留存率、覆盖率等。我们分析一个事物发展程度就可以从数量跟质量两个角度入手分析，以全面衡量事物发展程度。

刚才说过，指标用于衡量事物发展程度，那这个程度是好还是坏，这就需要通过不同维度来对比，才能知道是好还是坏。

**维度**

维度：是事物或现象的某种特征，如性别、地区、时间等都是维度。其中时间是一种常用、特殊的维度，通过时间前后的对比，就可以知道事物的发展是  
好了还是坏了，如用户数环比上月增长10%、同比去年同期增长20%，这就是时间上的对比，也称为纵比;

另一个比较就是横比，如不同国家人口数、GDP的比较，不同省份收入、用户数的比较、不同公司、不同部门之间的比较，这些都是同级单位之间的比较，  
简称横比;

维度可以分为定性维度跟定量维度，也就是根据数据类型来划分，数据类型为字符型(文本型)数据，就是定性维度，如地区、性别都是定性维度;数据类型  
为数值型数据的，就为定量维度，如收入、年龄、消费等，一般我们对定量维度需要做数值分组处理，也就是数值型数据离散化，这样做的目的是为了使规  
律更加明显，因为分组越细，规律就越不明显，最后细到成最原始的流水数据，那就无规律可循。

最后强调一点，只有通过事物发展的数量、质量两大方面，从横比、纵比角度进行全方位的比较，我们才能够全面的了解事物发展的好坏。

```
select * from
(select * from 
gdm.gdm_teacher_base_info
where 
dt='2018-05-08'
and school_id=255767
and authentication_state=1
and subject='ENGLISH'
and is_cheating_teacher=1
and is_true_teacher=1)a
left join(
select * from
vbawork_business.vb_user_session_online_stat
where
dt='2018-05-08'
and user_type='1'
)b
on a.teacher_id=b.user_id
```

@2018/5/10

**开启hadoop**  
进入sbin文件夹  
输入hadoop namenode -format，格式化namenode  
输入./start-all.sh开启所有服务  
输入jps可以查看所有Java进程和pid  
用浏览器访问localhost：8088和localhost：50070  

```
--2.中学全量数据
select d.group_name,a.*,c.*,e.*,f.*,h.*,g.group_adm_name,g.school_adm_name
from
(select distinct  school_id,school_name,school_level,province_name,city_name,county_name,student_total_count,
student_authed_count,tm_inc_reg_stu_count,tm_inc_auth_stu_count
from gdm.gdm_crm_school_dimension
where dt='2018-05-09'
and disabled=false
and school_level='MIDDLE'
)a
left join
(select distinct id,authentication_state,vip   ---authentication_state为学校鉴定状态，vip为学校级别，该字段有问题，弃用
from fdm.fdm_mysql_vox_school_chain
where dp='ACTIVE'
)c on a.school_id=c.id
left join
(select distinct group_name,province_name
from vbawork_business.vb_agent_group_region_ref
where dt='2018-05'
and group_name in ('北区','南区')
and group_name is not null
) d on a.province_name=d.province_name
left join 
(select school_id,
count (distinct case when (substr(first_login_time,1,7)='2018-04' or substr(first_login_time,1,7)='2018-05') then teacher_id end) register_num_Mar,
count(distinct case when (substr(first_login_time,1,7)='2018-04' or substr(first_login_time,1,7)='2018-05') and auth_state =1 then teacher_id end) auth_num_Mar,
count(distinct case when first_login_time is not null then teacher_id end) register_num_all,
count(distinct case when auth_state =1 then teacher_id end) auth_num_all
from gdm.gdm_teacher_login_status  --老师登录信息表
where dt='2018-05-09'
group by school_id
) e on a.school_id=e.school_id
left join
(select distinct school_id,auth_fin_3_suit_num,new_settle_num,short_flow_stu_num,long_flow_stu_num,auth_fin_1_suit_num,auth_fin_2_suit_num,no_auth_fin_1_suit_num,
no_auth_fin_2_suit_num,no_auth_fin_3_suit_num
from vbawork_business.vb_17_market_school_kpi_day   --学校业绩表 
where dt='2018-04-30'
and subject='ENGLISH'
)f on a.school_id=f.school_id
left join
(select distinct school_id,auth_fin_3_suit_num,new_settle_num,short_flow_stu_num,long_flow_stu_num,auth_fin_1_suit_num,auth_fin_2_suit_num,no_auth_fin_1_suit_num,
no_auth_fin_2_suit_num,no_auth_fin_3_suit_num
from vbawork_business.vb_17_market_school_kpi_day   --学校业绩表 
where dt='2018-05-09'
and subject='ENGLISH'
)h on a.school_id=h.school_id

left join
(select distinct school_id,trim(school_adm_name) school_adm_name,--每个学校的专员
  trim(group_adm_name) group_adm_name,--市经理
group_name,--所属部门
region_name--所属大区
from tmp.school_region_ref_info
where dt='2018-05-09'
---and school_level='1' 
)g on a.school_id=g.school_id
```

**进入MySQL数据库**  
在终端输入/usr/local/mysql/bin/mysql -u root -p  

@2018/5/16  

```  
SET mapreduce.map.memory.mb=4096;  
SET mapreduce.reduce.memory.mb=8192;  
SET mapreduce.map.java.opts=-Xmx9216m;
SET hive.exec.parallel=TRUE;  
```  

@2018/6/23   
好久没更新了哈   
最近在研究python数据可视化，比较困扰我的一个问题是matplotlib中文显示乱码的问题   
https://blog.csdn.net/cartoonjh/article/details/79050484   
这个博客的思路是可行的   
还有从csv导入数据时，应转换为gbk编码，代码如下：
```   
df = pd.read_csv("/路径/data.csv",encoding="gbk")
```   
附上一个运行成功的代码：   
```   
# -*- coding: utf-8 -*- 
from pylab import mpl
import json
import pandas as pd  
import numpy as np #科学计算
from pandas import Series,DataFrame
import matplotlib.pyplot as plt
from matplotlib.font_manager import _rebuild
_rebuild()
mpl.rcParams['font.sans-serif']=[u'SimHei']
mpl.rcParams['axes.unicode_minus']=False

df = pd.read_csv("/路径/data.csv",encoding="gbk")
df.other_city.value_counts().plot(kind='kde')
plt.title(u"专员城市分布")
plt.ylabel(u"人数")
plt.show()
```   

@2018/6/27   
最近由于要用excel分析的同类型数据太多，需要重复劳动，尝试用python代替excel操作…感觉有点像机器学习的数据预处理过程，哈哈
```
import numpy as np
import pandas as pd 

df=pd.DataFrame(pd.read_csv('/Users/wuyizhan/Desktop/统测报告/data.csv'))

df.head() 
df.info() #查看表格基本信息

datause=df[['学生ID','学校名称','班级名称','总成绩']]
datause['班级数统计']=datause['学校名称']+datause['班级名称']

#classcount表，计算班级数开始

classcountB=datause[['学校名称','班级数统计']]
classcountL=classcountB.drop_duplicates(subset='班级数统计',keep='first')
classcount=pd.pivot_table(classcountL,index=['学校名称'],values=['班级数统计'],aggfunc=len)
classcount

#计算班级数结束

dfnew=pd.pivot_table(datause,index=['学校名称'],values=['学生ID','总成绩'],aggfunc={'学生ID':len,'总成绩':np.sum}) #数据透视表
dfnew

dfnew.columns=['学生人数','学校总成绩和']
dfnew['学校满分']=dfnew['学生人数']*100  #总分数值可改
dfnew['学校得分率']=dfnew['学校总成绩和']/dfnew['学校满分']
dfnew['学生平均得分']=dfnew['学校总成绩和']/dfnew['学生人数']
dfnew['学生平均失分']=100-dfnew['学生平均得分'] #总分数值可改

dfTable=pd.merge(dfnew,classcount,how='inner',on='学校名称') #将dfnew和classcount融合
dfTable

lastTable=dfTable[['学生人数','班级数统计','学生平均得分','学生平均失分','学校得分率']]
lastTable

lastTable.to_csv ("/Users/wuyizhan/Desktop/统测报告/result.csv" , encoding="utf-8-sig")
```   

@2018/6/29   
```   
import numpy as np
import pandas as pd 

#整体合并用outer取交集

#读取数据

df=pd.DataFrame(pd.read_csv('/Users/wuyizhan/Desktop/统测报告/data.csv'))

#删除无用的列

datause=df[['年级','学生ID','学校名称','班级名称','总成绩','听单词选择图片','听句子选句子','听句子选答语','听句子判断图片','听对话判断']]
datause['班级数统计']=datause['学校名称']+datause['班级名称']
datause

#计算标准差

gp=datause.groupby('学校名称')['总成绩'].std()
gpstd=gp.to_frame()
gpstd.columns=['该校该年级平均分标准差']

#计算优、良、合格、待合格人数

thebest=datause[datause['总成绩']>42.5] #计算各校优秀人数
dfbest=pd.pivot_table(thebest,index=['学校名称'],values=['学生ID'],aggfunc=len)
dfbest.columns=['该校该年级优秀学生数']

theok=datause[(datause['总成绩']<42.5)&(datause['总成绩']>37.5)]
dfok=pd.pivot_table(theok,index=['学校名称'],values=['学生ID'],aggfunc=len)
dfok.columns=['该校该年级良学生数']

thepass=datause[(datause['总成绩']<37.5)&(datause['总成绩']>30)]
dfpass=pd.pivot_table(thepass,index=['学校名称'],values=['学生ID'],aggfunc=len)
dfpass.columns=['该校该年级合格学生数']

thebad=datause[datause['总成绩']<30]
dfbad=pd.pivot_table(thebad,index=['学校名称'],values=['学生ID'],aggfunc=len)
dfbad.columns=['该校该年级待合格学生数']

pepcount1=pd.merge(dfbest,dfok,how='outer',on='学校名称') #子表融合
pepcount2=pd.merge(pepcount1,dfpass,how='outer',on='学校名称')
pepcount=pd.merge(pepcount2,dfbad,how='outer',on='学校名称')
pepcount

#classcount 计算班级数

classcountB=datause[['学校名称','班级数统计']]
classcountL=classcountB.drop_duplicates(subset='班级数统计',keep='first')
classcount=pd.pivot_table(classcountL,index=['学校名称'],values=['班级数统计'],aggfunc=len)
classcount

#dfnew 总分数据透视表，计算平均分

dfnew=pd.pivot_table(datause,index=['学校名称'],values=['学生ID','总成绩'],aggfunc={'学生ID':len,'总成绩':np.sum})
dfnew

dfnew.columns=['学生人数','学校总成绩和']
dfnew['学校满分']=dfnew['学生人数']*50  #总分数值可改,此处为50
dfnew['学校得分率']=dfnew['学校总成绩和']/dfnew['学校满分']
dfnew['学生平均得分']=dfnew['学校总成绩和']/dfnew['学生人数']
dfnew['学生平均失分']=50-dfnew['学生平均得分'] #总分数值可改

dfnew1=dfnew[['学生人数','学生平均得分','学生平均失分','学校得分率']]

#wordpic 听单词选择图片数据透视表

wordpic=pd.pivot_table(datause,index=['学校名称'],values=['学生ID','听单词选择图片'],aggfunc={'学生ID':len,'听单词选择图片':np.sum})
wordpic

wordpicc=datause[datause['听单词选择图片']==10] #计算满分人数
wordpiccount=pd.pivot_table(wordpicc,index=['学校名称'],values=['学生ID'],aggfunc=len)
wordpiccount.columns=['听单词选择图片满分人数']

wordpic.columns=['wordpic总分','学生人数']  #总分是指将该学校所有参与该题的学生得分全部加起来
wordpic['wordpic满分']=wordpic['学生人数']*10  #听单词选图片分值可改
wordpic['听单词选择图片得分率']=wordpic['wordpic总分']/wordpic['wordpic满分']
wordpic['听单词选择图片平均得分']=wordpic['wordpic总分']/wordpic['学生人数']
wordpic['听单词选择图片平均失分']=10-wordpic['听单词选择图片平均得分'] #听单词选图片分值可改

wordpicL=pd.merge(wordpic,wordpiccount,how='outer',on='学校名称')

wordpic1=wordpicL[['听单词选择图片平均得分','听单词选择图片平均失分','听单词选择图片得分率','听单词选择图片满分人数']]

#sensen 听句子选句子数据透视表

sensen=pd.pivot_table(datause,index=['学校名称'],values=['学生ID','听句子选句子'],aggfunc={'学生ID':len,'听句子选句子':np.sum})
sensen

sensenc=datause[datause['听句子选句子']==10] #计算满分人数
sensencount=pd.pivot_table(sensenc,index=['学校名称'],values=['学生ID'],aggfunc=len)
sensencount.columns=['听句子选句子满分人数']

sensen.columns=['sensen总分','学生人数']  #总分是指将该学校所有参与该题的学生得分全部加起来
sensen['sensen满分']=sensen['学生人数']*10  #听句子选句子分值可改
sensen['听句子选句子得分率']=sensen['sensen总分']/sensen['sensen满分']
sensen['听句子选句子平均得分']=sensen['sensen总分']/sensen['学生人数']
sensen['听句子选句子平均失分']=10-sensen['听句子选句子平均得分'] #听句子选句子分值可改

sensenL=pd.merge(sensen,sensencount,how='outer',on='学校名称')

sensen1=sensenL[['听句子选句子平均得分','听句子选句子平均失分','听句子选句子得分率','听句子选句子满分人数']]

#senans 听句子选答语数据透视表

senans=pd.pivot_table(datause,index=['学校名称'],values=['学生ID','听句子选答语'],aggfunc={'学生ID':len,'听句子选答语':np.sum})
senans

senansc=datause[datause['听句子选答语']==10] #计算满分人数
senanscount=pd.pivot_table(senansc,index=['学校名称'],values=['学生ID'],aggfunc=len)
senanscount.columns=['听句子选答语满分人数']

senans.columns=['senans总分','学生人数']  #总分是指将该学校所有参与该题的学生得分全部加起来
senans['senans满分']=senans['学生人数']*10  #听句子选答语分值可改
senans['听句子选答语得分率']=senans['senans总分']/senans['senans满分']
senans['听句子选答语平均得分']=senans['senans总分']/senans['学生人数']
senans['听句子选答语平均失分']=10-senans['听句子选答语平均得分'] #听句子选答语分值可改

senansL=pd.merge(senans,senanscount,how='outer',on='学校名称')

senans1=senansL[['听句子选答语平均得分','听句子选答语平均失分','听句子选答语得分率','听句子选答语满分人数']]

#senpic 听句子判断图片数据透视表

senpic=pd.pivot_table(datause,index=['学校名称'],values=['学生ID','听句子判断图片'],aggfunc={'学生ID':len,'听句子判断图片':np.sum})
senpic

senpicc=datause[datause['听句子判断图片']==10] #计算满分人数
senpiccount=pd.pivot_table(senpicc,index=['学校名称'],values=['学生ID'],aggfunc=len)
senpiccount.columns=['听句子判断图片满分人数']

senpic.columns=['senpic总分','学生人数']  #总分是指将该学校所有参与该题的学生得分全部加起来
senpic['senpic满分']=senpic['学生人数']*10  #听句子判断图片分值可改
senpic['听句子判断图片得分率']=senpic['senpic总分']/senpic['senpic满分']
senpic['听句子判断图片平均得分']=senpic['senpic总分']/senpic['学生人数']
senpic['听句子判断图片平均失分']=10-senpic['听句子判断图片平均得分'] #听句子判断图片分值可改

senpicL=pd.merge(senpic,senpiccount,how='outer',on='学校名称')

senpic1=senpicL[['听句子判断图片平均得分','听句子判断图片平均失分','听句子判断图片得分率','听句子判断图片满分人数']]

#conver 听对话判断数据透视表

conver=pd.pivot_table(datause,index=['学校名称'],values=['学生ID','听对话判断'],aggfunc={'学生ID':len,'听对话判断':np.sum})
conver

converc=datause[datause['听对话判断']==10] #计算满分人数
convercount=pd.pivot_table(converc,index=['学校名称'],values=['学生ID'],aggfunc=len)
convercount.columns=['听对话判断满分人数']

conver.columns=['conver总分','学生人数']  #总分是指将该学校所有参与该题的学生得分全部加起来
conver['conver满分']=conver['学生人数']*10  #听对话判断分值可改
conver['听对话判断得分率']=conver['conver总分']/conver['conver满分']
conver['听对话判断平均得分']=conver['conver总分']/conver['学生人数']
conver['听对话判断平均失分']=10-conver['听对话判断平均得分'] #听对话判断分值可改

converL=pd.merge(conver,convercount,how='outer',on='学校名称')

conver1=converL[['听对话判断平均得分','听对话判断平均失分','听对话判断得分率','听对话判断满分人数']]

#将所有子表融合

last1=pd.merge(dfnew1,classcount,how='outer',on='学校名称')
last2=pd.merge(last1,wordpic1,how='outer',on='学校名称')
last3=pd.merge(last2,sensen1,how='outer',on='学校名称')
last4=pd.merge(last3,senans1,how='outer',on='学校名称')
last5=pd.merge(last4,senpic1,how='outer',on='学校名称')
last6=pd.merge(last5,conver1,how='outer',on='学校名称')
last7=pd.merge(last6,pepcount,how='outer',on='学校名称')
last=pd.merge(last7,gpstd,how='outer',on='学校名称')

last.to_csv ("/Users/wuyizhan/Desktop/统测报告/result.csv",encoding="utf-8-sig")
```

@2018/8/15   

**利用Python按列拆分excel表格**   

```   
# -*- coding: utf-8 -*-
import sys

reload(sys)
sys.setdefaultencoding("utf8")

import pandas as pd

data = pd.read_excel("/Users/wuyizhan/Desktop/zhongxue@20180815/data.xlsx")
rows = data.shape[0]  # 获取行数 shape[1]获取列数
department_list = []

for i in range(rows):
    temp = data[u"市"][i]
    if temp not in department_list:
        department_list.append(temp)  # 将销售部门的分类存在一个列表中

for department in department_list:
    new_df = pd.DataFrame()

    for i in range(0, rows):
        if data[u"市"][i] == department:
            new_df = pd.concat([new_df, data.iloc[[i], :]], axis=0, ignore_index=True)

    new_df.to_excel(str(department) + ".xls", index=False, encoding="utf-8-sig")  # 将每个销售部门存成一个新excel
```   
2018/09/16   
git基本操作   
```
wuyizhandeMacBook-Pro:admin-panel wuyizhan$ git add -A
wuyizhandeMacBook-Pro:admin-panel wuyizhan$ git commit -m 'test'
[master b660e9b] test
 1 file changed, 1 insertion(+), 2 deletions(-)
wuyizhandeMacBook-Pro:admin-panel wuyizhan$ git pull
Already up to date.
wuyizhandeMacBook-Pro:admin-panel wuyizhan$ git push origin master
```

2018/09/22
```
wuyizhandeMacBook-Pro:admin-panel wuyizhan$ git add -A
wuyizhandeMacBook-Pro:admin-panel wuyizhan$ git commit -m 'submit'
[master fd4d00d] submit
 2 files changed, 9 insertions(+), 9 deletions(-)
wuyizhandeMacBook-Pro:admin-panel wuyizhan$ git push origin master
To gitee.com:animeboy/admin-panel.git
 ! [rejected]        master -> master (fetch first)
error: failed to push some refs to 'git@gitee.com:animeboy/admin-panel.git'
hint: Updates were rejected because the remote contains work that you do
hint: not have locally. This is usually caused by another repository pushing
hint: to the same ref. You may want to first integrate the remote changes
hint: (e.g., 'git pull ...') before pushing again.
hint: See the 'Note about fast-forwards' in 'git push --help' for details.
wuyizhandeMacBook-Pro:admin-panel wuyizhan$ git pull
remote: Enumerating objects: 124, done.
remote: Counting objects: 100% (88/88), done.
remote: Compressing objects: 100% (42/42), done.
remote: Total 46 (delta 23), reused 0 (delta 0)
Unpacking objects: 100% (46/46), done.
From gitee.com:animeboy/admin-panel
   feabd80..3090db6  master     -> origin/master
Merge made by the 'recursive' strategy.
 mock.js                                                  |  14 ++-
 mock/get-doc-list.json                                   |  16 +++
 mock/get-task-comments.json                              |  47 ++++++++
 src/components/comment/index.vue                         |  78 +++++---------
 src/const/apilist.js                                     |  18 +++-
 src/const/routes.js                                      |   7 +-
 src/main.js                                              |   2 +-
 src/store/index.js                                       |  16 +++
 src/store/modules/project.js                             | 155 +++++++++++++++++++++++----
 src/styles/index.scss                                    |  47 ++++++++
 src/views/project/components/DocList.vue                 |  34 ++++++
 src/views/project/components/DocSearch.vue               |  37 +++++--
 src/views/project/components/TaskDetail/TaskDoc.vue      |  11 +-
 src/views/project/components/TaskDetail/TaskProgress.vue |   6 +-
 src/views/project/components/TaskDetail/TaskTurnover.vue |  12 +--
 src/views/project/components/TaskList.vue                |  47 +-------
 src/views/project/components/TaskSearch.vue              |  73 +++----------
 src/views/project/detail/index.vue                       |  12 ++-
 src/views/project/docs/index.vue                         |  63 +++++------
 src/views/project/list/index.vue                         |  62 +++++++++--
 src/views/project/tpl/components/CreateTpl.vue           | 122 +++++++++++++++++----
 src/views/project/tpl/components/TplCard.vue             |  15 ++-
 src/views/project/tpl/index.vue                          |  66 ++++--------
 23 files changed, 633 insertions(+), 327 deletions(-)
 create mode 100644 mock/get-doc-list.json
 create mode 100644 mock/get-task-comments.json
wuyizhandeMacBook-Pro:admin-panel wuyizhan$ git push origin master
Counting objects: 13, done.
Delta compression using up to 8 threads.
Compressing objects: 100% (11/11), done.
Writing objects: 100% (13/13), 1.16 KiB | 1.16 MiB/s, done.
Total 13 (delta 8), reused 0 (delta 0)
remote: Powered by Gitee.com
To gitee.com:animeboy/admin-panel.git
   3090db6..aebd55f  master -> master
wuyizhandeMacBook-Pro:admin-panel wuyizhan$ git pull
Already up to date.
wuyizhandeMacBook-Pro:admin-panel wuyizhan$ pm2 start mock
[PM2] Spawning PM2 daemon with pm2_home=/Users/wuyizhan/.pm2
[PM2] PM2 Successfully daemonized
[PM2] Starting /Users/wuyizhan/Desktop/admin-panel/mock in fork_mode (1 instance)
[PM2] Done.
┌──────────┬────┬──────┬──────┬────────┬─────────┬────────┬─────┬───────────┬──────────┬──────────┐
│ App name │ id │ mode │ pid  │ status │ restart │ uptime │ cpu │ mem       │ user     │ watching │
├──────────┼────┼──────┼──────┼────────┼─────────┼────────┼─────┼───────────┼──────────┼──────────┤
│ mock     │ 0  │ fork │ 9388 │ online │ 0       │ 0s     │ 0%  │ 19.1 MB   │ wuyizhan │ disabled │
└──────────┴────┴──────┴──────┴────────┴─────────┴────────┴─────┴───────────┴──────────┴──────────┘
 Use `pm2 show <id|name>` to get more details about an app
wuyizhandeMacBook-Pro:admin-panel wuyizhan$ vi /usr/local/etc/nginx/servers/admin.conf
wuyizhandeMacBook-Pro:admin-panel wuyizhan$ nginx -s reload
nginx: [error] open() "/usr/local/var/run/nginx.pid" failed (2: No such file or directory)
wuyizhandeMacBook-Pro:admin-panel wuyizhan$ nginx
wuyizhandeMacBook-Pro:admin-panel wuyizhan$ nginx -s reload
wuyizhandeMacBook-Pro:admin-panel wuyizhan$ pm2 stop nginx
[PM2][ERROR] Process nginx not found
┌──────────┬────┬──────┬─────┬─────────┬─────────┬────────┬─────┬────────┬──────────┬──────────┐
│ App name │ id │ mode │ pid │ status  │ restart │ uptime │ cpu │ mem    │ user     │ watching │
├──────────┼────┼──────┼─────┼─────────┼─────────┼────────┼─────┼────────┼──────────┼──────────┤
│ mock     │ 0  │ fork │ 0   │ errored │ 15      │ 0      │ 0%  │ 0 B    │ wuyizhan │ disabled │
└──────────┴────┴──────┴─────┴─────────┴─────────┴────────┴─────┴────────┴──────────┴──────────┘
 Use `pm2 show <id|name>` to get more details about an app
wuyizhandeMacBook-Pro:admin-panel wuyizhan$ node mock
> Mock started on http://localhost:3000
events.js:183
      throw er; // Unhandled 'error' event
      ^

Error: listen EADDRINUSE :::3000
    at Object._errnoException (util.js:992:11)
    at _exceptionWithHostPort (util.js:1014:20)
    at Server.setupListenHandle [as _listen2] (net.js:1355:14)
    at listenInCluster (net.js:1396:12)
    at Server.listen (net.js:1480:7)
    at Object.<anonymous> (/Users/wuyizhan/Desktop/admin-panel/mock.js:111:4)
    at Module._compile (module.js:652:30)
    at Object.Module._extensions..js (module.js:663:10)
    at Module.load (module.js:565:32)
    at tryModuleLoad (module.js:505:12)
wuyizhandeMacBook-Pro:admin-panel wuyizhan$
wuyizhandeMacBook-Pro:admin-panel wuyizhan$
wuyizhandeMacBook-Pro:admin-panel wuyizhan$ pm2 ls
┌──────────┬────┬──────┬─────┬─────────┬─────────┬────────┬─────┬────────┬──────────┬──────────┐
│ App name │ id │ mode │ pid │ status  │ restart │ uptime │ cpu │ mem    │ user     │ watching │
├──────────┼────┼──────┼─────┼─────────┼─────────┼────────┼─────┼────────┼──────────┼──────────┤
│ mock     │ 0  │ fork │ 0   │ errored │ 15      │ 0      │ 0%  │ 0 B    │ wuyizhan │ disabled │
└──────────┴────┴──────┴─────┴─────────┴─────────┴────────┴─────┴────────┴──────────┴──────────┘
 Use `pm2 show <id|name>` to get more details about an app
wuyizhandeMacBook-Pro:admin-panel wuyizhan$ node mock
> Mock started on http://localhost:3000
events.js:183
      throw er; // Unhandled 'error' event
      ^

Error: listen EADDRINUSE :::3000
    at Object._errnoException (util.js:992:11)
    at _exceptionWithHostPort (util.js:1014:20)
    at Server.setupListenHandle [as _listen2] (net.js:1355:14)
    at listenInCluster (net.js:1396:12)
    at Server.listen (net.js:1480:7)
    at Object.<anonymous> (/Users/wuyizhan/Desktop/admin-panel/mock.js:111:4)
    at Module._compile (module.js:652:30)
    at Object.Module._extensions..js (module.js:663:10)
    at Module.load (module.js:565:32)
    at tryModuleLoad (module.js:505:12)
wuyizhandeMacBook-Pro:admin-panel wuyizhan$ ps -aux | grep 3000
ps: No user named 'x'
wuyizhandeMacBook-Pro:admin-panel wuyizhan$
wuyizhandeMacBook-Pro:admin-panel wuyizhan$ ps -au | grep 3000
ps: option requires an argument -- u
usage: ps [-AaCcEefhjlMmrSTvwXx] [-O fmt | -o fmt] [-G gid[,gid...]]
          [-g grp[,grp...]] [-u [uid,uid...]]
          [-p pid[,pid...]] [-t tty[,tty...]] [-U user[,user...]]
       ps [-L]
wuyizhandeMacBook-Pro:admin-panel wuyizhan$ netstat -nlp | grep 3000
netstat: option requires an argument -- p
Usage:  netstat [-AaLlnW] [-f address_family | -p protocol]
        netstat [-gilns] [-f address_family]
        netstat -i | -I interface [-w wait] [-abdgRtS]
        netstat -s [-s] [-f address_family | -p protocol] [-w wait]
        netstat -i | -I interface -s [-f address_family | -p protocol]
        netstat -m [-m]
        netstat -r [-Aaln] [-f address_family]
        netstat -rs [-s]

wuyizhandeMacBook-Pro:admin-panel wuyizhan$ ps
  PID TTY           TIME CMD
 3480 ttys000    0:00.01 /bin/bash -l
 3500 ttys000    0:00.26 node mock
 3501 ttys001    0:00.01 /bin/bash -l
 9501 ttys001    0:00.28 npm
 9503 ttys001    0:19.49 node /Users/wuyizhan/Desktop/admin-panel/node_modules/.bin/webpack
 3919 ttys002    0:00.02 -bash
 9307 ttys003    0:00.04 /bin/bash -l
wuyizhandeMacBook-Pro:admin-panel wuyizhan$ pm2 ls
┌──────────┬────┬──────┬─────┬─────────┬─────────┬────────┬─────┬────────┬──────────┬──────────┐
│ App name │ id │ mode │ pid │ status  │ restart │ uptime │ cpu │ mem    │ user     │ watching │
├──────────┼────┼──────┼─────┼─────────┼─────────┼────────┼─────┼────────┼──────────┼──────────┤
│ mock     │ 0  │ fork │ 0   │ errored │ 15      │ 0      │ 0%  │ 0 B    │ wuyizhan │ disabled │
└──────────┴────┴──────┴─────┴─────────┴─────────┴────────┴─────┴────────┴──────────┴──────────┘
 Use `pm2 show <id|name>` to get more details about an app
```   
```
export BASE_URL=localhost:10000/api
```


