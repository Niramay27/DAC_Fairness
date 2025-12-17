# DAC_Fairness

This repository investigates **fairness and bias properties of the Descript Audio Codec (DAC)** by probing whether **discrete acoustic codebooks encode gender-specific or semantically separable information**.
---

## Motivation

Modern TTS systems combine large language models with neural audio codecs to generate expressive and controllable speech. While bias in language models has been extensively studied, **bias propagation through audio codecs remains largely unexplored**.

DAC is widely used as a backend for speech generation. If its discrete codebooks encode demographic attributes (e.g., gender), such biases could propagate invisibly into downstream generative models.

This repository explores the following question:

> **Do DAC codebooks encode gender-specific acoustic patterns in a separable or interpretable way?**

---

## Understanding the DAC Model (Section 4.4)

ParlerTTS uses a **Deep Acoustic Codec (DAC)**—similar in spirit to Encodec—for discretizing and reconstructing waveform audio.

### DAC Architecture Overview

- **Encoder**: Maps continuous audio into latent representations
- **Residual Vector Quantization (RVQ)**:
  - Multiple quantization stages (L codebooks)
  - Each stage outputs a discrete token
  - Final representation is the sum of embeddings across stages
- **Decoder**: Reconstructs waveform audio from discrete tokens

### Key Characteristics

- **Multi-stage RVQ**: Captures progressively finer acoustic details
- **Token Vocabulary Size**: Typically 1024–2048 entries per codebook
- **Latent Dimensionality**: ~128D per token
- **Objective**: High-fidelity audio reconstruction, not interpretability

Each token represents a compressed spectral-acoustic feature chunk. These tokens are concatenated over time to synthesize complete audio signals.

---

## Hypothesis and Experimental Setup (Section 4.5)

### Hypothesis

If gender-specific acoustic characteristics (e.g., pitch, timbre, articulation) are encoded in DAC codebooks, then:

- **Swapping gender words in prompts** should yield
- **Systematically different RVQ token distributions**, especially in early codebooks

### Experimental Design

1. Use identical descriptive prompts while **changing only the gender term**
   - Example:  
     _“A confident **female** engineer speaking in Delhi Hindi”_  
     vs  
     _“A confident **male** engineer speaking in Delhi Hindi”_

2. Generate speech using ParlerTTS
3. Intercept the **RVQ token embeddings** produced by the GPT decoder
4. Analyze and visualize token distributions across gender-altered prompts

The goal is to determine whether **token-level separation or clustering** emerges.

---

## Findings and Visualizations (Section 4.6)

### Token Extraction

- Each RVQ token corresponds to a **128-dimensional embedding**
- Tokens were collected across multiple generations and prompts

### Dimensionality Reduction

To visualize the high-dimensional token space:

- **PCA** for linear structure
- **UMAP** for nonlinear neighborhood preservation

### Clustering Methods

- Hierarchical clustering
- HDBSCAN (density-based clustering)
- Silhouette score analysis for cluster quality

### Observations

- Token distributions were **highly uniform and entropic**
- No consistent gender-wise separation was observed
- Clusters were:
  - Small in number (2–3 at most)
  - Unstable across runs
  - Not aligned with gender distinctions

These results held across:
- Different codebook stages
- Different prompts
- Different dimensionality reduction techniques

### Interpretation

- DAC codebooks appear optimized for **coverage and reconstruction fidelity**
- Tokens do **not map cleanly to interpretable attributes** such as gender
- Semantic attributes are likely **entangled or averaged out** at the codec level

---

## Conclusion and Project Outcome (Section 4.7)

### Main Conclusion

Despite an initially promising hypothesis, we find **no conclusive evidence** that DAC codebooks encode **isolated or separable gender-specific acoustic representations**.

### Why the Hypothesis Failed

- DAC tokens are optimized for **entropy maximization**, not disentanglement
- Tokens represent **low-level acoustic features**, not semantic concepts
- RVQ aggregation blurs attribute-specific signals
- Clustering in high-dimensional token space is inherently unstable

### Implications

- Bias in TTS systems is more likely introduced by:
  - Prompt encoders (e.g., T5)
  - Autoregressive token predictors (e.g., GPT)
- Audio codecs act as **information-preserving but semantically agnostic components**

### Future Directions

- Designing **interpretable discrete token spaces** for audio
- End-to-end bias tracing across multimodal pipelines
- Black-box probing methods for generative speech models

---

## Repository Structure

```text
DAC_Fairness/
│
├── assets/                  # Official DAC assets
├── conf/                    # DAC configuration files
├── dac/                     # Core DAC implementation (official)
├── scripts/
│   └── visualize_codebooks.ipynb   # Custom analysis notebook
├── tests/                   # DAC test files
│
├── plots_codebooks/          # Codebook-level visualizations
├── plots_hdbscan_umap/       # HDBSCAN + UMAP plots
├── plots_heirarchical/       # Hierarchical clustering results
├── plots_pca/                # PCA projections
├── plots_silhouette/         # Silhouette score analyses
│
├── Dockerfile
├── Dockerfile.dev
├── docker-compose.yml
├── requirements.txt
├── setup.py
├── LICENSE
├── README.md
```

## Code Attribution

⚠️ **Important Notice**

All code in this repository **except** the analysis notebook listed below is taken directly from the **official Descript Audio Codec (DAC) repository**.

🔗 **Official DAC Repository**  
https://github.com/descriptinc/descript-audio-codec

### Custom Contributions

The following components were added as part of this work:

- `scripts/visualize_codebooks.ipynb`
- Generated visualization directories:
  - `plots_codebooks/`
  - `plots_hdbscan_umap/`
  - `plots_heirarchical/`
  - `plots_pca/`
  - `plots_silhouette/`

No modifications were made to the core DAC implementation.

---

## How to Run

Clone the repository and install dependencies:

```bash
git clone https://github.com/<your-username>/DAC_Fairness.git
cd DAC_Fairness
pip install -r requirements.txt
```
Launch the analysis notebook:

``` bash
jupyter notebook scripts/visualize_codebooks.ipynb
```

