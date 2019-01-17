
Using ConvNetQuake to train model based on wenchuan aftershocks and automatically detect earthquakes in continuous waveform
============= 

We use ConvNetQuake to train our own model for events classification based on the wenchuan aftershocks and use it to classify earthquakes from one day continous waveform(2008-07-25) of one station (MXI).In this day there are about 43 "obvious" earthquakes and more than 170 "trival" earthquakes (see the image below).We use ConvNetQuake and STA/LTA to identify them automatically,both two methods can detect all the "obvious" earthquakes,but CNN win STA/lTA in detecting "trival" earthquakes,besides,CNN has less false recognition than STA/LTA.

For  ConvNetQuake refer to:
Perol., T, M. Gharbi and M. Denolle. Convolutional Neural Network for Earthquake detection and location. [preprint arXiv:1702.02073](https://arxiv.org/abs/1702.02073), 2017.

赵明,陈石,Dave.A.Yuen,2019,基于深度学习卷积神经网络的地震波形自动分类与识别,地球物理学报，62(1),待刊

Zhao M,Chen S,Dave Yuen. 2019. Waveform classification and seismic recognition by convolution neural network. Chinese J. Geophys. (in Chinese),62(1),in press.

A video tutorial can be found at:

[How to detect earthquake automatically using CNN](https://tianchi.aliyun.com/forum/videoStream.html?spm=5176.11409386.0.0.53a1311fQtoscI&postsId=4929#postsId=4929)

The continious wavaform of MXI,2008-07-25
![The data of MXI,2008-07-25,and marked events](./XX.MXI_dayplot_origin_data.png)

The hand picked events:
![The hand picked events](./XX.MXI_dayplot_hand_picks.png)

All CNN identified slices with prob>0.1,abosolute ampitude
![All CNN identified slices with prob>0.1 merged and plot,abosolute ampitude](./XX.MXI_dayplot_cnn.png)

We also use classic STA/LTA Algorithm  for a comparison:
![STA/LTA Algorithm](./XX.MXI_dayplot_obspy.png)

All CNN identified slices with prob>0.1,normalized ampitude
![All CNN identified slices with prob>0.1 merged and plot,normalized ampitude](./XX.MXI_dayplot_norm_cnn.png)

## Installation
* Download repository
* Install dependencies: `pip install -r requirements.txt.b`
* if it doesn't work,try: `pip install -r requirements.txt`
* Add directory to python path: `./setpath.sh`
* export PYTHONPATH=~/ConvNetQuake
## Data of the continous waveform

Download the [data](https://pan.baidu.com/s/1N_gwRC95qwQHnfNX94cdgQ) (roughly 110 Mb) and symlink to data ln -s data Downloads/data
The continuous waveform data is in ./data

The `data` directory contains:
* `XX.MXI.2008207000000.mseed`: the continious waveform data 
* `dayplot.py`: a script to plot the continious waveform
* `merge_dayplot.py`: a merge script
* `XX.MXI_dayplot_[0-64800].png`: marked earthquakes of the day 
* `XX.MXI_dayplot_hand_picks.png`:hand picked events


## Train data

We provide a catalog ("MXI_catalog_for_train.txt") for train dataset,you can send a request to [CENC](http://news.ceic.ac.cn/index.html?time=1523511012) or [dmc](http://www.seisdmc.ac.cn/) and download the related waveform data.
We also provided a train dataset with about 2000 events samples,please download from  [mz](https://pan.baidu.com/s/11JlOYsqJR82Wk3N93klfcg)

## build your own dataset 

For events:

 ./bin/preprocess/create_dataset_events.py --stream_dir stream/ --catalog MXI_catalog_for_dataset.csv --output_dir wenchuan_train_test/positive --save_mseed True --plot True
 
For noises:

 ./bin/preprocess/create_dataset_noise.py --stream stream/XX.MXI.mseed --catalog  MXI_catalog_for_noise.csv --output_dir wenchuan_train_test/negative --max_windows=30000
 
 However,the stream data is quite large(1.6GB),and my baidu disk is full,so it is provided upon request.We strongly recommend you use your own data stream and catalog (just follow the format like MXI_catalog_for_dataset.csv) to build datasets.

## Train

./bin/train --dataset wenchuan_train_test --checkpoint_dir output/convnetquake --n_clusters 2


## Tensorboard for real-time monitor

tensorboard --logdir output/convnetquake/ConvNetQuake

## Trained model
We also privided a model which was trained on over 20000 earthquakes slices (30s) and over 60000 slices of noises (30s) 
The directory `trained_model` contains:
* `convnetquake`: trained model 

## Detecting events in continuous waveform data

### From mseed

./bin/predict_from_stream.py --stream_path data --checkpoint_dir trained_model/ConvNetQuake  --n_clusters 2 --window_size 30 --window_step 31 --output predict_MXI_one_day --plot --save_sac


It will generate a dir "predict_MXI_one_day",which contains:
 
* `viz`: the image of events,the name of the image contain its probility(prob) and its starttime,like "MXI_0.50053_2008-07-25T03_07_18.000000Z.png"
* `viz_not`: the image of noise,notice the  higher prob,the more likely it is an events,actually when the prob>0.1 there is a large chance it is an event. 
* `sac`: the slice data of viz

## Using overlapping windows
It proved using overlapping windows will get better results,however,the events will be identified more than once,you can choose the one with highest probility.To run,just uncomment "

#lists = np.arange(0,30,5)

" in predict_from_stream.py

## the classic sta/lta Algorithm for compare
./trigger_by_sta_lta.py --stream_path data --output out_STA_LTA_MXI  --save_mseed --window_step 20
## A hand-picked catalog for benchmark
We provide a hand-picked catalog(`MXI_20080725_hand_pick_PS.txt`) for users to compare with their own results.A qulified trained model should be able to find out all the earthquakes in this catalog.


