---
title: "Alg 二叉树的最近公共祖先"
date: 2020-11-22T21:15:00+08:00
tags:
- 算法
categories: 
- 算法
---

## 二叉树的最近公共祖先

### 题目描述

给定一个二叉树, 找到该树中两个指定节点的最近公共祖先。

百度百科中最近公共祖先的定义为：“对于有根树 T 的两个结点 p、q，最近公共祖先表示为一个结点 x，满足 x 是 p、q 的祖先且 x 的深度尽可能大（一个节点也可以是它自己的祖先）。”

<!--more-->

例如，给定如下二叉树: root = [3,5,1,6,2,0,8,null,null,7,4]

![](https://assets.leetcode-cn.com/aliyun-lc-upload/uploads/2018/12/15/binarytree.png)

来源：力扣（LeetCode）
链接：https://leetcode-cn.com/problems/lowest-common-ancestor-of-a-binary-tree
著作权归领扣网络所有。商业转载请联系官方授权，非商业转载请注明出处。

**说明：**

- 所有节点的值都是唯一的。
- p、q 为不同节点且均存在于给定的二叉树中。

### 思路

#### 思路一：

题目说明提出p、q为不同节点，且一定存在于二叉树中，那么从root节点开始，一定能找到一条到p或q节点的路径。

因此对树做个遍历，找到root->p和root->q的路径，再对这两个路径求最后一个公共点，就是最近公共祖先

#### code

```go
type TreeNode struct {
	Val   int
	Left  *TreeNode
	Right *TreeNode
}

type res struct {
	val  *TreeNode
	path []*TreeNode
	ok   bool
}

func lowestCommonAncestor(root, p, q *TreeNode) *TreeNode {
	r1, r2 := res{
		val:  p,
		path: make([]*TreeNode, 0),
		ok:   false,
	}, res{
		val:  q,
		path: make([]*TreeNode, 0),
		ok:   false,
	}
	findPath(root, &r1, &r2)
	i, j := 0, 0
	for ; i < len(r1.path) && j < len(r2.path); i, j = i+1, j+1 {
		if r1.path[i] != r2.path[j] {
			break
		}
	}
	return r1.path[i-1]
}

func findPath(root *TreeNode, r1, r2 *res) {
	if root == nil {
		return
	}
	if !r1.ok {
		r1.path = append(r1.path, root)
	}
	if !r2.ok {
		r2.path = append(r2.path, root)
	}
	if root == r1.val {
		r1.ok = true
	}
	if root == r2.val {
		r2.ok = true
	}
	findPath(root.Left, r1, r2)
	findPath(root.Right, r1, r2)
	if !r1.ok {
		r1.path = r1.path[:len(r1.path)-1]
	}
	if !r2.ok {
		r2.path = r2.path[:len(r2.path)-1]
	}
}
```

#### 思路二：

题目说明提出p、q为不同节点，且一定存在于二叉树中。

那么从root节点开始，分为四种情况，

1. 其中一个节点为root节点，那么直接返回root
2. 两个节点都在左子树，那么将左孩子作为root，递归查找
3. 两个节点都在右子树，那么将右孩子作为root，递归查找
4. 两个节点分别在左子树和右子树，那么结果为root

对于题目中的样例，

- 情况一：假设p、q分别为5，1，那么对于root-3，我们能在左子树找到节点p，能在右子树找到节点q，因此结果就是root
- 情况二：假设p、q分别为6，2，那么对于root-3，我们只能在左子树找到结果，右子树为nil，因此结果一定在左子树的节点上，继续以节点5为root来看，回到了一的情况中，能在左子树和右子树找到结果，因此结果为节点5
- 情况三：假设p、q分别为0，8，那么对于root-3，我们只能在右子树找到结果，左子树为nil，因此结果一定在右子树的节点上，继续以节点1为root来看，回到了一的情况中，能在左子树和右子树找到结果，因此结果为节点1

总结来说，对于目标节点p和q，以某一节点r为起点，如果我们在左子树和右子树分别找到p和q，那么节点r就是最近公共祖先，如果我们只在某一个子树找到两个节点，则表示当前节点一定不是最近公共祖先，而是在子树中，因此递归往下查找。

#### code

```go
func lowestCommonAncestor(root, p, q *TreeNode) *TreeNode {
	if root == nil {
		return nil
	}
	// 当前节点等于p或q，表示找到了目标节点
	if root == p || root == q {
		return root
	}
	// 尝试在左子树和右子树中，查找目标节点，如果找不到就返回nil
	left := lowestCommonAncestor(root.Left, p, q)
	right := lowestCommonAncestor(root.Right, p, q)
	// left和right都不为nil，表示分别在左子树和右子树中找到了节点，因此当前节点就是最近公共祖先。
	if left != nil && right != nil {
		return root
	}
	// right为nil，表示两个节点都在左子树找到，因此最近公共祖先在左子树，而不是当前节点。
	if left != nil {
		return left
	}
	return right
}
```

