# Project roadside NO2 from the monitor to the building facade

A harbor-format benchmark task, authored for the AfterQuery Experts programme as data
annotation for AI evaluation. Task slug `afterquery/project-facade-no2` — **Science / Air Quality**.

An AI agent gets a container, this brief and nothing else. It works alone for
4 hours. Then a sealed verifier it never sees scores the result `0` or `1`.

## What it asks for

A council's air quality officer is preparing the nitrogen dioxide section of the annual status report from ten continuous monitoring sites, five at the roadside and five at urban background locations. For each site the officer needs the mean concentration over the monitoring period at the monitor and the mean at the nearest building facade, which for a roadside site means applying the fall-off-with-distance adjustment.

The full brief is in [`instruction.md`](instruction.md).

| | |
| --- | --- |
| **Deliverable** | `/app/no2_report.csv` |
| **Agent budget** | 4 hours (14400s) |
| **Expert estimate** | 3 hours |
| **Machine** | 1 CPU, 2048 MB RAM, 0 GPU |
| **Verifier budget** | 300s |
| **Tags** | `air-quality`, `nitrogen-dioxide`, `monitoring-network`, `time-series`, `python` |

## Why it is hard

This is a local-authority air quality officer's or an air quality consultant's job at reporting time: average each continuous analyser's nitrogen dioxide over the period, and for roadside sites carry the mean from the monitor to the nearest facade with the fall-off-with-distance adjustment, which needs the site's kerb distances and mapped background. The data is synthetic and made truth first. Ten inlets - five roadside, five urban background - were given 5376 quarter-hour means over eight weeks: the town's weather over a common daily shape, and at roadside inlets exhaust plumes on top.

<details>
<summary>Full note</summary>

Each sealed line is the team note's own arithmetic on the record really made at that site. The downloads and the site list were written afterwards; nothing is estimated at marking time. Some of the work is ordinary tidying. Three downloads are plainly not as the note describes - one column is in parts per billion, one in milligrams per cubic metre, one file has semicolons and decimal commas - and the headers and values say so at a glance. Calibration and fault rows are flagged and the note says to leave them out. A reading of the note to the letter gets 6 of 10 sites; with the downloads tidied, 8. The rest cannot be got by tidying. The site list says which analyser stands at which site, and for 2 analysers it is out of date: after a service one roadside and one background analyser went back to each other's cabinets. A background record then receives a roadside site's fall-off adjustment and a roadside record receives none, and each of the two sites comes out at least 15.4 per cent wrong while looking entirely reasonable - a quiet road and a busy district have levels of the same kind, and every background record here has the same relative spread as the roads' have, so a summary of a download does not say what sort of inlet made it. What does say it is how a quarter-hour stands to the ones either side. A kerbside inlet catches passing exhaust plumes, so its record is full of single quarter-hours standing well clear of their neighbours; a background inlet's record moves only as the weather moves. Taken as the root-mean-square distance of a reading from the mean of its two neighbours over the spread of the record, the five roadside records come to 0.39 or more and the five background records to 0.15 or less - and one analyser listed at a road is quiet and one listed at a background site is lively. A period mean never compares neighbouring readings, so the reporting arithmetic does not meet this; someone who has looked at roadside and background traces side by side knows to look. The note says the site types come from the consultants' network review, so it is the analyser column that gives way, and with one odd record of each kind the two analysers can only have changed places with each other. tests/town_check.py marks 5 other stances on the downloads against the sealed means and asserts that the reference gets 10 and none of the others more than 9.

</details>

## How it is verified

tests/grader.py reads /app/no2_report.csv with the csv module and sets each site's monitor_mean and facade_mean against /tests/sealed.json. A site is right when both are within 0.5 per cent; test_no2_report.py passes only when no site is out, so the reward is 1 for ten right sites and 0 otherwise. The agent's code is not imported or run.

<details>
<summary>Full note</summary>

The sealed means exist only in the verifier image under /tests (mode 700), which the agent's container never mounts. Half a per cent is far more than a correct reading needs - it reproduces the sealed figures to the third decimal, the only slack being the rounding of a ppb column - and far less than any wrong one costs: a site worked from the wrong analyser's record is at least 15.4 per cent out, and an untidied unit is out by a factor of two or a thousand. Nothing in /app gives or implies any site's mean, and there is no total, duplicate or reference series to test an answer against; the right reading has to be reasoned to. test.sh writes 0 to /logs/verifier/reward.txt first and 1 only after pytest passes, so an empty container scores 0; the reference solution scores 1. The generator is seeded and is shipped with the tests.

</details>

<details>
<summary>Reference approach (spoiler)</summary>

solution/no2_report.py, standard library only, reads /app and writes /app/no2_report.csv. It reads each download with its own delimiter and decimal mark, converts a ppb column at 1.9125 and a milligram column by a thousand, and keeps the rows whose status is ok. For each record it takes the root mean square of a reading's distance from the mean of its two neighbours, over the record's spread, ranks the ten records and calls the five liveliest roadside. The two analysers whose records contradict the type of the site they are listed at are taken to have changed places. Each site's monitor mean is then the mean of the record that was really made there, and a roadside site's facade mean follows from the note's fall-off formula with that site's distances and background. It gets 10 of 10 in the task's container.

</details>

## Layout

```
instruction.md          the brief - the only thing the agent is told
task.toml               artifacts, timeouts, resources, metadata
environment/            Dockerfile and data for the agent's container
solution/solve.sh       reference solution; scores 1 by construction
tests/                  verifier image, grading logic and ground truth
```

## Running it

With [harbor](https://github.com/laude-institute/harbor) installed and Docker running:

```bash
uv tool install harbor

harbor run -p . -a oracle -e docker   # reference solution - must score 1
harbor run -p . -a nop    -e docker   # nothing happens    - must score 0
```

Those two runs are the calibration the bundle has to satisfy: the oracle run substitutes
`solution/solve.sh` for the agent and must score `1`, the nop run does nothing and must
score `0`. Anything in between means the verifier is measuring the wrong thing.

## Reading this

`tests/` and `solution/` hold the answers. To try the task honestly, read only
`instruction.md` and `environment/`.

---

Part of a set of 38 tasks indexed at
[afterquery-data-annotation](https://github.com/helexdev1004/afterquery-data-annotation).
