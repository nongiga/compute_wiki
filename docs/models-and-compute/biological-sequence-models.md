---
tags:
  - infrastructure
---

# Open-Weights Biological Sequence Models

The open-weights protein, DNA, RNA and genome models a safety or biosecurity project is likely to be
asked to measure: what exists, what the licence allows, and the loading behaviour that silently changes
the numbers. This page is the counterpart of
[`open-weights-models.md`](open-weights-models.md) for sequence models rather than chat models. It does
not cover generation or sequence design. For how to run and read a benchmark of one of these models, see
[`../evaluation/biological-sequence-model-evals.md`](../evaluation/biological-sequence-model-evals.md);
for scoring a chat or agentic model on hazardous biology, see
[`../evaluation/dangerous-capability-evals.md`](../evaluation/dangerous-capability-evals.md).

Aliases: protein language model (pLM), DNA language model, genomic language model (gLM), genomic
foundation model, RNA language model, biological foundation model, ESM, ESM-2, ESM C, ESM Cambrian,
ESM3, ProtT5, ProtTrans, ProGen2, SaProt, Ankh, Nucleotide Transformer (NT), DNABERT-2, HyenaDNA,
Caduceus, GENA-LM, GPN, GPN-MSA, Evo, Evo 2, gLM2, METAGENE-1, Bacformer, RNA-FM, RiNALMo, embeddings,
`from_pretrained`, Hugging Face, `fair-esm`, MultiMolecule, `trust_remote_code`.

Everything below was checked against the repository, model card, config or issue tracker on
2026-11-15; the date matters, because several of these facts changed during 2026. Licences and access
terms are reported as found and are not legal advice.

## At a glance

| You are about to… | Worth checking | Because |
|---|---|---|
| pick a protein model for embeddings | whether its licence permits your use | three of the common ones are non-commercial or unclear, and two changed terms during 2026 |
| pick a DNA model | its tokenisation | single-nucleotide, k-mer and byte-pair models are not interchangeable, and one benchmark found single-nucleotide tokens better on splice sites by 0.22 Matthews correlation |
| feed a sequence longer than the training crop | the model's positional scheme | learned positions raise an error or degrade; rotary and relative positions do not warn you |
| use a structure-aware protein model | whether you have the structure tokens it needs | one model takes amino acids plus Foldseek states and fails on a plain sequence |
| mix an RNA model with DNA input | its alphabet | one RNA model has no `T` and silently maps it to unknown; another converts `U` to `T` internally |
| compare two checkpoints of one family | that both loaded | one 2026 re-upload loaded as random weights with only a warning |
| cite a model in a paper | the version and revision | two of these changed tokeniser, context or conclusions between preprint versions under one identifier |

## Protein and RNA models

| Model | Hugging Face ids | Licence (2026-11-15) | Context | Tokenisation |
|---|---|---|---|---|
| **ESM-2** | `facebook/esm2_t{6_8M,12_35M,30_150M,33_650M,36_3B,48_15B}_UR50D` | MIT, not gated | 1022 residues trained; rotary, so no hard error | one token per amino acid, vocab 33, BOS and EOS added |
| **ESM C** (300M, 600M, 6B) | `biohub/ESMC-{300M,600M,6B}` | cards list MIT alongside a second entry (`other`), not gated | 2048 tokens local, 16,384 via the API | amino acids, chain-break token `\|` |
| **ESM3** (open 1.4B) | `biohub/esm3-sm-open-v1` | **card carries no licence field**; repository is MIT; not gated; 7B and 98B are API-only | 2048 tokens | amino acids, plus 4,096-codebook structure tokens |
| **ProtT5-XL-U50** | `Rostlab/prot_t5_xl_uniref50` (and a half-precision encoder) | conflicting: code MIT, README says Academic Free License v3.0, cards state none | 512-token crops; relative positions, no hard limit | SentencePiece over single residues; input must be **space-separated** |
| **ProGen2** | `hugohrban/progen2-*` (community ports); original weights from the `salesforce/progen` GitHub repository | BSD-3-Clause | 1024 tokens; 2048 for `progen2-base` | one character per residue plus terminal tokens `1` and `2` |
| **SaProt** | `westlake-repl/SaProt_{35M,650M}_AF2`, `_650M_PDB`, `_1.3B_AFDB_OMG_NCBI` | MIT, not gated | 1024 residues | each token is one amino acid **plus** one Foldseek 3Di state |
| **Ankh** (base, large, Ankh3) | `ElnaggarLab/ankh-{base,large}`, `ankh3-*` | **CC BY-NC-SA 4.0, non-commercial and share-alike** | relative positions, benchmarked to 1024 | one token per residue; passed as a list with `is_split_into_words=True` |
| **RNA-FM** / mRNA-FM | `cuhkaih/rnafm`; MultiMolecule port | code MIT, official weights Apache-2.0, **MultiMolecule port AGPL-3.0** | 1022 nucleotides; learned positions | single nucleotides over `A C G U`; **no `T`** |
| **RiNALMo** (35M, 150M, 650M) | weights from the authors' release; MultiMolecule port | code Apache-2.0, **weights CC BY 4.0**; port AGPL-3.0 | 1022 nucleotides trained; rotary allows longer | single nucleotides; **`U` is converted to `T` internally** |

