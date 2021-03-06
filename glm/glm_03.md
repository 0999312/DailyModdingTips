# AddLootTableModifier——用额外的战利品表作为GLM的参数  
本章节基于Minecraft Forge 1.18.2 40.0.44。**低版本内容可能会有所变化。**  
*****  
## `AddLootTableModifier`的起源与发展
&emsp;&emsp;这个类是先由 Commoble 编写，后被 vectorwing 用于 Farmers’ Delight 的奖励箱修改。具体的出处目前暂不得知，有可能是 Commoble 直接为 vectorwing 编写了这个类也说不定。目前能确定的首次使用是Farmer's Delight。  
&emsp;&emsp;因为代码简短清晰、可拓展性强、易于使用等优点集于一身，它是GLM的优秀实例之一。 本章节提供的代码则是由作者一定优化过后的版本。
*****  
## `AddLootTableModifier`代码详解
完整代码[^完整代码]如下。
```java
import com.google.gson.JsonObject;
import net.minecraft.resources.ResourceLocation;
import net.minecraft.util.GsonHelper;
import net.minecraft.world.item.ItemStack;
import net.minecraft.world.level.storage.loot.LootContext;
import net.minecraft.world.level.storage.loot.LootTable;
import net.minecraft.world.level.storage.loot.predicates.LootItemCondition;
import net.minecraftforge.common.loot.GlobalLootModifierSerializer;
import net.minecraftforge.common.loot.LootModifier;
import javax.annotation.Nonnull;
import java.util.List;

public class AddLootTableModifier extends LootModifier {
    private final ResourceLocation lootTable;

    public AddLootTableModifier(LootItemCondition[] conditionsIn, ResourceLocation lootTable) {
        super(conditionsIn);
        this.lootTable = lootTable;
    }

    public boolean canApplyModifier() {
        return true;
    }

    @Nonnull
    @Override
    protected List<ItemStack> doApply(List<ItemStack> generatedLoot, LootContext context) {
        if(this.canApplyModifier()) {
            LootTable extraTable = context.getLootTable(this.lootTable);
            extraTable.getRandomItemsRaw(context, LootTable.createStackSplitter(generatedLoot::add));
        }
        return generatedLoot;
    }

    public static class Serializer extends GlobalLootModifierSerializer<AddLootTableModifier> {
        @Override
        public AddLootTableModifier read(ResourceLocation location, JsonObject object, LootItemCondition[] conditions) {
            ResourceLocation lootTable = new ResourceLocation(GsonHelper.getAsString(object, "lootTable"));
            return new AddLootTableModifier(conditions, lootTable);
        }

        @Override
        public JsonObject write(AddLootTableModifier instance) {
            JsonObject object = this.makeConditions(instance.conditions);
            object.addProperty("lootTable", instance.lootTable.toString());
            return object;
        }
    }
}
```  
&emsp;&emsp;首先它有一个额外的战利品表`ID`作为成员变量，用于我们读取具体的战利品表。在`doApply`方法中进行读取并将其添加进最终的返回值当中。具体内容根据该战利品表决定，所以我们只需要获取这个战利品表就可以了。恰好`doApply`方法有一个`LootContext`参数，可以利用它直接获取战利品表。战利品表实例有一个`getRandomItemsRaw`方法，参数是`LootContext`和`Consumer<ItemStack>`。前者可以直接返回`doApply`所用的参数，后者则是一个`Consumer`。  
&emsp;&emsp;这个`Consumer<ItemStack>`则是关键所在：我们这里不直接使用`generatedLoot::add`，而是使用`LootTable.createStackSplitter`来正确设置`ItemStack`的物品数量并添加`ItemStack`。如果直接使用`add`的话，有可能会导致战利品箱子里没有正确添加应有数量的物品。关于`Consumer`的知识请学习函数式编程，这里简单提一下：它接收泛型作为参数并返回一个`void`方法。  
&emsp;&emsp;有人会注意到`canApplyModifier`目前毫无作用——它看似只能返回`true`。事实上这个方法写在这里是为了实现**利用配置文件而确定是否启用这个修改器**的功能而抽象出来的，实际应用中它需要以匿名内部类的形式被覆写或直接改为调用配置文件中的指定布尔值。  
&emsp;&emsp;序列化器以静态成员类形式写在主类当中。同时我们对`read/write`进行了一些简单的覆写。`read`方法的写法非常简单：从`json`中获取`lootTable`对应的字符串来读取出指定战利品表`ID`，然后直接创建新实例。`conditions`已经为你自动读取完成了，所以不需要额外写代码读取。`write`方法则是如此先`make`出来一个带战利品表条件的`JsonObject`之后再添加一个`lootTable`代表我们需要的战利品表`ID`写入到`JSON`里，返回这个`JsonObject`。  
&emsp;&emsp;即便无法看懂以上内容**也没有关系**。这个代码是可以直接复制并使用的。具体使用方法见概述与第二章内容。这个类也适用于需要数据包生成器的场合，因为我们的序列化器允许我们这样做。

[^完整代码]: 不包括package语句
