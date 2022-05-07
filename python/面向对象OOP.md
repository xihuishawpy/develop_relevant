
## 1、类的基本认识

示例1：

```python

class Document():
    def __init__(self, title, author, context):
        print('init function called')
        self.title = title
        self.author = author
        self.__context = context # __开头的属性是私有属性

    def get_context_length(self):
        return len(self.__context)

    def intercept_context(self, length):
        self.__context = self.__context[:length]

harry_potter_book = Document('Harry Potter', 'J. K. Rowling', '... Forever Do not believe any thing is capable of thinking independently ...')

print(harry_potter_book.title)
print(harry_potter_book.author)
print(harry_potter_book.get_context_length())

harry_potter_book.intercept_context(10)

print(harry_potter_book.get_context_length())

print(harry_potter_book.__context)

########## 输出 #########################

init function called
Harry Potter
J. K. Rowling
77
10

---------------------------------------------------------------------------
AttributeError                            Traceback (most recent call last)
<ipython-input-5-b4d048d75003> in <module>()
     22 print(harry_potter_book.get_context_length())
     23 
---> 24 print(harry_potter_book.__context)

AttributeError: 'Document' object has no attribute '__context'
```

![20220507144859](https://cdn.jsdelivr.net/gh/xihuishawpy/PicBad@main/blogs/pictures/20220507144859.png)

> 想成为一个优秀的工程师，那就一定要积极锻炼**直觉思考**和**快速类比**的能力，尤其是在找不到 bug 的时候。这才是编程学习中能给人最快进步的方法和路径。

**类，一群有着相同属性和函数的对象的集合。**

**init** 表示构造函数，意即<u>一个对象生成时会被自动调用的函数</u>。能看到， harry_potter_book = Document(...)这一行代码被执行的时候，'init function called'字符串会被打印出来。

如果一个属性以 __（此处为双下滑线） 开头，说明该属性是**私有属性，是指不希望在类的函数之外的地方被访问和修改的属性**（只能调用类中的函数进行修改）。可以看到，title 和 author 能够很自由地被打印出来，但是 print(harry_potter_book.__context)就会报错。

## 2、类的几种函数

示例2：

```python

class Document():
    
    WELCOME_STR = 'Welcome! The context for this book is {}.'
    
    def __init__(self, title, author, context):
        print('init function called')
        self.title = title
        self.author = author
        self.__context = context
    
    # 类函数
    @classmethod
    def create_empty_book(cls, title, author):
        return cls(title=title, author=author, context='nothing')
    
    # 成员函数
    def get_context_length(self):
        return len(self.__context)
    
    # 静态函数
    @staticmethod
    def get_welcome(context):
        return Document.WELCOME_STR.format(context)


empty_book = Document.create_empty_book('What Every Man Thinks About Apart from Sex', 'Professor Sheridan Simove')


print(empty_book.get_context_length())
print(empty_book.get_welcome('indeed nothing'))

########## 输出 ##########

init function called
7
Welcome! The context for this book is indeed nothing.
```

几个问题：

1. 在一个类中定义一些常量，每个对象都可以方便访问这些常量而不用重新构造？
   **只需要和函数并列地声明并赋值**，比如这段代码中的 WELCOME_STR，用全大写来表示常量

2. 如果一个函数不涉及到访问修改这个类的属性，而放到类外面有点不恰当，怎么做才能更优雅呢？
   
   类函数、成员函数和静态函数，前两者产生的影响是动态的，能够访问或者修改对象的属性；而静态函数则与类没有什么关联，最明显的特征便是，静态函数的第一个参数没有任何特殊性。

   - 静态函数
  一般而言，静态函数可以用来做一些简单独立的任务，既方便测试，也能优化代码结构。静态函数还可以通过在函数前一行加上 **@staticmethod** 来表示
   
   - 类函数
    **第一个参数一般为 cls，表示必须传一个类进来**。<u>类函数最常用的功能是实现不同的 init 构造函数</u>(把init构造函数中的变量修改了~)，比如上文代码中使用 create_empty_book 类函数，来创造新的书籍对象，其 context 一定为 'nothing'。类函数需要装饰器 **@classmethod** 来声明。

    - 成员函数
    成员函数则是我们最正常的类的函数，它**不需要任何装饰器声明，第一个参数 self 代表当前对象的引用**，可以通过此函数，来实现想要的查询 / 修改类的属性等功能。


3. 类是一群相似的对象的集合，那么可不可以是一群相似的类的集合呢？
   类的继承

## 3、类的继承

类的继承，指的是一个类既拥有另一个类的特征，也拥有不同于另一个类的独特特征。在这里的第一个类叫做子类，另一个叫做父类，特征其实就是类的属性和函数。

示例3 ：

```python

class Entity():
    def __init__(self, object_type):
        print('parent class init called')
        self.object_type = object_type
    
    def get_context_length(self):
        raise Exception('get_context_length not implemented')
    
    def print_title(self):
        print(self.title)

class Document(Entity):
    def __init__(self, title, author, context):
        print('Document class init called')
        Entity.__init__(self, 'document')
        self.title = title
        self.author = author
        self.__context = context
    
    def get_context_length(self):
        return len(self.__context)
    
class Video(Entity):
    def __init__(self, title, author, video_length):
        print('Video class init called')
        Entity.__init__(self, 'video')
        self.title = title
        self.author = author
        self.__video_length = video_length
    
    def get_context_length(self):
        return self.__video_length

harry_potter_book = Document('Harry Potter(Book)', 'J. K. Rowling', '... Forever Do not believe any thing is capable of thinking independently ...')
harry_potter_movie = Video('Harry Potter(Movie)', 'J. K. Rowling', 120)

print(harry_potter_book.object_type)
print(harry_potter_movie.object_type)

harry_potter_book.print_title()
harry_potter_movie.print_title()

print(harry_potter_book.get_context_length())
print(harry_potter_movie.get_context_length())

########## 输出 ##########

Document class init called
parent class init called
Video class init called
parent class init called
document
video
Harry Potter(Book)
Harry Potter(Movie)
77
120
```


