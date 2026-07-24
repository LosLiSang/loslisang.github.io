---
title: Leetcode 3144. 分割字符频率相等的最少子字符串
date: 2024-08-28T16:00:30+08:00
updated:
tags: [LeetCode, 动态规划, 前缀和]
categories: [技术, LeetCode]
keywords: []
description:
top_img:
comments:
cover:
toc:
---

> 给你一个字符串 s ，你需要将它分割成一个或者更多的 平衡 子字符串。比方说，s == "ababcc" 那么 ("abab", "c", "c") ，("ab", "abc", "c") 和 ("ababcc") 都是合法分割，但是 ("a", "bab", "cc") ，("aba", "bc", "c") 和 ("ab", "abcc") 不是，不平衡的子字符串用粗体表示。
> 请你返回 s 最少 能分割成多少个平衡子字符串。

样例:
> 输入：s = "fabccddg"
> 输出：3
> 解释：
> 我们可以将 s 分割成 3 个子字符串：("fab, "ccdd", "g") 或者 ("fabc", "cd", "dg") 。

按照动态规划的思路来看, 把问题分割成一个个的子问题

假设dp[i] 表示s[i..]平衡子字符串的最少个数, 那么dp[i-1] = min(dp[i] + 1, if 可以得到平衡子字符串 dp[i+1], dp[i+2], .. dp[n])

问题在于如何判断s[i..j] 是否是一个平衡子字符串

实现一个check函数

```rust
fn check(chars: &Vec<char>, i: usize, j: usize) -> bool{
    let mut cnt = HashMap::new();
    for k in i..j{
        *cnt.entry(chars[k]).or_insert(0) += 1;
    }
    cnt.values().max() == cnt.values().min()
}
```

代码

```rust
use std::collections::{HashMap};

pub struct Solution;
impl Solution {
    pub fn minimum_substrings_in_partition(s: String) -> i32 {
        fn check(chars: &Vec<char>, i: usize, j: usize) -> bool{
            let mut cnt = HashMap::new();
            for k in i..j{
                *cnt.entry(chars[k]).or_insert(0) += 1;
            }
            cnt.values().max() == cnt.values().min()
        }
        
        let len = s.len(); 
        let mut dp:Vec<i32> = (0..len as i32 +1).collect();
        
        let chars: Vec<char> = s.chars().collect();
        for i in 1..len+1{
            dp[i] = dp[i].min(dp[i-1] + 1);
            for j in 0..i{
                if check(&chars, i, j){
                    dp[i] = dp[i].min(dp[j]);
                }
            }
        }
        dp[len]
    }
}

```

但是使用check函数会超时

进一步优化

```rust

use std::collections::HashMap;
impl Solution {
    pub fn minimum_substrings_in_partition(s: String) -> i32 {
        let len = s.len();
        let mut dp: Vec<i32> = (0..len as i32 + 1).collect();
        let chars: Vec<char> = s.chars().collect();
        for i in 1..len {
            let mut cnt = HashMap::new();
            for j in (0..i+1).rev() {
                *cnt.entry(chars[j]).or_insert(0) += 1;
                if cnt.values().max() == cnt.values().min() {
                    dp[i+1] = dp[i+1].min(dp[j] + 1);
                }
            }
        }
        dp[len]
    }
}

```
