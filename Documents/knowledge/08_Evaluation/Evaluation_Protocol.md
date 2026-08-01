# Evaluation Protocol

Standard protocol for every [[05_Experiments]] run:

1. Evaluate on held-out patches ([[Patch_Generation]]).
2. Compute [[PSNR]], [[SSIM]], [[SAM]] per band and mean.
3. Report [[Spectral_Metrics]] band-wise.
4. Record results in the experiment note and in [[Results]].
5. Log visualizations under `outputs/visualizations/`.

## Related

- [[PSNR]], [[SSIM]], [[SAM]]
