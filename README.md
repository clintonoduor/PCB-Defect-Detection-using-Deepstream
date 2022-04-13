
<div align="center">

# PCB-Defect-Detection-using-Deepstream
<br>
</div>

Manual visual inspection is one of the most complex and expensive tasks for PCB manufacturing companies. Over the years, Printed Circuit Boards have become much smaller and more densely packed with components making manual visual inspection less scalable. With increased demands from the electronics industry, many defects go unnoticed which may lead to poor company reputation ,and reduced business. The figure below shows an image of a PCB automated optical inspection procedure at PCBWay.
<br>
<br>

<div align="center">
<p>
   <a align="left" href="https://www.etechnophiles.com/wp-content/uploads/2021/05/1730328404807.jpg" target="_blank">
   <img width="450" src="https://github.com/clintonoduor/PCB-Defect-Detection-using-Deepstream/blob/main/assets/optical.jpg"></a>
</p>
</div>
<br>
<div align="center">
image source: https://www.etechnophiles.com/pcb-testing-methods-and-properties-7-ways-to-ensure-best-quality-pcbs/
</div>

<br>

 This project uses deeplearning with object detection to automatically detect 6 PCB Defects using YoloV5 and Deepstream SDK running on an Nvidia jetson AGX Xavier. 



The defects include:

* Missing hole
* Mouse bite
* Open circuit
* Short circuit
* Spurious copper
* spur
</div>

### Steps

