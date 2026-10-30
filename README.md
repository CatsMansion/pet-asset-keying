# pet-asset-keying

Cut anthropomorphized character art to transparent PNG — a single image, or a whole sequence of video frames. **No AI model required** (no rembg, no torch, no onnx): numpy + OpenCV + SciPy only.

## The one idea

Character art has **line art**. The boundary already sits on the line. So do not hunt for an edge inside the anti-aliasing blur band — cut along the line instead.

Cut inside that band and only two outcomes exist: you eat the character, or you leave a halo. That halo is not a bug in your code; it is in the source pixels. Five different open-source matting models produce the same fringe on the same image.

## Two methods, non-overlapping weaknesses

| | Method | Strong at | Weak at |
|---|---|---|---|
| Line art present | `scripts/line_keying.py` — flood from the frame border; the dark line blocks it | Hard edges, params barely matter (TOL 20→65 changes the result by 3.5%), 0.2 s/frame | Hair, gaps in the line |
| Fine structures | `scripts/keying.py` — traditional matting ∩ isnet-anime (either side says background ⇒ remove) | Hair, small enclosed gaps | Needs a model |

Chain them: line art where lines exist, matting for fine structures.

## Numbers from real work

- Border contamination on a 1024×1536 character: **7724 px → 0 px** (cyan background, inspected at 1:1).
- Calibrated defaults live in `SKILL.md` (TOL_OUT 30 / SEED_MAX 12 / TOL_HOLE 40 / MAX_REMOVE_FRAC 0.20).
- Enclosed spaces (crescent shapes, braid-versus-shoulder gaps) need a seed taken from the **most background-like non-edge pixel**. Geometric centroids fail on 10% of blobs.

## Also in here

- A **tested-and-backfired** list: six post-processing tricks that made results worse, each with what it actually did.
- Two silent-failure bugs worth more than the code. `cv2.floodFill` does not validate its seed — it flooded roughly 950k px and hit half the candidate blobs. And a duplicated mask block double-counted removals until the safety cap rolled the whole frame back. **Cap-style logic hides bugs**: every over-limit-skip branch needs a counter or a warning.
- `scripts/temporal_keying.py` for frame sequences.

## Install

```bash
npx skills add CatsMansion/pet-asset-keying
```

MIT.
