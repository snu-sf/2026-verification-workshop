# 2026 Verification Workshop

Workshop materials for Rocq and CRIS.

## Workshop content

### Day 1: behaviors and contextual refinement

Day 1 follows one continuous question: how does a modular implementation
produce only behaviors permitted by its specification?

Open these files in order:

| Order | File | Question carried through the file |
|---:|---|---|
| 1 | [`day1/lectures/Behavior.v`](day1/lectures/Behavior.v) | What is one observable behavior of a closed interaction tree? |
| 2 | [`day1/lectures/ModuleIntro.v`](day1/lectures/ModuleIntro.v) | How do calls, linking, and compilation produce that closed tree? |
| 3 | [`day1/lectures/RefinementIntro.v`](day1/lectures/RefinementIntro.v) | How do local simulations establish composable contextual-refinement edges? |
| 4 | [`day1/exercises/Optimizations.v`](day1/exercises/Optimizations.v) | How does `ISim` match returns, I/O, a terminating iterator, related local states, and a call into unknown context code? |
| 5 | [`day1/exercises/KVSortedList.v`](day1/exercises/KVSortedList.v) | How can an abstract map be refined by a sorted list whose lookup uses `ITree.iter`? |
| 6 (optional) | [`day1/exercises/KVSortedListAdvanced.v`](day1/exercises/KVSortedListAdvanced.v) | How can sorted-list insertion traverse with `ITree.iter` while maintaining a cursor invariant? |

The three files under `day1/lectures/` are complete guided demos. The two core
starter files under `day1/exercises/` preserve the canonical theorem names and
contain instructor-guided demonstrations and six `STOP` blocks closed with
exactly seven uses of `Admitted`, so they compile before the exercises are
solved. The optional advanced starter adds one `Admitted` proof for iterative
`put`. Replace each `Admitted` with a proof and `Qed`. The matching files under
[`day1/answers/`](day1/answers/) contain completed proofs throughout.

The corresponding presentation is
[`day1/slides/Day1.md`](day1/slides/Day1.md). A generated PDF is provided as
[`day1/slides/Day1.pdf`](day1/slides/Day1.pdf), with the presenter prompts
extracted in
[`day1/slides/Day1-notes.txt`](day1/slides/Day1-notes.txt).

#### Day 1 timing

| Block | Time | Material |
|---|---:|---|
| Theory 1 | 60 minutes | Rocq foundations and interaction trees (professor-led) |
| Theory 2 | 60 minutes | `Behavior.v` through refinement and the first stateless simulation |
| Hands-on block 1 | 120 minutes | optimization exercises, the unknown-call demonstration, then the KV representation relation |
| Break | 30 minutes | |
| Hands-on block 2 | 90 minutes | KV `put`, iterator induction for `get`, guided final assembly, and the optional iter-based `put` extension |

### Day 2: separation logic

Day 2 develops separation-logic specifications and refinement proofs in this
order:

1. [`day2/mem/`](day2/mem/README.md) specifies a memory module with
   `ghost_map` and `mono_nat`.
2. [`day2/stack/`](day2/stack/README.md) verifies a linked-list stack through
   the memory specification.
3. [`day2/map/`](day2/map/README.md) verifies linked-list and binary-search-tree
   maps against one abstract API and includes an open-ended `try/` scaffold.
4. [`day2/refcell/`](day2/refcell/README.md) introduces fractional ownership
   and verifies a dynamically checked reference cell.

Paired `_answer` directories contain the completed exercises. Build all Day 2
starters and answers with `make day2`; `make sl` remains available as an
alias.

### Day 3: prime clients and safe memory

Day 3 uses the shared Imp language and memory modules in `day3/imp_system/`.
The [`day3/prime/`](day3/prime/README.md) problem set verifies an IO-facing
nth-prime client and its linked-list library. Its completed counterpart is
[`day3/prime_answer/`](day3/prime_answer/README.md). The
[`day3/prime_safe/`](day3/prime_safe/README.md) variant proves memory safety
through body-preserving specification layers, with completed proofs in
[`day3/prime_safe_answer/`](day3/prime_safe_answer/README.md).

Build all Day 3 files with `make day3`; `make prime` remains available as an
alias.

## Requirements

