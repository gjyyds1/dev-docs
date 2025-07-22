# Modrinth API 无需 Token 验证的端点列表

## 注意

本文档使用Grok深度研究整理得出，仅供参考，不保证准确性。请以官方文档为准。

以下是根据 [Modrinth API 文档](https://docs.modrinth.com/api/) 整理的不需要 token 验证的 API 端点列表。这些端点主要用于获取公开数据，例如项目、版本、用户、标签、团队和统计信息。它们通常是只读操作（GET 请求），或某些用于检索数据的 POST 请求。根据文档，大多数获取公开数据的请求不需要身份验证，而创建、修改或访问私有数据的请求（如创建项目、修改版本或获取用户通知）则需要 token。

## 概述
Modrinth 是一个专注于 Minecraft 内容的平台，提供 mod、插件、数据包、着色器、资源包和 modpack。API 允许开发者与 Modrinth 的数据库交互，支持跨域资源共享（CORS），所有响应对所有站点公开。API 使用八位 base62 ID 标识项目、版本、用户等资源。速率限制为每分钟 300 次请求，所有请求需包含唯一的 User-Agent 头。

文档明确指出：“大多数请求不需要 token。通常，只有以下类型的请求需要 token：创建数据（如版本创建）、修改数据（如编辑项目）、访问私有数据（如草稿项目、通知、电子邮件和支付数据）。” 因此，本列表包括无需 token 的端点，主要为获取公开数据的 GET 请求和部分 POST 请求。

## 端点列表

### 1. 项目相关端点
这些端点用于获取与 Modrinth 平台上的项目（例如 mod、modpack 等）相关的信息，均为公开数据。

| 端点 | 方法 | 描述 | 备注 |
|------|------|------|------|
| `/search` | GET | 根据条件搜索项目 | 支持多种搜索参数，如项目类型、分类等 |
| `/project/{id|slug}` | GET | 获取特定项目的详细信息 | ID 或 slug 可互换使用 |
| `/projects` | GET | 通过 ID 获取多个项目的详细信息 | 接受 ID 列表 |
| `/projects_random` | GET | 获取随机项目列表 | 用于发现新项目 |
| `/project/{id|slug}/check` | GET | 检查项目 slug 或 ID 是否可用 | 用于验证标识有效性 |
| `/project/{id|slug}/dependencies` | GET | 获取项目的所有依赖项 | 返回依赖的项目和版本 |
| `/project/{id|slug}/version` | GET | 列出项目的所有版本 | 返回版本列表 |
| `/project/{id|slug}/members` | GET | 获取项目的团队成员 | 返回团队成员信息 |

### 2. 版本相关端点
这些端点用于获取与项目版本相关的信息，包括通过哈希检索版本的 POST 请求。

| 端点 | 方法 | 描述 | 备注 |
|------|------|------|------|
| `/version/{id}` | GET | 获取特定版本的详细信息 | 使用版本 ID |
| `/versions` | GET | 通过 ID 获取多个版本的详细信息 | 接受版本 ID 列表 |
| `/version_file/{hash}` | GET | 通过文件哈希获取版本 | 支持 SHA1 或 SHA512 哈希 |
| `/version_files` | POST | 通过多个文件哈希获取版本 | 接受哈希列表，返回版本信息 |
| `/version_files/update` | POST | 通过哈希和指定的加载器及游戏版本获取最新版本 | 支持过滤加载器和游戏版本 |

### 3. 用户相关端点
这些端点用于获取用户的公开信息。注意，某些用户相关端点（如获取关注项目）可能因涉及私有数据而需要 token。

| 端点 | 方法 | 描述 | 备注 |
|------|------|------|------|
| `/user/{id|username}` | GET | 获取用户的公开信息 | 返回用户名、简介等公开数据 |
| `/users` | GET | 通过 ID 获取多个用户的公开信息 | 接受用户 ID 列表 |
| `/user/{id|username}/projects` | GET | 获取用户拥有的项目 | 返回用户创建的项目列表 |

**注意**：`GET /user/{id|username}/follows`（获取用户关注的项目）和 `GET /user/{id|username}/notifications`（获取用户通知）可能需要 token，因为它们涉及用户特定数据，建议测试确认。

### 4. 标签相关端点
这些端点用于获取 Modrinth 平台支持的各种标签和分类，均为公开数据。

| 端点 | 方法 | 描述 | 备注 |
|------|------|------|------|
| `/tag/category` | GET | 获取项目类别列表 | 如“冒险”、“技术”等 |
| `/tag/loader` | GET | 获取支持的 mod 加载器列表 | 如 Fabric、Forge 等 |
| `/tag/game_version` | GET | 获取支持的游戏版本列表 | 如 1.19.4、1.20.1 等 |
| `/tag/license` | GET | 获取许可证列表 | 已弃用，建议使用 SPDX ID |
| `/tag/donation_platform` | GET | 获取捐赠平台列表 | 如 Patreon、PayPal 等 |
| `/tag/report_type` | GET | 获取报告类型列表 | 用于提交报告的类型 |
| `/tag/project_type` | GET | 获取项目类型列表 | 如 mod、modpack 等 |
| `/tag/side_type` | GET | 获取 side 类型列表 | 如客户端、服务器等 |

### 5. 团队相关端点
这些端点用于获取团队信息，均为公开数据。

| 端点 | 方法 | 描述 | 备注 |
|------|------|------|------|
| `/team/{id}/members` | GET | 获取团队的成员 | 返回团队成员列表 |
| `/teams` | GET | 获取多个团队的成员 | 接受团队 ID 列表 |

### 6. 统计信息
| 端点 | 方法 | 描述 | 备注 |
|------|------|------|------|
| `/statistics` | GET | 获取 Modrinth 实例的统计信息 | 如项目数、下载量等 |

### 7. 其他
| 端点 | 方法 | 描述 | 备注 |
|------|------|------|------|
| `/updates/{id|slug}/forge_updates.json` | GET | 为 Forge mod 开发者获取项目的 Forge 更新 JSON | 用于 Forge 模组更新 |

## 使用说明
- **User-Agent 要求**：所有请求必须包含唯一的 User-Agent 头，例如 `github_username/project_name/1.0.0` 或 `github_username/project_name/1.0.0 (contact@example.com)`。仅使用 HTTP 客户端库的 User-Agent（如 `okhttp/4.9.3`）可能导致请求被阻止。
- **速率限制**：每分钟 300 次请求，相关头包括：
  - `X-Ratelimit-Limit`：总限制。
  - `X-Ratelimit-Remaining`：剩余请求数。
  - `X-Ratelimit-Reset`：限制重置时间（Unix 时间戳）。
  - 如需更高限制，可联系 `admin@modrinth.com`。
- **服务器地址**：
  - 生产环境：`https://api.modrinth.com/v2`
  - 测试环境：`https://staging-api.modrinth.com/v2`
- **CORS 支持**：API 支持跨域资源共享，响应对所有站点公开。
- **ID 格式**：项目、版本、用户等使用八位 base62 ID 或 slug。

## 注意事项
- **潜在需要 token 的端点**：某些端点（如 `GET /user/{id|username}/follows` 和 `GET /user/{id|username}/notifications`）可能因涉及用户特定数据而需要 token，尽管文档未明确说明。建议在实际使用时测试确认。
- **API 版本**：当前为 v2，未来版本（如 v3）可能引入变化。GitHub token 认证将在 v3 中停止，建议使用个人访问 token。
- **OpenAPI 规范**：Modrinth 提供 OpenAPI 3.0.0 规范，可在 [https://docs.modrinth.com/openapi.yaml](https://docs.modrinth.com/openapi.yaml) 获取，详细列出端点和认证要求。
- **推荐工具**：测试 API 可使用 cURL、ReqBIN、Postman 或 Insomnia。
- **文档更新**：API 可能随时间更新，建议定期检查 [Modrinth API 文档](https://docs.modrinth.com/api/) 和 [OpenAPI 规范](https://docs.modrinth.com/openapi.yaml)。

## 参考资料
- [Modrinth API 文档](https://docs.modrinth.com/api/)
- [Modrinth OpenAPI 规范](https://docs.modrinth.com/openapi.yaml)
- [OAuth 指南](https://docs.modrinth.com/guide/oauth/)
- [支持页面](https://support.modrinth.com/)
- [GitHub 仓库](https://github.com/modrinth/labrinth)