### 制作CocoaPods公开库

#### 步骤

1. 托管框架源码到Git
2. 创建框架描述信息
3. 上传框架描述信息到 `https://github.com/CocoaPods/Specs`
4. 命令行 `pod setup`，创建本地索引库
5. 命令行 `pod install`，将框架集成到项目中

#### 创建 pod spec

`pod spec` 命令用于创建框架的描述信息文件，文档如下：https://guides.cocoapods.org/syntax/podspec.html

```
pod spec create [库名]
pod spec create CHTimerLib
```

- version：这个spec的映射版本，保证Git的release与此对应
- homepage：项目主页
- source：框架源代码的托管地址
- tag：与version对应
- source_files：框架源代码的目录、文件、文件类型等规则

```
Pod::Spec.new do |spec|
  spec.name         = "CHTimerLib"
  spec.version      = "1.0.0"
  spec.summary      = "iOS常用的三种定时器"
  spec.description  = <<-DESC
  为iOS中三种常用的定时器NSTimer、CADisplayLink、GCD，添加快捷创建的方法。
                   DESC
  spec.homepage     = "https://chocklee.github.io"
  spec.license      = { :type => 'MIT', :file => 'LICENSE' }
  spec.author       = { "ChanghaoLi" => "chocklee@yeah.net" }
  spec.platform     = :ios, "10.0"
  spec.source       = { :git => "https://github.com/chocklee/CHTimerLib.git", :tag => "#{spec.version}" }
  spec.source_files = "Classes", "Classes/**/*.{h,m}"
  spec.exclude_files = "Classes/Exclude"
  spec.frameworks   = 'Foundation'
  spec.requires_arc = true
end
```

#### 注册Trunk账号

文档如下：https://guides.cocoapods.org/making/getting-setup-with-trunk.html

第一个是激活的收件邮箱，第二个是Github用户名，第三个是描述，可不填写

```
pod trunk register chocklee@yeah.net 'chocklee' --description='macbook pro'
```

你的邮箱会收到一封邮件，打开邮件里面的链接，会有类似 `you can back termainal` 的提示，现在回到终端。

#### 验证podspec文件

验证库是否有错误和警告，根据错误提示修复问题，命令如下：

```
pod lib lint CHTimerLib.podspec
```

忽略警告的命令：

```
pod lib lint CHTimerLib.podspec --allow-warnings
```

当显示 `passed validation` 后，执行下面的命令：

#### 推送库文件到CocoaPods远程仓库

```
pod trunk push CHTimerLib.podspec --allow-warnings
```

提示信息如下：

```
Updating spec repo `master`

--------------------------------------------------------------------------------
 🎉  Congrats

 🚀  CHTimerLib (1.0.0) successfully published
 📅  October 17th, 00:38
 🌎  https://cocoapods.org/pods/testLib
 👍  Tell your friends!
--------------------------------------------------------------------------------
```

#### 搜索自己的库

```
pod search CHTimerLib
```

如果库无法搜索到，是因为本地仓库没有更新的原因，需要删除本地仓库索引文件，命令如下：

```
rm ~/Library/Caches/CocoaPods/search_index.json

pod setup
```

或者是执行命令更新仓库

```
pod repo update
```

#### 更新cocoapods公有库版本

1. 需要修改podspec文件版本号，也就是spec.version的值，提交改动的代码到git仓库，重新打包一个Release版本。

2. 校验podspec文件，push到CocoaPods远程仓库

   ```
   pod cache clean --all // 清除pod缓存
   pod lib lint CHTimerLib.podspec --allow-warnings // 校验
   pod trunk push CHTimerLib.podspec --allow-warnings // 提交到CocoaPods官方仓库
   ```

3. 如果在自己的库中有引用到其他公有库，例如MBProgressHUD、AFNetworking等库，则需要在后面加上命令：–use-libraries

```
// podspec描述文件
  s.subspec 'Category' do |cc|
  cc.source_files = 'Classes/Category/*.{h,m}'
  cc.public_header_files = 'Classes/Category/*.h'
  cc.dependency 'MBProgressHUD'
end

// 引用了第三方库
pod lib lint CHTimerLib.podspec --allow-warnings --use-libraries

pod trunk push CHTimerLib.podspec --allow-warnings --use-libraries
```

使用更新后的公有库

```
pod update --no-repo-update
pod install
```

### 创建本地私有CocoaPods库

使用命令创建私有库，并按照提示回答几个问题：

```
pod lib create [库名]
pod lib create IQFramework
```

- 私有库使用什么语言？
- 私有库中是否需要包含一个demo工程？
- 私有库是否需要包含一个测试框架？
- 私有库的类前缀是什么？

将私有库的文件夹放在要引入的工程的根目录下，并修改工程的podfile，执行pod install命令：

```
target 'InterviewQuestions' do
  use_frameworks!

  pod 'IQFramework', :path => 'IQFramework'
end
```

这里有一个 `path` 关键字，它表示在 `pod install` 执行时，在指定的路径下寻找 `.podspec` 文件。

### 创建远程私有CocoaPods库

1. 创建私有 CocoaPods 远程索引库（在Github上创建私有索引库项目）；

   查看本地索引库，执行如下命令：

   ```
   pod repo
   ```

   执行后会看到，官方的索引库信息：

   ```
   master
   - Type: git (master)
   - URL:  https://github.com/CocoaPods/Specs.git
   - Path: /Users/chanli/.cocoapods/repos/master
   ```

   添加私有远程索引库至本地，执行如下命令：

   此命令有两个参数，1.私有远程索引库项目名，2.Github上创建的索引库地址

   ```
   pod repo add [私有远程索引库项目名] [GitHub上创建的索引库地址]
   pod repo add MyPodRepositorySpecs https://github.com/chocklee/MyPodRepositorySpecs.git
   ```

   添加完成后，可到 `~/.cocoapods/repos` 路径下查看。

2. 创建私有 Git 远程仓库；

   在GitHub上创建一个项目私有库，用来存放真正需要pod安装的代码，**这个库是真正的代码库，第一步创建的是索引库，是为项目库做索引的**

3. 创建 Pod 所需要的项目工程文件，并上传到 Git 远程私有库；

   ```
   pod lib create [库名]
   pod lib create IQFrameworkSwift
   ```

4. 验证 `podspec` 描述文件；

   ```
   pod lib lint
   pod lib lint --allow-warnings (warning 验证不通过时使用此命令)
   ```

5. 向私有 CocoaPods 远程索引库提交 `podspec` 描述文件；

   ```
   pod repo push [私有远程索引库项目名] [.podspec文件] --allow-warnings
   pod repo push IQFrameworkSwift IQFrameworkSwift.podspec --allow-warnings
   ```

6. 使用 Pod 库；

   在podfile中添加远程私有索引库地址：

   ```
   source 'https://github.com/chocklee/MyPodRepositorySpecs.git'
   
   target 'InterviewQuestions' do
     use_frameworks!
   
     pod 'IQFrameworkSwift'
   end
   ```



