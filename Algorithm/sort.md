<!--
 * @Author: your name
 * @Date: 2020-06-20 18:59:52
 * @LastEditTime: 2020-06-20 22:16:55
 * @LastEditors: Please set LastEditors
 * @Description: In User Settings Edit
 * @FilePath: \undefinedc:\Users\conan\Desktop\LongTime\StupidBirdFliesFirst\Algorithm\sort.md
--> 
# 排序算法
![](sort.jpg)
排序算法一直是所有考察的重点，总计九种：

## 冒泡排序
冒泡排序从小到大排序：一开始交换的区间为0~N-1，将第1个数和第2个数进行比较，前面大于后面，交换两个数，否则不交换。再比较第2个数和第三个数，前面大于后面，交换两个数否则不交换。依次进行，最大的数会放在数组最后的位置。然后将范围变为0~N-2，数组第二大的数会放在数组倒数第二的位置。依次进行整个交换过程，最后范围只剩一个数时数组即为有序。

![](sort.gif)

```c++
void BubbleSort(int array[], int n)
{
    int i, j, k;
    for(i=0; i<n-1; i++)
        for(j=0; j<n-1-i; j++)
        {
            if(array[j]>array[j+1])
            {
                k=array[j];
                array[j]=array[j+1];
                array[j+1]=k;
            }
        }
}
```

## 直接选择排序
选择排序从小到大排序：一开始从0~n-1区间上选择一个最小值，将其放在位置0上，然后在1~n-1范围上选取最小值放在位置1上。重复过程直到剩下最后一个元素，数组即为有序。

```c++
void selectSort(int array[], int n)
{
    int i, j ,min ,k;
    for( i=0; i<n-1; i++)
    {
        min=i; //每趟排序最小值先等于第一个数，遍历剩下的数
        for( j=i+1; j<n; j++) //从i下一个数开始检查
        {
            if(array[min]>array[j])
            {
                min=j;
            }
        }
        if(min!=i)
        {
            k=array[min];
            array[min]=array[i];
            array[i]=k;
        }
    }
}
```

## 直接插入排序
插入排序从小到大排序：首先位置1上的数和位置0上的数进行比较，如果位置1上的数大于位置0上的数，将位置0上的数向后移一位，将1插入到0位置，否则不处理。位置k上的数和之前的数依次进行比较，如果位置K上的数更大，将之前的数向后移位，最后将位置k上的数插入不满足条件点，反之不处理。

![](sort2.gif)

```c++
//a[]为待排序数组，n为数组长度
void InsertSort(int a[], int n)
{
    for (int j = 1; j < n; j++)
    {
        int key = a[j]; //待排序第一个元素
        int i = j - 1;  //代表已经排过序的元素最后一个索引数
        while (i >= 0 && key < a[i])
        {
            //从后向前逐个比较已经排序过数组，如果比它小，则把后者用前者代替，
            //其实说白了就是数组逐个后移动一位,为找到合适的位置时候便于Key的插入
            a[i + 1] = a[i];
            i--;
        }
        a[i + 1] = key;//找到合适的位置了，赋值,在i索引的后面设置key值。
    }
}
```

## 快速排序
快速排序从小到大排序：在数组中随机选一个数（默认数组首个元素），数组中小于等于此数的放在左边部分，大于此数的放在右边部分，这个操作确保了这个数是处于正确位置的，再对左边部分数组和右边部分数组递归调用快速排序，重复这个过程。

![](sort3.gif)

```c++
void quicksort(int a[], int left, int right) {
    int i, j, t, privotkey;
    if (left > right)   //（递归过程先写结束条件）
        return;

    privotkey = a[left]; //temp中存的就是基准数（枢轴）
    i = left;
    j = right;
    while (i < j) {
        //顺序很重要，要先从右边开始找（最后交换基准时换过去的数要保证比基准小，因为基准选取数组第一个数）
        while (a[j] >= privotkey && i < j) {
            j--;
        }
        a[i] = a[j];
        //再找左边的
        while (a[i] <= privotkey && i < j) {
            i++;
        }
        a[j] = a[i];
    }
    //最终将基准数归位
    a[i] = privotkey;

    quicksort(a, left, i - 1);//继续处理左边的，这里是一个递归的过程
    quicksort(a, i + 1, right);//继续处理右边的 ，这里是一个递归的过程
}

int main()
{
	int array[]={34,65,12,43,67,5,78,10,3,70},k;
	int len=sizeof(array)/sizeof(int);
	cout<<"The orginal arrayare:"<<endl;
	for(k=0;k<len;k++)
		cout<<array[k]<<",";
	cout<<endl;
	quickSort(array,0,len-1);
	cout<<"The sorted arrayare:"<<endl;
	for(k=0;k<len;k++)
		cout<<array[k]<<",";
	cout<<endl;
	system("pause");
	return 0;
}
```