## DNA and genome models

| Model | Hugging Face ids | Licence (2026-11-15) | Tokenisation |
|---|---|---|---|
| **Nucleotide Transformer v2** | `InstaDeepAI/nucleotide-transformer-v2-{50m,100m,250m,500m}-multi-species` | **CC BY-NC-SA 4.0, non-commercial**; v3 gated and non-commercial | 6-mers, falling back to single bases around `N` |
| **DNABERT-2** | `zhihan1996/DNABERT-2-117M` | repository Apache-2.0; **model card states no licence** | byte-pair encoding, vocabulary 4,096 |
| **HyenaDNA** | `LongSafari/hyenadna-*-seqlen-hf` | weights BSD-3-Clause, code Apache-2.0 | single characters; **pads on the left, ships no attention mask** |
| **Caduceus** | `kuleshov-group/caduceus-*-131k` | Apache-2.0 | character level, reverse-complement equivariant |
| **GENA-LM** | `AIRI-Institute/gena-lm-bert-{base,large}-t2t`, `moderngena-*` | code MIT; **cards carry no licence field** | byte-pair encoding, 32,000 |
| **GPN** / GPN-MSA | `songlab/gpn-brassicales`, `songlab/gpn-msa-sapiens` | MIT, not gated | single nucleotide (7-token and 6-token vocabularies) |
| **Evo 1 / 1.5** | `togethercomputer/evo-1-131k-base`, `evo-1.5-8k-base` | Apache-2.0 | byte level, single nucleotide |
| **Evo 2** | `arcinstitute/evo2_{1b_base,7b,20b,40b}` | Apache-2.0 | character level, vocabulary 512 |
| **gLM2** | `tattabio/gLM2_650M`, `gLM2_650M_embed` | Apache-2.0 | mixed: amino-acid tokens for coding regions, nucleotides between them |
| **METAGENE-1** | `metagene-ai/METAGENE-1` | weights Apache-2.0; **pre-training data not public** | byte-pair encoding, vocabulary 1,024 |
| **Bacformer** | `macwiatrak/bacformer-{masked,causal,large-masked}-*` | Apache-2.0 | **input is one pooled protein embedding per gene**, not a sequence |

## Licence and access traps

**Aliases:** non-commercial weights, CC BY-NC-SA, AGPL, gated repository, licence drift.

**Four of these are not freely reusable, and the constraint is not always on the page you read.**
Ankh's models are CC BY-NC-SA 4.0, which is non-commercial *and* share-alike. Nucleotide Transformer's
code and v1/v2 weights are CC BY-NC-SA 4.0, and v3 is additionally gated. ProtT5 carries three
different statements across its repository, README and cards. DNABERT-2 and GENA-LM have no licence
field on their model cards even though their code repositories are licensed, and the open ESM3
checkpoint's card has no licence field either, so those weights are effectively unlicensed unless you
ask. ESM C's cards list MIT together with a second entry, so read the card rather than assuming one
licence covers the family.

**A permissive code licence does not cover the weights, and the two are often different.** HyenaDNA
ships Apache-2.0 code with BSD-3-Clause weights; RiNALMo ships Apache-2.0 code with CC BY 4.0 weights;
RNA-FM's MultiMolecule port is AGPL-3.0, a network-copyleft licence, while the official weights are
Apache-2.0. If you serve a model behind an interface, the port you installed may be the binding
licence.

**Terms changed during 2026, so a 2025 paper's access statement is not current.** ESM C and the open
ESM3 weights were non-commercial or API-only until mid-2026 and are MIT and ungated now. Read the
licence, not the paper.

## Loading traps that change the numbers

**Aliases:** silent failure, random weights, `trust_remote_code`, padding, mixed precision, revision
pin, `model.eval()`.

