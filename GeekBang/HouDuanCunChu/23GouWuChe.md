```json
{
  "updated_by": "KelipuTe",
  "updated_at": "2020-08-03",
  "tags": "极客时间,GeekBang,后端存储实践课"
}
```

---

## 如何设计购物车系统

首先，我们来看购物车系统的主要功能是什么。就是在用户选购商品时，下单之前，暂存用户想要购买的商品。购物车对数据可靠性要求不高，性能也没有特别的要求。思考到这里是很重要的，要明确一个功能模块在系统中的位置和重要程度，然后才能做出一个良好的设计。

### 设计购物车存储时需要把握什么原则

作为一个开发者，首先要仔细考虑，用户在使用购物车的时候，会存在哪些场景。下面给出几个常见的问题场景：1、用户没登录，在浏览器中加购，关闭浏览器再打开，刚才加购的商品还在不在？2、用户没登录，在浏览器中加购，然后登录，刚才加购的商品还在不在？3、关闭浏览器再打开，上一步加购的商品在不在？4、再打开手机，用相同的用户登录，第二步加购的商品还在不在呢？

如果你不仔细把这些问题考虑清楚，用户在使用购物车的时候，就会感觉你的购物车系统不好用，不是加购的商品莫名其妙地丢了，就是购物车莫名其妙地多出来一些商品。要解决上面这些问题，其实只要在存储设计时，把握这几个原则就可以了：1、如果未登录，需要临时暂存购物车的商品；2、用户登录时，把暂存购物车的商品合并到用户购物车中，并且清除暂存购物车；3、用户登陆后，购物车中的商品，需要在浏览器、手机APP和微信等等这些终端中都保持同步。

所以购物车系统需要两类购物车：一类是未登录情况下的暂存购物车，一类是登录后的用户购物车。

### 如何设计暂存购物车的存储

首先要思考暂存购物车应该存在哪里？服务端显然不太合适，存到服务端第一没办法保证用户第二次打开网页的时候能拿到前一次的数据，第二这么做会浪费服务端资源。所以我们只能选择客户端来进行存储。客户端的存储可以选择的不太多：Session、Cookie和LocalStorage，其中浏览器的LocalStorage和App的本地存储是类似的，我们都以LocalStorage来代表。

选好了存储的位置，下面就是选要用哪一种存储方式了。SESSION是不太合适的，原因是，SESSION的保留时间短，而且SESSION的数据实际上还是保存在服务端的。使用Cookie存储，实现起来比较简单，加减购物车、合并购物车的过程中，由于服务端可以读写Cookie，这样全部逻辑都可以在服务端实现，并且客户端和服务端请求的次数也相对少一些。使用LocalStorage存储，实现相对就复杂一点儿，客户端和服务端都要实现一些业务逻辑，但LocalStorage的好处是，它的存储容量比Cookie的4KB上限要大得多，而且不用像Cookie那样，无论用不用，每次请求都要带着，可以节省带宽。

所以，选择Cookie或者是LocalStorage来存储暂存购物车都是没问题的，你可以根据它俩各自的优劣势来选择。这里没有绝对的哪个好哪个不好，都要视具体情况而定，选择最合适的。

### 如何设计用户购物车的存储

用户购物车必须要保证多端的数据同步，所以数据必须保存在服务端。在数据库建一张表保存数据是最常规的思路。也可以选择像Redis这样的缓存数据库。我们上面提到过，购物车对数据可靠性要求不高，所以牺牲购物车数据可靠性，减少主数据库的压力，保证整个系统的高可用性，这个交易是划算的。

我们设计存储架构的过程就是一个不断做选择题的过程。很多情况下，可供选择的方案不止一套，选择的时候需要考虑实现复杂度、性能、系统可用性、数据可靠性、可扩展性等等非常多的条件。需要强调的是，**这些条件每一个都不是绝对不可以牺牲的**，不要让一些所谓的常识禁锢了你的思维。

### 用户购物车能不能用MySQL存储用Redis做缓存呢

首先明确一点，这么做技术上是可行的，但关键在于为什么要这么做，值不值得这么做。因为购物车是在不停的变化的，缓存基本上起不到什么作用，换句话说就是收益有限。但是缓存机制的引入增加了系统的复杂度，而且同一份数据在两个位置存储，还为数据不一致留下了隐患。