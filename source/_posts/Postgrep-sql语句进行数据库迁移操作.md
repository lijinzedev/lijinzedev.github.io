---
title: Postgrep sql语句进行数据库迁移操作
top: false
cover: false
toc: true
mathjax: true
categories:
  - sql
tags:
  - sql
  - 数据迁移
date: 2023-11-15 09:46:54
password:
summary:
---

# 数据新增与删除

```
insert into mine_config(mine_code, page_type)
select mcr.mine_code, 0
from mining_codes_relation mcr
         left join mine_config mc on mcr.mine_code = mc.mine_code
where mc.mine_code is null;

```

## 跨库操作

### dblink操作数据库

```sql
SELECT * FROM dblink('host=172.31.54.110 port=5432 user=postgres password=pg-dev123* dbname=mining'::text,
                     'SELECT * FROM excavation_heading_collection_info'::text)  as t(

                                                                                     id bigint,
                                                                                     work_face_id varchar(256),
                                                                                     work_face_name varchar(256),
                                                                                     mine_code varchar(256),
                                                                                     mine_name varchar(256),
                                                                                     company_code varchar(256),
                                                                                     company_name varchar(256),
                                                                                     create_date_time timestamp,
                                                                                     day_process numeric,
                                                                                     is_delete integer,
                                                                                     type_mode integer,
                                                                                     compensation numeric,
                                                                                     remark varchar(2000),
                                                                                     large_dist numeric,
                                                                                     version integer,
                                                                                     update_time timestamp,
                                                                                     position_label varchar(255),
                                                                                     target varchar(255),
                                                                                     working_mode varchar(4),
                                                                                     new_remark jsonb,
                                                                                     is_deal integer,
                                                                                     has_station_change integer,
                                                                                     adjust_value numeric,
                                                                                     adjust_value_filter numeric) where t.work_face_id = '1665894702413606913' order by id desc ;

```

### 同步示例 

####  掘进面

```sql
delete
from excavation_heading_collection_info where mine_code = '371725003374' ;
select from excavation_heading_collection_info where mine_code = '371725003374';
WITH old AS (SELECT *
             FROM dblink('host=172.31.54.110 port=5432 user=postgres password=pg-dev123* dbname=mining'::text,
                         'SELECT * FROM excavation_heading_collection_info'::text) as t(
                                                                                        id bigint,
                                                                                        work_face_id varchar(256),
                                                                                        work_face_name varchar(256),
                                                                                        mine_code varchar(256),
                                                                                        mine_name varchar(256),
                                                                                        company_code varchar(256),
                                                                                        company_name varchar(256),
                                                                                        create_date_time timestamp,
                                                                                        day_process numeric,
                                                                                        is_delete integer,
                                                                                        type_mode integer,
                                                                                        compensation numeric,
                                                                                        remark varchar(2000),
                                                                                        large_dist numeric,
                                                                                        version integer,
                                                                                        update_time timestamp,
                                                                                        position_label varchar(255),
                                                                                        target varchar(255),
                                                                                        working_mode varchar(4),
                                                                                        new_remark jsonb,
                                                                                        is_deal integer,
                                                                                        has_station_change integer,
                                                                                        adjust_value numeric,
                                                                                        adjust_value_filter numeric))

insert into excavation_heading_collection_info( work_face_id, work_face_name, mine_code, mine_name, company_code,
                                               company_name, create_date_time, day_process, is_delete, type_mode,
                                               compensation, remark, large_dist, version, update_time, position_label,
                                               target, working_mode, new_remark, is_deal, has_station_change,
                                               adjust_value, adjust_value_filter)

SELECT work_face_id, work_face_name, mine_code, mine_name, company_code,
       company_name, create_date_time, day_process, is_delete, type_mode,
       compensation, remark, large_dist, version, update_time, position_label,
       target, working_mode, new_remark, is_deal, has_station_change,
       adjust_value, adjust_value_filter from old  where  mine_code = '371725003374';



select count(*) from excavation_heading_collection_info where  mine_code = '371725003374' ;

```

#### 回采面

