# Deep Image Retrieval

This repository contains the models and the evaluation scripts (in Python3 and Pytorch 1.0) of the papers:

**[1] End-to-end Learning of Deep Visual Representations for Image Retrieval**
Albert Gordo, Jon Almazan, Jerome Revaud, Diane Larlus, IJCV 2017 [\[PDF\]](https://arxiv.org/abs/1610.07940)

**[2] Learning with Average Precision: Training Image Retrieval with a Listwise Loss**
Jerome Revaud, Rafael S. Rezende, Cesar de Souza, Jon Almazan, arXiv 2019 [\[PDF\]](https://arxiv.org/abs/1906.07589)


Both papers tackle the problem of image retrieval and explore different ways to learn deep visual representations for this task. In both cases, a CNN is used to extract a feature map that is aggregated into a compact, fixed-length representation by a global-aggregation layer*. Finally, this representation is first projected with a FC, and then L2 normalized so images can be efficiently compared with the dot product.


![dir_network](https://user-images.githubusercontent.com/228798/59742085-aae19f80-9221-11e9-8063-e5f2528c304a.png)

All components in this network, including the aggregation layer, are differentiable, which makes it end-to-end trainable for the end task. In [1], a Siamese architecture that combines three streams with a triplet loss was proposed to train this network.  In [2], this work was extended by replacing the triplet loss with a new loss that directly optimizes for Average Precision.

![Losses](https://user-images.githubusercontent.com/228798/59742025-7a9a0100-9221-11e9-9d58-1494716e9071.png)

\* Originally, [1] used R-MAC pooling [3] as the global-aggregation layer. However, due to its efficiency and better performace we have replaced the R-MAC pooling layer with the Generalized-mean pooling layer (GeM) proposed in [4]. You can find the original implementation of [1] in Caffe following [this link](https://europe.naverlabs.com/Research/Computer-Vision/Learning-Visual-Representations/Deep-Image-Retrieval/).

## Pre-requisites

In order to run this toolbox you will need:

- Python3 (tested with Python 3.7.3)
- PyTorch (tested with version 1.0.1)
- The following packages: matplotlib, tqdm, scikit-learn

With conda you can run the following commands:

```
conda install numpy matplotlib tqdm scikit-learn
conda install pytorch torchvision cudatoolkit=10.0 -c pytorch
```

## Installation

```
# Download the code
git clone git@es.naverlabs.com:jon-almazan/deep-image-retrieval.git

# Create env variables
cd deep-image-retrieval
export DIR_ROOT=$PWD
export DB_ROOT=/PATH/TO/YOUR/DATASETS
# for example: export DB_ROOT=$PWD/dirtorch/data/datasets
```


## Evaluation


### Pre-trained models

The table below contains the pre-trained models that we provide with this library, together with their mAP performance on some of the most well-know image retrieval benchmakrs: [Oxford5K](http://www.robots.ox.ac.uk/~vgg/data/oxbuildings/), [Paris6K](http://www.robots.ox.ac.uk/~vgg/data/parisbuildings/), and their Revisited versions ([ROxford5K and RParis6K](https://github.com/filipradenovic/revisitop)).


| Model | Oxford5K | Paris6K |  ROxford5K (med/hard) | RParis6K (med/hard) |
|---	|:-:|:-:|:-:|:-:|
|  [Resnet101-TL-MAC]() |  84.2 	|  91.0| 63.6 / 37.1 	|   76.7 / 55.7  |
|  [Resnet101-TL-GeM](https://drive.google.com/open?id=1vhm1GYvn8T3-1C4SPjPNJOuTU9UxKAG6) | 85.5 | 93.4 | 64.8 / 41.6	|  78.9 / 59.4  |
|  [Resnet50-AP-GeM]() | 87.9 	| 91.9 | 65.8 / 41.7| 77.6 / 57.3 |
|  [Resnet101-AP-GeM](https://drive.google.com/open?id=1UWJGDuHtzaQdFhSMojoYVQjmCXhIwVvy) | 89.3 | 93.0 | 67.4 / 42.8|  80.4/61.0 |
|  [Resnet101-AP-GeM-LM18]()** |  88.4	| 93.0 | 66.5 / 43.1	|   80.2 / 60.4  |


The name of the model encodes the backbone architecture of the network and the loss that has been used to train it (TL for triplet loss and AP for Average Precision loss). All models use **Generalized-mean pooling (GeM)** [3] as the global pooling mechanism, except for the model in the first row that uses MAC [3] \(i.e. max-pooling), and have been trained on the **Landmarks-clean** [1] dataset (the clean version of the [Landmarks dataset](http://sites.skoltech.ru/compvision/projects/neuralcodes/)) directly **fine-tuning from ImageNet**. These numbers have been obtained using a **single resolution** and applying **whitening** to the output features (which has also been learned on Landmarks-clean). For a detailed explanation of all the hyper-parameters see [1] and [2] for the triplet loss and AP loss models, respectively.

** For the sake of completeness, we have added an extra model, `Resnet101-AP-LM18`, which has been trained on the [Google-Landmarks Dataset](https://www.kaggle.com/google/google-landmarks-dataset), a large dataset consisting of more than 1M images and 15K classes.

### Reproducing the results

The script `test_dir.py` can be used to evaluate the pre-trained models provided and to reproduce the results above:

```
python -m dirtorch.test_dir --dataset DATASET --checkpoint PATH_TO_MODEL
		[--whiten DATASET] [--whitenp POWER] [--aqe ALPHA-QEXP]
		[--trfs TRANSFORMS] [--gpu ID] [...]
```

- `--dataset`: selects the dataset (eg.: Oxford5K, Paris6K, ROxford5K, RParis6K) [**required**]
- `--checkpoint`: path to the model weights [**required**]
- `--whiten`: applies whitening to the output features [default 'Landmarks_clean']
- `--whitenp`: whitening power [default: 0.25]
- `--aqe`: alpha-query expansion parameters [default: None]
- `--trfs`: input image transformations (can be used to apply multi-scale) [default: None]
- `--gpu`: selects the GPU ID (-1 selects the CPU)

For example, to reproduce the results of the Resnet101-AP_loss model on the RParis6K dataset download the model `Resnet101-AP-GeM` and run:

```
cd $DIR_ROOT
export DB_ROOT=/PATH/TO/YOUR/DATASETS

python -m dirtorch.test_dir --dataset RParis6K
		--checkpoint dirtorch/data/Resnet101-AP-GeM.pt
		--whiten Landmarks_clean --whitenp 0.25 --gpu 0
```

And you should see the following output:

```
>> Evaluation...
 top1 not implemented!
 * mAP-easy = 0.911001
 * mAP-medium = 0.80115
 * mAP-hard = 0.604583
```

**Note:** this script integrates an automatic downloader for the Oxford5K, Paris6K, ROxford5K, and RParis6K datasets (kudos to Filip Radenovic ;)). The datasets will be saved in `$DB_ROOT`.

## Feature extractor

You can also use the pre-trained models to extract features from your own datasets or collection of images. For that we provide the script `feature_extractor.py`:

```
python -m dirtorch.extract_features --dataset DATASET --checkpoint PATH_TO_MODEL
		--output PATH_TO_FILE [--whiten DATASET] [--whitenp POWER]
		[--trfs TRANSFORMS] [--gpu ID] [...]
```

where `--output` is used to specify the destination where the features will be saved. The rest of the parameters are the same as seen above.

The library provides a generic class dataset (`ImageList`) that allows you to specify the list of images by providing a simple text file.

```
--dataset 'ImageList("PATH_TO_TEXTFILE" [, "IMAGES_ROOT"])'
```

Each row of the text file should contain a single path to a given image:

```
/PATH/TO/YOUR/DATASET/images/image1.jpg
/PATH/TO/YOUR/DATASET/images/image2.jpg
/PATH/TO/YOUR/DATASET/images/image3.jpg
/PATH/TO/YOUR/DATASET/images/image4.jpg
/PATH/TO/YOUR/DATASET/images/image5.jpg
```

Alternatively, you can also use relative paths, and use `IMAGES_ROOT` to specify the root folder.


## Citations

Please consider citing the following papers in your publications if this helps your research.

```
@article{GARL17,
 title = {End-to-end Learning of Deep Visual Representations for Image Retrieval},
 author = {Gordo, A. and Almazan, J. and Revaud, J. and Larlus, D.}
 journal = {IJCV},
 year = {2017}
}

@inproceedings{RARS19,
 title = {Learning with Average Precision: Training Image Retrieval with a Listwise Loss},
 author = {Revaud, J. and Almazan, J. and Rezende, R.S. and de Souza, C.R.}
 booktitle = {ArXiv},
 year = {2019}
}
```

## Contributors

This library has been developed by Jerome Revaud, Rafael de Rezende, Cesar de Souza, Diane Larlus, and Jon Almazan at **[Naver Labs Europe](https://europe.naverlabs.com)**.


**Special thanks to Filip Radenovic.** We have used in this library the ROxford5K and RParis6K downloader from his awesome **[CNN-imageretrieval repository](https://github.com/filipradenovic/cnnimageretrieval-pytorch)**. Consider checking it out if you want to train your own models for image retrieval!

## References

[1] Gordo, A., Almazan, J., Revaud, J., Larlus, D., [End-to-end Learning of Deep Visual Representations for Image Retrieval](https://arxiv.org/abs/1610.07940). IJCV 2017

[2] Revaud, J., de Souza, C., Rezende, R.S., Almazan, J., [Learning with Average Precision: Training Image Retrieval with a Listwise Loss](https://arxiv.org/abs/1906.07589). ArXiv 2019

[3] Tolias, G., Sicre, R., Jegou, H., [Particular object retrieval with integral max-pooling of CNN activations](https://arxiv.org/abs/1511.05879). ICLR 2016

[4] Radenovic, F., Tolias, G., Chum, O., [Fine-tuning CNN Image Retrieval with No Human Annotation](https://arxiv.org/pdf/1711.02512). TPAMI 2018
