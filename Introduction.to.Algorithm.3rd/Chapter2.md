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
