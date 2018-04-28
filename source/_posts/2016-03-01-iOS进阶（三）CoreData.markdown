---
title:  "iOS进阶（三）CoreData"
date:   2016-03-01 17:00:00 +8:00
categories: 
  - iOS
tags: 
  - Objective-C
---
## CoreData与SQLite

SQlite

1. 基于C借口，需要使用sql语句，代码繁琐。
2. 在处理大量数据时，表关系更直观。
3. 在OC中不是可视化。

CoreData

1. 可视化，有undo/redo能力。
2. 可以实现多种文件格式NSSQLiteStoreType、NSBinaryStoreType、NSInMemoryStoreType、NSXMLStoreType。
3. 苹果官方API支持，与iOS结合紧密。

对于不同的情况可以选用以上两种数据库。

## CoreData核心类的关系

- 实体管理类：`NSManagedObject`
- 实体描述类：`NSEntityDescription`
- 数据管理器类：`NSManagedObjectContext`
- 数据连接器类：`NSPersistentStoreCoordinator`
- 数据模型类：`NSManagedObjectModel`

下图清楚的展示了各类的关系

![关系图](https://raw.githubusercontent.com/qq1216137037/qq1216137037.github.com/master/_postsImage/CoreData%E6%A0%B8%E5%BF%83%E5%AF%B9%E8%B1%A1.jpg)

## CoreData的简单操作

首先创建一个项目，在创建的时候勾选`Use Core Data`

![勾选][selectUseCoreData]

你会发现在项目列表下出现了一个`CoreData.xcdatamodeld`文件，在`AppDelegate.h`文件中多出了许多代码。选中`CoreData.xcdatamodeld`文件，点击Add Enitity按钮，将实体重命名为`Student`，然后添加3条属性。

![添加实体与属性][addEntityAndAttribute]

选择菜单栏的Editor->Create NSManagedObject SubClass...->选中CoreData->选中Student->Next->Next->Create

封装一个插入学生的方法

```objc
// 封装一个插入学生的方法
- (void)insertStudentWithName:(NSString *)name gender:(NSString *)gender age:(NSNumber *)age
{
    // 创建一个模型
    Student *stu1 = [NSEntityDescription insertNewObjectForEntityForName:@"Student" inManagedObjectContext:self.managedObjectContext]; // 在临时数据库中，按照实体Student，加载出一个视图管理对象(数据模型)
    // 模型赋值有两种方式，KVC赋值可以，属性赋值也可以。
    stu1.name = name;
    [stu1 setValue:gender forKey:@"gender"];
    stu1.age = age;
    // 数据管理类执行同步操作，将数据写入数据库。
    [self.managedObjectContext save:nil];
}
```

在`- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions`方法中书写以下代码。

```objc
- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions {
    // coreData 完成增删改查
    // 1.插入数据
    [self insertStudentWithName:@"巴达" gender:@"雄" age:@21];
    [self insertStudentWithName:@"大宝" gender:@"女" age:@31];
    [self insertStudentWithName:@"秋香" gender:@"女" age:@18];
    
    // 2.查询数据
    // 查询需要先创建查询请求对象
    /*
    NSFetchRequest *request = [[NSFetchRequest alloc] initWithEntityName:@"Student"]; // 请求对象如果这样创建，得到的信息是数据库中所有的记录。
    // 让数据管理对象执行请求
    NSArray *arr = [self.managedObjectContext executeFetchRequest:request error:nil];
     */
    
    /*
    // 为了更加方便的完成查询操作，查询请求可以添加谓词和排序。
    NSFetchRequest *request = [[NSFetchRequest alloc] init];
    // 得到一个实体描述
    NSEntityDescription *entity = [NSEntityDescription entityForName:@"Student" inManagedObjectContext:self.managedObjectContext];
    request.entity = entity;
    // 添加谓词
    NSPredicate *predicate = [NSPredicate predicateWithFormat:@"name = %@ and gender = %@", @"秋香", @"女"];
    request.predicate = predicate;
    // 添加排序
    NSSortDescriptor *descriptor = [[NSSortDescriptor alloc] initWithKey:@"age" ascending:YES]; // 创建按照年龄升序排列的排序条件
    request.sortDescriptors = @[descriptor];
    // 执行查询
    NSArray *arr = [self.managedObjectContext executeFetchRequest:request error:nil];
    // 循环打印查询到的数组
    for (Student *stu in arr) {
        NSLog(@"name is %@, gender is %@, age is %@", stu.name, stu.gender, stu.age);
    }
     */
    
    /*
    // 3.修改数据
    // 修改数据，首先找到需要修改的数据，修改完成之后，同步到数据库。
    NSPredicate *predicate = [NSPredicate predicateWithFormat:@"name = %@", @"中二"];
    request.predicate = predicate;
    NSArray *arr = [self.managedObjectContext executeFetchRequest:request error:nil];
    Student *stu = [arr lastObject];
    stu.name = @"中大二";
    stu.gender = @"妖";
    // 修改完成之后进行save操作
    [self.managedObjectContext save:nil];
     */
    
    /*
    // 4.删除数据
    // 找到你要删除的数据，然后让数据管理器对象执行delete方法。
    NSPredicate *predicate = [NSPredicate predicateWithFormat:@"name = %@", @"巴达"];
    request.predicate = predicate;
    NSArray *arr = [self.managedObjectContext executeFetchRequest:request error:nil];
    Student *stu = [arr lastObject];// 取到要删除的对象
    // 删除临时数据库中某个对象的方法
    [self.managedObjectContext deleteObject:stu];
    [self.managedObjectContext save:nil];
     */
    
    //[self.managedObjectContext save:nil];
    
    NSLog(@"%@", [self applicationDocumentsDirectory]);
    return YES;
}
```

以上代码包含了所有对数据库的增删改查，去掉注释符号即可使用。

>注意：每次添加删除修改数据以后都必须执行同步数据库操作，即执行`[self.managedObjectContext save:nil];`

## 数据迁移

一. 设置coreData的配置为可迁移
在`Appdelegate.m`中的`- (NSPersistentStoreCoordinator *)persistentStoreCoordinator `方法中创建一个字典并且修改`_persistentStoreCoordinator`的`addPersistentStoreWithType: cafiguration: URL: options: error:`方法中的`option`参数设置为字典。

```objc
NSDictionary *dict = @{ NSMigratePersistentStoresAutomaticallyOption:@YES, NSInferMappingModelAutomaticallyOption:@YES}; // 支持版本迁移，支持数据迁移。
if (![_persistentStoreCoordinator addPersistentStoreWithType:NSSQLiteStoreType configuration:nil URL:storeURL options:dict error:&error]) {
...
}
```

二. 添加一个新的集合版本

选中`CoreData.xcdatamodeld`，菜单栏->Editor->Add Model Version...->Finish。`CoreData.xcdatamodeld`出现了下下拉选项，点击小三角，出现了一个`CoreData2.xcdatamodel`。

三. 在新的版本里给表添加新的字段（2、3的顺序不可以错）

点击`CoreData2.xcdatamodel`，添加一条属性。

四. 选择使用新的版本

`alt` + `commond` + `0`，找到Model Version，将Current中的数值改为CoreData2。

五. 桥接两个版本，把老数据库的数据桥接到新数据库

`shift` + `commond` + `n`选择Mapping Model->Next->选中CoreData.xcdatamodel->Next->选中CoreData2.xcdatamodel->将Targets中的CoreData打勾->Create。

六. 生成新的model类

删除之前创建的Student类，选择`CoreData2.xcdatamodel`再创建一次。

>注意：当执行完以上步骤后，需要重新同步数据库，否则应用程序中的数据库不会改变。

[selectUseCoreData]: https://raw.githubusercontent.com/qq1216137037/qq1216137037.github.com/master/_postsImage/%E5%8B%BE%E9%80%89UseCoreData.png
[addEntityAndAttribute]: https://raw.githubusercontent.com/qq1216137037/qq1216137037.github.com/master/_postsImage/%E6%B7%BB%E5%8A%A0%E5%AE%9E%E4%BD%93%E4%B8%8E%E5%B1%9E%E6%80%A7.png
