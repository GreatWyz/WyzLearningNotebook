# WyzLearningNotebook
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

