### Segment Trees

- A Segment Tree is a data structure that stores information about array intervals as a tree.
- This allows answering range queries over an array efficiently, while still being flexible enough to allow quick modification of the array. 

> Segment Tree is useful when the array has many "update" operations in intervals. [Read more](https://cp-algorithms.com/data_structures/segment_tree.html)

Video Tutorial - https://www.youtube.com/watch?v=2bSS8rtFym4&ab_channel=TECHDOSE

#### C++ Code
<img width="820" src="https://user-images.githubusercontent.com/27401142/182177229-6db97527-7130-403a-ae90-0032b5e4f39c.png">

```cpp
SUM RANGE QUERY 🔴🔴🔴🔴🔴🔴🔴🔴🔴🔴🔴🔴🔴
void updateValue(vector<int>& st, int start, int end, int updatingIdx, int diff, int idx) {
    // To be updated idx is outside range
    if (updatingIdx < start || updatingIdx > end)
        return;

    //Idx is in the range of this node, update the value
    // of this node and its children
    st[idx] += diff;
    if (start != end) {
        int mid = start + (end - start) / 2;
        updateValue(st, start, mid, updatingIdx, diff, 2 * idx + 1);
        updateValue(st, mid + 1, end, updatingIdx, diff, 2 * idx + 2);
    }
}

int getSum(vector<int>& st, int start, int end, int searchStartIdx, int searchEndIdx, int idx) {
    // Case 1: Full overlaping condition
    // Segment of the node is part of the range
    // --searchStartIdx-- start ---- end ---searchEndIdx---
    if (searchStartIdx <= start && searchEndIdx >= end)
        return st[idx];

    // Case 2: NO-overlap condition
    // ---end -- searchStartIdx--
    // ---searchEndIdx -- start--
    if (end < searchStartIdx || start > searchEndIdx)
        return 0;

    // Case 3: Partial Overlap
    int mid = start + (end - start) / 2;
    return getSum(st, start, mid, searchStartIdx, searchEndIdx, 2 * idx + 1) +
           getSum(st, mid + 1, end, searchStartIdx, searchEndIdx, 2 * idx + 2);
}

int fillSTValues(vector<int>& nums, vector<int>& st, int start, int end, int idx) {
    // If there is only single element in array, store in st
    // and return it
    if (start == end) {
        st[idx] = nums[start];
        return nums[start];
    }

    // Split at any pivot into left and right subtrees
    // and store their sum into current node
    int mid = start + (end - start) / 2;
    st[idx] = fillSTValues(nums, st, start, mid, 2 * idx + 1) +
              fillSTValues(nums, st, mid + 1, end, 2 * idx + 2);

    return st[idx];
}

void buildST(vector<int>& nums, vector<int>& st) {
    //allocate memory for segement tree
    int n = nums.size();
    int height = ceil(log2(n));

    //Maximum size of segment tree
    int max_size = 2 * pow(2, height) - 1;

    st.resize(max_size);
    fillSTValues(nums, st, 0, n - 1, 0);
}

int main() {
    vector<int> nums = {1, 3, 5, 7, 9, 11};
    int n = nums.size();

    //build segment tree
    vector<int> st;
    buildST(nums, st);

    //Find sum in range - output = 32
    int searchStartIdx = 2, searchEndIdx = 5;
    cout << getSum(st, 0, n - 1, searchStartIdx, searchEndIdx, 0) << endl;

    //Update value
    int newValue = 7, updatingIdx = 2;
    int diff = newValue - nums[updatingIdx];
    updateValue(st, 0, n - 1, diff, updatingIdx, 0);

    //Find sum range again - output = 34
    cout << getSum(st, 0, n - 1, searchStartIdx, searchEndIdx, 0) << endl;

    return 0;
}






RANGE MINIMUM QUERY 🔴🔴🔴🔴🔴🔴🔴🔴🔴🔴🔴🔴🔴
#include <iostream>
#include <vector>
#include <algorithm>
using namespace std;
int getMid(int s, int e)
{
	return s + (e - s) / 2;
}

void updateValueUtil(vector<int> &st, int si, int L, int R, int i, int delta, int new_val)
{
	if (i < L || i > R)
		return;

	if (R == L)
	{
		st[si] += delta;
	}
	else
	{
		int mid = getMid(L, R);
		updateValueUtil(st, si * 2 + 1, L, mid, i, delta, new_val);
		updateValueUtil(st, si * 2 + 2, mid + 1, R, i, delta, new_val);
		st[si] = min(st[si * 2 + 1], st[si * 2 + 2]);
	}
}

void updateValue(vector<int> &arr, vector<int> &st, int n, int i, int new_val)
{
	if (i < 0 || i > n - 1)
	{
		cout << "Invalid input";
		return;
	}
	int delta = new_val - arr[i];
	arr[i] = new_val;
	updateValueUtil(st, 0, 0, n - 1, i, delta, new_val);
}

int getMinUtil(vector<int> &st, int L, int R, int sL, int sR, int si)
{
	if (sL <= L && sR >= R)
		return st[si];
	if (R < sL || L > sR)
		return INT_MAX;
	int mid = getMid(L, R);
	return min(getMinUtil(st, L, mid, sL, sR, 2 * si + 1),
		getMinUtil(st, mid + 1, R, sL, sR, 2 * si + 2));
}

int getMin(vector<int> &st, int n, int L, int R)
{
	if (L < 0 || R > n - 1 || L > R)
	{
		printf("Invalid Input");
		return -1;
	}
	return getMinUtil(st, 0, n - 1, L, R, 0);
}

int constructSTUtil(vector<int> &st, int si, vector<int> &arr, int L, int R)
{
	if (L == R)
	{
		st[si] = arr[L];
		return arr[L];
	}
	int mid = getMid(L, R);
	st[si] = min(constructSTUtil(st, si * 2 + 1, arr, L, mid),
		constructSTUtil(st, si * 2 + 2, arr, mid + 1, R));
	return st[si];
}

int getMaxSize(int n)
{
	int x = (int)(ceil(log2(n)));
	return 2 * (int)pow(2, x) - 1;
}

vector<int> constructST(vector<int> &arr)
{
	int max_size = getMaxSize(arr.size());
	vector<int> st(max_size, 0);
	constructSTUtil(st, 0, arr, 0, arr.size() - 1);
	return st;
}

void printSegmentTree(vector<int> st, int max_size)
{
	for (int i = 0; i < max_size; i++)
	{
		cout << st[i] << " ";
	}
	cout << endl;
}

int main()
{
	vector<int> arr{ 5, 2, 7, 1, 3 };
	vector<int> st = constructST(arr);
	printSegmentTree(st, getMaxSize(arr.size()));
	cout << "Min of values in given range = " << getMin(st, arr.size(), 2, 4) << endl;

	updateValue(arr, st, arr.size(), 3, 11);
	printSegmentTree(st, getMaxSize(arr.size()));
	cout << "Updated min of given range = " << getMin(st, arr.size(), 2, 4) << endl;

	updateValue(arr, st, arr.size(), 3, 1);
	printSegmentTree(st, getMaxSize(arr.size()));
	cout << "Updated min of given range = " << getMin(st, arr.size(), 2, 4) << endl;
	return 0;
}



XOR RANGE QUERY 🔴🔴🔴🔴🔴🔴🔴🔴🔴🔴🔴🔴🔴🔴🔴
#include <iostream>
#include <vector>
using namespace std;
int getMid(int s, int e)
{
	return s + (e - s) / 2;
}

void updateValueUtil(vector<int> &st, int si, int L, int R, int i, int prev_val, int new_val)
{
	if (i < L || i > R)
		return;
	st[si] = (st[si] ^ prev_val) ^ new_val; // xor the previous value to nullify with node before xoring new value
	if (R != L)
	{
		int mid = getMid(L, R);
		updateValueUtil(st, si * 2 + 1, L, mid, i, prev_val, new_val);
		updateValueUtil(st, si * 2 + 2, mid + 1, R, i, prev_val, new_val);
	}
}

void updateValue(vector<int> &arr, vector<int> &st, int n, int i, int new_val)
{
	if (i < 0 || i > n - 1)
	{
		cout << "Invalid input";
		return;
	}
	int temp = arr[i];
	arr[i] = new_val;
	updateValueUtil(st, 0, 0, n - 1, i, temp, new_val);
}

int getXorUtil(vector<int> &st, int L, int R, int sL, int sR, int si)
{
	if (sL <= L && sR >= R)
		return st[si];
	if (R < sL || L > sR)
		return 0;
	int mid = getMid(L, R);
	return getXorUtil(st, L, mid, sL, sR, 2 * si + 1) ^
		getXorUtil(st, mid + 1, R, sL, sR, 2 * si + 2);
}

int getXor(vector<int> &st, int n, int L, int R)
{
	if (L < 0 || R > n - 1 || L > R)
	{
		printf("Invalid Input");
		return -1;
	}
	return getXorUtil(st, 0, n - 1, L, R, 0);
}

int constructSTUtil(vector<int> &st, int si, vector<int> &arr, int L, int R)
{
	if (L == R)
	{
		st[si] = arr[L];
		return arr[L];
	}
	int mid = getMid(L, R);
	st[si] = constructSTUtil(st, si * 2 + 1, arr, L, mid) ^
		constructSTUtil(st, si * 2 + 2, arr, mid + 1, R);
	return st[si];
}

int getMaxSize(int n)
{
	int x = (int)(ceil(log2(n)));
	return 2 * (int)pow(2, x) - 1;
}

vector<int> constructST(vector<int> &arr)
{
	int max_size = getMaxSize(arr.size());
	vector<int> st(max_size, 0);
	constructSTUtil(st, 0, arr, 0, arr.size() - 1);
	return st;
}

void printSegmentTree(vector<int> st, int max_size)
{
	for (int i = 0; i < max_size; i++)
	{
		cout << st[i] << " ";
	}
	cout << endl;
}

int main()
{
	vector<int> arr{ 8,5,3,7,6 };
	vector<int> st = constructST(arr);
	printSegmentTree(st, getMaxSize(arr.size()));
	cout << "Xor of values in given range = " << getXor(st, arr.size(), 2, 4) << endl;

	updateValue(arr, st, arr.size(), 3, 11);
	printSegmentTree(st, getMaxSize(arr.size()));
	cout << "Updated xor of given range = " << getXor(st, arr.size(), 2, 4) << endl;

	updateValue(arr, st, arr.size(), 3, 7);
	printSegmentTree(st, getMaxSize(arr.size()));
	cout << "Updated xor of given range = " << getXor(st, arr.size(), 2, 4) << endl;
	return 0;
}
```
