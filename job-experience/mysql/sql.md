几类常用SQL
===
### UPSERT(累积)
```sql
	insert into t1 
	select * from
	tmp_day_exercise_detail as main
	on duplicate key update  
		t1.total= t1.total+ main.total,
	    t1.correct=t1.correct+main.total,
	    t1.updated_at=main.updated_at,
	    t1.frequency=t1.frequency+1;
```

### 分组排序
```sql
	SELECT
	    *,
	    @rank:=CASE
			when  @pre_group != main.clazz_id THEN 1 
	        WHEN
	            @pre_group = main.clazz_id
	                AND @pre_value != main.total
	        THEN
	            @rank + 1
	        ELSE @rank
	    END AS rank,
	    @pre_group:=main.clazz_id pre_group,
	    @pre_value:=main.total as pre_value
	FROM
	    (
		SELECT 
	        dcr.origin_clazz_id clazz_id, ds.passport_id
	    FROM
	        stcr sser
	    LEFT JOIN dim_students ds ON sser.passport_id = ds.passport_id
	    LEFT JOIN dim_clazz_relation dcr ON dcr.origin_ts_id = ds.origin_id
	    WHERE
	        sser.calc_date = end_date
	            AND sser.`interval` = date_interval
	            AND sser.`type` = "student_overview"
				AND dcr.dim_type = 'student'
	            AND dcr.quit_date > @end_datetime_srt
	            AND dcr.enroll_date <= @end_datetime_srt
	    GROUP BY dcr.origin_clazz_id , ds.passport_id
	    ORDER BY clazz_id DESC, sser.total DESC) AS main,
	    (SELECT @pre_group:=-1) AS pg,
	    (SELECT @pre_value:=-1) AS pv,
	    (SELECT @rank:=1) AS r;
	END
```

### 关联更新
```sql
	update stcr as a 
	inner join  stcr as b on a.passport_id=b.passport_id 
	set a.total_trend=a.total-b.total,a.total_trend_rate=(a.total-b.total)/b.total,a.correct_rate_trend=a.correct_rate-b.correct_rate
	where
	   a.calc_date=end_date and a.`type`='student_overview' and 
	   b.calc_date=`get_previous_date`(end_date,7) and b.`type`='student_overview';
```

