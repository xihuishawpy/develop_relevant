

实现一个搜索引擎

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

先定义一个基类 SearchEngineBase :

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

实现一个最基本的搜索引擎：

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



