# pigai-org-essay

**English** | [简体中文](README.zh-CN.md)

An **agent skill** for completing, revising and submitting English writing assignments on
**批改网 (pigai.org)** — a legacy PHP app that grades every submission instantly and shows four
sub-scores plus sentence-by-sentence feedback. The skill turns that feedback loop into a procedure
that actually survives the site.

## What makes this site hard

- **The editor refuses paste.** When the teacher enables 禁止粘贴, `locator.fill()` triggers a modal
  (`您的老师启用了"禁止粘贴"选项`) and can leave the tab unresponsive — every subsequent CDP call times
  out on `Emulation.setFocusEmulationEnabled`. The only reliable input path is real keystrokes
  (`Input.insertText` / `pressSequentially`), chunked, verified field-by-field.
- **Long typing gets truncated mid-word.** Type paragraph by paragraph and compare the field value to
  the intended string after each chunk; when the tail is missing, `ctrl+End` and type the remainder.
- **The scored unit is not one box.** 书信 tasks split into five separately graded fields — 日期,
  信头, 正文, 信尾, 签名 — and the template's placeholders (`Month , Year`, `Dear        ,`) are hints,
  not auto-fill. "Dear" is *not* inserted for you.
- **Only the body counts toward 字数**, so a correct-looking page can still be under the limit.
- **Resubmission replaces the displayed score**, which makes the revision loop a real search over
  drafts rather than an accumulation.

## Workflow

1. Read the task row (作业号, 标题, 类型, 截止时间, 要求) and the word limit (usually 120–180, enforced).
2. Confirm which fields the task requires; fill 日期 / 信头 / 正文 / 信尾 / 签名, and select the class in
   `#stu_class` — a blank class makes the work hard to grade.
3. Type (never paste), verifying each field's exact value.
4. Submit (`#dafen`), accept the confirm dialog, confirm the URL moves to `?c=v2&a=view&eid=…`,
   提交次数 incremented, and `.dfScore` shows a number.
5. Read 词汇 / 句子 / 篇章结构 / 内容相关 plus 按句点评 and 体检报告, then revise against the
   measured sensitivities in [references/scoring-notes.md](references/scoring-notes.md): prompt
   vocabulary drives 内容相关 hardest, clause-rich sentences beat short ones, connectives move
   篇章结构 weakly.
6. Resubmit; stop at the target or after two attempts that fail to improve — and re-enter the best
   draft, because the last submission wins.

## Evidence, not vibes

One task, ten attempts, 100-point scale: the best result (86.5) came from balanced long sentences
plus the addressee's name and the word "friend"; splitting into 13 short sentences scored 83 and an
even shorter version 81. 内容相关 swung 0.43 → 0.63 on prompt vocabulary alone — the largest single
lever on the page. A 90+ was not reachable on that task; the skill says so instead of promising a
number. Full table: `references/scoring-notes.md`.

## Layout

| Path | What |
|---|---|
| `SKILL.md` | The procedure: field map, anti-paste typing, submit-and-read, revision loop |
| `references/browser-playbook.md` | Tab claiming, field selectors, typing helper, failure modes + CDP mapping |
| `references/scoring-notes.md` | Measured metric behaviour, letter skeleton, practical score ceiling |
| `agents/openai.yaml` | Optional Codex/ChatGPT skill metadata |

## Boundaries

- **Submission is an irreversible, graded external action.** The skill shows the draft and waits for
  an explicit go-ahead; it never submits on its own initiative.
- It writes the assignment; it does not invent experiences, citations or sources the author didn't
  have.
- Respect your course's rules on AI assistance and on resubmission counts — some teachers cap 提交次数,
  and some require the work to be unaided. Read the task page and your syllabus before using this.

## Requirements

Any agent that can attach to your **own signed-in Chrome** (this skill assumes the tab is already
logged in; it never types credentials) — either through a browser-tool API like the one in the
snippets, or over CDP/Playwright using the mapping table in `references/browser-playbook.md`.

## Install

```bash
git clone https://github.com/CakewthinSUm/pigai-org-essay.git \
  ~/.hermes/skills/productivity/pigai-org-essay      # Hermes Agent
# other agents: ~/.claude/skills/, $HOME/.agents/skills/, .opencode/skills/ …
```

Then: *「帮我把批改网那篇作文交了，尽量到 87 分」* / *"finish and submit the pigai.org essay."*

## Status

Selectors and behaviours come from real submissions on the site. The snippets are written against a
`tab.ax` / `tab.playwright` browser surface from one agent toolkit; the CDP/Playwright equivalents are
documented in the playbook but have not been re-run end to end by the maintainer — treat that path as
documented, not tested.

## License

MIT — see [LICENSE](LICENSE).
