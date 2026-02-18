# 反爬虫功能与配置说明

为了提升 `akshare` 爬虫的稳健性并降低被目标网站封锁的风险，本项目实现了一套集中的反爬虫增强机制。

## 核心机制

### 1. User-Agent 随机切换
- **实现原理**：在 `akshare/utils/user_agents.py` 中维护了一个现代浏览器的 User-Agent 列表。
- **功能说明**：每次通过 `ak_get` 或 `ak_post` 发送请求时，系统会自动从列表中随机抽取一个 User-Agent 注入到请求头中。如果用户手动传入了 `User-Agent`，则优先使用用户传入的值。

### 2. 随机请求延迟
- **实现原理**：在发送网络请求前，系统会自动触发一个随机的等待时间。
- **功能说明**：随机延迟范围设定在 0.1s 到 0.5s 之间。此举旨在模拟人类行为，打破固定频率的请求模式，从而有效规避目标服务器的频率检测（Rate Limiting）。

### 3. 配置化 Cookies 管理
- **实现原理**：通过 `akshare/utils/context.py` 实现全局配置中心，支持 Cookies 的统一注入。
- **功能说明**：用户可以动态设置全局 Cookies，这些 Cookies 会被自动添加到所有通过统一包装器发出的请求中。

## 使用指南

### 开发者调用

在编写新的爬虫接口或重构旧接口时，请**务必**使用 `akshare.request` 模块中的 `ak_get` 或 `ak_post` 替代直接调用 `requests.get` 或 `requests.post`。

示例：
```python
from akshare.request import ak_get

url = "https://example.com/api"
params = {"symbol": "600000"}
# ak_get 会自动处理 UA、Cookies 和延时
r = ak_get(url, params=params)
data = r.json()
```

### 动态配置 Cookies

对于需要长期有效 Cookies 或特定会话凭证的接口（如东方财富网），用户可以在程序启动时或运行过程中动态更新 Cookies：

```python
import akshare as ak

# 设置全局 Cookies
new_cookies = {
    "JSESSIONID": "xxxxxx",
    "qgqp_b_id": "yyyyyy",
    # ... 其他 cookie 键值对
}
ak.set_cookies(new_cookies)

# 之后的请求将自动带上上述 Cookies
df = ak.stock_individual_info_em(symbol="000002")
```

## 维护与扩展

- **更新 UA 库**：若需增加更多现代浏览器的 User-Agent，请修改 `akshare/utils/user_agents.py`。
- **调整延时**：系统默认采用 0.1s - 0.5s 的随机延时。若特定场景需要调整，可修改 `akshare/request.py` 中的 `_handle_anti_crawler_features` 函数。

---
**提示**：由于本项目已对 300+ 个接口进行了大规模重构，所有基于 `requests` 的基础请求均已具备上述增强功能。
