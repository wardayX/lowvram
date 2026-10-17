# LowVRAM Lab

**Running quantized image and video generation workflows on a constrained NVIDIA T4 GPU using Google Colab and ComfyUI.**

LowVRAM Lab documents practical experiments in adapting large generative models to limited GPU memory. The repository focuses on reproducible notebook workflows, staged model loading, quantized inference, output measurements, and best-effort content moderation.

> **Project status:** Experimental research/demo workflows. Results and safety behavior may vary with model revisions, custom-node versions, runtime availability, and hardware.

## Demonstrations

The examples below show one connected workflow: generate a source image with Qwen-Image 2.1, animate it with MiniMax H3, then optionally apply RIFE frame interpolation.

| Stage | Artifact | What it demonstrates |
|---|---|---|
| Text-to-image | `examples/qwen_source_image.png` | Qwen-Image 2.1 Q8_0 text-to-image output |
| Image-to-video | `examples/minimax_h3_preview.mp4` | MiniMax H3 animation of the source image |
| Post-processing | `examples/rife_final_output.mp4` | RIFE-interpolated final video |

Place the actual generated files at these paths, or update the paths to match your filenames. Only publish outputs you have reviewed and are allowed to redistribute.

## Key workflows

### 1. Qwen-Image 2.1 — Q8_0 text-to-image and image editing

Notebook: [`notebooks/Qwen_Image_2_1_Q8_0_Colab_T4.ipynb`](notebooks/Qwen_Image_2_1_Q8_0_Colab_T4.ipynb)

The notebook exposes two modes:

- **Text-to-image:** generate an image from a prompt without reference images.
- **Image editing:** optionally provide up to three reference images for editing or multi-image conditioning.

The workflow uses a Q8_0 GGUF diffusion model, a Qwen3-VL INT8 text encoder, and a Qwen VAE. The text encoder and diffusion model are loaded in stages to reduce simultaneous memory pressure. In the recorded run, standard VAE decoding ran out of memory and the tiled-decoding fallback completed successfully.

#### Recorded Qwen run

| Setting / measurement | Recorded value |
|---|---:|
| Diffusion model file | 7.070 GiB |
| Text encoder file | 8.709 GiB |
| VAE file | 0.629 GiB |
| Sampling steps | 30 |
| Sampler / scheduler | Euler / simple |
| CFG | 1.0 |
| Seed | `2143205672` |
| Sampling time estimate | About 12 min 48 sec |
| Output moderation score | 0.000 |
| Configured moderation threshold | 0.85 |
| Peak GPU-wide VRAM | Not measured reliably in this run |

The 12 min 48 sec figure is estimated from the reported progress-bar rate (30 steps at about 25.61 seconds per step); it is not the total end-to-end notebook runtime. Model-file sizes are not equivalent to runtime VRAM usage. The moderation score is a classifier result, not a guarantee that an image is safe. Add actual PNG dimensions and file size after reading them from the generated file.

### 2. MiniMax H3 — image-to-video

Notebook: [`notebooks/MiniMax_H3_I2VA_Colab_T4_Public.ipynb`](notebooks/MiniMax_H3_I2VA_Colab_T4_Public.ipynb)

The notebook uses an image as the opening frame and a motion prompt to generate a short video. The public configuration disables the optional behavior/anatomy LoRA catalogue. The recorded configuration used a Q4_0 GGUF model, remote Qwen3-VL conditioning, and a dedicated H3 Turbo adapter at seven sampling steps, with a fast profile and a configured 640-pixel long-edge cap.

#### Recorded run

| Measurement | Recorded value |
|---|---:|
| Main generation-cell execution time | About 15 minutes |
| RIFE post-processing cell time | About 21 seconds |
| Actual final resolution | 544 × 544 |
| Actual final frame rate | 96 FPS |
| Actual final frame count | 289 |
| Actual final duration | About 3.01 seconds |
| Final MP4 size | About 1.14 MiB |
| Peak GPU-wide VRAM | Not measured reliably |

The execution times are notebook cell runtimes, not independently measured pure sampling time. Final metadata refers to the RIFE-processed MP4. Record pre-RIFE metadata separately if you include a before/after comparison.

