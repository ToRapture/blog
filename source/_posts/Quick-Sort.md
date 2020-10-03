---
title: Quick Sort
date: 2020-04-13 12:01:00
tags:
- Algorithm
---

## Implementation

```cpp
#include <bits/stdc++.h>
#define DBG(x) cerr << #x << " = " << x << endl

using namespace std;
typedef long long LL;

void qsort(vector<int> &vec, int s, int e) {
    if (s >= e) return;
    int pv = vec[s];
    int l = s, r = e;
    for (int i = s + 1, left = e - s; left > 0; left--) {
        if (vec[i] < pv) {
            vec[l++] = vec[i++];
        } else {
            swap(vec[i], vec[r--]);
        }
    }
    assert(l == r);
    vec[l] = pv;
    qsort(vec, s, l - 1);
    qsort(vec, l + 1, e);
}

int main(int argc, char **argv) {
    srand(time(NULL));
    vector<int> vec(500000);
    for (int i = 0; i < vec.size(); i++) vec[i] = random();
    vector<int> s = vec;

    sort(s.begin(), s.end());
    qsort(vec, 0, vec.size() - 1);

    assert(s == vec);

    return 0;
}
```

Golang version updated at 2020-04-20
```go

type less []int

func (l less) Less(i, j int) bool {
	return l[i] < l[j]
}

func (l less) Swap(i, j int) {
	l[i], l[j] = l[j], l[i]
}

func (l less) Len() int {
	return len(l)
}

func quickSort(data less, s, e int) {
	if s >= e {
		return
	}

	data.Swap(s, rand.Int()%(e-s+1)+s)

	l, r := s, e
	for i, left := s+1, e-s; left > 0; left-- {
		if data.Less(i, l) {
			data.Swap(i, l)
			i++
			l++
		} else {
			data.Swap(i, r)
			r--
		}
	}
	quickSort(data, s, l-1)
	quickSort(data, l+1, e)
}

func sortArray(nums []int) []int {
	rand.Seed(time.Now().Unix())
	data := less(nums)
	quickSort(data, 0, data.Len()-1)
	return nums
}
```

## Proof

### Correctness of partition
$s$ is the first index in the subarray of $vec$, $e$ is the last index in the subarray of $vec$.
$vec[s:e]$ denotes the whole subarray, inclusive.
At the start of the **for** loop:
- $i$ denotes the index of the element to be compared with the pivot
- $left$ denotes the number of remaining elements to be compared
- $l$ denotes the minimum position to put the element which is smaller than the pivot
- $r$ denotes the maximum position to put the element which is greater than or equal to the pivot

We say there are $x$ elements $<$ pivot, then there are supposed to be $e - s - x$ elements $>=$ pivot.
After the **for** loop terminates, $l$ will be $s + x$, and $y$ will be $e - (e - s - x) = s + x$ too, so $l = r$.
The elements on the left side of $l \ or\  r$ are all $<$ pivot, and the elements of the right side of $l \ or \ r$ are all $>=$ pivot, then we put the pivot on the position $l \ or \ r$.
Then all the elements on the left side of the pivot will be smaller than the pivot, and all the elements on the right side of the pivot will be greater than or equal to the pivot.
Thus, the partition is correct.

### Correctness of recursion
Proof by induction.
Suppose that $P(n)$ is a predicate defined on $n \in [1, \infty)$ meaning the $Quick Sort$ is correct on a given array sized $n$.
If we can show that:
1. $P(1)$ is true
2. ($\forall k < n \ P(k)) \implies P(n)$

Then $P(n)$ is true for all integers $n >= 1$.
$P(1)$ is obviously true.

> To use quicksort to sort an array of size 𝑛, we partition it into three pieces: the first 𝑘 subarray, the pivot (which will be in its correct place), and the remaining subarray of size 𝑛−𝑘−1. By the way partition works, every element in the first subarray will be less than or equal to the pivot and every element in the other subarray will be greater than or equal to the pivot, so when we recursively sort the first and last subarrays, we will wind up having sorted the entire array.

> We show this is correct by strong induction: since the first subarray has 𝑘<𝑛 elements, we can assume by induction that it will be correctly sorted. Since the second subarray has 𝑛−𝑘−1<𝑛 elements, we can assume that it will be correctly sorted. Thus, putting all the pieces together, we will wind up having sorted the array.[^induction_proof]

[^induction_proof]: https://cs.stackexchange.com/a/63667