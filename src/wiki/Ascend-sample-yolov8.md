## YOLOv4

## YOLOv5

## YOLOv6

## YOLOv7

## YOLOv8

- pt2onnx.py

- onnx2om.py
  - aipp.cfg
- 在[sampleYOLOV7](https://gitee.com/ascend/samples/tree/master/inference/modelInference/sampleYOLOV7)中添加yolov8的后处理方法



- input_shape: (1,3,640,640)
- output_shape: (1，84，8400)
  - N、(xywh,classScore*80)、modelOutputBoxNum

```c
Result SampleYOLOV8::GetResult(
    std::vector<InferenceOutput> &inferOutputs,
    string imagePath, 
    size_t imageIndex,
    bool release
) {
  uint32_t outputDataBufId = 0;
  float *classBuff = static_cast<float *>(inferOutputs[outputDataBufId].data.get());
  // confidence threshold
  float confidenceThreshold = 0.35;

  // class number
  size_t classNum = 80;

  //// number of (x, y, width, hight)
  size_t offset = 4;

  // total number of boxs yolov8 [1,84,8400]
  size_t modelOutputBoxNum = 8400;

  // read source image from file
  cv::Mat srcImage = cv::imread(imagePath);
  int srcWidth = srcImage.cols;
  int srcHeight = srcImage.rows;

  // filter boxes by confidence threshold
  vector<BoundBox> boxes;
  size_t yIndex = 1;
  size_t widthIndex = 2;
  size_t heightIndex = 3;

  // size_t all_num = 1 * 84 * 8400 ; // 705,600

  for (size_t i = 0; i < modelOutputBoxNum; ++i) {
    float maxValue = 0;
    size_t maxIndex = 0;
    for (size_t j = 0; j < classNum; ++j) {
      float value = classBuff[(offset + j) * modelOutputBoxNum + i];
      if (value > maxValue) {
        // index of class
        maxIndex = j;
        maxValue = value;
      }
    }

    if (maxValue > confidenceThreshold) {
      BoundBox box;
      box.x = classBuff[i] * srcWidth / modelWidth_;
      box.y = classBuff[yIndex * modelOutputBoxNum + i] * srcHeight / modelHeight_;
      box.width = classBuff[widthIndex * modelOutputBoxNum + i] * srcWidth / modelWidth_;
      box.height = classBuff[heightIndex * modelOutputBoxNum + i] * srcHeight / modelHeight_;
      box.score = maxValue;
      box.classIndex = maxIndex;
      box.index = i;
      if (maxIndex < classNum) {
        boxes.push_back(box);
      }
    }
  }

  ACLLITE_LOG_INFO(
      "filter boxes by confidence threshold > %f success, boxes size is %ld",
      confidenceThreshold, boxes.size());

  // filter boxes by NMS
  vector<BoundBox> result;
  result.clear();
  float NMSThreshold = 0.45;
  int32_t maxLength = modelWidth_ > modelHeight_ ? modelWidth_ : modelHeight_;
  std::sort(boxes.begin(), boxes.end(), sortScore);
  BoundBox boxMax;
  BoundBox boxCompare;
  while (boxes.size() != 0) {
    size_t index = 1;
    result.push_back(boxes[0]);
    while (boxes.size() > index) {
      boxMax.score = boxes[0].score;
      boxMax.classIndex = boxes[0].classIndex;
      boxMax.index = boxes[0].index;

      // translate point by maxLength * boxes[0].classIndex to
      // avoid bumping into two boxes of different classes
      boxMax.x = boxes[0].x + maxLength * boxes[0].classIndex;
      boxMax.y = boxes[0].y + maxLength * boxes[0].classIndex;
      boxMax.width = boxes[0].width;
      boxMax.height = boxes[0].height;

      boxCompare.score = boxes[index].score;
      boxCompare.classIndex = boxes[index].classIndex;
      boxCompare.index = boxes[index].index;

      // translate point by maxLength * boxes[0].classIndex to
      // avoid bumping into two boxes of different classes
      boxCompare.x = boxes[index].x + boxes[index].classIndex * maxLength;
      boxCompare.y = boxes[index].y + boxes[index].classIndex * maxLength;
      boxCompare.width = boxes[index].width;
      boxCompare.height = boxes[index].height;

      // the overlapping part of the two boxes
      float xLeft = max(boxMax.x, boxCompare.x);
      float yTop = max(boxMax.y, boxCompare.y);
      float xRight = min(boxMax.x + boxMax.width, boxCompare.x + boxCompare.width);
      float yBottom = min(boxMax.y + boxMax.height, boxCompare.y + boxCompare.height);
      float width = max(0.0f, xRight - xLeft);
      float hight = max(0.0f, yBottom - yTop);
      float area = width * hight;
      float iou = area / (boxMax.width * boxMax.height + boxCompare.width * boxCompare.height - area);

      // filter boxes by NMS threshold
      if (iou > NMSThreshold) {
        boxes.erase(boxes.begin() + index);
        continue;
      }
      ++index;
    }
    boxes.erase(boxes.begin());
  }

  ACLLITE_LOG_INFO(
      "filter boxes by NMS threshold > %f success, result size is %ld",
      NMSThreshold, result.size());

  // opencv draw label params
  const double fountScale = 0.5;
  const uint32_t lineSolid = 2;
  const uint32_t labelOffset = 11;
  const cv::Scalar fountColor(0, 0, 255);  // BGR
  const vector<cv::Scalar> colors{cv::Scalar(255, 0, 0), cv::Scalar(0, 255, 0), cv::Scalar(0, 0, 255)};

  int half = 2;
  for (size_t i = 0; i < result.size(); ++i) {
    cv::Point leftUpPoint, rightBottomPoint;
    leftUpPoint.x = result[i].x - result[i].width / half;
    leftUpPoint.y = result[i].y - result[i].height / half;
    rightBottomPoint.x = result[i].x + result[i].width / half;
    rightBottomPoint.y = result[i].y + result[i].height / half;
    cv::rectangle(srcImage, leftUpPoint, rightBottomPoint, colors[i % colors.size()], lineSolid);
    string className = label[result[i].classIndex];
    string markString = to_string(result[i].score) + ":" + className;

    ACLLITE_LOG_INFO("object detect [%s] success", markString.c_str());

    cv::putText(
        srcImage, 
        markString,
        cv::Point(leftUpPoint.x, leftUpPoint.y + labelOffset),
        cv::FONT_HERSHEY_COMPLEX, fountScale, fountColor);
  }
  string savePath = "out_" + to_string(imageIndex) + ".jpg";
  cv::imwrite(savePath, srcImage);
  if (release) {
    free(classBuff);
    classBuff = nullptr;
  }
  return SUCCESS;
}
```



## YOLOv9

- [sampleYOLOv8]()

```bash
git clone https://github.com/WongKinYiu/yolov9
conda create -n yolov9 python==3.10
conda activate yolov9
pip install -r requirements.txt
```

```bash
python export.py --weights yolov9-c-converted.pt --include onnx
atc --model=yolov9-c-converted.onnx \
	--framework 5 \
	--output yolov9-c-converted \
	--input_shape "images:1,3,640,640" \
	--soc_version Ascend310P3 \
	--insert_op_conf aipp.cfg
```

## YOLOv10

- input_shape: (1,3,640,640)
  - NCHW
  - 和v8一样，rbg,chw,归一化，（需要注意的是，yolov10、yolov8预处理都没有标准化，yolov5有）
- output_shape: (1，300，6)
  - N、maxBoxNumber、（left,top,right,bottom,confidence,classification）

- https://github.com/THU-MIG/yolov10
- 模型输入1，3，640，640-->

```C
if (frame.empty()) {
    ERROR_LOG("The input of v10detect model is empty！");
    return FAILED;
}

//原图的宽和高
this->src_Width_ = frame.cols; 
this->src_Height_ = frame.rows; 

//================前处理==================
cv::resize(frame, frame, cv::Size(v10detect_modelHeight_,v10detect_modelWidth_)); // RESIZE
cv::cvtColor(frame, frame, cv::COLOR_BGR2RGB); // BGR2RGB 
cv::Mat nchwMat(frame.rows, frame.cols * frame.channels(), CV_32FC1);
uint8_t* ptMat = frame.ptr<uint8_t>(0);
int height = frame.rows;
int width = frame.cols;
int channels = frame.channels();
int area = height * width;

for (int c = 0; c < channels; ++c)
{
    for (int h = 0; h < height; ++h)
    {
        for (int w = 0; w < width; ++w)
        {
            int srcIdx = h * width * channels + w * channels + c;
            int dstIdx = c * area + h * width + w;
            float pixelValue = ptMat[srcIdx]/ 255.0f;
            nchwMat.at<float>(dstIdx) = pixelValue;
        }
    }
}
```

```C
//计算仿射矩阵和仿射逆矩阵
void Objectv10Detect::Affine(uint32_t src_Width, uint32_t src_Height, uint32_t Dst_Width, uint32_t Dst_Height,
            std::vector<std::vector<float>>& Affine_Matrix, std::vector<std::vector<float>>& Inverse_Affine_Matrix) {
    float scalw = static_cast<float>(Dst_Width) / src_Width;
    float scalh = static_cast<float>(Dst_Height) / src_Height;
 
    // 仿射变换矩阵
    Affine_Matrix = {{scalw, 0, 0},{0, scalh, 0}};
 
    // 仿射逆矩阵
    Inverse_Affine_Matrix = {{1 / scalw, 0, 0},{0, 1 / scalh, 0}};
}
 
 
Result Objectv10Detect::Postprocess(aclmdlDataset* modelOutput,std::vector<std::vector<float>>& results)
{
    // Get feature vector data
    uint32_t dataSize = 0;
    float* featureData = (float*)GetInferenceOutputItem(dataSize, modelOutput,
                                                       0);
 
    if (featureData == nullptr) return FAILED;
 
    //输出是300，6
    results.resize(300);
    for (size_t i = 0; i < 300; ++i) {
        results[i].assign(featureData + i * 6, featureData + (i + 1) * 6);
    }
 
    if (v10detect_runMode_ == ACL_HOST) {
        delete[] ((float*)featureData);
    }
 
    //保留置信度大于阈值的
    std::vector<std::vector<float>> updated_results;
    for (const std::vector<float>& row : results) {
        if ( row[4] >= this->conf_th_) {
            updated_results.push_back(row);
        }
    }
    //更新results
    results=updated_results;
    
    //解码关键点
    std::vector<std::vector<float>> M, IM; // 声明仿射矩阵 M、仿射逆矩阵 IM
    Objectv10Detect::Affine(this->src_Width_, this->src_Height_, this->v10detect_modelWidth_, this->v10detect_modelHeight_, M, IM); // 计算仿射矩阵
    for (int i=0;i<results.size();++i)
    {
        // 提取信息:需要注意的是，v10输出不再是cx,cy,w,h而是修改为了left,top,right,bottom
        float left = results[i][0];//left 
        float top = results[i][1];//top  
        float right = results[i][2];//right  
        float bottom = results[i][3];//bottom  
 
        //仿射变换解码
        left = IM[0][0] * left + IM[0][2];
        top = IM[1][1] * top + IM[1][2];
        right = IM[0][0] * right + IM[0][2];
        bottom = IM[1][1] * bottom + IM[1][2];
 
        results[i][0]=left;
        results[i][1]=top;
        results[i][2]=right;
        results[i][3]=bottom;
    }
    
    return SUCCESS;
}
```

