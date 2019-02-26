# LeetCode



##  23. Merge k Sorted Lists

将多个有序的链表合并成一个有序的链表。

用到的数据结构：

优先队列、小根堆

使用优先队列，对链表进行排序，使得在队首的链表的第一个结点的值最小。

这样，每次都直接从优先队列中获取第一个链表的第一个结点，再将剩下的链表放回优先队列中待用。这样每次都保证取出来的结点是最小的结点。

```cpp
class Solution
{
  public:
    struct compare
    {
        bool operator()(const ListNode *l, const ListNode *r)
        {
            return l->val > r->val;
        }
    };
    ListNode *mergeKLists(vector<ListNode *> &lists)
    { //priority_queue
        priority_queue<ListNode *, vector<ListNode *>, compare> q;
        for (auto l : lists)
        {
            if (l)
                q.push(l);
        }
        if (q.empty())
            return NULL;

        ListNode *result = q.top();
        q.pop();
        if (result->next)
            q.push(result->next);
        ListNode *tail = result;
        while (!q.empty())
        {
            tail->next = q.top();
            q.pop();
            tail = tail->next;
            if (tail->next)
                q.push(tail->next);
        }
        return result;
    }
};
```



##  32. Longest Valid Parentheses

求最长合法匹配的括号子串。





## 37.Sudoku Solver

解数独。回溯算法，遍历整个数独，找到空白位置'.'

尝试将1-9分别填入该空中，判断填入后数独是否合法，如果合法就继续往下搜索，如果不合法，就继续尝试。如果1-9均不能满足，说明该层搜索不存在解，将其还原成'.'，往上回溯。

```cpp
class Solution
{
  public:
    bool isValid(vector<vector<char>> &board, int row, int col, char c)
    {
        for (int i = 0; i < 9; i++)
        {
            if (board[i][col] == c)
                return false; //check row
            if (board[row][i] == c)
                return false; //check column
            if (board[3 * (row / 3) + i / 3][3 * (col / 3) + i % 3] == c)
                return false; //check 3*3 block
        }
        return true;
    }

    void solveSudoku(vector<vector<char>> &board)
    {
        if (board.size() == 0 || board[0].size() == 0)
            return;
        solve(board);
    }

    bool solve(vector<vector<char>> &board)
    {

        for (int i = 0; i < board.size(); i++)
        {
            for (int j = 0; j < board[0].size(); j++)
            {
                if (board[i][j] == '.')
                {
                    for (char ch = '1'; ch <= '9'; ch++)
                    {

                        if (isValid(board, i, j, ch))
                        {
                            board[i][j] = ch;
                            if (solve(board))
                                return true;
                            else
                                board[i][j] = '.';
                        }
                    }

                    return false;
                }
            }
        }
        return true;
    }
};

```

## 41.First Missing Positive

求出没有出现的最小正整数。

通过将对应的数交换到对应位置来判断。比如1应该被交换到nums\[0\]，2为nums\[1\]，这样比较nums\[i\]和i+1的关系就能得出是缺少哪一个数了。

对每一个处于\[1,n\]之间的数，都是需要做换位操作的。对其进行换位知道找到对应的位置，使得nums\[i\]==nums\[nums\[i\]-1\]。不在这个区间内的数可以忽略不管，没有影响。

```cpp
class Solution
{
  public:
    int firstMissingPositive(vector<int> &nums)
    {

        for (int i = 0; i < nums.size(); i++)
        {
            while (nums[i] >= 1 && nums[i] <= nums.size() &&
             nums[i] != nums[nums[i] - 1])
                swap(nums[i], nums[nums[i] - 1]);
        }

        for (int i = 0; i < nums.size(); i++)
        {
            if (nums[i] != (i + 1))
                return i + 1;
        }

        return nums.size() + 1;
    }
};
```

## 42.Trapping Rain Water

数组给出一组高度，求去由此构成的形状能蓄多少水。如图所示。

![](.gitbook/assets/image%20%286%29.png)

