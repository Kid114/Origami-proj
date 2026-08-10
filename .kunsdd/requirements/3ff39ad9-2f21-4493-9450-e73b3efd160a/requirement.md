# Origami Tutorial Generator

## Background

AI-based limb recognition is mature enough to extract precise structural information of objects (mainly organisms), including an abstract *tree* of the object's limbs — limbs connected at joints, each with a length. There is also a software called Treemaker that transforms such a tree into an origami crease pattern and optimizes it locally.

However, Treemaker is usually not practical for humans: its crease patterns contain angles that are difficult to fold by hand. This project therefore first builds a similar generator that outputs **box-pleated** crease patterns (foldable on a 45°/90° grid) instead of Treemaker-style arbitrary angles. Box pleating is more controllable and easier to fold, and provides a stable baseline for further aesthetic design.

## Scope Decisions (personal project)

- **Platform**: web application — browser-based folding simulator, Python backend for photo recognition and crease pattern generation. Easy to share with the general public.
- **Input**: a single photo of one roughly centered object, longest side ≥ 512 px. No strict background requirement (clean backgrounds give the best results); photos with no clear object or multiple objects are rejected with an explanation.
- **Data**: photo recognition reuses pretrained vision models (segmentation + skeleton extraction); only post-processing and limb-tree extraction are built in this project. No model training from scratch.
- **Formats**: crease patterns stored in the FOLD format (JSON, the Origami Simulator standard) and rendered to SVG for display; tutorial steps rendered as PNG images from the simulator.
- **Non-goals**: no commercial features, accounts, or payments; no multi-object or video input; no physical (robot) folding; no mobile apps.

## Goals

End-to-end pipeline: photo → box-pleated crease pattern → foldable visualization → step-by-step folding tutorial. The final tutorial must be understandable by a general audience with no origami expertise.

1. **Photo recognition**: From a photo of a single object, extract a tree assignment of limbs with length proportions rounded to whole numbers (to comply with box pleating), plus limb textures and shapes to inform the end design.
2. **Box-pleating crease pattern generator**: Research and build a Treemaker-like generator that outputs box-pleated crease patterns. Because local optimization alone cannot guarantee a valid global assignment, AI proposes the initial assignment of box-pleated limbs. Foldability is verified in two stages: flat-foldability rules (Maekawa/Kawasaki conditions) for the 2D crease pattern, then a full 3D fold check in the simulator.
3. **Folding strategy library**: Collect common shapes/patterns that origami can represent and that are implementable on box-pleated limbs; record the folding strategy for each. Start with a curated set of \~15 patterns (cylinder, cone, box, flat shape, etc.) — enough when every limb in the validation set maps to a known strategy.
4. **Folding visualization software**: Build an interactive browser simulator so design intentions are reflected on crease patterns (e.g., folds are recorded as the paper is folded). AI proposes fold topology (mountain/valley assignment and layer ordering); geometric constraints verify the proposal and reject invalid ones. The paper must never self-intersect at any time.
5. **AI-driven folding workflow**: Use AI with the simulator from step 4 to approximate the object in the input photo. Resemblance is measured by silhouette overlap (IoU) plus limb landmark distance; folding stops when per-iteration improvement falls below a threshold or the step budget (default: 50 folds, tunable) is reached.
6. **Tutorial generation**: Convert the folding history into a numbered series of steps, each rendered as an image with fold instructions (mountain/valley, landmarks, short text hint) understandable by a general audience.
7. **Future extensions**: e.g., per-object-class folding strategy selection (for instance, when the object is purely a 2D shape), video input, animated 3D previews.

Steps 1–4 are independent building blocks (step 3 requires step 2's patterns); step 5 integrates the outputs of steps 1–4; step 6 depends on step 5.

## Failure Handling

The system halts with a clear explanation — never broken output — when:

- No object or multiple objects are detected in the photo.
- The limb tree is too complex for the paper size / grid resolution, so no valid assignment is found within the search budget (default: 5 min per photo, tunable).
- The shape is not representable by box pleating (e.g., mathematically impossible goals).

## Acceptance Criteria

- Validation set: \~15 photos across object categories (animals, plants, simple objects).
- ≥ 80% of validation photos produce a tutorial whose crease pattern is verified foldable in the simulator.
- Spot check: one non-expert folds 3 generated tutorials to a recognizable result.
- Every failure terminates with a clear explanation instead of broken output.

## Deferred Decisions (tuned during implementation)

- Exact photo set for the validation set (categories: animals, plants, simple objects).
- Default values for the search budget (step 2) and fold step budget (step 5) — starting points above, to be tuned with the first prototype.
- Strategy library contents beyond the initial \~15 patterns.

