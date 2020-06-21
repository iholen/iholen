---
layout: post
title:  "算法题-TopK问题"
date:   2020-04-23 22:43:05 +0800
categories: algorithm
author: iholen
introduction: TopK问题
---
### TopK问题
从N个数中找出最大的K个数

* 示例代码(从100万个数字中找出最大的10个数)

```java
public class TopKProblem {

    public static void main(String[] args) throws Exception {
//        buildRandomNum();
        findTopK();
    }
    // 打印出最大的十个数
    public static void printTopK() throws Exception {
        Heap heap = new Heap(10);

        FileReader fileReader = new FileReader("randomInt.txt");
        BufferedReader bufferedReader = new BufferedReader(fileReader);
        String line;
        int value;
        while ((line = bufferedReader.readLine()) != null) {
            value = Integer.parseInt(line);
            if (!heap.isFull()) {
                heap.insert(Integer.parseInt(line));
            } else if (heap.top() < value) {
                heap.pop();
                heap.insert(value);
            }
        }

        heap.print();

        bufferedReader.close();
        fileReader.close();
    }
    // 实现生成100万个随机数到文件中
    public static void buildRandomNum() {
        Random random = new Random();
        FileWriter fileWriter = null;
        BufferedWriter bufferedWriter = null;
        try {
            fileWriter = new FileWriter("randomInt.txt");
            bufferedWriter = new BufferedWriter(fileWriter);

            for (int i = 0; i < 1000000; i++) {
                bufferedWriter.write(random.nextInt(1000000) + "\n");
            }
        } catch (IOException e) {
            e.printStackTrace();
        } finally {
            if (bufferedWriter != null) {
                try {
                    bufferedWriter.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
            if (fileWriter != null) {
                try {
                    fileWriter.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
        }

    }

}
```
