```cpp
#include "opencv.hpp"
#include "opencv2/highgui/highgui.hpp"
#include "opencv2/imgproc/imgproc.hpp"
#include "opencv2/core/core.hpp"
#include "vector"
#include "iostream"
#include "time.h"
#include "stdlib.h"
#include "stdio.h"


using namespace cv;
using namespace std;

Mat horizontalProjectionMat(Mat src)//水平投影  
{
	Mat src1, src_gray, src_binary, paintX;

	//创建图像框，用于绘制投影图  (黑底，0 黑，  1 白)
	paintX = Mat::zeros(src.rows, src.cols, CV_8UC1);//Mat::zeros 返回指定的大小和类型的零数组。
	imshow("paintx", paintX);
	

	blur(src, src1, Size(7, 7));//均值滤波

	//转化为灰度图像
	cvtColor(src1, src_gray, CV_RGB2GRAY);//颜色空间转换函数
	imshow("灰度化1", src_gray);

	//二值化图像
	threshold(src_gray, src_binary, 70, 255, CV_THRESH_BINARY);
	//第三个参数，阈值的设定值
	//CV_THRESH_BINARY的意思是只要src_gray（x,y）大于70，src_gray（x,y）就会被设定为255
	//threshold(src_gray, src_binary, 100, 255, CV_THRESH_OTSU );

	imshow("二值化1", src_binary);
	int* v = new int[src.cols * 4];
	//动态 int* array = new int[100]; 分配了长度为100的数组array
	//其中src.cols * 4为1024*4为具体的数值4096
	memset(v, 0, src.cols * 4);//初始化内存函数 将v数组全部赋值为0
	int i,j;
	

	for (i = 0; i<src_binary.cols; i++)           //列  
	{
		for (j = 0; j<src_binary.rows; j++)                //行  
		{
			if (src_binary.at<uchar>(j, i) == 255)                //统计的是白色像素的数量  
				v[i]++;                                          //255代表白色  0代表黑色
		}
	}
	//for (i = 0; i < src_binary.cols; i++)
	//{
	//	cout << v[i] <<endl ;
	//}
	//srcImage.at<uchar>(j, i) //表示的是  j 行 i 列 的这个像素
	//srcImage.at<uchar>(Point(j, i)) //表示的是 坐标（j,i）的像素
	//v[i]就是第i列白色像素的个数
	//绘制垂直方向上的投影  
	//绘制垂直方向上的头像

	for (i = 0; i<src_binary.cols; i++)   //列
	{
		for (j = 0; j<v[i]; j++)          //行
		{
			paintX.at<uchar>(j, i) = 255;  //填充白色的像素  
		}
	}
	imshow("垂直投影", paintX);


	Mat dst;
	int startIndex = 0;//记录进入字符区的索引  
	int endIndex = 0;//记录进入空白区域的索引  
	bool inBlock = false;//是否遍历到了字符区内  
	for (int i = 0; i <src.cols; i++)
	{
		if (!inBlock && v[i] != 0)//进入字符区  ！false=true
		{
			inBlock = true;
			startIndex = i;

		}
		else if (inBlock && v[i] == 0)//进入空白区  
		{
			endIndex = i;
			inBlock = false;
			dst = src(Range(0, src.rows), Range(startIndex, endIndex + 1));//从原图中截取有图像的区域 
		}
	}
	delete[] v;
	imshow("轨道", dst);
	return dst;
}


Mat blur(Mat &img)
{
	Mat out;
	blur(img, out, Size(7, 7));//均值滤波
	return out;
}

Mat Grayscale(Mat &img) {
	Mat out;
	cvtColor(img, out, CV_RGB2GRAY);
	return out;
}

Mat TwoValued(Mat &img)
{
	Mat out;
	threshold(img, out, 100, 255, CV_THRESH_OTSU + CV_THRESH_BINARY);//灰度值小于等于阈值的变为0（黑色），大于的变为255（白色），
	return out;
}

Mat paint(Mat &img){
	int x, y;
	for (x = 0; x <img.rows; x++) //行
	{
		for (y = 0; y < img.cols; y++)                //列
		{
			if (y<30 || y>(img.cols - 30) || x<3 || x>(img.rows - 3))       //统计白色像素的数量（数值范围需修改）
			{
				img.at<uchar>(x, y) = 255;
			}
		}
	}
	return img;
}

Mat Contour(Mat &src, int a, int b)//缺陷识别与计算分类模块
//Mat Contour(Mat &src)
{

	RNG rng(12345);//随机生成一些数值
	vector<vector<Point>> g_vContours;//用于检测物体的轮廓
	vector<Vec4i> g_vHierarchy;//用于检测物体的轮廓

	Mat img = blur(src);
	imshow("均值模糊", img);

	img = Grayscale(img);
	imshow("灰度化", img);

	img = TwoValued(img);
	imshow("二值化", img);

	img = paint(img);
	imshow("去除虚假轮廓", img);

	findContours(img, g_vContours,  RETR_TREE, CHAIN_APPROX_SIMPLE, Point(0, 0));
	//findContours(img, g_vContours, g_vHierarchy, RETR_TREE, CHAIN_APPROX_SIMPLE, Point(0, 0));
	//遍历每个轮廓   检测物体的轮廓
	//vector(向量): C++中的一种数据结构,确切的说是一个类.它相当于一个动态的数组,
	//当程序员无法知道自己需要的数组的规模多大时,用其来解决问题可以达到最大节约空间的目的.
	//vector <int> a;(等于声明了一个int数组a[],大小没有指定,可以动态的向里面添加删除)
	//vector< Mat > hull(g_vContours.size());//定义了下面凸包检测函数的第二个参数
	//for (unsigned int i = 0; i < g_vContours.size(); i++)
	//{
	//	convexHull(Mat(g_vContours[i]), hull[i], false);//检测凸包的函数
	//}
	
		cout << g_vContours.size() << endl;
	//外围轮廓的个数   g_vContours.size()
	Mat drawing = Mat::zeros(img.size(), CV_8UC3);//初始化结果图
	imshow("初始化结果图",drawing);
	for (unsigned int i = 0; i < g_vContours.size(); i++){
		cout << "第" << i << "个轮廓" << endl;
		cout << g_vContours[i] << endl;
	}

	vector<Rect> boundRect(g_vContours.size());//Rect 矩形的表示
	vector<RotatedRect> roRect(g_vContours.size());
	Point2f rect[4];

	Mat copyImg = Mat::zeros(img.size(), CV_8UC3);//初始化结果图

	int width = 0;
	int height = 0;
	int x = 0;
	int y = 0;
	for (unsigned int i = 0; i < g_vContours.size(); i++)
	{
		double g_dConArea = contourArea(g_vContours[i], true);//*****缺陷最小外接凸边形轮廓的面积******
		cout << "缺陷最小外接凸边形轮廓的面积" << g_dConArea << endl;
		double g_dConLeng = arcLength(g_vContours[i], true);//*****缺陷最小外接凸边形轮廓的周长*****
		cout << "缺陷最小外接凸边形轮廓的周长" << g_dConLeng << endl;
		//g_dConArea计算图像轮廓的面积
		//g_dConLeng计算图形轮廓的周长
		if (g_dConArea>300)//数值需修改
	/*	for (unsigned int s = 0; g_dConArea>300; s++)*/
		{
			unsigned int s = 0;
			s++;
			//绘制轮廓及其凸包
			//drawContours(drawing, hull, i, Scalar(255, 0, 0), 1, 8, vector<Vec4i>(), 0, Point());
			/*drawContours(src, hull, i, Scalar(255, 0, 0), 1, 8, vector<Vec4i>(), 0, Point());*/

			//由轮廓（点集）确定出正外接矩形并绘制  
			boundRect[i] = boundingRect(Mat(g_vContours[i]));
			//获得正外接矩形的左上角坐标及宽高    
			width = boundRect[i].width;
			height = boundRect[i].height;
			x = boundRect[i].x;
			y = boundRect[i].y;

			rectangle(copyImg, Rect(x, y, width, height), Scalar(0, 0, 255), 2, 8);
			rectangle(src, Rect(x, y, width, height), Scalar(0, 0, 255), 2, 8);

			//由轮廓（点集）确定出最小外接矩形并绘制，旋转矩形主要成员有center、size、 angle、points()  
			roRect[i] = minAreaRect(Mat(g_vContours[i]));
			cout << "roRect[i].center" << roRect[i].center << endl;
			cout << "roRect[i].size" << roRect[i].size << endl;
			cout << "roRect[i].size.width" << roRect[i].size.width << endl;
			cout << "roRect[i].size.heigth" << roRect[i].size.height << endl;

			//roRect[i].points(rect);//把最小外接矩形四个端点复制给rect数组
			//for (int j = 0; j<4; j++)
			//{
			//	line(copyImg, rect[j], rect[(j + 1) % 4], Scalar(255, 0, 0), 2, 8);
			//	line(src, rect[j], rect[(j + 1) % 4], Scalar(255, 0, 0), 2, 8);
			//}

			imshow("最小外接矩形轮廓", copyImg);
			imshow("轨道最小外接矩形轮廓", src);

			//由旋转矩形的center成员得出中心点  
			Point center = roRect[i].center;
			//缺陷最小外接矩形轮廓宽和高
			double widthRotated = roRect[i].size.width;//*****缺陷最小外接矩形轮廓宽*****
			double heightRotated = roRect[i].size.height; //*****缺陷最小外接矩形轮廓高*****
			char widthStr[20] = { 0 };
			char heightStr[20] = { 0 };
			sprintf(widthStr, "width:%.2f", widthRotated);
			sprintf(heightStr, "height:%.2f", heightRotated);
			//缺陷正外接矩形轮廓面积
			double tmparea = boundRect[i].width*boundRect[i].height;//******缺陷正外接矩形轮廓面积*****
			//缺陷轮廓定位
			double total = a * b + (b - roRect[i].center.y);
			//缺陷轮廓的灰度均值和灰度方差
			Mat img_crop;
			getRectSubPix(src, roRect[i].size, roRect[i].center, img_crop);//从原图像中提取提取一个感兴趣的矩形区域图像
			Mat grayResult;
			cvtColor(img_crop, grayResult, CV_BGR2GRAY);// CV_RGB2GRAY
			Mat tempMean, tempStddv;
			double MEAN, STDDV;// mean and standard deviation of the flame region
			double m = mean(grayResult)[0];
			/*cout << "mean=" << m << endl;*/
			meanStdDev(grayResult, tempMean, tempStddv);
			MEAN = tempMean.at<double>(0, 0);//缺陷轮廓灰度均值mean
			STDDV = tempStddv.at<double>(0, 0);//缺陷轮廓灰度方差stddv
			//缺陷轮廓的圆形度、矩形度和高宽比
			double factor = (g_dConArea * 4 * CV_PI) / (pow(g_dConLeng, 2));//*****缺陷最小外接凸边形轮廓的圆形度*****			
			double g_dConrect = g_dConArea / tmparea;//*****缺陷轮廓的矩形度*****				
			double g_dhw = heightRotated / widthRotated;//*****缺陷最小外接矩形轮廓的高宽比*****

			cout << "第" << s << "个最小外接凸边形面积：" << g_dConArea << endl;
			cout << "第" << s << "个最小外接凸边形周长：" << g_dConLeng << endl;
			cout << "第" << s << "个最小外接矩形轮廓角度：" << roRect[i].angle << endl;
			cout << "第" << s << "个最小外接矩形轮廓中心点位置：" << roRect[i].center << endl;
			cout << "第" << s << "个正外接矩形轮廓宽：" << boundRect[i].width << endl;
			cout << "第" << s << "个正外接矩形轮廓高：" << boundRect[i].height << endl;
			cout << "第" << s << "个正外接矩形轮廓面积：" << tmparea << endl;
			cout << "第" << s << "个缺陷轮廓灰度均值mean=" << MEAN << endl;
			cout << "第" << s << "个缺陷轮廓灰度方差stddv=" << STDDV << endl;
			cout << "第" << s << "个最小外接凸边形圆形度：" << factor << endl;
			cout << "第" << s << "个缺陷轮廓矩形度：" << g_dConrect << endl;
			cout << "第" << s << "个缺陷轮廓高宽比：" << g_dhw << endl;

			char Str[20] = { 0 };
			char Str1[20] = { 0 };
			char Str2[20] = { 0 };
			char Str3[20] = { 0 };
			char Str4[20] = { 0 };
			char location[20] = { 0 };
			sprintf(Str, "%u", i);
			sprintf(location, "C=%0.2f", total);//缺陷轮廓定位
			sprintf(Str1, "B=%0.2f", g_dConArea);//缺陷最小外接凸边形轮廓的面积
			sprintf(Str2, "D=%0.2f", g_dhw);//缺陷最小外接矩形轮廓高宽比
			sprintf(Str3, "E=%0.2f", factor);//缺陷轮廓的圆形度
			sprintf(Str4, "F=%0.2f", g_dConrect);//缺陷轮廓的矩形度
			putText(src, Str, Point(roRect[i].center.x + 30, roRect[i].center.y - 20), CV_FONT_HERSHEY_COMPLEX_SMALL, 1, Scalar(64, 64, 255), 0.5, 8);
			putText(src, Str1, Point(roRect[i].center.x + 30, roRect[i].center.y), CV_FONT_HERSHEY_COMPLEX_SMALL, 0.85, Scalar(0, 255, 0));
			putText(src, location, Point(roRect[i].center.x + 30, roRect[i].center.y + 20), CV_FONT_HERSHEY_COMPLEX_SMALL, 0.85, Scalar(0, 255, 0));
			putText(src, Str2, Point(roRect[i].center.x + 30, roRect[i].center.y + 40), CV_FONT_HERSHEY_COMPLEX_SMALL, 0.85, Scalar(0, 255, 0));
			putText(src, Str3, Point(roRect[i].center.x + 30, roRect[i].center.y + 60), CV_FONT_HERSHEY_COMPLEX_SMALL, 1, Scalar(0, 255, 0), 0.5, 8);
			putText(src, Str4, Point(roRect[i].center.x + 30, roRect[i].center.y + 80), CV_FONT_HERSHEY_COMPLEX_SMALL, 1, Scalar(0, 255, 0), 0.5, 8);

			if (g_dhw > 5 || g_dhw < 0.2 && factor < 0.3 && g_dConrect<0.3)
				putText(src, "A:Crack", Point(roRect[i].center.x + 50, roRect[i].center.y - 20), CV_FONT_HERSHEY_COMPLEX_SMALL, 0.85, Scalar(0, 255, 0));//裂纹
			else
				putText(src, "A:Scar", Point(roRect[i].center.x + 50, roRect[i].center.y - 20), CV_FONT_HERSHEY_COMPLEX_SMALL, 0.85, Scalar(0, 255, 0));//疤痕

		}
	}
	return src;

}

int main()
{

	Mat image;//mat类的对象

	Directory dir;
	string path;
	//string path1 = "F:/VS/铁轨检测/钢轨缺陷检测/钢轨缺陷检测/image2/";  // folder path
	//string path1 = "C:/Users/HP/Desktop/PIC/";  // folder path
	string path1 = "D:\\钢轨图片\\PIC\\";  // folder path
	string exten1 = "*.bmp";   //  scan all files with "*"/ 用“*”扫描所有文件
	bool addPath1 = false;
	vector<string> filenames = dir.GetListFiles(path1, exten1, addPath1);
	cout << "The number of files in " << path1 << " :";
	cout << filenames.size() << "\n";
	for (int i = 0; i < filenames.size(); i++)
	{
		path = path1 + filenames[i]; // add two strings
		cout << path1 << endl;
		cout << filenames[i] << endl;
		cout << path << endl;
		Mat img = imread(path);
		cout << img.rows << "行数and列数" << img.cols << endl;

		Mat image = horizontalProjectionMat(img);
		imshow("222", image);

		int k = 1024;
		Mat src = Contour(image, i, k);
		imshow("333", src);
		/*Mat src = Contour(image);*/

		//string Img_Name = "F:\\VS\\铁轨检测\\钢轨缺陷检测\\钢轨缺陷检测\\out\\" + to_string(i) + ".bmp";
		string Img_Name = "C:\\Users\\HP\\Desktop\\out11\\" + to_string(i) + ".bmp";
		imwrite(Img_Name, src);
	}
	waitKey(0);
	return 0;
}
```
