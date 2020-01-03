# 算法

#### 给定一个字符串，请你找出其中不含有重复字符的 最长子串 的长度。

**示例 1:**

```
输入: "abcabcbb"
输出: 3 
解释: 因为无重复字符的最长子串是 "abc"，所以其长度为 3。
```

**示例 2:**

```
输入: "bbbbb"
输出: 1
解释: 因为无重复字符的最长子串是 "b"，所以其长度为 1。
```

**示例 3:**

```
输入: "pwwkew"
输出: 3
解释: 因为无重复字符的最长子串是 "wke"，所以其长度为 3。
     请注意，你的答案必须是 子串 的长度，"pwke" 是一个子序列，不是子串。
```

## 题目地址

> https://leetcode-cn.com/problems/longest-substring-without-repeating-characters/


## 解题思路

1. 声明两个指针初始分别指向字符串的第一个值，start：0，end：0
2. 声明一个对象：map
3. 判断当前元素是否存在于map对象中，不存在则加入`map['a'] = i`，存在则计算start指针是否需要移动
4. 取历史最大的start。`因为start指针的移动是基于元素出现的位置索引+1来移动的`.需要控制start指针不能再向后移动.
    如果无脑移动start指针位置.如字符串:`abba`

## 答案

```javascript
var lengthOfLongestSubstring = function (s) {
    let start = 0;
    let max = 0;
    let len = s.length;
    let map = {};
    for (let end = 0; end < len; end++) {
        let c = s[end];
        if (map[c] >= 0) {
            start = Math.max(start, map[c] + 1);
        }

        map[c] = end;
        max = Math.max(max, end - start + 1);
    }

    return max;
};
```