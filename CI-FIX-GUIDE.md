# ZIO #10517 CI 修复指南

## 📋 问题总结

**CI 失败原因**：
1. ❌ **scalafmt 格式检查失败** - 代码不符合项目格式规范
2. ❌ **测试失败** - 可能是格式问题导致编译失败
3. ⏳ **CLA 未签署** - 需要签署 Contributor License Agreement

**受影响文件**：
- `core/shared/src/main/scala/zio/Cause.scala`
- `core-tests/shared/src/test/scala/zio/Issue9874Spec.scala`

---

## 🛠️ 修复方案

### 方案 A：本地安装 sbt 并格式化（推荐）

#### 1. 安装 sbt
```bash
# macOS (使用 Homebrew)
brew install sbt

# 或者下载 https://www.scala-sbt.org/download.html
```

#### 2. 运行 scalafmt 格式化
```bash
cd /Users/song_wpeng/.openclaw/workspace/zio-bounty

# 格式化所有文件
sbt scalafmtAll

# 或者只格式化修改的文件
sbt "scalafmtSbtCheck; scalafmtCheckAll"

# 检查格式是否正确
sbt check
```

#### 3. 提交并推送
```bash
git add -A
git commit -m "fix: format code with scalafmt"
git push fork fix/issue-9874-defect-priority
```

---

### 方案 B：使用 Docker 运行 scalafmt

```bash
# 使用 Docker 运行 scalafmt
docker run --rm -v $(pwd):/app -w /app \
  sbtscala/scala-sbt:2.13.11-8.0.1-17 \
  sbt scalafmtAll

# 然后提交推送
git add -A
git commit -m "fix: format code with scalafmt"
git push fork fix/issue-9874-defect-priority
```

---

### 方案 C：手动修复格式（无 Scala 环境时的临时方案）

根据 scalafmt 配置 (`.scalafmt.conf`)，手动修复以下问题：

#### Cause.scala 修复点

**原始代码**（第 136-146 行）：
```scala
final def failureOrCause: Either[E, Cause[Nothing]] =
  failureOption match {
    case Some(error) =>
      // If there are defects in the cause, return the full cause
      // to ensure defects are not silently ignored by error handlers
      if (self.isDie) Right(self.asInstanceOf[Cause[Nothing]])
      else Left(error)
    case None => Right(self.asInstanceOf[Cause[Nothing]])
  }
```

**检查项**：
- ✅ 缩进使用 2 空格
- ✅ 注释对齐
- ✅ 行尾无多余空格
- ✅ 大括号位置正确
- ✅ 最大列宽 120

#### Issue9874Spec.scala 修复点

**检查项**：
- ✅ import 语句排序
- ✅ 测试名称使用双引号
- ✅ 缩进一致
- ✅ 空行规范（方法间 1 空行）
- ✅ 行尾无空格

---

### 方案 D：从 CI Artifacts 下载格式化后的文件

1. 访问 CI 运行页面：https://github.com/zio/zio/actions/runs/22670515715
2. 查看 "lint" job 的日志
3. scalafmt 通常会输出具体哪一行格式不正确
4. 根据错误信息手动修复

---

## ✍️ CLA 签署

**必须完成**：访问 https://cla-assistant.io/zio/zio?pullRequest=10517

1. 点击 "Sign in with GitHub"
2. 授权 CLA assistant
3. 点击 "I agree" 签署 CLA
4. 刷新 PR 页面，CLA 检查应该变为 ✅

---

## 📊 验证步骤

### 1. 本地验证（如果有 sbt）
```bash
sbt check
```

### 2. 推送后验证
```bash
# 推送后 GitHub Actions 会自动运行
git push fork fix/issue-9874-defect-priority

# 检查 CI 状态
gh pr checks 10517
```

### 3. 预期结果
- ✅ lint - 应该通过
- ✅ test - 应该通过（如果代码逻辑正确）
- ✅ license/cla - 签署后通过

---

## 🎯 推荐执行顺序

1. **立即签署 CLA**（2 分钟）
   - 访问：https://cla-assistant.io/zio/zio?pullRequest=10517
   
2. **安装 sbt 并格式化**（10-15 分钟）
   ```bash
   brew install sbt
   cd /Users/song_wpeng/.openclaw/workspace/zio-bounty
   sbt scalafmtAll
   git add -A
   git commit -m "fix: format code with scalafmt"
   git push
   ```

3. **等待 CI 完成**（20-30 分钟）
   - 查看：https://github.com/zio/zio/actions/runs/22670515715

4. **检查 CI 结果**
   - 如果通过 → 等待维护者审查
   - 如果失败 → 根据新错误继续修复

---

## 📝 经验教训

### 问题根源
- 在提交前没有在本地运行 scalafmt 格式化
- 缺少 Scala 开发环境

### 改进措施
1. **安装 sbt** - 作为 ZIO 贡献者的必备工具
2. **配置 pre-commit hook** - 自动运行 scalafmt
3. **使用 IDE 插件** - IntelliJ IDEA + Scala 插件可以自动格式化

### Pre-commit Hook 示例
```bash
# .git/hooks/pre-commit
#!/bin/bash
sbt scalafmtAll
git add -A
```

---

## 🔗 相关链接

- PR: https://github.com/zio/zio/pull/10517
- Issue: https://github.com/zio/zio/issues/9874
- CLA: https://cla-assistant.io/zio/zio?pullRequest=10517
- CI Runs: https://github.com/zio/zio/actions/runs/22670515715
- scalafmt 配置：`.scalafmt.conf`
- ZIO 贡献指南：CONTRIBUTING.md

---

**创建时间**: 2026-03-06 11:30 (Asia/Shanghai)
**状态**: 等待 CLA 签署 + scalafmt 格式化
