---
description: "检查 Rust 代码中的静默失败（Silent Failure）模式，识别掩盖错误的代码"
---

# Rust 静默失败检查命令

你是一个 Rust 代码质量专家。你的任务是检查 Rust 代码中的静默失败（Silent Failure）模式，即那些掩盖、忽略或不当处理错误的代码。

## 检查范围

1. **源代码路径**: 需要检查的 Rust 项目路径（默认：当前工作目录）
2. **目标文件**: `src/**/*.rs`, `tests/**/*.rs`, `benches/**/*.rs`, `examples/**/*.rs`

## 检查步骤

### 步骤 1: 扫描所有 Rust 文件

使用 Glob 工具找到所有 Rust 源文件：
```bash
Glob: "**/*.rs"
```

或者根据需要指定特定目录：
```bash
Glob: "src/**/*.rs"
Glob: "tests/**/*.rs"
```

### 步骤 2: 搜索静默失败模式

使用 Grep 工具搜索以下模式：

#### 模式 1: `assert!(result.is_ok())` 缺少错误详情
```bash
Grep pattern: 'assert!\(.+\.is_ok\(\)\)'
output_mode: content
-n: true
-C: 2
```
**问题**: 断言失败时不会显示具体错误信息，调试困难。

#### 模式 2: `if let Err(e)` 或 `if let Err(_)` 忽略错误
```bash
Grep pattern: 'if\s+let\s+Err\([^)]*\)'
output_mode: content
-n: true
-C: 3
```
**问题**: 捕获错误但不处理，只是继续执行。

#### 模式 3: `let _ = xxx()` 忽略返回值
```bash
Grep pattern: 'let\s+_\s*=\s*\w+'
output_mode: content
-n: true
-C: 2
```
**问题**: 完全忽略函数返回值，可能忽略错误。

#### 模式 4: `.ok()` 忽略错误
```bash
Grep pattern: '\.ok\(\)'
output_mode: content
-n: true
-C: 2
```
**问题**: 将 Result 转为 Option，丢弃错误信息。

#### 模式 5: 测试中的 `unwrap_or()` / `unwrap_or_default()`
```bash
Grep pattern: '\.(unwrap_or|unwrap_or_default)\('
output_mode: content
-n: true
-C: 2
```
**问题**: 提供默认值掩盖真实错误。

#### 模式 6: `eprintln!("⚠️` 或 `println!("⚠️` 只打印警告不中断
```bash
Grep pattern: '(eprintln|println)!\("⚠️'
output_mode: content
-n: true
-C: 3
```
**问题**: 在测试中只打印警告，不 panic 或返回，导致后续失败。

#### 模式 7: match 语句中只打印警告
```bash
Grep pattern: 'Err\([e)]\)\s*=>\s*\{\s*(eprintln|println|println)!\('
output_mode: content
-n: true
-C: 4
```
**问题**: match Err 分支只打印日志不处理错误。

#### 模式 8: `if let Ok(_)` 只处理成功分支
```bash
Grep pattern: 'if\s+let\s+Ok\([^)]*\)\s*='
output_mode: content
-n: true
-C: 3
```
**问题**: 只处理成功情况，忽略失败情况。

### 步骤 3: 分析上下文

对每个匹配结果：
1. 读取完整的代码上下文（使用 Read 工具）
2. 判断是否是真正的静默失败
3. 排除可接受的情况（见下文"可接受的静默失败"）

### 步骤 4: 生成报告

生成一份详细的检查报告，包含：

## ✅ 静默失败检查报告

### 📋 概览
- 检查时间: [当前时间]
- 扫描文件数: [数量]
- 发现问题总数: [数量]
- 严重问题: [数量]
- 警告: [数量]

### 🔴 严重问题（必须修复）

#### 问题 1: [模式名称]
- **位置**: `[文件名:行号]`
- **模式**: `[具体的代码模式]`
- **代码片段**:
```rust
[展示问题代码]
```
- **问题**: [详细说明为什么这是静默失败]
- **影响**: [可能导致的问题]
- **修复建议**:
```rust
[提供正确的修复代码]
```

### 🟡 警告（建议修复）

#### 警告 1: [模式名称]
- **位置**: `[文件名:行号]`
- **模式**: `[具体的代码模式]`
- **代码片段**:
```rust
[展示问题代码]
```
- **问题**: [详细说明]
- **改进建议**:
```rust
[提供改进代码]
```

### ✅ 可接受的静默失败

以下情况被认为是可接受的，已在检查中排除：
- 清理操作失败（如数据库清理、文件删除）
- `.env` 文件加载失败（使用默认配置）
- 并发测试中的资源释放失败
- 已经有详细日志记录的非关键错误

### 📊 统计数据

- **检查的文件数**: [数量]
- **检查的代码行数**: [估算]
- **发现的模式**:
  - `assert!(result.is_ok())`: [数量]
  - `if let Err(e)`: [数量]
  - `.ok()`: [数量]
  - `unwrap_or()`: [数量]
  - 警告打印不中断: [数量]
- **真实问题率**: [百分比]%

### 🛠️ 修复优先级

