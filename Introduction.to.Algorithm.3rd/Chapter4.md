Divided Conquer

```
FIND-MAXIMUM-SUBARRAY(A,low,high)
if hight==low
    return (low,high, A[low])
else mid=(low+high)/2
    (left-low,left-high,left-sum)=
        FIND-MAXIMUM-SUBARRAY(A,low,mid)
    (right-low,right-high,right-sum)=
        FIND-MAXIMUM-SUBARRAY(A,mid+1,high)
    (cross-low,cross-high,cross-sum)=
        FIND-MAX-CROSSING-SUBARRAY(A,low,mid,high)
    if left-sum>=right-sum and left-sum>=cross-sum
        return (left-low, left-high,left-sum)
    elseif right-sum>=left-sum and right-sum>=cross-sum
        return (right-low,right-high,right-sum)
    else
        return (cross-low,cross-high,cross-sum)

FIND-MAX-CROSSING-SUBARRAY(A,low,mid,high)
left-sum=-POSIVITIVE_INFINITE
sum=0
for i=mid downto low
    sum=sum+A[i]
    if sum>left-sum
        left-sum=sum
        max-left=t
right-sum=-POSIVITIVE_INFINITE
sum=0
for j=mid+1 to high
    sum=sum+A[j]
    if sum>right-sum
        right-sum=sum
        max-right=j
return (max-left,max-right,left-sum+right-sum)
```

C++

```
class Solution {
public:
    int maxSubArray(vector<int>& nums) {
        if (nums.empty()) return 0;
        return helper(nums, 0, (int)nums.size() - 1);
    }
    int helper(vector<int>& nums, int left, int right) {
        if (left >= right) return nums[left];
        int mid = left + (right - left) / 2;
        int lmax = helper(nums, left, mid - 1);
        int rmax = helper(nums, mid + 1, right);
        int mmax = nums[mid], t = mmax;
        for (int i = mid - 1; i >= left; --i) {
            t += nums[i];
            mmax = max(mmax, t);
        }
        t = mmax;
        for (int i = mid + 1; i <= right; ++i) {
            t += nums[i];
            mmax = max(mmax, t);
        }
        return max(mmax, max(lmax, rmax));
    }
};
```

Java

```
public class Solution {
	
	public int maxSubArray(int[] nums) {
		if(nums.length==0)
			return 0;
		return helper(nums,0,nums.length-1);
	}
	
	public int helper(int[] nums, int left, int right) {
		if(left>=right)
			return nums[left];
		int mid=left+(right-left)/2;
		int lmax=helper(nums,left,mid-1);
		int rmax=helper(nums,mid+1,right);
		int mmax=nums[mid],t=mmax;
		for(int i=mid-1;i>=left;--i) {
			t+=nums[i];
			mmax=Math.max(mmax, t);
		}
		t=mmax;
		for(int i=mid+1;i<=right;i++) {
			t+=nums[i];
			mmax=Math.max(mmax, t);
		}
		return Math.max(mmax, Math.max(lmax, rmax));
	}

}
```
