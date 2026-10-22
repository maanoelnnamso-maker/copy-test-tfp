# Copy test and lead fill: build template

The structure of a per-client outbound build that runs on Railway (one web service, one Postgres database, one Slack app, one EmailBison workspace).
It holds no client data and no engine code. It is the map, the config skeleton and the rules, so a new client build can be stood up the same way every time.

## What a build does

```
  companies            contacts                 bank                     campaigns
  ---------            --------                 ----                     ---------
  source by vertical   find people (titles)     verified contacts        copy test: 3 angles, written copy per lead
  declared by a        fit check the company    waiting to be used       fill campaign: loaded as is, no copy written
  person, never        BEFORE paying to verify
  inferred             verify the email         held rows with a reason
```

Three jobs run inside the one web service and report to the client's Slack channel:

| Job | What it does |
|---|---|
| Producer | Picks sourced companies, finds and verifies contacts, keeps the bank topped up. Several chunks run side by side, each company claimed once. |
| Consumer | Takes contacts from the bank into campaigns. Either the copy-test fill (research and emails written per lead) or the fill-campaign loader (no copy). |
| Nightly | Reads the campaigns' own EmailBison counters, decides the copy test, runs the scale reviews and posts a short status. |

## The three ideas worth keeping

1. **Declared, not inferred.** Verticals, titles, excluded roles, employee band and countries are written down by a person in the client's config. A model may draft them from the onboarding form. A person approves.
2. **Check fit before spending.** The company is judged by a cheap model on its own website before any email is verified. A non-fit lead that reaches a campaign cannot be detached by API, so mistakes are permanent.
3. **People press the scale button.** Nothing scales a campaign past its first stage without a person pressing a Slack button after a review of real EmailBison numbers.

## Folder map of a client build

```
app/                 web service, jobs, stages, EmailBison and LLM clients      (engine, not in this template)
clients/<client>/    everything that is specific to one client                  (skeleton in clients/_template)
  client.toml        workspace, host, key variable, ICP bounds, verticals, research settings
  copy_test.toml     angles, segments, decision rules, scale gate, fill target
  copy_rules.toml    client copy bans and required patterns, added to a shared floor
  titles.txt         buyer titles, best first
  exclusions.md      the ONLY misfits, and excluded roles
  icp.md  pitch.md   who it is for and what is sold, in the client's words
  fit_prompt.txt     the company fit rule given to the cheap model
  do_not_contact.txt names that must never be emailed
  prompts/<angle>/   research, email 1 and email 2 prompts for each angle
```

## Where to start

1. `docs/RUNBOOK.md`: standing up a new client, step by step.
2. `docs/SCALING.md`: how a copy test is decided and how any campaign scales.
3. `docs/FILL_CAMPAIGNS.md`: loading campaigns that need no written copy.
4. `clients/_template/`: copy this folder to `clients/<client>/` and fill every file.
5. `.env.example`: the variable names a service needs. Values live only in Railway and on the operator's machine.
