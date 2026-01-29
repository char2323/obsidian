# 二分查找

有两种区间的表现形式
- 当使用闭区间的形式的时候，首先注意 `left = 0; right = nums.size()-1`，不要超出数组范围。`while(left<=right)` 这里要用 `<=`，等于的时候是有意义的。
- 当使用左闭右开的形式的时候，`right = nums.size()` 即可。`while(left<right)`，这里取等于号没有意义。此外，`if(nums[middle]>target)` 时，`right = middle` 即可，因为 `middle` 取不到，其实也就是 `middle-1]` 的意思。
- 还要注意 `int middle = left + ((right - left) / 2);` 防止溢出。

时间复杂度：$O(\log n)$
空间复杂度：$O(1)$

完整代码：

```cpp
int search(vector<int>& nums, int target) {
	int left = 0;
	int right = nums.size() - 1;
	while(left<=right){
		int mid = left + (right - left) / 2;
		if(nums[mid] == target){
			return mid;
		} else if(nums[mid] < target){
			left = mid + 1;	
		} else {
			right = mid - 1;	
		}	
	}
	return -1;
}
```

# 双指针（快慢指针）

当要求原地修改数组或者链表的时候，往往考虑双指针，否则 `new` 一个新的数组或者链表即可。
我们让慢指针 `slow` 走在后面，快指针 `fast` 走在前面探路，找到一个不重复的元素就赋值给 `slow` 并让 `slow` 前进一步。
这样，就保证了 `nums[0..slow]` 都是无重复的元素，当 `fast` 指针遍历完整个数组 `nums` 后，`nums[0..slow]` 就是整个数组去重之后的结果。

```cpp
#include <vector>
using namespace std;

class Solution {
public:
    int removeDuplicates(vector<int>& nums) {
        if (nums.size() == 0) {
            return 0;
        }
        int slow = 0, fast = 0;
        while (fast < nums.size()) {
            if (nums[fast] != nums[slow]) {
                slow++;
                // 维护 nums[0..slow] 无重复
                nums[slow] = nums[fast];
            }
            fast++;
        }
        // 数组长度为索引 + 1
        return slow + 1;
    }
};
```