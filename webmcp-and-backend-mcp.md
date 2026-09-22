# WebMCP and Backend MCP: Where Should an Agent Interface Live?

*An English-first observation, with a Chinese translation below*

## The short answer

If a team controls both the frontend and backend of an application, the core agent interface should usually be a backend MCP backed by the system's authoritative services. WebMCP is better understood as a browser-facing adapter: it helps an agent use a live web application with less guesswork, but it is not a replacement for a stable backend integration.

That distinction matters for internal systems such as firewall management. A backend MCP can expose authoritative, auditable operations directly. WebMCP can improve the experience of an agent working inside the browser, especially when the task depends on the current page, the current user session, or a workflow that exists only in the frontend.

## The central observation: WebMCP should follow the frontend framework

The most important long-term value of WebMCP is not that every team hand-writes another agent API in page scripts. That would create a second surface to keep in sync with the UI, permissions, state model, and backend behavior.

The stronger direction is framework integration. A mature Vue, React, Angular, or similar stack could derive WebMCP tools from the same component metadata, form schemas, route definitions, permission model, and domain actions that already produce the human UI. The framework would own registration, lifecycle, schemas, state refresh, and error handling. Developers would declare the business meaning and consequence level once.

In that model, WebMCP becomes a progressive enhancement of the existing frontend rather than a parallel application interface. The backend MCP remains the direct interface for authoritative system operations; WebMCP becomes the browser-native projection of selected frontend actions.

## What WebMCP is trying to solve

Traditional browser automation asks an agent to infer intent from a human interface:

1. find the right menu;
2. understand a button or form;
3. select the correct row;
4. handle pagination and dialogs;
5. type values;
6. verify the result.

That flow works, but every step leaves room for interpretation. WebMCP lets a web page declare structured tools with names, natural-language descriptions, JSON Schemas, and callbacks. Instead of guessing what a button means, an agent can discover a tool such as:

```text
listNatPolicies
getDnsConfig
searchTicket
submitApplication
filterResults
```

The immediate benefit is faster and more reliable browser actuation. The page tells the agent what an interaction means.

WebMCP is therefore a web-platform mechanism for making an existing web application agent-friendly. It is not primarily a way to publish a backend service to every client.

## Who proposed it and how mature it is

The proposal is being developed in the W3C Web Machine Learning Community Group. The initial proposal lists contributors from Microsoft and Google, including Brandon Walderman, Andrew Nolan, David Bokan, Khushal Sagar, and Hannah Van Opstal. The proposal is dated August 13, 2025.

Chrome currently describes WebMCP as a proposed web standard. Its implementation is in early preview and Origin Trial stages rather than being a stable, universal browser capability. Local experiments require an experimental Chrome flag, and the API, permission model, and security guidance are still being discussed.

There is meaningful developer interest around the public repository, but repository stars, forks, demos, and brand experiments are not the same thing as broad production adoption. The current evidence supports “early ecosystem and active experimentation,” not “mature standard.”

## Why backend MCP is usually the better core interface

When the team controls the backend, the direct path is:

```text
Agent → Backend MCP → Domain services → Database, device, or business system
```

For a firewall management system, that interface can expose:

```text
get_dns_config
list_nat_policies
get_nat_policy_detail
get_policy_hit_count
get_current_config_section
```

This path does not depend on an open tab, DOM structure, iframe behavior, frontend stores, pagination, keyboard focus, or a particular page layout. It is a better fit for:

- authoritative state queries;
- batch work and scheduled tasks;
- audit trails;
- permission enforcement;
- configuration revision checks;
- idempotent operations;
- independent CLI or background agents.

The backend remains the sole writer or the sole gateway to the writer. That makes the final acceptance condition easier to define: the returned state should match the authoritative system state.

## What WebMCP adds when the application is under our control

If we have full application development rights, the usual WebMCP problems are engineering problems rather than protocol barriers. A good implementation can make a tool call the backend, check a configuration revision, update the page state, and return a verified result.

A robust write tool could:

1. read the current backend state;
2. verify the expected revision;
3. ask for confirmation when the action is consequential;
4. perform the change through the domain service;
5. read the result back;
6. update the frontend state;
7. record the audit event.

That does not make WebMCP the core business interface. It makes WebMCP a thin browser adapter over the same domain services used by the backend MCP.

The ideal architecture is therefore:

```text
Shared domain actions
        │
        ├── Backend API
        ├── Backend MCP
        ├── WebMCP tools
        ├── UI forms and buttons
        ├── Permission checks
        └── Audit and tests
```

WebMCP should not become a parallel business-logic implementation.

## Where WebMCP has independent value

WebMCP is useful when the browser context is part of the task:

- the user is already signed in to a third-party system;
- the action depends on the current tenant, tab, filter, or selected object;
- the site has a complex wizard, editor, calendar, or form;
- the backend API is unavailable or cannot be changed;
- the website wants browser agents to discover its capabilities without a separately configured server;
- the user needs to watch and review a visible, human-centered workflow.

