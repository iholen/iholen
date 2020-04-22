---
layout: post
title:  "数据结构之哈夫曼树自实现"
date:   2020-04-22 21:43:05 +0800
categories: algorithm
author: iholen
introduction: 数据结构之哈夫曼树自实现
---
### 最优二叉树(哈夫曼树)
哈夫曼树是带权路径长度最短的树，权值较大的结点离根较近。

* 给定一组带权的叶子节点
```java
["a", "b", "c", "d", "e", "f", "g"]
```
* 对应的权值
```java
[5, 3, 8, 1, 4, 7, 2]
```
* 代码实现
```java
public class HuffmanTree {

    public static void main(String[] args) {
        String[] values = {"a", "b", "c", "d", "e", "f", "g"};
        int[] weights = {5, 3, 8, 1, 4, 7, 2};

        Node[] nodes = new Node[values.length];
        PriorityQueue<Node> queue = new PriorityQueue<>();
        for (int i = 0; i < values.length; i++) {
            Node node = new Node(values[i], weights[i], null, null, null);
            queue.offer(node);
            nodes[i] = node;
        }
        while (queue.size() > 1) {
            Node left = queue.poll();
            Node right = queue.poll();
            Node parent = new Node(null, left.getWeight() + right.getWeight(), left, right, null);
            left.setParent(parent);
            right.setParent(parent);
            queue.offer(parent);
        }
        Node head = queue.poll();
    }

}

class Node implements Comparable<Node> {

    private String value;
    private int weight;
    private Node left;
    private Node right;
    private Node parent;

    public Node(String value, int weight, Node left, Node right, Node parent) {
        this.value = value;
        this.weight = weight;
        this.left = left;
        this.right = right;
        this.parent = parent;
    }

    public String getValue() {
        return value;
    }

    public void setValue(String value) {
        this.value = value;
    }

    public int getWeight() {
        return weight;
    }

    public void setWeight(int weight) {
        this.weight = weight;
    }

    public Node getLeft() {
        return left;
    }

    public void setLeft(Node left) {
        this.left = left;
    }

    public Node getRight() {
        return right;
    }

    public void setRight(Node right) {
        this.right = right;
    }

    public Node getParent() {
        return parent;
    }

    public void setParent(Node parent) {
        this.parent = parent;
    }

    @Override
    public int compareTo(Node o) {
        return weight - o.weight;
    }

}
```
* 构建后，树的结构

![Huffman](/iholen/assets/images/huffman.jpg)
