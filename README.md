# Awesome-Pixel-Space-Diffusion

Pixel-space diffusion has recently emerged as a distinct line of research in visual geometry, monocular depth estimation, point-map prediction, and 3D scene generation. In contrast to latent diffusion pipelines that rely on a VAE or other learned tokenizers, pixel-space diffusion models operate directly on the target signal itself, such as depth maps, point maps, rendered views, or 3D Gaussian attributes. This design is especially attractive for geometry tasks, where small local distortions can significantly damage downstream 3D quality.

## Summary

| Title | Year | Main task | Main idea | Publication status | Code / project |
|---|---:|---|---|---|---|
| **Pixel-Perfect Depth with Semantics-Prompted Diffusion Transformers** | 2025 | Monocular depth estimation | Direct pixel-space diffusion for depth; SP-DiT and Cascade DiT improve efficiency and detail preservation | **NeurIPS 2025** | Paper: https://arxiv.org/abs/2510.07316 · Project: https://pixel-perfect-depth.github.io/ · Code: https://github.com/gangweix/pixel-perfect-depth |
| **Pixel-Perfect Visual Geometry Estimation** | 2026 | Monocular and video visual geometry estimation | Extends Pixel-Perfect Depth to a broader geometry setting with PPD and PPVD; adds semantics-consistent video modeling | **arXiv preprint** | Paper: https://arxiv.org/abs/2601.05246 · Code: https://github.com/gangweix/pixel-perfect-depth |
| **PointDiT: Pixel-Space Diffusion for Monocular Geometry Estimation** | 2026 | Monocular point-map estimation | Plain ViT diffuses directly on raw XYZ point-map patches, conditioned on DINOv3 image tokens; no VAE or tokenizer | **ICML 2026** | Paper: https://arxiv.org/abs/2607.02515 · Project: https://haofeixu.github.io/pointdit/ |
| **PixWorld: Unifying 3D Scene Generation and Reconstruction in Pixel Space** | 2026 | Unified 3D reconstruction and generation | Pixel-space flow matching supervised on rendered views; one model for reconstruction, image-to-3D, and text-to-3D | **arXiv preprint** | Paper: https://arxiv.org/abs/2607.05373 · Project: https://sensengao.github.io/PixWorld/ · Code: https://github.com/SensenGao/PixWorld |
| **PixGS: Pixel-Space Diffusion for Direct 3D Gaussian Splat Generation** | 2026 | 3D Gaussian Splat generation | Direct diffusion over Gaussian attributes to bypass latent bottlenecks in 3DGS generation | **arXiv preprint** | Paper: https://arxiv.org/abs/2607.01803 |
| **GeometryCrafter: Consistent Geometry Estimation for Open-world Videos with Diffusion Priors** | 2025 | Video point-map / geometry estimation | Latent-space video diffusion with a dedicated point-map VAE for temporally consistent open-world geometry | **ICCV 2025** | Paper: https://arxiv.org/abs/2504.01016 · Project: https://geometrycrafter.github.io/ · Code: https://github.com/TencentARC/GeometryCrafter |
| **Repurposing Diffusion-Based Image Generators for Monocular Depth Estimation (Marigold)** | 2024 | Monocular depth estimation | Fine-tunes Stable Diffusion for depth prediction in latent space | **CVPR 2024** | Paper: https://arxiv.org/abs/2312.02145 · Project: https://marigoldmonodepth.github.io/ · Code: https://github.com/prs-eth/marigold |
| **Back to Basics: Let Denoising Generative Models Denoise (JiT)** | 2025 | Image generation methodology | Pixel-space ViT diffusion with direct clean-data prediction; important conceptual precursor to later geometry works | **arXiv preprint** | Paper: https://arxiv.org/abs/2511.13720 |

## Detailed Paper Summaries

### 1. Pixel-Perfect Depth with Semantics-Prompted Diffusion Transformers

- **Core problem**
  Existing generative monocular depth methods often inherit latent diffusion pipelines from image generation. They encode depth into a VAE latent space and then denoise there. This introduces reconstruction artifacts, especially around boundaries and thin structures, which later appear as flying pixels in point clouds.
