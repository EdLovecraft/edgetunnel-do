# edgetunnel DO

基于 [cmliu/edgetunnel](https://github.com/cmliu/edgetunnel) 修改，将计算从 Worker 卸载到 Durable Object，绕过免费版 Worker 10ms CPU 时间限制。

## 与原版的区别

| | 原版 (KV) | 本版 (Durable Object) |
| :--- | :--- | :--- |
| **CPU 时间限制** | 免费版 10ms | 免费版 30s |
| **存储** | KV Namespace | DO Storage (SQLite) |
| **多线程下载** | 容易触发 CPU 超限断连 | 每个 WebSocket 连接独立 DO 实例，并行处理 |
| **部署方式** | Dashboard / Wrangler | 仅 Wrangler |
| **免费额度** | 100,000 请求/天 | 100,000 请求/天 + 13,000 GB-s duration |

## 部署

### 前置条件

- [Node.js](https://nodejs.org/) >= 18
- Cloudflare 账号

### 步骤

1. **克隆项目并安装依赖**

   ```bash
   git clone https://github.com/EdLovecraft/edgetunnel-do.git
   cd edgetunnel-do
   npm install
   ```

2. **登录 Cloudflare**

   ```bash
   npx wrangler login
   ```

3. **配置环境变量**

   编辑 `wrangler.toml`，在末尾添加你的变量：

   ```toml
   [vars]
   ADMIN = "你的管理员密码"
   # UUID = "90cd4a77-141a-43c9-991b-08263cfe9c10"  # 可选，强制固定 UUID
   ```

   或者部署后在 Dashboard 的 Worker 设置中添加环境变量。

4. **首次部署**

   首次部署需要包含 DO 迁移配置，在 `wrangler.toml` 中确保有：

   ```toml
   [[migrations]]
   tag = "v1"
   new_sqlite_classes = ["EDTDO"]
   ```

   然后执行部署：

   ```bash
   npx wrangler deploy
   ```

   > 部署成功后可以移除 `[[migrations]]` 段落，后续部署不再需要。

5. **绑定自定义域（可选）**

   在 Cloudflare Dashboard 中，进入该 Worker 的 **设置** > **触发器** > **添加自定义域**，填入你已托管在 CF 的域名，例如 `vless.example.com`。

6. **访问后台**

   访问 `https://<你的域名>/admin`，输入管理员密码即可登录。

### wrangler.toml 完整示例

```toml
name = "edgetunnel"
main = "_worker.js"
compatibility_date = "2024-09-23"

[durable_objects]
bindings = [
  { name = "DO", class_name = "EDTDO" }
]

# 首次部署时需要，部署成功后可移除
[[migrations]]
tag = "v1"
new_sqlite_classes = ["EDTDO"]

[vars]
ADMIN = "你的管理员密码"
# UUID = ""
# PROXYIP = ""
# URL = ""
```

## 环境变量说明

| 变量名 | 必填 | 示例 | 备注 |
| :--- | :---: | :--- | :--- |
| **ADMIN** | 是 | `123456` | 后台管理面板登录密码 |
| **KEY** | 否 | `CMLiussss` | 快速订阅路径密钥，访问 `/CMLiussss` 即可获取节点 |
| **UUID** | 否 | `90cd4a77-141a-43c9-991b-08263cfe9c10` | 强制固定 UUID，仅支持 UUIDv4 格式 |
| **PROXYIP** | 否 | `proxyip.cmliussss.net:443` | 全局自定义反代 IP |
| **URL** | 否 | `https://example.com` | 主页伪装地址（可填 URL 或 `1101`） |
| **GO2SOCKS5** | 否 | `*.google.com,blog.example.com` | 强制走 SOCKS5 的域名名单 |

## 高级用法

通过 PATH 路径动态切换代理方案：

- **指定 PROXYIP**
  ```
  /proxyip=proxyip.cmliussss.net
  /?proxyip=proxyip.cmliussss.net
  ```

- **指定 SOCKS5**
  ```
  /socks5=user:password@127.0.0.1:1080
  /socks5://user:password@127.0.0.1:1080
  ```

- **指定 HTTP 代理**
  ```
  /http=user:password@127.0.0.1:8080
  /http://user:password@127.0.0.1:8080
  ```

## 致谢

- [cmliu/edgetunnel](https://github.com/cmliu/edgetunnel) - 原版项目
- [zizifn/edgetunnel](https://github.com/zizifn/edgetunnel)

## 免责声明

本项目仅供教育、研究及个人安全测试目的。使用者须遵守所在地区法律法规，作者不对滥用行为承担责任。
