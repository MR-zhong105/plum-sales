# Claude Code 多智能体工作流编排

> 最后更新: 2026-08-09

## 目录
- [工作流简介](#工作流简介)
- [核心概念](#核心概念)
- [Workflow 脚本结构](#workflow-脚本结构)
- [编排模式](#编排模式)
- [实际案例](#实际案例)
- [调试与监控](#调试与监控)

---

## 工作流简介

工作流 (Workflow) 是 Claude Code 的自动化编排系统，允许定义复杂的多步骤任务，通过多个智能体并行或顺序执行。

**核心价值：**
- 并行处理独立任务
- 顺序执行依赖任务
- 错误恢复与重试
- 进度监控
- 结果汇总

---

## 核心概念

### Agent (智能体)

独立的任务执行单元：

```javascript
const result = await agent(
  "Fix the authentication bug in src/auth.js",
  {
    label: "auth-fix",           // 显示标签
    phase: "fix",                // 阶段分组
    model: "claude-sonnet-4-6",  // 指定模型
    isolation: "worktree"        // 隔离模式
  }
);
```

### Phase (阶段)

任务的逻辑分组：

```javascript
phase('Review')
const reviews = await parallel([
  () => agent("Check security", {phase: 'Review'}),
  () => agent("Check performance", {phase: 'Review'}),
  () => agent("Check tests", {phase: 'Review'})
])
```

### Pipeline (管道)

顺序处理每个项目：

```javascript
const results = await pipeline(
  files,
  f => agent("Review file", {file: f}),
  r => agent("Validate review", {review: r})
)
```

### Parallel (并行)

同时执行多个任务：

```javascript
const results = await parallel([
  () => agent("Task 1"),
  () => agent("Task 2"),
  () => agent("Task 3")
])
```

---

## Workflow 脚本结构

### 基本结构

```javascript
export const meta = {
  name: 'my-workflow',
  description: 'Description of the workflow',
  phases: [
    { title: 'Research', detail: 'Search for patterns' },
    { title: 'Implement', detail: 'Write code' },
    { title: 'Review', detail: 'Code review' }
  ]
};

// 脚本主体
async function main() {
  // 工作流逻辑
}

export default main;
```

### 完整示例

```javascript
export const meta = {
  name: 'full-stack-feature',
  description: 'Implement a full-stack feature with tests',
  phases: [
    { title: 'Research', detail: 'Search for patterns and APIs' },
    { title: 'Design', detail: 'Design the implementation' },
    { title: 'Backend', detail: 'Implement backend' },
    { title: 'Frontend', detail: 'Implement frontend' },
    { title: 'Test', detail: 'Write and run tests' },
    { title: 'Review', detail: 'Code review' }
  ]
};

export default async function main() {
  // Phase 1: Research
  phase('Research')
  const apiDocs = await agent('Search for API documentation', {
    label: 'api-research'
  })
  
  // Phase 2: Design
  phase('Design')
  const design = await agent('Design implementation', {
    label: 'design',
    args: { docs: apiDocs }
  })
  
  // Phase 3: Backend + Frontend (parallel)
  phase('Implement')
  const [backend, frontend] = await parallel([
    () => agent('Implement backend', {label: 'backend'}),
    () => agent('Implement frontend', {label: 'frontend'})
  ])
  
  // Phase 4: Test
  phase('Test')
  await agent('Run tests', {label: 'testing'})
  
  // Phase 5: Review
  phase('Review')
  const review = await agent('Code review', {label: 'review'})
  
  return { backend, frontend, review }
}
```

---

## 编排模式

### 1. 并行模式

```javascript
const results = await parallel([
  () => agent('Fix auth bug', {label: 'auth'}),
  () => agent('Fix payment bug', {label: 'payment'}),
  () => agent('Fix user bug', {label: 'user'})
])
```

### 2. 管道模式

```javascript
const results = await pipeline(
  issues,
  issue => agent('Analyze issue', {issue}),
  analysis => agent('Fix issue', {analysis})
)
```

### 3. 循环模式

```javascript
const bugs = []
while (bugs.length < 10) {
  const found = await agent('Find bugs')
  bugs.push(...found)
  log(`${bugs.length}/10 found`)
}
```

### 4. 漏斗模式

```javascript
// 多阶段筛选
const all = await parallel(finders.map(f => () => agent(f.prompt)))
const deduped = dedupeByFile(all.flatMap(r => r.bugs))
const verified = await parallel(deduped.map(f => () => 
  agent(`Verify ${f.title}`)
))
const confirmed = verified.filter(v => v.isReal)
```

---

## 实际案例

### 案例 1: 代码审查工作流

```javascript
export const meta = {
  name: 'code-review',
  description: 'Multi-perspective code review'
}

export default async function main() {
  // 多维度审查
  const dimensions = [
    { key: 'security', prompt: 'Check for security vulnerabilities' },
    { key: 'performance', prompt: 'Check performance issues' },
    { key: 'testing', prompt: 'Check test coverage' },
    { key: 'style', prompt: 'Check code style' }
  ]
  
  const results = await parallel(
    dimensions.map(d => () => 
      agent(d.prompt, {label: `review:${d.key}`})
    )
  )
  
  // 汇总结果
  return results.flatMap(r => r.findings || [])
}
```

### 案例 2: 测试生成工作流

```javascript
export const meta = {
  name: 'test-generator',
  description: 'Generate comprehensive tests'
}

export default async function main() {
  // 1. 分析代码
  const analysis = await agent('Analyze code structure')
  
  // 2. 生成测试计划
  const plan = await agent('Generate test plan', {analysis})
  
  // 3. 并行生成测试
  const tests = await parallel(
    plan.cases.map((c, i) => () => 
      agent(`Generate test: ${c}`, {label: `test-${i}`})
    )
  )
  
  // 4. 运行测试
  await agent('Run all tests')
  
  return tests
}
```

### 案例 3: Bug 修复工作流

```javascript
export const meta = {
  name: 'bug-fix',
  description: 'Systematic bug fix workflow'
}

export default async function main() {
  // 1. 复现 bug
  const repro = await agent('Create bug reproduction')
  
  // 2. 定位原因
  const cause = await agent('Find root cause', {repro})
  
  // 3. 实现修复
  const fix = await agent('Implement fix', {cause})
  
  // 4. 验证修复
  const verified = await agent('Verify fix', {fix})
  
  return { repro, cause, fix, verified }
}
```

---

## 调试与监控

### 进度显示

```javascript
log(`Found ${bugs.length} bugs, ${budget.remaining()}k tokens left`)
```

### 错误处理

```javascript
try {
  const result = await agent('Complex task')
  return result
} catch (error) {
  log(`Task failed: ${error.message}`)
  return null
}
```

### 预算控制

```javascript
const FLEET = budget.total ? Math.floor(budget.total / 100_000) : 5

while (budget.total && budget.remaining() > 50_000) {
  const result = await agent('Find more bugs')
  bugs.push(...result.bugs)
  log(`${bugs.length} found, ${Math.round(budget.remaining()/1000)}k remaining`)
}
```

### 工作流状态

```bash
# 查看运行中的工作流
/workflows

# 查看工作流日志
/workflows log <workflow-id>

# 停止工作流
/workflows stop <workflow-id>
```

---

## 最佳实践

### 1. 明确阶段

```javascript
phases: [
  { title: 'Research', detail: 'Gather information' },
  { title: 'Design', detail: 'Plan implementation' },
  { title: 'Implement', detail: 'Write code' },
  { title: 'Test', detail: 'Verify correctness' },
  { title: 'Review', detail: 'Quality check' }
]
```

### 2. 合理并行

```javascript
// 独立任务并行
const [a, b, c] = await parallel([
  () => agent('Task A'),
  () => agent('Task B'),
  () => agent('Task C')
])

// 依赖任务顺序
const result = await pipeline(
  items,
  item => agent('Process item', {item}),
  processed => agent('Validate', {data: processed})
)
```

### 3. 错误隔离

```javascript
const results = await parallel(
  items.map(item => () => 
    agent('Process', {item}).catch(err => ({error: err.message}))
  )
)
```

### 4. 日志记录

```javascript
log(`Starting phase: ${phase}`)
log(`Progress: ${completed}/${total}`)
log(`Results: ${JSON.stringify(data)}`)
```

---

## 参考资料

- [Claude Code Workflow 文档](https://docs.anthropic.com/en/docs/claude-code/workflows)
- [Workflow 示例仓库](https://github.com/anthropics/claude-code/tree/main/workflows)

---

## 相关资源

- [Anthropic Workflow 官方文档](https://docs.anthropic.com/en/docs/claude-code/workflows)
- [多智能体编排模式](https://docs.anthropic.com/en/docs/claude-code/agents)
- [工作流最佳实践](https://docs.anthropic.com/en/docs/claude-code/workflows-best-practices)
