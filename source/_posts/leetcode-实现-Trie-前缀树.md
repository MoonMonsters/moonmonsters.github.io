title: Leetcode --- 实现 Trie (前缀树)
author: _Tao
tags: []
categories:
  - Leetcode
date: 2020-05-23 18:56:00
---
[前缀树](https://leetcode-cn.com/problems/implement-trie-prefix-tree)

实现一个 Trie (前缀树)，包含 insert, search, 和 startsWith 这三个操作。

示例:
```python
Trie trie = new Trie();
trie.insert("apple");
trie.search("apple");   // 返回 true
trie.search("app");     // 返回 false
trie.startsWith("app"); // 返回 true
trie.insert("app");   
trie.search("app");     // 返回 true
```

说明:

你可以假设所有的输入都是由小写字母 a-z 构成的。
保证所有输入均为非空字符串。


leetcode上该代码已经通过了，自己在原基础上加了注释和遍历的功能。
这个就是字典树:
-  插入一个单词时，遍历此单词各个字符
- 如果单个字符存在，则取出该字符对应的节点
- 如果字符不存在，则创建字符节点，并加入到上一个字符的子节点(next)中
- 循环

<!-- more -->

一张图可以解释这个功能:
![字典树](https://qxinhai.oss-cn-shenzhen.aliyuncs.com/pic/20180717203719720.png)

```python
class Node(object):

    def __init__(self, value):
        # 节点的值
        self.value = value
        # 该节点是否成为一个单词的结尾字符
        self.end = False
        # 该节点的子节点
        self.next = dict()

    def __str__(self):
        return self.value

class Trie(object):

    def __init__(self):
        """
        Initialize your data structure here.
        """
        self.root = Node('')

    def insert(self, word: str):
        """
        Inserts a word into the trie.
        """
        cur = self.root
        for w in word:
            node = cur.next.get(w)
            # 判断子节点中是否存在w这个节点，如果没有则创建
            if not node:
                node = Node(w)
                # 将新创建的节点更新到next中
                cur.next[w] = node
            # 移动指针位置
            cur = node
        # word的所有值遍历完成，将最后一个节点的end值设置为True
        cur.end = True

    def search(self, word: str):
        """
        Returns if the word is in the trie.
        """
        node = self.root
        for w in word:
            node = node.next.get(w)
            # 遍历，如果node不存在，则说明该单词不存在
            if not node:
                return False
        # 有可能word是前缀，直接返回end值即可
        return node.end

    def startsWith(self, prefix: str):
        """
        Returns if there is any word in the trie that starts with the given prefix.
        """
        node = self.root
        for p in prefix:
            node = node.next.get(p)
            # node不存在，说明前缀错误，直接返回False
            if not node:
                break
        else:
            # prefiex遍历完成，前缀存在，返回True
            return True

        return False


    def traverse(self, node, word=''):
        """
        遍历所有单词
        """
        if node.end:
            print(word)
        elif node:
            for node in node.next.values():
                self.traverse(node, word+node.value)

def random_words():
    import random
    import string
    for x in range(10000):
        yield ''.join(random.choice(string.ascii_lowercase) for y in range(random.randint(5, 15)))


def test():
    words = random_words()
    t = Trie()
    for word in words:
        t.insert(word)
    t.insert('hello')
    t.insert('world')
    t.insert('python')
    # t.traverse(t.root)

    import time
    start = time.time()
    print('search python: ' + str(t.search('python')))
    print('search time: ' + str(time.time() - start))

if __name__ == '__main__':
    test()
```