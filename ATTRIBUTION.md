# Attribution and Acknowledgments

## AI assistance disclosure

This work was developed in collaboration with Claude (Anthropic) as a thinking partner across the design, drafting, and methodology-refinement phases of the project. The collaboration included:

- Iterative drafting of artifact content, task prompts, and rubric criteria during the design phase
- Verification checks against the artifact bundle during eval validation
- Methodology refinement discussion that surfaced the three iterative corrections documented in `methodology.md`
- Drafting and revision of this public methodology writeup across six versions

All design decisions, clinical judgments, and methodology choices are the author's. The substantive corrections to the eval design — restoring stable-failure criteria, removing prompt leakage, and the eval-data-integrity diagnosis — originated in author-driven pushback questions during the design process. Claude executed against those decisions, drafted candidate content, and surfaced consequences, but the design judgment was human.

The decision to document the corrections publicly is part of an ongoing commitment to honest methodology reporting in clinical AI evaluation work.

## Acknowledgments

**Levi Lian and Anthony Humay** (Raycaster) — for the Expert Network opportunity, for the structural feedback that prompted the multi-task refactor, and for greenlighting this approach-only methodology publication. The eval itself remains with Raycaster; this repository documents only the design approach.

**The clinical AI evaluation literature** — the methodology draws on borrowed principles from prior work, cited in `methodology.md`. The most direct lineages are Shin/Bhat/Ramanathan (Clinical Pharmacology & Therapeutics 2026) for rubric-based LLM evaluation of clinical trial documents, NOHARM for asymmetric concordance methodology, HealthBench for multi-axis rubric design, and the Harvey Labs open-source benchmark format for task structure.

## How to cite this work

See [`CITATION.cff`](CITATION.cff) for machine-readable citation metadata. The recommended citation format is:

```
Opabode, A. (2026). Clinical Trial AI Evaluation — Methodology Notes.
GitHub repository. https://github.com/hemjay07/clinical-trial-ai-eval-methodology
```

A Zenodo DOI is assigned upon first release and should be used in formal citations when available. A more formal preprint covering the methodology arc may follow on arXiv.