```sql


delete
from excavation_mining_collection_info where mine_code = '371725003374' ;
select from excavation_mining_collection_info where mine_code = '371725003374';
WITH old AS (SELECT *
             FROM dblink('host=172.31.54.110 port=5432 user=postgres password=pg-dev123* dbname=mining'::text,
                         'SELECT * FROM excavation_mining_collection_info'::text) as t(
                                                                                        id bigint,
                                                                                        work_face_id varchar(256),
                                                                                        work_face_name varchar(256),
                                                                                        mine_code varchar(256),
                                                                                        mine_name varchar(256),
                                                                                        company_code varchar(256),
                                                                                        company_name varchar(256),
                                                                                        create_date_time timestamp,
                                                                                        day_process numeric,
                                                                                        is_delete integer,
                                                                                        type_mode integer,
                                                                                        compensation numeric,
                                                                                        remark varchar(2000),
                                                                                        large_dist numeric,
                                                                                        version integer,
                                                                                        update_time timestamp,
                                                                                        position_label varchar(255),
                                                                                        target varchar(255),
                                                                                        working_mode varchar(4),
                                                                                        new_remark jsonb,
                                                                                        is_deal integer,
                                                                                        has_station_change integer,
                                                                                        adjust_value numeric,
                                                                                        adjust_value_filter numeric))

insert into excavation_mining_collection_info( work_face_id, work_face_name, mine_code, mine_name, company_code,
                                               company_name, create_date_time, day_process, is_delete, type_mode,
                                               compensation, remark, large_dist, version, update_time, position_label,
                                               target, working_mode, new_remark, is_deal, has_station_change,
                                               adjust_value, adjust_value_filter)

SELECT work_face_id, work_face_name, mine_code, mine_name, company_code,
       company_name, create_date_time, day_process, is_delete, type_mode,
       compensation, remark, large_dist, version, update_time, position_label,
       target, working_mode, new_remark, is_deal, has_station_change,
       adjust_value, adjust_value_filter from old  where  mine_code = '371725003374';



select count(*) from excavation_mining_collection_info where  mine_code = '371725003374' ;

```

#### workcal表

```sql
delete
from work_face_cal
where mine_code = '371725003374';
select *
from work_face_cal
where mine_code = '371725003374';



WITH old AS (SELECT *
             FROM dblink('host=172.31.54.110 port=5432 user=postgres password=pg-dev123* dbname=mining',
                         'SELECT * FROM work_face_cal') as t(
                                                             id bigint,
                                                             work_face_id varchar(255),
                                                             compensation numeric,
                                                             end_time_footage_base timestamp,
                                                             work_face_type varchar(30),
                                                             target_coordinates jsonb,
                                                             central_coordinate jsonb,
                                                             work_model varchar(2),
                                                             cumulative_length_total numeric,
                                                             base_station_coordinates jsonb,
                                                             air_coordinates jsonb,
                                                             trans_coordinates jsonb,
                                                             ranging_distance numeric,
                                                             mine_code varchar(255),
                                                             mine_name varchar(255),
                                                             company_name varchar(255),
                                                             work_face_name varchar(255),
                                                             company_code varchar(255),
                                                             bind_enum varchar(25),
                                                             target_coordinates_origin jsonb,
                                                             base_station_coordinates_origin jsonb,
                                                             exact_distance numeric,
                                                             excavated_length numeric,
                                                             update_time timestamp,
                                                             footage_start_date timestamp,
                                                             footage_start_coordinate jsonb,
                                                             footage_excavated_length numeric,
                                                             footage_start_base_station jsonb,
                                                             footage_start_dist numeric,
                                                             full_cumulative_length_total numeric
                 ))
insert
into work_face_cal(work_face_id, compensation, end_time_footage_base, work_face_type, target_coordinates,
                   central_coordinate, work_model, cumulative_length_total, base_station_coordinates, air_coordinates,
                   trans_coordinates, ranging_distance, mine_code, mine_name, company_name, work_face_name,
                   company_code, bind_enum, target_coordinates_origin, base_station_coordinates_origin, exact_distance,
                   excavated_length, update_time, footage_start_date, footage_start_coordinate,
                   footage_excavated_length, footage_start_base_station, footage_start_dist,
                   full_cumulative_length_total)
SELECT work_face_id,
       compensation,
       end_time_footage_base,
       work_face_type,
       target_coordinates,
       central_coordinate,
       work_model,
       cumulative_length_total,
       base_station_coordinates,
       air_coordinates,
       trans_coordinates,
       ranging_distance,
       mine_code,
       mine_name,
       company_name,
       work_face_name,
       company_code,
       bind_enum,
       target_coordinates_origin,
       base_station_coordinates_origin,
       exact_distance,
       excavated_length,
       update_time,
       footage_start_date,
       footage_start_coordinate,
       footage_excavated_length,
       footage_start_base_station,
       footage_start_dist,
       full_cumulative_length_total
from old
where mine_code = '371725003374'



```



# 如何修改序列？

### 步骤1：查找表中的最大ID

首先，我们需要确定表中ID列的最大值。可以使用下面的SQL查询来获取这个值：

```sql
SELECT MAX(ID) FROM 表名;
```

### 步骤2：修改序列的当前值

一旦得到了最大ID值，我们可以使用下面的ALTER SEQUENCE语句来修改序列的当前值：

```sql
ALTER SEQUENCE 序列名 RESTART WITH 最大ID值;
```

