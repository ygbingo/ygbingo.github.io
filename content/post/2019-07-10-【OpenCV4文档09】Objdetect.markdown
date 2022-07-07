---
layout:     post
title:      【OpenCV4文档09】Objdetect
subtitle:   如何进行人脸识别？
date:       2019-07-10
author:     Bingo
header-img: img/post-bg-open4v.png
catalog: true
tags:
    - 机器学习
    - 图像处理
    - Java
    - OpenCV
    - Objdetect
---

# 级联分类器
- 支持语言：C++，Java，Python
- 编译条件：> OpenCV 2.0
学习如何从图片或视频中找到目标。

### 目标
- 使用 cv::CascadeClassifier 类从视频流中检测物体，将用到如下方法：
    - cv::CascadeClassifier::load 加载一个.xml分类文件，可以是Haar或LBP分类器
    - cv::CascadeClassifier::detectMultiScale 用来执行检测

### 代码
Java Code
```java
import java.util.List;
import org.opencv.core.Core;
import org.opencv.core.Mat;
import org.opencv.core.MatOfRect;
import org.opencv.core.Point;
import org.opencv.core.Rect;
import org.opencv.core.Scalar;
import org.opencv.core.Size;
import org.opencv.highgui.HighGui;
import org.opencv.imgproc.Imgproc;
import org.opencv.objdetect.CascadeClassifier;
import org.opencv.videoio.VideoCapture;
class ObjectDetection {
    public void detectAndDisplay(Mat frame, CascadeClassifier faceCascade, CascadeClassifier eyesCascade) {
        Mat frameGray = new Mat();
        Imgproc.cvtColor(frame, frameGray, Imgproc.COLOR_BGR2GRAY);
        Imgproc.equalizeHist(frameGray, frameGray);
        // -- Detect faces
        MatOfRect faces = new MatOfRect();
        faceCascade.detectMultiScale(frameGray, faces);
        List<Rect> listOfFaces = faces.toList();
        for (Rect face : listOfFaces) {
            Point center = new Point(face.x + face.width / 2, face.y + face.height / 2);
            Imgproc.ellipse(frame, center, new Size(face.width / 2, face.height / 2), 0, 0, 360,
                    new Scalar(255, 0, 255));
            Mat faceROI = frameGray.submat(face);
            // -- In each face, detect eyes
            MatOfRect eyes = new MatOfRect();
            eyesCascade.detectMultiScale(faceROI, eyes);
            List<Rect> listOfEyes = eyes.toList();
            for (Rect eye : listOfEyes) {
                Point eyeCenter = new Point(face.x + eye.x + eye.width / 2, face.y + eye.y + eye.height / 2);
                int radius = (int) Math.round((eye.width + eye.height) * 0.25);
                Imgproc.circle(frame, eyeCenter, radius, new Scalar(255, 0, 0), 4);
            }
        }
        //-- Show what you got
        HighGui.imshow("Capture - Face detection", frame );
    }
    public void run(String[] args) {
        String filenameFaceCascade = args.length > 2 ? args[0] : "../../data/haarcascades/haarcascade_frontalface_alt.xml";
        String filenameEyesCascade = args.length > 2 ? args[1] : "../../data/haarcascades/haarcascade_eye_tree_eyeglasses.xml";
        int cameraDevice = args.length > 2 ? Integer.parseInt(args[2]) : 0;
        CascadeClassifier faceCascade = new CascadeClassifier();
        CascadeClassifier eyesCascade = new CascadeClassifier();
        if (!faceCascade.load(filenameFaceCascade)) {
            System.err.println("--(!)Error loading face cascade: " + filenameFaceCascade);
            System.exit(0);
        }
        if (!eyesCascade.load(filenameEyesCascade)) {
            System.err.println("--(!)Error loading eyes cascade: " + filenameEyesCascade);
            System.exit(0);
        }
        VideoCapture capture = new VideoCapture(cameraDevice);
        if (!capture.isOpened()) {
            System.err.println("--(!)Error opening video capture");
            System.exit(0);
        }
        Mat frame = new Mat();
        while (capture.read(frame)) {
            if (frame.empty()) {
                System.err.println("--(!) No captured frame -- Break!");
                break;
            }
            //-- 3. Apply the classifier to the frame
            detectAndDisplay(frame, faceCascade, eyesCascade);
            if (HighGui.waitKey(10) == 27) {
                break;// escape
            }
        }
        System.exit(0);
    }
}
public class ObjectDetectionDemo {
    public static void main(String[] args) {
        // Load the native OpenCV library
        System.loadLibrary(Core.NATIVE_LIBRARY_NAME);
        new ObjectDetection().run(args);
    }
}
```

