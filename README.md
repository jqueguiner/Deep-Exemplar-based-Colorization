# Deep Exemplar-based Colorization

This is the implementation of paper [**Deep Exemplar-based Colorization**](https://arxiv.org/abs/1807.06587) by Mingming He*, [Dongdong Chen*](http://www.dongdongchen.bid/),
[Jing Liao](https://liaojing.github.io/html/index.html), [Pedro V. Sander](http://www.cse.ust.hk/~psander/) and 
[Lu Yuan](http://www.lyuan.org/) in ACM Transactions on Graphics (SIGGRAPH 2018) (*indicates equal contribution).


## Introduction

**Deep Exemplar-based Colorization** is the ﬁrst deep learning approach for exemplar-based local colorization. 
Given a reference color image, our convolutional neural network directly maps a grayscale image to an output colorized image.

![image](https://github.com/msracver/Deep-Exemplar-based-Colorization/blob/master/demo/data/representative.jpg)

The proposed network consists of two sub-networks, **Similarity Sub-net** which computes the semantic similarities between 
the reference and the target, and **Colorization Sub-net** which selects, propagates and predicts the chrominances channels of the target.

The input includes a grayscale target image, a color reference image and bidirectional mapping functions. We use [*Deep Image Analogy*](https://github.com/msracver/Deep-Image-Analogy) as default to generate bidirectional mapping functions. It is applicable to replace with other dense correspondence estimation algorithms.

The code of the part **Color Reference Recommendation** is now released. Please refere to [Gray-Image-Retrieval](https://github.com/hmmlillian/Gray-Image-Retrieval) for more details.

For more results, please refer to our [Supplementary](http://www.dongdongchen.bid/supp/deep_exam_colorization/index.html).


## License

© Microsoft, 2017. Licensed under a MIT license.


## Getting Started

### Prerequisites
- **Similarity Sub-net**: 
  - Windows (64bit)
  - NVIDIA GPU (CUDA 8.0 & CuDNN 5)
  - Visual Studio 2013

- **Colorization Sub-net**:
  - Pytorch & the 3rd party Python libraries (OpenCV, scikit-learn and scikit-image)

### Build
**Similarity Sub-net** is implemented in C++ combined with CUDA and requires compiling in Visual Studio as follows:
- Build [Caffe](http://caffe.berkeleyvision.org/) at first. Just follow the tutorial [here](https://github.com/Microsoft/caffe).
- Edit ```similarity_combo.vcxproj``` under ```similarity_subnet\windows\similarity_combo\``` to make the CUDA version in it match yours.
- Open solution ```Caffe``` and add ```similarity_combo.vcxproj```.
- Build project ```similarity_combo```.
- (Optional) If you use *Deep Image Analogy*, please add ```deep_image_analogy.vcxproj``` under ```similarity_subnet\windows\deep_image_analogy\``` and build it.

### Download Models
You need to download models before running a demo.
- Go to ```demo\models\similarity_subnet\vgg_19_gray_bn\``` folder and download:  
  https://www.dropbox.com/s/mnsxsfv5non3e81/vgg19_bn_gray_ft_iter_150000.caffemodel?dl=0
- Go to ```demo\models\colorization_subnet\``` folder and download: 
  https://www.dropbox.com/s/ebtuwj7doteelia/example_net.pth?dl=0
- (Optional) If you use *Deep Image Analogy*, please go to ```demo\models\deep_image_analogy\vgg19\``` folder and download:  
  http://www.robots.ox.ac.uk/~vgg/software/very_deep/caffe/VGG_ILSVRC_19_layers.caffemodel

### Demo
We prepare an example under the folder ```demo\``` with:

(1) Input data folder ```example\``` including two parts:
- A folder ```input\``` with the input images (grayscale target images and color reference images) inside.
- A file ```pairs.txt``` to specify a target, a reference and a flag (1 as default) as an example in each line, e.g., 
  ```
  in1.jpg ref1.jpg 1
  in2.jpg ref2.jpg 1
  ...
  ```

(2) Executable script ```run.bat``` including three commands:
- (Optional) A command to generate bidirectional mapping functions using *Deep Image Analogy*:
  ```
  deep_image_analogy.exe [MODEL_DIR] [INPUT_ROOT_DIR] [START_LINE_ID] [END_LINE_ID] [GPU_ID]
  e.g., exe\deep_image_analogy.exe models\deep_image_analogy\ example\ 0 2 0
  ```  
  (Note if you use other algorithms for bidirectional mapping functions, please generate flow files referring to the format of those by *Deep Image Analogy* and put them to the folder ```example\flow\```.)

- A command to generate similarity maps for colorization (**Similarity Subnet**):
  ```
  similarity_combo.exe [MODEL_DIR] [INPUT_ROOT_DIR] [START_LINE_ID] [END_LINE_ID] [GPU_ID]
  e.g., exe\similarity_combo.exe models\similarity_subnet\ example\ 0 2 0
  ```

- A command to do colorization with our pretrained model (**Colorization Subnet**):
  ```
  python test.py --short_size [SHORT_EDGE_SIZE] --test_model [MODEL_FILE] --data_root [INPUT_ROOT_DIR] --out_dir [OUTPUT_DIR] --gpu_id [GPU_ID]
  e.g., python ..\colorization_subnet\test.py --short_size 256 --test_model models\colorization_subnet\example_net.pth --data_root example\ --out_dir example\res\ --gpu_id 0
  ```
### Demo for Linux / Docker
#### Building the docker
```
docker build -t deep-colorization -f Dockerfile .
```

#### Before running
If you want to run the provided demo

This section requires nvidia-docker (install instruction available here :[https://github.com/nvidia/nvidia-docker/wiki/Installation-(version-2.0)](https://github.com/nvidia/nvidia-docker/wiki/Installation-(version-2.0))

#### Running the docker for the demo
```
 nvidia-docker run -it --ipc=host deep-colorization
```

#### Running the demo
once in the Docker
```
(torch) root@84ccb98c1b2e:/src/app# ls
colorization_subnet  demo  requirements.txt  similarity_subnet
(torch) root@84ccb98c1b2e:/src/app# cd demo/
(torch) root@84ccb98c1b2e:/src/app/demo# ls
data  example  exe  models  run.bat  run.sh run-custom.sh
(torch) root@84ccb98c1b2e:/src/app/demo# ./run.sh
```

#### Inputs
inputs look like:
```
(torch) root@3a808ffe15a4:/src/app/demo/example/input# ls
in1.jpg  in2.JPEG  ref1.jpg  ref2.JPEG
```
with in*.jpg being the original images to colorize and ref*.jpg the colorized image to transfer from

#### Outputs
outputs will be place under the /src/app/demo/example/res folder

```
(torch) root@3a808ffe15a4:/src/app/demo/example/res# ls
in1_ref1.png  in2_ref2.png
```

#### Running the demo on your local images
If you want to run on your custom local images
```
nvidia-docker run -it --ipc=host -v /your/local/path/to/images:/src/app/custom-examples deep-colorization
```

Once in the docker
```
# /src/app/demo/run-custom.sh
```

Or directly 
```
nvidia-docker run -it --ipc=host -v /your/local/path/to/images:/src/app/custom-examples deep-colorization /src/app/demo/run-custom.sh
```

results will be placed on your local machine under /your/local/path/to/images/res

### Run
We provide pre-built executable files in folder ```demo\exe\```, please try them.

### Tips
Our test input images are resized to w x h (min(w, h)=256) considering the cost of computing bidirectional mapping functions by *Deep Image Analogy*. But we also support higher resolution input images.


## Citation
If you find **Deep Exemplar-based Colorization** helpful for your research, please consider citing:
```
@article{he2018deep,
  title={Deep exemplar-based colorization},
  author={He, Mingming and Chen, Dongdong and Liao, Jing and Sander, Pedro V and Yuan, Lu},
  journal={ACM Transactions on Graphics (TOG)},
  volume={37},
  number={4},
  pages={47},
  year={2018},
  publisher={ACM}
}
```
