<div align="center">

[![Everything Is AWESOME](https://github.com/clintonoduor/PCB-Defect-Detection-using-Deepstream/blob/main/pcbscreenshot.png?raw=true)](https://www.youtube.com/watch?v=op_TjAQFLfs)
 

<br>


# PCB-Defect-Detection-using-Deepstream

</div>
<br>

Manual visual inspection is one of the most complex and expensive tasks for PCB manufacturing companies. Over the years, Printed Circuit Boards have become much smaller and more densely packed with components making manual visual inspection less scalable. With increased demands from the electronics industry, many defects go unnoticed which may lead to poor company reputation ,and reduced business.
<br>

<div align="center">
<p>
   <a align="left" href="https://www.etechnophiles.com/wp-content/uploads/2021/05/1730328404807.jpg" target="_blank">
   <img width="450" src="https://github.com/clintonoduor/PCB-Defect-Detection-using-Deepstream/blob/main/optical.jpg"></a>
</p>
</div>
<br>


 This project uses deeplearning with object detection to automatically detect 6 PCB Defects using YoloV5 and Deepstream SDK running on an Nvidia jetson AGX Xavier. 



The defects include:

<br>
<div align="center">
 missing hole 
 </div>
 <div align="center">
 mouse bite
 </div>
 <div align="center">
 open circuit
 </div>
 <div align="center">
 short circuit
 </div>
 <div align="center">
 spurious copper  

</div>
<br>


### Model Training

The model was trained using YoloV5 on Google Collab. The dataset used is hosted in Roboflow to make it easier for others to be able to train and reproduce the model in Colab without the hussle of re-uploading the data. 

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

   
### Generating wts & cfg files for TensorRT engine

Since YoloV5 is based on pytorch, we will first need to convert the outputs of our trained model to a format that can easily be converted to TensorRT engine on the Jetson. To do this,we first put the gen_wts_yoloV5.py file to the YoloV5 folder and run the following script:

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

###### Running application using a sample video (.MP4)

1. Go to deepstream_app_config.txt > Source 0 then edit the URI with the path of your sample MP4 video as shown below: 
   

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

**NB:** This dataset used for training this model was synthetic hence the tarined model cannot be validated using real world data. However the instructions below could be used to run the model using a USB cam or CSI camera:

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
### Improving Perfomance