1. **立即修复**: 测试中空投/设置失败后继续执行
2. **高优先级**: 核心业务逻辑中的错误忽略
3. **中优先级**: 改进错误信息的可读性
4. **低优先级**: 非关键路径的日志改进

## 检查要点

### 严重问题判断标准

以下情况被视为严重问题：
- ✅ **测试中关键操作失败后继续执行**: 空投、数据库初始化、外部依赖设置失败
- ✅ **核心业务逻辑忽略错误**: 交易、支付、数据处理等关键流程
- ✅ **错误处理不完整**: catch 了错误但只打印日志
- ✅ **使用默认值掩盖错误**: `unwrap_or_default()` 在不应该有默认值的地方

### 可接受的静默失败

以下情况被认为是可接受的：
- ✅ **清理操作**: `if let Err(e) = cleanup() { eprintln!("..."); }`
- ✅ **可选配置加载**: `.env` 文件、配置文件加载失败
- ✅ **并发测试的资源释放**: 清理失败不应阻塞测试
- ✅ **已有详细日志**: 使用 `log::error!` 或 `tracing::error!` 记录
- ✅ **显式处理**: 有注释说明为什么忽略此错误

### 代码模式分类

#### 🔴 高风险模式
1. **测试中的 match Err 只打印警告**:
```rust
// ❌ 错误
match services.set_token_balance(req).await {
    Ok(_) => {},
    Err(e) => println!("⚠️ 失败: {}", e),  // 测试继续！
}

// ✅ 正确
match services.set_token_balance(req).await {
    Ok(_) => {},
    Err(e) => panic!("❌ 空投失败，无法继续: {}", e),
}
```

2. **assert!(result.is_ok()) 不显示错误**:
```rust
// ❌ 错误
let result = some_operation();
assert!(result.is_ok());
let value = result.unwrap();

// ✅ 正确
let result = some_operation();
let value = result.expect("some_operation should succeed");
```

#### 🟡 中风险模式
1. **使用 `.ok()` 丢弃错误**:
```rust
// ❌ 可能有问题
let value = some_operation().ok();

// ✅ 更好
let value = some_operation()
    .inspect_err(|e| log::warn!("Operation failed: {}", e))?;
```

2. **`if let Ok()` 忽略 Err**:
```rust
// ❌ 可能有问题
if let Ok(value) = some_operation() {
    // 使用 value
}
// Err 被忽略

// ✅ 更好
match some_operation() {
    Ok(value) => { /* 使用 value */ }
    Err(e) => log::warn!("Operation failed: {}", e),
}
```

#### 🟢 低风险模式
1. **清理操作**:
```rust
// ✅ 可接受
if let Err(e) = cleanup() {
    eprintln!("⚠️ 清理失败: {}", e);
}
```

2. **可选配置**:
```rust
// ✅ 可接受
let config = dotenvy::dotenv().ok();
```

## 常见修复模式

### 1. 测试中的关键操作失败

**修复前**:
```rust
match services.airdrop(req).await {
    Ok(resp) => { /* 成功 */ }
    Err(e) => println!("⚠️ 空投失败: {}", e),  // 继续执行
}
```

**修复后**:
```rust
match services.airdrop(req).await {
    Ok(resp) => { /* 成功 */ }
    Err(e) => panic!("❌ 空投失败，无法继续测试: {}", e),
}
```

### 2. 改进断言错误信息

**修复前**:
```rust
assert!(result.is_ok());
let value = result.unwrap();
```

**修复后**:
```rust
let value = result.expect("operation_name should succeed");
// 或
let value = result.unwrap_or_else(|e| {
    panic!("operation_name failed: {}", e);
});
```

### 3. 避免使用 `.ok()`

**修复前**:
```rust
let value = some_operation().ok();
```

**修复后**:
```rust
let value = match some_operation() {
    Ok(v) => Some(v),
    Err(e) => {
        log::warn!("Operation failed: {}", e);
        None
    }
};
```

### 4. 完整处理 Result

**修复前**:
```rust
if let Ok(value) = result {
    // 只处理成功
}
```

**修复后**:
```rust
match result {
    Ok(value) => { /* 处理成功 */ }
    Err(e) => {
        log::error!("Operation failed: {}", e);
        // 错误处理逻辑
    }
}
```

## 输出格式

使用 Markdown 格式输出报告，包含：
- 清晰的标题层级
- emoji 图标标识状态（🔴 严重、🟡 警告、✅ 可接受）
- 代码引用（文件名:行号格式）
- 代码对比（修复前/修复后）
- 可执行的修复建议
- 统计数据和优先级

## 工具组合

建议使用以下工具组合：
1. **Grep**: 搜索模式
2. **Read**: 读取完整上下文
3. **Glob**: 查找文件
4. **Bash**: 可选，运行 `cargo clippy` 获取额外警告

## 执行检查

开始执行静默失败检查任务。按照以下顺序：
1. 扫描所有 Rust 文件
2. 搜索所有静默失败模式
3. 分析上下文，排除可接受的情况
4. 生成详细报告

**重要**: 排除测试清理操作、配置加载等可接受的静默失败。
