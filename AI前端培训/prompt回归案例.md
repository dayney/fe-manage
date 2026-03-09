# Prompt 回归测试实战案例 (Vue 3)

本文档旨在通过一个具体的 Vue 3 案例，阐述如何为团队的标准化 Prompt 建立自动化回归测试，从而系统性地保障 AI 辅助开发的质量与稳定性。

---

### **1. 场景 (The Scenario)**

假设你的团队使用 Vue 3 和 TypeScript，一个高频的重复性任务是**创建一个新的 SFC (Single File Component)**。为了保证代码风格和结构的一致性，每个新组件都必须遵循以下规范：

-   `ComponentName.vue`: 使用 `<script setup lang="ts">` 的组合式 API，并包含 `<style scoped>`。
-   `ComponentName.stories.ts`: 配套的 Storybook 故事文件，用于组件文档和测试。

这是一个非常适合用标准 Prompt 自动化的任务。

### **2. “黄金 Prompt” (The "Golden" Prompt for Vue 3)**

经过团队的共同打磨，你们沉淀出了一个针对 Vue 3 的“黄金 Prompt”，用于快速生成组件骨架：

```prompt
你是一位资深 Vue 3 工程师，精通 TypeScript、组合式 API 和 Storybook。

请为我创建一个名为 {{componentName}} 的新组件，它接收以下 props：
{{propsDefinition}}

请严格遵循以下要求生成两个文件：

1.  **`{{componentName}}.vue`**:
    - 必须使用 `<script setup lang="ts">` 语法。
    - 必须使用 `defineProps` 来定义 props 类型。
    - 模板部分必须包含 props 的基本使用。
    - 必须包含 `<style scoped>` 块，并有一个根 class `.root`。

2.  **`{{componentName}}.stories.ts`**:
    - 必须导入该 Vue 组件。
    - 必须包含一个默认导出（meta），并正确设置 `component` 和 `title`。
    - 必须至少包含一个名为 `Default` 的 story，并为其提供 args。
```

### **3. 问题：为什么需要回归测试？ (The Problem)**

这个“黄金 Prompt”虽然好用，但它的质量可能会因为以下原因而退化：

-   **模型升级**: Cursor 底层的 AI 大模型更新换代，可能导致对指令的理解发生细微变化，破坏原有的输出格式。
-   **Prompt 误改**: 团队成员在“优化”Prompt 时，不小心删除了关键约束（如 `<style scoped>`），导致生成的代码存在缺陷。
-   **团队规范变更**: 团队决定引入 UnoCSS/Tailwind CSS，需要更新 Prompt，并确保新版 Prompt 在各种情况下都能正确工作。

如果没有回归测试，这些问题只能在 Code Review 甚至运行时才被发现，非常被动。

### **4. 搭建回归测试 (Setting up the Test for Vue 3)**

我们在项目中创建一个 `prompts-regression` 目录来存放测试用例。

**`prompts-regression/create-vue-component/case-01.json` (测试用例 1)**

```json
{
  "prompt_template_path": "./prompt-vue.txt",
  "variables": {
    "componentName": "VButton",
    "propsDefinition": "- `label`: string\n- `disabled`: boolean"
  },
  "assertions": [
    { "type": "file_exists", "path": "VButton.vue" },
    { "type": "file_exists", "path": "VButton.stories.ts" },
    { "type": "file_content_contains", "path": "VButton.vue", "snippet": "<script setup lang=\"ts\">" },
    { "type": "file_content_contains", "path": "VButton.vue", "snippet": "defineProps<{" },
    { "type": "file_content_contains", "path": "VButton.vue", "snippet": "label: string" },
    { "type": "file_content_contains", "path": "VButton.vue", "snippet": "<style scoped>" },
    { "type": "file_content_contains", "path": "VButton.stories.ts", "snippet": "import VButton from './VButton.vue'" },
    { "type": "command_succeeds", "command": "npx eslint ./VButton.vue --max-warnings=0" },
    { "type": "command_succeeds", "command": "npx vue-tsc --noEmit" }
  ]
}
```

这个测试用例清晰地定义了：

-   **输入 (Input)**: 使用哪个 Prompt 模板，以及传入的变量（组件名 `VButton` 和 props 定义）。
-   **断言 (Assertions)**:
    -   **文件检查**: 确保 `.vue` 和 `.stories.ts` 文件都已创建。
    -   **内容检查**: 确保关键代码片段（如 `<script setup>`, `<style scoped>`, `defineProps`）存在。
    -   **质量检查**: 对生成的文件自动运行 **ESLint** 和 **Vue-TSC**，确保代码符合规范且没有类型错误。这是保障质量的关键环节。

### **5. 自动化脚本 (The Regression Script)**

我们可以编写一个框架无关的 Node.js 脚本 (`scripts/run-prompt-regression.js`) 来执行这些测试。脚本的核心逻辑是：

1.  读取 `.json` 测试用例文件。
2.  根据模板和变量，组装成最终的 Prompt。
3.  调用 AI 模型 API，获取生成的代码文件。
4.  将生成的文件写入一个临时目录。
5.  遍历 `assertions` 数组，逐一执行检查。
6.  测试通过后清理临时目录，失败则抛出错误并退出。

*(注：该脚本的具体实现可参考相关开源实践，核心是流程自动化。)*

### **6. 集成与结果 (Integration & Outcome)**

-   **集成**: 在 `package.json` 中添加一个脚本，并将其加入到 CI/CD 流水线中（如 GitHub Actions, GitLab CI）。

    ```json
    "scripts": {
      "test:prompt": "node scripts/run-prompt-regression.js"
    }
    ```

-   **结果**: 
    -   **测试通过 (Pass)**: 当 Prompt 被修改或模型升级后，如果 CI 流水线依然是绿色的，团队就能确信这个核心的自动化工具没有被破坏。
    -   **测试失败 (Fail)**: 如果 AI 生成的代码缺少类型定义，CI 会立刻变红，并给出明确的错误信息，如 `Command failed: npx vue-tsc --noEmit`。问题在代码合并前就被精准拦截，避免了将有缺陷的自动化流程扩散到整个团队。

---

通过这种方式，Prompt 回归测试成为团队 AI 工程化能力的重要一环，它将 Prompt 从“个人技巧”提升为“可维护、可测试的团队资产”。
