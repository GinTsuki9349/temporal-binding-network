# Temporal Binding Network


This repository implements the model proposed in the paper:

Evangelos Kazakos, Arsha Nagrani, Andrew Zisserman, Dima Damen, <strong>EPIC-Fusion: Audio-Visual Temporal Binding for Egocentric Action Recognition</strong>, <em>ICCV</em>, 2019

[Project's webpage](https://ekazakos.github.io/TBN/)

[ArXiv paper](https://arxiv.org/abs/1908.08498)

**Tested using python 3.6.8, Pytorch 1.1, and CUDA 8.0**

## Data preparation

### Visual data

This step assumes that you've downloaded the RGB and Flow frames of EPIC dataset using [this script](https://github.com/epic-kitchens/download-scripts/blob/master/download_frames_rgb_flow.sh), and your copy of the dataset has the same folder structure 
provided in the script (which can be found [here](https://data.bris.ac.uk/data/dataset/3h91syskeag572hl6tvuovwv4d)). Also you should untar each video's frames in its corresponding folder. 

`dataset.py` uses a unified folder structure for all datasets, which is the same as the one used in the [TSN code](https://github.com/yjxiong/tsn-pytorch). Example of the folder structure for RGB and Flow:

```
├── dataset_root
|   ├── video1
|   |   ├── img_0000000000
|   |   ├── x_0000000000
|   |   ├── y_0000000000
|   |   ├── .
|   |   ├── .
|   |   ├── .
|   |   ├── img_0000000100
|   |   ├── x_0000000100
|   |   ├── y_0000000100
|   ├── .
|   ├── .
|   ├── .
|   ├── video10000
|   |   ├── img_0000000000
|   |   ├── x_0000000000
|   |   ├── y_0000000000
|   |   ├── .
|   |   ├── .
|   |   ├── .
|   |   ├── img_0000000250
|   |   ├── x_0000000250
|   |   ├── y_0000000250
```
        
To map the folder structure of EPIC to the above folder structure I've used symlinks. Use the following script to convert
the original folder structure of EPIC to the folder structure above:

```
python preprocessing_epic/symlinks.py /path/to/rgb_flow/ /path/to/output
```

### Audio data

This step assumes that you've downloaded the videos of EPIC using [this script](https://github.com/epic-kitchens/download-scripts/blob/master/download_videos.sh).

To extract the audio from the videos, run:

```
python preprocessing_epic/extract_audio.py /path/to/videos /path/to/ouput
```

To load the audio in `dataset.py`, Im using a dictionary, where the keys are the video names and the values are the extracted audio from the previous step. To save the extracted audio into a dictionary, run:

```
python preprocessing_epic/wav_to_dict.py /path/to/audio /path/to/output
```

If you don't want to save the audio in a dictionary, and prefer to load the wav files directly in `dataset.py`, you can
change set `use_audio_dict=False` in `TBNDataset` in `dataset.py`.


## Training

To reproduce the results of the full RGB, Flow, Audio model, run:
```
python train.py epic RGB Flow Spec --train_list /path/to/EPIC_train_action_labels.pkl --val_list /path/to/EPIC_val_action_labels.pkl --visual_path /path/to/rgb+flow --audio_path /path/to/audio --arch BNInception 
--num_segments 3 --dropout 0.5 --epochs 80 -b 128 --lr 0.01 --lr_steps 60 --gd 20 --partialbn --eval-freq 1 -j 40 
--pretrained_flow_weights
```

Individual modalities can be trained, as well as any combination of 2 modalities. 
To train audio, run:
```
python train.py epic Spec --train_list /path/to/EPIC_train_action_labels.pkl --val_list /path/to/EPIC_val_action_labels.pkl 
--audio_path /path/to/audio --arch BNInception --num_segments 3 
--dropout 0.5 --epochs 80 -b 128 --lr 0.001 --lr_steps 60 --gd 20 --partialbn --eval-freq 1 -j 40 
```

To train RGB, run:
```
python train.py epic RGB  --train_list /path/to/EPIC_train_action_labels.pkl --val_list /path/to/EPIC_val_action_labels.pkl 
--visual_path /path/to/rgb+flow --arch BNInception --num_segments 3 --dropout 0.5 
--epochs 80 -b 128 --lr 0.01 --lr_steps 60 --gd 20 --partialbn --eval-freq 1 -j 40 
```

To train flow, run:
```
python train.py epic Flow  --train_list /path/to/EPIC_train_action_labels.pkl --val_list /path/to/EPIC_val_action_labels.pkl 
--visual_path /path/to/rgb+flow --arch BNInception --num_segments 3 --dropout 0.5 
--epochs 80 -b 128 --lr 0.001 --lr_steps 60 --gd 20 --partialbn --eval-freq 1 -j 40 --pretrained_flow_weights
```

Example of training RGB+Audio (any other combination can be used):
```
python train.py epic RGB Spec --train_list /path/to/EPIC_train_action_labels.pkl --val_list /path/to/EPIC_val_action_labels.pkl --visual_path /path/to/rgb+flow --audio_path /path/to/audio --arch BNInception 
--num_segments 3 --dropout 0.5 --epochs 80 -b 128 --lr 0.01 --lr_steps 60 --gd 20 --partialbn --eval-freq 1 -j 40 
```

`EPIC_train_action_labels.pkl` and `EPIC_val_action_labels.pkl` should be the result of spliting the original [EPIC_train_action_labels.pkl](https://github.com/epic-kitchens/annotations/blob/master/EPIC_train_action_labels.csv) into training and validation set. 

## Testing

To compute scores, save scores and labels, and print the accuracy of the validation set using the full modalities, run:

```
python test.py epic RGB Flow Spec path/to/checkpoint --test_list /path/to/EPIC_val_action_labels.pkl --visual_path /path/to/rgb+flow --audio_path /path/to/audio --arch BNInception --scores_root scores/ --test_segments 25 --test_crops 1  --dropout 0.5 -j 40
```

To compute and save scores of the test sets (S1/S2) (since we do not have access to the labels), run:

```
python test.py epic RGB Flow Spec path/to/checkpoint --test_list /path/to/EPIC_test_s1_timestamps.pkl /path/to/EPIC_test_s2_timestamps.pkl --visual_path /path/to/rgb+flow --audio_path /path/to/audio --arch BNInception --scores_root scores/ --test_segments 25 --test_crops 1  --dropout 0.5 -j 40
```

`EPIC_test_s1_timestamps.pkl` and `EPIC_test_s2_timestamps.pkl` can be found [here](https://github.com/epic-kitchens/annotations). Similarly testing can be done for any combination of modalities, or individual modalities.

## Publication

Please cite our paper if you find this code useful:

```
@InProceedings{kazakos2019TBN,
    author    = {Kazakos, Evangelos and Nagrani, Arsha and Zisserman, Andrew and Damen, Dima},
    title     = {EPIC-Fusion: Audio-Visual Temporal Binding for Egocentric Action Recognition},
    booktitle = {IEEE/CVF International Conference on Computer Vision (ICCV)},
    year      = {2019}
}
```
