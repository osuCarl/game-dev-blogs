# Object Pooling 对象池
想象一个常见的游戏场景，玩家控制一把枪，每次按下空格键的时候，都会向前方发射一枚子弹。
一个简单的实现方式如下：
```csharp
public class Gun : MonoBehaviour
{
    [SerializeField] Bullet bulletPrefab;

    void Update()
    {
        if (Input.GetKeyDown(KeyCode.Space))
        {
            Instantiate(bulletPrefab, transform.position, Quaternion.identity);
        }
    }
}

public class Bullet : MonoBehaviour
{
    [SerializeField] Vector3 speed;

    void Update()
    {
        transform.position += speed * Time.deltaTime;
    }

    void OnBecameInvisible()
    {
        Destroy(gameObject);
    }
}
```
这种实现方式的问题在于，每次按下空格键都会创建一个新的子弹对象，而每个子弹对象在离开屏幕后都会被销毁。如果玩家频繁按下空格键，就会导致频繁的创建和销毁对象，进而引发垃圾回收，影响游戏性能。为了解决这个问题，我们可以使用对象池技术。对象池是一种预先创建一定数量的对象，并在需要时重复使用这些对象的技术。这样可以减少频繁的创建和销毁对象，提升游戏性能。幸运的是，Unity从2021.1版本开始引入了<a href="https://docs.unity3d.com/2021.3/Documentation/ScriptReference/Pool.ObjectPool_1-ctor.html">内置的对象池API</a>，简化了对象池的实现。
下面是使用Unity内置对象池API实现的子弹发射系统：
```csharp
public class Gun : MonoBehaviour
{
    [SerializeField] Bullet bulletPrefab;
    IObjectPool<Bullet> bulletPool;

    void Awake()
    {
        bulletPool = new ObjectPool<Bullet>(
            CreateBullet,
            OnBulletGet,
            OnBulletRelease,
            OnBulletDestroy,
            maxSize:3
            );
    }
    
    Bullet CreateBullet()
    {
        Bullet bullet = Instantiate(bulletPrefab, transform.position, Quaternion.identity);
        bullet.SetPool(bulletPool);
        return bullet;
    }
    
    void OnBulletGet(Bullet bullet)
    {
        bullet.gameObject.SetActive(true);
        bullet.transform.position = transform.position;
    }
    
    void OnBulletRelease(Bullet bullet)
    {
        bullet.gameObject.SetActive(false);
    }
    
    void OnBulletDestroy(Bullet bullet)
    {
        Destroy(bullet.gameObject);
    }

    void Update()
    {
        if (Input.GetKeyDown(KeyCode.Space))
        {
            bulletPool.Get();
        }
    }
}

public class Bullet : MonoBehaviour
{
    [SerializeField] Vector3 speed;
    IObjectPool<Bullet> bulletPool;

    public void SetPool(IObjectPool<Bullet> pool)
    {
        bulletPool = pool;
    }

    void Update()
    {
        transform.position += speed * Time.deltaTime;
    }

    void OnBecameInvisible()
    {
        bulletPool.Release(this);
    }
}
```
在创建对象池时，需要指定各个回调方法。CreateBullet 方法用于在池为空时创建新Bullet实例。OnBulletGet 方法在从池中取出实例时调用。OnBulletRelease 方法在将实例归还给池时调用。OnBulletDestroy 方法在池中的实例超过最大数量时调用，用于销毁多余的实例。maxSize参数指定了池中允许存在的最大实例数量。创建之后，通过调用bulletPool.Get()方法获取一个Bullet实例，通过调用bulletPool.Release(bullet)方法将Bullet实例归还给池。这样就实现了子弹对象的复用，避免了频繁的创建和销毁对象，提高了游戏性能。
需要注意的是，maxSize并不是可以获得对象的最大数量。事实上，ObjectPool并没有限制能够获得对象的数量。如果池中没有可用对象，会调用CreateFunc创建一个新对象返回。只有当Release的对象数量超过maxSize时，才会调用actionOnDestroy销毁多余的对象。