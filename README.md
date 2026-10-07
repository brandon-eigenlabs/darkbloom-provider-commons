# Darkbloom provider economics

What a Darkbloom provider earns per resident GB-hour, per model and per hardware class, and
which configuration decisions actually move it. **Want to contribute data from your
machine? Start with [`CONTRIBUTING.md`](CONTRIBUTING.md).**

A **Research Commons hub**: a data-only git repository (manifests in `registry/`,
content-addressed blobs in `store/`). It contains **no code**. The `commons` tool lives in
a separate repository — [research-common/research-commons](https://github.com/research-common/research-commons) — and you
update it there, never through this hub.

## Read it

```bash
git clone https://github.com/research-common/research-commons.git ~/research-commons   # the tool, once
export PATH="$HOME/research-commons/bin:$PATH"
git clone <this hub> && cd <this hub>                             # the data
commons list --type collection --tips-only   # the current collection (ids change on every endorsement)
commons collection show <cl-id>       # endorsed + claimed members, open work
commons verify <id>                   # re-derive it yourself
```

`commons` finds this hub automatically from any directory inside it (the `.commons-hub`
marker file); `commons hub where` says which data root is in use.

## Contribute

The hub-specific procedure, including what data is wanted and how it is anonymised, is
[`CONTRIBUTING.md`](CONTRIBUTING.md). The general mechanics:

1. Mint a signing key and tell the maintainers your address (`commons peer whoami`).
2. Fork this hub (or push a branch if you have write access), publish from inside your
   clone, commit `registry/` + `store/`, and open a pull request.
3. CI runs `commons hub check`: the PR may only touch `registry/` and `store/`, every
   blob must match its hash, every signature must verify.
4. A maintainer ingests it through the gate — `commons pull <your-remote> --branch
   <branch>` — which additionally checks signer trust. Endorsement = the maintainer
   republishing the collection with your artifact as a member; until then it shows as
   CLAIMED via your `--link part-of:cl-…`.

Full guide: `docs/COLLABORATING.md` in the tool repository.

## Maintainers

| handle | signing address |
|---|---|
| brandon-curtis | `0x44AA8c1d7213E36902c72AF22dC68C4BFcA649Ab` |
