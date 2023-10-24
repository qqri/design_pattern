**전략 패턴 ( Strategy Pattern )**

객체들이 할 수 있는 행위 각각에 대해 전략 클래스를 생성하고, 유사한 행위들을 캡슐화 하는 인터페이스를 정의하여,

객체의 행위를 동적으로 바꾸고 싶은 경우 직접 행위를 수정하지 않고 전략을 바꿔주기만 함으로써 행위를 유연하게 확장하는 방법을 말합니다.



간단히 말해서 객체가 할 수 있는 행위들 각각을 전략으로 만들어 놓고, 동적으로 행위의 수정이 필요한 경우 전략을 바꾸는 것만으로 행위의 수정이 가능하도록 만든 패턴입니다.







**1. 전략 패턴 사용 이유**

예를 들어, 기차( Train )와 버스( Bus ) 클래스가 있고, 이 두 클래스는 Movable 인터페이스를 구현했다고 가정하겠습니다.

그리고 Train과 Bus 객체를 사용하는 Client도 있습니다.



![img](https://t1.daumcdn.net/cfile/tistory/9916204B5BF8DAD105)

이 구조를 코드로 표현하면 다음과 같습니다.

```
public interface Movable {
    public void move();
}
public class Train implements Movable{
    public void move(){
        System.out.println("선로를 통해 이동");
    }
}
public class Bus implements Movable{
    public void move(){
        System.out.println("도로를 통해 이동");
    }
}
public class Client {
    public static void main(String args[]){
        Movable train = new Train();
        Movable bus = new Bus();

        train.move();
        bus.move();
    }
}
```

기차는 선로를 따라 이동하고, 버스는 도로를 따라 이동합니다.

그러다 시간이 흘러 선로를 따라 움직이는 버스가 개발되었다고 가정해봅시다.



그러면 Bus의 move() 메서드를 다음과 같이 바꿔주기면 하면 끝납니다.

```
public void move(){
    System.out.println("선로를 따라 이동");
}
```



그런데 이렇게 수정하는 방식은 **SOLID의 원칙 중 OCP( Open-Closed Principle )에 위배**됩니다.

OCP에 의하면 기존의 move()를 수정하지 않으면서 행위가 수정되어야 하지만, 지금은 Bus의 move() 메서드를 직접 수정했지요.



또한 지금과 같은 방식의 변경은 시스템이 확장이 되었을 때 유지보수를 어렵게 합니다.

예를 들어, 버스와 같이 도로를 따라 움직이는 택시, 자가용, 고속버스, 오토바이 등이 추가된다고 할 때, 모두 버스와 같이 move() 메서드를 사용합니다.

만약에 새로 개발된 선로를 따라 움직이는 버스와 같이, 선로를 따라 움직이는 택시, 자가용, 고속버스 ... 등이 생긴다면,

택시, 자가용, 고속버스의 move() 메서드를 일일이 수정해야 할 뿐더러, 같은 메서드를 여러 클래스에서 똑같이 정의하고 있으므로 메서드의 중복이 발생하고 있습니다.



즉, 지금과 같은 수정 방식의 문제점은 다음과 같습니다.

- OCP 위배
- 시스템이 커져서 확장이 될 경우 메서드의 중복문제 발생

따라서 이를 해결하고자 전략 패턴을 사용하려고 합니다.









**2. 전략 패턴 구현**

이번에는 위와 같이 선로를 따라 이동하는 버스가 개발된 상황에서 시스템이 유연하게 변경되고 확장될 수 있도록 전략 패턴을 사용해보도록 하겠습니다.



1)

먼저 전략을 생성하는 방법입니다.



현재 운송수단은 선로를 따라 움직이든지, 도로를 따라 움직이든지 두 가지 방식이 있습니다.

즉, 움직이는 두 방식에 대해 Strategy 클래스를 생성하도록 합니다. ( RailLoadStrategy, LoadStrategy )



그리고 두 클래스는 move() 메서드를 구현하여, 어떤 경로로 움직이는지에 대해 구현합니다.



또한 두 전략 클래스를 캡슐화 하기 위해 MovableStrategy 인터페이스를 생성합니다.

