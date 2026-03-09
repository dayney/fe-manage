# RAG 与代码上下文：项目文档/接口/设计接入与检索

摘要
- 目标：让 AI 在生成代码前“带着证据”地理解你的项目，从而输出既符合团队规范、又贴合真实业务与设计的实现。
- 核心观点：Cursor Rule 定义“如何做”（长期统一约束）；RAG 提供“做什么”（最新事实依据）。两者互补，缺一不可。

## 一、概念层区别
- Cursor Rule（规则）
  - 作用：给模型注入长期、稳定、不随任务变化的约束与偏好（目录结构、命名规范、代码风格、提交规范、技术选型禁用清单等）。
  - 生效方式：作为“系统提示/全局约束”常驻生效（可在项目内或全局 Cursor Rules）。
  - 特点：确定性强、通用性高、可继承到所有对话与编辑操作。
- RAG（检索增强）
  - 作用：为具体任务提供“最新/局部/易变”的事实性上下文（接口定义、业务文档、设计规格、运行时日志、第三方 API 文档）。
  - 生效方式：通过 @文件/@文件夹、接入 Swagger/ApiFox、Figma、DevTools MCP 等按需检索注入。
  - 特点：时效性强、任务相关、范围可控（只影响当前生成/编辑回合）。
- 互补关系（一句话）
  - Cursor Rule 像“团队法典/硬性规范”，是“如何做”的固定约束；
  - RAG 像“项目资料库/参考事实”，是“做这件事需要知道的具体信息”；
  - 两者互补：Rule 负责“行为边界与风格”，RAG 负责“上下文与事实依据”。

## 二、什么时候用哪一个
- 放进 Cursor Rule 的（长期稳定的规范）
  - 目录结构、编码风格、变量/文件命名规则、组件分层约定、状态管理原则、提交信息规范、不可使用的库清单、安全与隐私边界等。
- 用 RAG 注入的（会变/常更新的事实）
  - OpenAPI/ApiFox 接口定义、环境变量说明、业务流程文档、设计系统/组件库 Token、变更记录、运行日志/监控数据、特定页面的 Figma 图层信息。

## 三、在 Cursor 的执行路径上的差异
- 仅有 Rule
  - 效果：模型默认按你定义的风格组织输出（自动创建正确目录、遵循命名规则、不使用被禁用库），但可能“对不上事实”。
- 仅有 RAG
  - 效果：模型“带着证据”生成贴合事实的实现（按当前接口签名生成请求、按设计 Token 输出样式），但可能风格不一致。
- Rule + RAG（最佳）
  - 效果：既“像我们团队写的”，又“对上我们项目事实”。

## 四、冲突与优先级建议
- 冲突示例：Rule 禁用 moment，但文档示例用了 moment。
- 原则：以 Rule 为“硬约束”，以 RAG 为“事实参考”。
- 做法：在 Prompt 顶部明确优先级：
  - “若文档示例与项目规则冲突，以项目规则为准，替换实现但保持行为一致。”
  - 针对常见冲突（如 moment → date-fns）在 Rule 中提供“替代表达”。

## 五、常见误区
- 只用 Rule、不接 RAG：风格统一，但具体实现易“拍脑袋”，对不上最新接口/设计。
- 只用 RAG、不设 Rule：能对上事实，但输出风格不一致、可维护性差、容易引入被禁用依赖。
- 把高频变更信息放进 Rule：导致规则臃肿且频繁编辑，维护成本高。应迁移到 RAG 来源。

## 六、组合用法的最佳实践
- 目录与示例
  - Cursor Rule（长期）：项目结构、命名/风格、组件分层、提交规范、禁止清单。
  - RAG 源（可变）：@api/openapi.json、@docs/business.md、@design/tokens.json、Figma/ApiFox MCP。
