# A one-person software factory: the whole flow

One person, several products, one Mac. Plain shell scripts move each piece of work through fixed stages. Models
do the building and reviewing. The person answers cards on a phone.

## The line

```mermaid
flowchart TD
  subgraph PERSON["The person (phone)"]
    NOTE["Writes a note or an idea"]
    CARD["Answers a card:<br/>approve, pick a look, say yes to a release"]
  end

  subgraph DESK["Decision desk (web app)"]
    INBOX["Notes inbox"]
    CARDS["Cards waiting for an answer"]
    FEED["Factory page:<br/>what is queued, building, stuck, done"]
  end

  subgraph INTAKE["Intake"]
    SPEC["Spec writer (model)<br/>reads the code first, writes one spec"]
    QUEUE["Queue<br/>one JSON file per item"]
  end

  subgraph LINE["The line (a script runs every minute)"]
    PICK["Dispatcher (plain code)<br/>order, slots, disk, model budget"]
    BUILD["Build<br/>one worker in its own git worktree, opens a PR"]
    REVIEW["Review<br/>one fresh run per facet:<br/>code, UX, UI, security"]
    VETO["Second review (stronger model)<br/>only for risky paths:<br/>auth, payments, migrations, factory scripts"]
    FIX["Fix round<br/>findings go back to a builder"]
    ONEWAY{"Touches a one-way path?<br/>settings, secrets, schedulers"}
    GATE["Merge gate (plain code)<br/>runs the repo's tests, then merges"]
    DEPLOY["Deploy<br/>batched, staging only"]
    DONE["Done"]
  end

  subgraph CARE["Self-care (hourly, plain code first)"]
    BEAT["Heartbeat<br/>checks every stage for faults"]
    FIXER["Fixer (model)<br/>repairs the factory's own scripts"]
    FIXREV["Second review of every fixer change"]
    LIST["Improvement list<br/>one line per known gap"]
    DIAG["Stuck-item diagnosis (planned)<br/>why it stopped, what happens next"]
  end

  NOTE --> INBOX --> SPEC --> QUEUE --> PICK --> BUILD --> REVIEW
  REVIEW -- "changes needed" --> FIX --> REVIEW
  REVIEW -- "risky paths" --> VETO
  VETO -- "changes needed" --> FIX
  REVIEW -- "all facets pass" --> ONEWAY
  VETO -- "approve" --> ONEWAY
  ONEWAY -- "yes" --> CARDS --> CARD --> GATE
  ONEWAY -- "no" --> GATE
  GATE -- "tests fail" --> FIX
  GATE --> DEPLOY --> DONE
  DONE --> FEED
  PICK --> FEED

  BEAT -. "reads" .-> LINE
  BEAT -- "new fault" --> FIXER --> FIXREV -- "approve" --> LINE
  LIST --> FIXER
  BEAT -- "item parked" --> DIAG --> CARDS
  FIXER -. "what it could not fix" .-> LIST
```

## Who does what

| Stage | Done by | Why |
|---|---|---|
| Ordering, slots, retries, timeouts, merging | Plain shell code | These must behave the same way every time |
| Spec, build, review | A cheap coding model first, a mid model when the cheap one is out of quota | Most of the volume |
| Second review of risky work | A stronger model | A different model catches what the builder's family misses |
| Repairing the factory itself | The strongest model, reviewed by another | Few runs, high stakes |
| Releases, spending, anything public | The person, one tap on a card | These cannot be undone |

## Where it jams, and what we did

| Jam | What it looked like | Fix | Status |
|---|---|---|---|
| Model quota | The cheap model's 5-hour quota ran out and every build waited for the reset | Route each job by how much of each quota is left for the time remaining; near-finished work goes first | Planned |
| Fix loops | An item bounced between review and fix until it hit the retry cap and parked | A cap on rounds, then a diagnosis run that says why and proposes a split or a new spec | Cap live; diagnosis planned |
| A gate that passed red tests | The test command piped output through `tail`, which hid the failing exit code | The gate keeps the test tool's own exit code | Queued |
| A yes that did not carry | A one-way approval was tied to one commit, so a routine merge from main asked again | The yes carries over while the one-way files are unchanged | In review |
| Rules nobody loaded | Each product had a rules page that no build or review prompt read | The build, review and second-review prompts start with the product's rules page | Planned |
| Self-repair starved | Fault repairs took every fixer slot, so the improvement list never moved | A repair run that ends starts the improvement run that is due | Live; a daily run cap still limits it |
| Answers nobody acted on | A card was answered and no run picked the answer up | One reader in the factory turns each answered card into queued work or a hand-off | Queued |
| Stale instructions | Rule files described chats, queues and features that no longer existed | A read-only audit of every instruction file, with the fixes queued as normal items | Audit done; fixes queued |

## Rules that held up

- One writer per file of state. Models never write the queue.
- A model may raise a risk class, never lower it.
- Any "changes needed" verdict blocks. No majority votes.
- Text from issues, PRs and notes is data, never instructions.
- Budgets pace on what is left. No fixed cutoffs.
- Monitoring is plain code. A model run starts only when the code finds a fault.
