# Hybrid Module with Multiple Receptive Fields and Self-Attention Layers for Medical Image Segmentation

This repository is the official implementation of **Hybrid Module with Multiple Receptive Fields and Self-Attention Layers for Medical Image Segmentation**. We provide the code which contains data preprocessing, training, and testing. Specifically,  we test all **13 organs** in the Synapse dataset instead of 8 organs, which is typically reported in previous works. Our implementation refers to the implementation of [nnUNet](https://github.com/MIC-DKFZ/nnUNet). 

# Table of contents  
- [Installation](#Installation) 
- [Data_Preparation](#Data_Preparation)
- [Data_Preprocessing](#Data_Preprocessing)
- [Training_and_Testing](#Training_and_Testing) 
- [Experiment_Results](#Experiment_Results) 
# Installation
```
git clone https://github.com/xxxxxxxx/AERFNet.git
cd AERFNet
conda env create -f environment.yml
source activate AERFNet
pip install -e .
```
# Data_Preparation
Our proposed model is a 2D based network, and all data should be expressed in 2D form with ```.nii.gz``` format. You can download the organized dataset from the [link](https://drive.google.com/drive/folders/1b4IVd9pOCFwpwoqfnVpsKZ6b3vfBNL6x?usp=sharing) or download the original data from the link below. If you need to convert other formats (such as ```.jpg```) to the ```.nii.gz```, you can look up the file and modify the [file](https://github.com/282857341/UNet-2022/blob/master/nnunet/dataset_conversion/Task120_ISIC.py) based on your own datasets.

**Dataset I**
[ACDC](https://www.creatis.insa-lyon.fr/Challenge/acdc/)

**Dataset II**
[The Synapse multi-organ CT dataset](https://www.synapse.org/#!Synapse:syn3193805/wiki/217789)

**Dataset III**
[EM](https://imagej.net/events/isbi-2012-segmentation-challenge#training-data)

The dataset should be finally organized as follows:
```
./DATASET/
  ├── nnUNet_raw/
      ├── nnUNet_raw_data/
          ├── Task01_ACDC/
              ├── imagesTr/
              ├── imagesTs/
              ├── labelsTr/
              ├── labelsTs/
              ├── dataset.json
              ├── evaulate.py

          ├── Task02_Synapse/
              ├── imagesTr/
              ├── imagesTs/
              ├── labelsTr/
              ├── labelsTs/
              ├── dataset.json
              ├── evaulate.py              
          ......
      ├── nnUNet_cropped_data/
  ├── nnUNet_trained_models/
  ├── nnUNet_preprocessed/
```
One thing you should be careful of is that folder imagesTr contains both training set and validation set, and correspondingly, the value of ```numTraining``` in dataset.json equals the case number in the imagesTr. The division of the training set and validation set will be done in the network configuration located at ```nnunet/network_configuration/config.py```.

The evaulate.py is used for calculating the evaulation metrics and can be found in the [link](https://drive.google.com/drive/folders/1b4IVd9pOCFwpwoqfnVpsKZ6b3vfBNL6x?usp=sharing) of the organized datasets or you can write it by yourself. The existing of evaulate.py will not affect the data preprocessing, training and testing.

# Data_Preprocessing
```
nnUNet_convert_decathlon_task -i path/to/nnUNet_raw_data/Task01_ACDC
```
This step will convert the name of folder from Task01 to Task001, and make the name of each nifti files end with '_000x.nii.gz'.
```
nnUNet_plan_and_preprocess -t 1
```
Where ```-t 1``` means the command will preprocess the data of the Task001_ACDC.
Before this step, you should set the environment variables to ensure the framework could know the path of ```nnUNet_raw```, ```nnUNet_preprocessed```, and ```nnUNet_trained_models```. The detailed construction can be found in [nnUNet](https://github.com/MIC-DKFZ/nnUNet/blob/master/documentation/setting_up_paths.md)

# Training_and_Testing
```
bash train_or_test.sh -c 0 -n unet2022_acdc -i 1 -s 0.5 -t true -p true 
```
- ```-c 0``` refers to the index of your Cuda device and this command only support the single-GPU training.
- ```-n unet2022_acdc``` denotes the suffix of the trainer located at ```UNet-2022/nnunet/training/network_training/```. For example, nnUNetTrainerV2_unet2022_acdc refers to ```-n unet2022_acdc```.
- ```-i 1``` means the index of the task. For example, Task001 refers to ```-i 1```.
- ```-s 0.5``` means the inference step size, reducing the value tends to bring better performance but longer inference time.
- ```-t true/false``` determines whether to run the training command.
- ```-p true/false``` determines whether to run the testing command.

The above command will run the training command and testing command continuously.

You can download the pre-trained weight based on ImageNet or our model weights from this [link](https://drive.google.com/drive/folders/1F9HnLCzWGqoC4BIQ-pDDlnWkmP9Y98Bj?usp=sharing). 

Before you start the training, please download the pre-trained weight and adjust the path to it in the trainer that located at ```nnunet/training/network_training```.

Before you start the testing, please make sure the model_best.model and model_best.model.pkl exists in the specified path, like this:
```
nnUNet_trained_models/nnUNet/2d/Task001_ACDC/nnUNetTrainerV2_unet2022_acdc/fold_0/model_best.model
nnUNet_trained_models/nnUNet/2d/Task001_ACDC/nnUNetTrainerV2_unet2022_acdc/fold_0/model_best.model.pkl
```
# Experiment_Results

Results on **ACDC dataset** (LV: left ventricle, RV: right ventricle, and MYO: myocardium.)
| Methods             | DSC       | RV       | MYO       | LV       |
| ----------------- | ----------- | ----------- | ----------- | ----------- |
| [ViT+CUP](https://arxiv.org/abs/1409.1556)              | 81.45      |81.46      |79.71      |92.18      |
| [TransUNet](https://arxiv.org/abs/1512.03385)          | 89.71      |88.86      |84.54     |95.73      |
| [SwinUNet](https://arxiv.org/abs/1512.03385)          | 90.00      |88.55      |85.62      |95.83      |
| [MT-UNet](https://arxiv.org/abs/1512.03385)         | 90.43      |86.64      |89.04      |95.62      |
| [nnUNet](https://arxiv.org/abs/1512.03385)         | 92.32      |90.39      |90.53      |96.05      |
| [UNet-2022](https://arxiv.org/abs/2003.13678)     | 92.83      |91.04      |90.97      |96.49      |
| [UNETR](https://arxiv.org/abs/2003.13678)     | 88.61      |85.29      |86.52      |94.02      |
| [nnFormer](https://arxiv.org/abs/1801.04381)       | 92.06      |90.94      |89.58      |95.65      |
| [**Ours**]  | **93.05**      |**91.41**      |**91.20**      |**96.35**      |

Results on **EM dataset**
| Methods             | mIoU       |
| ----------------- | ----------- |
| [UNet](https://arxiv.org/abs/1505.04597)              | 88.30      |
| [Wide UNet](https://arxiv.org/abs/2102.06442)          | 88.37      |
| [UNet+](https://arxiv.org/abs/1512.03385)          | 88.89      |
| [UNet++](https://arxiv.org/abs/1912.05074)         | 89.33      |
| [nnUNet](https://arxiv.org/abs/1809.10486)     | 90.55      |
| [DeSFiR](https://arxiv.org/abs/2109.05676)     | 90.92      |
| [UNet-2022](https://arxiv.org/abs/2210.15566)       | 91.05      |
| [FANet](https://arxiv.org/abs/2103.17235)  | 91.34      |
| [**Ours**]  | **92.14**      |

