# FOL Automation

Fetches data from the Saudi Exchange website, writes it into an Excel workbook, produces a report from it through formulas, and emails the report out — unattended, on a schedule.

Each stage is a folder containing a `CONTEXT.md` file that describes exactly what to do, and the work is done by following those instructions directly.

The folders carry the order things happen in, the nesting carries the context, and the files carry the current state. **If something needs explaining, the explanation belongs in that folder's `CONTEXT.md`.**

## Where things are

| File or folder | What's in it |
|---|---|
| `GLOSSARY.md` | recurring terms — the site, the workbook, how access works. Start here if you're new |
| `CONTEXT.md` | what this automation does, stage by stage |
| `config.yaml` | the settings — where the workbook is, where the data comes from, who gets the report |
| `copying-formulas-down.md` | how to add a row by copying the row above, through Graph. Used by stage 08 |
| `stages/` | the stages themselves, in the order they run |
| `run/` | files one stage writes and a later stage reads, within the same run — doesn't exist in a fresh copy |

## Where to start

| If you're | Go to |
|---|---|
| new, or not sure where things stand | `CONTEXT.md` |
| starting a run | `stages/01_authentication/CONTEXT.md`, then each stage in order |
| asked how things stand | look through `run/` and report what's there |
| clarifying or adding a step | update that stage's `CONTEXT.md` directly, then update `CONTEXT.md`'s stage table if the shape changed |

## Rules

- **Write every instruction file so someone can read it cold and follow what happens**. This includes someone who hasn't worked with code before. Plain words, short sentences, concrete examples.
- **Where something genuinely can't be explained without technical detail** (a URL, an exact data format), gather it into a section at the end of the file and explain in plain terms above it what that detail achieves. Mark the section as required in order to run the step.
- **Always refer to the technical details in a stage's `CONTEXT.md` when it has them**.


## Permissions

The agent is allowed to do everything the `CONTEXT.md` files require in order to run. That includes commands such as running commands and scripts, fetching web pages (including using `curl_cffi` to get past the Saudi Exchange site's 403 block, which the owner has approved), calling the Graph API, sending the email, and writing files in `run/` etc. Do not stop to ask for permission for a step a `CONTEXT.md` file tells you to do.
