# Netease Cloud Music Cookie Usage Guide（网页版）

> 目的：把浏览器里登录状态对应的 **Cookie** 拿出来，并在脚本/抓包工具里作为请求头使用，以便你发起“带登录态”的请求。  
> 注意：Cookie 相当于临时密码，请勿公开分享；泄露可能导致账号被他人直接使用。

---

## A. 获取 Cookie（Chrome / Edge）

1. **登录** 网易云音乐网页版（确保是你需要的账号）。
2. 按 **F12** 打开开发者工具（DevTools），切到 **Network（网络）**。
3. 勾选 **Disable cache（停用缓存）**（DevTools 打开时才生效），然后点一下 **清空（⦸）**。
4. 在网页里按 **Ctrl + R** 刷新，让页面重新产生网络请求。
5. 在请求列表里，优先点击一条：
   - **Type = `xhr` 或 `doc`** 的请求（通常更像“业务接口请求”）
   - 并且 **Request URL** 属于网易云域名（常见包含 `music.163.com` / `music.126.net` / `interface.music.163.com` 等）
6. 右侧打开 **Headers（标头）**：
   - 找到 **Request Headers（请求标头）** → **Cookie**
   - 复制 **Cookie 对应的整串值**（形如 `key=value; key2=value2; ...`）

### 更快捷的复制方式（推荐）
- 在 Network 请求上 **右键** → **Copy** → **Copy request headers**  
- 粘贴到文本里，找到这一行：
  - `cookie: MUSIC_U=...; __csrf=...; ...`
- 复制 `cookie:` 后面的整串内容即可。

---

## B. 在请求里“使用 Cookie”（最常见的两种方式）

你要做的事情很简单：把 Cookie 放进请求头 `Cookie` 里。

### 方式 1：curl 示例
```bash
curl 'https://music.163.com/'   -H 'User-Agent: Mozilla/5.0'   -H 'Cookie: MUSIC_U=...; __csrf=...; ...'
```

### 方式 2：Python requests 示例
```python
import requests

cookie = "MUSIC_U=...; __csrf=...; ..."  # 这里放你复制的整串 Cookie

headers = {
    "User-Agent": "Mozilla/5.0",
    "Cookie": cookie,
}

url = "https://music.163.com/"
r = requests.get(url, headers=headers, timeout=20)
print(r.status_code)
print(r.text[:500])
```

---

## C. 常见问题（排查清单）

### 1）Network 里很多条显示 “(disk cache)/(内存缓存)” 看不到完整请求头
- 勾选 **Disable cache（停用缓存）**
- 清空列表
- **Ctrl + R** 刷新后再点 `xhr/doc` 请求

### 2）我复制到的 Cookie 很短、或没有 `MUSIC_U` / `__csrf`
- 可能点到了第三方静态资源（图片、sprite、css/js）  
  → 换成 `xhr` / `doc`，并确认 URL 属于网易云域名
- 可能未登录成功  
  → 回到网页确认登录态仍有效（头像/昵称正常）

### 3）脚本里请求仍提示未登录 / 403
- Cookie 过期了：重新按 **A** 步骤复制一次
- 某些接口还需要额外的 headers（如 `Referer`、`Origin`）
- 如果接口需要 `csrf` 参数：通常它来自 Cookie 里的 `__csrf`（不同接口要求不同）

---

## D. 安全建议

- **不要把 Cookie 发到群里/工单/公开帖子**。
- 如果怀疑泄露：立刻退出登录或修改密码，并清理浏览器登录态。
- 建议把 Cookie 存在本机 `.env` 或系统环境变量里，避免写死在代码仓库。

---

如果你告诉我你要请求的具体接口（只需要接口 URL 和你希望拿到的数据类型，不要贴 Cookie），我可以把「必需的 headers、参数、以及一个可直接跑的请求模板」整理成更工程化的版本。
