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