These are the sequence-model instances of the instrument check on
[`../evaluation/biological-sequence-model-evals.md`](../evaluation/biological-sequence-model-evals.md#step-5-check-the-instrument-numerically-before-scoring),
which is where the general rule lives. They are not repeated there.

**Pitfalls:**

- **Weights can load as random with only a warning.** The republished ESM C 6B weights loaded as random
  through the `transformers` classes, reported in the maintainers' issue tracker and fixed in a later
  loader. Symptom: plausible-looking embeddings whose downstream score is near chance. Check that a
  forward pass on a fixed input differs from a randomly initialised copy of the same architecture.
- **One import name, two packages.** `fair-esm` (ESM-2) and `esm` (ESM C, ESM3) both install a Python
  module called `esm`, so they cannot coexist in one environment. Symptom: `ImportError`, or worse, the
  wrong model class resolving silently.
- **Left padding with no attention mask.** HyenaDNA pads on the left and ships no mask, so padding
  enters any mean over tokens. Pool over real tokens explicitly.
- **Dropout left on.** RiNALMo keeps dropout layers active until `model.eval()` is called, so the same
  sequence gives different embeddings run to run. Symptom: an embedding that is not reproducible for
  one input.
- **Alphabet mismatches are silent.** RNA-FM's official alphabet has no `T` and maps it to the unknown
  token, so DNA input degrades without error; RiNALMo converts `U` to `T` internally. The same string
  means different things to the two models.
- **Structure tokens are required, not optional.** SaProt expects amino acid plus Foldseek 3Di state
  per position and fails on a plain sequence; `#` stands in where structure is unknown.
- **Terminal tokens are part of the score.** ProGen2 expects `1` at the N-terminus and `2` at the
  C-terminus, and the reverse pass needs the string reversed including them, so a likelihood computed
  without them is not the model's likelihood.
- **Space-separated input.** ProtT5 tokenises SentencePiece over single residues and expects residues
  separated by spaces; passing a plain string silently produces different tokens.
- **Truncation is silent.** ESM-2's extraction script drops residues past 1022 by default. Models with
  learned positions (ESM-1b, RNA-FM) raise an error instead, which is the safer failure.
- **Pin the revision.** Evo 1 had to be loaded at a specific revision tag to be correct, and gLM2's
  released checkpoint changed tokeniser and context after its preprint, so a citation without a
  revision may not describe the weights you ran.

## Choosing a readout

Which layer, which pooling rule and which likelihood score are properties of the measurement rather
than of the model, and they move results by more than the differences usually being claimed. That
material, with its published cases, is on
[`../evaluation/biological-sequence-model-evals.md`](../evaluation/biological-sequence-model-evals.md#step-3-choose-the-readout-layer-pooling-rule-and-scoring-rule-on-data-you-will-not-report)
and is not repeated here. The one point worth stating beside the model list: a repository quickstart's
layer is not a recommendation, and at least one model card extracts the layer its own paper reports as
worst.

## Cross-references

- Running and reading a benchmark of one of these models:
  [`../evaluation/biological-sequence-model-evals.md`](../evaluation/biological-sequence-model-evals.md)
- Open-weights chat models, interpretability artefacts and access routes:
  [`open-weights-models.md`](open-weights-models.md)
- Scoring a chat or agentic model on hazardous biology, and the two uplift trials:
  [`../evaluation/dangerous-capability-evals.md`](../evaluation/dangerous-capability-evals.md)
- Safeguards applied to biological models themselves, including pre-training data filtering:
  [`../alignment-science/unlearning.md`](../alignment-science/unlearning.md)

---

## Common questions

### Which protein model should I start with for embeddings?

ESM-2 650M, because it is MIT, ungated, widely reported, and small enough to extract on one GPU; ESM C
300M is the current-generation equivalent and is also MIT now. Check the licence before choosing Ankh or
Nucleotide Transformer for anything commercial.

### Can I use Nucleotide Transformer or Ankh in a commercial project?

Not under their stated terms. Both are CC BY-NC-SA 4.0, which is non-commercial and share-alike, and
Nucleotide Transformer v3 is gated on top of that. Apache-2.0 or MIT alternatives exist in both the DNA
and protein families.

### My embeddings are not reproducible between runs. Where do I look first?

In order: dropout left on (call `model.eval()`), batch padding entering the pooled mean, and default
mixed precision. All three have documented instances in these models, and all three are caught by
comparing a sample against a single-sequence, float32, processor-only run.

---

Last verified: 2026-11-15. New page. Every licence, identifier, context length, tokenisation and trap
above was read on the model's repository, model card, `config.json`, `tokenizer_config.json` or issue
tracker on that date, as part of a structured pass over twenty models. Every Hugging Face identifier in
the two tables was then re-resolved against the Hugging Face API, and all of them return the model with
the licence and gating stated here; two licence cells were corrected at that step (the open ESM3
checkpoint's card carries no licence field, and ESM C's cards list MIT with a second entry). where a repository and a model
card disagree, both are reported rather than reconciled. Parameter counts and context lengths are as
stated by the maintainer, not measured. Licence statements are reported as found and are not legal
advice; several changed during 2026, which is why the date is given. Not verified: whether the ESM C 6B
loading failure persists in the current loader (the issue is closed), and the training crop length for
SaProt. One-line summaries are Claude's wording. Drafted by Claude; pending MATS research-staff review.