## Safety and limitations

The notebooks include best-effort checks, not a guarantee of safe output:

1. **Prompt screening:** conservative keyword-based checks block some prompts before generation. These can miss paraphrases and can reject benign prompts.
2. **Input image moderation:** an image classifier checks the uploaded opening/reference image.
3. **Generated image moderation:** the Qwen workflow checks the saved image against a configured classifier threshold.
4. **Video moderation:** the MiniMax workflow is intended to moderate decoded video frames with an image classifier. If only selected frames are sampled, unsafe content between sampled frames can be missed.
5. **Output handling:** before claiming flagged media cannot remain in output folders, verify that every ComfyUI save location and downloaded copy is covered by the same staging, moderation, and cleanup policy.

The reported Qwen output score of 0.000 passed the notebook's configured threshold of 0.85. This is only the classifier's score for that output. It does not prove that all unsafe content will be detected. Classifiers can produce false positives (for example, benign swimwear or beach scenes) and false negatives.

Before presenting the video pipeline as fail-closed, test benign content, policy-compliant disallowed test cases, classifier errors, unreadable video, and failed output scans. Confirm unapproved media is not placed in the approved output directory.

## Reproducing the workflows

1. Open the relevant notebook in Google Colab.
2. Select a GPU runtime compatible with the notebook; the recorded runs used a Tesla T4.
3. Run the notebook from the first cell in order.
4. Review and accept the model and dependency licenses before downloading or using them.
5. Set the prompt, seed, resolution, sampler, and other options.
6. Save the run log, including model revisions, settings, output metadata, elapsed time, and VRAM measurements.
7. Inspect the output visually before publishing it.

Colab availability, GPU type, memory, model downloads, dependency versions, and custom-node behavior can change. A successful past run does not guarantee an identical result in a future runtime.

## Repository layout

```text
LowVRAM-Lab/
├── README.md
├── LICENSE
├── .gitignore
├── notebooks/
│   ├── Qwen_Image_2_1_Q8_0_Colab_T4.ipynb
│   └── MiniMax_H3_I2VA_Colab_T4_Public.ipynb
├── examples/
│   ├── qwen_source_image.png
│   ├── minimax_h3_preview.mp4
│   ├── rife_final_output.mp4
│   └── preview_frames/
├── benchmarks/
│   └── results.csv
├── docs/
│   ├── reproducibility.md
│   ├── safety-and-limitations.md
│   └── model-licenses.md
└── results/
    └── README.md
```

## What to publish

Recommended: public notebooks with outputs cleared of private paths and tokens; small reviewed demonstration media; a CSV of measured results; exact model, ComfyUI, custom-node and dependency versions; and known limitations.

Do not commit model weights, API keys, access tokens, private datasets, caches, or unrelated runtime files. GitHub blocks regular Git files larger than 100 MiB and recommends keeping repositories small. Use Git LFS or release assets only when needed for larger media. See [GitHub repository limits](https://docs.github.com/en/repositories/creating-and-managing-repositories/repository-limits) and [Git LFS documentation](https://docs.github.com/en/repositories/working-with-files/managing-large-files/about-git-large-file-storage).

## Licenses and attribution

This repository integrates third-party models, custom nodes, and tools. Their licenses may differ from the license chosen for this repository. Before publishing, add exact model repositories, revisions, license names, custom-node repositories, and redistribution restrictions to `docs/model-licenses.md`. Do not assume weights can be redistributed merely because related code is open source.

## Roadmap

- [ ] Verify exact source-image dimensions and file size.
- [ ] Capture reliable GPU-wide peak-memory measurements during fresh runs.
- [ ] Record pre-RIFE and post-RIFE video metadata side by side.
- [ ] Validate generated-video moderation and fail-closed output handling.
- [ ] Pin model and custom-node revisions for reproducibility.
- [ ] Add a short architecture diagram and representative preview frames.

## Acknowledgements

Built using Google Colab, ComfyUI, Qwen-Image 2.1, MiniMax H3, RIFE, and their relevant model/custom-node ecosystems. See `docs/model-licenses.md` for precise attribution and licensing details.
#   l o w v r a m  
 