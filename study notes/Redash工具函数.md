# Redash基于CK的工具函数

## 一、字段提取

- `visitParamExtractString(all_ext,'d_reason')`

## 二、字符串操作

- 查找子串位置：`position(url,'/grocery/')`

- 字符串截取：`substring(url, position(url,'/grocery/')+9)`、`substring(visitParamExtractString(all_ext,'i_retcode'),1,10)`

  **注意：索引是从1开始的**

- 正则提取：`extract(visitParamExtractString(all_ext, 'ua'), 'Chrome/([\\d\\.]+)') AS chrome_version`

  ```sql
  SELECT extract(visitParamExtractString(all_ext, 'ua'), 'Chrome/([\\d\\.]+)') AS chrome_version,
         count(visitParamExtractString(all_ext, 'u_caller')) AS total,
         count(if(visitParamExtractString(all_ext, 'd_result') = '1',visitParamExtractString(all_ext, 'u_caller'),NULL)) AS suc,
         suc / total * 100 AS ratio
  FROM wireless_music.ods_d_kuyin_h5_pageoplog_real
  WHERE procdate >= today() - INTERVAL 30 DAY
    AND procdate < today()
    AND opcode = 'HT03005'
    AND visitParamExtractString(all_ext, 'u_isptype') = '1'
    AND visitParamExtractString(all_ext, 'd_ordertype') = 'webjs'
  GROUP BY chrome_version
  HAVING total > 300
  ORDER BY ratio DESC
  ```

- 字符串分割：`splitByChar('?',url)[1]`

  - 这里要注意分割后得到的数组，索引是从1开始的
  
- 字符串替换：`replaceRegexpAll(visitParamExtractString(all_ext, 'u_opname'),'移动|联通|电信', '')`


## 三、日期操作

- 日期（精确到时间）：`toDateTime(substring(ctm, 1, 19))`
- 日期（精确到天）：`toDate(substring(ctm, 1, 19))`
- 获取年度：`toYear(procdate)`
- 精确到月：`toYYYYMM(procdate)`
- 日期格式化输出：`formatDateTime(procdate,'%Y- %m-%d')`

  ```sql
  SELECT toYear(procdate) AS YEAR,
         formatDateTime(procdate, '%m-%d') AS MonthDay,
         count(DISTINCT if(opcode='HT03039' --  AND (visitParamExtractString(all_ext,'i_strategy_diyon')='3'
   --       OR visitParamExtractString(all_ext,'i_strategy_diyon')='8')
  
                           AND visitParamExtractString(all_ext,'u_isptype')='3', visitParamExtractString(all_ext,'u_caller'), NULL)) AS entwebjs,
         count(DISTINCT if(opcode='HT03040' --  AND (visitParamExtractString(all_ext,'i_strategy_diyon')='3'
   --       OR visitParamExtractString(all_ext,'i_strategy_diyon')='8')
  
                           AND visitParamExtractString(all_ext,'u_isptype')='3', visitParamExtractString(all_ext,'u_caller'), NULL)) AS loadwebjs,
         count(DISTINCT if(opcode='HT03005'
                           AND visitParamExtractString(all_ext,'u_isptype')='3'
                           AND visitParamExtractString(all_ext,'d_result')='1'
                           AND visitParamExtractString(all_ext,'d_ordertype')='webjs' , visitParamExtractString(all_ext,'u_caller'), NULL)) AS ordersuc,
         ordersuc/entwebjs*100 AS ordersucratio1,
         ordersuc/loadwebjs*100 AS ordersucratio2
  FROM wireless_music.ods_d_kuyin_h5_pageoplog
  WHERE (procdate >= '{{beginDate}}'
         AND procdate <= '{{endDate}}'
         OR procdate >= toDate('{{beginDate}}') - INTERVAL 1 YEAR
         AND procdate <= toDate('{{endDate}}') - INTERVAL 1 YEAR)
    AND visitParamExtractString(all_ext,'i_channelno') = '{{channelNo}}'
    AND opcode IN ('HT00004',
                   'HT03039',
                   'HT03040',
                   'HT03005')
  GROUP BY procdate
  ORDER BY MonthDay,
           YEAR
  ```

- 小时：`toHour(toDateTime(substring(ctm, 1, 19)))`
- 小时：`toHour(parseDateTimeBestEffort(ctm))`
- 分钟：`toMinute(toDateTime(substring(ctm, 1, 19)))`
- 当天：`today()`
- 昨天：`yesterday()`
- 前天：`yesterday() - toIntervalDay(1)`
- 七天前：`today() - INTERVAL 7 DAY`
- 当前时间：`now()`
- 十分钟前：`now() - INTERVAL 10 MINUTE`
- 一小时前：`now() - INTERVAL 1 HOUR`
- 以十分钟作为间隔：`toStartOfInterval(parseDateTimeBestEffort(ctm), INTERVAL 10 MINUTE) AS interval`

  ```sql
  SELECT toStartOfInterval(parseDateTimeBestEffort(ctm), INTERVAL 10 MINUTE) AS interval,
         count(if(visitParamExtractString(all_ext, 'i_retcode')='000000',1, NULL)) AS sucpv,
         count(1) AS totalpv,
         sucpv/totalpv*100 AS ratio
  FROM wireless_music.ods_d_kuyin_h5_pageoplog_real
  WHERE procdate >= '{{开始日期}}'
    AND opcode='FT02001'
    AND visitParamExtractString(all_ext, 'i_opdesc')='openMonthRingCallback'
  GROUP BY interval
  ORDER BY interval DESC
  
  -- 如果想展示的是到某个时间点结束时的统计数据，则如下
  SELECT toStartOfInterval(parseDateTimeBestEffort(ctm) + INTERVAL 5 MINUTE, INTERVAL 5 MINUTE) AS interval,
         count(1) AS pv
  FROM wireless_music.ods_d_kuyin_h5_pageoplog_real
  WHERE procdate = today()
    AND opcode='HT00003'
    AND visitParamExtractString(all_ext, 'i_business_type')='ring'
  GROUP BY interval
  ORDER BY interval DESC
  ```

