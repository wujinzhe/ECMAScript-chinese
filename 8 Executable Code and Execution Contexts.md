# 8 执行代码与执行上下文

### 8.1 词法环境（Lexical Environment）

**词法环境**是用于定义标识符与特定变量和函数在ECMAScript代码的词法嵌套结构上的关联的类型规范。**词法环境**由一个**环境记录项**和一个可能为空的外部**词法环境**引用组成。一般情况下**词法环境**与ECMAScript代码的某些特定语法结构相关联，比如函数声明、语句块或TryStatement的Catch子句，每次执行这些代码都会创建一个新的**词法环境**。


**环境记录项**记录了对应**词法环境**范围内创建的标识符绑定。它被称为**词法环境**的**环境记录项**。

**外部词法环境引用**用于表示词法环境的逻辑嵌套关系模型，（内部）**词法环境**的外部引用是逻辑上包含内部**词法环境**的**词法环境**。外部**词法环境**自然也可能有多个内部**词法环境**。例如，如果一个 FunctionDeclaration 包含两个嵌套的 FunctionDeclaration，那么每个内嵌函数的**词法环境**都是外部函数本次执行所产生的**词法环境**。

**全局环境**是没有外部环境的**词法环境**。**全局环境**的**外部环境**引用为空。**全局环境**的**环境记录项**会预先定义一些标识符绑定并且关联一个全局对象，这个全局对象提供一些全局环境的标识符绑定，在代码执行的时候，全局对象可以添加额外的属性并且可以修改原始的属性

**模块环境**是一个**词法环境**，它包含模块的顶层声明的绑定。它还包含模块显式导入的绑定。**模块环境**的外部环境是一个**全局环境**。

**函数环境**是一个**词法环境**，它与调用函数对象是一样的，**函数环境**会创建一个this绑定，**函数环境**会获取支持super方法调用的所需状态

**词法环境**和**环境记录项**是纯粹的规范机制，而不需要 ECMAScript 的实现保持一致。ECMAScript程序不可能直接访问或者更改这些值。

#### 8.1.1 环境记录项（Environment Records）

本规范中的**环境记录项**的两种基本类型有：**声明式环境记录项**和**对象式环境记录项**，**声明式环境记录项**用于定义ECMAScript语法元素的作用，比如函数声明，变量声明和catch语句，它们直接将标识符与值进行绑定，**对象式环境记录项**用于定义ECMAScript元素的作用，比如WithStatement(它将关联的标识符与一些对象的属性绑定起来)，**全局式环境记录项**和**函数式环境记录项**是专门用于脚本全局声明和函数内部顶级声明的。

出于规范目的，环境记录项的值是记录规范类型的值，可以认为存在于简单的面向对象级别中，其中环境记录是一个抽象类，包含三个具体的子类:声明式环境记录、对象式环境记录和全局式环境记录。函数式环境记录和模块式环境记录是声明式环境记录的子类。抽象类包含表14中定义的抽象方法。这些抽象方法对于每个具体的子类都有不同的具体算法。

Table 14: 环境记录的抽象方法


方法| 作用
------|---
HasBinding(N) | 确定一个环境记录是否有字符串值N的绑定，如果有，返回true;如果没有，返回false。
CreateMutableBinding(N, D) | 在环境记录中创建一个新的但未初始化的可变绑定。字符串值N是绑定名的文本。如果布尔参数D为真，则绑定可能随后被删除。
CreateImmutableBinding(N, S)|在环境记录中创建一个新的但未初始化的不可变绑定。字符串值N是绑定名的文本。如果S为真，那么在初始化它之后尝试设置它总是会抛出一个异常，不管引用该绑定的操作的严格模式设置是什么。
InitializeBinding(N, V)|在环境记录中设置已存在但未初始化的绑定的值。字符串值N是绑定名的文本。V是绑定的值（可以是任何类型）
SetMutableBinding(N, V, S)|在环境记录中设置已存在的可变绑定的值。字符串值N是绑定名的文本。V是绑定的值，可以为任何类型。S是布尔类型。如果S为true那么则无法设置绑定，并且抛出一个TypeError异常。
GetBindingValue(N, S)|从环境记录返回已经存在的绑定的值。字符串值N是绑定名的文本。S用于判断是否是严格模式，或者其他需要严格模式的引用。如果S为true，那么绑定不存在，抛出一个ReferenceError异常。如果绑定存在但未初始化，则抛出ReferenceError，而不考虑S的值。
DeleteBinding(N)|从环境记录中删除绑定。字符串值N是绑定名的文本。如果存在N的绑定，则删除绑定并返回true。如果绑定存在但无法删除，返回false。如果绑定不存在，返回true。
HasThisBinding()|确定环境记录是否有this绑定。如果有返回true，否则返回false。
HasSuperBinding()|确定环境记录是否有super方法绑定。如果有返回true，否则返回false。
WithBaseObject()|如果此环境记录被with关联，则返回with对象。否则返回undefined。

