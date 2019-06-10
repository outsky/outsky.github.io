---
layout: post
title:  文件系统
date:   2016-05-28 15:06:00 +0800
categories: code
tags: swift
---

## 沙箱

iOS app在安装的时候都会被分配一个独立的沙箱(sandbox)，它的目录访问基本上都被限制在这个沙箱中。

![An iOS app operating within its own sandbox][1]

沙箱中有多个目录，每个目录有不同的用途、访问限制、备份策略等。  

![Dirs compare][2]

-----

## 定位

### App Bundle

只能通过`NSBundle`来获取App Bundle中的文件。

{% highlight objc %}
NSURL* url = [[NSBundle mainBundle] URLForResource:@"MyImage" withExtension:@"png"];
{% endhighlight %}

### 标准目录

可以通过`NSFileManager`的`URLsForDirectory:inDomains:`函数来获取标准目录

-------

## 管理

通过创建自定义的文件夹来管理文件，使目录结构更清晰

### 创建

#### 目录

NSFileManager：

- createDirectoryAtURL:withIntermediateDirectories:attributes:error:
- createDirectoryAtPath:withIntermediateDirectories:attributes:error:

例：  
在~/Library/Application Support目录中创建一个文件夹

{% highlight objc %}
- (NSURL*)applicationDirectory
{
    NSString* bundleID = [[NSBundle mainBundle] bundleIdentifier];
    NSFileManager*fm = [NSFileManager defaultManager];
    NSURL*    dirPath = nil;
 
    // Find the application support directory in the home directory.
    NSArray* appSupportDir = [fm URLsForDirectory:NSApplicationSupportDirectory
                                    inDomains:NSUserDomainMask];
    if ([appSupportDir count] > 0)
    {
        // Append the bundle ID to the URL for the
        // Application Support directory
        dirPath = [[appSupportDir objectAtIndex:0] URLByAppendingPathComponent:bundleID];
 
        // If the directory does not exist, this method creates it.
        // This method is only available in OS X v10.7 and iOS 5.0 or later.
        NSError*    theError = nil;
        if (![fm createDirectoryAtURL:dirPath withIntermediateDirectories:YES
                   attributes:nil error:&theError])
        {
            // Handle the error.
 
            return nil;
        }
    }
 
    return dirPath;
}
{% endhighlight %}

#### 文件

- createFileAtPath:contents:attributes: (NSFileManager)
- writeToURL:atomically: (NSData)
- writeToURL:atomically: (NSString)
- writeToURL:atomically:encoding:error: (NSString)

### 拷贝、移动

NSFileManager:

- copyItemAtURL:toURL:error:
- copyItemAtPath:toPath:error:
- moveItemAtURL:toURL:error:
- moveItemAtPath:toPath:error:

例：  
备份~/Library/Application Support/bundleID/Data目录

{% highlight objc %}
- (void)backupMyApplicationData {
   // Get the application's main data directory
   NSArray* theDirs = [[NSFileManager defaultManager] URLsForDirectory:NSApplicationSupportDirectory
                                 inDomains:NSUserDomainMask];
   if ([theDirs count] > 0)
   {
      // Build a path to ~/Library/Application Support//Data
      // where  is the actual bundle ID of the application.
      NSURL* appSupportDir = (NSURL*)[theDirs objectAtIndex:0];
      NSString* appBundleID = [[NSBundle mainBundle] bundleIdentifier];
      NSURL* appDataDir = [[appSupportDir URLByAppendingPathComponent:appBundleID]
                               URLByAppendingPathComponent:@"Data"];
 
      // Copy the data to ~/Library/Application Support//Data.backup
      NSURL* backupDir = [appDataDir URLByAppendingPathExtension:@"backup"];
 
      // Perform the copy asynchronously.
      dispatch_async(dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0), ^{
         // It's good habit to alloc/init the file manager for move/copy operations,
         // just in case you decide to add a delegate later.
         NSFileManager* theFM = [[NSFileManager alloc] init];
         NSError* anError;
 
         // Just try to copy the directory.
         if (![theFM copyItemAtURL:appDataDir toURL:backupDir error:&anError]) {
            // If an error occurs, it's probably because a previous backup directory
            // already exists.  Delete the old directory and try again.
            if ([theFM removeItemAtURL:backupDir error:&anError]) {
               // If the operation failed again, abort for real.
               if (![theFM copyItemAtURL:appDataDir toURL:backupDir error:&anError]) {
                  // Report the error....
               }
            }
         }
 
      });
   }
}
{% endhighlight %}

### 删除

NSFileManager:

- removeItemAtURL:error:
- removeItemAtPath:error:

### 设置是否自动备份

通过调用NSURL的`setResourceValue:forKey:error:`设置对应NSURL的`NSURLIsExcludedFromBackupKey`属性来修改此url的备份策略。  
如果有很多的文件需要修改，可以把它们都放入一个自定义文件夹，然后设置那个文件夹即可。

------

## 优化tips

- 只读取需要的文件
- 只在需要的时候读取(lazily)
- 只在内容改变的时候写入
- 使用新的API(block)替换旧的callback形式的API
- 将对同一文件小而频繁的读写操作组织在一起，批处理
- 复用同一文件的NSURL(定位文件并生成URL很耗时)
- 选择合理的read buffer大小
- 写入空文件时，避免跳过头部x字节(系统会填0，耗时)
- 容易计算的值不要记到文件中，计算比从磁盘中读取要快得多

------

## 参考

[File System Programming Guide][3]


  [1]: ../res/img/20160528/1.png "1.png"
  [2]: ../res/img/20160528/2.jpg "2.jpg"
  [3]: https://developer.apple.com/library/ios/documentation/FileManagement/Conceptual/FileSystemProgrammingGuide/Introduction/Introduction.html
