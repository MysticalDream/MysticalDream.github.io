---
title: "n 皇后问题"
date: 2022-03-25T19:38:56+08:00
draft: false
# summary: "博客的概述" # 文章简单描述，会展示在主页
# categories:
# - category 1
# - category 2

tags:
- java
- 算法

---


# 题目

> **n 皇后问题** 研究的是如何将 `n` 个皇后放置在` n×n` 的棋盘上，并且使皇后彼此之间不能相互攻击。
> 
> 给你一个整数` n` ，返回所有不同的 **n 皇后问题** 的解决方案。
> 
> 每一种解法包含一个不同的 **n 皇后问题** 的棋子放置方案，该方案中` 'Q' `和` '.' `分别代表了皇后和空位。
![N皇后](https://jsdelivr.codeqihan.com/gh/MysticalDream/images/assets/202311121948512.png)
提示：
1 <= n <= 9
> 来源：力扣（LeetCode）
> 
>  链接：https://leetcode-cn.com/problems/n-queens

# 思路

题目的**使皇后彼此之间不能相互攻击**的意思是任意两个皇后不能出现在**同一行、同一列和同一条斜线上**，如下图所示，红线的地方不能再摆放皇后
![皇后摆放规则](https://jsdelivr.codeqihan.com/gh/MysticalDream/images/assets/202311121948362.png)
根据规则我们可以断定每个皇后都是在不同的行，所以我们可以建立一个大小为`n`一维数组`row`表示`每一行`皇后`所在列`的位置。对于摆放位置是否符合的判断，我们只需要判断是否在同一列或则同一斜线上

>同一列判断比较简单
>同一斜线我们可以假设两点坐标分别是(x1,y1),(x2,y2)
如果他们在棋盘上的同一斜线则符合
 $\frac{y1-y2}{x1-x2}$= ±1
 >```java
> 即 |y1-y2|=|x1-x2|
> ```

不难得到判断条件是
```java
//r是当前尝试摆放的行,i是r行之前的行，也就是摆放好的那几行，即[0,r)
row[i] == row[r] || Math.abs(row[i] - row[r]) == Math.abs(i - r)
```
1. **我们假设`n`为`4`，即在`4×4`的棋盘上.我们可以从第一行的第一列开始放皇后，显然第一次放这个皇后是不会有冲突的，所以可以放在第一行第一列，即`row[0]=0`，那么第一行的皇后处理完成。**
2. **接着到下一行也就是第二行，寻找皇后可以放置的位置，首先在第一列尝试，这里第一列由于第一行的皇后已经占据，所以不能放在第一列，接着到下一列，这时候发现会和第一个皇后在同一个斜线上，所以也不能放置，再接着下一列，这时候发现这个位置是符合规则的，所以这一列可以放置皇后，即`row[1]=2`。**
3. **同理接着下一行，这一行第一列和第一行的皇后冲突了，第二列和第二行的皇后在同一个斜线上，第三列和第二行的皇后在同一列上，第四列和第二行的皇后在同一个斜线上，这一行所有的列都不行。**
4. **我们需要返回上一行，上一行皇后的位置在`row[1]`也就是`2`，说明这一行的这个皇后不能放置在这里，因为放在这里下一行就不能放置皇后了，又因为这一行的前几列已经尝试摆放过，那么只能继续下一列，所以在这一行原来列的基础上，前进到下一列，发现这一列是符合规则的，所以可以放置皇后，即`row[1]=3`**
5. **之后就到下一行重新开始寻找，即从这一行的第一列开始尝试摆放皇后，依此类推,直到摆放完最后一行，数组`row`中的值就是一种符合规则的摆法。**



# 测试通过的代码

```java
class Solution {
	static int[] row;
	static List<List<String>> lists;

	public List<List<String>> solveNQueens(int n) {
		row = new int[n];
		lists = new ArrayList<>(150);
		next(0);
		return lists;
	}

	public void next(int r) {
		if (r == row.length) {
			addResult();
			return;
		}

		for (int j = 0; j < row.length; j++) {
			row[r] = j;
			if (check(r)) {
				next(r + 1);
			}
		}

	}

	public boolean check(int r) {

		for (int i = 0; i < r; i++) {
			if (row[i] == row[r] || Math.abs(row[i] - row[r]) == Math.abs(i - r)) {
				return false;
			}
		}
		return true;
	}

	public void addResult() {

		List<String> list = new ArrayList<>(row.length);
		for (int i = 0; i < row.length; i++) {
			StringBuilder stringBuilder = new StringBuilder(row.length);
			for (int j = 0; j < row.length; j++) {
				if (j == row[i]) {
					stringBuilder.append("Q");
				} else {
					stringBuilder.append(".");
				}
			}
			list.add(stringBuilder.toString());
		}
		lists.add(list);
	}
}
```
