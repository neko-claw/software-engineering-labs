# Lab1 第二部分 Git 使用操作记录

实验手册对应页：
- 第 21 页：场景(1) 仓库创建与提交
- 第 22 页：场景(1) 推送到 GitHub 上
- 第 23 页：场景(2) 分支管理
- 第 24 页：场景(3) 在 IDE 中使用 Git 管理程序

本次操作在隔离演示仓库 `out/git-lab-demo` 中完成，避免污染当前正式仓库历史。远程仓库使用 `https://github.com/shizhenneko/Lab1-2023112455`。

## 结果概览

- 已在 `out/git-lab-demo` 中重新初始化本地仓库并完成多次提交、撤销最后一次提交、查看完整历史。
- 已将演示仓库的 `master` 分支推送到 GitHub。
- 已创建 `B1`、`B2`、`C4` 分支，完成两轮冲突合并。
- 已创建并推送学号分支 `2023112455`。
- 已保存命令日志、终端截图和 GitHub 页面截图。

## 对应证据

### 场景(1)：仓库创建与提交
- `R0-R2`：
  - 日志：`logs/01_r0_before_init.txt`、`logs/02_r1_init_and_status.txt`、`logs/03_r2_first_commit.txt`
  - 截图：`screenshots/03_r2_first_commit.png`
- `R3-R5`：
  - 日志：`logs/04_r3_diff_after_first_edit.txt`、`logs/05_r4_second_commit.txt`、`logs/06_r5_third_commit.txt`
  - 截图：`screenshots/04_r3_diff_after_first_edit.png`
- `R6-R7`：
  - 日志：`logs/07_r6_undo_last_commit.txt`、`logs/08_r7_full_history_after_reset.txt`
  - 截图：`screenshots/07_r6_undo_last_commit.png`

### 场景(1)：推送到 GitHub 上
- `R8`：
  - 日志：`logs/10_r8_remote_setup.txt`
  - 截图：`screenshots/10_r8_remote_setup.png`
- `R9`：
  - 日志：`logs/11_r9_push_master.txt`
  - 截图：`screenshots/11_r9_push_master.png`

### 场景(2)：分支管理
- `R1-R5`：
  - 日志：`logs/12_r1_branch_list_and_checkout_master.txt`、`logs/13_r2_create_b1_b2.txt`、`logs/14_r3_create_c4_from_b2.txt`、`logs/15_r4_commit_on_c4.txt`、`logs/16_r5_commit_on_b1.txt`
- `R6`：
  - 日志：`logs/17_r6_merge_c4_into_b1.txt`
  - 截图：`screenshots/17_r6_merge_c4_into_b1.png`
- `R7-R9`：
  - 日志：`logs/18_r7_commit_on_b2.txt`、`logs/19_r8_branch_merge_status.txt`、`logs/20_r9_merge_unmerged_into_student_branch.txt`
  - 截图：`screenshots/19_r8_branch_merge_status.png`
- `R10-R11`：
  - 日志：`logs/21_r10_push_student_branch.txt`、`logs/22_r11_version_tree.txt`
  - 截图：`screenshots/21_r10_push_student_branch.png`、`screenshots/22_r11_version_tree.png`

### 场景(2)：GitHub Web 页面查看
- `R12`：
  - 页面地址：`https://github.com/shizhenneko/Lab1-2023112455/tree/2023112455`
  - 截图：`screenshots/github-2023112455.png`

## 最终关键对象

- 演示仓库目录：`C:\Users\86159\Desktop\Lab1-2023112455\out\git-lab-demo`
- 证据目录：`C:\Users\86159\Desktop\Lab1-2023112455\out\git-evidence`
- 远程仓库：`https://github.com/shizhenneko/Lab1-2023112455`
- 学号分支：`2023112455`