- `git`
- `make`
- standard Unix tools such as `find` and `grep`
- [opam](https://opam.ocaml.org/doc/Install.html) 2.1 or newer

Linux and macOS can use the commands below directly. Windows users should run
the build inside WSL.

## Install

### 1. Create an opam switch

```sh
opam init
opam update
opam switch create cris-workshop ocaml-base-compiler.4.14.1
eval "$(opam env --switch=cris-workshop --set-switch)"
```

Activate the switch again after opening a new terminal:

```sh
eval "$(opam env --switch=cris-workshop --set-switch)"
```

### 2. Install the workshop CRIS snapshot

```sh
opam repo add rocq-released https://rocq-prover.org/opam/released
opam pin add -y --jobs=N rocq-cris \
  'git+https://github.com/snu-sf/CRIS.git#c0bcd04e7ddfed32f1d7b8e5e2e328e3b5957bdd'
```

This installs the exact
[CRIS workshop snapshot](https://github.com/snu-sf/CRIS/commit/c0bcd04e7ddfed32f1d7b8e5e2e328e3b5957bdd),
Rocq 9.0.0, and the exact CRIS dependencies. Check the active installation:

```sh
opam list --installed rocq-cris
opam pin list | grep '^rocq-cris'
coqc --version
```

The final command should report version `9.0.0`.

### 3. Clone the workshop

```sh
git clone https://github.com/snu-sf/2026-verification-workshop.git
cd 2026-verification-workshop
```

### 4. Check the workshop

```sh
make check
make day2
make day3
```

`make check` builds Day 1, confirms the seven intended core starter `Admitted`
commands and one optional advanced `Admitted`, and verifies the completed
lecture and answer proofs. The other two commands build every starter and
answer file for their respective days. A successful setup finishes each
command with exit code 0.

## Editor setup

Launch the editor from a terminal where `cris-workshop` is active. Open the
workshop repository root so the editor can find `_CoqProject`.

### VS Code

Use [VsRocq](https://marketplace.visualstudio.com/items?itemName=rocq-prover.vsrocq).

```sh
opam install vsrocq-language-server.2.4.3
code --install-extension rocq-prover.vsrocq
command -v vsrocqtop
code .
```

Set `vsrocq.path` to the path printed by `command -v vsrocqtop` when VS Code
cannot locate the language server.

### Vim or Neovim

Use [Coqtail](https://github.com/whonore/Coqtail#installation). It requires Vim
with `+python3` or Neovim with `pynvim`, plus `coqidetop` on `PATH`.

### Emacs

Use the current [Proof General](https://github.com/ProofGeneral/PG#installing-proof-general)
package. Install it through NonGNU ELPA or MELPA, then start Emacs from the
active opam switch.

### Unicode input

CRIS source files use Unicode mathematical notation.

- **VS Code:** install
  [latex-input](https://marketplace.visualstudio.com/items?itemName=yellpika.latex-input),
  type a LaTeX-style name such as `\Sigma`, and accept the completion to insert
  `Σ`.
- **Vim/Neovim:** in Insert mode, `Ctrl-V u03a3` inserts `Σ`. See
  [Coqtail's Unicode input note](https://github.com/whonore/Coqtail#unicode-input)
  for native key sequences and optional plugins.
- **Emacs:** select the built-in `TeX` input method with
  `M-x set-input-method RET TeX RET`, type `\Sigma`, and use `C-\` to toggle the
  input method. See the GNU Emacs documentation on
  [input methods](https://www.gnu.org/software/emacs/manual/html_node/emacs/Select-Input-Method.html)
  and [`insert-char`](https://www.gnu.org/software/emacs/manual/html_node/emacs/Inserting-Text.html).

### Editor check

Open [`day1/lectures/Behavior.v`](day1/lectures/Behavior.v) and process it
from the first line to the end.

## Repository layout

| Path | Purpose |
|---|---|
| `day1/lectures/` | Complete demos of behaviors, modules, and contextual refinement |
| `day1/exercises/` | Two core Day 1 starters and an optional iterative-`put` extension |
| `day1/answers/` | Completed counterparts with the same theorem names |
| `day1/slides/` | Marp source, generated PDF, and presenter notes for Day 1 |
| `day2/lib/` | Shared Day 2 proof libraries |
| `day2/mem{,_answer}/` | Memory exercise and completed counterpart |
| `day2/stack{,_answer}/` | Linked-list stack exercise and completed counterpart |
| `day2/map/` | Linked-list, BST, and open-ended map exercises |
| `day2/refcell/` | Fractional-memory and reference-cell exercises |
| `day3/imp_system/` | Shared Imp language, memory, and safe-memory modules |
| `day3/prime{,_answer}/` | Nth-prime client exercise and completed counterpart |
| `day3/prime_safe{,_answer}/` | Body-preserving memory-safety exercise and completed counterpart |

## Troubleshooting

Check the active switch and Rocq version:

```sh
eval "$(opam env --switch=cris-workshop --set-switch)"
command -v coqc
coqc --version
```

Reinstall CRIS and rebuild the workshop after a load-path or `.vo` error:

```sh
opam reinstall --jobs=N rocq-cris
make clean
make check
make day2
make day3
```

The [VsRocq FAQ](https://github.com/rocq-prover/vsrocq/blob/main/docs/FAQ.md),
[Coqtail documentation](https://github.com/whonore/Coqtail), and
[Proof General manual](https://proofgeneral.github.io/doc/master/userman/)
cover editor-specific problems.

## References

- [Rocq opam installation](https://rocq-prover.org/docs/using-opam)
- [Rocq 9.0 Reference Manual](https://rocq-prover.org/doc/V9.0.0/refman/)
- [A Tour of Rocq](https://rocq-prover.org/docs/tour-of-rocq)
- [Software Foundations](https://softwarefoundations.cis.upenn.edu/lf-current/index.html)
- [Interaction Trees project](https://deepspec.github.io/InteractionTrees/)
- [Interaction Trees paper](https://doi.org/10.1145/3371119)
- [Refinement Tutorial](https://github.com/dongjaelee1/refinement-tutorial)
- [Iris 4.4.0 Proof Mode documentation](https://gitlab.mpi-sws.org/iris/iris/-/blob/iris-4.4.0/docs/proof_mode.md)
- [Iris Proof Mode paper](https://iris-project.org/pdfs/2017-popl-proofmode-final.pdf)
- [Iris lecture notes](https://iris-project.org/tutorial-material.html)
- [CRIS paper](https://doi.org/10.1145/3808317)
- [Conditional Contextual Refinement](https://sf.snu.ac.kr/ccr/)
