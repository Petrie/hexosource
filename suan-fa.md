# 算法

## POI相关

[http://www.cnblogs.com/LBSer/p/4471742.html](http://www.cnblogs.com/LBSer/p/4471742.html)

## 排序算法

### 插入排序

#### 直接插入排序

```c
void print(int a[], int n ,int i){  
    cout<<i <<":";  
    for(int j= 0; j<8; j++){  
        cout<<a[j] <<" ";  
    }  
    cout<<endl;  
}  

void InsertSort(int arr[], int len)
{
    int i,j,temp;

    for (i = 1 ; i < len ; i++ )
    {
        temp = arr[i];

        for (j = i; j > 0 && arr[j-1] > temp; j--)
        {
            arr[j] = arr[j-1];
        }

        arr[j] = temp;
    }
}

int main(){  
    int a[8] = {3,1,5,7,2,4,9,6};  
    InsertSort(a,8);  
    print(a,8,8);  
}
```

#### 希尔排序

```c
void print(int a[], int n ,int i){  
    cout<<i <<":";  
    for(int j= 0; j<8; j++){  
        cout<<a[j] <<" ";  
    }  
    cout<<endl;  
}  
/** 
 * 直接插入排序的一般形式 
 * 
 * @param int dk 缩小增量，如果是直接插入排序，dk=1 
 * 
 */  

void ShellInsertSort(int a[], int n, int dk)  
{  
    for(int i= dk; i<n; ++i){  
        if(a[i] < a[i-dk]){          //若第i个元素大于i-1元素，直接插入。小于的话，移动有序表后插入  
            int j = i-dk;     
            int x = a[i];           //复制为哨兵，即存储待排序元素  
            a[i] = a[i-dk];         //首先后移一个元素  
            while(x < a[j]){     //查找在有序表的插入位置  
                a[j+dk] = a[j];  
                j -= dk;             //元素后移  
            }  
            a[j+dk] = x;            //插入到正确位置  
        }  
        print(a, n,i );  
    }  

}  

/** 
 * 先按增量d（n/2,n为要排序数的个数进行希尔排序 
 * 
 */  
void shellSort(int a[], int n){  

    int dk = n/2;  
    while( dk >= 1  ){  
        ShellInsertSort(a, n, dk);  
        dk = dk/2;  
    }  
}  
int main(){  
    int a[8] = {3,1,5,7,2,4,9,6};  
    //ShellInsertSort(a,8,1); //直接插入排序  
    shellSort(a,8);           //希尔插入排序  
    print(a,8,8);  
}
```

### 选择排序

#### 简单选择排序

```c
void print(int a[], int n ,int i){  
    cout<<"第"<<i+1 <<"趟 : ";  
    for(int j= 0; j<8; j++){  
        cout<<a[j] <<"  ";  
    }  
    cout<<endl;  
}  
/** 
 * 数组的最小值 
 * 
 * @return int 数组的键值 
 */  
int SelectMinKey(int a[], int n, int i)  
{  
    int k = i;  
    for(int j=i+1 ;j< n; ++j) {  
        if(a[k] > a[j]) k = j;  
    }  
    return k;  
}  

/** 
 * 选择排序 
 * 
 */  
void selectSort(int a[], int n){  
    int key, tmp;  
    for(int i = 0; i< n; ++i) {  
        key = SelectMinKey(a, n,i);           //选择最小的元素  
        if(key != i){  
            tmp = a[i];  a[i] = a[key]; a[key] = tmp; //最小元素与第i位置元素互换  
        }  
        print(a,  n , i);  
    }  
}  
int main(){  
    int a[8] = {3,1,5,7,2,4,9,6};  
    cout<<"初始值：";  
    for(int j= 0; j<8; j++){  
        cout<<a[j] <<"  ";  
    }  
    cout<<endl<<endl;  
    selectSort(a, 8);  
    print(a,8,8);  
}
```

#### 简单选择排序的改进版：二元选择排序

```c
void SelectSort(int r[],int n) {  
    int i ,j , min ,max, tmp;  
    for (i=1 ;i <= n/2;i++) {    
        // 做不超过n/2趟选择排序   
        min = i; max = i ; //分别记录最大和最小关键字记录位置  
        for (j= i+1; j<= n-i; j++) {  
            if (r[j] > r[max]) {   
                max = j ; continue ;   
            }    
            if (r[j]< r[min]) {   
                min = j ;   
            }     
      }    
      //该交换操作还可分情况讨论以提高效率  
      tmp = r[i-1]; r[i-1] = r[min]; r[min] = tmp;  
      tmp = r[n-i]; r[n-i] = r[max]; r[max] = tmp;   

    }   
}
```

#### 堆排序

[http://bubkoo.com/2014/01/14/sort-algorithm/heap-sort/](http://bubkoo.com/2014/01/14/sort-algorithm/heap-sort/)

### 交换排序

#### 冒泡排序

```c
void bubbleSort(int a[], int n){  
    for(int i =0 ; i< n-1; ++i) {  
        for(int j = 0; j < n-i-1; ++j) {  
            if(a[j] > a[j+1])  
            {  
                int tmp = a[j] ; a[j] = a[j+1] ;  a[j+1] = tmp;  
            }  
        }  
    }  
}
```

#### 冒泡排序算法的改进

```c
void Bubble_1 ( int r[], int n) {  
    int i= n -1;  //初始时,最后位置保持不变  
    while ( i> 0) {   
        int pos= 0; //每趟开始时,无记录交换  
        for (int j= 0; j< i; j++) { 
            if (r[j]> r[j+1]) {  
                pos= j; //记录交换的位置   
                int tmp = r[j]; r[j]=r[j+1];r[j+1]=tmp;  
            }   
        }
        i= pos; //为下一趟排序作准备  
     }   
}
```

#### 改进二

```c
void Bubble_2 ( int r[], int n){  
    int low = 0;   
    int high= n -1; //设置变量的初始值  
    int tmp,j;  
    while (low < high) {  
        for (j= low; j< high; ++j) //正向冒泡,找到最大者  
            if (r[j]> r[j+1]) {  
                tmp = r[j]; r[j]=r[j+1];r[j+1]=tmp;  
            }   
        --high;                 //修改high值, 前移一位  
        for ( j=high; j>low; --j) //反向冒泡,找到最小者  
            if (r[j]<r[j-1]) {  
                tmp = r[j]; r[j]=r[j-1];r[j-1]=tmp;  
            }  
        ++low;                  //修改low值,后移一位  
    }   
}
```

#### 快速排序

```c
void print(int a[], int n){  
    for(int j= 0; j<n; j++){  
        cout<<a[j] <<"  ";  
    }  
    cout<<endl;  
}  

void swap(int *a, int *b)  
{  
    int tmp = *a;  
    *a = *b;  
    *b = tmp;  
}  

int partition(int a[], int low, int high)  
{  
    int privotKey = a[low];                             //基准元素  
    while(low < high){                                   //从表的两端交替地向中间扫描  
        while(low < high  && a[high] >= privotKey) --high;  //从high 所指位置向前搜索，至多到low+1 位置。将比基准元素小的交换到低端  
        swap(&a[low], &a[high]);  
        while(low < high  && a[low] <= privotKey ) ++low;  
        swap(&a[low], &a[high]);  
    }  
    print(a,10);  
    return low;  
}  


void quickSort(int a[], int low, int high){  
    if(low < high){  
        int privotLoc = partition(a,  low,  high);  //将表一分为二  
        quickSort(a,  low,  privotLoc -1);          //递归对低子表递归排序  
        quickSort(a,   privotLoc + 1, high);        //递归对高子表递归排序  
    }  
}  

int main(){  
    int a[10] = {3,1,5,7,2,4,9,6,10,8};  
    cout<<"初始值：";  
    print(a,10);  
    quickSort(a,0,9);  
    cout<<"结果：";  
    print(a,10);  

}
```

快速排序改进

```c
void print(int a[], int n){  
    for(int j= 0; j<n; j++){  
        cout<<a[j] <<"  ";  
    }  
    cout<<endl;  
}  

void swap(int *a, int *b)  
{  
    int tmp = *a;  
    *a = *b;  
    *b = tmp;  
}  

int partition(int a[], int low, int high)  
{  
    int privotKey = a[low];                 //基准元素  
    while(low < high){                   //从表的两端交替地向中间扫描  
        while(low < high  && a[high] >= privotKey) --high; //从high 所指位置向前搜索，至多到low+1 位置。将比基准元素小的交换到低端  
        swap(&a[low], &a[high]);  
        while(low < high  && a[low] <= privotKey ) ++low;  
        swap(&a[low], &a[high]);  
    }  
    print(a,10);  
    return low;  
}  


void qsort_improve(int r[ ],int low,int high, int k){  
    if( high -low > k ) { //长度大于k时递归, k为指定的数  
        int pivot = partition(r, low, high); // 调用的Partition算法保持不变  
        qsort_improve(r, low, pivot - 1,k);  
        qsort_improve(r, pivot + 1, high,k);  
    }   
}   
void quickSort(int r[], int n, int k){  
    qsort_improve(r,0,n,k);//先调用改进算法Qsort使之基本有序  

    //再用插入排序对基本有序序列排序  
    for(int i=1; i<=n;i ++){  
        int tmp = r[i];   
        int j=i-1;  
        while(tmp < r[j]){  
            r[j+1]=r[j]; j=j-1;   
        }  
        r[j+1] = tmp;  
    }   

}   



int main(){  
    int a[10] = {3,1,5,7,2,4,9,6,10,8};  
    cout<<"初始值：";  
    print(a,10);  
    quickSort(a,9,4);  
    cout<<"结果：";  
    print(a,10);  

}
```

### 归并排序

* 迭代版

  ```c
  int min(int x, int y) {
      return x < y ? x : y;
  }
  void merge_sort(int arr[], int len) {
      int* a = arr;
      int* b = (int*) malloc(len * sizeof(int));
      int seg, start;
      for (seg = 1; seg < len; seg += seg) {
          for (start = 0; start < len; start += seg + seg) {
              int low = start, mid = min(start + seg, len), high = min(start + seg + seg, len);
              int k = low;
              int start1 = low, end1 = mid;
              int start2 = mid, end2 = high;
              while (start1 < end1 && start2 < end2)
                  b[k++] = a[start1] < a[start2] ? a[start1++] : a[start2++];
              while (start1 < end1)
                  b[k++] = a[start1++];
              while (start2 < end2)
                  b[k++] = a[start2++];
          }
          int* temp = a;
          a = b;
          b = temp;
      }
      if (a != arr) {
          int i;
          for (i = 0; i < len; i++)
              b[i] = a[i];
          b = a;
      }
      free(b);
  }
  ```

* 递归版

  ```c
  void merge_sort_recursive(int arr[], int reg[], int start, int end) {
      if (start >= end)
          return;
      int len = end - start, mid = (len >> 1) + start;
      int start1 = start, end1 = mid;
      int start2 = mid + 1, end2 = end;
      merge_sort_recursive(arr, reg, start1, end1);
      merge_sort_recursive(arr, reg, start2, end2);
      int k = start;
      while (start1 <= end1 && start2 <= end2)
          reg[k++] = arr[start1] < arr[start2] ? arr[start1++] : arr[start2++];
      while (start1 <= end1)
          reg[k++] = arr[start1++];
      while (start2 <= end2)
          reg[k++] = arr[start2++];
      for (k = start; k <= end; k++)
          arr[k] = reg[k];
  }
  void merge_sort(int arr[], const int len) {
      int reg[len];
      merge_sort_recursive(arr, reg, 0, len - 1);
  }
  ```

  ​

### 基数排序

## 查询算法

### 顺序查找

```c
//顺序查找
int SequenceSearch(int a[], int value, int n)
{
    int i;
    for(i=0; i<n; i++)
        if(a[i]==value)
            return i;
    return -1;
}
```

### 二分查找

```c
//二分查找（折半查找），版本1
int BinarySearch1(int a[], int value, int n)
{
    int low, high, mid;
    low = 0;
    high = n-1;
    while(low<=high)
    {
        mid = (low+high)/2;
        if(a[mid]==value)
            return mid;
        if(a[mid]>value)
            high = mid-1;
        if(a[mid]<value)
            low = mid+1;
    }
    return -1;
}

//二分查找，递归版本
int BinarySearch2(int a[], int value, int low, int high)
{
    int mid = low+(high-low)/2;
    if(a[mid]==value)
        return mid;
    if(a[mid]>value)
        return BinarySearch2(a, value, low, mid-1);
    if(a[mid]<value)
        return BinarySearch2(a, value, mid+1, high);
}
```

### 二叉查找树

**二叉查找树**（BinarySearch Tree，也叫二叉搜索树，或称二叉排序树Binary Sort Tree）或者是一棵空树，或者是具有下列性质的二叉树：

* 若任意节点的左子树不空，则左子树上所有结点的值均小于它的根结点的值；
* 若任意节点的右子树不空，则右子树上所有结点的值均大于它的根结点的值；
* 任意节点的左、右子树也分别为二叉查找树。

**二叉查找树性质**：**对二叉查找树进行中序遍历，即可得到有序的数列。**

### 2-3查找树

**2-3查找树定义**：和二叉树不一样，2-3树运行每个节点保存1个或者两个的值。对于普通的2节点\(2-node\)，他保存1个key和左右两个子节点。对应3节点\(3-node\)，保存两个Key，2-3查找树的定义如下：

* 要么为空，要么：
* 对于2节点，该节点保存一个key及对应value，以及两个指向左右节点的节点，左节点也是一个2-3节点，所有的值都比key要小，右节点也是一个2-3节点，所有的值比key要大。
* 对于3节点，该节点保存两个key及对应value，以及三个指向左中右的节点。左节点也是一个2-3节点，所有的值均比两个key中的最小的key还要小；中间节点也是一个2-3节点，中间节点的key值在两个跟节点key值之间；右节点也是一个2-3节点，节点的所有key值比两个key中的最大的key还要大。

### 红黑树

2-3查找树能保证在插入元素之后能保持树的平衡状态，最坏情况下即所有的子节点都是2-node，树的高度为lgn，从而保证了最坏情况下的时间复杂度。但是2-3树实现起来比较复杂，于是就有了一种简单实现2-3树的数据结构，即红黑树（Red-Black Tree）。

#### 基本思想

红黑树的思想就是对2-3查找树进行编码，尤其是对2-3查找树中的3-nodes节点添加额外的信息。红黑树中将节点之间的链接分为两种不同类型，红色链接，他用来链接两个2-nodes节点来表示一个3-nodes节点。黑色链接用来链接普通的2-3节点。特别的，使用红色链接的两个2-nodes来表示一个3-nodes节点，并且向左倾斜，即一个2-node是另一个2-node的左子节点。这种做法的好处是查找的时候不用做任何修改，和普通的二叉查找树相同。

#### 红黑树的定义：

红黑树是一种具有红色和黑色链接的平衡查找树，同时满足：

* 红色节点向左倾斜
* 一个节点不可能有两个红色链接
* 整个树完全黑色平衡，即从根节点到所以叶子结点的路径上，黑色链接的个数都相同。

#### 应用场景

* 著名的linux进程调度[Completely Fair Scheduler](https://en.wikipedia.org/wiki/Completely_Fair_Scheduler),用红黑树管理进程控制块
* epoll在内核中的实现，用红黑树管理事件块
* nginx中，用红黑树管理timer等
* Java的TreeMap实现

### B树

**B树**可以看作是对2-3查找树的一种扩展，即他允许每个节点有M-1个子节点。

* 根节点至少有两个子节点
* 每个节点有M-1个key，并且以升序排列
* 位于M-1和M key的子节点的值位于M-1 和M key对应的Value之间
* 其它节点至少有M/2个子节点

#### 应用场景

* 文件系统及数据库场景
* ​

### B+树

**B+**树是对B树的一种变形树，它与B树的差异在于：

* 有k个子结点的结点必然有k个关键码；
* 非叶结点仅具有索引作用，跟记录有关的信息均存放在叶结点中。
* 树的所有叶结点构成一个有序链表，可以按照关键码排序的次序遍历全部记录。

**B和B+树的区别在于，B+树的非叶子结点只包含导航信息，不包含实际的值，所有的叶子结点和相连的节点使用链表相连，便于区间查找和遍历。**

**但是B树也有优点，其优点在于，由于B树的每一个节点都包含key和value，因此经常访问的元素可能离根节点更近，因此访问也更迅速。**

### 跳跃表

## 树

* 二叉查找树又称二叉搜索树、有序二叉树（英语：ordered binary tree），排序二叉树
* 平衡树（自平衡二叉查找树）是计算机科学中的一类改进的二叉查找树。
  * [AVL树](https://zh.wikipedia.org/wiki/AVL%E6%A0%91)
  * [红黑树](https://zh.wikipedia.org/wiki/%E7%BA%A2%E9%BB%91%E6%A0%91)
  * [2-3树](https://zh.wikipedia.org/wiki/2-3%E6%A0%91)

## 常见算法

> [http://www.geeksforgeeks.org/top-10-algorithms-in-interview-questions/](http://www.geeksforgeeks.org/top-10-algorithms-in-interview-questions/)

#### 倒排索引

### 斐波那契数

#### 循环迭代方法

```php
<?php
  function fib_interation($n){
    $fib = array();
      if($n < 0){
      return 0;
      }
      for($fib[0] = 0, fib[1] = 1, $i = 2; $i<=$n; $i++){
      $fib[$i] = $fib[$i - 1] + $fib[$i-2];
      }

      return $fib[$n];
  }
```

#### 递归方法

```php
<?php
  function fib_recursive($n){
    if($n < 0) return 0;
      elseif ($n == 1){
      return 1;
      }else{
      return fib_recursive($n-1) + fib_recursive($n-2);
      }
  }
```

