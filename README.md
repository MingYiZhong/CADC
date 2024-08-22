# CADC察打一体经验指导
## 1. 目标检测
### 1.准备数据集
1. 准备原始数据集：  
- 需要准备的工具  
无人机（垂直起降最好，旋翼次之）  
一比一真实天井以及围挡若干套（红色、蓝色）  
不同字体的0-8数字  
- 方法：  
   1. 将比赛用的摄像头以某种方式放到无人机上，无人机飞到比赛需要的合适高度，分别拍摄目标在不同方位的图片(黑色的为目标位置）：  
   ![alt text](demo.png)  
   2. 将目标截取下来，得到天井以及单个数字的照片,放到datasets/img这个文件夹中，天井命名为r_m.jpg数字命名为n_m.jpg（r为固定名字,n为数字大小，m为第多少张图片）,例如：  
   ![alt text](r-6.png), ![alt text](1-2.png),![alt text](2-3.png), 天井只需要一组，数字则需要很多组。
2. 生成合成数据集：
- 需要准备的工具  
笔记本电脑(支持nvidia的cuda和cudnn,同时下载好vscode,能使用jupyter notebook),多台笔记本可同时运行提高效率
- 方法    
[下载并学会使用labelme](https://blog.csdn.net/zhanzhengrecheng/article/details/141287558 "labelme教程")  
标签为0-8九个数字（9通过算法判断）,r(红蓝天井),left,right,center,up（四个关键点）以及`_backgrounds_`,labels文件我直接放在了E:/目录下,内容如下：
[labels.txt](../labels.txt)  
打开labelme,图片文件夹选择img,更改输出路径为json,标注格式请参考文件夹原本自带示例。需要注意的是，只需要标注天井，数字后面有程序自动标注，但必须先标注类别再标注关键点。  
待标注完后，更改输出路径为polygon_json,标注格式依然参考文件夹自带示例，也只需要标注天井。此文件夹作用为生成蒙版，消除背景的干扰。  
待标注完成后,单个数字的合成数据集准备工作已经完成。  
接着将img的图片复制到img_two,天井只需要一张图片即可,命名为well.jpg,天井的照片需要通过P图等方式消除背景。
打开function.ipynb,首先运行第一段程序。接着根据提示修改第二段程序中的下列变量并运行:  
`
    times = 1000 # 生成图片的数量`  
    `mode = train #train对应生成训练集，val对应生成验证集`  
    `height_num = 19 #well上对应的数字目标高`  
    `width_num = 10 #well上对应的数字目标宽`  
    `top_num = 35 #well上数字目标的顶端`  
    `left_num = 10 #well上数字目标的最左端`  
    `width_img = 38 #图像宽度`  
    `height_img = 65 #图像高度`  
    `imagePath":"E:/neo_CADC/key_point/datasets/images/comp_{}/{}.png".format(mode,img_name),#这里需要修改datasets的路径`  
    `cv2.imwrite('E:/neo_CADC/key_point/datasets/images/comp_{}/{}.png'.format(mode,img_name),final_img)#这里需要修改datasets的路径`。
    运行完代码，images/comp_train或者images/comp_val就会生成二位数的合成数据,并会在json/zcomp_train或者json/zcomp_val中生成json文件。  
    接着打开主目录下的function_r.ipynb,有新的图片加入时,需要先运行`通过labelme的多边形标注获得蒙版`这段程序以获得图像蒙版。接着修改程序中涉及到的文件路径。同时修改`创建和保存合成数据(json)`中此处的数（生成图片数量）  
    ![alt text](1724318520284.png)  
    然后从`引入原始数据集`开始往下运行  
    ![alt text](1724318443845.png)  
    此时就会在images/train或者images/val中生成图片,并会在json/train或者json/val中生成json文件。
    最后将images/comp_train或者images/comp_val的图片分别复制到images/train或者images/val中，json文件夹也如此操作。  
    途中可以用labelme来检验生成的图片是否正确。  
    最后回到主目录，先运行json2coco.py,再运行coco2yolo.py,注意修改程序中的`mode = 'val' #train 或者 val`。至此，合成数据集生成结束，将datasets下的images和labels压缩到一个压缩包中，命名为Posedata,即可为训练做好准备。
   
      


