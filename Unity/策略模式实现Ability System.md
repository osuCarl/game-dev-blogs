# 策略模式实现Ability System

想象游戏中玩家具有多种能力（技能），比如“治疗”、“传送”、“火球术”等等。我们可以使用枚举类型来表示这些能力，并通过Switch语句来执行不同的能力行为。代码如下：
```csharp
public class AbilityRunner : MonoBehaviour
{
    enum Ability
    {
        Heal,
        Teleport,
        FireBall
    }

    private Ability currentAbility = Ability.Heal;

    public void UseAbility()
    {
        switch (currentAbility)
        {
            case Ability.Heal:
                print("Heal");
                break;
            case Ability.Teleport:
                print("teleport");
                break;
            case Ability.FireBall:
                print("FireBall");
                break;
        }
    }
}
```
上面的代码存在什么问题？通过Switch语句来实现不同的能力行为，代码的可扩展性和可维护性较差。如果我们想要添加新的能力，比如“冰冻”，我们需要修改`Ability`枚举并在`UseAbility`方法中添加新的case分支，违反了开闭原则。能力的实现细节被硬编码在`AbilityRunner`类中，违反了单一职责原则。

为了解决这个问题，我们可以使用策略模式，将每个能力的实现封装在独立的类中，并通过接口来定义能力的行为。这样，我们可以轻松地添加新的能力，而不需要修改现有的代码。

```csharp
public interface IAbility
{
    void Use(GameObject gameObject);
}

public class AbilityRunner : MonoBehaviour
{
    IAbility currentAbility = new AbilityFireBall();

    public void UseAbility()
    {
        currentAbility.Use(gameObject);
    }
}

public class AbilityFireBall :MonoBehaviour, IAbility
{
    public void Use(GameObject gameObject)
    {
        Debug.Log("using FireBall");
    }
}

public class AbilityTeleport :ScriptableObject, IAbility
{
    public void Use(GameObject gameObject)
    {
        Debug.Log("Teleport");
    }
}

public class AbilityHeal : IAbility
{
    public void Use(GameObject gameObject)
    {
        Debug.Log("Heal");
    }
}
```

通过这种方式，我们将每个能力的实现封装在独立的类中，并通过`IAbility`接口来定义能力的行为。`AbilityRunner`类只负责调用当前能力的`Use`方法，而不需要关心具体的实现细节。事实上，实现能力的类可以是`MonoBehaviour`、`ScriptableObject`，可以和场景中的其他对象进行交互，也可以是Asset里的数据和预制体，从而打开了一系列的可能性，比如可配置的能力，使用Particle System等其他Unity组件等等。

