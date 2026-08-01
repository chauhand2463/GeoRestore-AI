# Model Architecture

## Current Architecture

The current architecture uses a GAN-based restoration pipeline.

Generator:
STGAN-CR

Discriminator:
[[Discriminator]] (PatchGAN)

## Input

Cloudy LISS-IV image

## Output

Cloud-free reconstructed image

## Loss

- Adversarial loss
- Reconstruction loss
- [[Spectral_Consistency]] loss

## Related

- [[Generator]]
- [[Discriminator]]
- [[Loss_Functions]]
- [[GAN]]
- [[Diffusion]] (alternative direction, not selected)
- [[Architecture_Decisions]]
- [[Experiment_003]]
