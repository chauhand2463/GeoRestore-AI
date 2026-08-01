# Requirements

## Functional

- Accept cloudy LISS-IV patches and produce cloud-free reconstructions.
- Maintain band count and spectral consistency ([[Spectral_Consistency]]).
- Emit georeferenced outputs ([[GeoTIFF]]).

## Non-functional

- Reproducible training runs (see [[05_Experiments]]).
- Configurable via `configs/`.
- Reasonable inference speed for patch-based processing.

## Related

- [[Overview]]
- [[Objectives]]
