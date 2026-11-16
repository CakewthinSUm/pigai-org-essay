# Scoring Notes and Draft Template

## What the four metrics respond to

Observed on one 书信 task (满分 100, 120–180 words). Same prompt throughout; only the drafting
strategy changed:

| Attempt | 词汇 | 句子 | 篇章结构 | 内容相关 | Total |
| --- | --- | --- | --- | --- | --- |
| Long complex sentences, generic wording | 0.952 | 0.818 | 0.864 | 0.465 | 85.5 |
| Split into 13 short sentences | — | lower | — | — | 83 |
| Same idea, even shorter sentences | — | lower | — | — | 81 |
| Added the addressee's name + one formal verb | 0.955 | 0.783 | 0.863 | 0.609 | 85.0 |
| Balanced long sentences + name + "friend" | 0.958 | 0.826 | 0.840 | 0.630 | 86.5 |
| More connectors, "foreign friend", no name | 0.955 | 0.842 | 0.810 | 0.434 | 85.0 |

Takeaways:

- 句子 rewards long, clause-rich sentences. Shortening sentences to hit the 体检报告 `句子数 11~17`
  band *lowered* the total.
- 内容相关 tracks prompt vocabulary far more than plot quality. Putting the addressee's name and the
  word "friend" into the body produced the largest single jump (+0.14).
- 篇章结构 moves with explicit discourse connectives, but reacts weakly compared with 内容相关.
- 口语表达 flags (`let me know`) cost little and are easy to remove: `inform me of`, `send me`.
- 体检报告 reference bands are advisory. 形符数 195–230, 类符数 93–115, 段落 3–5 are safe targets;
  chasing 句子数 at the cost of sentence length backfires.

## Letter skeleton (structure only — write your own body)

The shape that scored best, with the graded fields spelled out:

```text
<Month> <Day>, <Year>            ← date field
Dear <Name>,                     ← salutation field; the template does NOT add "Dear"

Paragraph 1: why I am writing + why this subject, naming the addressee once.
Paragraph 2: the body — three reasons for the recommendation, each carried by one
             relative clause ("which has…", "whose…"), not by a bulleted list.
Paragraph 3: warm closing that addresses the person by name again + one forward-looking detail.

Yours sincerely,                 ← closing phrase field
<Signature>                      ← signature field (your own name)
```

Reusable structure: salutation → why I am writing + why this subject → three reasons, each with a
relative clause → warm closing that names the addressee → sign-off.

Keep the graded fields distinct: 日期 / 信头 / 正文 / 信尾 / 签名 are five separate inputs, and a
paragraph pasted into the salutation field will be penalised even if the text is correct.

## Practical limits

A 90+ score was not reachable in ten attempts on this task; 内容相关 stayed near 0.63 while the other
metrics sat above 0.83. Say so plainly instead of promising a number, and stop when two consecutive
attempts fail to improve.