These are real cases, but they are different from the main case of an internal system whose backend we own.

## The framework question

The most promising long-term direction is for WebMCP to follow frontend frameworks, component libraries, and business schemas instead of being maintained as a second hand-written API.

A mature framework could manage:

- tool registration and unregistration with component lifecycle;
- JSON Schema generation;
- route and permission visibility;
- form state and parameter conversion;
- loading, cancellation, and error states;
- refreshing the page after a tool call.

The developer would still need to declare business meaning: whether a tool is read-only, whether it is consequential, what permission it requires, and which revision must match. A framework can recognize a button; it cannot safely infer whether “Apply” means save a draft, publish a policy, restart a service, or delete an old version.

That is why the likely mature model is generated adapters over a shared domain schema, not independent WebMCP business logic.

## Decision for an internal firewall system

For a system like our firewall management environment, the priority should be:

```text
Backend MCP
    authoritative, auditable system capability

Desktop browser control
    visual review, legacy UI, and fallback access

WebMCP
    optional browser-context adapter
```

The browser remains valuable even without WebMCP. It can inspect the page, verify the rendered result, operate an embedded CLI, and handle a system that does not yet have a clean API. If the backend MCP exists, however, WebMCP should not be introduced merely to wrap the same backend functions again.

The practical conclusion is simple:

> WebMCP is a promising browser-side agent interface, but its durable value depends on being integrated into frontend frameworks and shared business schemas. For an application whose backend we control, backend MCP should remain the core integration, while the frontend framework can generate or maintain WebMCP as a browser-facing projection when that context is useful.

## Sources

