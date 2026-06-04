---
name: markdown-architecture-visualizer
description: 将 Markdown、架构说明、技术方案、系统设计文档转换为 16:9 4K 高清架构示意图。Use when Codex needs to read Chinese, English, or bilingual Markdown and create an architecture diagram, system diagram, solution diagram, workflow architecture image, cloud architecture visual, or technical diagram as a bitmap image.
---

# 架构图生成器

## 概览

将一份 Markdown 文档理解为可视化架构，并调用生图模型生成适合汇报、方案评审、产品说明或技术文档使用的 16:9 高清架构示意图。

支持中文、英文、以及中英混合 Markdown。中文文档默认使用中文标签；英文文档默认使用英文标签；中英混合文档保留关键术语原文，避免强行翻译服务名、产品名、接口名、模型名和云服务名。

## 工作流

1. 读取用户提交的 Markdown 文件或 Markdown 内容。
2. 提取架构信息：
   - 系统目标
   - 核心模块
   - 用户、客户端、服务、数据库、缓存、队列、模型、第三方系统等节点
   - 数据流、调用链路、事件流或部署关系
   - 外部依赖、边界系统、输入输出
   - 关键约束、风险、非功能目标
3. 先用简短中文总结你理解到的架构，指出主要节点、上下游关系、数据流向、调用路径和分层逻辑。
4. 选择视觉风格：
   - 用户明确指定风格时，直接使用用户指定风格。
   - 用户未指定风格时，一律使用默认“现代简约风”，不要根据 Markdown 内容自动切换风格。
5. 在需要用户确认风格时使用这句默认提示：

   ```text
   我已理解这份 Markdown。未指定风格时将默认采用“现代简约风”。如需其他风格，请明确指定“深色科技风”“企业咨询风”“云原生风”或自定义风格。
   ```

6. 构造结构化生图提示词。
7. 按“智能体适配”规则选择生图方法并生成位图图片。
8. 生成后检查：比例是否为 16:9、是否像架构图、主要节点是否出现、文字是否简洁可读、箭头是否表达方向、线条交叉是否过多。
9. 如用户要求保存到项目，移动或复制最终图片到项目目录；否则以内联预览为主。

## 智能体适配

本 Skill 支持在 OpenClaw、Codex、Claude Code 等智能体环境中使用。执行时先识别当前智能体和可用工具，再决定是否可以生图。

### Codex

- 如果当前智能体是 Codex，并且具备内置生图工具，优先使用 Codex 的 `image2` 生图能力完成架构图生成。
- 在 Codex 环境中，`image2` 通常由内置 `image_gen` 工具承载；调用时使用已经整理好的结构化提示词，并明确 `3840x2160, 16:9, crisp readable architecture diagram`。
- 如果 `image2` / `image_gen` 调用失败，按顺序尝试其他可用生图方法：
  - 已配置的 OpenAI 图像 API 或本地 imagegen CLI。
  - 当前环境提供的其他图像生成工具或插件。
  - 用户明确指定的外部生图方法。
- 不要在失败后静默切换到低质量、非位图、不可控或不支持架构图文本的方案。降级前说明原因和限制。

### OpenClaw

- 如果当前智能体是 OpenClaw，优先使用 OpenClaw 环境中可用的图像生成工具、图像模型连接器或 OpenAI 图像 API。
- 使用同一份结构化提示词，不要因为智能体不同而改变架构抽象、中文标签、风格选择和 16:9 4K 要求。
- 如果 OpenClaw 没有可用生图能力，输出错误信息并停止，不要伪造已生成图片。

### Claude Code

- 如果当前智能体是 Claude Code，先检查是否有可调用的图像生成工具、MCP 工具、API 脚本或用户提供的生图命令。
- 如果有可用生图能力，使用本 Skill 的结构化提示词调用该能力。
- 如果 Claude Code 只能生成文本，不能生成图片，则输出错误信息，并附上可复制的结构化生图 prompt，方便用户拿到其他生图工具中使用。

### 无生图能力时的错误提示

如果当前智能体不具备任何可用生图能力，必须明确报错：

```text
当前智能体环境没有可用的生图工具，无法直接生成架构图图片。我已完成 Markdown 架构理解和生图提示词整理；请为当前智能体配置 image2、image_gen、OpenAI 图像 API、MCP 生图工具或其他图像生成能力后重试。
```

报错时可以继续提供结构化 prompt，但不要声称图片已经生成。

## 默认风格规则

默认主风格为“现代简约风”。如果用户没有明确指定风格，始终使用“现代简约风”，不要根据内容主题自动切换风格。

- 现代简约风：默认风格。适合产品方案、技术方案、SaaS 系统、中台系统、微服务架构和一般系统架构展示。
- 深色科技风：仅当用户明确指定该风格时使用。
- 企业咨询风：仅当用户明确指定该风格时使用。
- 云原生风：仅当用户明确指定该风格时使用。

