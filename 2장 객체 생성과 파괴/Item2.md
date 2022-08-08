생성자와 정적 팩터리 메서드는 똑같은 제약이 하나 있다. 바로 **선택적 매개변수가 많을 때 적절히 대응하기 어렵다**는 점이다.

### 점층적 생성자 패턴 - 확장하기 어렵다!!

```
public class NutritionFacts {
    private final int servingSize; //필수
    private final int servings; //필수
    private final int calories; //선택
    private final int fat; //선택
    private final int sodium; //선택
    private final int carbohydrate; //선택

    public NutritionFacts(int servingSize, int servings) {
        this(servingSize, servings, 0);
    }

    public NutritionFacts(int servingSize, int servings, int calories) {
        this(servingSize, servings, calories, 0);
    }

    public NutritionFacts(int servingSize, int servings, int calories, int fat) {
        this(servingSize, servings, calories,  fat, 0);
    }

    public NutritionFacts(int servingSize, int servings, int calories, int fat, int sodium) {
        this(servingSize, servings, calories,  fat,  sodium, 0);
    }

    public NutritionFacts(int servingSize, int servings, int calories, int fat, int sodium, int carbohydrate) {
        this.servingSize = servingSize;
        this.servings = servings;
        this.calories = calories;
        this.fat = fat;
        this.sodium = sodium;
        this.carbohydrate = carbohydrate;
    }
}
```

```
public class Main {
    public static void main(String[] args) {
        NutritionFacts cocaCola = new NutritionFacts(240, 8, 100, 0, 35, 27);
    }
}
```

위의 코드는 이펙티브 자바 책 내에 있는 예제 코드다. 위와 같은 방식을 점증적 생성자 패턴이라고 하는데

필수 매개변수를 받는 생성자 1개, 그리고 선택매개변수를 하나씩 늘려가면서 생성자를 만드는 패턴이다.

**위와 같은 방식의 단점은 다음과 같다.**

1\. 사용자가 설정하길 원치 않는 매개변수까지 포함하기 쉬운데, 어쩔 수 없이 그런 매개변수에도 값을 지정해줘야 한다.

ex) 위 코드에서는 지방(fat)에 0을 넘겼다. 지금같은 경우는 매개변수가 6개로 적어서 나빠보이진 않겠지만, 수가 늘어난다면 상상도 하기 싫다..

2\. 매개변수가 많아지면 코드를 작성하거나 읽기가 어렵다.

코드를 읽을 때 각 값의 의미가 무엇인지 헷갈릴 것이고, 매개변수가 몇 개 인지도 주의해서 세어 보아야 한다. 타입이 같은 매개변수가 연달아 늘어서 있으면 찾기 어려운 버그로 이어질 수도 있다. 클라이언트가 실수로 매개변수의 순서를 바꿔서 건네줘도 컴파일러는 알아채지 못하고, 결국 런타임에서 엉뚱한 동작을 하게 된다.

---

#### **자바빈즈 패턴**

매개변수가 없는 생성자로 객체를 만든 후, setter 메서드를 호출해 원하는 매개변수 값을 설정하는 방식

```
public class NutritionFacts {
    private int servingSize = -1;
    private int servings = -1;
    private int calories = 0;
    private int fat = 0;
    private int sodium = 0;
    private int carbohydrate = 0;

    public NutritionFacts() {}

    public void setServingSize(int servingSize) {
        this.servingSize = servingSize;
    }

    public void setServings(int servings) {
        this.servings = servings;
    }

    public void setCalories(int calories) {
        this.calories = calories;
    }

    public void setFat(int fat) {
        this.fat = fat;
    }

    public void setSodium(int sodium) {
        this.sodium = sodium;
    }

    public void setCarbohydrate(int carbohydrate) {
        this.carbohydrate = carbohydrate;
    }
}
```

이전 점층적 생성자 패턴의 단점들이 자바빈즈 패턴에서는 더 이상 보이지 않는다. 물론, 코드가 길어지긴 했지만 인스턴스를 만들기가 쉽고, 그 결과 더 읽기 쉬운 코드가 되었다.

```
public class Main {
    public static void main(String[] args) {
        NutritionFacts cocaCola = new NutritionFacts();
        cocaCola.setServingSize(240);
        cocaCola.setServings(8);
        cocaCola.setCalories(100);
        cocaCola.setFat(0);
        cocaCola.setSodium(35);
        cocaCola.setCarbohydrate(27);
    }
}
```

**하지만 이 역시 자신만의 심각한 단점이 있다.**