- [WebMCP API Proposal — W3C Web Machine Learning Community Group](https://webmachinelearning.github.io/webmcp/docs/proposal.html)
- [WebMCP official repository](https://github.com/webmachinelearning/webmcp)
- [Chrome WebMCP documentation](https://developer.chrome.com/docs/ai/webmcp)
- [Chrome WebMCP Origin Trial](https://developer.chrome.com/blog/ai-webmcp-origin-trial)
- [Google I/O 2026: WebMCP and the agentic web](https://developer.chrome.com/blog/chrome-at-io26)

---

# WebMCP 与后端 MCP：Agent 接口应该放在哪里？

*英文为主文，以下为中文对应版本*

## 结论先行

如果一个团队同时掌握应用的前端和后端开发权，那么核心 Agent 接口通常应该优先做成由权威业务服务支撑的后端 MCP。WebMCP 更适合作为浏览器侧适配层：它让 Agent 更容易使用正在打开的 Web 应用，但不应该替代稳定的后端集成。

对于防火墙管理这类内部系统，这个区别很重要。后端 MCP 可以直接暴露权威、可审计的查询和操作；WebMCP 则适合处理依赖当前页面、当前登录会话，或者只存在于前端流程中的任务。

## 核心观察：WebMCP 应该跟随现有前端框架

WebMCP 最重要的长期价值，不是让每个团队在页面脚本中手工再写一套 Agent API。那样会产生一套需要和页面、权限、状态模型及后端行为同步维护的平行接口。

更强的方向是框架整合。成熟的 Vue、React、Angular 或类似前端栈，可以从现有的组件元数据、表单 schema、路由定义、权限模型和业务动作中派生 WebMCP 工具。框架负责工具注册、生命周期、参数 schema、状态刷新和错误处理；开发者只需要一次性声明业务含义和操作风险等级。

这样，WebMCP 就是现有前端的渐进增强，而不是另一套平行应用接口。后端 MCP 继续作为权威系统操作的直接入口；WebMCP 则成为选定前端动作在浏览器中的原生投影。

## WebMCP 解决什么问题

传统浏览器自动化要求 Agent 从人类界面推断意图：

1. 找到正确菜单；
2. 理解按钮或表单；
3. 选中正确行；
4. 处理分页和弹窗；
5. 输入值；
6. 验证结果。

这条路径可以工作，但每一步都有解释空间。WebMCP 允许网页声明带有名称、自然语言描述、JSON Schema 和回调函数的结构化工具。Agent 不需要猜测按钮含义，而可以发现：

```text
listNatPolicies
getDnsConfig
searchTicket
submitApplication
filterResults
```

它的直接价值是减少浏览器操作中的歧义，提高 Agent 操作页面的速度和可靠性。

因此，WebMCP 是让现有 Web 应用更适合 Agent 使用的 Web 平台机制。它首先解决的是浏览器侧交互问题，而不是把后端系统能力发布给所有客户端。

## 谁提出的，成熟度如何

WebMCP 正在 W3C Web Machine Learning Community Group 中推进。初始提案列出的贡献者来自 Microsoft 和 Google，包括 Brandon Walderman、Andrew Nolan、David Bokan、Khushal Sagar 和 Hannah Van Opstal，提案日期为 2025 年 8 月 13 日。

Chrome 目前将 WebMCP 定义为拟议中的 Web 标准。它仍处于早期预览和 Origin Trial 阶段，而不是所有浏览器都稳定支持的通用能力。本地实验还需要打开 Chrome 的实验性开关，API、权限模型和安全规则仍在持续讨论。

它已经有一定开发者关注度，也有演示项目和大型网站进行实验，但仓库 star、fork、demo 和品牌试验不能等同于大规模生产采用。当前更准确的判断是：WebMCP 处在活跃实验和生态形成阶段，尚未成为成熟普及的标准。

## 为什么后端 MCP 更适合作为核心接口

当团队控制后端时，直接链路是：

```text
Agent → 后端 MCP → 业务服务 → 数据库、设备或业务系统
```

对于防火墙管理系统，可以直接提供：

```text
get_dns_config
list_nat_policies
get_nat_policy_detail
get_policy_hit_count
get_current_config_section
```

这条链路不依赖页面是否打开、DOM、iframe、前端 store、分页、键盘焦点或页面布局，更适合：

- 查询权威状态；
- 批量和定时任务；
- 审计；
- 权限控制；
- 配置版本校验；
- 幂等操作；
- CLI 和后台 Agent。

后端继续作为唯一写入方，或者至少作为唯一写入网关，最终一致性和验收条件会更清晰。

## 应用完全由我们控制时，WebMCP 增加什么

如果我们拥有完整开发权限，WebMCP 的许多问题都属于工程问题，而不是协议无法解决的问题。一个正确实现的工具可以调用后端、校验配置版本、在高风险操作前要求确认、更新页面状态并返回已验证结果。

但是，这并不意味着 WebMCP 应该成为核心业务接口。更合理的做法是让它作为后端 MCP 所使用的同一套业务服务之上的浏览器适配层。

理想架构是：

```text
统一业务动作
        │
        ├── 后端 API
        ├── 后端 MCP
        ├── WebMCP 工具
        ├── 页面表单和按钮
        ├── 权限校验
        └── 审计和测试
```

WebMCP 不应该复制一套平行业务逻辑。

## WebMCP 什么时候有独立价值

WebMCP 在浏览器上下文本身属于任务的一部分时更有价值：

- 用户已经登录第三方系统；
- 操作依赖当前租户、标签页、筛选条件或选中的对象；
- 网站有复杂向导、编辑器、日历或表单；
- 后端 API 无法使用或无法修改；
- 网站希望浏览器 Agent 无需单独配置服务器即可发现能力；
- 用户需要观看和复核一个可见的人机协同流程。

这些场景是真实的，但不同于“我们拥有内部系统后端”的主场景。

## 前端框架问题

WebMCP 最有希望的长期方向，是随着前端框架、组件库和业务 schema 一起生成，而不是维护第二套手写接口。

成熟框架可以负责：

- 随组件生命周期注册和注销工具；
- 生成 JSON Schema；
- 根据路由和权限决定工具是否可见；
- 处理表单状态和参数转换；
- 处理加载、取消和错误；
- 工具调用后刷新页面。

但开发者仍然需要声明业务语义：工具是否只读、是否有重大影响、需要什么权限、必须匹配哪个版本。框架可以识别按钮，却无法安全推断“应用”到底是保存草稿、发布策略、重启服务，还是删除旧版本。

因此，成熟形态更可能是从统一业务 schema 生成多个适配层，而不是单独维护 WebMCP 业务逻辑。

## 对内部防火墙系统的取舍

对于我们的防火墙管理系统，优先级应当是：

```text
后端 MCP
    权威、可审计的系统能力

桌面浏览器操作
    页面复核、旧系统兼容和兜底

WebMCP
    可选的浏览器上下文适配层
```

即使没有 WebMCP，浏览器仍然有价值：它可以检查页面、复核渲染结果、操作内嵌 CLI，并处理暂时没有干净 API 的系统。但如果后端 MCP 已经存在，就不应该仅仅为了重新包装同一组后端函数而引入 WebMCP。

最终判断是：

> WebMCP 是有前景的浏览器侧 Agent 接口，但它真正持久的价值取决于能否整合进前端框架和统一业务 schema。对于后端由我们控制的应用，后端 MCP 应该保持核心地位；当页面上下文有价值时，再由前端框架生成或维护 WebMCP 作为浏览器侧投影。

## 参考资料

- [WebMCP API Proposal — W3C Web Machine Learning Community Group](https://webmachinelearning.github.io/webmcp/docs/proposal.html)
- [WebMCP 官方仓库](https://github.com/webmachinelearning/webmcp)
- [Chrome WebMCP 文档](https://developer.chrome.com/docs/ai/webmcp)
- [Chrome WebMCP Origin Trial](https://developer.chrome.com/blog/ai-webmcp-origin-trial)
- [Google I/O 2026：WebMCP 与 Agentic Web](https://developer.chrome.com/blog/chrome-at-io26)
