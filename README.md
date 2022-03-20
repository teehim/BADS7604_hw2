# Real/Toy Dog Classification (POWER DS TEAM)

## Highlights
- การใช้ pretrained model ที่สามารถทำนาย 1 ใน class ที่ต้องการได้ดีอยู่แล้ว มาเป็น feature extractor นั้นทำให้ได้ผลที่ดี
- การทำให้การทำนายสุนัขของเล่นนั้นมีประสิทธิภาพมากขึ้น มีผลให้ประสิทธิภาพการทำนายสุนัขจริงๆลดลงเล็กน้อย
- การใช้ EfficientNet ที่มี Top-1 Accuracy ของ Imagenet สูงสุดจาก model ที่ทำการทดลองทั้งหมด มาเป็น feature extractor ทำให้ได้ Accuracy ที่ดีที่สุดบน Test Data ของเราด้วย

## Introduction

การ classify สุนัขของเล่นนั้น เป็นสิ่งที่ pretrained model ที่ทำการ train ด้วย Imagenet dataset นั้นยังทำได้ไม่ดีนัก
เนื่องจากใน 1000 classes ของ imagenet มีเพียง 5 class เท่านั้นที่จัดเป็นสุนัขของเล่น ('miniature_poodle','toy_terrier','toy_poodle','miniature_schnauzer','miniature_pinscher')
และ 5 classes นั้น ยังเป็นการแยกสุนัขของเล่นตามสายพันธุ์อีกด้วย ทำให้ไม่ครอบคลุมสุนัขของเล่นทั้งหมด ทางกลุ่มเราจึงต้องการที่จะสร้าง model ที่สามารถแบ่งแยกระหว่างสุนัขจริง และสุนัขของเล่นได้อย่างมีประสิทธิภาพ
ด้วยการรวมรวบรูปภาพของสุนัขจริง และสุนัขของเล่นขึ้นมาเพื่อเป็น dataset สำหรับใช้ในการ train model สำหรับแบ่งแยกระหว่างสุนัขจริง และสุนัขของเล่น

## Data

ทางกลุ่มเราได้ทำการรวบรวมรูปภาพโดยแบ่งเป็น 2 classes คือ 
1. สุนัขจริง (Real Dog) 
2. สุนัขของเล่น (Toy Dog)

#### Data source

โดยเราได้ทำการรวบรวมรูปภาพจากเว็บไซต์ต่อไปนี้
1. www.flickr.com
2. www.pexels.com
3. www.pinterest.com
4. www.pixabay.com

โดยจำนวนรูปภาพที่เราสามารถรวบรวมได้สำหรับแต่ล่ะ class มีจำนวนทั้งหมด 1148 รูป แบ่งเป็น
- Real Dog: 471 รูป
- Toy Dog: 677 รูป

#### Data pre-processing

รูปภาพทั้งหมดจะถูก preprocess โดยจาก resize ให้อยู่ในขนาด 224 x 224 สำหรับ vgg16, ResNet50 และ 380 x 380 สำหรับ efficientnet-b4 และแปลงให้กลายเป็น array (3-channels)
จากนั้นจึงนำ array ที่ได้ไปผ่าน function preprocess_input ของ keras ตาม network ที่เลือกใช้ (e.g. vgg16)

#### Data Augmentation

เราได้ทำกระบวนการ Data augmentation เพิ่มเติมโดยมีการทำ augment ทั้งหมด 4 แบบดังนี้
1. horizontal flip
2. vertical flip
3. zoom (zoom_range=0.2)
4. rotation (rotation_range=20)

#### Data splitting
    
เราได้ทำการแบ่ง data ออกเป็น 3 ส่วนดังนี้
- train 50%
- validation 20%
- test 30%

## Network architecture

เราเลือกใช้ pretrained model 32 ตัว ได้แก่ VGG16 ,Efficientnet-B4 และ ResNet50 เป็น model ที่ใช้ในการเปรียบเทียบประสิทธิภาพ และใช้สำหรับทำ feature extraction โดยรายละเอียดของ 3 model จะมีดังนี้

#### VGG16

ในส่วนของ VGG16 จะมี Architecture ดังรูปต่อไปนี้

<img src="https://github.com/teehim/BADS7604_hw2/blob/master/images/vgg_arch.JPG?raw=true" style="width:700px;">

#### EfficientNet-B4

