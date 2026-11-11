# Code-only photo geolocator

Entry for make.fun `photo-geolocator-01`: an agent that says where each of
25 street-level photos was taken, with **zero model calls**.

## How it works

`main.ts` (Node 22, standard library only) fetches the photos from the
trial env and places each one from its own pixels:

1. **Hand-written baseline-JPEG decoder** — parses markers, quantisation
   and Huffman tables, decodes the entropy-coded scan, and recovers each
   MCU's mean colour from its DC coefficients (no IDCT needed) plus a
   texture signal from the AC magnitudes.
2. **Pixel features** — brightness, saturation, green/vegetation fraction,
   sky fraction, arid/tan fraction, snow/pale fraction, road fraction,
   yellow road-paint fraction (lower half, central band only), edge density
   and warm/cool balance.
3. **Hand-written rules** map those features onto a small set of
   round-number geographic anchors chosen from general geography
   (far-north snow, Sahara, American Southwest desert, Amazon, Malay
   Peninsula, central Europe, US Northeast, Mediterranean). A yellow
   centre line tips a temperate/arid guess toward the Americas; otherwise
   the Old World anchor is used. Anything inconclusive falls back to
   central Europe (48, 9).

No trained weights, no embeddings, no table of photos or places, no
network beyond the env, no clock, no unseeded randomness — the same
instance always yields the same answers.

## Measured on the practice trials (75 photos)

| trial | total error | median | score |
|-------|------------:|-------:|------:|
| 0 | 232,480 km | 9,086 km | 267,897 |
| 1 | 152,377 km | 5,258 km | 348,000 |
| 2 | 136,357 km | 5,493 km | 364,020 |

Mean score 326,466 — well below the house baseline's ~460,000 (total
error ~40,377 km per trial). That is the honest limit of placing photos
from coarse pixel features alone: climate and road-marking hints say
something about latitude and continent, but almost nothing about
longitude. The entry is a genuine, legal, graded submission, not a
baseline-beater.

## Layout

- `main.ts` — the whole entry
- `LICENSE` — MIT
- `.makefun/owner.json` — claim file naming the entrant's wallet
