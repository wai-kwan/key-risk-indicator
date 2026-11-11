---
name: kri
description: Develops quantifiable Key Risk Indicators (KRIs) for enterprise risks, up to 10 per risk per run, leading indicators first, each with Low (Green), Medium (Amber) and High (Red) trigger thresholds, saved to kri_register.csv. Works from a risk register of any filename (default risk_register.csv as produced by the erm skill, or a register from the risk-discovery skill), a partial or non-standard register, or a single risk in the prompt; if parameters are missing it invokes the erm skill's Completion Mode, when available, to complete them first. The user can choose the risks and the number of KRIs, and re-run to add or replace KRIs. Use whenever the user mentions KRIs, key risk indicators, early warning indicators, risk monitoring metrics, risk thresholds or appetite triggers, or wants to monitor a risk or risk register, even if they do not say "KRI".
license: Apache-2.0
compatibility: Any agentskills.io compatible workspace. Designed to consume the register output of the erm skill (https://github.com/wai-kwan/enterprise-risk-management) or the risk-discovery skill (https://github.com/wai-kwan/risk-discovery), or any other register file, and to invoke erm's Completion Mode (erm 1.2.0 or later) when it is available, but does not require either skill.
metadata:
  tags: "enterprise-risk-management, erm, kri, key-risk-indicators, early-warning, risk-monitoring, risk-management, ai-agents, skills, ai-agents-skill"
  written_by: wai-kwan
  github: "https://github.com/wai-kwan"
  version: "1.2.0"
  domain: Enterprise Risk Management, Key Risk Indicators, KRI, Risk Monitoring, AI Agents Skill
  standard: Enterprise Risk Management Workflow Specification
---

# Key Risk Indicator (KRI) Skill

This skill designs measurable indicators, with traffic light thresholds, that warn the organisation early that a risk is moving towards materialising. It is the companion of the **erm** skill (https://github.com/wai-kwan/enterprise-risk-management): erm identifies and rates the risks, this skill says what to measure to know a risk is starting to go wrong.

## Companion Skills
Both companions share the erm 14 column register format. Neither is required.
- **erm** (https://github.com/wai-kwan/enterprise-risk-management): identifies and rates risks, and its **Completion Mode** (erm 1.2.0 or later) completes a partial register. This skill uses it to fill missing risk parameters (see "Completing Missing Risk Parameters with the erm Skill").
- **risk-discovery** (https://github.com/wai-kwan/risk-discovery): discovers risks through a facilitated workshop or interview and saves them as an erm format register, which this skill reads like any other register. A rank only session leaves the rating columns empty, so erm proposes the ratings before KRIs are built.

## When to Use This Skill
- The user has a risk register (the erm skill's `risk_register.csv` or any other file), full, partial or in a different format, and wants KRIs.
- The user describes one risk in the prompt and wants KRIs, with no file.
- The user wants early warning indicators or Green/Amber/Red triggers for a risk, or more, different or redone KRIs after an earlier run.

---

## Step 0: Work Out What the User Wants
Settle four things from the prompt and workspace. Ask only when something essential is missing and cannot be reasonably assumed; otherwise state your assumption and proceed.
1. **Input source** (see "Accepted Inputs"). If the register is missing or incomplete, decide whether the erm skill can complete it.
2. **Scope**: which risks (see "Selecting Risks").
3. **KRI count** (see "Number of KRIs").
4. **Run mode**: first run, add more, or replace (see "Re-running the Skill"). Check whether `kri_register.csv` already exists.

---

## Accepted Inputs
Any of the following works. The full erm register is preferred because its Causes and Effects make KRIs more precise, but it is never required.

### Which file to read
- **Default**: `risk_register.csv` in the active workspace directory (where erm saves it).
- **User specified**: any other file the user names (for example `supply_risks.csv` or an uploaded spreadsheet) or a path. Use exactly that file and do not substitute `risk_register.csv`.
- **Pasted text**: save it to a temporary working file and treat it as a user register with no original file name (new files then use the base name `risk_register`).
- **Never rename, move, overwrite or edit the user's original register file.** New or updated registers are always written to new files.
- Read files with a real parser (Python's `csv` module, or an Excel reader), never by splitting on commas or newlines, because erm registers hold physical line breaks inside quoted Causes, Effects and Controls cells.

### A. Full erm risk register (preferred)
The 14 column layout: `Risk ID, Category, Risk Title, Description, Causes (Drivers), Effects (Consequences), Likelihood Scale (1-5), Likelihood, Impact Scale (1-5), Impact, Risk Score, Risk Criticality, Preventive Controls, Reductive Controls`

### B. Partial or non-standard register
Columns may be missing, renamed, reordered or blank, or the structure may differ altogether (a plain list of risks, a risk matrix export, a table in a document). Do not reject it and do not ask the user to reformat it. Instead:
1. **Extract the in-scope risks**: one row or entry per risk, then apply the user's selection.
2. **Extract available parameters**, mapping columns by meaning and not by exact header (for example "Threat", "Risk Name" or "Risk Event" is the title, "Root cause" or "Drivers" is Causes, "Consequence" is Effects, "Probability" is likelihood, "Owner" is ignored). Keep supplied values as they are and ignore columns that cannot be mapped.
3. **Proceed with what you have.** The only hard minimum is a **risk description** per risk. A Risk Title alone is accepted as the description, and a description with no title gets a concise title written from a risk perspective.
4. **Fill the gaps**, by invoking the erm skill when available, otherwise by inference:
   - **Risk ID missing**: assign R1, R2, ... in the order given (or the next unused ID if `kri_register.csv` exists).
   - **Causes or Effects missing, or only one given**: obtain them from erm, which aims for 2 to 3 of each and keeps the user's own items first. Only if erm is unavailable, infer the most plausible ones in the same way (2 to 3 per risk, never more than 3, keeping any the user gave) and mark the inferred ones in "Linked Cause or Effect" (for example `Inferred: lack of AI talent`).
   - **Risk Criticality missing**: erm supplies ratings if available. Otherwise, if Likelihood Scale and Impact Scale are present, compute score = likelihood x impact and map it (20 to 25 CRITICAL, 10 to 19 HIGH, 5 to 9 MODERATE, 1 to 4 LOW); if only a Risk Score exists, map that; if nothing exists, use `Not rated` and the MODERATE sizing band.
   - **Controls missing**: ignore them.

### C. A single risk (or a few) described in the prompt, no file
Treat the sentence as the Risk Description (derive a title), obtain Causes, Effects and ratings from erm when available or infer Causes and Effects as in B. Assign Risk ID `R1` (or the user's ID, or the next unused ID if `kri_register.csv` exists) and `Not rated` criticality unless the user states one. Do not demand a file and do not ask the user to run erm first: invoke it yourself when available.

### If nothing usable is available
With no file, pasted data or described risk, ask for a risk description or a register. Never invent risks.

State in the closing message which input type and file were used and what was inferred or obtained from erm.

---

## Completing Missing Risk Parameters with the erm Skill
When the register is not available or incomplete, use the erm skill's **Completion Mode** to fill the gaps before generating KRIs.

**When it applies:** the input is type B or C, or any in-scope risk lacks Causes, Effects, or a rating (Likelihood, Impact, Risk Score or Risk Criticality), or has only one Cause or one Effect. A register missing only Controls or Category is complete enough. This includes rank only registers from the risk-discovery skill, whose rating columns are empty. Skip this step if the user says not to use erm, or if the in-scope risks were already completed in an earlier run (reuse that register, see below).

**Naming the new register.** Write the completed register to a new file: the original file name without extension, plus `_kri`, plus `.csv` (`risk_register.csv` becomes `risk_register_kri.csv`, `Q3_risks.xlsx` becomes `Q3_risks_kri.csv`). With no original file the base name is `risk_register`. If that `_kri` file already exists from an earlier run, use it as the working register for that original (original values plus erm completed fields) and update it only for risks still missing parameters.

1. **Check availability.** Look for a skill named `erm` among the session's skills, or its `SKILL.md` in the usual skill folders (for example `/mnt/skills/*/erm/`, `~/.claude/skills/erm/`, `.claude/skills/erm/`, the workspace). Do not download or install it unless asked.
2. **If available, invoke its Completion Mode** by reading its `SKILL.md` and following the "Completion Mode" section:
   - Input: the in-scope risks from the user's register (or the existing `_kri` working register), or the risk described in the prompt. Pass any context known about the organisation or project. If erm pauses to ask for clarification, answer from the user's description rather than stopping.
   - Output: tell erm to save the completed register as `<original name>_kri.csv` in the active workspace directory. This file is this skill's working register, so it may be updated on later runs. It is never the user's original.
   - erm keeps the user's values and Risk IDs, fills the empty fields, adds Causes or Effects after the user's own where only one was given, recalculates Risk Score and Risk Criticality, and leaves out columns that do not belong in the register. It never writes to the input file. Afterwards confirm that the user's original still has the same name and contents, and delete any temporary files you created (for example for pasted text).
3. **Use erm's register as it is.** Read it with a CSV parser and do not re-merge it with the original: erm has already kept every user value. In particular keep the Causes and Effects erm added after the user's own, since every Cause needs a leading KRI. Keep erm's report of which fields it generated or added for the closing message.
4. **Generate the KRIs** from the completed register as for type A. Causes and Effects from erm do not get the `Inferred:` prefix, because they come from erm's assessment rather than a guess.
5. **If erm is unavailable**, fails, or is declined, carry on as in types B and C: infer, mark `Inferred:`, use `Not rated`. No new register file is written. Do not stop.
6. **If the installed erm is older than 1.2.0** (its `SKILL.md` has no "Completion Mode" section), use its standard workflow instead: pass the user's risks as the Risk Context with `N` equal to the number of risks needing completion so it invents none, run it in a scratch directory so its `risk_register.csv` cannot overwrite a user file, then merge its output into the user's risks by title or ID, filling only the missing fields, and write `<original name>_kri.csv`. Suggest updating erm to get Completion Mode.

In the closing message say whether erm was used and name the `_kri` file. List the risks where erm generated or added Causes, Effects or ratings, and say that erm's ratings and added content are proposals for the risk owner to review. If erm was unavailable, add one line that installing the erm skill (https://github.com/wai-kwan/enterprise-risk-management) lets this skill generate fuller, rated risk parameters automatically.

---

## Selecting Risks
- **Default**: all in-scope risks, meaning every risk in the supplied register or prompt.
- The user may narrow by Risk ID (`R1`, `R2 and R5`, `R1 to R5`), title or keyword, Category (Strategic, Operational, Financial), Risk Criticality (for example "CRITICAL and HIGH only"), or "top N risks" (register order, which erm sorts by score).
- If a selection matches nothing (for example an unknown Risk ID), end and inform the user that the selection matched nothing and cannot proceed.
- Never generate KRIs outside the selected scope, and never delete existing KRIs of out-of-scope risks in `kri_register.csv`.

---

## Number of KRIs
- **Default** (no number given): use the sizing guide, within the hard limit of **10 KRIs per risk per run**: CRITICAL 7 to 10, HIGH 5 to 8, MODERATE (and `Not rated`) 4 to 6, LOW 3 to 4. Within the range, choose the count that fully covers the risk's Causes and key Effects without padding, and add no near duplicate KRIs just to hit a number.
- **User specified**: use exactly the number asked for each requested risk ("5 KRIs for R2"). One number with several risks means that many per risk; different numbers per risk are fine ("8 for R1 and 4 for R3").
- **Limits**: More than 10 in one run becomes 10, and tell the user they can re-run with the add option for further candidates. State any adjustment in the closing message.

---

## KRI Design Rules

**Leading first.** KRIs exist for early warning, so design leading indicators first.
- **Leading KRI**: measures a condition that tends to precede the risk event, so action is still possible. It maps to a **Cause** or to the health of a **Preventive Control** (for example percentage of systems with patches overdue more than 30 days, open AI engineering vacancies).
- **Lagging KRI**: measures an outcome after the event has begun or occurred. It maps to an **Effect** or a control failure (for example confirmed security incidents in the quarter, revenue loss attributed to FX movements).
- **Highest priority: generate leading indicators as far as possible.** First design every good leading KRI you can find, and make sure every Cause (supplied or inferred) has at least one.
- Lagging KRIs are still acceptable, but only **after the good leading KRIs have been exhausted**. If reaching the requested number would mean padding with weak, duplicate or hard to measure leading indicators, use a meaningful lagging KRI instead (typically one confirming impact on an Effect or signalling that Reductive Controls should be activated). There is no fixed lagging quota.
- If a metric is ambiguous, ask "can the organisation still prevent the event after seeing this movement?" Yes means leading, no means lagging.

**Quantifiable.** Every KRI is a number the organisation can actually collect (count, percentage, ratio, rate, duration, score or currency amount). Reject vague indicators such as "employee morale" unless made measurable (for example "voluntary attrition rate of AI engineers, rolling 12 months"). State the formula or data source, the unit, and the frequency (Daily, Weekly, Monthly, Quarterly or Annually, matching how fast it can change).

**Distinct, actionable and specific.** Each KRI tracks one thing, differs from the others for the same risk, points to a response a risk owner could take when it turns Amber or Red, and uses the register's or prompt's own context (industry, systems, scale) so it is not generic.

## Traffic Light Thresholds
- **Low (Green)**: within acceptable tolerance, routine monitoring only.
- **Medium (Amber)**: drifting towards the limit, investigate and prepare action.
- **High (Red)**: breached, high alert, management action required now.

Rules:
1. Thresholds are numeric and in the same unit as the measurement.
2. The three bands are **mutually exclusive and collectively exhaustive**: every possible value falls in exactly one band, with no gaps or overlap. Put each boundary value in exactly one band using `<`, `<=`, `>` or `>=`.
3. Direction depends on the metric: for "higher is worse" metrics Green is at the low end, for "lower is worse" metrics Green is at the high end. Write the operators explicitly.
4. Calibrate to the risk: a CRITICAL risk warrants tighter bands than a MODERATE one. Use sensible industry practice and the available context. The true risk appetite and baselines are usually unknown, so treat the numbers as starting points and say so in the closing message.
5. No commas inside threshold text, and never start text with a hyphen or minus sign. Express ranges with operators (`> 5% and <= 10%`) or the word "to".

---

## Re-running the Skill
Every run reads any existing `kri_register.csv` before writing, so earlier work is never lost by accident. Pick the mode from the user's words.

### Add mode (default when a `kri_register.csv` exists)
Triggers: "more KRIs", "additional", "other options", "give me 5 more for R2", "I want to choose from a wider set".
- Keep every existing row.
- Generate new KRIs for the selected risks that are **genuinely different** from existing ones (a different metric, data source or angle, not a reworded duplicate). Read the existing KRI names and measurements first and avoid them.
- Continue numbering from the highest existing KRI number for that risk (if `R2.K6` is last, the next is `R2.K7`). Never renumber existing KRIs, since the user may refer to them by ID.
- The limit of 10 applies to each run's batch, so after several runs a risk can hold more than 10 KRIs. This is intended: the file becomes a pool of candidates, and the user can delete the rows they do not want (tell them so).
- Leading KRIs stay the priority. Earlier runs probably used the obvious leading indicators, so look for fresh leading angles first (other data sources, other Causes, control health measures) before turning to lagging ones.

### Replace mode
Triggers: "redo", "regenerate", "start over", "these are not good", "replace the KRIs for R1".
- First copy the current file to `kri_register_previous.csv` (overwriting any older backup) so the user can recover it.
- Remove the existing rows of the selected risks only, then generate a fresh batch numbered from `K1`. Rows of other risks stay untouched.

### Feedback driven re-runs
If the user says what is wrong ("too generic", "thresholds too loose", "I cannot measure these", "need more leading ones", "our attrition baseline is 12%"), apply it in whichever mode fits. If they name KRI IDs to replace, replace only those rows (reusing nothing from the rejected wording) with new IDs in sequence. Use real baselines or appetite numbers the user supplies to set thresholds.

### Other cases
- **Unclear intent** with an existing file: choose Add mode (non destructive).
- **No existing file**: first run, IDs start at `K1`.
- **New risk in a later run** (no match in the file by Risk Title, then Risk ID): give it the next unused Risk ID. If the user's file reuses an ID for a different title, keep the user's file as the source of truth and flag the clash.

---

## KRI IDs and Ordering
- ID format `<Risk ID>.K<n>`, for example `R1.K1`.
- Rows are grouped by Risk ID in register order. Within a risk, a run's batch lists Leading KRIs first, then Lagging. In Add mode new rows follow the risk's existing rows.

---

## Output: `kri_register.csv`
Create or update a standard CSV file named **`kri_register.csv`** in the active workspace directory (or the outputs folder if the environment uses one, and present the file to the user). One row per KRI, exactly these 12 columns:

1. **Risk ID**
2. **Risk Title**
3. **Risk Criticality** (from the register, computed as in type B, or `Not rated`)
4. **KRI ID**
5. **KRI Name**: short title under 10 words.
6. **KRI Description and Measurement**: one sentence on what is tracked and why it signals the risk, then how it is measured (formula or data source, and unit). No line breaks.
7. **Measurement Frequency**: Daily, Weekly, Monthly, Quarterly or Annually.
8. **Indicator Type**: exactly `Leading` or `Lagging`.
9. **Linked Cause or Effect**: the Cause (leading) or Effect (lagging) tracked, shortened in your own words. Prefix `Inferred:` when it was not supplied by the user.
10. **Low (Green) Threshold**
11. **Medium (Amber) Threshold**
12. **High (Red) Threshold**

Formatting rules (same discipline as the erm skill, so the file opens cleanly in a spreadsheet):
- **Zero-whitespace commas**: a single comma and no space after it (`,"text`, never `, "text`).
- **Strict cell encapsulation**: wrap any field containing a comma or double quote entirely in double quotes, but prefer to avoid commas inside cells.
- **No line breaks inside cells**: each KRI is its own row.
- Never start a cell with a hyphen or minus sign.
- Write with a CSV library where possible so quoting is correct, and keep existing rows byte for byte except when replacing.

### Standard Template Layout
A worked example built from the erm sample register is in `references/example_kri_register.csv`. Header row:

```csv
Risk ID,Risk Title,Risk Criticality,KRI ID,KRI Name,KRI Description and Measurement,Measurement Frequency,Indicator Type,Linked Cause or Effect,Low (Green) Threshold,Medium (Amber) Threshold,High (Red) Threshold
```

Sample row:

```csv
R1,Inability to capitalise on the advancement of AI,CRITICAL,R1.K1,AI engineering vacancy rate,"Tracks unfilled AI roles that slow AI delivery. Measured as open AI roles divided by approved AI headcount.",Monthly,Leading,Lack of AI talent,<= 10%,> 10% and <= 20%,> 20%
```

## Self Check Before Replying
Review the written file and fix problems before replying:
- 12 columns per row, header matches the template, formatting rules respected (no leading hyphens, line breaks or spaces after delimiter commas).
- Each risk's new batch follows the sizing guide, or has exactly the number requested (up to 10).
- Indicator Type is only Leading or Lagging, leading KRIs were prioritised (lagging only because good leading ones were exhausted), and every supplied or inferred Cause has a leading KRI.
- Every threshold set is numeric, non overlapping, gap free, and points the right way for the metric.
- KRI IDs are unique and in sequence, and no existing ID was renumbered.
- If erm was invoked: the original register is unchanged (same name and contents), the new register is named `<original name>_kri.csv`, user values were not changed (erm may have added Causes or Effects after the user's own, never in place of them), the Causes and Effects erm added were kept and each has a leading KRI, and no scratch or temporary files remain.
- Add mode: no new KRI duplicates an existing one. Replace mode: the backup exists and out of scope rows are unchanged.

## Final Reply to the User
Keep it short and do not paste the full table. State:
- where `kri_register.csv` is, which input type was used, and whether erm was invoked to complete the register (and if so, which risks it completed and that its ratings and added content are proposals to review);
- how many risks and KRIs were generated in this run, the leading to lagging split, and the total in the file;
- assumptions, inferred causes and effects (and, if erm was unavailable, the line suggesting it), and any adjustment to the requested count;
- a reminder that thresholds are starting points to calibrate against their risk appetite and historical data;
- a re-run hint, for example "Say 'give me 5 more for R2' to add alternatives or 'redo R1' to replace them."
