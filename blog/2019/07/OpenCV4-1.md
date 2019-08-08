# Opencv 4.1
获取不同通道图层显示：
、、、java
int main(){
    Mat channels[3];
    Mat imageB;
    Mat imageG;
    Mat imageR;

    Mat srcMat=cv::imread("/home/jayqiu/Downloads/test.jpg");
    if(!srcMat.data ) {
        printf("你妹，读取srcImage1错误~！ \n");
        return false;
    }
    split(srcMat,channels);
    Mat gray;
//    cvtColor(srcMat,gray,COLOR_BGR2GRAY);
    imshow("BRG",srcMat);
    imageB = channels[0];
    imageG = channels[1];
    imageR = channels[2];

    imshow("B", imageB);
    imshow("G", imageG);
    imshow("R", imageR);


    waitKey(0);
    return 0;
}
、、、
