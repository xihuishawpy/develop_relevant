

## 案例：实现一个搜索引擎

检索内容：

```python

# 1.txt
I have a dream that my four little children will one day live in a nation where they will not be judged by the color of their skin but by the content of their character. I have a dream today.

# 2.txt
I have a dream that one day down in Alabama, with its vicious racists, . . . one day right there in Alabama little black boys and black girls will be able to join hands with little white boys and white girls as sisters and brothers. I have a dream today.

# 3.txt
I have a dream that one day every valley shall be exalted, every hill and mountain shall be made low, the rough places will be made plain, and the crooked places will be made straight, and the glory of the Lord shall be revealed, and all flesh shall see it together.

# 4.txt
This is our hope. . . With this faith we will be able to hew out of the mountain of despair a stone of hope. With this faith we will be able to transform the jangling discords of our nation into a beautiful symphony of brotherhood. With this faith we will be able to work together, to pray together, to struggle together, to go to jail together, to stand up for freedom together, knowing that we will be free one day. . . .

# 5.txt
And when this happens, and when we allow freedom ring, when we let it ring from every village and every hamlet, from every state and every city, we will be able to speed up that day when all of God's children, black men and white men, Jews and Gentiles, Protestants and Catholics, will be able to join hands and sing in the words of the old Negro spiritual: "Free at last! Free at last! Thank God Almighty, we are free at last!"
```

### 1、先定义一个基类 SearchEngineBase :

```python
class SearchEngineBase(object):
    def __init__(self):
        pass

    def add_corpus(self, file_path):
        with open(file_path, 'r') as fin:
            text = fin.read()
        self.process_corpus(file_path, text)

    def process_corpus(self, id, text):
        raise Exception('process_corpus not implemented.')

    def search(self, query):
        raise Exception('search not implemented.')

def main(search_engine):
    for file_path in ['1.txt', '2.txt', '3.txt', '4.txt', '5.txt']:
        search_engine.add_corpus(file_path)

    while True:
        query = input()
        results = search_engine.search(query)
        print('found {} result(s):'.format(len(results)))
        for result in results:
            print(result)
```

SearchEngineBase 可以被继承，继承的类分别代表不同的算法引擎。每一个引擎都应该实现 process_corpus() 和 search() 两个函数(因为在基类SearchEngineBase中，2种方法未定义)，对应于搜索引擎的索引器和检索器。main() 函数提供搜索器和用户接口，于是一个简单的包装界面就有了。

- add_corpus() ：负责读取文件内容，将文件路径作为 ID，连同内容一起送到 process_corpus 中；

- process_corpus ：对内容进行处理，然后文件路径为 ID ，将处理后的内容存下来。处理后的内容，就叫做索引（index）。

- search ：则给定一个询问，处理询问，再通过索引检索，然后返回。

### 2、实现一个最基本的搜索引擎

```python

class SimpleEngine(SearchEngineBase):
    def __init__(self):
        # 直接用类名调用父类方法（单继承的场景下没什么问题）
        super(SimpleEngine, self).__init__() 
        # 初始化私有变量，用来存储文件名到文件内容的字典
        self.__id_to_texts = {}     

    def process_corpus(self, id, text):
        self.__id_to_texts[id] = text

    def search(self, query):
        results = []
        for id, text in self.__id_to_texts.items():
            if query in text:
                results.append(id)
        return results

search_engine = SimpleEngine()
main(search_engine)


########## 输出 ##########


simple
found 0 result(s):
little
found 2 result(s):
1.txt
2.txt
```

SimpleEngine 实现了一个继承 SearchEngineBase 的子类，继承并实现了 process_corpus 和 search 接口(**函数重写**)，同时，也顺手继承了 add_corpus 函数（当然想重写也是可行的），因此我们可以在 main() 函数中直接调取。

注意：super(SimpleEngine, self)首先找到 SimpleEngine 的父类SearchEngineBase，然后把类 SimpleEngine 的对象转换为父类SearchEngineBase的对象。


### 3、实现一个 bag of words(词袋模型)

