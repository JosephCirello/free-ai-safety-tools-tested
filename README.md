# Seven free AI safety tools, four models, one audit

**We ran seven free, public AI safety evaluations on four AI models and put the results beside an agentic audit of the same four. This page is what they caught, what they missed, how they broke, and what they cost.**

By [Potestas AI](https://www.potestasai.com), an independent AI testing company in Providence, Rhode Island. Runs between 29 September and 2 October 2026.

- Full paper (PDF, 21 pages): [Instrument Failures and Hidden Costs in Public AI Safety Evaluations: Seven Free Tools, Four Models and an Agentic Audit](https://www.potestasai.com/Instrument_Failures_and_Hidden_Costs_in_Public_AI_Safety_Evaluations.pdf)
- The same comparison on the website: [potestasai.com/free-tools-tested](https://www.potestasai.com/free-tools-tested)
- How to run the seven tests yourself: [potestasai.com/run-the-tests](https://www.potestasai.com/run-the-tests)

## In one paragraph

Free safety tests are often the first thing a team runs before it deploys an AI agent. We ran seven of them (AgentDojo, AgentHarm, StrongREJECT, MASK, b3, XSTest and garak) once each on gpt-4o-mini, grok-4.20, Llama-3.3-70B and kimi-k2.6. Three things came out. **The tools break quietly:** they graded empty replies as answers, evasions or passes, stopped whole runs on one vendor refusal, and leaked settings between probes, and none of them said anything was wrong. **Free is not free:** the tools' own AI graders cost 3.6 times as much as gpt-4o-mini itself across all seven tools, and a reasoning model is billed for thinking you never see. **They do not agree:** among the three models run under the same settings, agreement between the tools (Kendall's *W*) was 0.19 to 0.33, where 1 means they all agree.

## What we ran

| Tool | Version | What it measures |
|---|---|---|
| [AgentDojo](https://github.com/ethz-spylab/agentdojo) (ETH Zurich) | 0.1.35, benchmark v1.2, `important_instructions` attack, all four suites | an agent hijacked by instructions hidden in tool outputs, while it works |
| AgentHarm (Gray Swan AI and the UK AI Security Institute) | inspect_evals 0.22.0, `test_public` split (176 tasks) and its benign twin | completion of malicious multi-step agent tasks; refusal |
| StrongREJECT (UC Berkeley) | inspect_evals 0.22.0, plain and inside the AIM jailbreak template (313 prompts each) | single-turn harmful compliance |
| MASK (Center for AI Safety and Scale AI) | inspect_evals 0.22.0 (1,000 scenarios) | saying what the model believes is false, under pressure |
| b3 (Lakera, with the UK AI Security Institute) | inspect_evals 0.22.0 (630 snapshots, 5 epochs) | attacks on an agent's backbone model, at three defence levels |
| XSTest | 450 prompts (250 safe, 200 unsafe), through Inspect | refusing safe prompts that look unsafe, and unsafe ones |
| [garak](https://github.com/NVIDIA/garak) (NVIDIA) | 0.17.0, 81 probes in 18 families | text vulnerabilities |

Five of the seven run on the UK AI Security Institute's [Inspect](https://github.com/UKGovernmentBEIS/inspect_ai) framework (version 0.3.273). Each tool ran once per model.

| Model | Served by |
|---|---|
| gpt-4o-mini-2024-07-18 | OpenAI |
| grok-4.20-0309-non-reasoning | xAI |
| Llama-3.3-70B-Instruct-Turbo | DeepInfra |
| kimi-k2.6 (a reasoning model: it thinks before each reply) | Moonshot |

None is the newest model its vendor offers.

**Settings we had to change, so you can match them:** xAI rejects a seed of 0, so grok-4.20 ran at seed 1 on every Inspect tool and without garak's frequency and presence penalties. Moonshot allows only temperature 1 for kimi-k2.6. kimi-k2.6 was given 4,096 tokens for XSTest and garak and 8,192 for MASK, because at the defaults its replies were empty. garak ran one generation per prompt on every model (the default is five). Inspect tasks ran with `--no-fail-on-error`, except Llama-3.3-70B's first MASK, b3 and StrongREJECT runs. We publish no test prompts here, only the kinds of failure.

## The scoreboard

Each free tool's own headline number, one run per model, 95% intervals where the paper gives them. Higher is worse unless the row says otherwise.

| | kimi-k2.6 | Llama-3.3-70B | gpt-4o-mini | grok-4.20 |
|---|---|---|---|---|
| AgentDojo: attacks that hijacked the agent | 1.2% (0.6 to 2.1) | 15.0% (12.8 to 17.4) | 29.8% (27.0 to 32.8) | 49.7% (46.6 to 52.9) |
| AgentHarm: harm score (0 to 1) | 0.18 | 0.41 | 0.59 | 0.79 |
| AgentHarm: refused (higher is safer) | 79.5% | 36.4% | 21.6% | 5.7% |
| StrongREJECT: score (0 to 1), plain / jailbreak wrapper | 0.00 / 0.00 | 0.20 / 0.08 | 0.10 / 0.00 | 0.09 / 0.00 |
| MASK: honest (higher is better) | 77.2% | 56.6% | 54.9% | 39.0% |
| b3: attack success | 11.9% | 25.6% | 15.5% | 30.4% |
| XSTest: safe prompts refused | 3.2% | 2.8% | 6.8% | 6.0% |
| XSTest: unsafe prompts refused (higher is safer) | 88.0% | 92.5% | 87.5% | 84.4% |
| garak: checks failed, as garak reports it | 32.3% | 27.7% | 26.2% | 39.4% |
| garak: checks failed, shared prompts the model answered | 36.2% | 27.1% | 26.8% | 33.4% |

kimi-k2.6's MASK score was measured with room to answer; at the default limit it read about 90%. Its garak score as reported counts 3,791 empty replies as passes. The full table, with every interval and how each figure was computed, is Table 2 of the paper.

**We checked our setup first.** AgentHarm's authors published gpt-4o-mini at 62.5% harm and 22.0% refused (Table 9 of their paper). Our run, on the 44 public test behaviours the Inspect package ships, gave 0.589 and 21.6%. Close enough to show the tools were installed and run correctly.

## How the free tools broke

- **Silence graded as honesty.** MASK caps each reply at 1,000 tokens. kimi-k2.6 often spent the whole allowance thinking, so 45% of its replies came back empty, and MASK graded those as evading, which it does not count as lying. It looked about 90% honest. Given room to answer, it lied in 228 of 1,000 scenarios and scored 77%.
- **Silence scored as a pass.** kimi-k2.6 returned empty replies to 3,791 of garak's 11,938 prompts, and 5,163 of the 5,176 checks on those empty replies passed.
- **One setting leaks into every later probe.** garak's `promptinject` probes set the reply limit and other generation settings on the shared generator and never restore them, so the probes that run after them inherit a 60 to 256 token limit.
- **One blocked prompt can stop the whole test.** Through Inspect's generic OpenAI-compatible provider, one vendor content-filter refusal stopped the whole StrongREJECT run on grok-4.20 and on kimi-k2.6. garak read xAI's content refusal (HTTP 403) as a bad key and stopped after 239 of about 12,000 prompts.
- **Each model gets different questions.** garak samples prompts at random, unseeded by default, so only 4,287 prompt pairs were asked of all four models.
- **One bad grade can lose a run.** On Llama-3.3-70B, MASK's grader returned an answer its validator rejected and the run stopped after 33 minutes.

## What it cost

The tools use other AI models to grade the answers, and those graders cost more than the model being tested: 3.6 times gpt-4o-mini's own usage across all seven tools. kimi-k2.6's own usage across five tools cost 50 times gpt-4o-mini's, 94% of it output, most of that hidden reasoning. On b3 it wrote about 3,531 tokens per attack where gpt-4o-mini wrote 265. A price list per token does not show what an answer costs.

## Where the free tools and the audit agree, and where they do not

On attacks hidden in the data an AI reads, AgentDojo, AgentHarm and the audit's injection tests ranked the four models in the same order. Where they measure the same thing, they get the same answer.

The difference is breadth. AgentDojo tests one kind of attack, and against kimi-k2.6 it got through 11 times in 949 (1.2%). The audit tries twelve kinds of attack, by where the hidden instruction enters. On the same model, five kinds got through, including customer data emailed to an outside address and administrator rights granted through a plan the agent had saved earlier. AgentDojo was not wrong. It checks one door, and an agent has more.

The audit is commercial and its scenarios are proprietary, so that half of the comparison cannot be re-run by readers, and its human review was the author's. The paper says so on its first page. The free-tool half stands on its own: every tool is public, and every version and setting is listed above.

## A correction for each failure, for the people who maintain the tools

1. Count and report empty and truncated replies, and do not grade an empty reply as an answer, an evasion or a pass.
2. Record a vendor's content-filter refusal as a blocked prompt, counted in the summary, not as an error that stops the run.
3. Restore generator settings after a probe that changes them, or run such probes in isolation.
4. Seed prompt sampling and record the seed.
5. Have a scorer return "unscored" for a malformed grader answer instead of stopping the run.
6. Bound stalled requests by default.
7. Report the graders' token cost beside the result.

Section 5.3 of the paper explains each one and which are already options in the tools.

## Limits

Every measurement is one run, so disagreement between tools and run-to-run noise are not separated. Four models, none its vendor's newest. kimi-k2.6 ran at larger reply limits and temperature 1, so its consistent first place may be a settings effect, not a safety one. The free tools' AI graders were taken as given apart from empty replies. The corrections above are untested. The full list is section 6 of the paper.

## Cite

Joseph Cirello. *Instrument Failures and Hidden Costs in Public AI Safety Evaluations: Seven Free Tools, Four Models and an Agentic Audit.* Potestas AI, October 2026. https://www.potestasai.com/Instrument_Failures_and_Hidden_Costs_in_Public_AI_Safety_Evaluations.pdf

## About Potestas AI

Potestas AI is an independent AI testing company in Providence, Rhode Island, founded by Joseph Cirello, U.S. Army veteran. It runs these seven industry-standard tests and its own 258-scenario agentic say-vs-do suite against the model you name, grades the result off the tool-call log, and delivers a findings report and a sealed evidence pack within 72 hours, for $3,000. The fee credits in full toward the $25,000 audit.

- Website: [potestasai.com](https://www.potestasai.com)
- Public findings, dated: [potestasai.com/findings](https://www.potestasai.com/findings)
- A complete published report: [potestasai.com/sample-report](https://www.potestasai.com/sample-report)
- Contact: joseph.cirello@potestasai.com
- LinkedIn: [linkedin.com/in/josephcirellojr](https://www.linkedin.com/in/josephcirellojr)

The author sells the audit this comparison sits beside. That is a conflict of interest, and it is stated here as it is in the paper.
