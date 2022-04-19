# Tricks

#### [1996. 游戏中弱角色的数量](https://leetcode-cn.com/problems/the-number-of-weak-characters-in-the-game/)

对于一个由游戏角色攻击力和防御力组成的数组，`int p[][]`。使用分组排序的方法确定角色的正确排序。

```java
Arrays.sort(p, (o1, o2) -> o1[0] == o2[0] ? o2[1] - o1[1] : o1[0] - o2[0]);
```

