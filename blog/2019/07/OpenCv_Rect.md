### OpenCV的Rect矩形类简介和使用


[摘自CSDN CAUC康辉](https://blog.csdn.net/kh1445291129/article/details/51149849)

```java

//如果创建一个Rect对象rect(100, 50, 50, 100)，那么rect会有以下几个功能：
rect.area();     //返回rect的面积 5000
rect.size();     //返回rect的尺寸 [50 × 100]
rect.tl();       //返回rect的左上顶点的坐标 [100, 50]
rect.br();       //返回rect的右下顶点的坐标 [150, 150]
rect.width();    //返回rect的宽度 50
rect.height();   //返回rect的高度 100
rect.contains(Point(x, y));  //返回布尔变量，判断rect是否包含Point(x, y)点
 
//还可以求两个矩形的交集和并集
rect = rect1 & rect2;
rect = rect1 | rect2;
 
//还可以对矩形进行平移和缩放  
rect = rect + Point(-100, 100);	//平移，也就是左上顶点的x坐标-100，y坐标+100
rect = rect + Size(-100, 100);	//缩放，左上顶点不变，宽度-100，高度+100
 
//还可以对矩形进行对比，返回布尔变量
rect1 == rect2;
rect1 != rect2;
 
//OpenCV里貌似没有判断rect1是否在rect2里面的功能，所以自己写一个吧
bool isInside(Rect rect1, Rect rect2)
{
	return (rect1 == (rect1&rect2));
}
 
//OpenCV貌似也没有获取矩形中心点的功能，还是自己写一个
Point getCenterPoint(Rect rect)
{
	Point cpt;
	cpt.x = rect.x + cvRound(rect.width/2.0);
	cpt.y = rect.y + cvRound(rect.height/2.0);
	return cpt;
}
 
//围绕矩形中心缩放
Rect rectCenterScale(Rect rect, Size size)
{
	rect = rect + size;	
	Point pt;
	pt.x = cvRound(size.width/2.0);
	pt.y = cvRound(size.height/2.0);
	return (rect-pt);
}

```
 