```python

import re

class BOWEngine(SearchEngineBase):
    def __init__(self):
        super(BOWEngine, self).__init__()
        self.__id_to_words = {}

    def process_corpus(self, id, text):
        self.__id_to_words[id] = self.parse_text_to_words(text)

    def search(self, query):
        query_words = self.parse_text_to_words(query)
        results = []
        for id, words in self.__id_to_words.items():
            if self.query_match(query_words, words):
                results.append(id)
        return results
    
    @staticmethod
    def query_match(query_words, words):
        for query_word in query_words:
            if query_word not in words:
                return False
        return True

    @staticmethod
    def parse_text_to_words(text):
        # 使用正则表达式去除标点符号和换行符
        text = re.sub(r'[^\w ]', ' ', text)
        # 转为小写
        text = text.lower()
        # 生成所有单词的列表
        word_list = text.split(' ')
        # 去除空白单词
        word_list = filter(None, word_list)
        # 返回单词的 set
        return set(word_list)

search_engine = BOWEngine()
main(search_engine)


########## 输出 ##########


i have a dream
found 3 result(s):
1.txt
2.txt
3.txt
freedom children
found 1 result(s):
5.txt
```

1. 子类BOWEngine继承了父类SearchEngineBase；
2. 将process_corpus、search进行了函数重写，覆盖父类的方法；
3. 用@staticmethod修饰query_match、parse_text_to_words函数，使它们成为静态方法；
4. main()函数中，调用了父类的add_corpus函数，将文件内容送到process_corpus函数中；

### 4、倒序索引

倒序索引，也就是说这次反过来，保留的是 `word -> id 的字典`。在 search 时，我们只需要把想要的 query_word 的几个倒序索引单独拎出来，然后从这几个列表中找共有的元素，那些共有的元素，即 ID，就是我们想要的查询结果。这样就避免了将所有的 index 过一遍的尴尬。

```python

import re

class BOWInvertedIndexEngine(SearchEngineBase):
    def __init__(self):
        super(BOWInvertedIndexEngine, self).__init__()
        self.inverted_index = {}

    # 建立倒序索引
    def process_corpus(self, id, text):
        words = self.parse_text_to_words(text)
        for word in words:
            if word not in self.inverted_index:
                self.inverted_index[word] = []
            self.inverted_index[word].append(id)

    def search(self, query):
        query_words = list(self.parse_text_to_words(query))
        query_words_index = list()
        for query_word in query_words:
            query_words_index.append(0)
        
        # 如果某一个查询单词的倒序索引为空，我们就立刻返回
        for query_word in query_words:
            if query_word not in self.inverted_index:
                return []
        
        result = []
        while True:
            
            # 首先，获得当前状态下所有倒序索引的 index
            current_ids = []
            
            for idx, query_word in enumerate(query_words):
                current_index = query_words_index[idx]
                current_inverted_list = self.inverted_index[query_word]
                
                # 已经遍历到了某一个倒序索引的末尾，结束 search
                if current_index >= len(current_inverted_list):
                    return result

                current_ids.append(current_inverted_list[current_index])

            # 然后，如果 current_ids 的所有元素都一样，那么表明这个单词在这个元素对应的文档中都出现了
            if all(x == current_ids[0] for x in current_ids):
                result.append(current_ids[0])
                query_words_index = [x + 1 for x in query_words_index]
                continue
            
            # 如果不是，我们就把最小的元素加一
            min_val = min(current_ids)
            min_val_pos = current_ids.index(min_val)
            query_words_index[min_val_pos] += 1

    @staticmethod
    def parse_text_to_words(text):
        # 使用正则表达式去除标点符号和换行符
        text = re.sub(r'[^\w ]', ' ', text)
        # 转为小写
        text = text.lower()
        # 生成所有单词的列表
        word_list = text.split(' ')
        # 去除空白单词
        word_list = filter(None, word_list)
        # 返回单词的 set
        return set(word_list)

search_engine = BOWInvertedIndexEngine()
main(search_engine)


########## 输出 ##########


little
found 2 result(s):
1.txt
2.txt
little vicious
found 1 result(s):
2.txt
```

