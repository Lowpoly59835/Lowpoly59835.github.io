[원본](http://www.davejsaunders.com/2017/05/06/memory-leak-lambdas.html)
포스팅을 번역했음
의역이나 오역이 있을 수 있음

How to leak memory with lambda expressions
Originally posted in May 2017

Lambda expressions are a great feature of C#, but it’s also quite easy to accidentally ‘leak’ memory when using them.

람다 표현식은 C#의 훌륭한 기능이지만, 이것을 사용할 때에 메모리 릭이 또한 우연히 발생한다.

When a lambda expression references a local variable, that expression (and the local variables it references) end up on the heap. If you use multiple lambda expressions within the same scope (a method, for example), all of them (and the multiple variables they reference) will have the same lifetime.

람다 표현식이 로컬 변수를 참조할 때, 표현식은 힙에 할당한다. 만약 당신이 같은 스코프에서 여러 람다 표현식을 사용한다면, 그 모든 것(여러 변수를 참조하는)은 같은 라이프 타임을 가지게 된다.

In other words; that lambda expression that references the huge array, but is only used in the scope of the method, won’t be disposed when the method returns like you expect.. it will hang around until the last lambda created in that method is collected.

다른 말로, 람다 표현식이 거대한 배열을 참조하지만, 이것은 함수의 스코프안에서만 사용된다. 당신이 예상하는 것처럼 함수가 리턴 될 때에 해제되지 않는다..이것은 함수에서 마지막으로 만들어진 람다가 해제될 때까지 유지될 것이다.

Why your local variable is heap-allocated
왜 당신의 로컬 변수가 힙에 할당되는가.
Lambda expressions don’t actually exist in the CLR, they are compiler magic, so we have to do something with them in order for the runtime to understand them.
람다 표현식은 컴파일러의 마법으로 CLR에서 존재하지 않는다, 그래서 우리는 런타임과 함께 이해해야한다.
If your expression doesn’t refer to any local variables, the compiler can create a static method and rewrite your code to call that instead.
만약 표현식이 어떤 로컬 변수를 참조하지 않는다면, 컴파일러는 전역 함수를 만들고 당신의 코드를 대신 호출한다.
If you reference a local variable though, things are a little more complicated..
만약 로컬 변수를 참조한다면, 조금 복잡해진다.

When you create a lambda expression, it’s perfectly possible that it could outlive the stack-frame. For example; it could be returned from the method or allocated to a member variable somewhere. That’s no good when you use a local variable in that lambda, because that variable might no longer exist when the lambda is used later.

람다 표현식을 생성할 때에, 이것은 스택 프레임보다 더 오래 생존하는게 가능하다. 예를들어, 함수가 리턴되거나 어떤 장소에 멤버 변수를 할당된다고 치자. 그것은 람다안에서 로컬 변수를 사용하기 좋지 않다, 왜냐면 변수는 나중에 람다를 사용할 때보다 더 오래 존재하지 않을 수도 있기때문이다.


Because of that, the compiler has to keep it all on the heap. It does this by creating a new class with a method containing the contents of your lambda expression. The variables referenced in your expression become member variables in this new class.

그렇기 때문에, 첨파일러는 힙에 모든 것을 유지시킨다. 이것은 당신의 람다 표현식의 내용을 포함하는 함수와 함께 새로운 클래스를 만든다. 변수는 새로운 클래스에서 멤버 변수가 되어 당신의 표현식에서 참조된다.

Your original code is then re-written to call this new method, safe in the knowledge that the the variables it uses will be there - kept alive in the new class.

당신의 오리지널 코드는 새로운 함수를 호출하게 재작성하고, 사용하는 변수가 존재한다는 것을 알고 안전하게 새 클래스에 유지한다.

Here’s the catch though - if you use multiple lambda expressions in the same method, they are created as methods in the same comiler generated class. That means that every variable used in those expressions will stay alive until that class can be disposed.
여기서 캐치해야할 것 - 만약 당신이 같은 함수에서 여러 람다 표현식을 사용한다면, 같은 클래스에서 다른 함수가 만들어진다.
Let’s look at an example..
예제를 보자.

Leaky lambda expressions in practice

public class MyClass
{
    public Func<int, int> Add1()
    {
        var bigThing = new byte[1024 * 1024 * 1024];
        Func<int> bigThingCount = () => bigThing.Length;
        Console.WriteLine(bigThingCount());

        var smallThing = 1;
        return x => x + smallThing;
    }
}

Here we have a (slightly contrived) method with two lambda expressions.
여기 우리는 두 개의 람다 표현식을 포함한 함수를 가지고 있다.

The first of them refers to a large byte array. It calculates the size of this array so we can write it to the console. We’d expect this to be cleaned up after the method returns.
람다 표현식의 첫 번째(bigThingCount)는 큰 바이트 배열을 참조한다. 배열의 크기를 계산하고 콘솔에 출력한다. 우리는 함수를 리턴한 후에 이것은 청소된다고 예상한다.
The other only closes over an integer, and is totally unrelated to the first. We return this lambda to the caller.
다른 하나는 integer(X)만을 닫고, 첫번쨰와 전혀 관련이 없다. 우리는 호출차에게 람다를 반환한다.
Let’s take a look and see how this second expression keeps the large byte array alive.
어떻게 두 번째 표현식이 가장 큰 바이트 배열을 유지하는지를 살펴보자.

This is what the compiler generates for this class:

public class MyClass
{
    [CompilerGenerated]
    private sealed class <>c__DisplayClass0_0
    {
        public byte[] bigThing;

        public int smallThing;

        internal int <Add1>b__0()
        {
            return this.bigThing.Length;
        }

        internal int <Add1>b__1(int x)
        {
            return x + this.smallThing;
        }
    }

    public Func<int, int> Add1()
    {
        MyClass.<>c__DisplayClass0_0 <>c__DisplayClass0_ = new MyClass.<>c__DisplayClass0_0();

        <>c__DisplayClass0_.bigThing = new byte[1073741824];
        Func<int> func = new Func<int>(<>c__DisplayClass0_.<Add1>b__0);
        Console.WriteLine(func());

        <>c__DisplayClass0_.smallThing = 1;
        return new Func<int, int>(<>c__DisplayClass0_.<Add1>b__1);
    }
}
Firstly, you can see that the compiler has generated a new class for us, named <>c_DisplayClass0_0. We can also see that both local variables (bigThing and smallThing) have been hoisted to become members of this common class.
우선적으로, 당신은 컴파일러가 우리를 위해 <>c_DisplayClass0_0라고 이름붙은 새로운 클래스를 생산하는 것을 볼 수 있다. 또한 우리는 로컬 변수또한 이 클래스의 멤버가 되어있는 것을 볼 수 있다.

The re-written Add1() method sets these properties, rather than creating local variables.
재작성한 Add1() 함수는 로컬변수를 만드는 대신에 이러한 속성들을 설정한다.

It then returns a function that refers to the method in our new class, therefore keeping everything alive until this returned function is collected.
이것은 우리의 새로운 클래스에서 함수를 참조하는 함수를 반환하고, 그것들은 리턴된 함수가 수집될때까지 모든 것이 살아있게 유지한다.

Lambdas are a great language feature, but use them with caution.

람다는 훌륭한 기능이지만, 주의해서 사용해야한다.

----------------------

이 내용에 관해서 간단한 예제와 함께 IL DASM을 통해 확인하는 글을 쓸 예정
어느정도 이 내용을 기반으로 하고있기에 
