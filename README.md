# 控制台日志输出规范

## 概述

使用自定义的轻量级日志系统（无第三方库依赖），通过双通道输出（控制台 + 文件）记录应用运行状态。

- **定义位置**: `server.js:17-66`
- **日志目录**: `./log/`
- **日志文件命名**: `BBBS-YYYY-MM-DD-HH-MM-SS.log`

---

## 日志级别

| 级别 | 方法 | 用途 | 控制台颜色 | ANSI 代码 |
|------|------|------|------------|-----------|
| INFO | `Logger.info()` | 普通操作记录 | 青色 | `\x1b[36m` |
| SUCCESS | `Logger.success()` | 操作成功 | 绿色 | `\x1b[32m` |
| WARN | `Logger.warn()` | 警告信息 | 黄色 | `\x1b[33m` |
| ERROR | `Logger.error()` | 错误信息 | 红色 | `\x1b[31m` |
| DEBUG | `Logger.debug()` | 调试信息 | 灰色 | `\x1b[90m` |
| REQUEST | `Logger.request()` | HTTP 请求日志 | 多色 | 见下文 |

---

## 控制台日志格式

### 通用格式

```
[LEVEL] <message>
```

### 示例

```bash
# INFO 级别
[INFO] Bloret BBS 运行于 http://localhost:21111

# SUCCESS 级别
[SUCCESS] 数据库迁移完成

# WARN 级别
[WARN] 数据库连接池接近上限

# ERROR 级别
[ERROR] 数据库连接失败: ECONNREFUSED

# DEBUG 级式
[DEBUG] OAuth 回调参数: {...}
```

---

## 请求日志格式 (`Logger.request()`)

### 控制台格式

```
<METHOD> <URL> <STATUS> [<USER>] <DURATION>ms [<IP>]
```

### 颜色规则

| 元素 | 条件 | 颜色 | ANSI 代码 |
|------|------|------|-----------|
| **HTTP 方法** | GET | 绿色 | `\x1b[32m` |
| | POST | 黄色 | `\x1b[33m` |
| | 其他 | 白色 | `\x1b[37m` |
| **状态码** | 2xx, 3xx | 绿色 | `\x1b[32m` |
| | 4xx | 黄色 | `\x1b[33m` |
| | 5xx | 红色 | `\x1b[31m` |
| **用户名** | 已登录 | 紫色 | `\x1b[35m` |
| | 未登录 | 灰色 | `\x1b[90m` |
| **响应时间** | - | 青色 | `\x1b[36m` |
| **IP 地址** | - | 灰色 | `\x1b[90m` |

### 示例

```bash
# 已登录用户的 GET 请求
GET /api/structure 200 [Admin] 15ms [::1]

# 已登录用户的 POST 请求
POST /api/post 201 [Detrital] 234ms [::ffff:127.0.0.1]

# 未登录用户的 GET 请求
GET /api/user 401 [Guest] 5ms [192.168.1.100]

# 服务器错误
GET /api/data 500 [Guest] 1234ms [10.0.0.1]
```

---

## 文件日志格式

### 格式

```
[<locale datetime>] [<LEVEL>] <message>
```

### 示例

```
[2026/2/25 10:26:09] [INFO] Bloret BBS 运行于 http://localhost:21111
[2026/2/25 10:26:09] [INFO] 终端命令已启用: 输入 'Bloriko Daily' 可手动生成每日总结。
[2026/2/25 10:26:15] [REQUEST] GET /api/structure 200 [Admin] 15ms [::1]
[2026/2/25 10:26:20] [ERROR] 数据库连接失败: ECONNREFUSED
```

### 特点

- 纯文本格式，无 ANSI 颜色码
- 使用 `toLocaleString()` 格式化时间（依赖运行环境 locale）
- 追加写入模式

---

## 日志文件管理

| 配置项 | 值 | 代码位置 |
|--------|-----|----------|
| 日志目录 | `./log/` | `server.js:18` |
| 文件命名 | `BBBS-YYYY-MM-DD-HH-MM-SS.log` | `server.js:22-25` |
| 写入模式 | 追加 (`flags: 'a'`) | `server.js:26` |
| 轮转机制 | 无 | - |
| 清理机制 | 无 | - |

---

## 使用示例

### 后端代码

```javascript
// 记录普通操作
Logger.info('板块创建成功:', boardName);

// 记录成功操作
Logger.success('用户同步完成:', userId);

// 记录警告
Logger.warn('数据库连接池接近上限');

// 记录错误
Logger.error('文件上传失败:', error.message);

// 记录调试信息
Logger.debug('OAuth 回调参数:', req.query);

// 请求日志（通过中间件自动记录）
app.use((req, res, next) => {
    const start = Date.now();
    res.on('finish', () => {
        const duration = Date.now() - start;
        Logger.request(req, res, duration);
    });
    next();
});
```

### 前端代码

```javascript
// 前端使用原生 console 方法
console.log('页面加载完成');
console.error('API 请求失败:', error);
console.warn('性能警告: 渲染时间过长');
```

---

## 注意事项

1. **无日志级别过滤**: 所有级别（包括 DEBUG）都会输出到控制台和文件
2. **无自动轮转**: 日志文件永久累积，需手动清理
3. **时间格式依赖 locale**: 不同机器可能显示不同时间格式
4. **前端日志**: 仅使用原生 `console` 方法，无统一规范
5. **空目录**: 项目中存在未使用的 `logs/` 目录（实际使用 `log/`）
