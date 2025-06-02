### Neo4j 实验
1
```cypher
MATCH (n:UserNode)
RETURN n.name AS name, n.userid AS userid
LIMIT 10;
```
2
```cypher
MATCH (business:BusinessNode {city: 'Ambridge'})
RETURN business
LIMIT 5;
```
3
```cypher
MATCH (u:UserNode)-[:Review]->(r:ReviewNode {reviewid: 'T_8OnmZbyRhnXutFWGqaRg'})
RETURN u;
```
4
```cypher
MATCH (u:UserNode {userid: "d9GraD1OjVyTEd1zPjp7Yg"})-[:Review]->(r:ReviewNode)-[:Reviewed]->(b:BusinessNode)
RETURN b.name AS business_name, b.stars AS business_stars, r.stars AS review_stars
LIMIT 20
```
5
```cypher
MATCH (u:UserNode {userid: "d9GraD1OjVyTEd1zPjp7Yg"})-[:Review]->(r:ReviewNode {stars: '5.0'})-[:Reviewed]->(b:BusinessNode)
RETURN b.name AS business_name, b.address AS business_address, b.stars AS business_stars
```
6
```cypher
MATCH (u:UserNode {userid: "AWCY8laHjH0-3HMT0LGpUA"})-[:Review]->(r:ReviewNode)-[:Reviewed]->(b:BusinessNode)
RETURN b.name AS business_name, r.stars AS review_stars
ORDER BY r.stars DESC
```
7
```cypher
MATCH (u:UserNode)
WHERE toInteger(u.fans) > 200
RETURN u.name AS name, toInteger(u.fans) AS fans
LIMIT 20
```
8
```cypher
MATCH (b:BusinessNode {businessid: "tyjquHslrAuF5EUejbPfrw"})-[:IN_CATEGORY]->(c:CategoryNode)
RETURN count(c) AS category_count

//查看执行计划
PROFILE
MATCH (b:BusinessNode {businessid: "tyjquHslrAuF5EUejbPfrw"})-[:IN_CATEGORY]->(c:CategoryNode)
RETURN count(c) AS category_count
```
9
```cypher
MATCH (b:BusinessNode {businessid: "KWywu2tTEPWmR9JnBc0WyQ"})-[:IN_CATEGORY]->(c:CategoryNode)
RETURN b.name AS business_name, collect(c.category) AS category_list
```
10
```cypher
MATCH (u:UserNode {userid: 'd7D4dYzF6THtOx9imf-wPw'})-[HasFreind]-(f:UserNode)
WITH f
MATCH (f)-[HasFreind]-(f2:UserNode)
WITH f, COUNT(DISTINCT f2) AS numberOfFoFs
RETURN f.name, numberOfFoFs
LIMIT 20
```
11
```cypher
MATCH (b:BusinessNode)-[:IN_CATEGORY]->(c:CategoryNode)
WHERE c.category = 'Salad'
WITH b.city AS cityName, COUNT(*) AS BusinessCount
RETURN cityName, BusinessCount
ORDER BY BusinessCount DESC
LIMIT 5
```
12
```cypher
MATCH (b:BusinessNode)
WITH b.name AS business_name, count(b) AS name_count
RETURN business_name, name_count
ORDER BY name_count DESC
LIMIT 10
```
13
```cypher
MATCH (u:UserNode)-[:Review]->(r:ReviewNode)-[:Reviewed]->(b:BusinessNode {businessid: "nh_kQ16QAoXWwqZ05MPfBQ"})
WITH u, (toInteger(u.useful) + toInteger(u.funny) + toInteger(u.cool)) AS total_score
RETURN u.name AS user_name, total_score
ORDER BY total_score DESC
```
14
```cypher
MATCH (b:BusinessNode)-[:IN_CATEGORY]->(c:CategoryNode {category: 'Propane'})
MATCH (b)<-[:Reviewed]-(r:ReviewNode)
WHERE r.stars = '5.0'
RETURN DISTINCT b.name AS businessName, b.city AS businessCity, b.address AS businessAddress;
```
15
```cypher
MATCH (u:UserNode)-[:Review]->(r:ReviewNode)-[:Reviewed]->(b:BusinessNode)
    // 查找用户、评论和商户之间的关系
WITH u, count(DISTINCT b) AS unique_business_count
    // 统计每个用户评论过的不同商户数，并命名为 unique_business_count
RETURN u.name AS user_name, toInteger(u.fans) AS fans, toInteger(u.useful) AS useful, unique_business_count
    // 返回用户的名称、粉丝数、"有用"评价数，以及评论过的不同商户数
ORDER BY unique_business_count DESC
    // 按照评论过的不同商户数降序排序，最多商户数的用户排前
LIMIT 20
// 返回前20个用户
```
18  Neo4j：
```cypher
MATCH (r:ReviewNode {reviewid: "TIYgnDzezfeEnVeu9jHeEw"})-[:Reviewed]->(b:BusinessNode)
RETURN b
```
MongoDB:
```javascript
var startTime = new Date();
var review = db.review.findOne({ review_id: "TIYgnDzezfeEnVeu9jHeEw" });
var businessInfo = db.business.findOne({ business_id: review.business_id });
var endTime = new Date();
var elapsedTime = endTime - startTime;
print("Business Information:", JSON.stringify(businessInfo));
print("Query Time:", elapsedTime, "ms");
```