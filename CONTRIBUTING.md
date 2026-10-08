# Contributing to Darkbloom provider economics

This hub collects what Darkbloom providers actually earn per resident GB-hour, per model and
per hardware class, so that model choice can be compared across machines instead of guessed
from one. A `$/GB·h` figure only means something next to figures from other hardware, so
**data from a machine class we don't have yet is the most valuable thing you can add.**

This page is the procedure. Read it top to bottom once; every step is a command. The general
Research Commons mechanics are in the tool's
[`docs/COLLABORATING.md`](https://github.com/research-common/research-commons/blob/main/docs/COLLABORATING.md).

## What you need

- An Apple Silicon Mac running a Darkbloom provider, serving normally (`darkbloom status`).
- macOS with Python **3.11+** and **node + npm** (the signer runs on node; without it every
  publish is silently unsigned and can't be attributed to you).
- [`dbyield`](https://github.com/brandon-eigenlabs/dbyield), the exporter. You cannot contribute without it,
  because the only accepted format is a `dbyield export` bundle (see *Privacy*).
- The Research Commons tool, **v0.3.0-alpha.1 or later**, the version this hub's CI runs.

## What to contribute

| Kind | What it is | Effort on your machine | How it's endorsed |
|---|---|---|---|
| **Census export** | `dbyield export` after at least 24 h of the sampler on your normal configuration | none: no restarts, no config changes | listed as CLAIMED at once, then endorsed under role `census` if it meets the bar below |
| **Campaign** | a pre-registered, round-robin experiment meeting the collection's `wanted:` criteria | 5–8 h or more of controlled restarts, which cost earnings | endorsed as an `instance` if it meets the criteria |

Start with a census export. **The census bar** (it's the collection's own rule, so read it
there with `commons cat <current collection>`; `collection show` doesn't print notes):
- at least 24 h of sampler coverage inside the export window;
- your normal configuration, with no experiment or configuration change during the window;
- the whole window inside one routing era (`METHOD.md` §17), so don't straddle a known
  coordinator deploy;
- published with `--link applies:sk-798a4418` and attested with `--observed` at the window's end;
- at most one endorsed census per machine per routing era.

A census says what a model earned on your hardware class as you chose to run it. It isn't a
comparison of models, and it's never pooled with campaign arms.

Before planning a campaign, read the criteria and the method: `commons collection show <current
collection>` (the `wanted:` line) and `commons cat sk-798a4418` (the method, rev 4).

## Steps

### 1. Install, once

```bash
git clone https://github.com/research-common/research-commons.git ~/research-commons
cd ~/research-commons && git checkout v0.3.0-alpha.1 && npm install --ignore-scripts
export PATH="$HOME/research-commons/bin:$PATH"

git clone https://github.com/brandon-eigenlabs/dbyield.git ~/dbyield
~/dbyield/install.sh --with-sampler      # puts dbyield on PATH and starts the sampler
```

The sampler only reads state; it never changes your provider. Leave it running.
`$/GB·h` has no denominator without it.

### 2. Make your identity, and send us your address

```bash
mkdir -p ~/.commons && chmod 700 ~/.commons
python3 -c "import secrets;print('0x'+secrets.token_hex(32))" > ~/.commons/signing.key
chmod 600 ~/.commons/signing.key
export COMMONS_SIGNING_KEY=~/.commons/signing.key COMMONS_AGENT=<your-handle>
```

Fork this hub, clone your fork, and from inside it run `commons peer whoami`. **Open an issue on
this hub with that address.** A maintainer registers you at `datasets-only` trust. Until they
do, ingest refuses your work by name, so do this first. One key per person, never shared.

### 3. Capture, then export

Wait until the sampler has at least 24 h of data, then:

```bash
dbyield ingest
dbyield export --hours 24        # writes ~/.darkbloom-yield/reports/export/dbyield-<ts>.json
```

`export` refuses to write anything that fails its schema or its identifier audit.

### 4. Publish into your hub clone

```bash
cd <your fork of this hub>
git checkout -b <handle>-<chip>-<date>
CL=$(commons list --type collection --tips-only | grep -o 'cl-[0-9a-f]\{8\}' | head -1)
commons publish dataset ~/.darkbloom-yield/reports/export/dbyield-<ts>.json \
  "Darkbloom yield: <chip> <RAM>, <window start>..<end>" \
  --tier T3 --criteria "dbyield <version> export; <chip> <RAM>; <window>; machine_ref is HMAC-SHA256 under a local salt" \
  --license CC-BY-4.0 --obtainability open \
  --link part-of:$CL --link applies:sk-798a4418
commons attest ds-… --observed <end of your capture window, ISO-8601 Z>
git add registry store && git commit -s -m "contrib: <chip> <RAM> census export"
commons hub check --base origin/main       # the same gate CI runs; must say OK
git push -u origin HEAD
```

Then open a pull request from your fork's branch to this hub's `main`.

- **Always look up the collection id as above.** Every endorsement publishes a new version, so
  any id written in a document goes stale.
- `--link applies:sk-798a4418` records that you followed the method. Without it your data
  can't be counted as an application of it.
- `--observed` is when the data was *captured*, not when you published. Pass it explicitly:
  re-attesting without it resets it to the current time.

### 5. Review

A maintainer ingests your branch through the trust gate (`commons pull`), not the merge
button. Your dataset shows as CLAIMED under the collection immediately and becomes ENDORSED
when a maintainer adds it as a member. If something is refused, the PR will say why.

## Privacy: what leaves your machine

An export holds **aggregates only**: earnings and exposure per model, your hardware class (chip,
RAM, GPU cores, macOS and Darkbloom versions, trust level) and a `machine_ref`. That's an HMAC
of your machine id under a salt that never leaves your Mac. It makes your exports joinable with
each other without identifying the machine, and nobody can reverse it.

It never contains `account_id`, `provider_id`, `provider_key` or `job_id`. This collection's
ingest policy refuses any file whose fields include those names, so a raw ledger can't be
published here even by mistake. **Publish only `dbyield export` output.** The policy is a floor,
not a guarantee: a renamed column would pass it, which is why the exporter does the stripping.

Anything published here is public and permanent. Retraction is best effort, not erasure.

## Rules that keep the hub checkable

- Change only `registry/` and `store/`. Never hand-edit an existing file under
  `registry/artifacts/`. Use `commons publish … --force` to change metadata, which the gates check.
- Use your own key, and sign your commits off (`-s`).
- Run `commons hub check --base origin/main` before every push.
- Don't publish code. Methods are published by maintainers as content-addressed artifacts
  (`sk-…`, `wf-…`), and you cite them by id.

## For agents setting this up for someone

Do the steps in order and stop at each point that needs the person:

1. **Stop for the person:** their consent to run the sampler on their machine.
2. Install (step 1). Check `node --version` and `commons --version` (v0.3.0-alpha.1 or later).
3. Make the key (step 2). **Never print or commit the key file**; the address is all that's
   shared. **Stop for the person** to open the issue with the address, or have them confirm you
   may open it.
4. Wait 24 h, export, publish, attest, and check (steps 3–4). **Stop for the person** before
   `git push` and before opening the pull request: these publish their data permanently.

Never do any of these:
- publish a file other than a `dbyield export` bundle;
- pass `--allow-unchecked-ingest` or `--force` to get past a refusal;
- run `dbyield experiment`, `darkbloom stop` or `restart`, or change `provider.toml` without the
  person's explicit go. These change what the provider earns.

If a publish is refused for a forbidden field name, the file is the problem, not the gate.

## Questions

Open an issue on this hub.
