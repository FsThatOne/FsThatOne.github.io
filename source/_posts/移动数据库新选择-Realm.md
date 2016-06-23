---
title: 移动数据库新选择-Realm
date: 2016-06-06 14:14:17
tags:
 - iOS
 - Realm
---
** Realm是一个跨平台移动数据库引擎，支持iOS、OS X（Objective-C和Swift）以及Android。**
<!-- more -->
<The rest of contents | 余下全文>

## 简介

[Realm](https://realm.io/cn/)是一个跨平台移动数据库引擎，支持iOS、OS X（Objective-C和Swift）以及Android。

2014年7月发布。由YCombinator孵化的创业团队历时几年打造，是第一个专门针对移动平台设计的数据库。目标是取代SQLite。

为了彻底解决性能问题，核心数据引擎C++打造，并不是建立在SQLite之上的ORM。如果对数据引擎实现想深入了解可以查看：[Realm 核心数据库引擎探秘](https://realm.io/cn/news/jp-simard-realm-core-database-engine/)。因此得到的收益就是比普通的ORM要快很多，甚至比单独无封装的SQLite还要快。

因为是ORM，本身在设计时也针对移动设备（iOS、Android），所以非常简单易用，学习成本很低。

## 优劣

### 优势

- 首先是realm的速度，realm拥有碾压级性能,虽然一般的手机客户端不会用到大数据的情况，但有些情况你还是会经常遇到的。比如要存全国的省市区，这个数据量就很大，而且还是关系型的。如果用coredata会灰常的慢，这个时候Realm就派上用场了。

- 其次是realm的使用，realm相对于coredata，不管是增删改查，还是数据库版本的更新，都要更加的简单，比较适合新手使用，降低了团队学习的成本。

- 这里要说一下数据库的版本更新，当我们的数据库中表的字段发生变化时，为了兼容老版本，需要对数据库进行升级。这个在coredata里是有的，但是需要操作的地方很多，既要在试图里修改，还要重新生产实例。但是realm就比较方便了，几行代码。而且如果改动特别大，可以直接创建新的库出来，比coredata要灵活。

- 最后realm发展到现在也相对比较成熟了，有团队专门去维护，热度也比较高，可以在一些功能上试用起来。

### 劣势

- 使用Realm会增大应用大概1MB的包体积,这是相对于CoreData的短板.

- 所有的存储对象对象都要继承自RealmObject,如果项目中的数据都继承自某个已经写好的基类,那么就需要做出一些调整;当然,这个问题在Swift中就不存在,因为swift是天生的面向协议编程范式,只需要遵循一个协议就好了.

- 不能自定义getter、setter,如果自定义了getter和setter,则会对存储有影响;这个问题在Swift中同样不存在,因为Swift中使用的是didSet和willSet这种通知机制.

- 查询的结果不是数组:为了能够支持查询结果的链式查询,realm自己定义了一个集合类型,虽然可以枚举,但是并不是熟悉的数组类型,又因为realm具有延迟加载(懒加载)的特性,所以不建议在数据层把这个集合转换成数组类型.

## 相关细节

- 属性property种类
Realm支持以下的属性property种类： BOOL,bool, int, NSInteger, long, float, double, CGFloat, NSString, NSDate 和NSData。
你可以使用RLMArray<NSObject> 和RLMObject来模拟对一或对多的关系
Realm也支持RLMObject继承。
- 属性property特性

  1. 请注意Realm忽略了objective-c的property attributes,像nonatomic, atomic, strong, copy, weak等等。所以，为了避免误解，我们推荐你在写入模型的时候不要使用任何的property attributes。但是，假如你设置了，这些attributes会一直生效直到RLMObject被写入realm数据库。
  2. 无论RLMObject在或不在realm中，你为getter和setter自定义的名字都能正常工作。

## 基本操作

#### 创建model

```
#import "City.h"
#import <Realm/Realm.h>

RLM_ARRAY_TYPE(City)

@interface Province : RLMObject

@property  NSString *enName;
@property  int pid;
@property  NSString *prefixLetter;
@property  NSString *shortName;
@property  int sId;
@property  int type;
@property  RLMArray<City *><City> *citys;

@end
```

这里是创建了一个省份对象,它有一个城市的属性,保存的都是该省份的所有城市,这是一种一对多的映射关系.
我们处理一对多的映射关系时,需要两个步骤:
1. 声明RLM_ARRAY_TYPE(City),这句话的意思是声明一个RLMArray<City> type. 这个声明可以放在两个地方,一个是需要关联的那个类的最上方,就像我写这样,也可以放在City的最后,也就是@end的下面.

2. 然后添加关联属性@property RLMArray<City* ><City> citys;这样声明看起来怪怪的,我们来分析一下:
  + RLMArray就是数组类型,没什么好说的;
  + <CityEntity* >官方解释是属性的特别化(generic specialization)，这可以阻止在编译时使用错误对象类型的数组;
  + <CityEntity>这个其实和RLMArray是连在一起的， 此RLMArray遵守的协议，可以让 Realm 知晓如何在运行时确定数据模型的架构。

#### 新增数据
首先我们要先造好数据:
```
Province* province=[[Province alloc] init];
//赋值
City *city = [[City alloc] init];
[province.citys addObject:city];
```

然后就是把记录插入到数据库中:
```
//获取单例
RLMRealm *realm = [RLMRealm defaultRealm];
//开启事物
[realm beginWriteTransaction];
[realm addObject:province];
[realm commitWriteTransaction];
```

#### 更新数据
最普通的更新数据:
```
[[RLMRealm defaultRealm] transactionWithBlock:^{
    Province *province=[provinceArray firstObject];
    province.shortName=@"浙江";
}];
```

也可以给模型设置一个主键,根据主键去更新数据库,更新需要拥有一个主键---PrimaryKey:
```
@implementation bookBook
+ (NSString *)primaryKey {
 return @"id";
}
@end
```

然后通过realm的接口调用:
```
// 创建一个含有主键"id"的模型
bookBook *cheeseBook = [[Book alloc] init];
cheeseBook.title = @"Cheese recipes";
cheeseBook.price = @9000;
cheeseBook.id = @1;
// 更新id为1的bookBook
[realm beginWriteTransaction];
[realm addOrUpdateObject:cheeseBook];
[realm commitWriteTransaction];
```

#### 删除数据
三种方式:
1. 单条记录删除:
```
[realm beginWriteTransaction];
[realm deleteObject:cheeseBook];
[realm commitWriteTransaction];
```
2. 多条记录删除:
```
//result是搜索的结果
[[RLMRealm defaultRealm] transactionWithBlock:^{
        [[RLMRealm defaultRealm] deleteObjects:result];
    }];
```
3. 全部删除:
```
[[RLMRealm defaultRealm] transactionWithBlock:^{
        [[RLMRealm defaultRealm] deleteAllObjects];
    }];
```

#### 数据模型定制
几个存在的类方法进一步指定模型信息:

`+attributesForProperty`:可以被重写来来提供特定属性property的属性值attrbutes例如某个属性值要添加索引。

`+defaultPropertyValues`可以被重写，用以为新建的对象提供默认值。

`+primaryKey`可以被重写来设置模型的主键。定义主键可以提高效率并且确保唯一性。

  `ignoredProperties`可以被重写来防止Realm存储模型属性。

## 总结
上面介绍了realm的基本操作，可以看到realm使用起来是非常简单的，而且他已经帮你把事务做好了。这就保证了realm在实际使用中的安全性，不会因为一些突发情况造成数据的紊乱。之后我会对他进行更深入的探讨，如果你感兴趣可以关注我。