1. [Model training](https://github.com/clintonoduor/PCB-Defect-Detection-using-Deepstream#model-training)
   
2. [TensorRT engine generation](https://github.com/clintonoduor/PCB-Defect-Detection-using-Deepstream#generating-wts--cfg-files-for-tensorrt-engine)
3. [Configuring Deepstream app](https://github.com/clintonoduor/PCB-Defect-Detection-using-Deepstream#configuring-deepstream)
4. [Optimizing Deepstream app for perfomance](https://github.com/clintonoduor/PCB-Defect-Detection-using-Deepstream#improving-perfomance-tricks)

### Model Training

The model was trained using YoloV5 on Google Colab. The dataset used is hosted on Roboflow to make it easier for others to be able to train and reproduce the model in Colab without the hussle of re-uploading the data. 

YoloV5 🚀 is a family of object detection architectures and models pretrained on the COCO dataset, and represents <a href="https://ultralytics.com">Ultralytics</a> open-source research into future vision AI methods, incorporating lessons learned and best practices evolved over thousands of hours of research and development. 

The choice to use YoloV5 was based on its inherent fast speed, smaller size, and its ability to learn to auto tune anchor boxes based on the distribution of the training set.

 The full code used to train the model for this project can be found by clicking the Colab icon below:

<div align="center">
    <a href="https://colab.research.google.com/drive/14ETRA3gC7nVnPUXXj7qjund3eFIQMKvv">
        <img src="https://github.com/ultralytics/yolov5/releases/download/v1.0/logo-colab-small.png" width="15%"/>
    </a>
 </div>
 <br>

 The dataset used was sourced from the <a href="https://robotics.pkusz.edu.cn/resources/datasetENG">Open Lab on Human Robot Interaction</a> of Peking University.

#### Training Results

Metrics were closely monitored and tracked using Weights & Biases for 1083 epochs as shown in the images below:

<br>

<div align="center">
<p>
 
   <img width="450" src="https://github.com/clintonoduor/PCB-Defect-Detection-using-Deepstream/blob/main/assets/map.png"></a>
</p>
</div>

<br>

<div align="center">
<p>
 
   <img width="450" src="https://github.com/clintonoduor/PCB-Defect-Detection-using-Deepstream/blob/main/assets/precision.png"></a>
</p>
</div>
<br>



<div align="center">
<p>
 
   <img width="450" src="https://github.com/clintonoduor/PCB-Defect-Detection-using-Deepstream/blob/main/assets/recall.png"></a>
</p>
</div>
   
### Generating wts & cfg files for TensorRT engine

[NVIDIA® TensorRT™](https://developer.nvidia.com/tensorrt), is a machine learning framework and SDK for high-performance deep learning inference for GPU accelerated workloads such as intelligent video analytics on jetson devices. It includes a deep learning inference optimizer and runtime that delivers low latency and high throughput for inference.

 Since YoloV5 is based on Pytorch, we will first need to convert our trained model to a format that can be easily converted to TensorRT engine when starting the deepstream application on the Jetson Xavier. To do this,we first put the gen_wts_yoloV5.py file to the YoloV5 folder and run the following script:

```
!python3 gen_wts_yoloV5.py -w /content/yolov5/runs/train/yolov5s_results/weights/best.pt -c /content/yolov5/models/custom_yolov5s.yaml
```
```
# -w is the path to our best.pt model while -c is the file to our input yaml file.
```
The script will output a wts and cfg file that we will put in our Deepstream-Yolo folder to build the TensorRT engine. The conversion script was cloned from [this repository](https://github.com/marcoslucianops/DeepStream-Yolo).

### Configuring Deepstream
The [NVIDIA DeepStream SDK](https://developer.nvidia.com/deepstream-sdk) delivers a complete streaming analytics toolkit for AI based video and image understanding and multi-sensor processing. DeepStream SDK features hardware-accelerated building blocks, called plugins that bring deep neural networks and other complex processing tasks into a stream processing pipeline.
##### Deepstream Installation
First install deepstream 6.0 using these [instructions](https://docs.nvidia.com/metropolis/deepstream/dev-guide/text/DS_Quickstart.html).
##### Editing Deepstream Config Files
```
In the Config_infer_primary_yoloV5.txt, we replace the model-file and custom-network-config with the paths to our generated weight and config files respectively. We also change the num-detected-classes with the number of our classes which is 6 as shown below.

[property]
gpu-id=0
net-scale-factor=0.0039215697906911373
model-color-format=0
custom-network-config=best.cfg
model-file=best.wts
model-engine-file=model_b1_gpu0_fp32.engine
#int8-calib-file=calib.table
labelfile-path=labels.txt
batch-size=1
network-mode=0
num-detected-classes=6
interval=0
gie-unique-id=1
process-mode=1
network-type=0
cluster-mode=4
maintain-aspect-ratio=1
parse-bbox-func-name=NvDsInferParseYolo
custom-lib-path=nvdsinfer_custom_impl_Yolo/libnvdsinfer_custom_impl_Yolo.so
engine-create-func-name=NvDsInferYoloCudaEngineGet

[class-attrs-all]
pre-cluster-threshold=0.5
```
##### Running the application

First compile the application using:

```
CUDA_VER=11.4 make -C nvdsinfer_custom_impl_Yolo
```
Then boost your Jetson clocks using the following commands:
```
$ sudo nvpmodel -m 0
$ sudo jetson_clocks
```

###### Running application using a sample video (.MP4)

1. Go to **deepstream_app_config.txt > [Source 0]** then edit the URI with the path of your sample MP4 video as shown below: 
   

```
[source0]
enable=1
type=3
uri=file:///opt/nvidia/deepstream/deepstream/samples/streams/pcb2.mp4
num-sources=1
gpu-id=0
cudadec-memtype=0
```

2.  Run the command below on your terminal

```
deepstream-app -c deepstream_app_config.txt
```

**NB:** This dataset used for training this model was synthetic hence the trained model cannot be validated using real world data. However the instructions below could be used to run the model using a USB cam or CSI camera:

###### Running application using a Webcam

To run the application using a USB cam, run the below command on your terminal:
```
deepstream-app -c deepstream_app_config_USB.txt
```

###### Running application using a CSI cam

To run the application using a CSI cam, run the command below on your terminal:
```
deepstream-app -c deepstream_app_config(CSI).txt
```

The application outputs a tiled display with on screen bounding boxes of the detected defects as shown in the image below:

<div align="center">

[![Video demo on youtube](https://github.com/clintonoduor/PCB-Defect-Detection-using-Deepstream/blob/main/assets/pcbscreenshot.png?raw=true)](https://www.youtube.com/watch?v=op_TjAQFLfs)

<br>
</div>
The model runs at an average of 51 FPS on a Jetson AGX Xavier, hence one xavier could be configured to handle approximately 10 streams running in parallel at 5 FPS. 

### Optimizing Deepstream app for Perfomance 

There are several techniques that could be used to increase the perfomance of the deepstream app:

##### 1.  Increasing the number of interval in the infer config file

```
network-mode=0
num-detected-classes=6
interval=1
gie-unique-id=1
process-mode=1
network-type=0
```
##### 2.  Disable tiles & on screen perfomance when not needed

To achieve this, you just need to go to the deepstream-app_config.txtfile and set enable = 0  for [tiled-display] [osd] as shown below:

```
[tiled-display]
enable=0
rows=1
columns=1
width=1280
height=720
gpu-id=0
nvbuf-memory-type=0
```
```
[osd]
enable=0
gpu-id=0
border-width=1
text-size=15
text-color=1;1;1;1;
text-bg-color=0.3;0.3;0.3;1
font=Serif
show-clock=0
clock-x-offset=800
clock-y-offset=820
clock-text-size=12
clock-color=1;0;0;0
nvbuf-memory-type=0
```
##### 3. Reducing model precision

Change network precision to fp16 by changing network-mode=0 to network-mode=2 in the  deepstream config infer file as shown below:

```
model-engine-file=model_b1_gpu0_fp32.engine
#int8-calib-file=calib.table
labelfile-path=labels.txt
batch-size=1
network-mode=2
num-detected-classes=6
```

##### 4. Always make sure the stream mux output width and height to the size of the input stream resolution

```
[streammux]
##Boolean property to inform muxer that sources are live
live-source=1
batch-size=1
##time out in usec, to wait after the first buffer is available
##to push the batch even if the complete batch is not formed
batched-push-timeout=40000
## Set muxer output width and height
width=1280
height=720
```
<div align="center">

### Acknowledgement

</div>

1. Open Lab on Human Robot Interaction of Peking University for use of dataset.
2. Passing custom YoloV5 models for deepstream applications: https://github.com/marcoslucianops/DeepStream-Yolo


<div align="center">

### Future works

</div>

1. Implement the project using deepstream python bindings
2. Performing analytics using deepstream analytics
3. Train using real PCB images with defects

