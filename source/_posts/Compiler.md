---
title: "RCC: Retarded C-like Compiler"
date: 2021-11-17 16:29:37
tags: C++

---
Course project for Compiler Principle at ZJU. Co-Authored with @[Tinghao(Vitus) Xie](http://vtu.life/posts/2020/06/RCC/).[**Visit Github repo.**](https://github.com/Luke-Skycrawler/rcc/)
```C++
int main(){
    int k = 3;
    for(i: 3 to 5){
        printf("i =%d\n", i);
    }
}
```
<!-- more -->

### Features
- [x] Advanced self-defined types (nested struct and arrays)
- [x] MACRO support (#define/#define f(X),#ifdef/ifndef and nested)
- [ ] Error detection and recovery (primary)

### Language definitions

This language is a miniature of C. We highlight our differences as follows:
- type system: char, int, double and n-dimensional array type; Pointer type is not supported in this version.
- no controled jumps, gotos and labels , i.e. break, continue and switch statements are not supported.
- `scanf` and `printf` are automaticly embedded in runtime. Do not override it.
- calling convention of scanf is modified. e.g. you shall use `scanf("%d",i);` to read the value into variable i and drop the & symbol.
- `for` loop switched to pascal-like, i.e. `for(i: 0 to n){}` and `for(i: n downto 0){}` snippet. `i` is only seen within the scope of this loop.
- unary operators are not supported in this version.

You are encouraged to peruse the test examples in order to get a better understanding of the gramma. Here is a quicksort implementation in our language:
```C++

int a[10005];
int sort(int left, int right)
{
    int i = left;
    int j = right;
    int key = a[left],tmp;

    if(left >= right)
    {
        return 0;
    }
    while(i < j)
    {
        while(i < j & key <= a[j])
        {
            j=j - 1;
        }
        tmp=a[i];
        a[i]=a[j];
        a[j]=tmp;

        while(i < j & key >= a[i])
        {
            i=i + 1;
        }
        tmp=a[i];
        a[i]=a[j];
        a[j]=tmp;
    }
    a[i] = key;
    sort(left, i - 1);
    sort(i + 1, right);
    return 0;
}

int main(){
    int k,n;
    scanf("%d",n);
    for(k:0 to n-1){
        scanf("%d",a[k]);
    }
    sort(0,n-1);
    for(k:0 to n-1){
        printf("%d\n",a[k]);
    }
    return 0;
}


```

