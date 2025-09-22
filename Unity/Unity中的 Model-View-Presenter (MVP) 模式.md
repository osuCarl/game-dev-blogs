# Unity中的 Model-View-Presenter (MVP) 模式

首先介绍传统的MVC模式。MVC将传统的应用程序（如 web application）分成三个部分，Model, View, Controller。Model负责具体的业务逻辑。View负责根据Model的数据渲染UI，并呈现给用户。Controller负责处理用户输入，并更新Model和View。在Unity中，渲染和获取用户输入往往由Unity引擎处理，所以MVC模式并不完全适用。MVP模式是MVC的一个变种，Model的职责和MVC一样，View现在既负责渲染UI，也负责获取用户输入。Presenter负责协调Model和View，通过View获取用户输入，并更新Model和View。这样的逻辑更符合Unity的工作方式。

下面是一个简单的例子。假设我们有一个游戏角色，有生命值（Health），每秒减少一定的生命值，并且在升级时重置生命值。
```csharp
public class Health : MonoBehaviour {
    [SerializeField] float fullHealth = 100f;
    [SerializeField] float drainPerWait = 2f;
    [SerializeField] float waitSeconds = 1f;
    [SerializeField] Image healthBar;

    WaitForSeconds wait;
    float currentHealth = 0;

    private void Awake() {
        currentHealth = fullHealth;
        wait = new WaitForSeconds(waitSeconds);
        StartCoroutine(HealthDrain());
    }

    public float GetHealth()
    {
        return currentHealth;
    }

    public float GetFullHealth()
    {
        return fullHealth;
    }

    private IEnumerator HealthDrain()
    {
        while (currentHealth > 0)
        {
            currentHealth -= drainPerWait;
            UpdateUI();
            yield return wait;
        }
    }

    private void UpdateUI()
    {
        healthBar.fillAmount = currentHealth / fullHealth;
    }
}
```

目前，Health类同时处理游戏逻辑和UI更新，违反了单一职责原则。我们可以通过引入MVP模式来重构代码。

```csharp
public class Health : MonoBehaviour {
    [SerializeField] float fullHealth = 100f;
    [SerializeField] float drainPerWait = 2f;
    [SerializeField] float waitSeconds = 1f;

    WaitForSeconds wait;
    float currentHealth = 0;
    public event Action OnHealthChanged;

    private void Awake() {
        currentHealth = fullHealth;
        wait = new WaitForSeconds(waitSeconds);
        StartCoroutine(HealthDrain());
    }

    public float GetHealth()
    {
        return currentHealth;
    }

    public float GetFullHealth()
    {
        return fullHealth;
    }

    private IEnumerator HealthDrain()
    {
        while (currentHealth > 0)
        {
            currentHealth -= drainPerWait;
            OnHealthChanged?.Invoke();
            yield return wait;
        }
    }
}

public class HealthPresenter : MonoBehaviour
{
    [SerializeField] Health health;
    [SerializeField] Image healthBar;

    // a naive way is to update every frame
    // void Update()
    // {
    //     UpdateUI();
    // }

    // a better way is to use observer pattern
    void OnEnable()
    {
        health.OnHealthChanged += UpdateUI;
    }

    void OnDisable()
    {
        health.OnHealthChanged -= UpdateUI;

    }

    void Start()
    {
        UpdateUI();
    }

    private void UpdateUI()
    {
        healthBar.fillAmount = health.GetHealth() / health.GetFullHealth();
    }
}
```
在这个重构后的代码中，Health类只负责管理生命值的逻辑，而HealthPresenter类负责更新UI。Health类通过事件OnHealthChanged通知Presenter，当生命值变化时，Presenter会更新UI。这种分离使得代码更清晰，更易于维护和测试。