이렇게 캡슐화를 하는 이유는 운송수단에 대한 전략 뿐만 아니라,

다른 전략들( 예를 들어, 주유방식에 대한 전략 등)이 추가적으로 확장되는 경우를 고려한 설계입니다.

![img](https://t1.daumcdn.net/cfile/tistory/99804D3E5BF8E1C80C)



이를 코드로 표현하면 다음과 같습니다.

```
public interface MovableStrategy {
    public void move();
}
public class RailLoadStrategy implements MovableStrategy{
    public void move(){
        System.out.println("선로를 통해 이동");
    }
}
public class LoadStrategy implements MovableStrategy{
    public void move() {
        System.out.println("도로를 통해 이동");
    }
}
```





2)

다음으로 운송 수단에 대한 클래스를 정의할 차례입니다.



기차와 버스 같은 운송 수단은 move() 메서드를 통해 움직일 수 있습니다.

그런데 이동 방식을 직접 메서드로 구현하지 않고, 어떻게 움직일 것인지에 대한 전략을 설정하여, 그 전략의 움직임 방식을 사용하여 움직이도록 합니다.



그래서 전략을 설정하는 메서드인 setMovableStrategy()가 존재합니다.

![img](https://t1.daumcdn.net/cfile/tistory/997EE1415BF8E4BD13)



이를 코드로 표현하면 다음과 같습니다.

```
public class Moving {
    private MovableStrategy movableStrategy;

    public void move(){
        movableStrategy.move();
    }

    public void setMovableStrategy(MovableStrategy movableStrategy){
        this.movableStrategy = movableStrategy;
    }
}
public class Bus extends Moving{

}
public class Train extends Moving{

}
```





3)

이제 Train과 Bus 객체를 사용하는 Client를 구현할 차례입니다.



Train과 Bus 객체를 생성한 후에, 각 운송 수단이 어떤 방식으로 움직이는지 설정하기 위해 setMovableStrategy() 메서드를 호출합니다.



그리고 전략 패턴을 사용하면 프로그램 상으로 로직이 변경 되었을 때, 얼마나 유연하게 수정을 할 수 있는지 살펴보기 위해

선로를 따라 움직이는 버스가 개발되었다는 상황을 만들어 버스의 이동 방식 전략을 수정했습니다.

```
public class Client {
    public static void main(String args[]){
        Moving train = new Train();
        Moving bus = new Bus();

        /*
            기존의 기차와 버스의 이동 방식
            1) 기차 - 선로
            2) 버스 - 도로
         */
        train.setMovableStrategy(new RailLoadStrategy());
        bus.setMovableStrategy(new LoadStrategy());

        train.move();
        bus.move();

        /*
            선로를 따라 움직이는 버스가 개발
         */
        bus.setMovableStrategy(new RailLoadStrategy());
        bus.move();
    }
}
```







이상으로 전략 패턴이 무엇인지에 대해 알아보았습니다.

Java에서도 여러 디자인 패턴을 사용하고 있는데 Strategy 패턴을 사용하고 있는 API는 다음과 같으며, 참고하시면 좋을 것 같습니다. ( [참고](https://stackoverflow.com/a/2707195)[링크](https://stackoverflow.com/a/2707195) )



### [Strategy](http://en.wikipedia.org/wiki/Strategy_pattern)

### recognizeable by behavioral methods in an abstract/interface type which invokes a method in an implementation of a *different* abstract/interface type which has been *passed-in* as method argument into the strategy implementation

- [`java.util.Comparator#compare()`](https://docs.oracle.com/javase/8/docs/api/java/util/Comparator.html#compare-T-T-), executed by among others `Collections#sort()`.
- [`javax.servlet.http.HttpServlet`](http://docs.oracle.com/javaee/7/api/javax/servlet/http/HttpServlet.html), the `service()` and all `doXXX()` methods take `HttpServletRequest` and `HttpServletResponse` and the implementor has to process them (and not to get hold of them as instance variables!).
- [`javax.servlet.Filter#doFilter()`](http://docs.oracle.com/javaee/7/api/javax/servlet/Filter.html#doFilter-javax.servlet.ServletRequest-javax.servlet.ServletResponse-javax.servlet.FilterChain-)