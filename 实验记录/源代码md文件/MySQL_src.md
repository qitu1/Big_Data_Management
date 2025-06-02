### MySQL for JSON 实验
#### a) . JSON基本查询:
1. 
```sql
select * from business 
where json_extract(business_info, '$.state') = 'CA' 
order by json_extract(business_info, '$.stars') desc 
limit 5;
```
2. 
```sql
select json_keys(business_info->'$.attributes') as attribute_keys,
       json_length(business_info->'$.attributes') as key_count
from business
where json_extract(business_info, '$.city') = 'Edmonton'
limit 5;
```
3. 
```sql
select 
    json_unquote(business_info->'$.name') as name,
    json_type(business_info->'$.name') as name_type,
    json_unquote(business_info->'$.stars') as stars,
    json_type(business_info->'$.stars') as stars_type,
    json_unquote(business_info->'$.hours') as hours,
    json_type(business_info->'$.hours') as hours_type,
    json_unquote(business_info->'$.latitude') as latitude,
    json_type(business_info->'$.latitude') as latitude_type
from business
limit 5;
```
4. 
```sql
select 
    json_unquote(business_info->'$.name') as name,
    json_unquote(business_info->'$.city') as city,
    business_info->'$.stars' as stars,
    json_unquote(business_info->'$.attributes.WiFi') as wifi
from 
    business
where 
    json_unquote(business_info->'$.state') = 'FL'
    and json_unquote(business_info->'$.attributes.HasTV') = 'True'
order by 
    stars desc,
    name asc
limit 20;
```
5. 查看执行计划：
```sql
explain format=json
select * 
from user 
where user_info->'$.cool' > 200;
```
实际执行一次查询：
```sql
select * 
from user 
where user_info->'$.cool' > 200;
```
MongoDB查询:
```json
db.user.find({ "compliment_cool": { $gt: 200 } });
```
执行计划：
```json
db.user.find({ "compliment_cool": { $gt: 200 } }).explain("executionStats");
```
MySQL中创建虚拟索引:
```sql
ALTER TABLE user
ADD COLUMN cool_value INT AS (JSON_UNQUOTE(user_info->'$.cool')) STORED;
```
```sql
CREATE INDEX idx_cool_value ON user (cool_value);
```
优化后的查询
```sql
SELECT *
FROM user
WHERE cool_value > 200;
```
优化后的查询计划
```sql
EXPLAIN FORMAT=JSON
SELECT *
FROM user
WHERE cool_value > 200;
```
#### b) . JSON增删改:
6. 查询语句
```sql
//describe user;
//describe business;
SELECT JSON_PRETTY(business_info) AS original_info
FROM business
WHERE business_id= '--eBbs3HpZYIym5pEw8Qdw';
```
新增键值对
```sql
UPDATE business
SET business_info = JSON_SET(
    business_info,
    '$.attributes.BikeParking', 'True',  -- 新增 "BikeParking":"True"
    '$.review_count', 42,                -- 修改评论数量为 42
    '$.attributes.WiFi', 'Paid'          -- 修改 "WiFi" 为 "Paid"
)
WHERE business_id = '--eBbs3HpZYIym5pEw8Qdw';
```
修改后查询语句
```sql
SELECT JSON_PRETTY(business_info) AS updated_info
FROM business
WHERE business_id = '--eBbs3HpZYIym5pEw8Qdw';
```
7. 
```sql
//找到 id 为 --agAy0vRYwG6WqbInorfg 的记录，并将其插入为id='change':
INSERT INTO user (user_id, user_info)
SELECT 'change', user_info
FROM user
WHERE user_id = '--agAy0vRYwG6WqbInorfg';

//使用 JSON_REMOVE 删除 JSON 中的 fans 和 useful 键:
UPDATE user
SET user_info = JSON_REMOVE(user_info, '$.fans', '$.useful')
WHERE user_id = 'change';

//使用 JSON_SET 添加新的键值对 city: 'New York'：
UPDATE user
SET user_info = JSON_SET(user_info, '$.city', 'New York')
WHERE user_id = 'change';

//查询 user_id = 'change' 的完整信息：
SELECT user_id, JSON_PRETTY(user_info) AS user_info 
FROM user 
WHERE user_id = 'change';
```
#### c) . JSON聚合:
8. 
```sql
SELECT 
    state, 
    JSON_OBJECTAGG(city, count) AS city_occ_num
FROM (
    SELECT 
        JSON_UNQUOTE(JSON_EXTRACT(business_info, '$.state')) AS state,
        JSON_UNQUOTE(JSON_EXTRACT(business_info, '$.city')) AS city,
        COUNT(*) AS count
    FROM business
    GROUP BY JSON_UNQUOTE(JSON_EXTRACT(business_info, '$.state')),
             JSON_UNQUOTE(JSON_EXTRACT(business_info, '$.city'))
) AS sub
GROUP BY state
ORDER BY state;
```
9. 
```sql
SELECT 
    user_id, 
    JSON_ARRAYAGG(
        JSON_OBJECT(
            'business_id', business_id,
            'tip_info', tip_info
        )
    ) AS tips_array
FROM tip
GROUP BY user_id
LIMIT 5;
```
#### d) . JSON实用函数的使用:
10. 
```sql
SELECT 
    JSON_UNQUOTE(business_info->'$.name') AS name,
    JSON_UNQUOTE(business_info->'$.attributes.WiFi') AS WiFi,
    JSON_UNQUOTE(business_info->'$.attributes.DogsAllowed')AS DogsAllowed,
    JSON_UNQUOTE(business_info->'$.attributes.HasTV') AS HasTV
FROM 
    business
WHERE 
    JSON_UNQUOTE(business_info->'$.city') = 'Edmonton'
    AND (
        JSON_UNQUOTE(business_info->'$.attributes.WiFi') LIKE "u'no'"
        OR JSON_UNQUOTE(business_info->'$.attributes.DogsAllowed') = 'True'
        OR JSON_UNQUOTE(business_info->'$.attributes.HasTV') = 'False'
    )
ORDER BY 
    name ASC
LIMIT 10;
```
11. 
```sql
SELECT 
    JSON_UNQUOTE(JSON_EXTRACT(user_info, '$.name')) AS name,
    JSON_EXTRACT(user_info, '$.funny') AS funny,
    JSON_EXTRACT(user_info, '$.cool') AS cool,
    JSON_EXTRACT(user_info, '$.useful') AS useful,
    JSON_ARRAY(
        JSON_EXTRACT(user_info, '$.funny'),
        JSON_EXTRACT(user_info, '$.useful'),
        JSON_EXTRACT(user_info, '$.cool')
    ) AS funny_useful_cool,
    (JSON_EXTRACT(user_info, '$.funny') + JSON_EXTRACT(user_info, '$.useful') + JSON_EXTRACT(user_info, '$.cool')) AS total
FROM 
    user
WHERE 
    JSON_EXTRACT(user_info, '$.useful') > 1000
LIMIT 10;
```
12. 
```sql
SELECT 
    JSON_PRETTY(
        JSON_MERGE_PRESERVE(
            (SELECT business_info FROM business WHERE business_id = '-1b2kNOowsPrPpBOK4lNkQ' LIMIT 1),
            (SELECT user_info FROM user WHERE user_id = '--7XOV5T9yZR5w1DIy_Dog' LIMIT 1)
        )
) AS merged_info;
```
13. 
```sql
SELECT 
    b.name,  
    b.has_tv,  
    a.attribute_key,  -- 选择每个商户属性的键
    a.attribute_value,  -- 选择每个商户属性的值
    ROW_NUMBER() OVER (PARTITION BY b.name ORDER BY a.attribute_key) AS attribute_index  -- 为每个商户的属性生成递增编号
FROM 
    (
        SELECT 
            business_id,  -- 选择商户ID
            JSON_UNQUOTE(JSON_EXTRACT(business_info, '$.name')) AS name,  -- 提取商户的名称并去掉引号
            JSON_UNQUOTE(JSON_EXTRACT(business_info, '$.attributes.HasTV')) AS has_tv,  -- 提取商户是否有电视的属性并去掉引号
            JSON_EXTRACT(business_info, '$.attributes') AS attributes  -- 提取商户的所有属性（attributes）
        FROM 
            business  -- 从 business 表中选择数据
        ORDER BY 
            CAST(JSON_EXTRACT(business_info, '$.review_count') AS UNSIGNED) DESC  -- 按商户的评论数降序排列
        LIMIT 
            3  -- 限制查询返回前三个商户
    ) AS b,  -- 这里将内嵌的查询结果别名为 b（即前 3 个商户）
    JSON_TABLE(  -- 使用 JSON_TABLE 函数展开商户的属性数据（attributes）
        b.attributes,  -- 传入刚才提取的商户属性
        '$.*' COLUMNS (  -- 遍历 attributes 中的每个字段（即属性）
            attribute_key VARCHAR(255) PATH '$.key',  -- 将属性的键提取为 attribute_key 列
            attribute_value VARCHAR(255) PATH '$.value'  -- 将属性的值提取为 attribute_value 列
        )
    ) AS a  -- 这个 JSON_TABLE 操作会将 JSON 对象展开为多行，并赋予别名 a
ORDER BY 
    b.name;  -- 最后按商户名称升序排序
```