1\. 객체 하나를 만들려면 메서드를 여러개 호출해야 하고, 객체가 완전히 생성되기 전까지는 일관성이 무너진 상태에 놓인다. 점층적 생성자 패턴에서는 매개변수들이 유효한지를 생성자에서만 확인하면 일관성을 유지할 수 있는데 그 장치가 완전히 사라졌다.

\=> 이처럼 **일관성이 무너지는 문제 때문에 자바빈즈 패턴에서는 클래스를 불변으로 만들 수 없다.** 스레드 안전성을 얻으려면 프로그래머가 추가 작업을 해줘야 한다.(freeze)

freeze는 생성이 끝난 객체를 수동으로 얼리고, 얼리기 전에는 사용할 수 없도록 하는데 다루기가 어려워서 실전에서는 사용하지는 않는다. ( 점층적 생성자 패턴의 안전성과 자바빈즈 패턴의 가독성을 겸비한 빌더 패턴의 등장)

#### **빌더 패턴**

Builder를 이용해 필수 매개변수로 객체를 생성하고, 일종의 setter를 사용하여 선택 매개변수를 초기화 한 뒤

build() 메서드를 호출하여 완전한 객체를 생성하는  패턴

클라이언트는 필요한 객체를 직접 만드는 대신, 필수 매개변수만으로 생성자(혹은 정적 팩터리)를 호출해 빌더 객체를 얻는다. 그런 다음 빌더 객체가 제공하는 일종의 setter 메서드들로 원하는 선택 매개변수들을 설정하고, 마지막으로 매개변수가 없는 build 메서드를 호출해 (보통은 불변인) 객체를 얻는다.

빌더는 생성할 클래스 안에 정적(static) 멤버 클래스로 만들어 두는 게 보통이다.

```
public class NutritionFacts {
    private final int servingSize;
    private final int servings;
    private final int calories;
    private final int fat;
    private final int sodium;
    private final int carbohydrate;

    private NutritionFacts(Builder builder) {
        servingSize = builder.servingSize;
        servings = builder.servings;
        calories = builder.calories;
        fat = builder.fat;
        sodium = builder.sodium;
        carbohydrate = builder.carbohydrate;
    }

    public static class Builder {
        // 필수 매개변수
        private final int servingSize;
        private final int servings;

        // 선택 매개변수
        private int calories = 0;
        private int fat = 0;
        private int sodium = 0;
        private int carbohydrate = 0;

        // 필수 매개변수만을 담은 Builder 생성자
        public Builder(int servingSize, int servings) {
            this.servingSize = servingSize;
            this.servings = servings;
        }

        // 선택 매개변수의 setter, Builder 자신을 반환해 연쇄적으로 호출 가능
        public Builder calories(int val) {
            calories = val;
            return this;
        }

        public Builder fat(int val) {
            fat = val;
            return this;
        }

        public Builder sodium(int val) {
            sodium = val;
            return this;
        }

        public Builder carbohydrate(int val) {
            carbohydrate = val;
            return this;
        }

        // build() 호출로 최종 불변 객체를 얻는다.
        public NutritionFacts build() {
            return new NutritionFacts(this);
        }
    }
}
```

위의 예시 코드에서 NutritionFacts 클래스는 불변이며, 모든 매개변수의 기본값들을 한곳에 모아뒀다. 빌더의 Setter 메서드들은 빌더 자신을 반환하기 때문에 연쇄적인 호출이 가능하다 (method chaining)

다음 아래 코드는 이 클래스를 사용하는 클라이언트의 코드이다.

```
NutritionFacts cocaCola = new NutriFacts.Builder(240, 8)
      .calories(100).sodium(35).carbohydrate(30).build();
```

이 클라이언트 코드는 쓰기도 쉽지만, 무엇보다도 읽기가 쉽다.

> 빌더 패턴과 자바 빈즈 패턴의 가장 큰 차이점은 불변성이다.  
> 자바 빈즈 패턴은 객체를 생성 하고 나서 값을 setter 메서드를 통해 넣는다. 그렇기 때문에 실수 또는 악의적인   
> 목적으로 setter 메서드를 통해 유효하지 않은 null 값 혹은 정확하지 않은 값이 들어 갈 수 있다.  
> 반면, 빌더 패턴은 객체 생성 전, 값을 setter 메서드를 통해 넣고, 다 넣었다면 마지막에 build 메서드를 호출해서  
> 객체를 생성한다.  
> 때문에 사용 중에 값이 변경될 우려도 없고, 그로 인해 불변성과 안전성이 올라 간다.  
> 빌더 패턴 사용시에는 public setter 메서드를 선언해서는 안된다.

예시코드 2-4 계층적으로 설계된 클래스와 잘 어울리는 빌더 패턴

