# Real/Toy Dog Classification (POWER DS TEAM)
## Introduction

การ classify สุนัขของเล่นนั้น เป็นสิ่งที่ pretrained model ที่ทำการ train ด้วย Imagenet dataset นั้นยังทำได้ไม่ดีนัก
เนื่องจากใน 1000 classes ของ imagenet มีเพียง 5 class เท่านั้นที่จัดเป็นสุนัขของเล่น ('miniature_poodle','toy_terrier','toy_poodle','miniature_schnauzer','miniature_pinscher')
และ 5 classes นั้น ยังเป็นการแยกสุนัขของเล่นตามสายพันธุ์อีกด้วย ทำให้ไม่ครอบคลุมสุนัขของเล่นทั้งหมด ทางกลุ่มเราจึงต้องการที่จะสร้าง model ที่สามารถแบ่งแยกระหว่างสุนัขจริง และสุนัขของเล่นได้อย่างมีประสิทธิภาพ
ด้วยการรวมรวบรูปภาพของสุนัขจริง และสุนัขของเล่นขึ้นมาเพื่อเป็น dataset สำหรับใช้ในการ train model สำหรับแบ่งแยกระหว่างสุนัขจริง และสุนัขของเล่น

## Data

ทางกลุ่มเราได้ทำการรวบรวมรูปภาพโดยแบ่งเป็น 2 classes คือ 1. สุนัขจริง (Real Dog) 2. สุนัขของเล่น (Toy Dog)

#### Data source

โดยเราได้ทำการรวบรวมรูปภาพจากเว็บไซต์ต่อไปนี้
1. www.flickr.com
2. www.pexels.com
3. www.pinterest.com
4. pixabay.com

โดยจำนวนรูปภาพที่เราสามารถรวบรวมได้สำหรับแต่ล่ะ class มีจำนวนทั้งหมด 1148 รูป แบ่งเป็น
- Real Dog: 471 รูป
- Toy Dog: 677 รูป

#### Data pre-processing

รูปภาพทั้งหมดจะถูก preprocess โดยจาก resize ให้อยู่ในขนาด 224 x 224 สำหรับ vgg16 และ 380 x 380 สำหรับ efficientnet-b4 และแปลงให้กลายเป็น array (3-channels)
จากนั้นจึงนำ array ที่ได้ไปผ่าน function preprocess_input ของ keras ตาม network ที่เลือกใช้ (e.g. vgg16)


#### Data splitting
    
เราได้ทำการแบ่ง data ออกเป็น 3 ส่วนดังนี้
- train 50%
- validation 20%
- test 30%

## Network architecture

เราเลือกใช้ pretrained model 2 ตัว ได้แก่ VGG16 และ Efficientnet-B4 เป็น model ที่ใช้ในการเปรียบเทียบประสิทธิภาพ และใช้สำหรับทำ feature extraction โดยรายละเอียดของ 2 model จะมีดังนี้

#### VGG16


#### Efficientnet-B4


## Training

#### VGG16 as Feature Extractor

    Trained on 1 NVDIA RTX 3080

    - Optimizer: Adam
    - Learning rate: 0.001
    - Loss Function: BinaryCrossentropy
    - Batch size: 128
    - Epoch: 10

    เวลาที่ใช้ในการ Train 22 วินาที

#### Efficientnet-B4 as Feature Extractor

    Trained on 1 NVDIA RTX 3080

    - Optimizer: Adam
    - Learning rate: 0.001
    - Loss Function: BinaryCrossentropy
    - Batch size: 32
    - Epoch: 10

    เวลาที่ใช้ในการ Train 66 วินาที

## Results

#### Train/Validation Data Result


#### Test Data Result


#### Pretrained Result Comparison

## Discussion


## Conclusion


## References
- https://towardsdatascience.com/complete-architectural-details-of-all-efficientnet-models-5fd5b736142

## Members
- (20%) 6220422048 กชกร เรืองศรี ()
- (0%) 6220422061 ไตรเทพ จันทร์เทพ (ติดต่อไม่ได้)
- (20%) 6220422065 สุธาสินี โพธิ์แจ่ม ()
- (20%) 6310422028 วรเมธ ปลอดโปร่ง ()
- (20%) 6310422031 ธนัตถ์กรณ์ ชื่นบรรลือสุข ()
- (20%) 6310422046 วีระศักดิ์ การุณย์ ()