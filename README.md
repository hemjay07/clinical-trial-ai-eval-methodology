# Clinical Trial AI Evaluation — Methodology Notes

[![DOI](https://zenodo.org/badge/DOI/10.5281/zenodo.21049650.svg)](https://doi.org/10.5281/zenodo.21049650)



A methodology note from designing one high-fidelity adversarial evaluation case for AI systems doing cross-document clinical trial operations work. Documents the design approach, validation results, and three corrections caught during development.

The eval was contributed to the [Raycaster](https://doc.raycaster.ai) Expert Network, a program where domain experts build high-fidelity benchmark scenarios for AI evaluation. The specific evaluation content (artifacts, finding lists, rubric criteria, scoring keys) is not public. This repository documents the methodology that produced it.

## What this is

This is a methodology note, not a benchmark. The benchmark is the larger eval program this case was contributed to. The single contributed scenario evaluates a clinical research associate's reasoning across thirty-plus artifacts spanning three trial sites and a sixty-day protocol-amendment implementation window, with four output tasks and forty-eight scoring criteria across four classes.

What this repository contains: the design principles, the patterns that make the eval discriminate, the validation approach, and the three corrections that materially changed the final version during development. What it does not contain: the eval itself.

## Contents

| File | Contents |
|---|---|
| [`methodology.md`](methodology.md) | The main artifact: ~3,000-word methodology note covering design moves, validation, and three corrections during development |
| [`ATTRIBUTION.md`](ATTRIBUTION.md) | AI assistance disclosure and acknowledgments |
| [`CITATION.cff`](CITATION.cff) | Machine-readable citation metadata |
| `README.md` | This file |

## How to read

Start with [`methodology.md`](methodology.md). It is the substantive content. The other files exist to support attribution and citation.

If you are short on time, three sections of `methodology.md` carry most of the weight: *Three design moves* (what makes the eval discriminate above baseline), *Three corrections during development* (what changed and why), and *Validation* (what the eval produced and the limitations of the validation).

## On the publication constraint

This document respects a constraint set by Raycaster: methodology only, no eval content. That means no specific findings, no rubric criterion text, no scoring keys, no patient identifiers, no trial pseudonym. The design patterns, decision rationale, and refinement arc are described at a level of abstraction that supports reuse by other evaluation designers without revealing what the eval grades.

If a specific pattern or decision sounds underspecified, that is usually intentional. The goal is general transferability of the methodology, not replication of the eval.

## Why this exists

Two reasons.

Benchmark methodology papers report what works. They rarely report what failed in the design phase, what assumptions turned out to be wrong, or what the iterative correction arc looked like. That asymmetry leaves a gap: people building new evaluation cases relearn the same lessons in private. The three corrections documented in `methodology.md` generalize beyond clinical trials. Stable-failure preservation, instance-level prompt leakage, and the eval-data-integrity-as-prerequisite principle show up in any benchmark design context. Naming the patterns makes them reusable.

Second, this is part of an ongoing portfolio of clinical AI evaluation work. A companion project, [ClinicalGuard](https://github.com/hemjay07/clinicalguard), evaluates AI adherence to specific clinical guidelines in deployment contexts. The two methodologies share an underlying voice and a similar commitment to honest reporting.

## Citation

If you find this useful and want to cite it, see [`CITATION.cff`](CITATION.cff) for machine-readable metadata. The persistent DOI is [`10.5281/zenodo.21049649`](https://doi.org/10.5281/zenodo.21049649).

## Author

**Mujeeb Opabode** (Dr. Abdulmujeeb Opabode; [@mujeeb](https://twitter.com/mujeeb)) — MD (University of Ibadan), engineer (Mono, YC W21). Clinical AI evaluation work. Other repositories: [ClinicalGuard](https://github.com/hemjay07/clinicalguard), [Claude Architect](https://github.com/hemjay07/claude-architect).

## License

This documentation is released under [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/). You may share, adapt, and build on this work with attribution.