##### 8.1.1.1  声明式环境记录（Declarative Environment Records）

每个声明式环境记录关联的范围都包括变量、常量、let、类、模块、导入和函数声明。声明式环境记录范围内包含的定义的标识符。


声明式环境记录的具体规范有下面方法定义。

###### 8.1.1.1.1 HasBinding ( N )

这个环境记录具体的方法HasBinding是为了在声明式环境记录中简单的确认某个标识符是否被在当前环境记录中被绑定

1. 当该函数被调用的时候，让envRec成为一个新的声明式环境记录
2. 如果名为N（参数）的标识符被绑定，返回true，否则返回false

###### 8.1.1.1.2 CreateMutableBinding ( N, D )

该方法为未初始化的名为N的标识符创建一个新的可变绑定，如果D的值为true，则新绑定是可删除的

1. 当该函数被调用的时候，定义envRec成为一个新的声明式环境记录
2. 前提：envRec没有绑定过N
3. 在envRec中为N创建一个可变并且未初始化的绑定。如果D为true则这个绑定可以被DeleteBingding删除
4. 返回 NormalCompletion(empty)

###### 8.1.1.1.3 CreateImmutableBinding ( N, S )
用于声明式环境记录的具体环境记录方法CreateImmutableBinding为未初始化的名称为N的标识符创建了一个新的不可变绑定。如果布尔参数S的值为true，则新的绑定将被标记为严格绑定。

1. 当该函数被调用的时候，定义envRec成为一个新的声明式环境记录
2. 前提: envRec并没有N的绑定
3. 在envRec中创建一个不可绑定的，未初始化的N标识符，如果S为true，则创建的绑定为严格绑定
4. 返回 NormalCompletion(empty)

###### 8.1.1.1.4 InitializeBinding ( N, V )
该方法用于设置标识符的当前绑定值，将V的值绑定到N标识符上，N必须存在且未初始化绑定

1. 当该函数被调用的时候，定义envRec成为一个新的声明式环境记录项
2. envRec必须存在N，且未初始化
3. 将envRec中的N设置为V
4. 记录envRec中N的绑定已经初始化
5. 返回 NormalCompletion(empty)

###### 8.1.1.1.5 SetMutableBinding ( N, V, S )
SetMutableBinding目的是在声明式环境记录中试图改变名为N的标识符的当前绑定，将N绑定的值设置成V，N的绑定通常已经存在，但在极少数环境下可能不是，如果是一个不可变绑定，当S为true时，抛出TypeError异常

1. 当该函数被调用的时候，envRec为声明式环境记录项
2. 如果envRec没有名为N的绑定，那么
    1. 如果S为true，抛出ReferenceError异常
    2. 执行envRec.CreateMutableBinding(N, true)
    3. 执行envRec.InitializeBinding(N, V)
    4. 返回NormalCompletion(empty)
3. 如果envRec中N的绑定为严格绑定，则S一定为true
4. 如果envRec中N的绑定没有初始化，则报出ReferenceError异常
5. 如果envRec中N的绑定为可变绑定，则将N的值设置为V
6. 如果改变一个不可变绑定，当S为true时，抛出TypeError异常
7. 返回 NormalCompletion(empty)