- **Key contribution**
  Pixel-Perfect Depth proposes performing diffusion directly in pixel space, avoiding the lossy VAE entirely. The paper argues that for geometry, this design change is not cosmetic but fundamental, because geometry quality is especially sensitive to small errors introduced by compression.
- **Technical design**
  The paper introduces two main modules:
  - **Semantics-Prompted DiT (SP-DiT)**, which injects semantic priors from pretrained vision encoders into the diffusion transformer.
  - **Cascade DiT**, which progressively increases token count during generation to improve computational efficiency while preserving detail.
- **Main benefits**
  The method produces cleaner depth maps and significantly better point-cloud reconstruction quality than earlier generative baselines. The strongest gains are at edges, thin structures, and detailed surfaces where latent-space reconstruction is weakest.
- **Why it matters**
  This paper is one of the clearest early demonstrations that pixel-space diffusion is practical for geometry estimation, not just for natural image generation.

### 2. Pixel-Perfect Visual Geometry Estimation

- **Core problem**
  While Pixel-Perfect Depth addresses single-image depth estimation, many real geometry applications require both stronger structural fidelity and temporal consistency across frames.
- **Key contribution**
  This paper generalizes the Pixel-Perfect idea into a broader geometry framework. It presents:
  - **PPD** for monocular depth / geometry estimation
  - **PPVD** for video geometry estimation
- **Technical design**
  Relative to the earlier PPD paper, the 2026 work introduces:
  - a broader framing around visual geometry rather than only monocular depth
  - **Semantics-Consistent DiT** for temporally stable conditioning
  - **reference-guided token propagation** to preserve frame-to-frame consistency at lower cost
- **Main benefits**
  The method aims to keep the same sharpness and artifact reduction of pixel-space diffusion while addressing temporal coherence, which is a major difficulty in video geometry estimation.
- **Why it matters**
  This work marks the transition from pixel-space depth estimation to pixel-space geometry foundation modeling for both images and videos.

### 3. PointDiT: Pixel-Space Diffusion for Monocular Geometry Estimation

- **Core problem**
  Monocular point-map estimation is fundamentally ambiguous. Deterministic regression models often average over plausible 3D solutions and therefore oversmooth geometric structure. Latent diffusion models can model uncertainty better, but they still inherit a VAE bottleneck that harms fine geometry.
- **Key contribution**
  PointDiT proposes a minimalist alternative: a plain ViT-based diffusion model that operates directly on raw point-map patches in pixel space. Instead of building a custom latent tokenizer, it trains the diffusion model from scratch on the target geometry representation itself.
- **Technical design**
  The important ingredients are:
  - direct diffusion over **XYZ point maps**
  - **DINOv3 token conditioning** from the RGB image
  - a plain ViT backbone rather than a hybrid convolution-transformer architecture
  - **x-prediction** rather than velocity/noise prediction, inspired by JiT
- **Main benefits**
  According to the paper, this design improves structural sharpness, preserves thin structures, and handles ambiguous regions such as transparent objects better than both deterministic regressors and latent-diffusion baselines.
- **Why it matters**
  PointDiT is especially important because it argues for a strong simplification thesis: if geometry is modeled directly in data space, then much of the extra architecture used in prior methods may be unnecessary.

### 4. PixWorld: Unifying 3D Scene Generation and Reconstruction in Pixel Space

- **Core problem**
  3D reconstruction and 3D generation are usually handled by separate model families. Reconstruction tends to use direct regression or feed-forward geometric prediction, while generation often uses latent diffusion over scene representations. These two worlds have rarely been unified cleanly.
- **Key contribution**
  PixWorld reformulates both reconstruction and generation under a single pixel-space diffusion framework. One model handles:
  - multi-view 3D reconstruction
  - image-to-3D generation
  - text-to-3D generation
- **Technical design**
  The model partitions inputs into clean views and noisy views and processes them with a two-stream diffusion transformer. Rather than supervising a latent scene code, it applies diffusion supervision directly on rendered images from the scene representation. It also adds a **geometry perception loss** computed in the feature space of a pretrained 3D foundation model such as VGGT or pi-cubed.
