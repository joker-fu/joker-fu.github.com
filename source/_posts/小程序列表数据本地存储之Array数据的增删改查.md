---
title: 小程序列表数据本地存储之Array数据的增删改查
copyright: true
date: 2018-08-06 22:21:40
categories: 小程序
tags: 小程序
---

#### 一

首先查看[小程序数据缓存API](https://developers.weixin.qq.com/miniprogram/dev/api/data.html#wxsetstorageobject)文档了解小程序数据缓存能力

##### 存储

1 异步存储 wx.setStorage(OBJECT)

代码示例：
```
wx.setStorage({
  key:"key",
  data:"value"
})
```

2 同步存储 wx.setStorageSync(KEY,DATA)

代码示例：
```
try {
	wx.setStorageSync('key', 'value')
} catch (e) {	
}
```

**注：无论异步、同步将数据存储在本地缓存中指定的 key 中，会覆盖掉原来该 key 对应的内容**

##### 读取

1 异步读取 wx.getStorage(OBJECT)

代码示例：
```
wx.getStorage({
  key: 'key',
  success: function(res) {
  	console.log(res.data)
  } 
})

```

2 同步读取 wx.getStorageSync(OBJECT)

代码示例：
```
try {
  var value = wx.getStorageSync('key')
  if (value) {
  	// Do something with return value
  }
} catch (e) {
  // Do something when catch error
}
```

### 二

我们就此了解了小程序基本的存取API，接下来就开始对一个列表Array数据进行增删改查，数据如下

```
// pages/list/list.js

const key = "users"

Page({

  /**
   * 页面的初始数据
   */
  data: {
    users: [{
        id: 10,
        name: "小陈",
        age: 20
      },
      {
        id: 11,
        name: "小宇",
        age: 30
      },
      {
        id: 12,
        name: "小明",
        age: 40
      }
    ]
  },

  /**
   * 生命周期函数--监听页面加载
   */
  onLoad: function(options) {
    this.save(this.data.users)
  },

  ......
  
  })
  
```

##### 存储

```
  /**
   * 存储
   */
  save: function(users) {
    if (!(users instanceof Array)) return;

    wx.setStorage({
      key: key,
      data: users,
    })
  },
 
```

##### 读取

```
  /**
   * 读取
   */
  read: function() {
    let that = this

    wx.getStorage({
      key: key,
      success: function(res) {
        that.setData({
          users: res.data
        })
      },
    })
  },

```

##### 插入

```

  /**
   * 插入
   */
  insert: function() {
    let value = {
      id: 111,
      name: "小陈",
      age: 20
    }
    this.read()
    //在users开头插入数据
    this.data.users.unshift(value)
    this.save(this.data.users)
  },

```


##### 更新

由存储API可知，更新只需要以相同key存入相关新的数据即可。

```
  /**
   * 更新
   */
  update: function() {
    let value = {
      id: 111,
      name: "大卫",
      age: 20
    }
    this.read()
    //更新0下标的值 如果要根据id来更新 则需要遍历出具体的index
    this.data.users[0] = value
    this.save(this.data.users)
  },

```

##### 删除

```
  /**
   * 删除
   */
  del: function() {
    this.read()
    //从下表0开始删除1个数据 如果要根据id来删除 则需要遍历出具体的index
    this.data.users.splice(0, 1)
    this.save(this.data.users)
  },
```

**注意： 本文街扫到的小程序本地存储API，在存储的时候都是覆盖之前存在的key，若需要操作之前缓存的数据就需要把之前的缓存数据读取出来，对其进行操作后再写入缓存。**

### 三

前面用到了JavaScript相关Array对象的一些方法，具体可查看[Array 对象方法](http://www.w3school.com.cn/jsref/jsref_obj_array.asp)

方法 |描述
---|---
concat() | 连接两个或更多的数组，并返回结果。
join() | 把数组的所有元素放入一个字符串。元素通过指定的分隔符进行分隔。
pop() | 删除并返回数组的最后一个元素
push() | 向数组的末尾添加一个或更多元素，并返回新的长度。
reverse() | 颠倒数组中元素的顺序。
shift() | 删除并返回数组的第一个元素
slice() | 从某个已有的数组返回选定的元素
sort() | 对数组的元素进行排序
splice() | 删除元素，并向数组添加新元素。
toSource() | 返回该对象的源代码。
toString() | 把数组转换为字符串，并返回结果。
toLocaleString() | 把数组转换为本地数组，并返回结果。
unshift() | 向数组的开头添加一个或更多元素，并返回新的长度。
valueOf() | 返回数组对象的原始值

### 四

同理，还有getStorageInfo removeStorage clearStorage 等的同步、异步方法。

