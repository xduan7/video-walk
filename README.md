<h1 style='font-size: 1.6em'>Space-Time Correspondence as a Contrastive Random Walk</h1>

<!-- ![](https://github.com/ajabri/videowalk/raw/master/figs/teaser_animation.gif) -->
<p align="center">
<img src="https://github.com/ajabri/videowalk/raw/master/figs/teaser_animation.gif" width="600">
</p>

This is the repository for *Space-Time Correspondence as a Contrastive Random Walk*, published at NeurIPS 2020, modified for more options and flexible usage.  

[[Paper](https://arxiv.org/abs/2006.14613)]
[[Project Page](http://ajabri.github.io/videowalk)]
[[Slides](https://www.dropbox.com/s/qrqb0ssjlh1tph1/jabri.nips.12min.public.key)]
[[Poster](https://www.dropbox.com/s/snpj68cssu3b4to/jabri.neurips2020.poster.pdf)]
[[Talk](https://youtu.be/UaOcjxrPaho)]

```
@inproceedings{jabri2020walk,
    Author = {Allan Jabri and Andrew Owens and Alexei A. Efros},
    Title = {Space-Time Correspondence as a Contrastive Random Walk},
    Booktitle = {Advances in Neural Information Processing Systems},
    Year = {2020},
}
```
Consider citing our work or acknowledging this repository if you found this code to be helpful :)

##  Requirements
- pytorch (>1.3)
- torchvision (0.6.0)
- cv2
- matplotlib
- skimage
- imageio

For visualization (`--visualize`):
- wandb
- visdom
- sklearn



## Train
An example training command is:
```
python -W ignore train.py --data-path /path/to/kinetics/ \
--frame-aug grid --dropout 0.1 --clip-len 4 --temp 0.05 \
--model-type scratch --workers 16 --batch-size 20  \
--cache-dataset --data-parallel --visualize --lr 0.0001
```

This yields a model with performance on DAVIS as follows (see below for evaluation instructions), provided as `pretrained.pth`:
```
 J&F-Mean    J-Mean  J-Recall  J-Decay    F-Mean  F-Recall   F-Decay
  0.67606  0.645902  0.758043   0.2031  0.706219   0.83221  0.246789
```

Arguments of interest:

* `--dropout`: The rate of edge dropout (default `0.1`).
* `--clip-len`: Length of video sequence.
* `--temp`: Softmax temperature.
* `--model-type`: Type of encoder. Use `scratch` or `scratch_zeropad` if training from scratch. Use `imagenet18` to load an Imagenet-pretrained network. Use `scratch` with `--resume` if reloading a checkpoint.
* `--batch-size`: I've managed to train models with batch sizes between 6 and 24. If you have can afford a larger batch size, consider increasing the `--lr` from 0.0001 to 0.0003.
* `--frame-aug`: `grid` samples a grid of patches to get nodes; `none` will just use a single image and use embeddings in the feature map as nodes.
* `--visualize`: Log diagonistics to `wandb` and data visualizations to `visdom`.

### Data

We use the official `torchvision.datasets.Kinetics400` class for training. You can find directions for downloading Kinetics [here](https://github.com/pytorch/vision/tree/master/references/video_classification). In particular, the code expects the path given for kinetics to contain a `train_256` subdirectory.

You can also provide `--data-path` with a file with a list of directories of images, or a path to a directory of directory of images. In this case, clips are randomly subsampled from the directory.


### Visualization
By default, the training script will log diagnostics to `wandb` and data visualizations to `visdom`.


### Pretrained Model
You can find the model resulting from the training command above at `pretrained.pth`.
We are still training updated ablation models and will post them when ready.

---

## Evaluation: Label Propagation
The label propagation algorithm is described in `test.py`.  The output of `test.py` (predicted label maps) must be post-processed for evaluation.

### DAVIS
To evaluate a trained model on the DAVIS task, clone the [davis2017-evaluation](https://github.com/davisvideochallenge/davis2017-evaluation) repository, and prepare the data by downloading the [2017 dataset](https://davischallenge.org/davis2017/code.html) and modifying the paths provided in `eval/davis_vallist.txt`. Then, run:


**Label Propagation:**
```
python test.py --filelist /path/to/davis/vallist.txt \
--model-type scratch --resume ../pretrained.pth --save-path /save/path \
--topk 10 --videoLen 20 --radius 12  --temperature 0.05  --cropSize -1
```
Though `test.py` expects a model file created with `train.py`, it can easily be modified to be used with other networks. Note that we simply use the same temperature used at training time.

You can also run the ImageNet baseline with the command below.
```
python test.py --filelist /path/to/davis/vallist.txt \
--model-type imagenet18 --save-path /save/path \
--topk 10 --videoLen 20 --radius 12  --temperature 0.05  --cropSize -1
```


**Post-Process:**  
```
# Convert
python eval/convert_davis.py --in_folder /save/path/ --out_folder /converted/path --dataset /davis/path/

# Compute metrics
python /path/to/davis2017-evaluation/evaluation_method.py \
--task semi-supervised   --results_path /converted/path --set val \
--davis_path /path/to/davis/
```

You can generate the above commands with the script below, where removing `--dryrun` will actually run them in sequence.
```
python eval/run_test.py --model-path /path/to/model --L 20 --K 10  --T 0.05 --cropSize -1 --dryrun
```


## Test-time Adaptation
To do.