至于 search() 函数，它会根据 query_words 拿到所有的倒序索引，如果拿不到，就表示有的 query word 不存在于任何文章中，直接返回空；拿到之后，运行一个“合并 K 个有序数组”的算法，从中拿到我们想要的 ID，并返回。


### 5、LRU 和多重继承

大量重复性搜索占据了 90% 以上的流量，于是给搜索引擎加一个缓存。

```python

import pylru

# LRUCache 定义了一个缓存类
class LRUCache(object):
    def __init__(self, size=32):
        self.cache = pylru.lrucache(size)

    def has(self, key):
        return key in self.cache
    
    def get(self, key):
        return self.cache[key]
    
    def set(self, key, value):
        self.cache[key] = value

class BOWInvertedIndexEngineWithCache(BOWInvertedIndexEngine, LRUCache):
    def __init__(self):
        super(BOWInvertedIndexEngineWithCache, self).__init__() 
        LRUCache.__init__(self)
    
    # 缓存的逻辑
    def search(self, query):
        # 调用 has() 函数判断是否在缓存中
        if self.has(query):
            print('cache hit!')
            # 调用 get 函数直接返回结果
            return self.get(query)
        
        result = super(BOWInvertedIndexEngineWithCache, self).search(query)
        self.set(query, result)
        
        return result

search_engine = BOWInvertedIndexEngineWithCache()
main(search_engine)


########## 输出 ##########


little
found 2 result(s):
1.txt
2.txt
little
cache hit!
found 2 result(s):
1.txt
2.txt
```

BOWInvertedIndexEngineWithCache 类，**多重继承**了两个类，多重继承有两种初始化方法：

1. 直接初始化该类的第一个父类：

   ```python
    super(BOWInvertedIndexEngineWithCache, self).__init__()
    ```

    super().\__init__() 只能调用第一个父类的构造函数；对于多重继承，如果想调用其他父类的构造函数，则必须指定（用下面的传统方法）

2. 对于多重继承，如果有多个构造函数需要调用， 必须用传统的方法: 

    ```python
    LRUCache.__init__(self) 
    ```

缓存的使用逻辑 —— BOWInvertedIndexEngineWithCache类的search方法 ：

调用 has() 函数判断是否在缓存中，如果在，调用 get 函数直接返回结果；如果不在，送入后台计算结果（倒排索引搜索），然后再塞入缓存。

另外，search() 函数被子类 BOWInvertedIndexEngineWithCache 再次重载，但是还需要调用 BOWInvertedIndexEngine 的 search() 函数，强行调用被覆盖的父类的函数:

```python 
super(BOWInvertedIndexEngineWithCache, self).search(query)
```

也就是，`当子类重写了父类的方法，如果想调用父类的方法，可以利用super()`

---

补充：

一、Python并没有真正的私有化支持，但可用下划线得到`伪私有`：

1. _xxx ： "单下划线 " 开始的成员变量叫做保护变量，意思是`只有类对象和子类对象自己能访问到这些变量`，需通过类提供的接口进行访问；
   
2. __xxx ： 类中的私有变量/方法名，`只有类对象自己能访问`，连子类对象也不能访问到这个数据。
   
3. \__xxx __ ：魔法函数，前后均有一个“双下划线” 代表python里`特殊方法专用的标识`，如 \__init __() 代表类的构造函数。


二、强制调用父类私有属性\方法

通过“**self._类名__属性名**”的方式调用父类的私有方法、属性

```python
class Father():
    def __init__(self,value):
        self.__value = value  # 私有属性
        
    def __action(self):  # 私有方法
        print('调用父类的方法')
    
    # 还可以在父类创建方法返回私有属性的值，然后子类调用父类方法去取得该私有属性的值
    def get_private_value(self): 
        print(self.__value)
 
class Son(Father):
    def action(self):
        print(self._Father__value)
        self._Father__action()
        
        
son = Son(520)
son.action()
# 子类调用父类方法去取得私有属性的值
son.get_private_value() 

# output: 
# 520
# 调用父类的方法
# 520
```

---

相关阅读：

1. [Python:类的继承，调用父类的属性和方法基础详解](https://blog.csdn.net/yilulvxing/article/details/85374142)
2. 