对每一个位置分别计算。对于一个位置来说，其能蓄多少水取决于其左边最大值leftmax和右边最大值rightmax，值为min\(leftmax,rightmax\)-height。

用两个指针分别指向左右两侧，如果左边的值较小，那么此时min\(leftmax,rightmax\)必定是取决于左边，所以只需要考虑左边部分。比较当前指针指向的值height\[left\]和Leftmax的大小，如果leftmax较小，就更新其值，如果leftmax较大，则可以容纳leftmax-height\[left\]。对于右侧同理。

```cpp
class Solution
{
  public:
    int trap(vector<int> &height)
    {

        int left = 0, right = height.size() - 1;
        int leftmax = 0, rightmax = 0;
        int ret = 0;
        while (left <= right)
        {
            if (height[left] < height[right])
            {
                if (height[left] > leftmax)
                    leftmax = height[left];
                else
                    ret += leftmax - height[left];
                left++;
            }
            else
            {
                if (height[right] > rightmax)
                    rightmax = height[right];
                else
                    ret += rightmax - height[right];
                right--;
            }
        }
        return ret;
    }
};
```

## 44.Wildcard Matching

类似正则表达式的字符串匹配。动态规划。dp\[i\]\[j\]表示s的前i个字符和p前j个字符是否匹配。注意dp\[i\]\[j\]和s\[i-1\]\[s\[j-1\]对应。

首先dp\[0\]\[0\]=true，因为两个空串是匹配的。然后，给dp\[0\]\[j\]赋值，当p\[j-1\]为'\*'才能和空串匹配。而对于dp\[i\]\[0\]，s非空而p为空，必定不匹配。

处理了为0的下标后，不用考虑i-1、j-1越界的情况，方便编写代码。

如果p\[j-1\]是'\*'，那么可以匹配s的最后一个字符，所以dp\[i\]\[j\] = dp\[i-1\]\[j\]。或者\*匹配空串，dp\[i\]\[j\] = dp\[i\]\[j-1\]。二者满足一个即可，所以用或运算。

如果p\[j-1\]是'?'，则可以和s\[i-1\]匹配。或者s\[i-1\]==p\[j-1\]，直接匹配。此时dp\[i\]\[j\] = dp\[i-1\]\[j-1\]

其他情况均不匹配，dp\[i\]\[j\]保持false，不需要赋值。

```cpp
class Solution
{
  public:
    bool isMatch(string s, string p)
    {

        vector<vector<bool>> dp(s.length() + 1,
         vector<bool>(p.length() + 1, false));
        dp[0][0] = true;

        for (int j = 1; j <= p.length(); j++)
        {
            dp[0][j] = dp[0][j - 1] && p[j - 1] == '*';
        }

        for (int i = 1; i <= s.length(); i++)
        {
            for (int j = 1; j <= p.length(); j++)
            {
                if (p[j - 1] == '*')
                    dp[i][j] = dp[i - 1][j] || dp[i][j - 1];
                else if (p[j - 1] == '?' || p[j - 1] == s[i - 1])
                    dp[i][j] = dp[i - 1][j - 1];
            }
        }
        return dp[s.length()][p.length()];
    }
};
```

还可以直接利用多指针直接解析，判断能否匹配。

i,j分别表示当前s、p的字符。先判断是否相同，p\[j\]是否是'?'。然后如果p\[j\]是'\*'，就用starindex来记录这个\*的下标，用si来记录此时i指针的值，用于回溯。记录后j++，因为'\*'可以当作空处理。如果之后不匹配再回退。当两个条件都不满足时，需要回退，重新匹配'\*'，用starindex找到上一个\*的位置，将j仍然赋值为其后一位。并将i也赋值为si（即之前i值）的后一位，这样表示用'\*'匹配了s中的第i个字符。

```cpp
class Solution
{
  public:
    bool isMatch(string s, string p)
    {
        int i = 0, j = 0;
        int si;
        int starindex = -1;
        while (s[i])
        {
            if (s[i] == p[j] || p[j] == '?')
            {
                i++;
                j++;
            }
            else if (p[j] == '*')
            {
                starindex = j;
                j++;
                si = i;
            }
            else if (starindex != -1)
            {
                j = starindex + 1;
                i = ++si;
            }
            else
                return false;
        }

        while (p[j] == '*')
            j++;
        return p[j] == 0;
    }
};
```

## 45.Jump Game II

从0开始跳，最少多少步能跳到末尾。nums\[i\]表示能跳的步数。

BFS。每一层遍历时求出最大能跳到的地方，如果满足就直接返回。

```cpp
class Solution
{
  public:
    int jump(vector<int> &nums)
    {

        if (nums.size() <= 1)
            return 0;
        int level = 0, currentMax = 0, i = 0, nextMax = 0;

        while (i < nums.size() && (currentMax - i + 1 > 0))
        {
            level++;
            for (; i <= currentMax; i++)
            {
                nextMax = max(nextMax, A[i] + i);
                if (nextMax >= n - 1)
                    return level;
            }
            currentMax = nextMax;
        }
        return 0;
    }
};
```

第一时间想到的解法是DP..

```cpp
class Solution
{
  public:
    int jump(vector<int> &nums)
    {

        vector<int> dp(nums.size() + 1, INT_MAX);
        dp[0] = 0;
        for (int i = 1; i < nums.size(); i++)
        {
            for (int j = 0; j < i; j++)
            {
                if (j + nums[j] >= i)
                    dp[i] = min(dp[i], dp[j] + 1);
            }
        }
        return dp[nums.size() - 1];
    }
};
```

## 51.N-Queens

N皇后问题。求出所有解。

第一次做的时候，是将整个map全部遍历一次，但实际上因为每一行不能出现相同的皇后，所以每次只要安排第row行的皇后即可。这样只需要遍历n次，并安排合理的col值即可。

此外，在判断两条斜线方向是否有冲突的时候，只需要判断上半部分即可。因为从上往下遍历，下面在没有遍历到的情况下全是'.' 当row==n时表示完成遍历，得到一个解，将其放入ret中返回。

```cpp
class Solution
{
  public:
    vector<vector<string>> ret;

    vector<vector<string>> solveNQueens(int n)
    {
        string temp(n, '.');
        vector<string> solution(n, temp);
        helper(solution, 0, n);
        return ret;
    }

    void helper(vector<string> &map, int row, int n)
    {
        if (row == n){
            ret.push_back(map);
            return ;
        }
        for (int col = 0; col < n; col++)
        {
            if (IsValid(map, row, col, n))
            {
                map[row][col] = 'Q';
                helper(map, row + 1, n);
                map[row][col] = '.';
            }
        }
    }

    bool IsValid(vector<string> &map, int row, int col, int n)
    {
        for (int i = 0; i <= row; i++)
        {
            if (map[i][col] == 'Q')
                return false;
        }

        for (int i = row - 1, j = col - 1; i >= 0 && j >= 0; i--, j--)
            if (map[i][j] == 'Q')
                return false;
        for (int i = row - 1, j = col + 1; i >= 0 && j < n; i--, j++)
            if (map[i][j] == 'Q')
                return false;
        return true;
    }
};
```

## 57.Insert Interval

合并区间。

```cpp
class Solution
{
  public:
    vector<Interval> insert(vector<Interval> &intervals, Interval newInterval)
    {
        auto comp = [](Interval &I1, Interval &I2) {
            return I1.start < I2.start;
        };
        intervals.push_back(newInterval);
        sort(intervals.begin(), intervals.end(), comp);
        vector<Interval> ret;
        int pre = 0;
        ret.push_back(intervals[pre]);
        for (int i = 1; i < intervals.size(); i++)
        {

            if (ret[pre].end < intervals[i].start)
            {
                ret.push_back(intervals[i]);
                pre++;
            }
            else if (ret[pre].end < intervals[i].end)
                ret[pre].end = intervals[i].end;
        }

        return ret;
    }
};
```







