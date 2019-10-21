
```
import collections
//创建一个namedtuple，每一张牌对应一个tuple
Card = collections.namedtuple('Card', ['rank', 'suit'])

// 创建一个扑克对象，存所有54张扑克
class FrenchDeck:
    ranks = [str(n) for n in range(2, 11)] + list('JQKA')
    suits = 'spades diamonds clubs hearts'.split()
    def __init__(self):
        self._cards = [Card(rank, suit) for suit in self.suits for rank in self.ranks]
    def __len__(self):
        return len(self._cards)
    def __getitem__(self, position):
        return self._cards[position]


//初始化一套扑克
deck = FrenchDeck()

//查看扑克属性
len(deck)

//随机生成一张
from random import choice
choice(deck)

//取其中一张
deck[0]
deck[10]

```
因为 __getitem__ 方法把 [] 操作交给了 self._cards 列表，所以我们的 deck 类自动支持切片（slicing）操作

```
In [14]: deck[:10]
Out[14]:
[Card(rank='2', suit='spades'),
 Card(rank='3', suit='spades'),
 Card(rank='4', suit='spades'),
 Card(rank='5', suit='spades'),
 Card(rank='6', suit='spades'),
 Card(rank='7', suit='spades'),
 Card(rank='8', suit='spades'),
 Card(rank='9', suit='spades'),
 Card(rank='10', suit='spades'),
 Card(rank='J', suit='spades')]
```
迭代和反向迭代

```
In [15]: for card in deck:
    ...:     print(card)
    

In [17]: for card in reversed(deck):
    ...:     print(card)
```


```
In [18]:  Card('Q', 'hearts') in deck
Out[18]: True
```
