# reproducible-science

Command-line tools that check whether a paper's claims match its artifacts.

1. **Three outcomes kept apart**: a value that disagrees, a value the artifact is silent on,
   and a check that never ran. A pass/fail report collapses them, and their remedies differ.
2. **Deterministic and offline.** A check is a digest comparison and a decimal comparison, so
   it runs on every commit rather than once before submission.
3. **Facts separated from verdict.** The engine computes no verdict; a policy maps outcomes to
   severity, so an unchecked citation is acceptable in a draft and disqualifying in a submission.
4. **One contract over all three kinds of evidence** — a quotation in a source, a value at a
   location in a machine-readable file, a cell in a table — through one engine and one report.

```bash
pip install "git+https://github.com/anon-submission-account/reproducible-science"
```

See our [Claude Code plugin](packages/repro/plugin/),
which traces results at the time of experimentation.

```text
/plugin marketplace add anon-submission-account/reproducible-science
```

**Recommended:** poppler (`pdftotext`), for reading PDF sources. `citations` falls back to a
pure-Python reader without it, so `pip install` is enough; poppler resolves more passages than
either fallback on a measured corpus, and every result records which extractor produced it.

```bash
brew install poppler                  # macOS
sudo apt install poppler-utils        # Debian/Ubuntu
pip install pdfplumber pypdf          # no system package available
```

## Documentation

[docs/](docs/) for the specification and the decision reference, [jupyterlite/](jupyterlite/)
for notebooks that run the tools in a browser.

## Install

| tool | what it checks |
|---|---|
| [`prereg`](packages/prereg) | that the plan you ran is the plan you froze, against a commit and a content hash |
| [`results`](packages/results) | that a reported number came from a recorded run over sealed inputs |
| [`citations`](packages/citations) | that a quoted passage occurs in the source cited for it |
| [`repro`](packages/repro) | that every declared claim holds against its pinned artifact, and which ones do not |

One install brings all four:

```bash
pip install "git+https://github.com/anon-submission-account/reproducible-science"
```

Each is also its own distribution, so a checkout can install one alone with
`pip install -e packages/prereg`.

## Getting started

`repro demo` writes a small worked example, runs the sequence below over it for real, and then
breaks it twice — so the first thing you watch the tool do is catch something, and the two
things it catches are not the same kind of failure.

Write down what your paper claims and where each claim comes from. `repro verify` checks every
one against the file it names and tells you which ones hold.

```console
$ prereg freeze                                    # lock the plan; names a git commit
$ results seal analysis.py data.csv                # hash the inputs, before the run
$ results run output.json --run-id exp_001         # hash the outputs, after it
$ results claim "ICC = 0.42" --run-id exp_001 --location "Table 2"
$ repro verify
repro.yaml

  ok    icc          metric   /icc = 0.42
  MISS  pvalue       metric   p: manuscript prints 0.031, results.json holds 0.051
  GONE  n            metric   /n does not resolve in results.json

  1 mismatch, 1 not_found, 1 verified
  policy publication: FAILED  (2 errors, 0 warnings)
    error   evidence.mismatch          pvalue/metric: p: manuscript prints 0.031, results.js
    error   evidence.not_found         n/metric: /n does not resolve in results.json
```

`ok` holds, `MISS` disagrees with the artifact, `GONE` names an address that no longer
resolves. Three further outcomes appear in longer runs: `--` for a check that could not run,
`none` for a claim offering no evidence, and `ERR` for a source that could not be read.

Quotations are matched against the sources they cite, page numbers included. Reported numbers
are compared against the value stored at an address you gave — never by searching a file for a
number that looks close enough. When a check cannot run at all, it reports that, and the run
does not pass.

Preregistration is optional — you mark a claim confirmatory, exploratory, or not applicable,
and only the first kind needs a plan behind it.

## Contributing

[Contributing](CONTRIBUTING.md) to get started, [Development](DEVELOPMENT.md) for the workspace
and release machinery, and [docs/SPEC.md](docs/SPEC.md) for what a decision means. Security
issues: [SECURITY.md](SECURITY.md).

MIT licensed.
