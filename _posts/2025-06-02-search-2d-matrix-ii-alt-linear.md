---
title: "Alternative O(M+N) solution for LeetCode 240: Search a 2D Matrix II"
date: 2025-06-02T16:44:30-04:00
categories:
  - blog
tags:
  - Algorithm
  - LeetCode
---

# Introduction
LeetCode's [Search a 2D Matrix II](https://leetcode.com/problems/search-a-2d-matrix-ii/) is a classic problem that can be solved using various approaches.
There is a well-known O(M+N) solution that starts from the top-right corner and moves left or down based on the comparison with the target. I won't get into details of that solution here, basically it works by eliminating one row or one column at each step, hence the O(M+N) time complexity. Here is a sample implementation:

```python
def searchMatrix(matrix, target):
    if not matrix or not matrix[0]:
        return False

    rows, cols = len(matrix), len(matrix[0])
    row, col = 0, cols - 1

    while row < rows and col >= 0:
        if matrix[row][col] == target:
            return True
        elif matrix[row][col] > target:
            col -= 1
        else:
            row += 1

    return False
```

This method is simple, efficient and ellegant, however, I have 2 issues with it:
1. It is not intuitive to start from the top-right or bottom-left corner. It feels like an arbitrary trick that one happens to come up with.
2. This solution has almost nothing to do with classical computer science algorithms or data structures, which makes it less satisfying for those who enjoy more structured approaches.

# Alternative O(M+N) Solution
I wanted to share an alternative O(M+N) solution that is less commonly discussed but still as effective; it has a more structured feel to it. The idea is to use binary search on the diagonal of the matrix, which seperates the matrix into four quadrants, due to the constraints of the problem, we know that the target can only be in the top-right or bottom-left quadrants. Treating these 2 quadrants as two separate matrices, we can apply the same binary search approach to each of them. Eventually, the sieze of the matrix will be reduced to a single row or a single column, at which point we can simply check if the target is in that row or column. Note that a special base case is when this single row or column contains only one element, in which case we can simply check if that element is equal to the target.

To illustrate this, let's consider the following matrix:

|      |      |      |      |
|------|------|------|------|
| <span style="color:green">1</span>  | <span style="color:green">3</span>  | <span style="color:green">5</span>  | <span style="color:green">7</span>  |
| <span style="color:green">10</span> | <span style="color:green">11</span> | <span style="color:green">16</span> | <span style="color:green">20</span> |
| <span style="color:green">23</span> | <span style="color:green">30</span> | <span style="color:green">34</span> | <span style="color:green">60</span> |
| <span style="color:green">61</span> | <span style="color:green">66</span> | <span style="color:green">70</span> | <span style="color:green">80</span> |

Let's say the target is 16. The diagonal of the matrix is `[1, 11, 34, 80]`. We can perform a binary search on this diagonal to find the closest elements to the target. In this case, we find that 11 and 34 are the closest element. 


|      |      |      |      |
|------|------|------|------|
| <span style="color:red">1</span>  | <span style="color:red">3</span>  | <span style="color:green">5</span>  | <span style="color:green">7</span>  |
| <span style="color:red">10</span> | <span style="color:red">11</span> | <span style="color:green">16</span> | <span style="color:green">20</span> |
| <span style="color:green">23</span> | <span style="color:green">30</span> | <span style="color:red">34</span> | <span style="color:red">60</span> |
| <span style="color:green">61</span> | <span style="color:green">66</span> | <span style="color:red">70</span> | <span style="color:red">80</span> |

This seperates the matrix into 4 quadrants. Due to the constraints of the problem, we know that the top left quadrant can only contain elements less than the target, and the bottom right quadrant can only contain elements greater than the target. Therefore, we can discard these 2 quadrants, and move on to the top right and bottom left quadrants. We can treat these two quadrants as two separate matrices and apply the same binary search approach to each of them, until we reach a single row or a single column.

## Time Complexity
To simplify calculating the time complexity, let's first consider the case where the matrix is square, i.e. M = N. In this case, in order to reach a single row or a single column, we need to perform log(N) iterations. In the first iteration, we will perform binary search once on diagnoal lenth of N, and the second iteration will perform binary search twice on diagonal length of (N/2) on average, and grows exponentially until we reach log(N) iterations. Therefore, the time complexity is:


