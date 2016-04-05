---    
layout:     post    
title:      一个成功的git 分支模型    
category: blog    
description:  翻译A successful Git branching model, 目的，精读文章，练习英语和打字？(maybe)
---    


这是一篇文章的翻译 , 英文原文：[A successful Git branching model](http://nvie.com/posts/a-successful-git-branching-model/)，为什么要做翻译呢？大概有很多理由吧，比如精读文章，比如练习英语，或者练习打字？(maybe)

这篇文章我将介绍一种成功的开发模式，我已经将它引入到我的项目(无论是工作的还是私人的)当中一年多了，而且它非常的成功，我想把它写出来已经有一段时间了，但我一直没有时间像现在这样做一次透彻的讨论。我仅仅讨论分支战略和发行管理的内容，而不涉及到任何项目的细节。    
本文章以Git 作为我们所有源代码的版本管理工具。  

![git-model](/images/gitflow/git-model@2x.png) 

## 为什么是Git？
对于Git与其它集中式源代码控制系统的利弊和缺点的比较的彻底讨论，请看[这里](http://git.or.cz/gitwiki/GitSvnComparsion)。这里总是有很多火药味很浓的战争。作为一个开发者，比起当今所有其它工具来说，我更喜欢Git。
Git真正改变了开发者对于合并和分支的思考方式。我是从经典的CSV/Subversuin转过来的，那时候，合并/分支和一些隔一会儿你要做一次的行为总是被认为有点吓人(小心合并冲突啊，会咬人的)。
      
但是使用Git，这些行为是一件非常廉价和简单的事情，它们真的会被认为是你日常工作流的核心部分之一。举个例子，在	CSV/Subverision的书籍中，分支与合并总会在后面一些章节中被讨论(提供给进阶使用者)，然而在每一本Git的书中，前三章就已经提高了(对于初级使用者)。   

它的简单和重复的自然特性而引发的结果是：分支和合并不再是什么可怕的事情。版本控制工具应该是有助于分支/合并而不是其它。  

工具已经说得够多了，让我们转向开发模型吧。这个我将展示的模型实质上没有更多的东西，就是一系列 每个团队成员必须遵从的程序，以保证进入一个可管理的软件开发流程。  
  
  

## 分布式而不是集中式  

![centr-decentr](/images/gitflow/centr-decentr@2x.png)   

我们使用的仓库设置即这种有一个中心“truth”仓库的分支模型工作的很好，注意这个仓库只是被认为是中心的(git是一个分布式版本控制工具(DVCS),其实没有一个这样技术层面上的中心仓库)我们将把这个仓库取名为“origin”这个所有的Git用户都熟悉的名字。 
 
每个开发者都从origin中拉取或者推送，但是除了中心推拉关系，每个开发者也可能从其它次级团队的点那里拉取修改。举例来说，这对于在一起工作的两个或者更多的关注于一个大的新特性的开发者们来说是非常有用的，可以不必过早地把尚在过程中的工作推送到Origin。在上图中，这里有很多次级团队，如Alice和Bob，Alice和David，以及Clair和Dvaid。  
技术上来说，这意味着Alice只需要定义一个Git远端，起名叫bob，并把它指向Bob的仓库就行了，反之亦然。  

## 主分支  

![main-branches](/images/gitflow/main-branches@2x.png)


 这个开发模型的核心部分是被已经存在的其它模型所启发的，中心仓库有两个具有无穷生命线的分支。   
 
 *  master
 *  develop   
 
 在origin的master分支对于每个Git 使用者都非常熟悉。与master并行存在的另一个分支叫做develop。   
 
 我们认为origin/master 是一个这样的主分支：它最新的源代码(source code of HEAD)处于 “产品预备”(译者注：随时能够发行一个产品版本来)的状态。  
 
 我们认为origin/develop是这样一个主分支：它最新的源代码反映了下一个发行版本的最新交付的开发变更。有时候叫它“integration“分支，这是任何夜晚自动生成的构建来源之处。  
 
 当develop分支的源代码到了一个稳定点准备发行的时候，所有的变更无论如何都应该合并回master，并且用一个发行版本号来打标签。如何做的细节晚些时候再讨论。    
 
 因此，每次变更合并回master，将会有一个新的产品发行版被定义。我们趋向于对此非常严格，所以理论上，当每次有master的提交时，我们应该用一个Git的钩子脚本去自动的生成和推送我们的软件到产品服务器。  
 
## 支撑型分支  
主分支master和develop旁边，我们的开发模型使用了一系列支撑型分支以达到这样的目标：团队成员之间并行开发，特性的易于跟踪，为产品release做准备以及辅助快速修复线上产品的问题。与主分支不同，这些分支总有一个有限的生命周期，直到它们最终被移除。

我们可能使用的不同分支类型如下：   
*   Feature branches(特性分支)    
*   Release branches(发布分支)    
*   Hotfix branches(修复分支)   

这些每一个分支都有特别的目的并且绑定了严格的规则比如哪些分支将成为它们的源分支而哪些分支将成为它们的合并目标，我们马上将这些流程走一遍。  

从技术视角来看，这些分支并不特殊。分支类型取决于我们怎么使用它们。它们当然普通的Git分支。

### 特性分支(Feature branches)

![Feature branches](/images/gitflow/fb@2x.png)

特性分支分裂于    
develop    
必须合并到    
develop    
分支名称惯例    
任何名称 ,除了master, develop, release-*, or hotfix-*

特性分支(或者有时叫做主题分支)用来开发一个为了即将到来或者遥远未来发行版本的新特性。当我们开始一个新特性的开发时，在那个时间点上，这个新特性的目标发布版本何时合并也许还是未知的。   

特性分支的本质就是存在于特性的开发阶段，并将最终合并回develop(确切的添加此新特性的到即将到来的发行版本)或者废弃(那些令人失望的实验场景)。    

特性分支一般仅仅存在于开发者的仓库中，而不在origin中(译者此处有异议，保存在origin中有一部分原因是backup)    


#### 创建一个特性分支

当开始一个新分支的工作时，分支分裂于develop分支  
{% highlight Git  %}
$ git checkout -b myfeature develop
Switched to a new branch "myfeature"
{% endhighlight %}

#### 合并一个完成的特性到develop

完成特性也许合并回develop 分支用于确切的添加此新特性的到即将到来的发行版本。
{% highlight Git  %}
$ git checkout develop
Switched to branch 'develop'
$ git merge --no-ff myfeature
Updating ea1b82a..05e9557
(Summary of changes)
$ git branch -d myfeature
Deleted branch myfeature (was 05e9557).
$ git push origin develop
{% endhighlight %}

 --no-ff 标记使得合并总是会创建一个新的提交对象，即使这个合并应该是一个快进行为。这样避免了丢失一个特性分支的历史存在信息而且将这个特性的所有提交作为整合成一组，请比较：
 ![merge-without-ff](/images/gitflow/merge-without-ff@2x.png)   
 
在后者的场景中，想从Git的历史中看出哪些提交是一起实现某一个特性的简直是不可能的，你不得不手动的读取所有的日志信息。在后者的情况下，还原一个完整的特性(例如一组提交)是一个真正的头疼的问题。如果 --no-ff  标记被使用的话，将变得更容易。  

  
没错，你将创建更多一些(空的)提交，但收益远远大于代价。   

### Release branches
分支分裂自    
develop     
必须合并回     
develop和master     
命名惯例     
release-*   

Release分支用于准备一个新的产品发行版， They allow for last-minute dotting of i’s and crossing t’s(不会翻译).进一步的，它允许次要bug的修改和为了发行而准备的元数据(版本号，生成日期等等)，当在Release分支做完所有的这些工作之后，develop分支将被清空以便于接受下一个大发行版本的特性。 
 
从develop分支生成一个新的release分支的关键时刻取决于当开发(几乎)已经达到新发行版本的期望状态。那时至少所有的那些即将被发布的特性必须及时的合并回develop分支。所有的未来的发行的版本的特性则不需要，它们必须等到它们的release分支产生。  
 
一个release分支开始的时候，才应该分配一个版本号而不是更早。那个时候之前，develop分支涉及到的变更是为了下一个发布，而不清楚下一个发布究竟是0.3还是1.0，直到release分支开始。这个决定是在release分支的开始做的，并且由项目的版本号规则执行生成。     

#### 创建release 分支 

release分支是由develop分支创建而来的，举例来说，版本1.1.5是当前的的产品发行版本而我们有一个大的发布即将到来。develop的状态已经为下一个发行版本准备好了，我们决定它将是版本1.2(相对于版本1.1.6或者2.0)，所以我们建立分支并且给这个release分支起一个带有新的版本号的名字。

{% highlight Git  %}
$ git checkout -b release-1.2 develop
Switched to a new branch "release-1.2"
$ ./bump-version.sh 1.2
Files modified successfully, version bumped to 1.2.
$ git commit -a -m "Bumped version number to 1.2"
[release-1.2 74d9424] Bumped version number to 1.2
1 files changed, 1 insertions(+), 1 deletions(-)
{% endhighlight %}

建立一个新分支并选择它以后，我们创建版本号。   
这里 bump-version.sh是一个虚拟的shell脚本，用于修改一些从工作区复制到新版本的文件(这当然可以是一个手动修改--总之一些文件被修改)然后，创建的版本号将被提交。    

这个新分支将存在一段时间，直到发布被肯定的推出。这段时间内，bug的修复将会被应用在这个分支上(而不是在develop分支上)。在这里添加一个大的特性是被严格禁止的。它们必须合并回develop，而后，等待下一个大的发布。  
   
#### 完成一个release分支。
当一个release分支准备成为一个真正的发布，一些行为必须被执行。首先，release分支必须被合并回master(记得master上的每一次提交都是一个新的发布吧)，接着，这个在master上的提交必须被打上标签，以便于未来方便的引用历史版本。最后，在release分支上的修改要被合并回develop上，以保证未来的发行版本包含这些bug的修复。   

Git上的前两步：
{% highlight Git  %}
$ git checkout master
Switched to branch 'master'
$ git merge --no-ff release-1.2
Merge made by recursive.
(Summary of changes)
$ git tag -a 1.2
{% endhighlight %}
    
现在发布完成，为未来的引用打好了标签。(也许你也想用-s 或者-u 标记给你的tag加密)   

为了保持release分支上的变更，我们需要合并回develop分支。   
在Git上
{% highlight Git  %}   
$ git checkout develop
Switched to branch 'develop'
$ git merge --no-ff release-1.2
Merge made by recursive.
(Summary of changes)
{% endhighlight %}
这一步也许会导致合并冲突(甚至也许从我们修改了版本号之后)，如果这样的话，修复提交它。   
现在我们真正完成了，release分支要被移除了，因为我们不需要它了。   
{% highlight Git  %} 
$ git branch -d release-1.2
Deleted branch release-1.2 (was ff452fe).
{% endhighlight %}

### 热修复分支Hotfix branches 
![hotfix-branches](/images/gitflow/hotfix-branches@2x.png)

热修复分支   
产生自master   
必须被合并回develop和master   
命名惯例hotfix-*   


热修复分支非常类似于release分支，它们都意味着准备一个新的产品发布，尽管它是非预期的。它们出现的必要性在于当线上产品版本产生了非预期状态。当一个产品版本的紧急bug必须被立即解决时，一个热修复分支将会产生自相应的tag，这个tag就是在master分支上的标记过的产品版本。   
 
本质上这是一个(在develop分支上的)团队成员可以继续工作，同时另一个人在准备快速的产品修复。   

#### 创建一个热修复分支
热修复分支创自于master分支。举例：1.2版本是当前的产品发行版本，它运行于生产环境。由于一个严重 bug它出问题了。但是在develop分支的修改还不稳定。我们需要生成一个热修复分支并开始修复问题。
{% highlight Git  %}
$ git checkout -b hotfix-1.2.1 master
Switched to a new branch "hotfix-1.2.1"
$ ./bump-version.sh 1.2.1
Files modified successfully, version bumped to 1.2.1.
$ git commit -a -m "Bumped version number to 1.2.1"
[hotfix-1.2.1 41e61bb] Bumped version number to 1.2.1
1 files changed, 1 insertions(+), 1 deletions(-)
{% endhighlight %}
不要忘记建立好分支后生成一个版本号码。  

然后，修复bug并提交，也许是一次或者多次。  
{% highlight Git  %}
$ git commit -m "Fixed severe production problem"
[hotfix-1.2.1 abbe5d6] Fixed severe production problem
5 files changed, 32 insertions(+), 17 deletions(-)
{% endhighlight %}    

#### 完成热修复分支
当完成后，bug修复需要被合并回master，同时为了保障在下一个release版本中也包含bug修复，也需要被合并回develop。这与release分支的完成完全类似。   
首先，更新maste，给release打标签
{% highlight Git  %}
$ git checkout master
Switched to branch 'master'
$ git merge --no-ff hotfix-1.2.1
Merge made by recursive.
(Summary of changes)
$ git tag -a 1.2.1
{% endhighlight %}  
(也许你也想用-s 或者-u 标记 给你的tag加密)

   
其次在develop中也包含bugfix
{% highlight Git  %}
$ git checkout develop
Switched to branch 'develop'
$ git merge --no-ff hotfix-1.2.1
Merge made by recursive.
(Summary of changes)
{% endhighlight %}
这里有一个打破规则的异常是：当一个release 分支当前已经存在，热修复变更必须合并到release分支当中，而不是develop。归并bugfix到release分支也将最终导致当release分支完成时候bugfix合并到develop分支。   
(如果develop分支的工作急需bugfix而不能等到release分支完成，你也需要安全的合并到develop分支当中)
    
最后，移除这个临时分支。
{% highlight Git  %}
$ git branch -d hotfix-1.2.1
Deleted branch hotfix-1.2.1 (was abbe5d6).
{% endhighlight %}

## 总结： 
其实对于分支模型这里没有什么真正震惊的新东西，但在我们的项目中文章开头处的大图非常有用.它体现了一个优雅的模型。它容易理解同时使得团队成员开发时有共同关于分支和发布流程的理解。







