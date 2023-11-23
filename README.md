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
Results on **Synapse dataset** (LV: left ventricle, RV: right ventricle, and MYO: myocardium.)
| \multirow{2}{*}{\textbf{Method}}  | \multirow{2}{*}{\textbf{Dim}} | \multicolumn{2}{c}{\textbf{Average}} | \multirow{2}{*}{Splee} | \multirow{2}{*}{RKid} | \multirow{2}{*}{LKid} | \multirow{2}{*}{Gall} | \multirow{2}{*}{Eso} | \multirow{2}{*}{Liver} | \multirow{2}{*}{Sto} | \multirow{2}{*}{Aorta} | \multirow{2}{*}{IVC} | \multirow{2}{*}{Veins} | \multirow{2}{*}{Pancr} | \multirow{2}{*}{RAd} | \multirow{2}{*}{LAd} |
|-----------------------------------|-------------------------------|--------------------------------------|------------------------|-----------------------|-----------------------|-----------------------|----------------------|------------------------|----------------------|------------------------|----------------------|------------------------|------------------------|----------------------|----------------------|
|                                   |                               | DSC \uparrow                         | HD  \downarrow         |                       |                       |                       |                      |                        |                      |                        |                      |                        |                        |                      |                      |                   |
| TransUNet~\cite{transunet2021}    | 2D                            | 70.17                                | 24.73                  | 84.78                 | 69.04                 | 76.09                 | 48.64                | 68.04                  | 94.24                | 73.51                  | 86.93                | 77.29                  | 65.61                  | 61.81                | 56.14                | 50.06             |
| SwinUNet~\cite{swinunet}          | 2D                            | 66.65                                | 24.01                  | 86.49                 | 72.62                 | 76.02                 | 55.14                | 63.22                  | 93.07                | 69.25                  | 80.84                | 66.92                  | 55.49                  | 52.17                | 52.39                | 42.87             |
| MT-UNet~\cite{MTUNet}             | 2D                            | 71.43                                | 22.40                  | 87.86                 | 74.93                 | 82.38                 | 57.91                | 73.64                  | 94.47                | 65.33                  | 86.22                | 75.09                  | 62.80                  | 55.52                | 60.03                | 52.39             |
| MISSFormer~\cite{MISSFormer}      | 2D                            | 76.87                                | 11.51                  | 87.81                 | 82.57                 | 86.31                 | 65.95                | 74.80                  | 95.72                | 79.13                  | 89.32                | 80.71                  | 69.29                  | 67.01                | 58.98                | 61.71             |
| nnUNet~\cite{nnunet}              | 2D                            | 79.92                                | 19.01                  | 90.09                 | 82.01                 | 87.16                 | 65.91                | 77.35                  | \underline{96.21}    | 79.84                  | \underline{91.28}    | \underline{85.22}      | 71.94                  | 73.98                | \textbf{69.39}       | 68.62             |
| UNet-2022~\cite{UNet2022}         | 2D                            | \underline{80.29}                    | 16.28                  | 89.65                 | 85.66                 | \underline{88.40}     | 58.91                | 76.58                  | 96.02                | 82.33                  | 90.91                | 84.70                  | \underline{75.20}      | \underline{77.81}    | 67.80                | 69.80             |
| V-Net~\cite{milletari2016v}       | 3D                            | 73.37                                | 18.94                  | 85.81                 | 83.00                 | 82.08                 | 60.06                | 67.79                  | 95.09                | 76.99                  | 85.49                | 81.16                  | 65.25                  | 63.50                | 50.11                | 57.47             |
| UNETR~\cite{hatamizadeh2022unetr} | 3D                            | 73.39                                | 15.81                  | 88.24                 | 83.77                 | 83.25                 | 62.89                | 70.69                  | 95.43                | 74.68                  | 85.63                | 77.83                  | 61.62                  | 59.42                | 61.48                | 49.14             |
| Swin UNETR~\cite{tang2022self}    | 3D                            | 77.04                                | 32.00                  | 86.61                 | 85.28                 | 85.12                 | \textbf{68.63}       | 75.84                  | 91.72                | 76.77                  | 89.09                | 83.81                  | 70.18                  | 65.03                | 60.10                | 63.33             |
| %TransBTSV2~\cite{}               | 3D                            | 81.45                                | 81.45                  | 81.45                 | 81.45                 | 81.45                 | 81.45                | 81.45                  | 81.45                | 81.45                  | 81.45                | 81.45                  | 81.45                  | 81.45                | 81.45                |
| nnUNet~\cite{nnunet}              | 3D                            | 78.46                                | \underline{11.07}      | \underline{91.17}     | \underline{86.20}     | 87.40                 | 63.92                | 75.31                  | 95.52                | 71.73                  | 89.91                | 83.76                  | 70.54                  | 70.16                | 68.84                | 65.53             |
| nnFormer~\cite{zhou2021nnformer}  | 3D                            | 80.24                                | 16.27                  | 88.79                 | 84.90                 | 83.23                 | 65.90                | \textbf{79.94}         | 95.80                | \underline{84.34}      | 89.97                | 84.07                  | 72.83                  | \textbf{78.83}       | 62.97                | \textbf{71.54}    |
| \textbf{Ours}                     | 2D                            | \textbf{82.13}                       | \textbf{10.89}         | \textbf{91.26}        | \textbf{87.81}        | \textbf{89.74}        | \underline{66.19}    | \underline{77.97}      | \textbf{96.48}       | \textbf{86.64}         | \textbf{91.87}       | \textbf{87.01}         | \textbf{77.76}         | 76.07                | \underline{69.03}    | \underline{69.89} |

Results on **ACDC dataset** (LV: left ventricle, RV: right ventricle, and MYO: myocardium.)
| Methods             | mIoU       |
| ----------------- | ----------- |
| [UNet](https://arxiv.org/abs/1409.1556)              | 88.30      |
| [Wide UNet](https://arxiv.org/abs/1512.03385)          | 88.37      |
| [UNet+](https://arxiv.org/abs/1512.03385)          | 88.89      |
| [UNet++](https://arxiv.org/abs/1512.03385)         | 89.33      |
| [nnUNet](https://arxiv.org/abs/2003.13678)     | 90.55      |
| [DeSFiR](https://arxiv.org/abs/2003.13678)     | 90.92      |
| [UNet-2022](https://arxiv.org/abs/1801.04381)       | 91.05      |
| [FANet](https://arxiv.org/abs/1611.05431)  | 91.34      |
| [**Ours**](https://arxiv.org/abs/1611.05431)  | **92.14**      |


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
| **Ours**  | **92.14**      |

