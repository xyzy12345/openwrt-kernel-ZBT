# 合并冲突解决指南 (Merge Conflict Resolution Guide)

## 问题 (Problem)

此 PR 与 main 分支存在冲突，因为两个分支都修改了 `.github/workflows/compile_kernel.yml` 文件。

## 原因 (Cause)

- **main 分支**: 您手动将 kernel 版本改为 6.12
- **此 PR 分支**: 也使用 6.12，但添加了关键修复（checkout、磁盘清理、空间验证）

## 解决方案 (Solution)

### 方法 1：通过 GitHub UI 合并（推荐）

1. 在 PR 页面点击 "Resolve conflicts"
2. 选择保留此 PR 的完整内容（包含所有修复）
3. 删除冲突标记 `<<<<<<<`、`=======`、`>>>>>>>`
4. 点击 "Mark as resolved"
5. 点击 "Commit merge"

### 方法 2：命令行合并

**⚠️ 注意**: 执行前请确认 main 分支没有其他重要更改需要保留。

```bash
# 切换到 main 分支
git checkout main

# 合并此 PR 分支
git merge copilot/fix-unknown-error

# 如有冲突，保留 copilot/fix-unknown-error 的 workflow 文件
git checkout --theirs .github/workflows/compile_kernel.yml

# 完成合并
git add .github/workflows/compile_kernel.yml
git commit -m "Merge PR: Fix workflow with disk cleanup and checkout"
git push
```

### 方法 3：手动应用修复到 main

如果不想合并 PR，可以直接修改 main 分支的 workflow，添加这些步骤：

```yaml
name: Compile Kernel for MT7981 (OpenWrt 6.12)
on:
  workflow_dispatch:

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      # ===== 添加这些步骤 =====
      - name: Checkout repository
        uses: actions/checkout@v4

      - name: Check initial disk space
        run: |
          echo "=== Disk Space Before Cleanup ==="
          df -h
          echo "=== Disk Usage by Directory ==="
          sudo du -h --max-depth=1 / 2>/dev/null | sort -hr | head -20

      - name: Free up disk space
        run: |
          echo "Cleaning up disk space..."
          sudo apt-get clean
          sudo apt-get autoclean
          sudo apt-get autoremove -y
          sudo rm -rf /usr/share/dotnet
          sudo rm -rf /usr/local/lib/android
          sudo rm -rf /opt/ghc
          sudo rm -rf /opt/hostedtoolcache/CodeQL
          sudo rm -rf /var/cache/apt/archives/*
          echo "Cleanup completed"

      - name: Check disk space after cleanup
        run: |
          echo "=== Disk Space After Cleanup ==="
          df -h
          AVAILABLE_KB=$(df / | tail -1 | awk '{print $4}')
          AVAILABLE_NUM=$(echo "$AVAILABLE_KB" | sed 's/[^0-9.]//g')
          
          if echo "$AVAILABLE_KB" | grep -q "G"; then
            AVAILABLE_GB="$AVAILABLE_NUM"
          elif echo "$AVAILABLE_KB" | grep -q "M"; then
            AVAILABLE_GB=$(awk "BEGIN {printf \"%.2f\", $AVAILABLE_NUM/1024}")
          elif echo "$AVAILABLE_KB" | grep -q "T"; then
            AVAILABLE_GB=$(awk "BEGIN {printf \"%.2f\", $AVAILABLE_NUM*1024}")
          else
            AVAILABLE_GB=$(awk "BEGIN {printf \"%.2f\", $AVAILABLE_NUM/1024/1024}")
          fi
          
          echo "Available space: ${AVAILABLE_KB}"
          echo "Available space in GB: ${AVAILABLE_GB}"
          
          if awk "BEGIN {exit !($AVAILABLE_GB < 15)}"; then
            echo "ERROR: Insufficient disk space. Need at least 15GB, have ${AVAILABLE_GB}GB"
            exit 1
          fi
          echo "Disk space check passed: ${AVAILABLE_GB}GB available"
      # ===== 结束添加 =====

      - name: Compile the Kernel
        uses: ophub/amlogic-s9xxx-armbian@main
        with:
          build_target: kernel
          kernel_version: 6.12
          kernel_auto: false
          kernel_sign: -mt7981-custom
          kernel_repo: padavanonly/immortalwrt-mt798x-6.6
          kernel_config: kernel/config_path
```

## 为什么需要这些修复 (Why These Fixes Are Needed)

1. **Checkout 步骤**: workflow 无法访问 `kernel/config_path` 目录
2. **磁盘清理**: 释放 10-20GB 空间（从 14GB 增加到 25GB+）
3. **空间验证**: 编译前确认有足够空间（需要 15GB+）

## 验证 (Verification)

合并后，手动触发 workflow 应该会成功，因为：
- ✅ 可以访问 kernel 配置文件
- ✅ 有足够的磁盘空间
- ✅ 使用正确的 kernel 版本 (6.12)
