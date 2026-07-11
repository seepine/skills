# 完整工作流示例

```bash
# === 第一步:创建 ===
git status --short                                    # 检查未提交代码
git checkout main && git pull origin main             # 切到 main 同步
git worktree add -b 20260101/fix-password-schema ../myproject-20260101-fix-password-schema
cd ../myproject-20260101-fix-password-schema          # 进入新 worktree 开发

# === 开发完成后 ===
git add . && git commit -m "fix: fix password schema"
git push origin 20260101/fix-password-schema


# === 第二步:合并 ===
cd ../myproject                                       # 回到主 worktree
git branch -vv                                        # 检查远程分支

# a. 若使用PR方式（若非 github 而是 gitea，则尝试使用 giteacli）
gh pr create --base main --head 20260101/fix-password-schema
# b. 直接合并方式
git merge --no-ff 20260101/fix-password-schema


# === 第三步:清理 ===
cd ../myproject
git worktree remove ../myproject-20260101-fix-password-schema
git worktree prune
git branch -d 20260101/fix-password-schema
git worktree list                                    # 验证
```
