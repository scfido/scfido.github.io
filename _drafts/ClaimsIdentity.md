转帖：http://www.cnblogs.com/savorboard/p/aspnetcore-identity.html

# 前言

在 ASP.NET Core 中，仍然沿用了 ASP.NET里面的 Identity 组件库，负责对用户的身份进行认证，总体来说的话，没有MVC 5 里面那么复杂，因为在MVC 5里面引入了OWIN的东西，所以很多初学者在学习来很费劲，对于 Identity 都是一头雾水，包括我也是，曾经在学 identity 这个东西前后花了一个多月来搞懂里面的原理。所以大部分开发者对于 Identity 并没有爱，也并没有使用它，会觉得被绑架。

值得庆幸的是，在 ASP.NET Core 中，由于对模块的抽象化逐渐清晰，以及中间件的使用，这使得 Identity 的学习和使用路线变得更加平易近人，下面就让我们一起来看看吧。

# Getting Started

在开始之前，让我们先忘记它和`Entity Framework`的关系，也忘记它和`Authentication`的关系，我们先学习几个英语单词。

有这么几个“单词”你可能需要弄明白：

## 1： Claims

大家应该都知道身份证长什么样子的，如下：

![](/assets/drafts/ClaimsIdentity/img/2017-12-12-10-49-16.png)

其中，**姓名**：奥巴马；**性别**：男；**民族**：肯尼亚；**出生**：1961.08.04，等等这些身份信息，可以看出都是一个一个的`键值对`，那如果我们想在程序中存这些东西，怎么样来设计呢？对，你可能想到了使用一个字典进行存储，一个Key，一个Value刚好满足需求。但是Key，Value的话感觉不太友好，不太面向对象，所以如果我们做成一个对象的话，是不是更好一些呢？最起码你可以用vs的智能提示了吧，我们修改一下，改成下面这样：
```cs
//我给对象取一个名字叫`Claim`你没有意见吧
public class Claim
{
    public string ClaimType { get; set; }

    public string ClaimValue { get; set; }
}
```
`ClaimType` 就是Key，`ClaimValue`就代表一个Value。这样的话，刚好可以存储一个键值对。这时候姓名：奥巴马是不是可以存进去了。

微软的人很贴心，给我们准备了一些默认的`ClaimType`呢？很多常用的都在里面呢，一起看看吧：

![](/assets/drafts/ClaimsIdentity/img/2017-12-12-11-10-32.png)

> 这里延伸第一个知识点：ClaimTypes



为了阅读体验，截图我只放了一部分哦。可以看到有什么Name，Email，Gender，MobilePhone等常用的都已经有了，其他的还有很多。细心的读者可能注意了，它的命名空间是`System.Security.Claims`，那就说明这个东西是.net 框架的一部分，嗯，我们暂时只需要知道这么多就OK了。

Claim 介绍完毕，是不是很简单，其他地方怎么翻译我不管，在本篇文章里面，它叫 **“证件单元”**。

## 2： ClaimsIdentity

在有了 **“证件单元”** 之后，我们就用它可以制造一张身份证了，那么应该怎么样制造呢？有些同学可能已经想到了，对，就是新建一个对象，然后在构造函数里面把身份证单元传输进去，然后就得到一张身份证了。我们给这张身份证取一个英文名字叫 **“ClaimsIdentity”**，这个名字看起来还蛮符合的，既有 Claims 表示其组成部分，又有表示其用途的 Identity（身份），很满意的一个名字。

实际上，在现实生活中，我们的身份证有一部分信息是隐藏的，有一部分是可以直接看到的。比如新一代的身份证里面存储了你的指纹信息你是看不到的，这些都存储在身份证里面的芯片中，那能看到的比如姓名啊，年龄啊等。我们在设计一个对象的时候也是一样，需要暴露出来一些东西，那这里我们的 ClaimsIdentity 就暴露出来一个 Name，Lable等。

我们造的身份证（ClaimsIdentity）还有一个重要的属性就是类型（AuthenticationType），等等，AuthenticationType是什么东西？看起来有点眼熟的样子。我们知道我们自己的身份证是干嘛的吧，就是用来证明我们的身份的，在你证明身份出示它的时候，其实它有很多种形式载体的，什么意思呢？比如你可以直接拿出实体形式的身份证，那也可以是纸张形式的复印件，也可以是电子形式的电子码等等，这个时候就需要有一个能够表示其存在形式的类型字段，对，这个AuthenticationType就是干这个事情的。

然后我们在给我们的身份证添加一些润色，让其看起来好看，比如提供一些方法添加 Claims 的，删除 Claims的，写到二进制流里面的啊等等，最终我们的身份证对象看起来基本上是这样了：

```cs
public class ClaimsIdentity
{
    public ClaimsIdentity(IEnumerable<Claim> claims){}
    
    //名字这么重要，当然不能让别人随便改啊，所以我不许 set，除了我儿子跟我姓，所以是 virtual 的
    public virtual string Name { get; }
    public string Label { get; set; }
    
    //这是我的证件类型，也很重要，同样不许 set
    public virtual string AuthenticationType { get; }
    
    public virtual void AddClaim(Claim claim);
    
    public virtual void RemoveClaim(Claim claim);
    
    public virtual void FindClaim(Claim claim);
}
```

