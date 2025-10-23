# On-Demand API 使用说明

实时爬取Twitter数据的HTTP接口。

---

## 快速开始

### 1. 启动服务

```bash
cd /home/dongpeng/repo/sn13/scraper/Social-Media-Data-Scraper
python src/main.py --enable-api-server --api-port 8000
```

服务地址：`http://localhost:8000`

### 2. 测试连接

```bash
curl http://localhost:8000/api/v1/health
```

返回 `"status": "healthy"` 即可使用。

---

## 使用示例

### 示例1：搜索关键词

```bash
curl -X POST "http://localhost:8000/api/v1/on_demand_scrape" \
  -H "Content-Type: application/json" \
  -d '{
    "keywords": ["bitcoin"],
    "limit": 20
  }'
```

### 示例2：搜索多个关键词（必须都包含）

```bash
curl -X POST "http://localhost:8000/api/v1/on_demand_scrape" \
  -H "Content-Type: application/json" \
  -d '{
    "keywords": ["bitcoin", "price"],
    "keyword_mode": "all",
    "limit": 50
  }'
```

### 示例3：搜索多个关键词（包含任一即可）

```bash
curl -X POST "http://localhost:8000/api/v1/on_demand_scrape" \
  -H "Content-Type: application/json" \
  -d '{
    "keywords": ["bitcoin", "ethereum"],
    "keyword_mode": "any",
    "limit": 50
  }'
```

### 示例4：指定时间范围

```bash
curl -X POST "http://localhost:8000/api/v1/on_demand_scrape" \
  -H "Content-Type: application/json" \
  -d '{
    "keywords": ["AI"],
    "start_date": "2025-10-01T00:00:00",
    "end_date": "2025-10-22T23:59:59",
    "limit": 100
  }'
```

### 示例5：搜索指定用户的推文

```bash
curl -X POST "http://localhost:8000/api/v1/on_demand_scrape" \
  -H "Content-Type: application/json" \
  -d '{
    "usernames": ["elonmusk"],
    "limit": 30
  }'
```

### 示例6：Python调用

```python
import requests

url = "http://localhost:8000/api/v1/on_demand_scrape"

payload = {
    "keywords": ["crypto"],
    "start_date": "2025-10-15T00:00:00",
    "limit": 50
}

response = requests.post(url, json=payload, timeout=120)
result = response.json()

print(f"获取到 {result['count']} 条推文")
print(f"耗时 {result['execution_time_seconds']} 秒")

# 访问第一条推文
if result['data']:
    tweet = result['data'][0]
    print(f"推文内容: {tweet['content']['text']}")
    print(f"作者: {tweet['content']['username']}")
    print(f"点赞数: {tweet['content']['like_count']}")
```

---

## 请求参数

| 参数 | 类型 | 必填 | 默认值 | 说明 |
|------|------|------|--------|------|
| `keywords` | 字符串数组 | 否* | - | 搜索关键词（1-10个） |
| `usernames` | 字符串数组 | 否* | - | Twitter用户名（1-10个） |
| `keyword_mode` | 字符串 | 否 | `"all"` | `"all"` = 必须包含所有关键词<br>`"any"` = 包含任一关键词即可 |
| `start_date` | 时间字符串 | 否 | - | 起始时间，格式 `YYYY-MM-DDTHH:MM:SS` |
| `end_date` | 时间字符串 | 否 | - | 结束时间，格式 `YYYY-MM-DDTHH:MM:SS` |
| `limit` | 整数 | 否 | `100` | 返回推文数量（1-500） |

**\*注意**：`keywords` 和 `usernames` 至少提供一个

---

## 响应格式

### 成功响应

```json
{
  "success": true,
  "count": 50,
  "query": "bitcoin since:2025-10-01",
  "execution_time_seconds": 18.5,
  "message": "Successfully fetched 50 tweets",
  "data": [
    {
      "uri": "https://twitter.com/user/status/123456",
      "datetime": "2025-10-20T10:30:00",
      "time_bucket_id": 460123,
      "source": 2,
      "label": "bitcoin",
      "content": {
        "username": "some_user",
        "text": "推文内容...",
        "url": "https://twitter.com/user/status/123456",
        "timestamp": "2025-10-20T10:30:00",
        "tweet_hashtags": ["#bitcoin"],
        "user_id": "123456789",
        "like_count": 42,
        "retweet_count": 15,
        "reply_count": 3,
        "view_count": 1500,
        "language": "en"
      },
      "content_size_bytes": 512
    }
  ]
}
```

### 字段说明

**顶层字段**：
- `success`: 是否成功
- `count`: 返回推文数量
- `query`: 实际执行的搜索条件
- `execution_time_seconds`: 执行耗时（秒）
- `data`: 推文数据数组

**推文字段（data数组中的每个对象）**：
- `uri`: 推文URL（唯一标识）
- `datetime`: 推文发布时间
- `content`: 推文详细信息（JSON对象）
  - `text`: 推文文本
  - `username`: 作者用户名
  - `like_count`: 点赞数
  - `retweet_count`: 转发数
  - `reply_count`: 回复数
  - `view_count`: 浏览数
  - `tweet_hashtags`: 标签数组

### 错误响应

```json
{
  "detail": "错误信息"
}
```

常见错误：
- `400`: 参数错误（检查请求参数）
- `408`: 请求超时（减少limit或缩小时间范围）
- `503`: 无可用账号（等待后重试）
- `500`: 服务器错误（查看日志或联系技术支持）

---

## 补充说明

### 性能参考

- 爬取20条推文：约10-15秒
- 爬取100条推文：约20-30秒
- 超时限制：90秒

### 使用建议

1. **limit设置**：根据需求设置，建议20-100条
2. **时间范围**：建议单次不超过1个月
3. **超时处理**：客户端超时设置建议120秒（略大于服务端90秒）

### 数据持久化

所有通过API爬取的数据会自动保存到数据库，可通过数据库查询历史数据。

### 日志位置

```
logs/{MMDDHHMM}/api_server.log
```

查看日志：
```bash
tail -f logs/{MMDDHHMM}/api_server.log
```

---

## 常见问题

**Q: 返回数据为空？**
A: 可能搜索条件无匹配结果，尝试调整关键词或扩大时间范围。

**Q: 响应超时？**
A: 减小limit（如100→50）或缩小时间范围。

**Q: 503错误？**
A: 账号池暂时无可用账号，等待5-10分钟后重试。

---

## 联系方式

技术问题请查看日志文件或联系开发团队。
