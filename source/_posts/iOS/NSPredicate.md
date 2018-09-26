---
title: NSPredicate
date: 2018-09-25 23:51:53
category: iOS
---

原文在[此](https://nspredicate.xyz/)

# 基本使用

## 格式化参数

查找 `name`是 Asriel 且 `money` 等于 50 的 `Person`

```swift
let fetchRequest = NSFetchRequest<Person>(entityName: "Person")
fetchRequest.predicate = NSPredicate(format: "name == %@ AND money == %i", "Asriel", 50)
```

完整的格式化说明符可以看[官方文档](https://developer.apple.com/library/archive/documentation/Cocoa/Conceptual/Strings/Articles/formatSpecifiers.html)，下面列举几个常用的：

- `%@` 用于字符串、日期、数组等
- `%i` 用于整数
- `%f` 用于浮点数
- `%K` 用于 Keypath，实体属性

```swift
let integerPredicate = NSPredicate(format: "money == %i", 10000)
let doublePredicate = NSPredicate(format: "perimeter > %f", 3.14159)
let stringPredicate = NSPredicate(format: "name == %@", "Asriel")

// 下面两个是一样的效果
let datePredicate = NSPredicate(format: "due_date < %@", Date())
let keyPathDatePredicate = NSPredicate(format: "%K < %@", "due_date", Date())
```

## 比较

基本的比较符号有：`==`，`>`，`<`，`!=` 等

```swift
let equalPredicate = NSPredicate(format: "name == %@", "Steve Jobs")
let notEqualPredicate = NSPredicate(format: "name != %@", "Steve Jobs")

let greaterPredicate = NSPredicate(format: "money > %i", 10000)
let greaterOrEqualPredicate = NSPredicate(format: "money >= %i", 10000)

let lesserPredicate = NSPredicate(format: "money < %i", 10000)
let lesserOrEqualPredicate = NSPredicate(format: "money <= %i", 10000)
```

复合比较：`OR`，`AND`

```swift
// 匹配所有条件
let andPredicate = NSPredicate(format: "name == %@ AND money >= %i", "Steve Jobs", 10000)

// 匹配条件之一
let orPredicate = NSPredicate(format: "name == %@ OR money >= %i", "Steve Jobs", 10000)
```

不区分大小写比较：在比较符号后面加`[c]`

```swift
// 匹配 "jobs", "Jobs", "jObS"
let caseInsensitivePredicate = NSPredicate(format: "name ==[c] %@", "Jobs")
```

# 高级技巧

## 使用替换变量来重用 NSPredicate

使用`$`来表示变量，通过方法`withSubstitutionVariables`来替换变量，参数是字典类型。

```swift
// Persons' name : ["Asriel", "Asgore", "Toriel", "Frisk", "Flowey"]
let context = appDelegate.persistentContainer.viewContext
let fetchRequest = NSFetchRequest<Person>(entityName: "Person")

let reusablePredicate = NSPredicate(format: "name BEGINSWITH $startingName")

// 替换 $startingName 为 'As'
fetchRequest.predicate = reusablePredicate.withSubstitutionVariables(["startingName" : "As"])

do {
  people = try context.fetch(fetchRequest)
  // ["Asriel", "Asgore"]
} catch let error as NSError {
  print("Could not fetch. \(error), \(error.userInfo)")
}

// 再次重用reusablePredicate
// 替换 $startingName 为 'F'
fetchRequest.predicate = reusablePredicate.withSubstitutionVariables(["startingName" : "F"])

do {
  people += try context.fetch(fetchRequest)
  // ["Asriel", "Asgore", "Flowey", "Frisk"]
} catch let error as NSError {
  print("Could not fetch. \(error), \(error.userInfo)")
}
```

可以使用多个替换变量，比如`name BEGINSWITH $startingName AND money > $amount`，然后调用`withSubstitutionVariables(["startingName" : "As", "amount": 50])`

## 过滤数组对象

`SELF` 在格式化字符串里面表示数组里面的  每个元素。

```swift
let names = ["Kim Kardashian", "Kim Jong Un", "Jimmy Kimmel", "Ken"]

var filteredNames : [String] = []

// SELF 表示数组的每个元素
let containPredicate = NSPredicate(format: "SELF CONTAINS %@", "Kim")

filteredNames = names.filter({ name in
    // 如果元素满足条件，返回true，否则返回false
	containPredicate.evaluate(with: name)
})

print("\(filteredNames)")
// ["Kim Kardashian", "Kim Jong Un", "Jimmy Kimmel"]
```

# 例子

**是否包含在数组里**

```swift
let wantedItemIDs = [1, 2, 3, 5, 8, 13, 21]

// 获取在wantedItemIDs里面的记录
let inclusivePredicate = NSPredicate(format: "item_id IN %@", wantedItemIDs)
```

**是否不包含在数组里**

```swift
let unwantedItemIDs = [1, 2, 3, 5, 8, 13, 21]

// 获取不在wantedItemIDs里面的记录
let exclusivePredicate = NSPredicate(format: "NOT (item_id IN %@)", unwantedItemIDs)
```

** 是否以特定字符串开头**

```swift
// Works for "Kim Jong Un", "Kim Kardashian"
let beginPredicate = NSPredicate(format: "name BEGINSWITH %@", "Kim")

// Works for "macintosh", "Macintosh"
let beginCaseInsensitivePredicate = NSPredicate(format: "name BEGINSWITH[c] %@", "mac")
// the [c] means case insensitive match
```

**是否包含特定字符串**

```swift
// Works for "Steven Paul Jobs", "Logan Paul"
let containPredicate = NSPredicate(format: "name CONTAINS %@", "Paul")

// Works for "Shop1", "shopping", "my shop", "bishop"
let containCaseInsensitivePredicate = NSPredicate(format: "name CONTAINS[c] %@", "shop")
// the [c] means case insensitive match
```

**是否以特定字符串结束**

```swift
// Works for "Steve Jobs", "Lisa Jobs"
let endPredicate = NSPredicate(format: "name ENDSWITH %@", "Jobs")

// Works for "mundane jobs", "Steve Jobs"
let endCaseInsensitivePredicate = NSPredicate(format: "name ENDSWITH[c] %@", "jobs")
// the [c] means case insensitive match
```

**通配符 **

`LIKE` 用于通配符匹配，下面介绍匹配字符代表的意思：

- `*` 代表 0 个或者多个字符
- `?` 代表 1 个字符

```swift
let filenameArr = ["img.png", "img1.png", "img2.png", "img10.png", "img100.png", "img200.txt", "img300.csv"]

let pngPredicate = NSPredicate(format: "SELF LIKE %@", "img*.png")

let imageArr = filenameArr.filter(){ filename in
	pngPredicate.evaluate(with: filename)
}
print(imageArr)
// ["img.png", "img1.png", "img2.png", "img10.png", "img100.png"]


let singleCharPngPredicate = NSPredicate(format: "SELF LIKE %@", "img?.png")

let imageArr2 = filenameArr.filter(){ filename in
	singleCharPngPredicate.evaluate(with: filename)
}
print(imageArr2)
// ["img1.png", "img2.png"]
```

** 正则表达式匹配**

`MATCHES` 用于正则表达式

`filename MATCHES 'img\\d{1,3}\\.png'` 会匹配到在`img`和`.png`之间包含 1 到 3 个  数字的文件名，但不包括`img1000.png`。注意反斜杠需要加转义字符`\`。

```swift
let filenameArr = ["img.png", "img1.png", "imgABC.png", "img10.png", "img100.png", "img9000.png", "img12345.png"]

// matches filename that has 1-3 digits between 'img' and '.png'
let regexPredicate = NSPredicate(format: "SELF MATCHES %@", "img\\d{1,3}\\.png")

let filteredArr = filenameArr.filter(){ filename in
    regexPredicate.evaluate(with: filename)
}
print(filteredArr)
// ["img1.png", "img10.png", "img100.png"]
```
