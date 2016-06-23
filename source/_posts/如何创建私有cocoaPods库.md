---
title: 如何创建私有cocoaPods库
date: 2016-03-08 17:55:09
tags:
 - cocoaPods
 - iOS
 - git
toc: true
---
   ** 如何创建私有cocoaPods库:**
<!-- more -->

<The rest of contents | 余下全文>
## 创建一个私有的Spec Repo
1. 何为Spec Repo?
  1.拿cocoaPods公共库来说,他是所有的Pods的一个索引，就是一个容器，所有公开的Pods都在这个里面，他实际是一个Git仓库remote端在GitHub上，但是当你使用了Cocoapods后他会被clone到本地的`~/.cocoapods/repos`目录下，可以进入到这个目录看到___master___文件夹就是这个官方的Spec Repo了。这个master目录的结构是这个样子的:
  ```
  ├── Specs
      └── [SPEC_NAME]
          └── [VERSION]
              └── [SPEC_NAME].podspec
  ```
  2.我们平时执行pod命令行指令时,如果后边加上`--no-repo-update`就会不更新本地索引库直接进行pod的更新,这样速度会快些,因为我们不更新Spec Repo,即不需要把所有公开的Pods都更新一遍,这样虽然能提升速度,但不能跟进Pods的最新版本.
  3.我们需要创建一个私有的SpecRepo,不能放在公开的Pods里供所有人使用,因此我们需要创建一个类似于master的私有Spec Repo，这里我们没有必要把现有的公开Pods都copy一份。所以重新创建一个Spec仓库(公司现在已经有自己的Spec库)。
2. 如何创建?
  1.上面说了他只是一个Git仓库,因此我们需要自己搭建一个Git仓库,然后执行下边的命令:

    ```
  命令行格式:
  #pod repo add [Private Repo Name] [Spec Repo URL]
  例如:
  $ pod repo add TestSpecRepo https://github.com/FsThatOne/TestSpecRepo.git
    ```
  2.此时如果成功的话进入到`~/.cocoapods/repos`目录下就可以看到TestSpecRepo这个目录了。至此第一步创建私有Spec Repo完成。
---
## 制作 CocoaPods 依赖库
1. 每个 Pods 依赖库必须有且仅有一个名称和依赖库名保持一致，后缀名为`.podspec`的描述文件。这里我们依赖库的描述文件名称应该为 `MyTestPods.podspec`。
    1.创建这个文件有两种途径:
      1.复制现有的podspec,修改对应的参数
      2.命令行创建:
      ```
      pod spec create MyTestPods
      ```
    2.创建出 `MyTestPods.podspec` 文件后，我们打开可以发现，该文件是 ruby 文件，里面有很多的内容，但是大多数都是我们不需要的，所以我们只需要根据项目的情况保留关键的一些内容就行：
      ```
      Pod::Spec.new do |s|

        s.name         = "MyTestPods"
        s.version      = "0.0.1"
        s.summary      = "A description of MyTestPods."

        s.description  = <<-DESC
                         A longer description of MyTestPods in Markdown format.
                         DESC

        s.homepage  = "https://github.com/FsThatOne/MyTestPods"

        s.license  = "MIT"

        s.author  = { "王正一" => "fsymbio@icloud.com" }

        s.source_files  = "MyTestPods/*.swift"

        s.framework  = "UIKit"

        s.requires_arc = true
      end

      ```
2.第二步,___将我们从项目抽取出来的可复用框架放到MyTestPods中___,然后我们需要对刚刚加的pod进行验证,在实际操作中,这一步可能遇到的问题最多,我会针对我遇到的情况日后慢慢作出补充.执行命令行:
  ```
  pod lib lint
  ```
  成功时会显示以下输出:
  ```
  -> MyTestPods (0.0.1)

  MyTestPods passed validation.
  ```

3.验证成功之后,我们只需要push到远程库,打上tag(这里的tag要与.podspec文件中的s.version一致),至此,我们的CocoaPods依赖库就创建成功了.

  ---

## 添加我们的 Podspec 到我们的 repo
1.执行命令行:
  ```
  pod repo push TestSpecRepo MyTestPods.podspec
  ```
  完成后可以看到效果:
  ```
  Validating spec
   -> MyTestPods (0.0.1)

  Updating the `MyTestPods' repo

  Already up-to-date.

  Adding the spec to the `MyTestPods' repo

   - [Add] MyTestPods (0.0.1)

  Pushing the `TestSpecRepo' repo

  To https://github.com/FsThatOne/TestSpecRepo.git
     9f32092..8d0ced5  master -> master
  ```

2.之后我们再进入~/.cocoapods/repos/TestSpecRepo,就可以看到我们创建的 Spec Repo 了.

  ---

## 测试私有pod.
```
pod search MyTestPods

-> MyTestPods (0.0.1)
   A description of MyTestPods.
   pod 'MyTestPods', '~> 0.0.1'
   - Homepage: https://github.com/FsThatOne/MyTestPods
   - Source:   https://github.com/FsThatOne/MyTestPods.git
   - Versions: 0.0.1 [TestSpecRepo repo]
```
接下来就是`pod install`了,如果在安装的时候显示没有找到:
```
Unable to find a specification for `MyTestPods`
```
就尝试在`pod file`中添加依赖库时指定url,或者在该文件中指定源:
```
#官方Cocoapods的源
source 'https://github.com/CocoaPods/Specs.git'
#本地私有源
source 'https://github.com/FsTHatOne/TestSpecRepo.git'
```
---
## 参考博客:
[CocoaPods创建私有Pods](http://www.liuchungui.com/blog/2015/10/19/cocoapodschuang-jian-si-you-pods/)

[如何创建私有 CocoaPods 仓库](http://www.jianshu.com/p/ddc2490bff9f)

[使用Cocoapods创建私有podspec](http://blog.wtlucky.com/blog/2015/02/26/create-private-podspec/)
