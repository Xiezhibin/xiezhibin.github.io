# Session的作用


## 
在没有session id 的情况之下, 不同用户超过5分钟以上的不同任务链条打上不同的session id的标记

create table if not exists event_log(
    user_code string,
    event string,
    event_id string,
    time Timestamp
)
comment '行为大宽表'
stored as parquet 


select 
sum(session_zero) over(partition by user_code order by time desc) as session,
time,
event_id,
event,
user_code 
from 
(
select 
case when(unix_timestamp(time) - unix_time(lag_time)) >= 5*60*1000 then 1
else 0
end as session_zero,
time,
event_id,
event,
user_code
from 
(select time, 
       lag(time) over(partition by user_code order by time desc) as lag_time,
       event,
       event_id,
       user_code
       from
       event_log
       where event in ('A', 'b', 'c')
) TBL1
) TBL2







