# 使用Decorator和Composite实现更复杂的Ability System

假如说我想要实现一个更复杂的能力系统，比如说我想要实现一个“火球术”能力，这个能力可以有不同的变种，比如说“延迟火球”、“爆炸火球”等等。每个变种都有自己独特的效果和行为。一种方法是使用继承来实现这些变种，但是这样会导致类的数量迅速增加，代码变得难以维护和扩展。更好的做法是使用装饰器模式。装饰器模式可以认为是定义一个包装类，这个包装类实现了和被包装类相同的接口，并且持有一个被包装类的引用。通过这种方式，我们可以动态地给对象添加新的行为，而不需要修改原有的代码。

```csharp
public interface IAbility
{
    void Use(GameObject gameObject);
}

public abstract class BaseAbilitySO : ScriptableObject, IAbility
{ 
    public virtual void Use(GameObject gameObject){}
}

public class AbilityRunner : MonoBehaviour
{
    IAbility currentAbility = new AbilityFireBall();

    public void UseAbility()
    {
        currentAbility.Use(gameObject);
    }
}

public class AbilityDelayDecorator : BaseAbilitySO
{
    [SerializeField]
    BaseAbilitySO wrappedAbility;

    [SerializeField]
    float delay = 2f;

    public override void Use(GameObject gameObject)
    {
        // example delay implementation, CoroutineRunner is a singleton MonoBehaviour that can run coroutines
        CoroutineRunner.Instance.StartCoroutine(DelayAbility(gameObject));
    }

    private IEnumerator DelayAbility(GameObject gameObject)
    {
        yield return new WaitForSeconds(delay);
        wrappedAbility.Use(gameObject);
    }
}

public class AbilityFireBall : BaseAbilitySO
{
    public override void Use(GameObject gameObject)
    {
        Debug.Log("using FireBall");
    }
}
```
上面的代码中，`AbilityDelayDecorator`是一个装饰器类，它持有一个被包装的能力对象，并在调用`Use`方法时添加了延迟效果。通过指定被包装对象为火球术，我们可以创建一个延迟火球术的能力，而不需要修改火球术的实现。

如果想要实现更复杂的复合能力，比如先同时释放三个火球，第一个火球立即释放，第二个火球延迟2秒释放，第三个火球施加一个持续伤害效果，我们可以使用组合模式。组合模式可以将多个对象组合成树形结构，暴露的接口和单个对象的接口一致。这样，我们可以像使用单个对象一样使用组合对象。

```csharp
public class AbilityComposite : BaseAbilitySO
{
    [SerializeField]
    List<BaseAbilitySO> abilities;

    public override void Use(GameObject gameObject)
    {
        foreach (var ability in abilities)
        {
            ability.Use(gameObject);
        }
    }
}
```
上面的代码中，`AbilityComposite`类持有一个能力列表，并在调用`Use`方法时依次调用每个能力的`Use`方法。通过这种方式，我们可以轻松地组合多个能力，创建出更复杂的能力行为。