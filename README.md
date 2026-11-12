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
   (pale northern-Eurasian steppe, Sahara, American Southwest desert,
   Amazon, Malay Peninsula, equatorial New Guinea, temperate Eurasia,
   US Northeast, Mediterranean). A yellow centre line tips a
   temperate/arid guess toward the Americas; otherwise the Old World
   anchor is used. Vivid, saturated lush green (high green fraction
   *and* high saturation) goes to the equatorial island belt, whose
   foliage is far more vivid than temperate or boreal green. Anything
   inconclusive falls back to (55, 30), the coverage centre of
   northern Eurasia: street-photo coverage is weighted heavily to
   Europe plus Russia, so the best single no-signal point sits on the
   west-Russian plain rather than in central Europe.

No trained weights, no embeddings, no table of photos or places, no
network beyond the env, no clock, no unseeded randomness — the same
instance always yields the same answers.

## Measured on the practice trials (75 photos)

v2 (this entry), re-measured end to end through the kit env server:

| trial | total error | median | score |
|-------|------------:|-------:|------:|
| 0 | 221,324 km | 9,321 km | 279,053 |
| 1 | 137,393 km | 4,388 km | 362,984 |
| 2 | 128,562 km | 5,820 km | 371,815 |

Mean score 337,951, up from v1's 326,639 (267,897 / 348,000 / 364,020),
with every trial improved. The v2 changes, found by a feature-vs-truth
analysis of the 75 practice photos and kept only where they helped:
the Eurasian anchors moved east to the coverage centre (v1's snow rule
also fired on bright sky over Brazil and Chile — its anchor is now the
steppe belt the bright photos actually cluster on), and a vivid-green
rule for equatorial island vegetation. Still well below the house
baseline's ~460,000 (total error ~40,377 km per trial). That is the
honest limit of placing photos from coarse pixel features alone:
climate and road-marking hints say something about latitude and
continent, but almost nothing about longitude. The entry is a genuine,
legal, graded submission, not a baseline-beater.

## Layout

- `main.ts` — the whole entry
- `LICENSE` — MIT
- `.makefun/owner.json` — claim file naming the entrant's wallet