> 注意 以下代码会在第二步丢失绑定 <br>
> function f(){eval("var x; x = (delete x, 0);")}

###### 8.1.1.1.6 GetBindingValue ( N, S )
该方法简单的返回其名为N的绑定标识符的值

1. 当该函数被调用的时候，envRec为声明式环境记录项
2. 前提：envRec已经绑定了名为N的标识符
3. 如果N是一个未初始化的绑定，则抛出ReferenceError异常
4. 返回envRec中N的值

###### 8.1.1.1.7 DeleteBinding ( N )
该方法只能删除显示指定可被删除的绑定

1. 当该函数被调用的时候，让envRec成为一个新的声明式环境记录
2. 前提：envRec存在名为N的绑定
3. 如果绑定N不能被删除，返回false
4. 删除envRec中的N绑定，返回true

###### 8.1.1.1.8 HasThisBinding ( )
一般情况下声明式环境记录项不提供this绑定

1. 返回 false

###### 8.1.1.1.9 HasSuperBinding ( )
一般情况下声明式环境记录项不提供super绑定

1. 返回 false

###### 8.1.1.1.10 WithBaseObject ( )
声明式环境记录的WithBaseObject对象都为undefined

1. 返回 undefined

##### 8.1.1.2 对象式环境记录（Object Environment Records） 
每个对象式环境记录都有一个关联的对象称作绑定对象，对象式环境记录的绑定的标识符名与绑定对象的属性名相同，属性键不是IdentifierName类型的字符串也不存在绑定的标识符中，无论属性的[[Enumerable]]属性的设置是什么，都包含在集合中。因为属性可以从对象中动态地添加和删除，所以对象环境记录绑定的标识符集可能会作为任何添加或删除属性的操作的副作用而改变。由于这种副作用而创建的任何绑定都被认为是可变绑定，即使相应属性的可写属性的值为false。对象环境记录不存在不可变绑定。

##### 8.1.1.3 函数式环境记录（Function Environment Records）

函数式环境记录是声明式环境记录，用于表示函数的顶级范围，如果函数不是箭头函数，则提供this绑定。如果一个函数不是一个箭头并且它引用了super，那么它的函数式环境记录也包含在函数内执行super方法调用的状态。


函数式环境记录的附加状态字段如下表 

表15

字段名 | 值类型 | 意思
---|---|---
[[ThisValue]]| any |调用函数的this值
[[ThisBindingStatus]] | "lexical"，"initialized"， "uninitialized"| 如果值是“lexical”，则这是一个箭头函数，并且没有本地的this值。
[[FunctionObject]]|Object|创建这个环境记录的函数对象
[[HomeObject]]|undefined or object|如果关联的函数具有super属性访问并且不是一箭头函数，那么[[HomeObject]]就一定是个函数对象。[[HomeObject]]]的默认值为undefined。
[[NewTarget]]|undefined，object|如果这个环境记录是由[[Construct]]内部方法创建的，那么[[NewTarget]]就是是[[Construct]]参数NewTarget的值。否则，为undefined。

函数环境记录支持表14中列出的所有声明性环境记录方法，并为所有这些方法共享相同的规范，除了HasThisBinding和HasSuperBinding。此外，函数环境记录支持表16中列出的方法:

表16：函数式环境记录的附加方法

方法 | 作用
---|---
BindThisValue(V) | 设置[[ThisValue]]的值，并且记录它为已初始化
GetThisBinding() | 返回该环境记录的值的this绑定。如果该绑定没有初始化，则抛出一个ReferenceError。
GetSuperBase()|


###### 8.1.1.3.1BindThisValue ( V )
1. 当方法调用的时候 envRec为函数式环境记录项
2. 前提 envRec.[[ThisBindingStatus]] 不为 "lexical"
3. 如果 envRec.[[ThisBindingStatus]]是"initialized" 则抛出ReferenceError异常
4. 设置envRec.[[ThisValue]] = V
5. 设置envRec.[[ThisBindingStatus]] = 'initialized'
6. return V