EfficientNet จะใช้หลักการ Compound Scaling Model ที่จะทำการ Scale Model ในทุกๆ dimension ไปพร้อมๆกัน (depth, width, input resolution)
โดยที่จะสามารถแบ่ง model ออกเป็น 7 Block ซึ่งในแต่ล่ะ version ของ EfficientNet ส่วนประกอบของแต่ล่ะ Block จะมากน้อยแตกต่างกันไป

โดย EfficientNet-B4 จะมี Architecture ดังรูปต่อไปนี้

<img src="https://github.com/teehim/BADS7604_hw2/blob/master/images/effnet_module.JPG?raw=true" style="width:700px;">

<img src="https://github.com/teehim/BADS7604_hw2/blob/master/images/effnet_sub_block.JPG?raw=true" style="width:700px;">

<img src="https://github.com/teehim/BADS7604_hw2/blob/master/images/effnet.png?raw=true" style="width:700px;">

#### ResNet50

ResNet (Residual Network)เป็นโมเดลที่พัฒนาขึ้นมาเพื่อแก้ปัญหา Vanishing Gradient ของ CNN
โดยใช้ Algorithm ที่เรียกว่า Skip Connections ที่สามารถส่งผ่าน Gradient ข้ามไปยัง Layer ต่างๆที่ต้องการได้
ไม่ต้องส่งผ่านแบบ sequential แบบ CNN ดังนั้นการทำแบบนี้ก็สามารถแก้ปัญหา Vanishing Gradient ได้

โครงสร้างหลักๆ ประกอบด้วย Conv 5 Layers และในขั้นตอนสุดท้ายจะใช้หลักการ Average pooling จากนั้นใช้ Softmax เป็น
Activation function ในการ classify

<img src="https://github.com/teehim/BADS7604_hw2/blob/master/images/resnet.png?raw=true" style="width:700px;">

#### Our Model

ในส่วนของ Model ของเรานั้น เราได้ทำการทดลองทั้งหมด 3 models ซึ่งจะใช้ pretrained model ข้างต้นมาเป็น feature extractor โดยทำการ freeze ทุกๆ layer และทำการตัด layer ที่เป็นส่วนของการ classify ออก
แล้วต่อด้วย Linear Layer ที่เราสร้างขึ้นมา โดยทั้ง 3 models จะมีรูปแบบของ layer ที่เหมือนกันหมดซึ่งประกอบด้วย

- Flatten Layer
- Dense Layer (1024 nodes, relu activation function)
- Dense Layer (2048 nodes, relu activation function)
- Dense Layer (1024 nodes, relu activation function)
- Dropout Layer (0.3 dropout rate)
- Output Layer (2 nodes, softmax activation function)

โดย model ที่ดีที่สุดที่เราเลือกคือ model ที่ใช้ EfficientNet เป็น feature extractor

