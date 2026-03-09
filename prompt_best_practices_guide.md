# Cursor Prompt 编写艺术：一位资深用户的高效指南

在 Cursor 中，你的核心技能正从“编写代码”转变为“编写指令”。一条精心设计的 Prompt，是区分“令人沮丧的反复修改”与“如魔法般一步到位”的关键。本指南整合了我作为资深用户在大量日常使用中沉淀出的最高效技巧。

## 核心原则：构建优质 Prompt 的基石

1.  **清晰胜于简洁**：指令要明确、详尽，哪怕感觉有些啰嗦。AI 不是读心者，模糊不清是你的头号敌人。
2.  **上下文为王**：AI 只知道你告诉它的信息。提供正确的上下文是不可或缺的。Cursor 的 `@` 符号引用是你的超能力。
3.  **迭代与优化**：你的第一版 Prompt 很少是最好的。从一个简单的指令开始，观察结果，然后用更多的约束和细节来优化它。

## 高影响力 Prompt 解析：C.R.I.S.P. 框架

为了持续写出高效的 Prompt，请使用 **C.R.I.S.P.** 框架。它能确保你覆盖所有必要元素。

### C - Context (上下文)

这是最关键的一步。你必须让 AI 聚焦于你代码库中的相关部分。

-   **`@文件`**: 引用一个或多个具体文件。
    -   *示例*: `"在 @App.tsx 中，添加一个新的状态变量..."`
-   **`@文件夹`**: 引入整个目录以获得更广的上下文。
    -   *示例*: `"检查 @src/components/ui 文件夹中的组件，然后创建一个风格一致的新 Button 组件。"`
-   **`@符号`**: 使用 `@` 加上函数/类/变量名来精确引用。
    -   *示例*: `"重构 @calculateTotalPrice 函数，使其能够处理折扣。"`
-   **`@文档`**: 引用你的文档，确保 AI 遵循项目特定的规范。
    -   *示例*: `"根据 @docs/api.md 中定义的 API，创建一个服务来获取用户数据。"`

### R - Role (角色)

告诉 AI 它应该扮演“谁”。这会设定回应的语气、风格和技术深度。

-   *普通*: `"写一个函数对数组进行排序。"`
-   *更优*: `"你是一位资深 TypeScript 工程师。请编写一个高效、通用的排序函数，并遵循函数式编程原则。"`

### I - Instruction (指令)

这是核心命令。使用强有力、无歧义的动词，并将复杂请求分解为更小的步骤。

-   *模糊*: `"修复这段代码。"`
-   *明确*: `"@useUserData 这个 hook 抛出了 'cannot read properties of undefined' 错误。请分析首次渲染时发生竞态条件的原因，并修复它，确保在数据被访问前其已存在。"`

### S - Style & Constraints (风格与约束)

定义输出的边界和期望的格式。这可以防止 AI 做出糟糕的选择或提供无法使用的代码。

-   **代码风格**: `"遵循 Airbnb JavaScript 风格指南。"`
-   **库/框架**: `"所有日期操作请使用 'date-fns'；不要使用 'moment.js'。"`
-   **语言特性**: `"将这段代码重构为使用现代 ES6+ 特性，如箭头函数和展开运算符。不要使用 'var'。"`
-   **输出格式**: `"仅提供函数的原始代码，不要包含任何解释或 markdown 格式。"`

### P - Pattern & Examples (模式与范例)

“身教”胜于“言传”。提供一个清晰的范例是让 AI 理解你期望的输出格式和逻辑的最快方法（这被称为“少样本提示” - Few-shot Prompting）。

-   *示例*:
    ```
    我想要转换一个用户对象。

    这是一个范例：

    输入:
    { "firstName": "John", "lastName": "Doe", "age": 30 }

    输出:
    { "fullName": "John Doe", "birthYear": 1995 }

    现在，请对以下对象应用相同的转换：
    { "firstName": "Jane", "lastName": "Smith", "age": 25 }
    ```

## 实战工作流与高级技巧

1.  **思维链 (Chain-of-Thought) 用于复杂逻辑**：如果任务复杂，可以要求 AI 在写代码前“一步步思考”。这能迫使它先进行推理，从而显著提高最终输出的质量。
    -   *Prompt*: `"我需要写一个函数来验证密码。首先，请一步步思考所需的验证规则（长度、特殊字符、数字等）。然后，基于这些规则编写 TypeScript 函数。"`

2.  **用 XML/JSON 结构化输出**：当你需要可预测、机器可读的输出时，指示 AI 以特定结构格式化其回应。这对于生成配置文件或结构化数据非常有用。
    -   *Prompt*: `"分析 @package.json 中的依赖项并进行分类。请以一个 JSON 对象的形式输出，包含 'runtime' 和 'development' 两个键。"`

3.  **“生成 → 优化”循环**：
    -   **第一步 (生成 - `Cmd+K`)**: 使用一个宽泛的 Prompt 生成骨架代码。`"创建一个名为 UserProfile 的新 React 组件，用于显示用户的姓名和邮箱。"`
    -   **第二步 (优化 - 选中代码 + `Cmd+K`)**: 选中生成的代码，应用具体的改进。`"为 props 添加 TypeScript 类型。将姓名加粗。添加一个显示加载动画的 loading 状态。"`

## 总结

高效的 Prompt 编写是一项需要练习的技能。从今天起，有意识地在日常任务中应用 **C.R.I.S.P. 框架**。留意哪些方法有效，哪些无效，并且不要害怕优化你的指令。通过从“代码编写者”转变为“代码指挥官”，你将在 Cursor 中解锁全新的生产力水平。

---

### 延伸学习与引用出处

为了进行更深入的研究，我强烈推荐来自大模型创建者的官方指南，它们为 Prompt 工程学提供了基本原则。

-   **Anthropic's Prompting Guide (Claude 官方)**: [https://docs.anthropic.com/claude/docs/prompt-engineering](https://docs.anthropic.com/claude/docs/prompt-engineering)
    -   *重要性*: Cursor 的默认模型通常基于 Claude，这使得该指南与我们的实践最为相关。
-   **OpenAI's Prompt Engineering Guide**: [https://platform.openai.com/docs/guides/prompt-engineering](https://platform.openai.com/docs/guides/prompt-engineering)
    -   *重要性*: 提供了普适性极佳的最佳实践，适用于与各类大语言模型互动。
-   **Cursor's Official Documentation**: [https://docs.cursor.com](https://docs.cursor.com)
    -   *重要性*: 解释了 Cursor 的特有功能，如 `@` 符号引用、自定义规则和其他编辑器集成。