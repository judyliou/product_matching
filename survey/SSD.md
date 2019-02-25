## Single Shot Detection (SSD)

##### Main Concept: 特徵分層提取
>_Since YOLO only does bounding box regression once, (1) the performance of **small objects** is weak; (2) it struggle to gerneralize to **new ratios and configurations**._
_Therefore, SSD combine advantages of Fast R-CNN (anchors) and YOLO (one-stage) and extracts features on multiple layers._


### 1. Network Architecture

![ssd_structure](https://upload.cc/i1/2019/02/25/iK91cM.png)
![VGG_structure](https://upload.cc/i1/2019/02/25/3e5XEa.png)
  
Use VGG16 network as a base and convert fc6 and fc7 to conv. layers and add 4 conv. layers with different sizes in the end.
<br/>

### 2. Default Boxes
![default box](https://upload.cc/i1/2019/02/25/InWMyb.jpg)
- After going through a certain of convolutions for feature extraction, we obtain **a feature layer of size m×n (number of locations) with p channels**, such as 8×8 or 4×4 above. And a 3×3 conv is applied on this m×n×p feature layer.
- **For each location, we got k bounding boxes.** These k bounding boxes have different sizes and aspect ratios. The concept is, maybe a vertical rectangle is more fit for human, and a horizontal rectangle is more fit for car.
- **For each of the bounding box, we will compute c class scores and 4 offsets relative to the original default bounding box shape.**
- Thus, we got **(c+4)kmn outputs.**
<br/>

### 3. Multi-scale feature maps
![box](https://upload.cc/i1/2019/02/25/3r4L1j.png)  
- The convolutional layers in the end decrease in size progressivley and allow predictions of detections at multiple scales.
- **Low-level features** contain more details (dots, lines, etc.), which are better for learning small size objects.
- **High-level features** are build on top of low-level features to detect objects with larger shapes.
<br/>

### 4. Loss Function
![loss function](https://upload.cc/i1/2019/02/25/mAnQvs.png)
- Parameters 
  *  _x:_ $x_{ij}^p = \lbrace 1,0 \rbrace$ indicates whether _i_-th default box match to _j_-th ground truth box of category _p_.
  *  _c:_ confidences of multiple classes
  *  _l:_ predicted box
  *  _g:_ ground truth box
  *  _alpha_ is set to 1
- The loss function consists of two terms: $L_{conf}$ and $L_{loc}$ where $N$ is the matched default boxes. 
  *  $L_{conf}$ (Confidence Loss): Softmax loss over multiple classes confidences
  *  $L_{loc}$ (Localization Loss): Smooth L1 loss of the regression



### 5. Training Details
- Matching Strategy
- Scales and Aspect Ratios of Default Boxes
- Hard Negative Mining
- Data Augmentation
#### 5.1 Matching Strategy
- Match **ground truth boxes** and **default boxes**
- _Step 1:_ Matching each ground truth box to the default box with the best jaccard overlap (IOU).
- _Step 2:_ Match default boxes to any ground truth with jaccard overlap higher than a threshold (0.5).
<br/>
#### 5.2 Scales and Aspect Ratios of Default Boxes
##### Scale
![scale](https://upload.cc/i1/2019/02/25/0ARhpN.png)
- $S_k:$ scale of the boxes for the _k_-th feature map
- $S_{min} = 0.2\ (scale\ of\ the\ lowest\ layer),S_{max} = 0.9\ (scale\ of\ the\ highest\ layer)$

##### Ratio
- For each scale we have 5 non-square aspect ratios:
<div align=left>  
<img src="https://upload.cc/i1/2019/02/25/pgaZnK.png">
</div>  
<br/>

- And one ratio of 1:1:
<div align=left>  
<img src="https://upload.cc/i1/2019/02/25/ok4IT9.png">
</div>
<br/>

- Therefore, we have **six bounding boxes** in total.

#### 5.3 Hard Negative Mining 
Instead of using all the negative examples, we sort them using the highest confidence loss for each default box and pick the top ones so that the **ratio between the negatives and positives is at most 3:1**.

This can lead to **faster optimization** and a **more stable training**.
<br/>

#### 5.4 Data Augmentation
Training images are randomly sampled by one of the following options with **sampled patch is [0.1, 1] of the original image size**:
- Entire original input image
- Sample a patch so that the minimum jaccard overlap with the objects is 0.1, 0.3, 0.5, 0.7, or 0.9
- Randomly sample a patch

#### 5.5 Atrous Convolution (Hole Algotirhm / Dilated Convolution)     
FC6 and FC7 use Atrous Convolution to **increase the receptive field while keeping the number of parameters same.**     
<div align=center>  
<img src="https://upload.cc/i1/2019/02/25/sfcxhQ.gif" height="200">
</div>  

<br/>



### Reference:
- [Review: SSD — Single Shot Detector (Object Detection)](https://towardsdatascience.com/review-ssd-single-shot-detector-object-detection-851a94607d11)
- [[目标检测]SSD：Single Shot MultiBox Detector - faiculty - CSDN博客](https://blog.csdn.net/neu_chenguangq/article/details/79057655)
- [SSD: Single Shot MultiBox Detector 介紹 – 王得懿 – Medium](https://medium.com/@bigwaterking01/ssd-single-shot-multibox-detector-%E4%BB%8B%E7%B4%B9-1fe95073c1a3)

