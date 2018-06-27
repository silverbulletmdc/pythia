# Pythia
A software suite for VQA+VisDial

Goal: building a software suite with VQA+VisDial models, datasets, data-loaders, etc. for enabling easy and fair comparisons and accelerating VQA research

## Getting Started

These instructions will get you a copy of the project up and running on your local machine for testing.


### Installing

1. Install Anaconda (Anaconda recommended: https://www.continuum.io/downloads).
2. Install cudnn/v7.0-cuda.9.0
3. Create envioment for vqa-suite
```bash
conda create --name vqa python=3.6

source activate vqa
pip install demjson pyyaml

pip install http://download.pytorch.org/whl/cu90/torch-0.3.1-cp36-cp36m-linux_x86_64.whl

pip install torchvision
pip install tensorboardX

```




### Quick start
We provide the files that are required to start our program directly. In this way, you can directly start our programs. 
instead of using the original train2014 and val2014 set, we split the val2014 to val2train2014 and minival2014. In this way,
train2014 + val2train2014 are used as training dataset while minival2014 is used as validation set.

Download data, this step may take some time
```bash

git clone git@github.com:fairinternal/VQA_suite.git
cd VQA_suite

mkdir data
cd data
wget https://s3-us-west-1.amazonaws.com/vqa-suite/data/vqa2.0_glove.6B.300d.txt.npy
wget https://s3-us-west-1.amazonaws.com/vqa-suite/data/vocabulary_vqa.txt
wget https://s3-us-west-1.amazonaws.com/vqa-suite/data/answers_vqa.txt
wget https://s3-us-west-1.amazonaws.com/vqa-suite/data/imdb.tar.gz
wget https://s3-us-west-1.amazonaws.com/vqa-suite/data/rcnn_10_100.tar.gz
wget https://s3-us-west-1.amazonaws.com/vqa-suite/data/detectron_23.tar.gz
wget https://s3-us-west-1.amazonaws.com/vqa-suite/data/large_vocabulary_vqa.txt
wget https://s3-us-west-1.amazonaws.com/vqa-suite/data/large_vqa2.0_glove.6B.300d.txt.npy
gunzip imdb.tar.gz 
tar -xf imdb.tar

gunzip rcnn_10_100.tar.gz 
tar -xf rcnn_10_100.tar
rm -f rcnn_10_100.tar

gunzip detectron_23.tar.gz
tar -xf detectron_23.tar
rm -f detectron_23.tar

```
Run model without finetune
```bash
cd ../
python train.py
```
if there is a out of memory error, try:
```bash
python train.py --config_overwrite '{data:{image_fast_reader:false}}'
```

Run model with finetune
```bash
python train.py --config config/keep/detectron23_finetune_minival.yaml

```
Check result
```bash
cd results/default/1234

```
The results folder contains following info
```angular2html
results
|_ default
|  |_ 1234
|  |  |_config.yaml
|  |  |_best_model.pth
|  |  |_best_model_predict_test.pkl 
|  |  |_best_model_predict_test.json (json file for predicted results on test dataset)
|  |  |_model_00001000.pth (mpdel snapshot at iter 1000)
|  |  |_result_on_val.txt
|  |  |_ ...
|  |_(other_cofig_setting)
|  |  |_...
|_ (other_config_file)
|

```
The log files for tensorbord is stored under boards/



### Preprocess dataset
If you want to start from the original VQA dataset and preprocess data by yourself, use the follow instructions. 
***This part is not necessary if you download all data from quick start.***

#### VQA v2.0
 
Dowload dataset 
```bash
cd ../
mkdir -p orig_data/vqa_v2.0
cd orig_data/vqa_v2.0
./../../data_prep/vqa_v2.0/download_vqa_2.0.sh

```

preprocess dataset
```bash
cd ../../VQA_suite 
mkdir data

export PYTHONPATH=.

python data_prep/vqa_v2.0/extract_vocabulary.py \
--input_files ../orig_data/vqa_v2.0/v2_OpenEnded_mscoco_train2014_questions.json \
 ../orig_data/vqa_v2.0/v2_OpenEnded_mscoco_val2014_questions.json \
 ../orig_data/vqa_v2.0/v2_OpenEnded_mscoco_test2015_questions.json \
--out_dir data/

python data_prep/vqa_v2.0/process_answers.py \
--annotation_file ../orig_data/vqa_v2.0/v2_mscoco_train2014_annotations.json \
--val_annotation_file ../orig_data/vqa_v2.0/v2_mscoco_val2014_annotations.json  \
--out_dir data/ --min_freq 9

python data_prep/vqa_v2.0/extract_word_glove_embedding.py  \
--vocabulary_file data/vocabulary_vqa.txt  \
--glove_file ../orig_data/vqa_v2.0/glove/glove.6B.300d.txt \
--out_dir data/

python data_prep/vqa_v2.0/build_vqa_2.0_imdb.py --data_dir ../orig_data/vqa_v2.0/ --out_dir data/

```

Dowload image features
```bash
cd data/
wget https://s3-us-west-1.amazonaws.com/vqa-suite/data/rcnn_10_100.tar.gz
wget https://s3-us-west-1.amazonaws.com/vqa-suite/data/detectron_23.tar.gz
gunzip rcnn_10_100.tar.gz 
tar -xvf rcnn_10_100.tar
rm -f rcnn_10_100.tar

gunzip detectron_23.tar.gz
tar -xvf detectron_23.tar
rm -f detectron_23.tar


``` 


Training

Example:
```
python train.py 
```

### Test with pretrained models
| Description | performance (test-dev) | Link |
| --- | --- | --- |
| baseline | 68.05 | https://s3-us-west-1.amazonaws.com/vqa-suite/pretrained_models/baseline_models.tar.gz |
| baseline + VG | 68.44 |https://s3-us-west-1.amazonaws.com/vqa-suite/pretrained_models/baseline_VG_models.tar.gz  |
| baseline +VG +VD+morrir | 68.98 |https://s3-us-west-1.amazonaws.com/vqa-suite/pretrained_models/most_data_models.tar.gz |
| dectectron_finetune | 68.49 | https://s3-us-west-1.amazonaws.com/vqa-suite/pretrained_models/detectron23_finetune_models.tar.gz|
| dectectron_finetune+VG |68.77 | https://s3-us-west-1.amazonaws.com/vqa-suite/pretrained_models/detectron23_ft_VG_models.tar.gz |


#### Best Pretrained Model
The best pretrained Model can be download from:
```bash
mkdir pretrained_models/
cd pretrained_models
wget https://s3-us-west-1.amazonaws.com/vqa-suite/pretrained_models/most_data_models.tar.gz
gunzip most_data_models.tar.gz 
tar -xf most_data_models.tar
rm -f most_data_models.tar
```

test the best model with vqa test2015 datasset
```bash

python run_test.py --config pretrained_models/most_data_models/config.yaml \
--model_path pretrained_models/most_data_models/best_model.pth \
--out_prefix test_best_model
```

The result can be found as a json file test_best_model.json , 
this file can be uploaded to test on evalAI(https://evalai.cloudcv.org/web/challenges/challenge-page/80/submission)

### Ensemble different models
download all the models above
```bash
python ensemble.py --res_dirs pretrained_models/ --out ensemble_5.json
```
The result is in ensemble_5.json, check on evalAI, get the overall accuracy is 70.97


