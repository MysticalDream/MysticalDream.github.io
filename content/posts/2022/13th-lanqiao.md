---
title: "2022年第十三届蓝桥杯省赛真题及部分答案解析(Java B组)"
date: 2022-04-09T17:24:51+08:00
draft: false
# summary: "博客的概述" # 文章简单描述，会展示在主页
categories:
- 算法

tags:
- 算法
- java

---

# 第十三届蓝桥杯大赛软件赛省赛



## 试题 A: 星期计算【填空题】
````
 本题总分： 5 分
 ````

 **【问题描述】**

> 已知今天是星期六，请问 20$^{22}$ 天后是星期几？
> 注意用数字 1 到 7 表示星期一到星期日

 **【答案提交】**
 >这是一道结果填空的题，你只需要算出结果后提交即可。本题的结果为一个整数，在提交答案时只填写这个整数，填写多余的内容将无法得分。

**第一种解法**

>利用java的大整数api求解

**代码**
```java
import java.math.BigInteger;

public class Main {

	public static void main(String[] args) {
		BigInteger bigInteger = BigInteger.valueOf(20).pow(22).mod(BigInteger.valueOf(7));
		int result = (6 + bigInteger.intValue()) % 7;
		System.out.println(result == 0 ? 7 : result);
	}

}
```

>输出结果:7

**第二种解法**