- **Main benefits**
  This design aligns training more closely with actual 3D scene fidelity, rather than proxy latent reconstruction quality. The model reports strong performance for both reconstruction and generation and supports very fast distilled inference.
- **Why it matters**
  PixWorld is one of the first papers to argue that pixel-space diffusion is not only useful for monocular geometry estimation, but can also serve as a unifying framework for broader 3D scene learning.

### 5. PixGS: Pixel-Space Diffusion for Direct 3D Gaussian Splat Generation

- **Core problem**
  3D Gaussian Splat generation often relies on adapting latent image generators or multi-stage pipelines. These systems may accumulate errors across modules and inherit the artifacts of compressed latent representations.
- **Key contribution**
  PixGS proposes direct diffusion over 3D Gaussian attributes, bringing the pixel-space philosophy into 3DGS generation.
- **Technical design**
  The method denoises Gaussian attributes directly and supplements supervision with:
  - depth
  - surface normals
  - high-frequency structural cues
- **Main benefits**
  The paper claims better asset quality and fast inference relative to previous multi-stage methods.
- **Why it matters**
  This work extends pixel-space diffusion beyond depth and point maps into explicit 3D scene representations used in current 3D content generation pipelines.

### 6. GeometryCrafter

- **Role in the literature**
  GeometryCrafter is not a pixel-space method, but it is a crucial latent-space baseline for comparison. It addresses video geometry estimation with a dedicated point-map VAE and a video diffusion model.
- **Main idea**
  Instead of relying on standard image VAE encoding for geometry, it builds a point-map-specific VAE to better handle unbounded 3D coordinates and temporal coherence.
- **Importance**
  PointDiT and Pixel-Perfect style methods are easier to understand when contrasted against GeometryCrafter. GeometryCrafter demonstrates the strengths of diffusion priors for geometry, but also highlights why later papers consider the latent bottleneck a limiting factor.

### 7. Marigold

- **Role in the literature**
  Marigold is one of the most influential diffusion-based depth estimation papers. It demonstrates that pretrained image diffusion models can transfer strong visual priors to monocular depth estimation.
- **Main idea**
  The method fine-tunes Stable Diffusion for depth prediction in latent space.
- **Importance**
  Marigold serves as a conceptual precursor for later generative geometry methods. However, later pixel-space papers specifically critique the VAE dependence that Marigold inherits from Stable Diffusion.

### 8. JiT: Back to Basics

- **Role in the literature**
  JiT is not a geometry paper, but it is a highly relevant methodological precursor.
- **Main idea**
  It shows that plain transformers can be trained as pixel-space diffusion models and that directly predicting clean data can be more effective than predicting noise-like targets in high-dimensional pixel space.
- **Importance**
  PointDiT explicitly builds on this line of thinking. JiT gives theoretical and empirical support for the claim that direct data-space prediction can be viable even without tokenizers and elaborate latent encoders.

## Thematic Survey

### 1. Why pixel space matters for geometry

Geometry estimation differs from natural image synthesis in one crucial respect: small local errors can have large downstream consequences. A slight boundary blur in RGB image generation may be acceptable, but the same blur in a depth map or point map can create severe 3D artifacts. These include:

- flying pixels at edges
- broken thin structures
- unstable surface normals
- degraded multi-view consistency
- poor downstream reconstruction quality

Latent-space models compress the target representation before denoising. That compression is often beneficial for efficiency, but it is dangerous for geometry because it can erase the exact high-frequency structures that reconstruction tasks care about most. Pixel-space diffusion attacks this issue directly by removing the latent bottleneck.

### 2. Historical progression

The current literature can be read as a three-stage progression.

#### Stage 1: Diffusion priors for geometry in latent space

Early diffusion-based geometry methods adapted existing latent image generators. Marigold is the clearest example. It showed that a pretrained diffusion model can provide surprisingly strong priors for monocular depth. GeometryCrafter extended this idea to video and point-map estimation, building a more geometry-specific latent pipeline.

At this stage, the field learned that diffusion priors are highly useful for ambiguous geometric inference. However, the methods still depended on latent compression.

