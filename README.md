# [【CVPR 2026 Highlight】 VX-CODE: Explaining Object Detectors via Collective Contribution of Pixels](https://arxiv.org/abs/2412.00666)

[📝 Paper](https://openaccess.thecvf.com/content/CVPR2026/html/Yamauchi_Explaining_Object_Detectors_via_Collective_Contribution_of_Pixels_CVPR_2026_paper.html) | [📌 Citation](#citation)

---

Official implementation of VX-CODE, proposed in “Explaining Object Detectors via Collective Contribution of Pixels.”
VX-CODE is a visual explanation method for object detectors that identifies regions contributing most to detections using greedy patch selection that considers both Shapley values and interactions.

![VX-CODE qualitative example](images/img1.jpg)

The overall framework is shown below:
![VX-CODE framework overview](images/img2.jpg)

## Highlights

- Ready-to-run script `run_vxcode.py` for COCO val images with Faster R-CNN or DETR.
- Download helpers for pretrained weights (`download_weights.sh`) and COCO 2017 val images/annotations (`download_coco_dataset.sh`).
- Minimal configs under `configs/` and sample outputs under `results/` to verify the pipeline.
- Built on [Detectron2](https://github.com/facebookresearch/detectron2) for the base implementations.
- The DETR implementation is sourced from [facebookresearch/detr](https://github.com/facebookresearch/detr) and adapted in this repository.

## Repository map

- `src/vx_code/run_vxcode.py`: entry point to build a detector, run VX-CODE, and save heatmaps/patch visualizations.
- `configs/*.yaml`: Detectron2 configs for Faster R-CNN and DETR variants.
- `weights/`: expected location for pretrained weights downloaded by the helper script.
- `datasets/`: place MS-COCO val2017 images and annotations here (Detectron2 directory layout).
- `src/`: VX-CODE implementation (`src/vx_code/explanations/vxcode.py`), visualization helpers, and bundled DETR code.

## Setup

1. Clone the repository and pull submodules (Detectron2 lives in `third_party/detectron2`):
   ```bash
   git clone https://github.com/tttt-0814/VX-CODE.git && cd VX-CODE
   ```
2. Install dependencies with:
   ```bash
   uv sync --no-build-isolation
   ```

## Data and weights

- COCO val2017 + annotations:
  ```bash
  bash download_coco_dataset.sh  # defaults to datasets/coco
  ```
  Detectron2 expects `datasets/coco/val2017` and `datasets/coco/annotations/instances_val2017.json`.
- Pretrained detector weights:
  ```
  bash download_weights.sh  # downloads Faster R-CNN and DETR checkpoints into weights/
  ```
  For DETR, convert the torch-hub checkpoint to the Detectron2 format before use:
  ```bash
  uv run python src/models/detr/d2/converter.py \
    --source_model weights/detr/detr-r50-e632da11.pth \
    --output_model weights/detr/detr-r50-e632da11-d2.pth
  ```

## Running VX-CODE

Run on Faster R-CNN (default config/weights):

```bash
uv run vxcode \
  --config configs/dev_faster_rcnn_R_50_FPN_1x.yaml \
  --weights weights/faster_rcnn/model_final_b275ba.pkl \
  --dataset coco_2017_val \
  --save_dir results/vxcode
```

Run on DETR:

```bash
uv run vxcode \
  --model_name detr \
  --config configs/dev_detr_256_6_6_torchvision.yaml \
  --weights weights/detr/detr-r50-e632da11-d2.pth \
  --dataset coco_2017_val \
  --save_dir results/vxcode
```

Useful flags for VX-CODE:

- `--vxcode_mode`: `del` (deletion) or `ins` (insertion) in patch selection.
- `--num_patches_per_step`: number of patches identified per step (corresponding to $r$ in the paper).

## Notes

- The scripts assume COCO is registered in Detectron2 as `coco_2017_val`. Adjust `--dataset` if you use a custom name.
- The code switches to CPU automatically when CUDA is unavailable, but VX-CODE runs are considerably faster on GPU.
- Increasing `--num_patches_per_step` (i.e., $r$) significantly raises the computational cost and runtime; we recommend setting it to $r \le 3$.

## Citation

If you use this repository in your research, please cite:

```
@InProceedings{Yamauchi_2026_CVPR,
    author    = {Yamauchi, Toshinori and Kera, Hiroshi and Kawamoto, Kazuhiko},
    title     = {Explaining Object Detectors via Collective Contribution of Pixels},
    booktitle = {Proceedings of the IEEE/CVF Conference on Computer Vision and Pattern Recognition (CVPR)},
    month     = {June},
    year      = {2026},
    pages     = {17046-17056}
}
```
