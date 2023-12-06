# KEPP
[CVPR 2024] KEPP: Why Not Use Your Textbook? Knowledge-Enhanced Procedure Planning of Instructional Videos

This repository gives the official implementation of KEPP:Why Not Use Your Textbook? Knowledge-Enhanced Procedure Planning of Instructional Videos (CVPR 2024)

In our project, we explore the capability of an agent to construct a logical sequence of action steps, thereby assembling a strategic procedural plan. This plan is crucial for navigating from an initial visual observation to a target visual outcome, as depicted in real-life instructional videos. Existing works have attained partial success by extensively leveraging various sources of information available in the datasets, such as heavy intermediate visual observations, procedural names, or natural language step-by-step instructions, for features or supervision signals. However, the task remains formidable due to the implicit causal constraints in the sequencing of steps and the variability inherent in multiple feasible plans. To tackle these intricacies that previous efforts have overlooked, we propose to enhance the agent's capabilities by infusing it with procedural knowledge. This knowledge, sourced from training procedure plans and structured as a directed weighted graph, equips the agent to better navigate the complexities of step sequencing and its potential variations. We coin our approach KEPP, a novel Knowledge-Enhanced Procedure Planning system, which harnesses a probabilistic procedural knowledge graph extracted from training data, effectively acting as a comprehensive textbook for the training domain. Experimental evaluations across three widely-used datasets under settings of varying complexity reveal that KEPP attains superior, state-of-the-art results while requiring only minimal supervision. The main architexture of our model is as follows.

![model](https://github.com/Ravindu-Yasas-Nagasinghe/KEPP/assets/56619402/6d50502b-1911-482f-8173-66171cf01d20)

## Setup
In a conda env with cuda available, run:
```shell
pip install -r requirements.txt
```
## Data Preparation
### CrossTask
1. Download datasets&features
```shell
cd {root}/dataset/crosstask
bash download.sh
```
2. move your datasplit files and action one-hot coding file to `{root}/dataset/crosstask/crosstask_release/`
```shell
mv *.json crosstask_release
mv actions_one_hot.npy crosstask_release
```
### COIN
1. Download datasets&features
```shell
cd {root}/dataset/coin
bash download.sh
```
### NIV
1. Download datasets&features
```shell
cd {root}/dataset/NIV
bash download.sh
```
## Train Step model
1. First generate the training and testing dataset json files. You can modify the dataset, train steps, horizon(prediction length), json files savepath etc. in `args.py`
```shell
cd {root}/step
python loading_data.py 
```
Dimensions for different datasets are listed below:

| Dataset	| observation_dim |	action_dim	| class_dim |
|----| ----| ----| ----| 
| CrossTask	| 1536(how) 9600(base) | 105	| 18 | 
| COIN 	| 1536	| 778	| 180 | 
| NIV	| 1536	| 48	| 5 | 

2. Train the step model
```shell
python main_distributed.py --multiprocessing-distributed --num_thread_reader=8 --cudnn_benchmark=1 --pin_memory --checkpoint_dir=whl --resume --batch_size=256 --batch_size_val=256 --evaluate
```
3. Generate first and last action predictions for train and test dataset.
## Generate paths from procedure knowlege graph
## Train plan model
1. Modify the `json_path_train` and `json_path_val` arguments of `args.py` in plan model as the outputs generated from procedure knowlwdge graph for train and test data respectively.
```shell
cd {root}/plan
python main_distributed.py --multiprocessing-distributed --num_thread_reader=8 --cudnn_benchmark=1 --pin_memory --checkpoint_dir=whl --resume --batch_size=256 --batch_size_val=256 --evaluate
```
## Inference

For Metrics
​Modify the max checkpoint path(L394) as the evaluated model in inference.py and run:
```shell
python inference.py --multiprocessing-distributed --num_thread_reader=8 --cudnn_benchmark=1 --pin_memory --checkpoint_dir=whl --resume --batch_size=256 --batch_size_val=256 --evaluate > output.txt
```
Results of given checkpoints:

| dataset | SR | mAcc |	MIoU |
| ---- | -- | -- | -- |
| Crosstask_T=4 |	21.02	| 56.08	| 64.15 |
| COIN_T=4 | 15.63 | 39.53 |	53.27 |
| NIV_T=4 |	22.71 |	41.59 |	91.49 |
