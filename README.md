# skills

A collection of Claude Code skills for statistical and data science workflows based on Frank Harrell's online books, course notes, and blog

## Subfolders

- **principles** — General statistical thinking and modeling principles
- **rworkflow** — R project workflow and best practices
- **rms** — Skills related to the `rms` R package (regression modeling strategies)
- **bbr** — Skills related to *Biostatistics for Biomedical Research* (BBR)
- **bayes** — Bayesian analysis methods and workflows
- **dist** -- `.skill` files for distributing packaged skills

## Usage

To apply a skill to Claude AI, download the `dist/*.skill` file of your choice and upload it to Claude skills by clicking the following in the top left corner of the Claude desktop: `Customized ... Skills` then clicking `+`. Click `+ Create Skill` and select `Upload`.  Browse to where you stored the `.skill` file downloaded from the `dist` folder in this `github` repository, and upload it.

Once installed the skill activates automatically when Claude detects a relevant query — no explicit invocation needed. For users who aren't sure whether a skill is triggering they can mention it explicitly: "using your rms skill, help me with..."

It is recommended to create a Claude Project for a major analysis so that Claude saves persistent contexts and automatically triggers skills.

Please report any errors or suggestions for the skills using Github `Issues`
