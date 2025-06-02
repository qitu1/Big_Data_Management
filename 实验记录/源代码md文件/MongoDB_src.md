
### MongoDB实验
#### a) . 条件查询与执行计划:
1. 
```json
db.business.find().skip(5).limit(5)
```
2. 
```json
db.business.find({ state: "CA", "attributes.BikeParking": "True" }).limit(5)
```
3. 
```json
 db.review.find(
    { useful: { $gt: 500 } }, 
    { business_id: 1, user_id: 1, useful: 1, _id: 0 }
	).limit(10)
```
4. 
```json
db.user.find({ useful: { $in: [10, 20, 30, 40, 50, 60, 70, 80, 90, 100] } }, { name: 1, useful: 1 }).limit(20);
```
5. 
```json
db.user.find({ fans: { $gte: 100, $lt: 200 }, useful: { $gte: 1000 } }, { name: 1, fans: 1, useful: 1 }).limit(10);
```
6. 
```json
//查询执行计划:
db.business.find().explain("executionStats");  

//统计数据：
db.business.count();
```
7. 
```json
db.business.find({ city: { $in: ["Westlake", "Las Vegas"] } }).limit(5)
```
8. 
```json
db.business.find({ categories: { $size: 5 } }, { name: 1, categories: 1, stars: 1 }).limit(10)
```
9. 
```json
//执行计划
db.business.find({ business_id: "5JucpCfHZltJh5r1JabjDg" }).explain("executionStats")

//添加索引优化
db.business.createIndex({ business_id: 1 })

//优化后的执行计划
db.business.find({business_id: "5JucpCfHZltJh5r1JabjDg" }).explain("executionStats");
```
#### b) . 聚合与索引:
10. 
```json
db.business.aggregate([
    { $group: { _id: "$state", count: { $sum: 1 } } },
    { $sort: { count: -1 } }
])
```
11.  
```json
//创建子集合
db.review.aggregate([
    { $limit: 500000 },  // 取前五十万条数据
    { $out: "Subreview" }  // 输出到新集合 Subreview
])

//创建索引：
db.Subreview.createIndex({ text: "text" })
db.Subreview.createIndex({ useful: 1 })

//查询内容包含关键词并排序：
db.Subreview.find(
    { 
        $text: { $search: "delicious" },  // 全文搜索关键词
        useful: { $gte: 50 }  // useful 字段大于等于 50
    },
    { review_id: 1, text: 1, useful: 1 }  // 返回指定字段
).sort({ review_id: 1 }) .limit(5);  // 限制返回 5 条
```
12.  
```json
 db.Subreview.aggregate([
    { $match: { useful: { $gt: 50 } } },
    { $group: { _id: "$business_id",       // 按 business_id 分组
            	avg_stars: { $avg: "$stars" }  }},
    {  $sort: { _id: 1 } },
{  $limit: 20  }
])
```
13. 
```json
//为 location 字段创建 2dsphere 索引：
db.business.createIndex({ loc: "2dsphere" });

//获取目标商家的地理坐标：
db.business.find(
  { business_id: "smkZUv_IeYYj_BA6-Po7oQ" },
  { "loc.coordinates": 1, _id: 0 }
)

db.business.aggregate([
  { $geoNear: { near: { type: "Point", coordinates: [-79.5617757296, 43.7675451093] }, distanceField: "dist.calculated", maxDistance: 2000, spherical: true, key: "loc" } },
  { $sort: { stars: -1 } },
  { $project: { name: 1, address: 1, stars: 1, _id: 0 } },
  { $limit: 20 }
])
```
14. 
```json
//创建索引：
db.Subreview.createIndex({ user_id: 1, date: 1 })

//查询代码：
db.Subreview.aggregate([
{$match:{'date':{$gte: "2000-01-01"}}}, 
{$group:{_id:"$user_id", total:{$sum:1}}}, 
{$sort: {total: -1}}, 
{$limit: 20}
])
```
