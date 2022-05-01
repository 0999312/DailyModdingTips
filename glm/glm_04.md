# 使用数据生成器快速生成GLM   
***
## 什么是数据生成器？  
&emsp;&emsp;数据生成器，英文名为 Data Generator，是 Minecraft 原版就有的用于生成数据包的工具。作为模组开发者，我们也经常会利用这个利器帮助自己处理好成千上万的资源与数据文件。Forge 不仅支持用数据生成器生成原版就支持的各种模型，战利品表和合成表等，而且支持语言文件以及我们今天所讲的GLM。  
&emsp;&emsp;目前有相当多的教程已经告诉你数据生成器如何使用了，在此不再进行说明。我们将直接讲解如何写一个`GlobalLootModifierProvider`来进行数据生成。  
***
## `GlobalLootModifierProvider`类
&emsp;&emsp;数据生成器需要一个“供应商”——`Provider`，所以我们来写一个GLM的“供应商”。  
```java
public class GLMProvider extends GlobalLootModifierProvider {
    public GLMProvider(DataGenerator gen, String modid) {
        super(gen, modid);
    }

    @Override
    protected void start() {
    }
}
```
&emsp;&emsp;这就是一个最简单的“供应商”了。构造方法的参数无需解释，让我们快速进入到`start`方法中。  
&emsp;&emsp;要想添加一个新的需要生成数据的修改器，只需要在`start`方法中调用`add`方法就可以了。Forge 提供的示例代码如下。  
```java
this.add("example_modifier", EXAMPLE_MODIFIER_SERIALIZER, new ExampleModifier(
  new LootItemCondition[] {
    WeatherCheck.weather().setRaining(true).build() // 雨天生效
  },
  "val1", // "prop1=val1"
  10, // "prop2=10"
  Items.DIRT // "prop3=minecraft:dirt"
));
```
&emsp;&emsp;从上面我们可以看出`add`方法总共有三个参数：第一个字符串参数代表最终生成结果的文件名，因为最后生成出来的是一个`JSON`文件所以需要指定文件名；第二个则是该修改器序列化器的实例，如果你注册规范的话直接获取实例就可以使用了；第三个则是一个该修改器的实例，直接创建新实例就可以了，但我们有必要来对其中最重要的参数——条件数组进行讲解。  
***
## 战利品表条件数组  
&emsp;&emsp;在概述中我们说过条件是按照有序加载的，具体的代码实现就是遍历数组。我们重点讲一下数组的元素怎么安排。
> &emsp;&emsp;如果你要表示**同时满足所有条件**完成与连接，直接添加一个数组元素就可以了。通常来说你的修改器继承的都是`LootModifier`类，所以他们一定是并列关系。  
> &emsp;&emsp;如果你要表示**满足任一条件**完成或连接，则在你的元素中调用`or`方法嵌套一个条件，生成器会自动转换。  
> &emsp;&emsp;条件可以组合使用**与或连接**。  
> &emsp;&emsp;如果你的情况比较特殊，则根据你的实际情况进行处理。  
***

## 原版战利品表条件列表
&emsp;&emsp;本表只含有原版和纯Forge 条件下所有的战利品表条件。  
&emsp;&emsp;未完待续，正在考虑使用更好的排版方式。  
```
AlternativeLootItemCondition：就是我们所经常使用的“或”连接，通常不会直接调用它而是使用condition.or()  
InvertedLootItemCondition：对条件取反。使用时调用InvertedLootItemCondition.invert(condition)  
LootItemRandomChanceCondition：取随机数，满足随机数时条件为真。使用时调用LootItemRandomChanceCondition.randomChance(float)  
LootItemRandomChanceWithLootingCondition：带抢夺附魔加成时取随机数，常用于怪物掉落物的处理，使用时调用LootItemRandomChanceWithLootingCondition.randomChanceAndLootingBoost(float, float)  
LootTableIdCondition：取特定战利品表，若当前修改的战利品表ID为所取值时为真。只能在使用GLM时候调用。  
LootTableIdCondition使用时调用LootTableIdCondition.builder(ID)  
CanToolPerformAction：判断手中工具是否可以使用某个特定的ToolAction。使用时调用CanToolPerformAction.canToolPerformAction(ToolAction)  
```
