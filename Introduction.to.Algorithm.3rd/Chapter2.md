Exercise 2.2-2

SELECTION-SORT(A)

```
n=A.lengeth
for j=1 to n-1
    smallest=j
    for i=j+1 to n
        if A[i]<A[smallest]
            smallest=i
    exchange A[j] with A[smallest]
```

```
#include <iostream>

using namespace std;

void select_sort(int a[], int n)
{
	int i, j, min, temp;

	for (i = 0; i < n - 1; i++)
	{
		min = i;

		for (j = i + 1; j < n; j++)
		{
			if (a[j] < a[min])
				min = j;
		}

		if (min != j)
		{
			temp = a[i];
			a[i] = a[min];
			a[min] = temp;
		}
	}
}
```

Exercise 2.3-2

```
#include <stdio.h>

void merge(int A[], int p, int q,int r)
{
    int i,j,k;

    int n1=q-p+1;
    int n2=r-q;

    int L[n1];
    int R[n2];

    for(i=0;i<n1;i++)
        L[i]=A[p];
    for(j=0;j<n2;j++)
        R[j]=A[q+j+1];

    for(i=0,j=0,k=p;k<=r;k++)
    {
        if(i==n1)
        {
            A[k]=R[j++];
        }
        else if(j==n2)
        {
            A[k]=L[i++];
        }
        else if(L[i]<=R[j])
        {
            A[k]=L[i++];
        }
        else
        {
            A[k]=R[j++];
        }
    }
}

void merge_sort(int A[], int p,int r)
{
    if(p<r)
    {
        int q=(p+r)/2;
        merge_sort(A,p,q);
        merge_sort(A,q+1,r);
        merge(A,p,q,r);
    }
}
```
