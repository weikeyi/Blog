---
title: CORS 预检踩坑笔记（Nginx / Cesium 地形）
published: 2025-12-16
tags: [Nginx, CORS]
category: 跨域
draft: true
---

## 核心认知

CORS 是否生效，看的是 **OPTIONS 预检**的响应头是否正确，而不是取决于有没有写 `add_header`。

> 预检失败 → 浏览器直接拦截后续真实 GET → 前端报各种 `ERR_FAILED` / `Failed to obtain terrain tile`

---

## 常见踩坑

### 坑 A：`add_header` 默认只对 2xx/3xx 生效

预检如果走到 `404/405/500`，响应可能不带 CORS 头 → 浏览器报 `No 'Access-Control-Allow-Origin'`。

**对策**：

```nginx
add_header 'Access-Control-Allow-Origin' '*' always;
```

### 坑 B：`location` 覆盖导致规则丢失

请求命中更具体的子 `location`（如 `.terrain`）时，父级的 CORS 配置可能不再生效/不完整。

**对策**：

命中的子 `location` 也要 `include` 同一份 CORS snippet。

---

## 解决方案

目标就两件事：

1. **任何状态码都带 CORS 头**
   → `add_header ... always;`

2. **预检 OPTIONS 直接放行**（不走文件/不走 405）
   → OPTIONS 返回 `204`，并带齐 CORS 头

**原则**：哪个 `location` 会命中，就保证哪个 `location` 有完整 CORS 策略（别只在外层写）