- 获取两个时间的时差（以天为单位）：`datediff('day', toDate('2025-02-18'), procdate) + 1`

## 四、格式转换

- `toInt32(visitParamExtractString(all_ext,'i_c_cost'))`
- `toInt32OrZero(visitParamExtractString(all_ext,'i_cost'))`
- 取第一个非空值：`coalesce(value1, value2, ..., valueN)`
- 四舍五入：`round(number, decimal_places)`
  - decimal_places为-1时可实现精确到十位的效果
- 空判断：`NULLIF(t.total_uv, 0)`

## 五、数字

- `sum`求和

  ```sql
  SELECT procdate,
         visitParamExtractString(all_ext,'d_pageorigin') AS pageorigin,
         count(DISTINCT uid) AS users,
         SUM(COUNT(DISTINCT uid)) OVER (PARTITION BY procdate) AS total
  FROM wireless_music.ods_d_kuyin_h5_pageoplog
  WHERE procdate >= '{{beginDate}}'
    AND procdate <= '{{endDate}}'
    AND visitParamExtractString(all_ext,'i_channelno') = '{{channelNo}}'
    AND opcode = 'HT01000'
  GROUP BY procdate
  ```

  

## 六、句式

- case语句

  ```sql                 WHEN coalesce(A.num, 0) = 0 THEN -100.0
  ROUND(CASE
  	WHEN coalesce(A.num, 0) = 0 THEN -100.0
  	WHEN coalesce(B.num, 0) = 0 THEN 100.0
  	ELSE (coalesce(A.num, 0) - coalesce(B.num, 0)) / coalesce(B.num, 0) * 100
  END, 2) AS percent
  ```
  
- coalesce

  ```sql
  SELECT COALESCE(NULLIF(visitParamExtractString(all_ext, 'd_word'), ''), NULLIF(visitParamExtractString(all_ext, 'd_song'), ''), NULLIF(visitParamExtractString(all_ext, 'd_singer'), '')) AS word,
         count(1) AS num
  FROM wireless_music.ods_d_kuyin_h5_pageoplog
  WHERE procdate >= '{{开始时间}}'
    AND procdate <= '{{结束时间}}'
    AND opcode='SE0000001'
    AND word != ''
  GROUP BY word
  ORDER BY num DESC
  LIMIT 1000
  ```

- sum

  ```sql
  SELECT procdate,
         visitParamExtractString(all_ext,'i_channelno') AS channel,
         count(DISTINCT uid) AS uv,
         sum(count(DISTINCT uid)) OVER (PARTITION BY procdate) AS totaluv
  FROM wireless_music.ods_d_kuyin_h5_pageoplog
  WHERE procdate >= today() - INTERVAL 15 DAY
    AND opcode = 'HT00004'
    AND (visitParamExtractString(all_ext,'i_business_type') = 'ring'
         OR visitParamExtractString(all_ext,'i_business_type') = 'vring')
  GROUP BY procdate,
           channel
  ORDER BY procdate DESC,
           uv DESC
  ```
  
- avg

  ```sql
  SELECT toStartOfInterval(parseDateTimeBestEffort(ctm), INTERVAL 1 HOUR) AS interval,
         visitParamExtractString(all_ext, 'i_channelno') AS channel,
         count(DISTINCT if(opcode = 'HT00004', uid, NULL)) AS uv,
         count(if(opcode = 'HT03005'
                  AND visitParamExtractString(all_ext, 'd_result') = '1', visitParamExtractString(all_ext, 'u_caller'), NULL)) AS ordersuc,
         ordersuc / uv * 100 AS ordersucratio,
         AVG(ordersucratio) OVER () AS avg_ordersucratio_overall
  FROM wireless_music.ods_d_kuyin_h5_pageoplog_real
  WHERE interval >= '2025-05-22 20:00:00'
    AND interval <= '2025-05-23 09:00:00'
    AND opcode IN ('HT00004',
                   'HT03005')
    AND visitParamExtractString(all_ext, 'i_channelno') = '{{渠道号}}'
  GROUP BY interval, channel
  ORDER BY interval
  ```

- nullIf用法

  ```sql
  SELECT coalesce(nullIf(extract(visitParamExtractString(all_ext, 'ua'), '(Android [\d\.]+)'),''),nullIf(extract(visitParamExtractString(all_ext, 'ua'), '(iPhone OS \d+)'),''),'unknown') AS SYSTEM,
         count(DISTINCT uid) AS num
  FROM wireless_music.ods_d_kuyin_h5_pageoplog
  WHERE procdate >= '2025-01-01'
    AND procdate <= today()
    AND opcode ='HT03005'
    AND visitParamExtractString(all_ext,'u_isptype') = '1'
    AND visitParamExtractString(all_ext,'d_result') = '0'
    AND visitParamExtractString(all_ext,'i_retcode') = '9997'
    AND visitParamExtractString(all_ext,'i_retdesc') = 'webjs加载超时'
    AND (visitParamExtractString(all_ext,'i_strategy_diyon')='3'
         OR visitParamExtractString(all_ext,'i_strategy_diyon')='8')
  GROUP BY SYSTEM
  ```
  
  
