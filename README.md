[![Everything Is AWESOME](https://github.com/clintonoduor/PCB-Defect-Detection-using-Deepstream/blob/main/pcbscreenshot.png?raw=true)](https://www.youtube.com/watch?v=op_TjAQFLfs)


# PCB-Defect-Detection-using-Deepstream
Manual visual inspection is one of the most complex and expensive tasks in PCB manufacturing companies. Over the years, Printed Circuit Boards have become much smaller and more densely packed with components making manual visual inspection less scalable. With increased demands from the electronics industry, many defects go unnoticed that may lead to poor company reputation ,and reduced number of contracts.

This project aims to use computer vision specifically object detection to automatically detect 6 PCB Defects using YoloV5 and Deepstream SDK.



The defects include:
-  missing_hole  
-  mouse_bite
-  open_circuit
-  short    
-  spurious_copper

The dataset used in this project has been sourced from the [Open Lab on Human Robot Interaction](https://robotics.pkusz.edu.cn/resources/datasetENG/) of Peking University.

### Model Training

The model was trained using YoloV5 on Gogle Collab, and dataset hosted in Roboflow to make it easier for others to be able to retrain the model in Colab without the hussle of reuploading the dataset. YOLOv5 🚀 is a family of object detection architectures and models pretrained on the COCO dataset, and represents <a href="https://ultralytics.com">Ultralytics</a> open-source research into future vision AI methods, incorporating lessons learned and best practices evolved over thousands of hours of research and development.

<br>
<br>
   <a href="https://colab.research.google.com/drive/14ETRA3gC7nVnPUXXj7qjund3eFIQMKvv"><img src="https://colab.research.google.com/assets/colab-badge.svg" alt="Open In Colab"></a>
<br>
<br>

The dataset for this project is pulled from Roboflow to make it easier for others to train the model without reuploading the dataset each time they want to train.
   
### Generating wts & cfg files for TensorRT engine
### Installing Deepstream 6.0 on Jetson Xavier
### Deepstream Config files
### Running the application
### Improving Perfomance