>利用计算器求解

  
![](https://jsdelivr.codeqihan.com/gh/MysticalDream/images/assets/202311122103921.png)
  
![](https://jsdelivr.codeqihan.com/gh/MysticalDream/images/assets/202311122103781.png)

>天数取模后余1，则取当前星期对应数字加一，即6+1=7

**第三种解法**

>如果直接求20$^{22}$再取余，20$^{22}$的结果基本类型是装不下的，所以可以在计算20$^{22}$的时候利用取余的运算规则先进行取余防止溢出。这个是常规的取余算法，当然还有快速幂取余法，不过对于这个题目，常规的算法就可以了

**代码**
```java
public class Main {

	public static void main(String[] args) {
		int ans = 1;
		for (int i = 0; i < 22; i++) {
			// 取余7防止溢出
			ans = ans * 20 % 7;
		}
		int result = (6 + ans) % 7;
		System.out.println(result == 0 ? 7 : result);
	}

}
```

>输出结果:7



## 试题 B: 山【填空题】
```
 本题总分： 5 分
 ```

**【问题描述】**

>这天小明正在学数数。
> 他突然发现有些正整数的形状像一座“山”，比如 123565321 、 145541 ，它们左右对称（回文）且数位上的数字先单调不减，后单调不增。
> 小明数了很久也没有数完，他想让你告诉他在区间[2022,2022222022]中有多少个数的形状像一座“山”。

**【答案提交】**
> 这是一道结果填空的题，你只需要算出结果后提交即可。本题的结果为一个整数，在提交答案时只填写这个整数，填写多余的内容将无法得分。

**第一种解法**
>暴力查找，先判断是否是回文数字，再判断是否是单调不减，大概要花个一分多钟就可以出结果了

**代码**
```java
public class Main {

	public static void main(String[] args) {
		int cnt = 0;
		for (int i = 2022; i <= 2022222022; i++) {
			if (isPalindrome(i) && isMonotonous(i)) {
				cnt++;
			}
		}
		System.out.println(cnt);
	}

	// 判断回文
	public static boolean isPalindrome(int num) {
		String snumString = "" + num;
		StringBuilder stringBuilder = new StringBuilder(snumString);
		return stringBuilder.reverse().toString().equals(snumString);
	}

	// 判断递增不减
	public static boolean isMonotonous(int num) {
		char[] chs = ("" + num).toCharArray();
		int len = chs.length;
		int mid = (len & 1) == 0 ? len >> 1 : (len >> 1) + 1;
		for (int i = 0; i < mid - 1; i++) {
			if (chs[i] > chs[i + 1]) {
				return false;
			}
		}
		return true;

	}

}
```

>输出结果:3138  


**第二种解法**

>同第一种一样也是转化成字符串求解，不过只翻转一半，大概也要花个一分多钟就可以出结果了

代码

```java
public class Main {

	public static void main(String[] args) {
		int cnt = 0;
		for (int i = 2022; i <= 2022222022; i++) {
			if (isPalindromeAndMonotonous(i)) {
				cnt++;
			}
		}
		System.out.println(cnt);

	}

	public static boolean isPalindromeAndMonotonous(int num) {
		char[] chs = ("" + num).toCharArray();
		int start = 0;
		int end = chs.length - 1;
		while (start < end) {
			if (chs[start] != chs[end] || chs[start] > chs[start + 1]) {
				return false;
			}
			start++;
			end--;
		}
		return true;
	}

}
```

>输出结果:3138

**第三种解法**

>也是翻转一半数字，不过是将数字本身翻转，大概要花10多秒就可以出结果了

代码

```java
public class Main {

	public static void main(String[] args) {
		int cnt = 0;
		for (int i = 2022; i <= 2022222022; i++) {
			if (isPalindromeAndMonotonous(i)) {
				cnt++;
			}
		}
		System.out.println(cnt);

	}

	public static boolean isPalindromeAndMonotonous(int num) {
		if (num != 0 && num % 10 == 0 || num < 0) {
			return false;
		}
		int reverseNum = 0;
		while (num > reverseNum) {
			int b = num % 10;
			if (reverseNum % 10 > b) {
				return false;
			}
			reverseNum = reverseNum * 10 + b;
			num /= 10;
		}
		return reverseNum == num || reverseNum / 10 == num;
	}

}
```

>输出结果:3138


**第四种解法**

>根据单调不减和回文的条件，只需要求数字的前一半部分，判断是否单调不减，对于单调不减的有两种符合条件的可能，之后判断这两个回文数字是否在范围内即可,大概要花不到1秒就可以出结果了

代码 1

```java
public class Main {

	public static void main(String[] args) {
		int cnt = 0;
		for (int i = 20; i <= 99999; i++) {
			String numString = "" + i;
			int j = 0, len = numString.length() - 1;
			while (j < len) {
				if (numString.charAt(j) > numString.charAt(j + 1)) {
					break;
				}
				j++;
			}
			if (j == len) {
				StringBuilder sBuilder1 = new StringBuilder(numString);
				String realString1 = numString + sBuilder1.reverse().toString();
				long num = Long.valueOf(realString1);
				if (num >= 2022 && num <= 2022222022) {
					cnt++;
				}
				StringBuilder sBuilder2 = new StringBuilder(numString.substring(0, len));
				String realString2 = numString + sBuilder2.reverse().toString();
				num = Long.valueOf(realString2);
				if (num >= 2022 && num <= 2022222022) {
					cnt++;
				}

			}
		}
		System.out.println(cnt);
	}
}
```
代码 2
```java
public class Main {
    
    public static void main(String[] args) {
        
        int start = 2022;
        int end = 2022222022;

        int cnt = 0;
        String startString = String.valueOf(start);
        String endString = String.valueOf(end);
        int startLength = startString.length();
        int endLength = endString.length();
        int realStart = 0;
        int realEnd = 0;

        for (int i = 0, len = startLength >>> 1; i < len; i++) {
            realStart = realStart * 10 + 1;
        }

        for (int i = 0, len = (endLength >>> 1) + (endLength & 1); i < len; i++) {
            realEnd = realEnd * 10 + 9;
        }

        for (int i = realStart; i <= realEnd; i++) {
            String numString = String.valueOf(i);
            int j = 0, len = numString.length() - 1;
            while (j < len) {
                if (numString.charAt(j) > numString.charAt(j + 1)) {
                    break;
                }
                j++;
            }
            if (j == len) {
                StringBuilder sBuilder1 = new StringBuilder(numString);
                String realString1 = numString + sBuilder1.reverse();
                long num = Long.parseLong(realString1);
                if (i > 0 && num >= start && num <= end) {
                    cnt++;
                }

                StringBuilder sBuilder2 = new StringBuilder(numString.substring(0, len));
                String realString2 = numString + sBuilder2.reverse();
                num = Long.parseLong(realString2);
                if (num >= start && num <= end) {
                    cnt++;
                }
            }
        }
        System.out.println(cnt);
    }

}

```

>输出结果:3138


## 试题 C: 字符统计【编程题】

```
时间限制: 1.0s 内存限制: 512.0MB 本题总分： 10 分
```
 **【问题描述】**

> 给定一个只包含大写字母的字符串S，请你输出其中出现次数最多的字母。如果有多个字母均出现了最多次，按字母表顺序依次输出所有这些字母。

 **【输入格式】**

> 一个只包含大写字母的字符串S.

 **【输出格式】**

> 若干个大写字母，代表答案。

 **【样例输入】**

> BABBACAC

 **【样例输出】**

> AB

 **【评测用例规模与约定】**


>对于 100 %的评测用例， 1 ≤|S|≤10${^6}$

**第一种解法**


代码

```java
import java.util.Scanner;

public class Main {

	public static void main(String[] args) {
		Scanner scanner = new Scanner(System.in);
		char[] chs = scanner.nextLine().toCharArray();
		int[] counts = new int[26];
		for (int i = 0; i < chs.length; i++) {
			counts[chs[i] - 'A'] += 1;
		}
		int max = counts[0];
		String result = "A";
		for (int i = 1; i < counts.length; i++) {
			if (counts[i] > max) {
				max = counts[i];
				result = "" + (char) (i + 'A');
			} else if (counts[i] == max) {
				result += (char) (i + 'A');
			}
		}
		System.out.println(result);
	}

}
```

结果

![](https://jsdelivr.codeqihan.com/gh/MysticalDream/images/assets/202311122103489.png)




## 试题 D: 最少刷题数【编程题】

```
时间限制: 1.0s 内存限制: 512.0MB 本题总分： 10 分
```
 **【问题描述】**

>小蓝老师教的编程课有N名学生，编号依次是 1...N。第i号学生这学期
刷题的数量是A$_i$。
>
>对于每一名学生，请你计算他至少还要再刷多少道题，才能使得全班刷题
比他多的学生数不超过刷题比他少的学生数。

 **【输入格式】**

> 第一行包含一个正整数N
> 第二行包含N个整数：A$_1$,A$_2$,A$_3$,...A$_N$.

 **【输出格式】**

> 输出N个整数，依次表示第 1...N号学生分别至少还要再刷多少道题。

**【样例输入】**

> 5 
> 12 10 15 20 6

 **【样例输出】**

> 0 3 0 0 7

 **【评测用例规模与约定】**


>对于 30 %的数据， 1$\leq$N $\leq$ 1000 ,0 $\leq$A${_i}$ $\leq$ 1000 :
对于 100 %的数据， 1$\leq$N$\leq$ 100000 , 0 $\leq$ A${_i}$ $\leq$ 100000 :

**简略说一下思路**
> 大致思路是找到已经有序的刷题数序列的处于中间的值，求刷题数和中值的差值，然后再根据情况调整刷题的数量。
> 
> 比如样例的排序后的序列是`6 10 12 15 20`，处于中间的是`12`，那么左边不是`12`的是`6 10`，右边不是`12`的是`15 20`，由于两边数量是相等的，对于左边的如果仅仅是刷题达到中间值的数量，会导致刷题比他多的学生数超过刷题比他少的学生数。比如`6和12`的差值是`6`，假如刷题数为`6`的学生再刷`6`题后，序列就会变成`10 12 12 15 20`，可以看到刷题比`12`多的学生`{15 20}`比刷题比`12`少的学生`{10}`多，不满足题意，这个时候只需要再刷一道就可以满足，即刷`6`题边`6+1`为`7`，序列变成`10 12 13 15 20`,这个序列就满足全班刷题比他多的学生数`不超过`刷题比他少的学生数。对于处于中间的`12`，由于左右两边数量相等，所以满足题意，不用再多刷题。对于右边的肯定是满足题意的，所以也不用刷题。
> 
> 再举一个例子，对于一个已经排序的序列`6 12 12 15 20`，处于中间的值是`12`，左边不是`12`的是`6`，右边不是`12`的是`15 20`。对于左边的仅仅刷题达到中间的值的数量，依旧不满足题意，和前一个例子一样需要刷的题数比与中间值的差值多一题。对于中间的`12`，由于左边的数量少于右边的数量，所以需要多刷一道题才能满足。对于右边的依旧不需要刷题。
> 
> 其实总的来说就是找到左右两边数量的大小关系，如果左边的小于或等于右边的数量，左边的刷题数需要是与中间值的差值加一，如果左边的小于右边的数量，中间值需要多刷一道。
> 
> 第一种解法中找中间的值的两边与中值相等的数量其实就是体现出了左右两边的数量关系。
> 
**第一种解法**

代码

```java
import java.util.Arrays;
import java.util.Scanner;

public class Main {

	public static void main(String[] args) {
		Scanner scanner = new Scanner(System.in);
		int n = scanner.nextInt();
		int[] nums = new int[n];
		int[] temp = new int[n];
		for (int i = 0; i < n; i++) {
			nums[i] = scanner.nextInt();
		}
		// 复制数组
		System.arraycopy(nums, 0, temp, 0, nums.length);
		// 排序数组
		Arrays.sort(temp);
		// 中间的下标
		int midIndex = n / 2;
		// 中间的值
		int midValue = temp[midIndex];
		// 
		int midOption = 0;
		//
		int option = 0;
		// 左边和中值相同值的数量
		int sameLeft = 0;
		// 右边和中值相同值的数量
		int sameRight = 0;

		for (int i = midIndex - 1, j = midIndex +(n & 1); i >= 0; i--, j++) {
			if (temp[i] == midValue) {
				sameLeft++;
			}
			if (temp[j] == midValue) {
				sameRight++;
			}
			if (temp[i] != temp[j]) {
				break;
			}
		}

		if (sameLeft >= sameRight) {
			option = 1;
		}
		if (sameLeft > sameRight) {
			midOption = 1;
		}

		for (int i = 0, len = nums.length; i < len; i++) {
			int count = 0;
			if (nums[i] == midValue) {
				count = midOption;
			} else {
				count = midValue - nums[i] + option;
				if (count < 0) {
					count = 0;
				}
			}
			if (i != n - 1) {
				System.out.print(count + " ");
			} else {
				System.out.println(count);
			}

		}
		scanner.close();

	}

}
```

结果（部分超时）


![](https://jsdelivr.codeqihan.com/gh/MysticalDream/images/assets/202311122104739.png)

**第二种解法**

代码

```java
import java.util.Arrays;
import java.util.Scanner;

public class Main {

    public static void main(String[] args) {
        Scanner scanner = new Scanner(System.in);
        int n = scanner.nextInt();
        int[] nums = new int[n];
        int[] temp = new int[n];
        for (int i = 0; i < n; i++) {
            nums[i] = scanner.nextInt();
        }
        // 复制数组
        System.arraycopy(nums, 0, temp, 0, nums.length);
        // 排序数组
        Arrays.sort(temp);
        // 中间的下标
        int midIndex = n >>> 1;
        // 中间的值
        int midValue = temp[midIndex];
        //
        int midOption = 0;
        //
        int option = 0;
        // 左边和中值相同值的数量
        int sameLeft = 0;
        // 右边和中值相同值的数量
        int sameRight = 0;

        int left = left(temp, 0, midIndex - 1, midValue);

        int right = right(temp, midIndex + (n & 1), n - 1, midValue);

        if (left != -1) {
            sameLeft = midIndex - left;
        }

        if (right != -1) {
            sameRight = 1 + right - midIndex;
        }

        if (sameLeft >= sameRight) {
            option = 1;
        }
        if (sameLeft > sameRight) {
            midOption = 1;
        }

        for (int i = 0, len = nums.length; i < len; i++) {
            int count = 0;
            if (nums[i] == midValue) {
                count = midOption;
            } else {
                count = midValue - nums[i] + option;
                if (count < 0) {
                    count = 0;
                }
            }
            if (i != n - 1) {
                System.out.print(count + " ");
            } else {
                System.out.println(count);
            }

        }
        scanner.close();

    }

    public static int left(int[] arr, int start, int end, int target) {

        int index = -1;

        while (start <= end) {

            int mid = start + ((end - start) >>> 1);
            if (arr[mid] >= target) {
                end = mid - 1;
                if (arr[mid] == target) {
                    index = mid;
                }
            } else {
                start = mid + 1;
            }
        }

        return index;
    }

    public static int right(int[] arr, int start, int end, int target) {
        int index = -1;
        while (start <= end) {
            int mid = start + ((end - start) >>> 1);
            if (arr[mid] > target) {
                end = mid - 1;
            } else {
                start = mid + 1;
                if (arr[mid] == target) {
                    index = mid;
                }
            }
        }
        return index;
    }

}

```
>感觉还是会超时，不过OJ运行过了，可能是巧合

结果


![](https://jsdelivr.codeqihan.com/gh/MysticalDream/images/assets/202311122104371.png)
**第三种解法**

>无需全部排序，通过选择算法找出中间值

代码
```java
import java.util.Scanner;

public class Main {

    public static void main(String[] args) {

        Scanner scanner = new Scanner(System.in);

        int n = scanner.nextInt();

        int[] nums = new int[n];
        int[] temp = new int[n];

        for (int i = 0; i < n; i++) {
            nums[i] = scanner.nextInt();
        }

        // 复制数组
        System.arraycopy(nums, 0, temp, 0, nums.length);

        //快速选择
        int midValue = select(temp, (n + 2) >>> 1);

        int l = 0, r = 0;

        for (int i = 0; i < n; i++) {
            if (nums[i] > midValue) {
                r++;
            } else if (nums[i] < midValue) {
                l++;
            }
        }

        int midOption = l < r ? 1 : 0;

        int option = l <= r ? 1 : 0;

        for (int i = 0; i < n; i++) {
            int count = 0;
            if (nums[i] == midValue) {
                count = midOption;
            } else if (nums[i] < midValue) {
                count = midValue - nums[i] + option;
            }

            if (i != n - 1) {
                System.out.print(count + " ");
            } else {
                System.out.println(count);
            }
        }

    }

    public static int select(int[] arr, int k) {
        return select0(arr, 0, arr.length - 1, k);
    }


    /**
     * 快速选择
     */
    private static int select0(int[] arr, int left, int right, int k) {
        if (left == right) {
            return arr[left];
        }

        int mid = partition(arr, left, right);

        int res = mid - left + 1;

        if (k == res) {
            return arr[mid];
        } else if (k < res) {
            return select0(arr, left, mid - 1, k);
        } else {
            return select0(arr, mid + 1, right, k - res);
        }
    }


    /**
     * 基于随机选取轴的划分
     */
    private static int partition(int[] arr, int l, int r) {


        int pivotIndex = (int) (Math.random() * (r - l + 1)) + l;

        arr[l] = new int[]{arr[pivotIndex], arr[pivotIndex] = arr[l]}[0];

        int pivot = arr[l], left = l, right = r + 1;

        while (left < right) {
            while (arr[++left] < pivot && left < r) {
            }

            while (arr[--right] > pivot) {

            }

            if (left >= right) {
                break;
            }

            arr[left] = new int[]{arr[right], arr[right] = arr[left]}[0];
        }
        arr[l] = arr[right];

        arr[right] = pivot;

        return right;
    }

}

```
结果

![](https://jsdelivr.codeqihan.com/gh/MysticalDream/images/assets/202311122104357.png)


## 试题 E: 求阶乘【编程题】

```
时间限制: 1.0s 内存限制: 512.0MB 本题总分： 15 分
```
**【问题描述】**

> 满足N!的末尾恰好有K个 0 的最小的N是多少?
> 如果这样的N不存在输出-1 。

**【输入格式】**

> 一个整数K。

 **【输出格式】**

>一个整数代表答案。

 **【样例输入】**

> 2

 **【样例输出】**

> 10

 **【评测用例规模与约定】**

> 对于 30 %的数据， 1 $\leq$ K $\leq$ 10${^6}$.
> 对于 100 %的数据， 1 $\leq$K $\leq$ 10$^{18}$.

**tips**
>在leetcode上也有一个类似的hard难度的题目[https://leetcode.cn/problems/preimage-size-of-factorial-zeroes-function/solution/jie-cheng-han-shu-hou-k-ge-ling-by-leetc-n6vj/](https://leetcode.cn/problems/preimage-size-of-factorial-zeroes-function/solution/jie-cheng-han-shu-hou-k-ge-ling-by-leetc-n6vj/)

**第一种解法**
>暴力查找

代码
```java
import java.util.Scanner;

public class Main {
	public static void main(String args[]) {
		Scanner scanner = new Scanner(System.in);
		long k = scanner.nextLong();
		long count;
		long a = 5;
		while (true) {
			long tempA = a;
			count = 0;
			while (tempA > 0) {
				tempA /= 5;
				count += tempA;
			}
			if (count < k) {
				a += 5;
			} else if (count == k) {
				System.out.println(a);
				break;
			} else {
				System.out.println(-1);
				break;
			}
		}
		scanner.close();
	}
}
```

结果（部分超时）

![](https://jsdelivr.codeqihan.com/gh/MysticalDream/images/assets/202311122104858.png)


**第二种解法**

>二分查找法，10$^{18}$对应的值可以多尝试几个就可以找到了，可以根据45、405、4005这样的大致范围找

代码

```java
import java.util.Scanner;

public class Main {

	public static void main(String[] args) {
		
		try (Scanner scanner = new Scanner(System.in)) {
			long k = scanner.nextLong();
			//末尾有1个零
			long start = 5;
			//末尾有10^18个零
			long end = 4000000000000000020L;
			//二分查找法
			while (start <= end) {
				long mid = start + ((end - start) >> 1);
				long midRes = trailingZeroes(mid);
				if (midRes > k) {
					end = mid - 1;
				} else if (midRes < k) {
					start = mid + 1;
				} else {
					while (mid % 5 != 0) {
						mid--;
					}
					System.out.println(mid);
					return;
				}
			}
			System.out.println(-1);
		}
	}

	public static long trailingZeroes(long n) {
		long res = 0;
		n /= 5;
		while (n > 0) {
			res += n;
			n /= 5;
		}
		return res;
	}

}
```
结果

![](https://jsdelivr.codeqihan.com/gh/MysticalDream/images/assets/202311122104037.png)

**第二种解法改进**

代码
```java
import java.util.Scanner;

public class Main {

    public static void main(String[] args) {

        try (Scanner scanner = new Scanner(System.in)) {
            long k = scanner.nextLong();
            long start = 4 * k;
            long end = 5 * k;
            //二分查找法
            while (start <= end) {
                long mid = start + ((end - start) >> 1);
                long midRes = trailingZeroes(mid);
                if (midRes > k) {
                    end = mid - 1;
                } else if (midRes < k) {
                    start = mid + 1;
                } else {
                    while (mid % 5 != 0) {
                        mid--;
                    }
                    System.out.println(mid);
                    return;
                }
            }
            System.out.println(-1);
        }
    }

    public static long trailingZeroes(long n) {
        long res = 0;
        n /= 5;
        while (n > 0) {
            res += n;
            n /= 5;
        }
        return res;
    }

}

```

结果


![](https://jsdelivr.codeqihan.com/gh/MysticalDream/images/assets/202311122105502.png)

## 试题 F: 最大子矩阵【编程题】

```
时间限制: 1.0s 内存限制: 512.0MB 本题总分： 15 分
```
 **【问题描述】**

> 小明有一个大小为N$\times$M的矩阵，可以理解为一个N行M列的二维数组。
>
>我们定义一个矩阵m的稳定度 f(m)为 f(m) =max(m)$-$min(m)，其中max(m)
表示矩阵m中的最大值，min(m)表示矩阵m中的最小值。现在小明想要从这
个矩阵中找到一个稳定度不大于limit的子矩阵，同时他还希望这个子矩阵的面
积越大越好（面积可以理解为矩阵中元素个数）。
>
>子矩阵定义如下：从原矩阵中选择一组连续的行和一组连续的列，这些行
列交点上的元素组成的矩阵即为一个子矩阵。

 **【输入格式】**

> 第一行输入两个整数N，M，表示矩阵的大小。
>
> 接下来N行，每行输入M个整数，表示这个矩阵。
最后一行输入一个整数limit，表示限制。

 **【输出格式】**

> 输出一个整数，分别表示小明选择的子矩阵的最大面积。

 **【样例输入】**

> 3 4  
>
> 2 0 7 9  
> 0 6 9 7  
> 8 4 6 4  
> 8

 **【样例输出】**

> 6



 **【样例说明】**

> 满足稳定度不大于 8 的且面积最大的子矩阵总共有三个，他们的面积都是 6 （**粗体**表示子矩阵元素）：

> **2 0** 7 9  
**0 6** 9 7  
 **8 4** 6 4

>2 0 **7 9**  
 0 6 **9 7**  
 8 4 **6 4**

> 2 0 7 9  
 0 **6 9 7**  
 8 **4 6 4**

 **【评测用例规模与约定】** 


![](https://jsdelivr.codeqihan.com/gh/MysticalDream/images/assets/202311122105361.png)




## 试题 G: 数组切分【编程题】

```
时间限制: 1.0s 内存限制: 512.0MB 本题总分： 20 分
```
**【问题描述】**

>已知一个长度为N的数组：A ${_1}$,A ${_2}$ ,A${_3}$ ...A${_N}$ 恰好是 1~N的一个排列。现
> 在要求你将A数组切分成若干个(最少一个，最多N个)连续的子数组，并且
>每个子数组中包含的整数恰好可以组成一段连续的自然数。

>例如对于A={1,3,2,4},一共有 5 种切分方法：

![](https://jsdelivr.codeqihan.com/gh/MysticalDream/images/assets/202311122105112.png)

**【输入格式】**

> 第一行包含一个整数N。第二行包含N个整数，代表A数组。

**【输出格式】**

> 输出一个整数表示答案。由于答案可能很大，所以输出其对 1000000007 取模后的值

**【样例输入】**

>4 
> 1 3 2 4

**【样例输出】**

> 5


**【评测用例规模与约定】**

> 对于 30 %评测用例， 1$\leq$N$\leq$ 20.
对于 100 %评测用例， 1 $\leq$N$\leq$ 10000 .



## 试题 H: 回忆迷宫【编程题】

```
时间限制: 1.0s 内存限制: 512.0MB 本题总分： 20 分
```
**【问题描述】**

>爱丽丝刚从一处地下迷宫中探险归来，你能根据她对于自己行动路径的回忆，帮她画出迷宫地图吗？
>
>迷宫地图是基于二维网格的。爱丽丝会告诉你一系列她在迷宫中的移动步骤，每个移动步骤可能是上下左右四个方向中的一种，表示爱丽丝往这个方向
>
> 走了一格。你需要根据这些移动步骤给出一个迷宫地图，并满足以下条件：

>1、爱丽丝能在迷宫内的某个空地开始，顺利的走完她回忆的所有移动步骤。  
> 2 、迷宫内不存在爱丽丝没有走过的空地。  
> 3 、迷宫是封闭的，即可通过墙分隔迷宫内与迷宫外。任意方向的无穷远处
>视为迷宫外，所有不与迷宫外联通的空地都视为是迷宫内。（迷宫地图为四联通，即只有上下左右视为联通）  
>4 、在满足前面三点的前提下，迷宫的墙的数量要尽可能少。

**【输入格式】**

>第一行一个正整数N，表示爱丽丝回忆的步骤数量。
>接下来一行N个英文字符，仅包含 **UDLR** 四种字符，分别表示上（Up）、
下（Down）、左（Left）、右（Right）。

**【输出格式】**

> 请通过字符画的形式输出迷宫地图。迷宫地图可能包含许多行，用字符‘*’ 表示墙，用‘ ’（空格）表示非墙。
>你的输出需要保证以下条件：  
>1 、至少有一行第一个字符为`‘*’`。  
 2 、第一行至少有一个字符为`‘*’`。  
3 、每一行的最后一个字符为`‘*’`。  
4 、最后一行至少有一个字符为`‘*’`。

**【样例输入】**

>17
> UUUULLLLDDDDRRRRU

**【样例输出】**
```
 *****
*     *
* *** *
* *** *
* *** *
*     *
 *****
```
**【样例说明】**
> 爱丽丝可以把第六行第六个字符作为起点。
> 
>外墙墙墙墙墙外  
>墙内内内内内墙  
>墙内墙墙墙内墙  
>墙内墙墙墙内墙  
>墙内墙墙墙内墙  
>墙内内内内内墙  
>外墙墙墙墙墙外

**【评测用例规模与约定】**

>对于所有数据， 0 <N$\leq$ 100.



## 试题 I: 红绿灯【编程题】

```
时间限制: 1.0s 内存限制: 512.0MB 本题总分： 25 分
```
**【问题描述】**

>&nbsp; &nbsp;&nbsp;&nbsp;爱丽丝要开车去上班，上班的路上有许多红绿灯，这让爱丽丝很难过。为了上班不迟到，她给自己的车安装了氮气喷射装置。现在她想知道自己上班最短需要多少时间。
>
>&nbsp;&nbsp;&nbsp;&nbsp;爱丽丝的车最高速度是$\frac{1}{V}$米每秒，并且经过改装后，可以瞬间加速到小于等于最高速的任意速度，也可以瞬间停止。
>
>&nbsp;&nbsp;&nbsp;&nbsp;爱丽丝家离公司有N米远，路上有M个红绿灯，第i个红绿灯位于离爱丽丝家A${_i}$米远的位置，绿灯持续B${_i}$秒，红灯持续C${_i}$秒。在初始时（爱丽丝开始计时的瞬间），所有红绿灯都恰好从红灯变为绿灯。如果爱丽丝在绿灯变红的瞬间到达红绿灯，她会停下车等红灯，因为她是遵纪守法的好市民。
>
>&nbsp;&nbsp;&nbsp;&nbsp;氮气喷射装置可以让爱丽丝的车瞬间加速到超光速（且不受相对论效应的影响！），达到瞬移的效果，但是爱丽丝是遵纪守法的好市民，在每个红绿灯前她都会停下氮气喷射，即使是绿灯，因为红绿灯处有斑马线，而使用氮气喷射装置通过斑马线是违法的。此外，氮气喷射装置不能连续启动，需要一定时间的冷却，表现为通过K个红绿灯后才能再次使用。（也就是说，如果K= 1，就能一直使用啦！）初始时，氮气喷射装置处于可用状态。

**【输入格式】**

> 第一行四个正整数N、M、K、V，含义如题面所述。
接下来M行，每行三个正整数A${_i}$、B${_i}$、C${_i}$，含义如题面所述。

**【输出格式】**

>输出一个正整数T，表示爱丽丝到达公司最短需要多少秒。

**【样例输入】**

>90 2 2 2  
>30 20 20  
>60 20 20

**【样例输出】**

>80

**【样例说明】**
 >&nbsp;&nbsp;&nbsp;&nbsp;爱丽丝在最开始直接使用氮气喷射装置瞬间到达第一个红绿灯，然后绿灯通过，以最高速行进 60 秒后到达第二个红绿灯，此时绿灯刚好变红，于是她等待 20 秒再次变为绿灯后通过该红绿灯，此时氮气喷射装置冷却完毕，爱丽丝再次使用瞬间到达公司，总共用时 80 秒。

**【评测用例规模与约定】**


![](https://jsdelivr.codeqihan.com/gh/MysticalDream/images/assets/202311122106272.png)




## 试题 J: 拉箱子【编程题】

```
时间限制: 1.0s 内存限制: 1.0GB 本题总分： 25 分
```
**【问题描述】**

>&nbsp;&nbsp;&nbsp;&nbsp;推箱子是一款经典电子游戏，爱丽丝很喜欢玩，但是她有点玩腻了，现在她想设计一款拉箱子游戏。
>
>&nbsp;&nbsp;&nbsp;&nbsp;拉箱子游戏需要玩家在一个N${\times}$M的网格地图中，控制小人上下左右移动，将箱子拉到终点以获得胜利。
>
>&nbsp;&nbsp;&nbsp;&nbsp;现在爱丽丝想知道，在给定地形（即所有墙的位置）的情况下，有多少种不同的可解的初始局面。

>&nbsp;&nbsp;&nbsp;&nbsp;【初始局面】的定义如下：  
> 1 、初始局面由排列成N${\times}$M矩形网格状的各种元素组成，每个网格中有
> 且只有一种元素。可能的元素有：空地、墙、小人、箱子、终点。
>
> 2 、初始局面中有且只有一个小人。  
>3 、初始局面中有且只有一个箱子。  
>4 、初始局面中有且只有一个终点。  
>&nbsp;&nbsp;&nbsp;&nbsp;【可解】的定义如下：  
> 通过有限次数的移动小人（可以在移动的同时拉箱子），箱子能够到达终点
> 所在的网格。  
>&nbsp;&nbsp;&nbsp;&nbsp;【移动】的定义如下：    
> 在一次移动中，小人可以移动到相邻（上、下、左、右四种选项）的一个网格中，前提是满足以下条件：  
>1 、小人永远不能移动到N${\times}$M的网格外部。  
>2 、小人永远不能移动到墙上或是箱子上。  
> 3 、小人可以移动到空地或是终点上。
>
>&nbsp;&nbsp;&nbsp;&nbsp;【拉箱子】的定义如下：  
>
>在一次合法移动的同时，如果小人初始所在网格沿小人移动方向的反方向上的相邻网格上恰好是箱子，小人可以拉动箱子一起移动，让箱子移动到小人初始所在网格。
>
>&nbsp;&nbsp;&nbsp;&nbsp;即使满足条件，小人也可以只移动而不拉箱子。

**【输入格式】**

>第一行两个正整数N和M，表示网格的大小。
>
> 接下来N行，每行M个由空格隔开的整数 0 或 1 描述给定的地形。其中
> 1 表示墙， 0 表示未知的元素，未知元素可能是小人或箱子或空地或终点，但不能是墙。

**【输出格式】**

>输出一个正整数，表示可解的初始局面数量。

**【样例输入】**

> 2 4  
> 0 0 0 0  
>1 1 1 0  

**【样例输出】**

>13

**【样例说明】**

> 13 种可解的初始局面示意图如下：

>人终箱空  
> 墙墙墙空
>********  
>人终空箱  
>墙墙墙空
>********  
>人空终箱  
> 墙墙墙空
>********  
>箱人终空  
> 墙墙墙空
> ********  
> 空人终箱  
>墙墙墙空
>********  
> 箱终人空  
> 墙墙墙空
>********  
>空终人箱  
>墙墙墙空
>********  
>箱终空人  
> 墙墙墙空
> ********  
>箱空终人  
>墙墙墙空
>********  
>空箱终人  
>墙墙墙空
> ********  
>箱终空空  
>墙墙墙人
>********  
> 箱空终空  
> 墙墙墙人
>********  
>空箱终空  
>墙墙墙人



**【评测用例规模与约定】**

>对于 30 %的数据，N,M$\leq$ 3.
> 对于 100 %的数据， 0 <N,M$\leq$ 10.

以上代码内容仅供参考，不一定正确


