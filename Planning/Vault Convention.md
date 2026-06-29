---
type: note
domain: Planning
status: stable
---

# Vault Convention

## Tag Convention

| Tag | 用途 | 典型位置 | 完成后的处理 |
| --- | --- | --- | --- |
| #idea | 值得发展的想法 | Daily Notes、00_Inbox | 提取后可移除 |
| #question | 尚未解决的问题 | Daily Notes、概念笔记 | 解答后移除 |
| #needs-review | 需要重新整理或复习 | 普通笔记 | 整理后移除 |
| #needs-verification | 结论或来源需要验证 | 技术、公式、阅读笔记 | 验证后移除 |
| #debug-log | 实际调试记录 | RM 项目笔记 | 通常保留 |
| #retrospective | 阶段、课程或项目复盘 | RM、UG、Planning | 保留 |

## Tag Rules

- 不用标签表达主题。
- 主题用文件夹、MOC、链接、Properties。
- 新标签先确认，不直接创建。
- `#excalidraw` 属于插件标签，不纳入知识管理规则。

## Property Convention

| Property | Values |
| --- | --- |
| type | moc, concept, literature, project, note, daily |
| domain | UG, RM, SelfStudy, Research, Planning |
| status | inbox, developing, stable, archived |

## Weekly Review

1. Check recent Daily Notes: #idea
2. Check recent Daily Notes: #question
3. Check 00_Inbox
4. Pick one:
   - add to existing note
   - extract as note
   - add 1–3 links
   - link to MOC
   - move if clear
   - keep in Inbox
5. Update 1–3 active MOCs
6. Check #needs-verification

## Monthly Maintenance

- check duplicate tags
- list one-use tags
- check old #needs-verification
- check important orphan notes
- check active MOCs
- mark finished courses/projects as archived
- check empty concept pages