### 效果
1. 以下是运行上面代码并使用内置网络摄像头的视频流作为输入的结果：
![](https://docs.opencv.org/4.1.0/Cascade_Classifier_Tutorial_Result_Haar.jpg)
确保程序将找到文件路径haarcascade_frontalface_alt.xml和haarcascade_eye_tree_eyeglasses.xml。
它们位于opencv / data / haarcascades中。

2. 这是使用文件lbpcascade_frontalface.xml（LBP训练）进行面部检测的结果。
对于眼睛，我们继续使用教程中使用的文件
![](https://docs.opencv.org/4.1.0/Cascade_Classifier_Tutorial_Result_LBP.jpg)

### 其他资源
1. Paul Viola和Michael J. Jones。强大的实时人脸检测功能。国际计算机视觉杂志，57（2）：137-154,2004。[223](https://docs.opencv.org/4.1.0/d0/de3/citelist.html#CITEREF_Viola04)
2. Rainer Lienhart和Jochen Maydt。用于快速物体检测的一组扩展的类似哈尔的功能。在图像处理中.2002年。会议录。2002年国际会议，第1卷，第I-900页。IEEE，2002。[126](https://docs.opencv.org/4.1.0/d0/de3/citelist.html#CITEREF_Lienhart02)
3. 关于人脸检测和跟踪的[视频讲座](https://www.youtube.com/watch?v=WfdYYNamHZ8)
4. [Adam Harvey关于人脸探测的有趣采访](https://web.archive.org/web/20171204220159/http://www.makematics.com/research/viola-jones/)
5. [OpenCV人脸检测](https://vimeo.com/12774628)：由Adam Harvey在Vimeo上可视化

# 训练级联分类器
本教程描述了opencv_traincascade应用程序及其参数。

### 介绍
使用增强级联的弱分类器包括两个主要阶段：训练和检测阶段。在对象检测教程中描述了使用基于HAAR或LBP的模型的检测阶段。本文档概述了训练您自己的弱分类器级联所需的功能。本指南将介绍所有不同阶段：收集培训数据，准备培训数据和执行实际模型培训。

为了支持本教程，将使用几个官方OpenCV应用程序：opencv_createsamples，opencv_annotation，opencv_traincascade和opencv_visualisation。

### 重要提醒
- 如果您遇到任何提及旧的opencv_haartraining工具的教程（已弃用且仍在使用OpenCV1.x界面），请忽略该教程并坚持使用opencv_traincascade工具。此工具是较新版本，根据OpenCV 2.x和OpenCV 3.x API以C ++编写。opencv_traincascade支持HAAR，如小波特征[222]和LBP（局部二进制模式）[124]特征。与HAAR特征相比，LBP特征产生整数精度，产生浮点精度，因此使用LBP进行训练和检测的速度比使用HAAR特征快几倍。关于LBP和HAAR检测质量，它主要取决于所使用的训练数据和所选择的训练参数。在训练时间的百分比内，可以训练基于LBP的分类器，该分类器将提供与基于HAAR的分类器几乎相同的质量。
- OpenCV 2.x和OpenCV 3.x（cv :: CascadeClassifier）的较新的级联分类器检测接口支持使用新旧模型格式。opencv_traincascade甚至可以保存（导出）旧格式的训练级联，如果由于某种原因您使用旧接口卡住了。然后，至少可以在最稳定的界面中训练模型。
- opencv_traincascade应用程序可以使用TBB进行多线程处理。要在多核模式下使用它，必须在启用TBB支持的情况下构建OpenCV。

### 准备训练数据
为了训练增强的弱分类器级联，我们需要一组正样本（包含您想要检测的实际对象）和一组负片图像（包含您不想检测的所有内容）。必须手动准备一组负样本，而使用opencv_createsamples应用程序创建一组正样本。

#### 负样本
负样本取自任意图像，不包含您要检测的对象。生成样本的这些负片图像应列在每行包含一个图像路径的特殊负片图像文件中（可以是绝对的或相对的）。请注意，负样本和样本图像也称为背景样本或背景图像，在本文档中可互换使用。

所描述的图像可以具有不同的尺寸。但是，每个图像应该等于或大于所需的训练窗口大小（对应于模型尺寸，大多数时候是对象的平均大小），因为这些图像用于将给定的负图像子采样为多个图像具有此培训窗口大小的样本。

一个负样本集文件的例子：
目录结构：
```linux
/img
  img1.jpg
  img2.jpg
bg.txt
```
bg.txt文件
```txt
img/img1.jpg
img/img2.jpg
```

在您尝试查找感兴趣的对象时，您的负窗口样本集将用于告知机器学习步骤，在这种情况下提升，不要寻找什么。

#### 正样本
正面样本由opencv_createsamples应用程序创建。它们被提升过程用于定义模型在尝试查找感兴趣的对象时应该实际查找的内容。该应用程序支持两种生成正样本数据集的方法。

1. 您可以从单个正对象图像生成一堆正样本。
2. 您可以自己提供所有正面信息，只使用该工具将其剪切掉，调整大小并将它们置于opencv所需的二进制格式中。

虽然第一种方法适用于固定物体，例如非常坚硬的徽标，但对于刚性较小的物体，它往往会很快失败。在这种情况下，我们建议使用第二种方法。通过使用opencv_createsamples应用程序，Web上的许多教程甚至声明100个真实对象图像可以产生比1000个人工生成的正面更好的模型。如果您决定采取第一种方法，请记住以下事项：

- 请注意，在将其提交给上述应用程序之前，您需要多个正面样本，因为它仅应用透视变换。
- 如果您需要一个健壮的模型，请采样涵盖您的对象类中可能出现的各种变化的样本。例如，在面部的情况下，你应该考虑不同的种族和年龄组，情绪和胡子风格。这在使用第二种方法时也适用。

第一种方法采用具有例如公司徽标的单个对象图像，并通过随机旋转对象，改变图像强度以及将图像放置在任意背景上来从给定对象图像创建大量正样本。随机性的数量和范围可以通过opencv_createsamples应用程序的命令行参数来控制。

命令行参数：
- ```-vec <vec_file_name>``` : 包含用于训练的正样本的输出文件的名称。
- ```-img <image_file_name>``` : 源对象图像（例如，公司徽标）。
- ```-bg <background_file_name>``` ：后台描述文件;包含一个图像列表，用作对象的随机扭曲版本的背景。
- ```-num <number_of_samples>``` ：要生成的正​​样本数。
- ```-bgcolor <background_color>``` ：背景颜色（假设当前是灰度图像）;背景颜色表示透明色。由于可能存在压缩瑕疵，因此可以通过-bgthresh指定颜色容差量。bgcolor-bgthresh和bgcolor + bgthresh范围内的所有像素都被解释为透明。
- ```-bgthresh <background_color_threshold>```
- ```-inv``` ：如果指定，颜色将被反转。
- ```-randinv``` ：如果指定，颜色将随机反转。
- ```-maxidev <max_intensity_deviation>``` : 前景样本中像素的最大强度偏差。
- ```-maxxangle <max_x_rotation_angle>``` ：朝向x轴的最大旋转角度必须以弧度给出。
- ```-show``` ：有用的调试选项。如果指定，将显示每个样品。按Esc将继续样品创建过程，而不显示每个样品。
- ```-w <sample_width>``` ：输出样本的宽度（以像素为单位）。
- ```-h <sample_height>```：输出样本的高度（以像素为单位）。

以这种方式运行opencv_createsamples时，以下过程用于创建示例对象实例：给定的源图像围绕所有三个轴随机旋转。所选角度受```-maxxangle```，```-maxyangle```和```-maxzangle```限制。然后像素的强度来自[bg_color-bg_color_threshold;
bg_color + bg_c​​olor_threshold]范围被解释为透明。白噪声被添加到前景的强度。如果指定```-inv```键，则反转前景像素强度。如果指定了```-randinv```键，则算法随机选择是否应将反演应用于此样本。最后，将获得的图像放置在背景描述文件的任意背景上，调整大小为```-w```和```-h```指定的所需大小，并存储到由```-vec```命令行选项指定的vec文件中。

也可以从先前标记的图像的集合获得正样本，这是构建稳健对象模型时的期望方式。此集合由类似于背景描述文件的文本文件描述。该文件的每一行对应一个图像。该行的第一个元素是文件名，后跟对象注释的数量，后跟描述边界（x，y，宽度，高度）边界的对象坐标的数字。

文件描述的例子：

目录结构：
```linux
/img
    img1.jpg
    img2.jpg
info.dat
```

info.dat:
```dat
img/img1.jpg  1  140 100 45 45
img/img2.jpg  2  100 200 50 50   50 30 25 25
```

Image img1.jpg包含具有以下边界矩形坐标的单个对象实例：（140,100,45,45）。
Image img2.jpg包含两个对象实例.

为了从这样的集合中创建正样本，应该指定```-info```参数而不是```-img```：
- ```-info <collection_file_name>``` ：标记图像集合的描述文件。

请注意，在这种情况下，```-bg```，```-bgcolor```，```-bgthreshold```，```-inv```，```-randinv```，```-maxxangle```，```-maxyangle```，```-maxzangle```等参数将被忽略，不再使用。在这种情况下，样本创建方案如下。通过从原始图像中剪切出所提供的边界框，从给定图像中获取对象实例。然后将它们调整为目标样本大小（由```-w```和```-h```定义）并存储在由```-vec```参数定义的输出vec文件中。没有应用失真，因此唯一影响的参数是```-w```，```-h```，```-show```和```-num```。

创建```-info```文件的手动过程也可以使用opencv_annotation工具完成。这是一个开源工具，用于在任何给定图像中可视地选择对象实例的感兴趣区域。以下小节将更详细地讨论如何使用此应用程序。

#### 另外注意
- opencv_createsamples实用程序可用于检查存储在任何给定的正样本文件中的样本。为此，只应指定-vec，-w和-h参数。
- vec文件的示例可在此处获得```opencv / data / vec_files / trainingfaces_24-24.vec```。它可用于训练具有以下窗口大小的面部检测器：```-w 24 -h 24```。

#### 使用OpenCV的集成注释工具
自OpenCV 3.x以来，社区一直在提供和维护一个开源注释工具，用于生成-info文件。如果OpenCV应用程序在哪里构建，则可以通过命令opencv_annotation访问该工具。

使用该工具非常简单。该工具接受几个必需参数和一些可选参数：
- ```--annotations```（必需）：注释txt文件的路径，您要在其中存储注释，然后将其传递给-info参数[example - /data/annotations.txt]。
- ```--images```（必填）：包含带有对象的图像的文件夹的路径[example - / data / testimages /]
- ```--maxWindowHeight```（可选）：如果输入图像的高度大于此处的给定分辨率，请使用```--resizeFactor```调整图像大小以便于注释。
- ```--resizeFactor```（可选）：使用```--maxWindowHeight```参数时用于调整输入图像大小的因子。

**请注意，可选参数只能一起使用。可以使用的命令的示例可以在下面看到。**
``` shell
opencv_annotation --annotations=/path/to/annotations/file.txt --images=/path/to/image/folder/
```

此命令将启动一个窗口，其中包含将用于注释的第一个图像和鼠标光标。
可以在此处找到有关如何使用注释工具的视频。
基本上有几个按键触发动作。
鼠标左键用于选择对象的第一个角，然后保持绘图直到你没事，并在注册第二个鼠标左键单击时停止。
每次选择后，您都有以下选择：
- 按```c```：确认注释，​​将注释变为绿色并确认存储;
- 按```d```：从注释列表中删除最后一个注释（便于删除错误的注释）;
- 按```n```：继续下一张图像;
- 按```ESC```：这将退出注释软件

最后，您将得到一个可用的注释文件，该文件可以传递给opencv_createsamples的```-info```参数。

### 级联训练
下一步是基于事先准备的正负数据集对弱分类器的增强级联进行实际训练。

按目的分组的opencv_traincascade应用程序的命令行参数：

- 常用参数：
    - ```-data <cascade_dir_name>```：应存储训练分类器的位置。应事先手动创建此文件夹。
    - ```-vec <vec_file_name>```：带有正样本的vec文件（由opencv_createsamples实用程序创建）。
    - ```-bg <background_file_name>```：后台描述文件。这是包含负样本图像的文件。
    - ```-numPos <number_of_positive_samples>```：每个分类器阶段的训练中使用的正样本数。
    - ```-numNeg <number_of_negative_samples>```：每个分类器阶段的训练中使用的负样本数。
    - ```-numStages <number_of_stages>```：要训练的级联阶段的数量。
    - ```-precalcValBufSize <precalculated_vals_buffer_size_in_Mb>```：预先计算的特征值的缓冲区大小（以Mb为单位）。分配的内存越多，训练过程就越快，但请记住，```-precalcValBufSize```和```-precalcIdxBufSize```的组合不应超过可用的系统内存。
    - ```-precalcIdxBufSize <precalculated_idxs_buffer_size_in_Mb>```：预先计算的特征索引的缓冲区大小（以Mb为单位）。分配的内存越多，训练过程就越快，但请记住，```-precalcValBufSize```和```-precalcIdxBufSize```的组合不应超过可用的系统内存。
    - ```-baseFormatSave```：这个参数在类Haar特征的情况下是实际的。如果已指定，则级联将以旧格式保存。这仅用于向后兼容性原因，并且允许用户坚持旧的已弃用接口，至少使用较新的接口训练模型。
    - ```-numThreads <max_number_of_threads>```：训练期间使用的最大线程数。请注意，实际使用的线程数可能会更低，具体取决于您的计算机和编译选项。默认情况下，如果您使用TBB支持构建OpenCV，则会选择最大可用线程，这是此优化所需的。
    - ```-acceptanceRatioBreakValue <break_value>```：此参数用于确定模型应该保持学习的准确程度以及何时停止。一个好的指导方针是训练不超过10e-5，以确保模型不会超过您的训练数据。默认情况下，此值设置为-1以禁用此功能。
- 级联参数：
    - ```-stageType <BOOST（默认值）>```：阶段的类型。目前仅支持增强分类器作为阶段类型。
    - ```-featureType <{HAAR（默认值），LBP}>```：功能类型：HAAR - 类Haar功能，LBP - 本地二进制模式。
    - ```-w <sampleWidth>```：训练样本的宽度（以像素为单位）。
    - ```-h <sampleHeight>```：训练样本的高度（以像素为单位）。必须具有与创建训练样本期间使用的值完全相同的值（opencv_createsamples实用程序）。
