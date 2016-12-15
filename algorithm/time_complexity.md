### 时间复杂度 Time complexity

#### 计算过程
1. 找到执行次数最多的语句
2. 计算语句执行次数的数量级
3. 用大O来表示结果

例子1：
```Java
public void printsum(int count){
    int sum = 1;
    for(int i= 0; i<n; i++){
       sum += i;
    }   
    System.out.print(sum);
}
```
执行次数最多的往往就是循环体类的语句，这里很明显是执行n次，那么时间复杂度是O(n)

例子2:
```Java
int i= 1;
while(i<n){
    i = i*2; // 2,4,8,16 ... 
}
```
2的t的次方<n 是执行下去的条件，t为循环次数，那么t=log2 n  2是对数底，那么时间复杂度是O(log2 n)

例子3:
```Java
int num=0;
for(int i=0;i<n;i++){
    for(int j=0;j<n;j++){
        num++;
    }
}
```
双重循环，供执行n*n次，那么时间复杂度是O(n的二次方)

参考：
- 算法时间复杂度的计算 [整理] http://univasity.iteye.com/blog/1164707
- 算法时间复杂度计算 http://www.jianshu.com/p/99bac69fdd97