嗯，到这里，我们的身份证看起来似乎很完美了，但是从面向对象的角度来说好像还少了点什么东西？ 对~，还是抽象，我们需要抽象出来一个接口来进行一些约束，约束什么呢？既然作为一个证件，那么肯定会涉及到这几个属性信息：  
1、名字。2、类型。3、证件是否合法。  
反应到接口里面的话就是如下，我们给接口取个名字叫：**“身份(IIdentity)”**：

> 这里延伸第二个知识点：IIdentity接口。

```cs
// 定义证件对象的基本功能。
public interface IIdentity
{
    //证件名称
    string Name { get; }
    
    // 用于标识证件的载体类型。
    string AuthenticationType { get; }
    
    //是否是合法的证件。
    bool IsAuthenticated { get; }
}
```

所以我们的 ClaimsIdentity 最终看起来定义就是这样的了：

```cs
public class ClaimsIdentity : IIdentity
{
    //......
}
```
ClaimsIdentity 介绍完毕，是不是发现也很简单，其他地方怎么翻译我不管，在本篇文章里面，它叫 **“身份证”**。

## 3： ClaimsPrincipal

有了身份证，我们就能证明我就是我了，有些时候一个人有很多张身份证，你猜这个人是干嘛的？ 对，不是黄牛就是诈骗犯。

但是，有些时候一个人还有其他很多种身份，你猜这个人是干嘛的？这就很正常了对不对，比如你可以同时是一名教师，母亲，商人。如果你想证明你同时有这几种身份的时候，你可能需要出示教师证，你孩子的出生证，法人代表的营业执照证。

在程序中，一个身份证不仅仅代表你这个人了，而是代表一个身份，是证明你自己的主要身份哦。如果一个人还有其他很多种身份，这个时候就需要有一个东西（载体）来携带着这些证件了对吧？OK，我们给需要携带证件的这个对象取一个贴切点的名字，叫 **“证件当事人（ClaimsPrincipal）”** 吧。

以下是 Principal 这个单词在词典给出的解释，我用它你应该没意见吧：

> principal  ['prɪnsəpl]  
adj. 主要的；资本的  
n. 首长；校长；资本；当事人

这个时候可能有同学会问了，是不是应该叫ClaimsIdentityPrincipal比较好呢？嗯，我也觉得应该叫 ClaimsIdentityPrincipal 可能更好一点，或许微软的人偷懒了，简写成了ClaimsPrincipal。

知道其功能后，代码就很好写了，和上面ClaimsIdentity一样的套路：

```cs
public class ClaimsPrincipal 
{
    //把拥有的证件都给当事人
    public ClaimsPrincipal(IEnumerable<ClaimsIdentity> identities){}
    
    //当事人的主身份呢
    public virtual IIdentity Identity { get; }
    
    public virtual IEnumerable<ClaimsIdentity> Identities { get; }
    
    public virtual void AddIdentity(ClaimsIdentity identity);
    
    //为什么没有RemoveIdentity ， 留给大家思考吧？
}
```

当时人看起来也几乎完美了，但是我们还需要对其抽象一下，抽象哪些东西呢？ 作为一个当事人，你应该有一个主身份吧，就是你的身份证咯，可能你还会用到角色（角色后面会详细介绍，这里你知道有这么个东西就行了）。

> 这里延伸第三个知识点：IPrincipal 接口。

```cs
public interface IPrincipal
{
    //身份
    IIdentity Identity { get; }
    
    //在否属于某个角色
    bool IsInRole(string role);
}
```

然后，我们的 证件当事人 看起来应该是这样的：

```cs
public class ClaimsPrincipal : IPrincipal 
{
   //...
}
```

`ClaimsPrincipal` 介绍完了，也很简单吧？ 其他地方怎么翻译我不管，在本篇文章里面，它叫 **“证件当事人”**。

想在，我们已经知道了 “证件单元（Claims）” ， “身份证（ClaimsIdentity）” ， “证件当事人（ClaimsPrincipal）”，并且整理清楚了他们之间的逻辑关系，趁热打铁，下面这个图是一个identity登入部分的不完全示意图，虚线圈出来的部分应该可以看懂了吧：

![](/assets/drafts/ClaimsIdentity/img/2017-12-12-11-19-20.png)


可以看出，首先我们在app这边有一些证件单元，然后调用`ClaimsIdentity`把证件单元初始化为一个身份证，然后再把身份证交给证件当事人由其保管。

才把 Getting Started 写完，发现已经这么长了，所以打算写成一个系列了，可能3 - 4篇吧。

# 总结

好了，本篇就先介绍到这里，在本篇博客中，我们学会了几个英文单词，并且知道了这些英文单词在程序中是扮演这怎么样一个对象。并且根据图我们知道了这些对象在整个认证系统种处在怎么样一个位置。 我发现如果想把 identity 讲清楚仅仅靠这一篇博客是不够的，下一篇我们将对`.NET Authentication`中间件进行抽丝剥茧，直到掌握.NET的整个认证系统后，我们再来看一下 Identiy 到底和 Entity Framework 有着怎样的爱恨情仇。

这仅仅是一个开始，大家如果觉得本篇博客对您有帮助的话，感谢您的【推荐】，如果你对 .NET Core 感兴趣可以关注我，我会定期在博客分享关于 .NET Core 的学习心得。