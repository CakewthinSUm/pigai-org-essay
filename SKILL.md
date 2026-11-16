---
name: pigai-org-essay
description: "Write, revise and submit 批改网 (pigai.org) English essays."
metadata:
  short-description: Write and submit pigai.org essays
---

# 批改网 (pigai.org) Essays

Deliver a finished, submitted composition — not just text. The platform is a legacy PHP app whose scripts are fragile, so drive it slowly and verify every step from page state.

## Before touching the editor

- Read the task row first: 作业号, 标题, 类型, 截止时间, and the 要求 text. Word limits are usually 120–180 and are enforced.
- Confirm the letter skeleton. 书信 tasks split into 日期 / 信头 / 正文 / 信尾, and each part is a separately graded field:
  - `#mail_header_ri` — date, e.g. `May 6, 2026`
  - `#mail_header_dear` — the whole salutation, e.g. `Dear <Name>,` (the template does **not** add "Dear" for you)
  - `#contents2` — body textarea, blank lines separate paragraphs
  - `#mail_header_f1` — closing phrase, e.g. `Yours sincerely,`
  - `#mail_header_f2` — signature
- Select the student's real class in `#stu_class` before submitting; a blank class makes the work hard to grade.
- The site counts **body words only** (the 字数 readout matches the body field). Stay inside the stated range with a few words of margin.

## Filling the editor (the part that breaks)

Teachers can enable 禁止粘贴, so clipboard-based input fails:

- Never use Playwright `locator.fill()`, `setValue()`, or any paste path. Here `fill()` raises a modal `您的老师启用了"禁止粘贴"选项` and can wedge the tab.
- Type instead: click the field, `ctrl+a`, `Delete`, then `ax.typeText(null, text)` (or `locator.pressSequentially` for short values).
- Long strings get truncated mid-word. Type paragraph by paragraph and verify exact equality after each chunk; when a tail is missing, click the field, `ctrl+End`, and type only the remainder.
- After a failed interaction a modal alert often appears. Dismiss it via `tab.getJsDialog()`; if the tab stops answering CDP calls (`Emulation.setFocusEmulationEnabled` timeouts), wait 3–10 seconds and retry, and reload only after the text is saved or can be retyped.

## Submit and read the result

- Click `#dafen` (提交作文) and accept the confirm dialog if one appears.
- Success looks like: URL moves to `?c=v2&a=view&eid=...`, 提交次数 increments, and a score appears in `.dfScore`.
- Read the four sub-scores (词汇 / 句子 / 篇章结构 / 内容相关) plus 按句点评 and 体检报告 before revising. Observed behaviour is in [references/scoring-notes.md](references/scoring-notes.md).
- Each resubmission replaces the displayed score, so re-enter the best draft if a revision scores lower.

## Revision loop

1. Fix concrete flags first — grammar warnings such as `has survived → survived`, and 口语表达 such as `let me know → inform me of`.
2. Then tune for the four metrics. Content relevance responds to echoing the prompt's own vocabulary in the body (letter, recommend, place, travel, China, visit, three reasons, the addressee's name). Discourse connectives lift 篇章结构. Long, clause-rich sentences score better than many short ones.
3. Resubmit, compare the total, and stop when the target is met or two consecutive attempts fail to improve.

Submission is a graded external action. If the user has not plainly asked you to submit, show the draft and get their go-ahead first.

## Browser mechanics

Use the Chrome binding and claim the existing 批改网 tab; setup, selectors, and snippets are in [references/browser-playbook.md](references/browser-playbook.md).
