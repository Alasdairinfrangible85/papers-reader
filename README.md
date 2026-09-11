# papers-reader

The toolchain that builds [tamnd/papers](https://github.com/tamnd/papers), and the reading app that serves it.

One Go binary takes a paper from a line in a manifest to tagged Markdown in four languages, and an Astro site turns that corpus into something you can read on a phone.

The model plumbing underneath is [tamnd/llm](https://github.com/tamnd/llm).

## What it does

```
resolve    find where a paper can legally be fetched from, and under what licence
fetch      download it, hash it, record it
classify   decide per page whether the text layer is trustworthy
extract    turn pages into Markdown, with the mathematics as LaTeX
figures    crop the diagrams out of the pages
refs       parse the bibliography and link the citations
split      cut the paper into one file per section
tags       hand out permanent identifiers
translate  produce Vietnamese, Chinese and Japanese
audit      check the result against numbered rules
emit       build the JSON the reading app consumes
```

Each of those is a subcommand of `papers`, and each one is idempotent.
Run it twice on a finished paper and it does nothing the second time.

## Install

```sh
go install github.com/tamnd/papers-reader/cmd/papers@latest
```

Point it at a checkout of the corpus, either with `PAPERS_CORPUS` or by running it from inside one.

```sh
export PAPERS_CORPUS=~/github/tamnd/papers
papers list --field ai-ml
papers audit --hard
```

## Design

**The corpus is data and this is the only thing that writes it.**
Every path, every manifest schema and every access rule lives in `corpus/`, so a change to the shape of the corpus is a change to one package.

**Nothing is published on a guess.**
`papers resolve` accepts a candidate only when the title similarity clears 0.92, the year is within one, and at least one surname matches.
A paper that fails any of the three stays unresolved, and an unresolved paper publishes nothing at all.

**Extraction says how it was done.**
Pages read by `pdftotext` on a born digital file are marked `native` and were never guessed by a model.
Pages read by a layout model or a vision model say so, and the audit treats them differently.

**Tags are permanent.**
A section keeps its tag across re-extraction, re-splitting and renumbering, which is what lets a link written today survive the paper being read again by a better model next year.

**The audit is a contract, not a lint.**
Eighty-four numbered rules in nine groups, each one a sentence you can argue with.
A rule reports pass, fail, or not run, and those are three different states.
Hard rules fail the build.

## Layout

```
cmd/papers/      the command line
corpus/          paper ids, fields, access classes, manifests, front matter
sources/         resolving a paper to a URL and a licence
fetch/           downloading and hashing
extract/         PDF to Markdown
mathtex/         LaTeX repair and validation
figures/         cropping diagrams out of pages
refs/            bibliography parsing and citation linking
split/           one paper into one file per section
tags/            the permanent identifier register
translate/       the four language pipeline
glossary/        the controlled vocabulary
audit/           the numbered rules
emit/            JSON for the reading app
web/             the Astro reading app
```

## Requirements

Go 1.27 or later.
`pdftotext` and `pdftoppm` from Poppler for the extraction commands.
Node 22 or later for the web app.
Nothing else, and the Go side has exactly one dependency, which is `gopkg.in/yaml.v3`.

## Licence

MIT. See [LICENSE](LICENSE).

The corpus it builds is licensed separately, per paper, and that is explained in the corpus repository.