#### Stage 2: Pixel-space diffusion for single-image geometry

The Pixel-Perfect line and PointDiT represent the shift from latent-space adaptation to geometry-native pixel-space diffusion.

These works share the view that:

- geometry should be modeled in the target space directly
- semantic conditioning from powerful visual encoders is still essential
- diffusion should be used not just as a denoiser, but as a probabilistic estimator for ambiguous structure

This stage establishes that pixel-space diffusion is feasible and often better than latent diffusion for fine geometric detail.

#### Stage 3: Pixel-space diffusion for unified 3D modeling

PixWorld and PixGS broaden the paradigm. Instead of focusing only on per-image geometry prediction, they extend pixel-space diffusion to:

- explicit 3D scene representations
- unified reconstruction and generation
- direct Gaussian-based 3D content generation

This stage suggests that pixel-space diffusion may become a general design principle for 3D learning, not just a specialized trick for depth maps.

### 3. Shared design patterns

Although the papers target different outputs, several common patterns appear.

#### Transformer-first architecture

Most recent pixel-space works use DiT or ViT-style backbones. This matches the patchified nature of pixel-space diffusion and avoids some of the complexity of hybrid convolution-transformer systems.

#### Strong conditioning from foundation models

Pixel-space diffusion does not mean unconditional generation. In fact, these models depend heavily on strong priors:

- DINOv3 in PointDiT
- vision foundation model features in Pixel-Perfect Depth
- geometry-aware 3D foundation model features in PixWorld

This indicates a broader trend: pixel-space diffusion is often paired with representation learning rather than replacing it.

#### Efficiency-oriented scheduling

Because pixel-space denoising is expensive, practical systems introduce special efficiency mechanisms:

- cascade token schedules
- reference-guided token propagation
- low-step or one-step generation
- post-training distillation

This is one of the main engineering barriers still facing the field.

#### Direct optimization on geometry-sensitive outputs

A major conceptual advantage of pixel-space diffusion is that training acts more directly on the outputs used in evaluation or downstream tasks. This avoids the mismatch introduced by intermediate latent reconstruction objectives.

### 4. Major advantages of the paradigm

The literature suggests five main advantages.

#### 4.1 Better high-frequency geometry

Pixel-space methods are consistently motivated by stronger edge fidelity, thinner structures, and cleaner boundaries. This is one of the most repeated claims across Pixel-Perfect Depth, PointDiT, and PixGS.

#### 4.2 Fewer bottleneck artifacts

By avoiding VAE compression, these methods reduce reconstruction artifacts that would otherwise be baked into the generative process before denoising even begins.

#### 4.3 Better treatment of uncertainty

Like latent diffusion, pixel-space diffusion is generative and probabilistic. But it applies that stochastic modeling directly to the geometric signal. This helps with inherently ambiguous monocular inference problems.

#### 4.4 Simpler end-to-end modeling

Some recent papers, especially PointDiT, argue that data-space diffusion can simplify the overall system by removing the need for custom geometry tokenizers, geometry VAEs, or hybrid decoders.

#### 4.5 Broader unification potential

PixWorld indicates that pixel-space diffusion may unify scene reconstruction and scene generation in a single training and inference framework.

### 5. Remaining limitations

Despite promising results, the field still has substantial open challenges.

#### 5.1 Computational cost

Pixel-space diffusion is expensive because it operates on high-dimensional signals directly. Even when successful, many systems require careful token scheduling, distillation, or low-step approximations to become practical.

#### 5.2 Limited benchmark maturity

Many of the newest papers are very recent and still pre-publication. The ecosystem of standardized evaluations across datasets, tasks, and representations is not yet fully mature.

#### 5.3 Incomplete code release

Code availability is uneven. Some papers have mature repositories, while others currently provide only project pages or partial release plans.

#### 5.4 Scaling beyond current tasks

It is still unclear how far the current recipe generalizes to:

- dynamic 4D reconstruction
- joint geometry and appearance modeling
- long-video consistent scene understanding
- large-scale embodied or robotics settings

### 6. Research outlook

Based on the current trajectory, the next major developments are likely to include:

- few-step or one-step pixel-space geometry diffusion
- unified models for depth, normals, point maps, and camera estimation
- stronger video and 4D consistency mechanisms
- hybrid use of 2D and 3D foundation-model features as guidance
- direct integration with scene representations such as 3D Gaussian fields

The broader implication is that geometry may benefit more than image synthesis from staying in data space. For natural images, latent compression is often an acceptable efficiency trade-off. For geometry, that trade-off appears much more costly.

## Comparative Discussion

### Pixel-space diffusion vs latent-space diffusion

Pixel-space diffusion and latent-space diffusion both try to model ambiguous conditional geometry distributions, but they differ in where uncertainty is represented.

- **Latent-space methods** are more efficient and often benefit from large pretrained generators, but they inherit the errors and biases of the latent encoder.
- **Pixel-space methods** are more expensive, but they preserve direct access to the target geometric signal and therefore avoid some structural degradation.

For geometry tasks that care strongly about local shape fidelity, the current literature increasingly favors the second trade-off.

### Pixel-space diffusion vs deterministic regression

Deterministic regression models are fast and often very strong on coarse metrics, but they tend to predict average solutions. This can wash out plausible alternatives and oversmooth surfaces or transparent regions. Diffusion models, including pixel-space ones, handle ambiguity more naturally by learning a conditional generative distribution.

This is one of the central arguments in PointDiT and one reason why generative geometry estimation is now receiving more attention.

## Recommended Reading Order

For a researcher entering this topic, a practical reading path is:

1. **Marigold** to understand the latent-diffusion starting point for geometry.
2. **GeometryCrafter** to see how latent diffusion was extended to point maps and videos.
3. **Pixel-Perfect Depth** to understand the first major move into pixel-space geometry estimation.
4. **Pixel-Perfect Visual Geometry Estimation** for the broader image/video geometry formulation.
5. **PointDiT** for the minimalist point-map formulation and the x-prediction perspective.
6. **PixWorld** for unified reconstruction and generation.
7. **PixGS** for extension into 3D Gaussian Splat generation.
8. **JiT** for the methodological foundation behind modern pixel-space transformer diffusion.

## Conclusion

Pixel-space diffusion is becoming a serious alternative to latent diffusion for geometry-centric tasks. The common argument across recent papers is that geometry should be modeled in a representation space that preserves its structural precision, rather than in a compressed latent space optimized primarily for reconstruction efficiency. The strongest evidence so far comes from depth and point-map estimation, where the reduction of flying pixels and the preservation of fine detail are repeatedly emphasized.

At the same time, the field is still early. Many of the newest contributions are from 2026 and remain at the preprint stage. Code release maturity varies, and large-scale evaluation standards are still forming. Even so, the direction is already clear: pixel-space diffusion is evolving from a niche design choice into a broader paradigm for geometry-native generative modeling.

## Sources

- PointDiT: https://arxiv.org/abs/2607.02515
- PointDiT project page: https://haofeixu.github.io/pointdit/
- Pixel-Perfect Visual Geometry Estimation: https://arxiv.org/abs/2601.05246
- Pixel-Perfect Depth with Semantics-Prompted Diffusion Transformers: https://arxiv.org/abs/2510.07316
- Pixel-Perfect Depth project page: https://pixel-perfect-depth.github.io/
- Pixel-Perfect Depth code: https://github.com/gangweix/pixel-perfect-depth
- PixWorld: https://arxiv.org/abs/2607.05373
- PixWorld project page: https://sensengao.github.io/PixWorld/
- PixWorld code: https://github.com/SensenGao/PixWorld
- PixGS: https://arxiv.org/abs/2607.01803
- GeometryCrafter: https://arxiv.org/abs/2504.01016
- GeometryCrafter project page: https://geometrycrafter.github.io/
- GeometryCrafter code: https://github.com/TencentARC/GeometryCrafter
- Marigold: https://arxiv.org/abs/2312.02145
- Marigold project page: https://marigoldmonodepth.github.io/
- Marigold code: https://github.com/prs-eth/marigold
- JiT / Back to Basics: https://arxiv.org/abs/2511.13720
