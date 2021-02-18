### 数组有值，但是length为0

如果声明一个数组后，动态的赋予键值对数据，那么除非键名是0、1、2这样的数组下标，否则数组的length为0。而且length的大小等于数组下标的最大值+1。

使用JSON.stringify()方法进行json转换时，length为0的数组即使的确有值，也不会被json处理。length不等于0但是中间有空值得数组，空值得位置会被填充null。

```javascript
let a = []; a[0] = 'a'; a[1] = 'a'; a[2] = 'a';
// length=3
// JSON.stringify(b)=[]

let b = []; b['a'] = 'a'; b['b'] = 'b'; b['c'] = 'c';
// length=0
// JSON.stringify(a)=["a","a","a"]

let c= []; c[2] = 'c';
// length=3
// JSON.stringify(c)=[null,null,"c"]
```

### Json

如果json数据是key-value形式的，而且key是数字形式的，使用JSON.parse()解析json字符串的时候会自动按数字大小进行排序。