- 模板 Prompt（组合使用）
  - “按项目规则创建页面骨架”
    - 规则：已在 Cursor Rule 中固化。
    - RAG：@docs/page-template.md @design/tokens.json。
    - 指令：基于上述文档，生成 UserList 页面骨架，严格遵循项目目录与命名规则（若文档示例冲突，以规则优先）。
  - “按最新接口实现”
    - 规则：使用 axios 封装、错误统一处理、不可使用 moment。
    - RAG：@apifox 获取“获取用户列表”接口定义。
    - 指令：依据接口真实签名生成 TS API 方法与类型，接入全局错误处理，不使用 moment。
- 更多可复用模板（示例）
  - “按设计 Token 统一样式”：
    - 规则：CSS 命名约定、禁止内联魔法数。
    - RAG：@design/tokens.json。
    - 指令：将该组件样式重构为使用 design tokens（颜色/字号/间距），保留现有 DOM 结构与交互。

## 七、落地建议
- 建 Rule（一次性+稳定维护）
  - 把“永远正确”的规范沉淀到 Cursor Rule，做到简洁、分层、可枚举、可测试（附示例与反例）。
- 建 RAG 源（持续更新）
  - 将易变资料统一入口化：/docs、/api/openapi.json、/design/tokens.json；对接 MCP（ApiFox、Figma、DevTools）。
- 工作流（建议固化为团队标准）
  - Chat 确认方案 → Composer 执行 → Edits 审阅 → Diff/Commit。
  - 每次任务在 Prompt 顶部声明：规则优先级、RAG 引用清单、输出要求。
- 评估与回归
  - 指标：开发时间、返工次数、PR 周期、Bug 率。
  - 回归：为关键 Prompt 建立“回归用例”（同一上下文多次生成结果一致性）。

## 八、一句话总结
- Cursor Rule 定义“怎么做”（长期的统一约束）；RAG 提供“做什么”（当前任务的真实信息）。两者合用，才能实现风格一致且对齐事实的高质量产出。

---

## 附：扩展示例（可用于演示）
- 项目文档 RAG
  - 你：@docs/component-guide.md 按团队规范创建一个 Form.Item 组件示例，包含错误态与辅助说明。
  - 产出：符合命名与分层约定的可复用组件，示例代码与文档一致。
- 接口 RAG（ApiFox MCP）
  - 你：@apifox 生成“创建订单”接口的 service 与类型定义，集成全局错误处理与重试策略。
  - 产出：基于实时接口定义的精确代码（URL/方法/参数/返回值/错误码）。
- 设计 RAG（Figma MCP）
  - 你：@figma 将“登录页”转为 React 组件，样式使用 @design/tokens.json 的主色与圆角体系。
  - 产出：结构与样式与设计稿一致，token 使用规范统一。
- 调试 RAG（Chrome DevTools MCP）
  - 你：检查当前页面 .login-button 的实时 CSS 规则，指出与设计不一致项并生成修复 PR。
  - 产出：基于实时 DOM/CSS 的差异报告与代码修改。

---

## 参考与最佳实践出处
- Anthropic Prompt Engineering（Claude 官方）
  - https://docs.anthropic.com/claude/docs/prompt-engineering
- OpenAI Prompt Engineering Guide
  - https://platform.openai.com/docs/guides/prompt-engineering
- Cursor 官方文档（Rules、@引用、Composer、MCP）
  - https://docs.cursor.com
- RAG 方法与工程实践
  - LangChain RAG Cookbook：https://python.langchain.com/docs/expression_language/cookbook/retrieval
  - LlamaIndex RAG Guide：https://docs.llamaindex.ai/en/stable/getting_started/concepts.html#rag
- OpenAPI 与接口管理
  - OpenAPI Spec：https://spec.openapis.org/oas/latest.html
  - ApiFox 文档中心：https://www.apifox.cn/help/
- 设计系统与 Tokens
  - Design Tokens（W3C 草案）：https://design-tokens.github.io/community-group/
  - Figma Tokens 插件：https://www.figma.com/community/plugin/843461159747178978
- MCP 生态
  - MCP 服务器索引与生态：https://mcp.so
