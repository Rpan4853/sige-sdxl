# Benchmark on DDIM and Progressive Distillation

## Setup

### Environment

* Create Conda environment:

  ```shell
  conda create -n sige python=3
  conda activate sige
  ```

  For Apple M1 Pro CPU results, we used [Intel Anaconda](https://repo.anaconda.com/archive/Anaconda3-2022.10-MacOSX-x86_64.pkg). The M1 Anaconda results are coming soon.

* Install [PyTorch](https://pytorch.org) >= 1.7. To reproduce our paper results, please use PyTorch 1.7.

* Install other dependencies:

  ```shell
  conda install pandas autopep8
  conda install tqdm -c conda-forge
  pip install blobfile torchprofile pyyaml lmdb clean-fid opencv-python gdown easydict dominate wget scikit-image lpips
  ```

* Install SIGE following [../README.md](../README.md#installation).

### Dataset

You could download the dataset with

```shell
python download_dataset.py
```

The dataset will be stored at `database/church_outdoor_sdedit`. The directory structure is like:

```text
database
└── church_outdoor_sdedit
    ├── edited
    ├── gt
    ├── masks
    └── original
```

Specifically,

* `original` stores the original images. 431 of them are the unpainted images by the [CoModGAN](https://github.com/zsyzzsoft/co-mod-gan) and the other 23 are the ground-truth images from [LSUN Church](https://github.com/fyu/lsun) dataset.
* `edited` stores the edited images. 431 of them are generated by the synthetic strokes and the other 23 are generated manually.
* `masks`  stores the difference masks between the original images and the edited images in `.npy` format.
* `gt` stores the corresponding ground-truth images from [LSUN Church](https://github.com/fyu/lsun) dataset. 

## Get Started

### DDIM

#### Quality Results

* Generate images:

  ```shell
  # For Vanilla DDIM
  python test.py \
  --config_path configs/church_ddim256-original.yml \
  --use_pretrained --save_dir results/vanilla_ddim
  
  # For DDIM with SIGE
  python test.py \
  --config_path configs/church_ddim256-sige.yml \
  --use_pretrained --save_dir results/sige_ddim
  ```

  The generated images will be stored in `results/vanilla_ddim` and `results/sige_ddim` correspondingly. 

  **Notice**: For SIGE inference, current codebase only supports caching the activations of a single step. As the diffusion models require multiple denoising steps, for each step, we first forward use the original model to pre-compute the activations and then forward the SIGE model to get the results.

* Metric Measurement:

  ```shell
  # FID
  python get_metric.py --metric fid --root $PATH_TO_GENERATED_IMAGE_DIRECTORY
  python --metric fid --root $PATH_TO_GENERATED_IMAGE_DIRECTORY
  
  # PSNR
  python --metric fid 
  ```
  


#### Efficient Results

You could measure the latency and MACs of **a single denoising step** with the following commands:

```shell
# For Vanilla DDIM
python test.py \
--config_path configs/church_ddim256-original.yml \
--mode profile --image_metas 432

# For DDIM with SIGE
python test.py \
--config_path configs/church_ddim256-sige.yml \
--mode profile --image_metas 432
```

Note:

* By default, these commands will test results on CUDA if CUDA is available. For CPU results, you could specify `--device cpu`.
* You could specify the test editing sample with `--image_metas`. It also support multiple samples, sperated by white space.

* You could change the number of the warmup and test rounds with `--warmup_times` and `--test_times`.

### Progressive Distillation

#### Quality Results

* Generate images:

  ```shell
  # For Vanilla Progressive Distillation
  python test.py \
  --config_path configs/church_pd128-original.yml \
  --use_pretrained --save_dir results/vanilla_pd
  
  # For Progressive Distllation with SIGE
  python test.py \
  --config_path configs/church_pd128-sige.yml \
  --use_pretrained --save_dir results/sige_pd
  ```

  The generated images will be stored in `results/vanilla_pd` and `results/sige_pd` correspondingly. 

  **Notice**: For SIGE inference, current codebase only supports caching the activations of a single step. As the diffusion models require multiple denoising steps, for each step, we first forward use the original model to pre-compute the activations and then forward the SIGE model to get the results.

#### Efficient Results

You could measure the latency and MACs of **a single denoising step** with the following commands:

```shell
# For Vanilla Progressive Distillation
python test.py \
--config_path configs/church_pd128-original.yml \
--mode profile --image_metas 432

# For Progressive Distllation with SIGE
python test.py \
--config_path configs/church_pd128-sige.yml \
--mode profile --image_metas 432
```

Note:

* To test the Progressive Distillation model with resolution 256x256, please use configurations `configs/church_pd256-original.yml` and `configs/church_pd256-sige.yml` correspondingly for vanilla PD and PD with SIGE. 
* By default, these commands will test results on CUDA if CUDA is available. For CPU results, you could specify `--device cpu`.
* You could specify the test editing sample with `--image_metas`. It also support multiple samples, sperated by white space.

* You could change the number of the warmup and test rounds with `--warmup_times` and `--test_times`.

### Metric Measurement

You could use the script `get_metric.py` to measure the PSNR, LPIPS and FID of your generated images. Specifically,

#### PSNR/LPIPS

```shell
# PSNR
python get_metric.py --metric psnr --root $PATH_TO_YOUR_GENERATED_IMAGES

# LPIPS
python get_metric.py --metric lpips --root $PATH_TO_YOUR_GENERATED_IMAGES
```

Note:

* By default, these commands will compute the metrics against the ground-truth images. If you want to compute the metrics against the images of the original model, please specify the directory of the original model results with `--ref_root` and `--mode original`.
* If you want to compute the metrics only at the edited regions, you could specify the mask root with `--mask_root database/church_outdoor_sdedit/masks `.

#### FID

```shell
python get_metric.py --metric fid --root $PATH_TO_YOUR_GENERATED_IMAGES
```