$$
S \;=\; 1\cdot \log N \;+\; 2\cdot \log\frac{N}{2} \;+\; 4\cdot \log\frac{N}{4} \;+\; \cdots \;+\; 2^{L}\cdot \log\frac{N}{2^{L}},
\qquad L = \log_{2}N,
$$

write each term in the form

$$
2^{i}\,\log\Bigl(\frac{N}{2^{\,i}}\Bigr)
\;=\;2^{i}\bigl[\log N \;-\; \log(2^{\,i})\bigr]
\;=\;2^{i}\bigl(\log N - i\bigr),
$$

where $\log$ is base 2 and $i=0,1,\dots,L$.  Hence

$$
S \;=\;\sum_{i=0}^{L} 2^{i}\,(\log N - i)
\;=\;\sum_{i=0}^{L} \bigl(2^{i}\,\log N \;-\; i\,2^{i}\bigr).
$$

Split it into two sums:

$$
S 
=\;(\log N)\sum_{i=0}^{L}2^{i}\;-\;\sum_{i=0}^{L}i\,2^{i}.
$$

1. $\displaystyle \sum_{i=0}^{L}2^{i} \;=\; 2^{\,L+1}-1 \;=\;2\cdot 2^{L}-1 \;=\;2N-1.$

2. For $\displaystyle \sum_{i=0}^{L} i\,2^{i}$, it is easier to re‐index by setting $j = L - i$.  Then $i = L - j$ and $2^{i} = 2^{\,L-j} = 2^{L}\,2^{-j} = N\cdot 2^{-j}$.  As $i$ runs from $0$ to $L$, $j$ runs from $L$ down to $0$.  Thus

$$
\sum_{i=0}^{L} i\,2^{i}
\;=\;\sum_{j=0}^{L} (L - j)\,\bigl(N\cdot 2^{-j}\bigr)
\;=\;N\sum_{j=0}^{L} \frac{\,L - j\,}{2^{\,j}}
\;=\;N\Bigl(L\sum_{j=0}^{L}\frac1{2^{j}} \;-\;\sum_{j=0}^{L}\frac{j}{2^{j}}\Bigr).
$$

But

* $\displaystyle \sum_{j=0}^{L}\frac1{2^{j}} = 2 - \frac{1}{2^{\,L}} = 2 - \frac{1}{N}.$
* $\displaystyle \sum_{j=0}^{L}\frac{j}{2^{j}}$ converges to $2$ as $L\to\infty$; more precisely
  $\sum_{j=0}^{L} \frac{j}{2^{j}} = 2 - O\!\bigl(\tfrac{L}{2^{\,L}}\bigr) = 2 - O\!\bigl(\tfrac{\log N}{N}\bigr).$

Therefore

$$
\sum_{i=0}^{L} i\,2^{i}
\;=\;N\Bigl[L\Bigl(2 - \tfrac1N\Bigr)\;-\;\Bigl(2 - O\bigl(\tfrac{\log N}{N}\bigr)\Bigr)\Bigr]
\;=\;N\bigl(2L - 2\bigr)\;-\;L \;+\; O(\log N).
$$

Since $L = \log N$, that becomes

$$
\sum_{i=0}^{L} i\,2^{i}
\;=\;2N\log N \;-\;2N \;-\;\log N \;+\;O(\log N)
\;=\;2N\log N \;-\;2N \;+\;O(\log N).
$$

Putting everything back into $S$:

$$
S 
=\;(\log N)\,(2N-1)\;-\;\bigl[\,2N\log N \;-\;2N \;+\;O(\log N)\bigr]
\;=\;2N\log N \;-\;\log N \;-\;2N\log N \;+\;2N \;-\;O(\log N).
$$

The $2N\log N$ terms cancel, leaving

$$
S \;=\;2N \;-\;\log N \;-\;O(\log N) \;=\;2N \;-\;O(\log N).
$$

Thus

$$
S = 2N - O(\log N) = \Theta(N),
$$

and in big-O notation

$$
\boxed{S = O(N).}
$$

### Extending to Non-Square Matrices
If the matrix is not square, i.e. M != N, we can still apply the same approach. The only difference is that we need to perform binary search on the diagonal of the matrix, and extend to the row or column on the edge of the matrix, which will have a length of max(M, N).


