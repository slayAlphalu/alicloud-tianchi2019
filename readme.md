！！！如果代码运行失败，请联系15355719098 or 邮件 yl3963@columbia.edu!!!
----------------------------------------------------------------------------------------------------------------------------------------------------


预测采用rulebased，整体思路：

将原始数据上传到odps 
建以下四张表（建表代码在coding/rawdata.sql 下面）

Table1 ：pred_set_roundb： ECommAI_ubp_round1_b_test.tar.gz
table 2:  action_roundb：ECommAI_ubp_round1_b_train.tar.gz
table3: user_feature_roundb ： ECommAI_ubp_round1_b_user_feature.tar.gz
table4: item_feature_roundb ：ECommAI_ubp_round1_b_item_feature.tar.gz


step1:根据全部用户的行为次数累加和排序，找出全局热搜的50个itemid，按序放入表roundb_hot50中
（建表代码
--建立roundb的热搜50 items
create table roundb_hot50 as 
select itemid,count(*) as freq
from action_roundb 
group by itemid 
order by count(*)DESC limit 50 ;
select * from roundb_hot50;
）


step2:
根据userid itemid 组合统计每天的行为总数（四种行为都算），并做时间衰减，例如20190620这一天的权重为1，20190619这一天的权重为1/2,...20190610这一天的权重是1/11.
将行为次数*权重加和，降序排名，group by userid ，筛选出每个用户最常互动的50个itemid。
关联预测集，只保留在预测集中出现过的userid，itemid；
若不足50个item，则用热搜50的商品顺序填充。 
填充的函数脚本为impute.py 


Step3: 对于预测集中没有任何互动行为的userid，直接推送热搜50，最终的表数据存在final_result_alphalu_v3_b