如果用户描述了自定义风格，优先使用用户的自定义风格。需要更细的视觉约束时读取 `references/style-guide.md`。

## 生图要求

- 画幅固定为 16:9。
- 目标质量为 4K 高清。若工具无法直接指定 4K 输出，在提示词中明确写入：`3840x2160, 16:9, crisp readable architecture diagram`。
- 输出类型为位图图片，不要默认改成 SVG、Mermaid、HTML 或 PPT，除非用户要求。
- 使用中文标签，标签必须短、清晰、可读；每个模块名称控制在 4 到 8 个字以内，避免大段说明文字。
- 架构图要优先表达结构关系，不要生成抽象海报、装饰性封面或纯氛围图。
- 避免水印、伪品牌 logo、无意义小字、不可读密集文字、过度 3D 装饰。
- 不要逐字复刻 Markdown 文本；先理解内容，再抽象成架构图。
- 不要臆造文档中没有的核心业务模块；如需补充客户端、接入层、基础服务、数据层、监控、安全、CI/CD 等通用模块，必须保持克制。
- 如果文档包含客户端、接入层、业务服务层、基础服务层、数据层、运维监控等内容，按逻辑层级组织。
- 箭头必须清晰表达方向，避免线条交叉过多。
- 画面需要具备产品方案汇报感，清晰、专业、可读，适合放入 PPT。
- 如架构关系复杂，优先保留主链路和关键边界，次要细节可以合并成分组节点。

## 结构化提示词模板

生成图片前，将 Markdown 内容整理成如下提示词。根据任务需要增删字段，但必须保留画幅、清晰度、节点和关系信息。

```text
Use case: infographic-diagram
Asset type: 16:9 4K architecture diagram
Audience: <汇报对象或读者，例如技术评审、CEO、客户方案会>
Language: <中文 / English / bilingual, preserve original technical terms>
Style: <用户确认的风格>
Diagram type: <系统架构图 / 数据流架构图 / 云架构图 / AI Agent 架构图 / 业务流程架构图>
Primary request: Convert the provided Markdown into a clear architecture diagram.
System goal: <一句话系统目标>
Main nodes: <4 到 8 个字的中文短标签节点列表>
Groups or layers: <客户端、接入层、业务服务层、基础服务层、数据层、运维监控等>
Connections: <A -> B: 调用/数据/事件/认证/推理/同步>
Must include: <必须出现的节点、边界、链路>
Avoid: copying Markdown verbatim, invented core business modules, unreadable tiny text, excessive paragraphs, watermark, fake logos, decorative poster layout
Composition: clear left-to-right flow or layered long layout, balanced spacing, readable Chinese labels, clear arrows, minimal line crossing, professional PPT-ready diagram layout
Generation backend: <Codex image2 / image_gen / OpenClaw image tool / Claude Code configured image tool / other available image model>
Output requirements: 3840x2160, 16:9, crisp readable architecture diagram, high contrast, clean typography
```

## 风格选择

内置以下四种主风格。需要更细的风格定义时，读取 `references/style-guide.md`。

- 现代简约风：白色或极浅蓝灰背景，浅色卡片、轻阴影、蓝/青/浅绿强调，适合 SaaS、产品方案、中台和微服务架构。
- 深色科技风：深色背景、半透明深色卡片、霓虹蓝/青/紫描边，适合 AI Agent、大模型、知识中枢和数据智能平台。
- 企业咨询风：白色或浅灰背景、深蓝/灰/浅蓝配色、矩阵和分层结构，适合管理层汇报、IT 规划、数字化转型和企业架构蓝图。
- 云原生风：浅色科技背景、云服务/容器/Pod/网关/数据库等统一扁平图标，适合 Kubernetes、云平台、DevOps 和微服务部署架构。
- 自定义：按用户提供的品牌、色彩、参考图或描述来构造风格。

## 交互规则

- 如果用户只提交 Markdown，没有指定风格，先总结理解并直接采用默认“现代简约风”，不要根据内容自动切换，也不需要额外询问风格。
- 如果用户已经指定风格，可以直接进入提示词构造和生图。
- 如果 Markdown 缺少关键架构信息，但仍能生成合理草图，先说明假设，再询问风格。
- 如果缺失信息会导致图完全不可用，先提出 1-3 个关键问题。
- 如果用户要求“先给我 prompt”，只输出结构化生图提示词，不调用生图工具。
- 如果用户要求“生成图”，按智能体适配规则调用可用生图能力，并报告最终使用的风格、生图后端和核心 prompt。
- 如果当前智能体不具备生图能力，按错误提示说明原因，并输出可复制的结构化 prompt。