1.  **Binary Search Cost**: At each step, a binary search is performed on an extended diagonal. The length of this diagonal is roughly proportional to $\max(m,n)$. The cost of this binary search is $O(\log(\max(m,n)))$.

2.  **Recursive Calls**: The problem states that the process is repeated in two sub-quadrants (upper right and lower left). If the pivot chosen by the diagonal binary search is reasonably central, these subproblems will have dimensions roughly $m/2 \times n/2$. Hence, the number of recursive calls is $\log(\max(m,n))$

3.  **Generalization**: Similar to the case with square matrices, the complexity is $O(\max(M,N))$. Because $\max(M,N) \le M+N \le 2\max(M,N)$, $O(\max(M,N))$ is equivalent to $O(M+N)$.

Thus, the overall time complexity is **$O(M+N)$**.

# Sample Implementation
```python
class SolutionTwo:
    def searchMatrix(self, matrix, target):
        
        # perform binary search on a "generalized diagonal path".
        def search_diagonal(matrix, target, r0, c0, r1, c1):
            H = r1 - r0 + 1 
            W = c1 - c0 + 1

            if H <= 0 or W <= 0:
                return (-1, -1)

            diag_len_main = min(H, W)
            path_len = max(H, W)

            # Helper function to get matrix coordinates for an index k on the generalized path
            def get_coords_on_path(k_on_path):
                if k_on_path < diag_len_main:
                    return (r0 + k_on_path, c0 + k_on_path)
                else:
                    k_on_extension = k_on_path - diag_len_main
                    if H > W:
                        return (r0 + diag_len_main + k_on_extension, c0 + diag_len_main - 1)
                    else:
                        return (r0 + diag_len_main - 1, c0 + diag_len_main + k_on_extension)
            
            low_k, high_k = 0, path_len - 1
            first_ge_k_on_path = path_len 

            while low_k <= high_k:
                mid_k = (low_k + high_k) // 2
                mid_r, mid_c = get_coords_on_path(mid_k)
                val_at_mid = matrix[mid_r][mid_c]

                if val_at_mid < target:
                    low_k = mid_k + 1
                else:
                    first_ge_k_on_path = mid_k
                    high_k = mid_k - 1
            
            if first_ge_k_on_path < path_len:
                final_pivot_r, final_pivot_c = get_coords_on_path(first_ge_k_on_path)
                return (final_pivot_r, final_pivot_c)
            else:
                return (-1, -1)

        def helper(r0, c0, r1, c1):
            if r0 > r1 or c0 > c1:
                return False
            
            if r0 == r1 and c0 == c1:
                return matrix[r0][c0] == target

            pi, pj = search_diagonal(matrix, target, r0, c0, r1, c1)
            
            if pi != -1: 
                is_1d_submatrix = (r0 == r1) or (c0 == c1)
                is_pivot_at_submatrix_endpoint = ((pi, pj) == (r0, c0) or (pi, pj) == (r1, c1))
                
                if is_1d_submatrix and is_pivot_at_submatrix_endpoint:
                     return matrix[pi][pj] == target 
            
            if pi == -1: 
                return False
            
            if matrix[pi][pj] == target:
                return True
            
            # search in the top-right and bottom-left quadrants
            if helper(r0, pj, pi, c1):
                return True
            if helper(pi, c0, r1, pj):
                return True
            
            return False

        if not matrix or not matrix[0]:
            return False
        return helper(0, 0, len(matrix) - 1, len(matrix[0]) - 1)
```

# Conclusion
While this alternative solution might be harder to implement and has more edge cases toconsider, it is in my opinion a more strcutured approach that actually utilizes basic computer science concepts such as binary search and recursion. It also has the same time complexity as the well-known O(M+N) solution, making it a valid alternative. I hope you find this approach interesting and useful for your coding practice!

# See Also
A simialr approach posted by a LeetCode user [here](https://leetcode.com/problems/search-a-2d-matrix-ii/solutions/3205428/a-different-o-m-n-solution-2d-binary-search-no-trickery-involved-methodical/). This approach splits the matrix into 4 quadrants using the middle row, instead of the diagonal.