[![Everything Is AWESOME](https://github.com/clintonoduor/PCB-Defect-Detection-using-Deepstream/blob/main/pcbscreenshot.png?raw=true)](https://www.youtube.com/watch?v=op_TjAQFLfs)


# PCB-Defect-Detection-using-Deepstream
Manual visual inspection is one of the most complex and expensive tasks in PCB manufacturing companies. Over the years, Printed Circuit Boards have become much smaller and more densely packed with components making manual visual inspection less scalable. With increased demands from the electronics industry, many defects go unnoticed that may lead to poor company reputation ,and reduced number of contracts.

This project aims to use computer vision specifically object detection to automatically detect 6 PCB Defects using YoloV5 and Deepstream SDK.



The defects include:
1.  missing hole 
2.  mouse bite
3.  open circuit
4.  short    
5.  spurious copper
6.  


The dataset used in this project has been sourced from the [Open Lab on Human Robot Interaction](https://robotics.pkusz.edu.cn/resources/datasetENG/) of Peking University.

### Model Training

The model was trained using YoloV5 on Google Collab, and dataset hosted in Roboflow to make it easier for others to be able to train and reproduce the model in Colab without the hussle of re-uploading the data each time.YOLOv5 🚀 is a family of object detection architectures and models pretrained on the COCO dataset, and represents <a href="https://ultralytics.com">Ultralytics</a> open-source research into future vision AI methods, incorporating lessons learned and best practices evolved over thousands of hours of research and development.
<br>
Click on the Colab link below to reproduce the training script and model on your colab
<div align="center">
    <a href="https://colab.research.google.com/drive/14ETRA3gC7nVnPUXXj7qjund3eFIQMKvv">
        <img src="https://github.com/ultralytics/yolov5/releases/download/v1.0/logo-colab-small.png" width="15%"/>
    </a>
 </div>

   
### Generating wts & cfg files for TensorRT engine

Since YoloV5 is based on pytorch, we will first need to convert the outputs of our trained model to a format that can easily be converted to TensorRT engine on the Jetson. <br>
To do this, we will first put the gen_wts_yoloV5.py file to the YoloV5 folder and run the following script:

```
!python3 gen_wts_yoloV5.py -w /content/yolov5/runs/train/yolov5s_results/weights/best.pt -c /content/yolov5/models/custom_yolov5s.yaml

```

```
# -w is the path to our best.pt model while -c is the file to our input yaml file.
```
The script will output a wts and cfg file that will put in our Deepstream-Yolo folder to build the TensorRT engine

#### 5. Copy generated cfg and wts files to DeepStream-Yolo folder
### Installing Deepstream 6.0 on Jetson Xavier
### Deepstream Config files
### Running the application
### Improving Perfomance

