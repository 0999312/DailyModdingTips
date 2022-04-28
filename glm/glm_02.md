# 一个没有额外参数的战利品表修改器  
本章节基于Minecraft Forge 1.18.2 40.0.44。**低版本内容可能会有所变化。**  
*****  
## 事先准备：写一个注册表  
&emsp;&emsp;我们快速来新建一个类：GLMRegistry 用于存放我们的GLM序列化器注册表，用于注册我们新增的修改器类型。  
```java
public class GLMRegistry {
    // 别忘了替换MODID哦
    public static final DeferredRegister<GlobalLootModifierSerializer<?>> GLM =  
    DeferredRegister.create(ForgeRegistries.Keys.LOOT_MODIFIER_SERIALIZERS, MODID);
}
```  
&emsp;&emsp;DeferredRegister是Forge最推荐使用的注册工具，用于注册绝大部分注册表。我们先不去讨论它的细节，只需要知道本章节需要这样一个东西来快速编写一个注册表。  
&emsp;&emsp;别忘了在Mod主类的构造方法里调用以下一行代码来启用注册表：  
```java
GLMRegistry.GLM.register(FMLJavaModLoadingContext.get().getModEventBus());
```  
***
## 第一个战利品表修改器——继承LootModifier类  
&emsp;&emsp;注册表完成之后，让我们新建一个类并继承LootModifier类。这将会是我们的第一个战利品表修改器。  
```java
public class ExampleLootModifier extends LootModifier {
    public ExampleLootModifier(LootItemCondition[] conditionsIn) {
        super(conditionsIn);
    }
    @Override
    protected List<ItemStack> doApply(List<ItemStack> generatedLoot, LootContext context) {
        return generatedLoot;
    }
}
```
&emsp;&emsp;修改器生效的重点在doApply方法上。返回的List就是最后该战利品表能得到的物品列表。我们来快速往这个List里加一个苹果，只需要在List里add一个苹果就可以了。  
```java
    @Override
    protected List<ItemStack> doApply(List<ItemStack> generatedLoot, LootContext context) {
        generatedLoot.add(new ItemStack(Items.APPLE));
        return generatedLoot;
    }
```
&emsp;&emsp;非常简单，但是现在还不能直接使用。我们已经说过GLM是数据驱动的，也就是说它需要序列化器。别忘了我们刚刚的注册表就是为了注册序列化器而准备的，所以现在来写序列化器的类。  
新建一个类并继承GlobalLootModifierSerializer。泛型里面填写的是你需要序列化的修改器的类名，代码如下。
```java
public class ExampleLootModifierSerializer extends GlobalLootModifierSerializer<ExampleLootModifier> {
    @Override
    public ExampleLootModifier read(ResourceLocation location, JsonObject object,
            LootItemCondition[] ailootcondition) {
        return new ExampleLootModifier(ailootcondition);
    }

    @Override
    public ExampleLootModifier write(ExampleLootModifier instance) {
        return new JsonObject();
    }
}
```
&emsp;&emsp;当然，你写成内部类也可以，我们这里暂时写成外部类的样式。对于我们的第一个序列化器而言这就已经是完成品了——因为我们没有什么特别的操作，不需要额外的write也不需要在read上花上什么功夫。  
&emsp;&emsp;read方法在读取一个JSON时候调用，而write方法则是把修改器实例写入到JSON时候才会调用。write方法通常也只会在使用数据生成器[^数据生成器]的时候才会起作用，因为我们暂时不需要所以直接如此返回，但不要返回null。
&emsp;&emsp;之后在注册表类里进行注册就可以使用了。序列化器的使用方法参见概述。

[^数据生成器]: 即Data Generator，详见第四章