8.1.1.3.2HasThisBinding ( )
1. 当方法调用的时候 envRec为函数式环境记录项
2. 如果envRec.[[ThisBindingStatus]] = "lexical", return false; 否则 return true

8.1.1.3.3HasSuperBinding ( )
Let envRec be the function Environment Record for which the method was invoked.
If envRec.[[ThisBindingStatus]] is "lexical", return false.
If envRec.[[HomeObject]] has the value undefined, return false; otherwise, return true.
8.1.1.3.4GetThisBinding ( )
Let envRec be the function Environment Record for which the method was invoked.
Assert: envRec.[[ThisBindingStatus]] is not "lexical".
If envRec.[[ThisBindingStatus]] is "uninitialized", throw a ReferenceError exception.
Return envRec.[[ThisValue]].
8.1.1.3.5GetSuperBase ( )
Let envRec be the function Environment Record for which the method was invoked.
Let home be envRec.[[HomeObject]].
If home has the value undefined, return undefined.
Assert: Type(home) is Object.
Return ? home.[[GetPrototypeOf]]().

##### 8.1.1.4 全局环境记录（Global Environment Records）

全局环境记录用于表示在公共领域中处理的所有ECMAScript脚本元素共享的最外层范围。全局环境记录为内置全局变量(第18条)、全局对象的属性以及脚本中出现的所有顶级声明(13.2.8,13.2.10)提供绑定。

......

属性可以直接在全局对象上创建。因此，全局环境记录的对象环境记录组件可以包含由FunctionDeclaration、GeneratorDeclaration、AsyncFunctionDeclaration、AsyncGeneratorDeclaration或VariableDeclaration声明和作为全局对象属性隐式创建的绑定显式创建的绑定。为了确定哪些绑定是使用声明显式创建的，全局环境记录使用其CreateGlobalVarBinding和CreateGlobalFunctionBinding具体方法维护一个名称绑定列表。

表17中列出了附加字段和表18中列出了附加方法。

表17：全局环境记录的附加字段
字段名|值|含义
---|---|---
ObjectRecord|Object Environment Record |绑定对象是全局对象。它包含全局内置绑定以及相关领域的全局代码中的FunctionDeclaration、GeneratorDeclaration、AsyncFunctionDeclaration、AsyncGeneratorDeclaration和VariableDeclaration绑定。
GlobalThisValue|Object|返回全局作用域的this值，可以使任意的ECMAScript对象值
DeclarativeRecord|Declarative Environment Record|包含关联域代码的全局代码中的所有声明的绑定，但FunctionDeclaration、GeneratorDeclaration、AsyncFunctionDeclaration、AsyncGeneratorDeclaration和VariableDeclaration绑定除外。
VarNames|List of String|关联域的全局代码中由FunctionDeclaration、GeneratorDeclaration、AsyncFunctionDeclaration、AsyncGeneratorDeclaration和VariableDeclaration声明绑定的字符串名称。

##### 8.1.1.5 模块环境记录（Module Environment Records）

模块环境记录是一个声明性的环境记录，用于表示ECMAScript模块的外部范围。除了常规的可变和不可变绑定之外，模块环境记录还提供不可变导入绑定，这些绑定提供对另一个环境记录中存在的目标绑定的间接访问。

表19：模块式环境记录中额外的方法

Method|目的
---|---
CreateImportBinding(N, M, N2)|在模块环境记录中创建不可变的间接绑定。字符串值N是绑定名的文本。M是模块记录，N2是存在于M的模块环境记录中的绑定。

#### 8.1.2 词法环境中的操作

本规范中使用以下抽象操作对词法环境进行操作:

##### 8.1.2.1 GetIdentifierReference ( lex, name, strict )

1. 如果词法环境lex为null，则
  a. 返回引用类型的值，其基值组件为undefined，其引用的名称组件为name，其严格引用标志为strict。
2. 定义envRec为lex的环境记录
3. 定义exists = envRec.HasBinding(name)
4. 如果exists为true, 则

## 8.3 执行上下文

执行上下文式一种规范策略，用于通过ECMAScript实现来跟踪运行中的代码，