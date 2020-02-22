### **一、cls含义**(classs)

python中cls代表的是类的本身，相对应的self则是类的一个实例对象。

 

### **二、cls用法**

cls可以在静态方法中使用，并通过cls()方法来实例化一个对象。

```python
class Person(object):
    def __init__(self, name, age):
        self.name = name
        self.age = age
        print('self:', self)

    # 定义一个build方法，返回一个person实例对象，这个方法等价于Person()。
    @classmethod
    def build(cls):
        # cls()等于Person()
        p = cls("Tom", 18)
        print('cls:', cls)
        return p


if __name__ == '__main__':
    person = Person.build()
    print(person, person.name, person.age)
```