[Model Summary](https://github.com/teehim/BADS7604_hw2/blob/master/model_summary.txt)

## Training

#### Model #1 (VGG16 as Feature Extractor)

    Trained on 1 NVDIA RTX 3080

    - Optimizer: Adam
    - Learning rate: 0.001
    - Loss Function: BinaryCrossentropy
    - Batch size: 64
    - Epoch: 10

    เวลาที่ใช้ในการ Train 43 วินาที

#### Model #2 (Efficientnet-B4 as Feature Extractor)

    Trained on 1 NVDIA RTX 3080

    - Optimizer: Adam
    - Learning rate: 0.001
    - Loss Function: BinaryCrossentropy
    - Batch size: 32
    - Epoch: 10

    เวลาที่ใช้ในการ Train 160 วินาที

#### Model #3 (ResNet50 as Feature Extractor)

    Trained on 1 NVDIA RTX 3080

    - Optimizer: Adam
    - Learning rate: 0.001
    - Loss Function: BinaryCrossentropy
    - Batch size: 64
    - Epoch: 10

    เวลาที่ใช้ในการ Train 50 วินาที

## Results

#### Train/Validation Data Result

ในส่วนของ Train vs Validation Accuracy ของทั้ง 3 models จะเป็นไปดังกราฟต่อไปนี้้

**1. VGG16**

<img src="https://github.com/teehim/BADS7604_hw2/blob/master/images/train_val_vgg16.png?raw=true" style="width:700px;"/>

    Best epoch accuracy:
        - train: 0.9716
        - validation: 0.9208

**2. EfficientNet-B4**

<img src="https://github.com/teehim/BADS7604_hw2/blob/master/images/train_val_eff.png?raw=true" style="width:700px;"/>

    Best epoch accuracy:
        - train: 0.9858
        - validation: 0.9667

**3. ResNet50**

<img src="https://github.com/teehim/BADS7604_hw2/blob/master/images/train_val_res.png?raw=true" style="width:700px;"/>

    Best epoch accuracy:
        - train: 0.9449
        - validation: 0.9417

ถ้าหากดูจาก Validation Accuracy แล้ว VGG16 และ EfficientNet-B4 ได้ค่า Accuracy ที่มากที่สุดจากทั้ง 3 models

---

#### Test Data Result

ในส่วนของ Test Data เราทำการเปรียบเทียบค่า Accuracy ของทั้ง 3 models และได้ผลลัพธ์ดังนี้

    - VGG16: 0.9304
    - EfficientNet-B4: 0.9623
    - ResNet50: 0.8840

ซึ่ง EfficientNet-B4 ได้ค่า Accuracy ที่มากที่สุด ซึ่งเป็นไปตามที่คาดการณ์ เพราะค่า Top-1 Accuracy บน Imagenet dataset ของ EfficientNet-B4 นั้นมีค่ามากที่สุดจากทั้ง 3 models

นอกจากนี้แล้วทางกลุ่มเราได้ใช้เทคนิค Grad-CAM มาเพื่อตรวจสอบว่า model นั้นทำการทำนายโดยดูจากรูปบริเวณของเล่นหรือไม่
เนื่องจากรูปสุนัขของเล่นส่วนมากที่เราได้ทำการรวบรวมมานั้นมีพื้นหลังที่เป็นสีขาวอย่างเดียว

ซึ่งเมื่อทำการสุ่มรูปสุนัขของเล่นที่มีพื้นหลังสีขาวขึ้นมาดูแล้ว พบว่า model นั้นทำนายโดยดูจากรูปบริเวณของเล่นเป็นส่วนใหญ่

<img src="https://github.com/teehim/BADS7604_hw2/blob/master/images/gradCAM1.png?raw=true" style="width:500px;"/>

<img src="https://github.com/teehim/BADS7604_hw2/blob/master/images/gradCAM2.png?raw=true" style="width:500px;"/>

<img src="https://github.com/teehim/BADS7604_hw2/blob/master/images/gradCAM3.png?raw=true" style="width:500px;"/>

---

#### Pretrained Result Comparison

ในส่วนของการเปรียบเทียบประสิทธิภาพระหว่าง Pretrained model กับ model ที่เราทำการสร้างขึ้นมา
ทางกลุ่มเราทำการเปรียบเทียบโดยดูจากค่า Accuracy ที่ได้จากบน Test Data โดยเราทำการดู Accuracy แยกเป็นต่อ class แทน เนื่องจากจะทำให้เห็นความแตกต่างของ Accuracy ของแต่ล่ะ class ว่ามีประสิทธิภาพที่ดี หรือแย่กว่าได้ชัดเจน
โดยเราทำการเปรียบเทียบโดยใช้ EfficientNet-B4 เนื่องจากเป็น model ที่ได้ค่า Test Accuracy มากที่สุด

---

**1. Real Dog Accuracy**

ในส่วนของ Real Dog เราจะทำการดู Accuracy ของ Test Data เฉพาะรูปที่เป็นสุนัขจริงๆเท่านั้น 
และเนื่องจาก Pretrained model นั้นมี class ที่เป็นไปได้ทั้งหมด 1000 class เราจึงทำการวัด Accuracy ด้วยการดูว่า class ที่ pretrained model ทำนายออกมานั้น เป็น class ที่จัดเป็นสุนัข หรือสัตว์สายพันธุ์ใกล้เคียงกัน (เช่น หมาป่า หรือสุนัขจิ้งจอก) หรือไม่ โดย class ทั้งหมดที่จัดว่าอยู่ในหมวดหมู่นี้มีทั้งหมด 206 classes

โดยค่า Accuracy จะได้ดังต่อไปนี้

    - Pretrained EfficientNet-B4: 0.9932
    - Model #2 (Efficientnet-B4 as Feature Extractor): 0.9731

จะสังเกตว่า Accuracy ของ Model ของเรานั้นได้ค่าที่ต่ำกว่า pretrained model

---

**2. Toy Dog Accuracy**

ในส่วนของ Toy Dog เราจะทำการดู Accuracy ของ Test Data เฉพาะรูปที่เป็นสุนัขของเล่นเท่านั้น 
โดย class ของ pretrained model ที่เราจัดว่าอยู่ในหมวดหมู่ของสุนัขของเล่นนั้นมีทั้งหมด 5 classes ('miniature_poodle','toy_terrier','toy_poodle','miniature_schnauzer','miniature_pinscher')

โดยค่า Accuracy จะได้ดังต่อไปนี้

    - Pretrained EfficientNet-B4: 0.3571
    - Model #2 (Efficientnet-B4 as Feature Extractor): 0.9540

จะสังเกตว่า Accuracy ของ Model ของเรานั้นได้ค่าที่สูงกว่า pretrained model มาก

---

## Discussion

เมื่อสังเกตจากการเปรียบเทียบ Test Accuracy ของแต่ล่ะ class ระหว่าง pretrained model และ model ของเราแล้วนั้น
จะเห็นว่าความสามารถในการทำนายว่ารูปนั้นเป็นสุนัขของเล่นหรือไม่ของ model ของเรานั้นทำได้ดีกว่า pretained model อย่างมาก แต่ก็ต้องแลกกับ Accuracy ของการทำนายสุนัขจริงๆที่ลดลงเล็กน้อย
ซึ่งเป็นไปตามสมมติฐานเนื่องจากรูปของสุนัขจริงๆ และสุนัขของเล่นนั้นมีความคล้ายกันมาก การที่ทำให้ model นั้นทำนายสุนัขของเล่นได้ดี จึงทำให้การทำนายสุนัขจริงๆนั้นแย่ลง เพราะโอกาสที่ model จะทำนายว่าสุนัขจริงๆเป็นสุนัขของเล่นนั้นมีมากขึ้น

## Conclusion

การทำ Transfer Learning โดยใช้ pretrained model ที่ train บน imagenet dataset แล้วมาใช้เป็น feature extractor ทำให้ได้ model ที่มีประสิทธิภาพที่ดี และไม่จำเป็นจะต้องทำการ train ในส่วนของ Convolutional Layer เลย ทำให้ทรัพยากรที่ใช้ในการ train นั้นลดลงมาก

## References
- www.flickr.com
- www.pexels.com
- www.pinterest.com
- pixabay.com
- https://towardsdatascience.com/complete-architectural-details-of-all-efficientnet-models-5fd5b736142
- https://medium.com/@mygreatlearning/what-is-vgg16-introduction-to-vgg16-f2d63849f615

```
    @article{DBLP:journals/corr/HeZRS15,
        author    = {Kaiming He and
                    Xiangyu Zhang and
                    Shaoqing Ren and
                    Jian Sun},
        title     = {Deep Residual Learning for Image Recognition},
        journal   = {CoRR},
        volume    = {abs/1512.03385},
        year      = {2015},
        url       = {http://arxiv.org/abs/1512.03385},
        eprinttype = {arXiv},
        eprint    = {1512.03385},
        timestamp = {Wed, 17 Apr 2019 17:23:45 +0200},
        biburl    = {https://dblp.org/rec/journals/corr/HeZRS15.bib},
        bibsource = {dblp computer science bibliography, https://dblp.org}
    }
```

## Members
- (20%) 6220422048 กชกร เรืองศรี (รวบรวมรูปภาพ + ทดลอง InceptionV3)
- (0%) 6220422061 ไตรเทพ จันทร์เทพ (ติดต่อไม่ได้)
- (20%) 6220422065 สุธาสินี โพธิ์แจ่ม (รวบรวมรูปภาพ + ทดลอง NasNet)
- (20%) 6310422028 วรเมธ ปลอดโปร่ง (รวบรวมรูปภาพ + ทดลอง VGG16)
- (20%) 6310422031 ธนัตถ์กรณ์ ชื่นบรรลือสุข (ทดลอง EfficientNet + สรุปผล)
- (20%) 6310422046 วีระศักดิ์ การุณย์ (รวบรวมรูปภาพ + ทดลอง ResNet50)

#### งานชึ้นนี้เป็นส่วนหนึ่งของวิชา BADS7604 การเรียนรู้เชิงลึก (Deep Learning) คณะสถิติประยุกต์ หลักสูตรวิทยาศาสตรมหาบัณฑิต การวิเคราะห์ธุรกิจและวิทยาการข้อมูล สถาบันบัณฑิตพัฒนบริหารศาสตร์