```
public abstract class Pizza{
   public enum Topping { HAM, MUSHROOM, ONION, PEPPER, SAUSAGE }
   final Set<Topping> toppings;

   // 추상 클래스는 추상 Builder를 가진다. 서브 클래스에서 이를 구체 Builder로 구현한다.
   abstract static class Builder<T extends Builder<T>> {
      EnumSet<Topping> toppings = EnumSet.noneOf(Topping.class);
      public T addTopping(Topping topping) {
         toppings.add(Objects.requireNonNull(topping));
         return self();
      }

      abstract Pizza build();

      // 하위 클래스는 이 메서드를 overriding하여 this를 반환하도록 해야 한다.
      protected abstract T self();
   }

   Pizza(Builder<?> builder) {
      toppings = builder.toppings.clone();
   }
}

public class NyPizza extends Pizza {
   public enum Size { SMALL, MEDIUM, LARGE }
   private final Size size;

   public static class Builder extends Pizza.Builder<Builder> {
      private final Size size;

      public Builder(Size size) {
         this.size = Objects.requireNonNull(size);
      }

      @Override public NyPizza build() {
         return new NyPizza(this);
      }

      @Override protected Builder self() { return this; }
   }

   private NyPizza(Builder builder) {
      super(builder);
      size = builder.size;
   }
}

public class Calzone extends Pizza {
   private final boolean sauceInside;

   public static class Builder extends Pizza.Builder<Builder> {
      private boolean sauceInside = false;

      public Builder sauceInside() {
         sauceInside = true;
         return this;
      }

      @Override public Calzone build() {
         return new Calzone(this);
      }

      @Override protected Builder self() { return this; }
   }

   private Calzone(Builder builder) {
      super(builder);
      sauceInside = builder.sauceInside;
   }
}
```

Pizza.Builder 클래스는 재귀적 타입 한정을 이용하는 제네릭 타입이다. 여기에 추상 메서드는 self를 더해 하위 클래스에서는 형변환하지 않고도 메서드 연쇄를 지원할 수 있다. self 타입이 없는 자바를 위한 이 우회 방법을 시뮬레이트한 셀프 타입 (simulated self-type) 관용구라고 한다.

Pizza의 하위 클래스가 2개가 있다. 하나는 일반적인 뉴욕 피자, 다른 하나는 칼초네 피자이다. 뉴욕 피자는 크기(size)

매개변수를 필수로 받고, 칼초네 피자는 소스를 안에 넣을지 선택(sauceInside) 하는 매개변수를 필수로 받는다.

각 하위 클래스의 빌더가 정의한 build 메서드는 해당하는 구체 하위 클래스를 반환하도록 선언한다.

NyPizza.Builder 는 NyPizza를 반환하고, Calzone.Builder는 Calzone을 반환한다.

하위 클래스 메서드가 상위 클래스의 메서드가 정의한 반환 타입(즉, Pizza) 가 아니라, 그 하위 타입을

반환하는 기능을 공변 반환 타이핑(covariant return typing)이라고 한다.

이를 이용하면 클라이언트가 형변환에 신경 쓰지 않고 빌더를 사용 할 수 있다.

```
public class Main {
    public static void main(String[] args) {
        NYPizza pizza = new NYPizza.Builder(SMALL)
                .addTopping(SAUSAGE)
                .addTopping(ONION)
                .build();

        Calzone calzone = new Calzone.Builder()
                .addTopping(HAM)
                .sauceInside()
                .build();
    }
}
```

위 코드는 계층적 빌더를 사용한 클라이언트의 코드다.  생성자로는 누릴 수 없는 사소한 이점이 있다.

빌더를 이용하면 가변인수 매개변수를 여러개 사용할 수 있다. 각각을 적절한 메서드로 나눠 선언하면 되는데 또는

메서드를 여러 번 호출하도록 하고 각 호출 때 넘겨진 매개변수들을 하나의 필드로 모을 수 있다.

(위의 코드에서 addTopping 메서드가 그렇다.)

#### **빌더 패턴의 단점**

객체를 만들려면, 그에 앞서 빌더부터 만들어야 한다. 빌더 생성 비용이 크지는 않은데 성능에 민감한 상황에서는

문제가 될 수 있다.

점증척 생성자 패턴보다는 코드가 장황해서 매개변수가 4개 이상은 되야 값어치를 한다.

정리

생성자나 정적 팩터리가 처리해야 할 매개변수가 많다면 빌더 패턴을 선택하자!!

빌더는 점층적 생성자보다 클라이언트 코드를 읽고 쓰기가 간결하고, 자바빈즈보다 훨씬 안전하다.

p.s 모든 코드는 이펙티브 자바 책을 가져왔습니다.
