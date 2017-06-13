本文介绍一种使用c++实现简单的（变量数小于10个）单纯形法的方法。

作为一个c++的初学者，我对编程也只是了解一些皮毛而已，所以写这篇文章的目的也在于和大家一起共同学习。

我们考虑一个简单的线性规划问题：
某工厂生产桌子、椅子和柜子。生产这三种产品需要木材以及两种不同的工序：木工和打磨。下表列出了三种产品的所需要的木材和工序。

|         | 柜子          | 桌子 | 椅子 |
| ：-----------： |:-------------:| ：-----:|:------:|
| 木材     | 8 尺 | 6尺 | 1尺 |
| 打磨      | 4小时     |   2小时 | 1.5小时|
| 木工 | 2小时      |    1.5小时 |0.5小时|

现在工厂库存有48尺的木材，打磨和木工的工人工作时间上限分别为20小时和8小时。柜子、桌子和椅子的售价分别为600元，300元和200元。

我们假设x1,x2,x3分别为三种产品的产量。根据上述问题我们可以建立一个线性规划问题：

![](https://latex.codecogs.com/gif.latex?Max%5C600x_1%20&plus;%20300x_2%20&plus;%20200x_3%20%5C%5C%20s.t.%5C%5C%208x_1&plus;%206x_2%20&plus;%20x_3%20%5Cleq%2048%5C%5C%20%5C4%20x_1%20&plus;%202%20x_2%20&plus;%201.5%20x_3%20%5Cleq%2020%20%5C%5C%20%5C2%20x_1%20&plus;%201.5%20x_2%20&plus;%200.5x_3%20%5Cleq%208%20%5C%5C%20x_1%2Cx_2%2Cx_3%5Cgeq%200)

我们使用c++的Sample Run，最后结果最优解为x1=1，x3=8，最后的收益为2800元。

```
 LINEAR PROGRAMMING

 MAXIMIZE (Y/N) ? Y

 NUMBER OF VARIABLES OF ECONOMIC FUNCTION ? 3

 NUMBER OF CONSTRAINTS ? 3

 INPUT COEFFICIENTS OF ECONOMIC FUNCTION:
       #1 ? 600
       #2 ? 300
       #3 ? 200
       Right hand side ? 0

 CONSTRAINT #1:
       #1 ? 8
       #2 ? 6
       #3 ? 1
       Right hand side ? 48

 CONSTRAINT #2:
       #1 ? 4
       #2 ? 2
       #3 ? 1.5
       Right hand side ? 20

 CONSTRAINT #3:
       #1 ? 2
       #2 ? 1.5
       #3 ? 0.5
       Right hand side ? 8


 RESULTS:

       VARIABLE #1: 2.000000
       VARIABLE #3: 8.000000

       ECONOMIC FUNCTION: 2800.000000

logout
Saving session...
...copying shared history...
...saving history...truncating history files...
...completed.

[Process completed]

```

以下是c++代码。首先将变量的系数存入数组中：
```cpp
#include <stdio.h>
#include <math.h>

#define CMAX  10  //变量数的上限
#define VMAX  10  //约束条件的上限


  int NC, NV, NOPTIMAL,P1,P2,XERR;
  double TS[CMAX][VMAX];

  void Data() {
    double R1,R2;
    char R;
    int I,J;
    printf("\n LINEAR PROGRAMMING\n\n");
    printf(" MAXIMIZE (Y/N) ? "); scanf("%c", &R);
    printf("\n NUMBER OF VARIABLES OF ECONOMIC FUNCTION ? "); scanf("%d", &NV);
    printf("\n NUMBER OF CONSTRAINTS ? "); scanf("%d", &NC);
    if (R == 'Y' || R=='y')
      R1 = 1.0;
    else
      R1 = -1.0;
    printf("\n INPUT COEFFICIENTS OF ECONOMIC FUNCTION:\n");
    for (J = 1; J<=NV; J++) {
      printf("       #%d ? ", J); scanf("%lf", &R2);
      TS[1][J+1] = R2 * R1;//如果是min问题，则目标函数所有的系数乘-1，变为max问题
    }
    printf("       Right hand side ? "); scanf("%lf", &R2);
    TS[1][1] = R2 * R1;//同上
    for (I = 1; I<=NC; I++) {
      printf("\n CONSTRAINT #%d:\n", I);
      for (J = 1; J<=NV; J++) {
        printf("       #%d ? ", J); scanf("%lf", &R2);
        TS[I + 1][J + 1] = -R2;
      }
      printf("       Right hand side ? "); scanf("%lf", &TS[I+1][1]);
    }
    printf("\n\n RESULTS:\n\n");
    for(J=1; J<=NV; J++)  TS[0][J+1] = J;
    for(I=NV+1; I<=NV+NC; I++)  TS[I-NV+1][0] = I;
  }
```
所有的系数已存入数组中，接下去定义三个函数:
```cpp
  void Pivot();//找到每次迭代的pivot
  void Formula();//对矩阵进行变换
  void Optimize();//每次迭代后判断是否已达最优解

  void Simplex() {
e10: Pivot();
     Formula();
     Optimize();
     if (NOPTIMAL == 1) goto e10;
  }

  void Pivot() {

    double RAP,V,XMAX;
    int I,J;

    XMAX = 0.0;
    for(J=2; J<=NV+1; J++) {
    if (TS[1][J] > 0.0 && TS[1][J] > XMAX) {
        XMAX = TS[1][J];
        P2 = J;
      }//找到目标函数中最大的非负系数
    }
    RAP = 999999.0;
    for (I=2; I<=NC+1; I++) {
      if (TS[I][P2] >= 0.0) goto e10;
      V = fabs(TS[I][1] / TS[I][P2]);
      if (V < RAP) {
        RAP = V;
        P1 = I;
      }//找到比例最大的行
e10:;}
    V = TS[0][P2]; TS[0][P2] = TS[P1][0]; TS[P1][0] = V;
  }

  void Formula() {;
    //Labels: e60,e70,e100,e110;
    int I,J;

    for (I=1; I<=NC+1; I++) {
      if (I == P1) goto e70;//跳过pivot所在的行
      for (J=1; J<=NV+1; J++) {
        if (J == P2) goto e60;//跳过pivot所在的列
        TS[I][J] -= TS[P1][J] * TS[I][P2] / TS[P1][P2];//进行初等矩阵行变换
e60:;}
e70:;}
    TS[P1][P2] = 1.0 / TS[P1][P2];
    for (J=1; J<=NV+1; J++) {
      if (J == P2) goto e100;
      TS[P1][J] *= fabs(TS[P1][P2]);
e100:;}
    for (I=1; I<=NC+1; I++) {
      if (I == P1) goto e110;
      TS[I][P2] *= TS[P1][P2];
e110:;}
  }   

  void Optimize() {
    int I,J;
    for (I=2; I<=NC+1; I++)
      if (TS[I][1] < 0.0)  XERR = 1;
    NOPTIMAL = 0;
    if (XERR == 1)  return;// 如果所有的系数为负数，则没有可行解
    for (J=2; J<=NV+1; J++)
      if (TS[1][J] > 0.0)  NOPTIMAL = 1;//如果目标函数所有的系数为非负，则已达到最优解
  }
```
最后输出结果：
```cpp
  void Results() {
    //Labels: e30,e70,e100;
    int I,J;
    
    if (XERR == 0) goto e30;
    printf(" NO SOLUTION.\n"); goto e100;
e30:for (I=1; I<=NV; I++)
    for (J=2; J<=NC+1; J++) {
      if (TS[J][0] != 1.0*I) goto e70;
      printf("       VARIABLE #%d: %f\n", I, TS[J][1]);
e70:  ;}
    printf("\n       ECONOMIC FUNCTION: %f\n", TS[1][1]);
e100:printf("\n");
  }

int main()  {

  Data();
  Simplex();
  Results();

}

//end of file simplex.cpp
```
