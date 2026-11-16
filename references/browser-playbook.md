# Browser Playbook — pigai.org

The snippets below are written against a browser tool that exposes `agent.browsers.get("chrome")`,
`chrome.user.openTabs()` / `claimTab()` / `nameSession()`, and per-tab `tab.ax` (`get` / `click` /
`typeText` / `pressKey` / `write`) plus `tab.playwright` and `tab.getJsDialog()`. They are the
reference implementation, not a portable one.

On a raw-CDP or Playwright stack, the same steps are:

| Snippet call | CDP / Playwright equivalent |
| --- | --- |
| claim the existing tab | attach to your Chrome (`--remote-debugging-port`) via `Target.getTargets` + `Target.attachToTarget` |
| `tab.playwright.locator(sel).click()` | `Runtime.evaluate` with `document.querySelector(sel).click()`, or a coordinate click via `Input.dispatchMouseEvent` |
| `tab.ax.typeText(null, text)` | focus the field, then `Input.insertText` (single shot) or `Input.dispatchKeyEvent` per key |
| `tab.ax.pressKey(null, "ctrl+a")` | `Input.dispatchKeyEvent` with `modifiers: 2` |
| `tab.ax.get("state")` | `Accessibility.getFullAXTree` (or just read `document.body.innerText` — this site's labels are plain text) |
| `tab.getJsDialog()` | `Page.javascriptDialogOpening` event → `Page.handleJavaScriptDialog` |
| `tab.playwright.evaluate(fn, arg)` | `Runtime.evaluate` with a serialised expression |

Everything below that is site knowledge — selectors, field semantics, the typing dance — transfers
unchanged.

## 1. Find or open the assignment

```js
const tabs = await chrome.user.openTabs();
const row = tabs.find(t => /pigai\.org/.test(t.url || ""));
const tab = row ? await chrome.user.claimTab(row) : await chrome.tabs.new();
if (!row) await tab.goto("https://www.pigai.org/index.php?c=write&a=waitComplete");
await tab.ax.write();
```

The unfinished list is `?c=write&a=waitComplete`. Each row's 查看 link is `index.php?c=v2&a=write&rid=<rid>`; that page exposes the requirement text and the fields below.

## 2. Field map

| Field | Selector | Content |
| --- | --- | --- |
| 日期 | `#mail_header_ri` | `May 6, 2026` |
| 信头 | `#mail_header_dear` | `Dear <Name>,` |
| 正文 | `#contents2` | essay body, `\n\n` between paragraphs |
| 信尾 | `#mail_header_f1` | `Yours sincerely,` |
| 签名 | `#mail_header_f2` | student name |
| 班级 | `#stu_class` | select by visible label |
| 提交 | `#dafen` | submits and grades |

Placeholders are hints, not auto-fill: 日期 shows `Month , Year` and 信头 shows `Dear        ,`, yet nothing is inserted for you.

## 3. Typing helper

```js
async function typeInto(tab, sel, text) {
  await tab.playwright.locator(sel).click({ timeoutMs: 8000 });
  await tab.ax.pressKey(null, "ctrl+a");
  await tab.ax.pressKey(null, "Delete");
  const chunks = text.split("\n\n");
  for (const [i, chunk] of chunks.entries()) {
    await tab.ax.typeText(null, i === 0 ? chunk : "\n\n" + chunk);
    await new Promise(r => setTimeout(r, 250));
  }
  return tab.playwright.evaluate(s => document.querySelector(s).value, sel);
}
```

Verify equality against the intended string. A value ending mid-word means keystrokes were cut off; finish with:

```js
await tab.playwright.locator("#contents2").click({ timeoutMs: 8000 });
await tab.ax.pressKey(null, "ctrl+End");
await tab.ax.typeText(null, remainingTail);
```

Do not use `locator.fill()` anywhere on this site.

## 4. Submit and score

```js
await tab.playwright.locator("#dafen").click({ timeoutMs: 10000 });
await new Promise(r => setTimeout(r, 2500));
const dlg = await tab.getJsDialog();
if (dlg?.type === "confirm") await dlg.accept();
if (dlg?.type === "alert") await dlg.dismiss();
await new Promise(r => setTimeout(r, 3000));
```

Read the result without parsing SVG:

```js
const ax = await tab.ax.get("state", { disableDiffing: true });
ax.split("\n")
  .filter(l => /词汇: 0|句子: 0|篇章结构: 0|内容相关: 0|text 8\d/.test(l))
  .join("\n");
```

The total is in `.dfScore` (`#stuScore`, `.circles-integer`); the four metrics are the `container` labels inside `.i_pm`.

## 5. Failure modes seen in practice

- **Modal loop / dead tab.** `ax.write()`, `screenshot()`, and `reload()` all time out on `Emulation.setFocusEmulationEnabled`. Cause is an open `alert()` from the anti-paste script. Wait several seconds, retry, then `getJsDialog()` and dismiss. Typed text survives; reload only if it does not.
- **Auto-save countdown.** The bar shows `NNN 秒后保存` and `保存数据` (`#save_bt`). Do not rely on it — verify field values before submitting.
- **Evaluation timeout.** Long `typeText` calls can exceed the default 30 s; pass `timeout_ms` (60–120 s) to the `js` tool for typing and submission calls.
- **Class dropdown.** Only the student's own classes plus 其他 are listed; pick the class matching the task's 指定班级.
