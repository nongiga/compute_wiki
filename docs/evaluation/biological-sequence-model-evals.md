---
tags:
  - evaluation
---

# Biological Sequence Model Evals: measuring what a DNA, protein or RNA model can do

How to run and read a benchmark of a biological sequence model, meaning a model pre-trained on
nucleotide or amino-acid sequence (ESM, Evo, Nucleotide Transformer, RNA-FM and the rest) rather than
on text. This is a safety-measurement page, not a biology-tooling page: biosecurity work needs to know
what these models actually carry, and that number comes out of a benchmark whose construction decides
it. The neighbouring page
[`dangerous-capability-evals.md`](dangerous-capability-evals.md#bio-and-chem-vct-lab-bench-and-what-benchmarks-miss)
covers the other half of the bio portfolio, where a chat or agentic model is scored for hazardous
knowledge and task performance, and already carries two results about biological models themselves
(pre-training data filtering undone cheaply, and agentic mutation design).

Aliases: biological language model (bioLM) evaluation, protein language model (pLM) benchmark, DNA language model (DNA LM) benchmark, genomic language model (gLM) evaluation, genomic foundation model benchmark, RNA language model benchmark, sequence model probing, embedding readout, mean pooling, layer selection, masked marginal likelihood, pseudo-log-likelihood, homology-aware split, gene-family split, clade holdout, leave-one-organism-out, sequence identity threshold, homology partitioning (GraphPart, MMseqs2, CD-HIT), homology leakage, data leakage, kmer and k-mer baseline, CLS token, summary token, AUROC, AUPRC, precision-recall, multiple comparisons, multiplicity, hyperparameter and hyper-parameter, from_pretrained, randomly initialised and randomly initialized, zero-shot variant effect prediction, deep mutational scanning (DMS), ESM, Evo, Nucleotide Transformer, DNABERT, HyenaDNA, Caduceus, RNA-FM, ProteinGym, FLIP, TAPE, BEND, DART-Eval, GUE, DGEB.

The setting in mind is a frozen open-weights sequence model, read with a probe or a likelihood, scored
against labels from a public database or an assay. The evidence below comes from a read of eleven DNA
model papers, thirteen protein model papers, the RNA, codon and single-cell literature, eight benchmark
suites and about twenty independent re-evaluations, plus the validation and statistics primary sources.
The effect sizes are what those papers report, mostly on one task each, so treat them as the size of a
trap rather than as a constant. Where this page extends a source rather than quoting it, it says so.

Every item here is a **recommended check**, not a requirement. The team running the experiment knows
its organism, assay and claim better than a page can.

Related pages: [`../engineering/measurement-validity.md`](../engineering/measurement-validity.md) (is
the instrument measuring what you think),
[`eval-statistics.md`](eval-statistics.md) (error bars when items share structure),
[`../engineering/statistics.md`](../engineering/statistics.md) (which test, how many samples),
[`../interpretability/probes.md`](../interpretability/probes.md) (fitting and reading a probe),
[`../engineering/research-rigor.md`](../engineering/research-rigor.md) (pre-registration and
commitment points), [`datasets-benchmarks.md`](datasets-benchmarks.md) (what a benchmark entry should
record).

## At a glance

| You are about to… | Worth checking | Because |
|---|---|---|
| state what your result generalises to | which unit the claim is about, and how many of that unit you have | the unit sets both the split and the error bar, and a chromosome hold-out is not a homology hold-out |
| pool labels from several assays or databases | whether the annotation was itself assigned by sequence similarity, and what condition it was measured under | a model can pass by approximating the rule that produced the labels |
| report a score from a random train and test division | whether relatives, gene families or repeated positions sit on both sides, which is homology leakage | the drop to an identity-aware division is larger than the model-versus-baseline gap being claimed |
| claim a pre-trained model beats the alternatives | which class of baseline you are beating, and whether it was tuned as hard as the model | profile and alignment-based baselines match or beat sequence-only models in most strict-split comparisons, while single-sequence search and composition baselines are often clearly beaten |
| pick a layer, a pooling rule or a likelihood score | whether that choice was made on the data you are about to report | the readout (that combination of choices) moves results by tens of points, and choosing on the reported cross-validation folds cost 12 points of accuracy on average on data built to carry no signal, and more than 20 points in a fifth of runs |
| call a result zero-shot | whether the evaluation sequences were in the pre-training corpus | contamination has changed a scaling conclusion, not only a score |
| adopt a public benchmark as a fixed yardstick | how its tasks were selected | several kept only tasks where existing models scored in a chosen band |
| say a model shows no signal on a task | a planted positive control and a smallest effect stated in advance | without them, absence of signal and failure to elicit look identical |
| read a claim that the model generated something functional | the funnel: generated, filtered, screened, human-selected, assayed, worked | one published pipeline filtered about 2 million generations down to 11 laboratory candidates, with a manual last step |
| put a confidence interval on an AUROC or a rank correlation, or an error bar on a benchmark number | what unit it resamples, and how many groups there are | the inflation scales with group size, so a family of ten inflates little and a chromosome of thousands inflates a lot |
| report a winning configuration | how many configurations were tried to find it | nothing in this literature corrects for the count, and most headlines are a maximum over readings |
| trust the numbers at all | that the checkpoint loaded, and in the right numerical precision | one released checkpoint loads as random weights with only a warning |

## Why this differs from a chat-model eval

Three differences are real, and two of them run in the field's favour.

Labels are cheap relative to the thing being measured, so cross-validation genuinely replaces a single
spent test set for most questions. A chat benchmark is consumed by being used; a sequence benchmark can
be rotated.

Leakage is measurable in advance. Sequence identity between the training and test sides is computable
before a model runs, unlike contamination of a chat benchmark, which is usually argued rather than
measured.

Relatedness is the dominant structure. Items share ancestry, and in a pair task one gene can appear in
many pairs. This has no chat-model analogue, and the strongest practice in the literature comes from
it: holding out a whole clade or species and reporting what that costs.

## Which steps apply to you

The steps below are written for several kinds of experiment, and some do not apply to yours. Read this
row first and skip the rest.

| Your setup | Steps that apply | Steps to skip |
|---|---|---|
| Amino-acid model, read with a probe on embeddings | 1, 2, 3, 4, 5, 6, 7, and the no-signal checks | the likelihood part of step 3; the reverse-complement part of step 5; the generated-sequences section |
| Nucleotide model, read with a probe on embeddings | all of them | the likelihood part of step 3; the generated-sequences section |
| Any model, scored by likelihood with no training | 1, 2, 4, 5, 6, 7 | step 3's layer and pooling parts (there is no probe to tune) |
| Generative claim ("the model designed something that works") | 1, 5, and the generated-sequences section | steps 3 and 6, which are about a fitted readout |

## Step 1: Name the unit your claim generalises over

This decides the statistics rather than following from them, and it is the choice most often left
implicit. A claim generalises over some unit, and the error bar and the split both have to use that
unit. The units actually used in this literature, in rough order of how often they appear:

| Unit | Used when | Where it appears |
|---|---|---|
| Sequence identity cluster | the default for protein work, and the only unit that stops a near-copy crossing the split | identity thresholds with MMseqs2, CD-HIT, UniRef clusters |
| Gene, protein or RNA family | the claim is about a new family rather than a new member | Rfam family splits in RNA structure; paralogue-group splits |
| Chromosome | the commonest unit in DNA work, and cheap to apply | held-out chromosomes in nearly every genome-model paper |
| Position within a protein | mutational scanning, where positions recur across items | held-out-position splits in fitness benchmarks |
| Assay or dataset | many small experiments, each with its own scale and noise | per-assay averaging in variant-effect suites |
| Time of deposition | the strongest unit available, because nothing later could leak | function-prediction challenges; structure targets released after a freeze |
| Organism, genus or clade | multi-species models, or a claim about an unseen lineage | a minority of papers, almost all multi-species DNA models |

**A chromosome hold-out is not a homology hold-out.** It is the most common split in DNA work and the
easiest to over-trust: paralogues sit on other chromosomes, so a held-out chromosome can still contain
near-copies of training sequence. Report both the chromosome split and an identity-aware split if the
claim is about new sequence rather than new loci.

Roberts et al. 2017 (doi:[10.1111/ecog.02881](https://doi.org/10.1111/ecog.02881), "Cross-validation
strategies for data with temporal, spatial, hierarchical, or phylogenetic structure") states the
dependence in both directions: blocking "is required" when the goal is to predict "into new dependence
structures (i.e. new locations, new time periods, new individuals or groups, or for new species within
a phylogeny)", and "when models are intended only to predict within the same spatial and
temporal ranges or on the same individuals or groups by which they were trained, random
cross-validation may yield fair error estimates because the model's conditionals do not change".

**Recommended check.** Write the claim as one sentence before building anything, name the unit it
generalises over, and count how many of that unit you have. The count matters as much as the choice:
step 6 explains what a small one does to an interval, and a unit with thousands of members per group
behaves very differently from one with ten. See
[`../engineering/research-rigor.md`](../engineering/research-rigor.md) for the commitment points.

## Step 1b: Say where the labels came from

A benchmark can be passed by a model that approximates whatever produced the labels. Two cases matter
here. First, **label circularity**: many functional annotations were themselves assigned by sequence
similarity, so predicting them from sequence partly recovers the assignment rule rather than the
biology. Second, **assay heterogeneity**: labels pooled from different experiments carry the experiment
with them, and when the experiment is confounded with the group being held out, a cross-group result is
partly a cross-experiment result. Essentiality, expression and fitness are all condition-dependent, so
the condition is part of the label.

**Recommended check.** Before splitting, record for each label: the assay or database that produced it,
whether that assignment used sequence similarity, and what condition it was measured under. Count
ambiguous or intermediate labels and say how they were handled rather than folding them into the
negative class. Where several assays contribute, report the metric per assay as well as pooled, since
class balance and noise differ between them. Most biological label sets are strongly imbalanced, so
report the area under the precision-recall curve (AUPRC) beside the area under the
receiver-operating-characteristic curve (AUROC); see
[`eval-statistics.md`](eval-statistics.md) for which one resolves what.

## Step 2: Divide the items by the unit that shares information

A random division puts near-copies on both sides, so the model can score by recognising a relative
rather than the property. The looser division is not an optimistic estimate of the same quantity; it
measures an easier task.

Published case: Dallago et al. 2021 (doi:[10.1101/2021.11.09.467890](https://doi.org/10.1101/2021.11.09.467890),
"FLIP: Benchmark tasks in fitness landscape inference for proteins") report ESM-1b at Spearman 0.92 on
one landscape with a random division and 0.28 when the training side holds only single mutants.
Bernett, Blumenthal and List (doi:[10.1101/2023.01.18.524543](https://doi.org/10.1101/2023.01.18.524543),
"Cracking the black box of deep sequence-based protein-protein interaction prediction", preprint; journal
version in Briefings in Bioinformatics 2024 not opened) find published
protein-protein interaction accuracies of 77 to 98 per cent fall to 56 per cent against a 50 per cent
chance baseline once test proteins share no similar sequence with training proteins. Penić et al. 2025
(doi:[10.1038/s41467-025-60872-5](https://doi.org/10.1038/s41467-025-60872-5), RiNALMo) score RNA-FM at
F1 0.676 on a random division of RNA secondary structures and about 0.30 averaged over nine held-out
families (0.30 in the main figure, 0.31 in a supplementary table, which disagree per family), below
RNAstructure, a thermodynamic method with no training, at 0.587. The same table is a counter-case for a
different model: fine-tuned RiNALMo "outperforms RNAstructure and CONTRAfold in eight out of nine
families by high margins", so the family split separates models rather than condemning all of them. The
leave-one-family-out division is from Szikszai et al. 2022
(doi:[10.1093/bioinformatics/btac415](https://doi.org/10.1093/bioinformatics/btac415)).
Lüftinger et al. 2021 (doi:[10.3389/fcimb.2021.610348](https://doi.org/10.3389/fcimb.2021.610348)) find
random cross-validation folds give balanced accuracy 4.45 points higher than genome-distance-aware
folds, significantly so in 60 of 77 organism and compound pairs, and, for one species, a separately
collected set ran 10.12 points below the random-fold estimate. (Those models are k-mer gradient-boosted
trees rather than sequence language models; the point is about the split, not the model.)

The cleanest demonstration is one model against itself. Two versions of the Bacformer preprint
(doi:[10.1101/2025.07.20.665723](https://doi.org/10.1101/2025.07.20.665723)) share a single identifier
and reach opposite conclusions on gene essentiality. Under the first version's random 60/20/20
division, Bacformer beats ESM-2 and Evo. Under the third version's genus-level division, four other
models beat it: ProtBERT 82.45, ESM-C 81.18, gLM2 80.57 and ESM-2 80.40 against Bacformer 79.49, all
area under the receiver-operating-characteristic curve (AUROC) as percentages over n = 11 genomes. The
third version's main text does not state the reversal, so a citation needs a version pin.

The opposite error exists. Roberts et al. 2017 show blocked folds can be "overly-pessimistic", because
a held-out clade becomes an extrapolation rather than a held-out sample.

**Recommended check.** Cluster items by the unit that shares information (a sequence identity
threshold, a gene or RNA family, the position within a protein, the organism, the clade) and confirm no
cluster spans the division. Report the random-division and the aware-division numbers side by side: the
gap is the leakage, and it is a result worth stating. Where a chromosome hold-out is used, note that it
is not a homology hold-out, since paralogues sit on other chromosomes. The generic form of this rule is
on [`../engineering/measurement-validity.md`](../engineering/measurement-validity.md) and
[`../interpretability/probes.md`](../interpretability/probes.md#pitfalls-cross-cutting); they are not
repeated here.

## Step 3: Choose the readout (layer, pooling rule and scoring rule) on data you will not report

A frozen model does not have one representation. Which layer, which pooling rule, whether the model is
fine-tuned, and which likelihood score is used are hyper-parameters of the measurement. Two failure
modes appear in the literature, and the two model families pick one each.

Published case, on the size of the lever: Dalla-Torre et al. 2025
(doi:[10.1038/s41592-024-02523-z](https://doi.org/10.1038/s41592-024-02523-z), Nucleotide Transformer)
report that "the highest model performance is never achieved by using embeddings from the final layer"
and a relative difference "as high as 38%" between the best and worst layer on one task, over roughly
760,000 probes fitted. Feng et al. 2025
(doi:[10.1038/s41467-025-65823-8](https://doi.org/10.1038/s41467-025-65823-8)) find averaging over
tokens beats the model's own summary token on 35 to 42 of 52 classification datasets per model, worth
a mean gain in the area under the receiver-operating-characteristic curve (AUROC) that the paper states
as 1.4 to 8.7 per cent depending on the model (its own
framing; the largest single case moves the curve from 0.689 to 0.864), and narrowing the spread between models
from 0.708-0.799 up to 0.795-0.822. Vieira et al. 2025
(doi:[10.1038/s41598-025-05674-x](https://doi.org/10.1038/s41598-025-05674-x)) find mean pooling beats
four alternatives by 5 to 20 percentage points of variance explained on mutational-scanning data.
Valeriani et al. 2023 (arXiv:[2302.00294](https://arxiv.org/abs/2302.00294)) find remote-homology
information in ESM-2 peaks in middle layers at about 0.8 nearest-neighbour overlap against 0.4 in the
last layer; Vieira et al. 2025 find the opposite ordering for one probe type on 40 mutational-scanning
sets, where the last or penultimate layer was most often best. So there is no universally best layer,
only a choice that must be made somewhere.

Published case, on where the choice is made: Schiff et al. 2024
(arXiv:[2403.03234](https://arxiv.org/abs/2403.03234), Caduceus) state a classifier regularisation
"selected from {1, 5, 10} by evaluating average test set AUROC". Brixi et al. 2026
(doi:[10.1038/s41586-026-10176-5](https://doi.org/10.1038/s41586-026-10176-5), Evo 2) is the
counter-example worth copying: layer, pooling window and penalty are chosen jointly by five-fold
cross-validation inside the training division, then the test set is scored once.

This page's synthesis, not any one paper's: across the thirteen protein model papers read, exactly one
compares pooling choices at all, and most take the final layer with mean pooling without reporting a
comparison. The DNA
papers search exhaustively and report the maximum. So the DNA literature risks selection bias and the
protein literature risks an unexamined default, and the default the protein papers pick is the final
layer, which the layer studies above find weakest for homology information.

**Recommended check.** Treat the readout as a hyper-parameter: select it inside the training folds,
record which setting won in each fold, and report the spread across settings. Unstable winners across
folds are themselves evidence that the selection is fitting noise. Do not take a repository quickstart's
layer as the paper's recommendation without checking the paper; one model card extracts the final layer
its own paper calls worst. For the selection-bias mechanism see
[`../engineering/statistics.md`](../engineering/statistics.md) and step 7 below.

## Step 4: Fit a baseline that uses no model, and name which class it is

Composition at k-mer resolution, homology search, alignment or profile methods, and a small network
trained from raw sequence each capture part of what these benchmarks score. The result turns on **which
class of baseline** you compare against, and reports that omit the class are not interpretable.

**What the comparisons read for this page actually show.** Profile- and alignment-based baselines
(hidden Markov model profiles, profile search, multiple-sequence-alignment methods, conservation scores)
matched or beat sequence-only language models in most strict-split comparisons. Single-sequence homology
search (BLASTp, MMseqs2), k-mer or one-hot composition, and random-weight twins were sometimes clearly
beaten. A 2026 review of genomic language models (Veiner and Supek, *Molecular Systems Biology*)
concludes that most studies find these models more accurate than simple supervised baselines, and that
the harder contest is against specialised expert models, where margins are small or absent. So the
honest framing is not that trivial baselines win, but that the margin is usually much smaller than the
headline and depends on the comparator.

**Cases where a no-model baseline matched or beat the model:**

Sanabria et al. 2024 (doi:[10.1038/s42256-024-00872-0](https://doi.org/10.1038/s42256-024-00872-0),
GROVER) report a tuned bag-of-k-mers control (term-frequency weighting with a random forest) reaching a
Matthews correlation coefficient (MCC, scaled as a percentage, as that paper reports it) of up to 87 on
promoter tasks and 72 on a transcription-factor binding task against 61 for Nucleotide Transformer; that
paper also reports tasks where the language models win, and splice-site prediction is the clearest.
Nguyen et al. 2024 (Evo, cited above) report that on promoter activity GC content alone reaches
correlation 0.35 against the 7-billion-parameter model's 0.43, and a mechanistic calculator reaches
0.62. Yang et al. (arXiv:[2406.15669](https://arxiv.org/abs/2406.15669), "CARE", NeurIPS Datasets and
Benchmarks; numbers from the current version) report BLASTp at 93.7 against the ESM-based classifier's
69.4 on the promiscuous split at the full four-digit enzyme number — but that split's test sequences
have high identity to the training set and the two methods are scored under different rules there, so it
is not a homology-held-out comparison. Kedzierska et al. 2025
(doi:[10.1186/s13059-025-03574-x](https://doi.org/10.1186/s13059-025-03574-x)) find 2,000 highly
variable genes separate cell types better than two single-cell foundation models, zero-shot, and Wong et
al. 2025 (doi:[10.1093/bioinformatics/btaf317](https://doi.org/10.1093/bioinformatics/btaf317)) find
scGPT from random weights matching the pre-trained model, difference 0.004, p = 0.89. Zhang, Bai and
Imoto 2023 (doi:[10.1093/bioinformatics/btad617](https://doi.org/10.1093/bioinformatics/btad617)) give
the null-corpus version at one remove: a small network fed only the k-mer embedding table of a DNABERT
pre-trained on 3 gigabases of random sequence reaches median area under the curve 0.8965 against 0.8991
for the table from the human-genome model, across 690 datasets. The fine-tuned models themselves differ
by more than that, 0.9 against 0.8624, so the embedding table carries almost none of the difference and
the fine-tuning carries it. Vishniakov et al. 2026, "Tokenization to Transfer: Do Genomic Foundation
Models Learn Good Representations?" (ICLR 2026; preprint
doi:[10.1101/2024.12.18.628606](https://doi.org/10.1101/2024.12.18.628606), whose first two versions
carry an earlier title) report a randomly initialised Caduceus reaching about 0.62 mean MCC on the
Nucleotide Transformer histone and enhancer tasks, above pre-trained Nucleotide Transformer 500M,
Nucleotide Transformer v2 50M and GENA-LM. The same paper reports the reverse for other models, "a
near-consistent trend of pretraining helping models trained using BPE or k-mer tokenizer, with the
extent of gain ranging from 0.05-0.25 MCC", so the untrained twin is a control to run, not a result to
expect.

**Cases where the pre-trained model clearly beat the no-model baseline, on identity-separated tests:**

On CARE's under-30-per-cent-identity split the ESM-based classifier is ahead of BLASTp at every enzyme
level, by 3.7 to 18.8 points (84.5 against 65.7 at the coarsest level), and a structure-search baseline
with no language model is the best method on one of the splits. In the production tools built on these
models, which all separate train and test at a stated 20 to 30 per cent identity threshold, tuned
non-language-model baselines lose on the same data: a retrained profile-based predecessor reaches
macro-F1 0.47 against 0.64 and 0.66 for the language-model version (Thumuluri et al. 2022, DeepLoc 2.0),
and a tuned substitution-matrix network reaches area under the curve 0.641 against the language model's
higher score. For RNA secondary structure under the leave-one-family-out division this page uses, one
pre-trained model loses to an untrained thermodynamic method while another beats it in eight of nine
families.

**Recommended check.** Run three baselines on exactly the scored items with the same tuning budget as
the model: composition features, single-sequence homology transfer, and a profile or alignment method if
one exists for the task. Add the same architecture with random weights. State which class each belongs
to beside the number, because "beats the baseline" means little without it. If the margin over all of
them is inside the interval from step 6, the supported claim is that the model is competitive, not that
it wins. The generic rule is on
[`../engineering/measurement-validity.md`](../engineering/measurement-validity.md#trivial-baselines-first);
equal tuning budgets matter because which method wins can flip with budget.

## Step 5: Check the instrument numerically before scoring

Every rule above assumes the numbers coming out of the model are the numbers the model would produce.
Several documented cases say otherwise, and none raises an error.

**Three of these are properties of the models and will not change.** HyenaDNA pads on the left and
ships no attention mask, so padding enters any average. Batch padding changes embeddings wherever
pooling does not exclude it. Two RNA models use opposite uracil and thymine alphabets, so the same
sequence string means different things to each.

**Five more were live on 2026-11-15 and are expected to be fixed, so read them as examples of the
failure class rather than as a current bug list.** One released checkpoint (ESM C 6B, republished
2026-09) loaded as random weights through the loader current in mid-September 2026, with only a warning;
the loader has since been superseded. One bacterial
model (Bacformer) returned wrong outputs with no error under the then-current major version of the
`transformers` library. Evo 1 had to be loaded at a specific revision tag. Vieira et al. 2025, already
cited above, report three of this kind: ESM-2 producing wrong embeddings on Apple Silicon GPUs through
Metal Performance Shaders, ESM C's default half-precision autocast not being reproducible run to run,
and batch padding changing AMPLIFY's embeddings. The checkpoint and library items were checked against
repositories, model cards and issue threads on that date and carry no paper.

Models are also not invariant to transformations biology says they should be. Ma 2025
(arXiv:[2509.18529](https://arxiv.org/abs/2509.18529), "Reverse-Complement Consistency for DNA Language
Models", single-author preprint, not peer-reviewed) reports fine-tuned Nucleotide Transformer v2 and
HyenaDNA changing their predicted class on 4 to 15 per cent of reverse-complemented benchmark sequences
even after reverse-complement augmentation. In that paper averaging the two strands at test time does
**not** recover what augmentation achieves: it trails augmentation on the Matthews correlation
coefficient in most of the comparisons reported, so measure the disagreement rather than assuming a fix.

The model-by-model instances of these, with the repository and issue references, are on
[`../models-and-compute/biological-sequence-models.md`](../models-and-compute/biological-sequence-models.md#loading-traps-that-change-the-numbers),
together with each model's licence and tokenisation.

**Recommended check.** Before any benchmark number: reproduce one published figure from the model's own
paper or card, or at minimum show the forward pass differs from a randomly initialised copy; compare a
sample of embeddings against a single-sequence, float32, processor-only run; and for a nucleotide model,
score each sequence and its reverse complement and report the disagreement rate. Record all three in
the run directory. Keep this check clearly separate in code and in the write-up from the deliberate
random-weight baseline of step 4.

## Step 6: Put the error bar on the unit from step 1

An interval answers "how much would this move if I drew the data again?". If the resampling unit is the
single item while the claim is about new families, new chromosomes or new species, the interval is
describing the wrong draw. The size of the problem scales with how many items sit in a group, which is
why this matters much more for some units than others.

**How much it matters, as arithmetic.** With a design effect written 1 + (m - 1) * rho, where m is the
group size and rho the within-group correlation: a gene family of 10 members with a strong rho of 0.1
gives about 1.9, while a chromosome or a whole genome holding thousands of genes gives a large number
even at a rho of 0.001. So an identity-cluster or family split, which is the common case, inflates the
variance modestly; a chromosome or organism split, which is rarer, inflates it a lot. Compute it rather
than assume it either way.

**What the field does.** Group-level uncertainty is computed, though rarely compared against the
item-level alternative. TraitGym calculates the area under the precision-recall curve (AUPRC) per
chromosome for a model trained on the remaining chromosomes, then a sample-size-weighted average with a
cluster-resampled standard error across at most 22 or 23 chromosomes, for a set of genomic language
models. Vieira et al. 2025 fit linear mixed-effects models treating the dataset as a random effect.
What no source read here reports is an intraclass correlation or a design effect for a sequence
benchmark, or a side-by-side comparison of a group-level interval with an item-level one, so the size
of the gap is undocumented rather than unmeasurable.

**Published case:** Cameron, Gelbach and Miller 2008
(doi:[10.1162/rest.90.3.414](https://doi.org/10.1162/rest.90.3.414)) find cluster-robust tests
over-reject with few (5 to 30) clusters. Bates, Hastie and Tibshirani
(arXiv:[2104.00673](https://arxiv.org/abs/2104.00673)) find naive cross-validation intervals undercover,
31 per cent miscoverage at a nominal 10 per cent. Obuchowski 1997 (PMID 9192452, abstract only) gives
the clustered form for an area under the receiver-operating-characteristic curve (AUROC) and states that
"the size of statistical tests that assume independence is inflated in the presence of intracluster
correlation". Felsenstein 1985 (doi:[10.1086/284325](https://doi.org/10.1086/284325)) is the origin of
the effective-sample-size point for related species.

**Recommended check, by how many groups you have.** The simplest analysis that works is to compute the
metric separately per group, then report the mean with a t interval on G - 1 degrees of freedom.

| Groups | What to use |
|---|---|
| 30 or more | a cluster-resampled standard error, or a bootstrap over groups |
| about 6 to 30 | the per-group metric with a t interval, or a sign-flip permutation test |
| fewer than 6 | report per-group numbers and call the result descriptive |

Three cautions. An item-level interval describes only the items you scored under the split you fitted,
and supports no statement about further items, which is the distinction step 2 rests on. When the claim
is a comparison, compute everything on the paired per-group difference rather than on either arm. And
the design effect is itself an estimate on roughly G - 1 degrees of freedom, so a value near 1 is weak
evidence of adequacy rather than proof of it, and is only readable from roughly 20 to 30 groups upward.

For an AUROC the two variances need named estimators rather than a generic clustered standard error of
a mean: DeLong's components for the item-level denominator, and the same components summed within groups
for the numerator. Unequal group sizes bias the ratio low, and the weighted group size is the sum of
squared sizes over the sum of sizes, not the arithmetic mean. The design effect itself is defined on
[`../engineering/statistics.md`](../engineering/statistics.md#clustered-items) as
(SE_clustered / SE_plain)^2 and used there to inflate a required sample size; here it is used the other
way, to ask whether an interval already computed was adequate. For the clustered machinery and the
resolution question see
[`eval-statistics.md`](eval-statistics.md#clustered-items-passages-images-problem-sheets-and-shared-tasks).

## Step 7: Report the size of the search

Published case: across the papers read for this page, corrections for multiple testing appear in a
handful and almost never for the comparison that carries the claim. One protein paper corrects, and only
for the choice of pooling; a few RNA and single-cell papers correct within a differential-expression
step rather than across models or tasks; one genomic-model paper corrects a model-versus-model probe
comparison. Hundreds of task-by-model comparisons elsewhere carry none. Varma and Simon 2006
(doi:[10.1186/1471-2105-7-91](https://doi.org/10.1186/1471-2105-7-91)) show the cost directly: on
40-sample data with no difference between classes (true error 50 per cent), choosing the setting that
minimised the cross-validated error and reporting that error gave 37.8 per cent, while repeating the
selection inside every fold recovered 54.2 per cent. Cawley and Talbot 2010 (JMLR 11:2079-2107) find
this bias "larger than the typical difference in performance between classifiers evaluated using an
unbiased protocol", and that under the biased protocol a significance test declared a winner where "the
difference in performance is entirely spurious".

**Recommended check.** Record the full grid examined (models, readouts, layers, poolings, classifier
settings) and report its size N beside the final number, with the statement that the configuration is
the maximum of N. Then run the label permutation on the whole pipeline, search included: shuffle the
labels, re-run the selection, and report the best score obtained. Anything clearly above chance is this
search's selection bias at this data size. A specification curve reports the distribution over the
whole grid instead of its maximum.

## Reading a claim about generated sequences

Some of these models generate as well as score, and a generation result is reported as "the model
produced something that works". This section is about how to read or audit that claim. It does not
cover how to run a generative design pipeline, which is out of scope for this wiki.

Three things decide whether the claim means anything, and the strongest published example reports all
three.

**The funnel.** How many candidates were produced, what filtered them, and how many were tested.
Nguyen et al. 2024 (doi:[10.1126/science.ado9336](https://doi.org/10.1126/science.ado9336), Evo) state
it in the open: they "filtered ~2 million Evo-generated sequences" for loci containing the required
domains and a detectable guide RNA, folded "the top-ranking 2000 generations", and selected 11
candidates for laboratory testing, where the last step was "manual inspection". One of those 11 worked
in vitro. A paper that reports the one success without the 2 million and the manual step is reporting a
different number.

**The comparator in the same assay.** Evo compares the generated nuclease with the natural enzyme run
in the same assay, and for a second system tests "the natural IS200 element ISSpn6 and IS605 element
ISHp608" as controls.

**A mechanism control.** A catalytically dead variant, or a dependence the molecule should show if it
works for the stated reason. Without one, activity can come from something other than the designed
function.

**The failure mode to look for: filtering before judging.** Zvyagin et al. (GenSLM) excluded the most
divergent 1.4 per cent of generated sequences before the phylogenetic comparison that was then used to
argue the generations were realistic, which makes the comparison partly a property of the filter.

**Recommended check.** For any generation claim, extract the funnel as a single line of numbers
(generated, machine-filtered, structurally screened, human-selected, assayed, worked) and put it beside
the headline. Ask whether the comparator ran in the same assay, whether a mechanism control was
included, and whether any filter was applied to the generations before the evidence step. For the
hazard-assessment reading of a generative biological model, see
[`dangerous-capability-evals.md`](dangerous-capability-evals.md#bio-and-chem-vct-lab-bench-and-what-benchmarks-miss)
and [`../alignment-science/unlearning.md`](../alignment-science/unlearning.md).

## Recommended checks before a claim that a model carries no signal

Absence of signal and failure to elicit look identical from one number.

Published case: Hewitt and Liang 2019 (arXiv:[1909.03368](https://arxiv.org/abs/1909.03368)) define the
control task, in which a random label is drawn once per *type* of input and every occurrence of that
type then carries it, so the task is learnable only by memorising type identity. That is why a probe can
reach 71 to 93 per cent on it, and why it is not the same thing as shuffling labels item by item: a linear
part-of-speech probe scores 97.2 per cent accuracy against 71.2 per cent on the control task, so a
selectivity (the real score minus the control score) of 26.0, while a more expressive probe scores 97.3
against 92.8, so a selectivity of only 4.5. A search of the protein and DNA sources
read for this page found no use of control tasks, description-length probing or whole-pipeline label
permutation, so this is a method the neighbouring field has and this one does not yet.

**Recommended check.** Two different controls, and they answer different questions. A per-item label
shuffle tests whether the pipeline leaks, and should land at chance. A control task in the sense above,
where the random label is fixed per type (per gene family, per domain), tests whether the probe is
strong enough to fit arbitrary structure, and its score is subtracted from the real one to give
selectivity. Report whichever you ran, named correctly, at the same probe capacity beside the real one,
and a planted positive control showing the pipeline finds a signal you injected. State a smallest
effect of interest in advance and test equivalence against it, rather than reading a non-significant
difference as absence. See
[`evals.md`](evals.md#sandbagging-and-capability-elicitation-is-a-low-score-real) for the chat-model
form of the same problem.

## Cross-cutting pitfalls

**A public benchmark may have been filtered using the models it now scores.** Zhou et al. 2024
(arXiv:[2306.15006](https://arxiv.org/abs/2306.15006), DNABERT-2 and its GUE benchmark) kept datasets
where existing models scored in a chosen performance band and restructured the rest, and the model
released alongside had its tokeniser chosen on that same benchmark. Notin et al. 2023 (ProteinGym,
NeurIPS Datasets and Benchmarks) removed thermostability assays "where none of the tested evolutionary
models had a Spearman correlation above 0.2". A score on such a benchmark is not comparable to a score
on an independently built task, and the benchmark cannot carry a headline claim about the model family
that shaped it. Record the filter in the benchmark's entry, as
[`datasets-benchmarks.md`](datasets-benchmarks.md) does for other suites.

**Zero-shot usually means unseen in fine-tuning, not unseen by the model.** These models are pre-trained
on reference genomes and sequence databases, so evaluation sequences are commonly inside the corpus,
unlabelled. The consequence can reach past the score. Hermann et al., "Beware of data leakage from protein LLM
pretraining" (Proceedings of Machine Learning Research 261:106-116, Machine Learning in Computational
Biology 2024; preprint
doi:[10.1101/2024.07.23.604678](https://doi.org/10.1101/2024.07.23.604678)) re-ran the FLIP
melting-temperature task on a pre-training-aware division. All seven ESM variants score higher on the
leaky division, for example ESM-1b 0.620 against 0.680 and ESM2 650M 0.523 against 0.618 in Spearman
correlation, and the step from 3 billion to 15 billion parameters gains nothing on the clean division
(0.674 against 0.670) while gaining on the leaky one. Read as a caution rather than a settled result:
the paper reports one run per model with no seeds or intervals, and the two divisions differ in more
than leakage. For a model pre-trained on a near-exhaustive database, overlap reporting is a formality: the honest
position is that the sequences are presumed seen and only the **labels** are unseen, and the claim
should say so in those words. The designs that recover a genuinely unseen evaluation are time-based
(sequences deposited after the checkpoint was frozen) or a collection deliberately withheld from all
pre-training, both of which are in the list of what this field does better, below.

**A negative set is manufactured, and its construction defines the task.** Zhang, Bai and Imoto 2023
(cited above) show task difficulty moving with the construction when negatives are built by matching
composition rather than by shuffling. Say which construction was used before comparing numbers across
papers.

**Likelihood scores are not interchangeable.** Meier et al. 2021
(doi:[10.1101/2021.07.09.450648](https://doi.org/10.1101/2021.07.09.450648), ESM-1v, NeurIPS; preprint
version read) report mean Spearman 0.582 for the masked
marginal, 0.572 for the one-pass wild-type marginal and 0.552 for the pseudo-log-likelihood on the same
task, so the choice is worth about 0.03 and must be fixed in advance rather than picked from the menu
after seeing the correlations.

## Reproducibility checklist

- [ ] The claim is written as one sentence, naming the population it generalises over.
- [ ] Items are clustered by the unit that shares information, and no cluster spans the division.
- [ ] Both the random-division and aware-division numbers are reported.
- [ ] The readout (layer, pooling, likelihood score) was selected inside the training folds only.
- [ ] A composition baseline, a homology-transfer baseline and a random-weight twin are in the headline table.
- [ ] The checkpoint was shown to be loaded, and precision and padding were checked against a reference run.
- [ ] For a nucleotide model, the reverse-complement disagreement rate is reported.
- [ ] The interval's resampling unit matches the claim, with the design effect and the number of groups.
- [ ] The size of the configuration search, N, is reported beside the final number.
- [ ] A shuffled-label run of the whole pipeline, and a planted-signal run, are both reported.

## Cross-references

The pages this one builds on are listed under Related pages at the top. Two neighbours are specific to
biology and easy to miss:

- Scoring a chat or agentic model for hazardous biological capability, and the two uplift trials:
  [`dangerous-capability-evals.md`](dangerous-capability-evals.md)
- Safeguards on biological models themselves (pre-training data filtering, unlearning):
  [`../alignment-science/unlearning.md`](../alignment-science/unlearning.md)

---

## Common questions

### Can I just use cross-validation instead of holding out a test set for a biological benchmark?

For most questions yes, and this is a real difference from chat-model work: labels are cheap relative to
assays, so the item set can be rotated rather than spent. The condition is that every label-touching
choice, the readout included, is repeated inside each fold. Cross-validation with the layer chosen once
on all the data estimates nothing useful.

### My held-out group count is small. How small is too small for a group-level interval?

Cluster-robust tests over-reject with 5 to 30 clusters. From about 6 groups up, the per-group metric
with a t interval is the simplest thing that holds its coverage. Below that, report per-group numbers
and call the result descriptive, because a two-sided sign test on four groups cannot return a p-value
below 0.125 whatever the effect size. Note that the number of groups is not the number of items: a
hundred thousand genes in four groups is a four-group problem.

### Which layer should I use for a protein or DNA model?

There is no single answer in the literature: remote-homology information peaks in middle layers of ESM-2,
the final layer is never best in Nucleotide Transformer, and the last or penultimate layer was most often
best for one probe type on 40 mutational-scanning sets. Choose it inside the training folds and report
the spread. The one thing to avoid is inheriting a repository quickstart's layer without checking it
against the paper.

### Does a benchmark win mean the model understands the biology?

Not on its own, and the answer depends on the comparator. Against a profile or alignment method, or a
conservation score, these models often fail to win. Against single-sequence homology search, composition
features or a random-weight twin, they frequently do win on identity-separated tests. Run all of them,
say which class each is, and the claim that survives is narrower and worth more.

---

Last verified: 2026-11-15. (Additions 2026-11-15: a pointer from step 5 to the model-by-model loading traps and licences on `models-and-compute/biological-sequence-models.md`.) Last verified: 2026-11-15. New page, from a structured read of eleven DNA model papers, thirteen protein
model papers, the RNA, codon and single-cell literature, eight benchmark suites, about twenty independent
re-evaluations and the validation and statistics primary sources, each evaluation task recorded against
eleven fixed columns with a verbatim quote. Two independent reviews then ran against it, one on
citations, numbers and coverage and one on utility, retrieval and durability; this version is the result.

Corrections made in response, each traced to a source: the reverse-complement flip-rate result moved
from Feng et al. to Ma 2025 (arXiv:2509.18529), and its test-time-averaging clause reversed, because
averaging trails augmentation in that paper's own table; the null-corpus DNABERT comparison restated,
because 0.8965 against 0.8991 compares small networks fed each model's k-mer embedding table and not the
models, which differ 0.9 against 0.8624; Vishniakov et al. now cited at its ICLR 2026 version, with its
own finding that pre-training helps byte-pair and k-mer tokenised models by 0.05 to 0.25 Matthews
correlation; Hermann et al. cited at its proceedings version (PMLR 261:106-116) with per-model numbers in
place of a mean that did not match its rows, and read as a caution since it reports one run per model;
the Apple-GPU, half-precision and padding items attributed to Vieira et al. rather than called
paper-less; the ESM C 6B loading failure dated to the loader current in mid-September 2026; the control
task defined correctly (a random label per input type, not per item) and distinguished from a per-item
shuffle; the 12-point optimism stated as a mean with its tail; the layer and pooling count reduced to
what a strict reading supports; the multiplicity claim narrowed; the RNA secondary-structure numbers
carrying the disagreement between that paper's own two tables, with its counter-case stated and Szikszai
et al. 2022 identified; the Roberts quotation restored verbatim; an uncited construction of negative sets
dropped.

The central generalisation was rewritten rather than defended. The page had said that where a
re-evaluation added a composition or homology baseline it matched or beat the pre-trained model in every
model family examined. Counter-examples exist in every family: on the under-30-per-cent-identity enzyme
split the language-model classifier leads by 3.7 to 18.8 points; the production tools built on these
models beat tuned non-language-model baselines on identity-separated tests; one RNA model beats an
untrained thermodynamic method in eight of nine held-out families; and a 2026 review concludes that most
studies find these models more accurate than simple supervised baselines, the harder contest being
against specialised expert models. The claim is now about the class of comparator, which is what the
evidence supports.

Limits, stated rather than left to the reader. The counts are over the papers read for this page, not
over the field. The most-cited production tools (SignalP, DeepTMHMM, DeepLoc, NetSurfP, BepiPred) were
read only for their split methodology during review, and all separate train and test at a stated 20 to
30 per cent identity threshold, so that subfield has stronger split discipline than the model papers do.
Quotations were checked against the saved text of the paper each is attributed to. Read at abstract level
only: Obuchowski 1997 (PMID 9192452), whose quoted sentence is from that abstract. Cited at the version
read rather than the version of record, and so stated: Bernett, Blumenthal and List; Meier et al.; Bates,
Hastie and Tibshirani; Cameron, Gelbach and Miller. Not verified, and stated on the page rather than
asserted: whether within-group correlation is large enough to change a benchmark interval. One-line
summaries are Claude's wording, not the authors'. Drafted by Claude; pending MATS research-staff review.
