# Finite State Machine 有限状态机

一个系统只有有限个状态，任一时刻只能处于一个状态，不同状态之间可以转换。可以使用有限状态机来抽象这个系统。LocoMotion就是一个FSM的例子。推荐使用有限状态机的场景有：实现AI行为，游戏状态管理等等。实现FSM的核心就是定义一系列状态，然后提供外部接口获取输入，根据当前所处的状态和不同的输入，切换到不同的状态。

一种实现方式如下
```csharp
public class LocoMotionFSM : MonoBehaviour
{
    public enum State
    {
        OnGround,
        InAir,
        Crouching
    }

    State currentState = State.OnGround;

    public void Jump(){
        switch (currentState)
        {
            case State.OnGround:
                // 执行跳跃逻辑
                currentState = State.InAir;
                break;
            case State.Crouching:
                // 执行站起来逻辑
                currentState = State.OnGround;
                break;
        }
    }

    public void Fall()
    {
        switch (currentState)
        {
            case State.OnGround:
                // 执行下落逻辑
                currentState = State.InAir;
                break;
            case State.Crouching:
                // 执行下落逻辑
                currentState = State.InAir;
                break;
        }
    }

    public void Land()
    {
        switch (currentState)
        {
            case State.InAir:
                // 执行落地逻辑
                currentState = State.OnGround;
                break;
        }
    }

    public void Crouch()
    {
        switch (currentState)
        {
            case State.OnGround:
                // 执行下蹲逻辑
                currentState = State.Crouching;
                break;
            case State.Crouching:
                // 执行站起来逻辑
                currentState = State.OnGround;
                break;
        }
    }

}
```
可以看到，上述的实现方式主要关注于状态的转换逻辑。关注的是对于某一个输入，当前系统应该做出的反应。这种 Transition-Centric 的方法将所有逻辑都塞在一个输入接口中，代码可读性和维护都比较差。那么有没有办法Flip一下，变成State-Centric 的方法呢，使用State Pattern:

```csharp
public interface ILocomotionContext
{
    // this interface is introduced to avoid circular dependency
    void SetState(ILocomotionState newState);
}

public interface ILocomotionState
{
    void Jump(ILocomotionContext context);
    void Fall(ILocomotionContext context);
    void Land(ILocomotionContext context);
    void Crouch(ILocomotionContext context);
}

public class LocoMotionFSM : MonoBehaviour, ILocomotionContext
{
    private ILocomotionState currentState = new OnGroundState();

    public void Jump()
    {
        currentState.Jump(this);
    }

    public void Fall()
    {
        currentState.Fall(this);
    }

    public void Land()
    {
        currentState.Land(this);
    }

    public void Crouch()
    {
        currentState.Crouch(this);
    }

    private void SetState(ILocomotionState newState)
    {
        currentState = newState;
    }
}

public class OnGroundState : ILocomotionState
{
    public void Jump(ILocomotionContext context)
    {
        // 执行跳跃逻辑
        context.SetState(new InAirState());
    }

    public void Fall(ILocomotionContext context)
    {
        // 执行下落逻辑
        context.SetState(new InAirState());
    }

    public void Land(ILocomotionContext context)
    {
        // Already on ground, do nothing
    }

    public void Crouch(ILocomotionContext context)
    {
        // 执行下蹲逻辑
        context.SetState(new CrouchingState());
    }
}

public class InAirState : ILocomotionState
{
    public void Jump(ILocomotionContext context)
    {
        // Already in air, do nothing
    }

    public void Fall(ILocomotionContext context)
    {
        // Already in air, do nothing
    }

    public void Land(ILocomotionContext context)
    {
        // 执行落地逻辑
        context.SetState(new OnGroundState());
    }

    public void Crouch(ILocomotionContext context)
    {
        // Can't crouch in air, do nothing
    }
}

public class CrouchingState : ILocomotionState
{
    public void Jump(ILocomotionContext context)
    {
        // 执行站起来逻辑
        context.SetState(new OnGroundState());
    }

    public void Fall(ILocomotionContext context)
    {
        // 执行下落逻辑
        context.SetState(new InAirState());
    }

    public void Land(ILocomotionContext context)
    {
        // Already crouching on ground, do nothing
    }

    public void Crouch(ILocomotionContext context)
    {
        // 执行站起来逻辑
        context.SetState(new OnGroundState());
    }
}
```

这种方式将每个状态的行为封装在各自的类中，实际开发时可以将不同类和接口存储在不同文件中，使得代码更易读、更易维护。每个状态类只关注自己的行为和状态转换逻辑，避免了大量的switch-case语句，提高了代码的可扩展性。
当然，可以看到上面的实现方式也存在一定的问题，比如每次状态切换的时候都创建一个新的状态对象，可能会带来一定的性能开销。可以通过使用对象池来优化这个问题。
