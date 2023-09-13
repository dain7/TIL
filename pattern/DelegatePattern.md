# Delegate Pattern 이란?
- 객체가 자신의 기능을 다른 객체에게 위임하여 기능을 실행하는 패턴
- 객체 간의 결합도를 낮춰서 유지보수성 및 확장성 Up!

## Example
A가 B의 기능을 사용하고 싶을 때, A가 B의 기능을 직접 구현하지 않고, C에게 위임하여 구현하는 것! \
이때 C를 Delegate 객체, 위임자 객체라고 한다. 

예를 들어 A가 동물의 울음소리를 출력하고 싶다고 하자. B는 다양한 울음소리가 존재한다. \
A는 B에 직접적으로 접근하지 않고, 울음소리를 출력하는 기능을 C에게 위임한다. \
C는 B에서 적절한 울음소리를 구현하여 사용할 수 있다. \
이처럼 A와 B의 <b>결합도</b>를 낮추면서 다양한 울음소리를 출력할 수 있다.

```java

public enum AnimalKind {
    CAT, DOG
}

public interface Animal {
    void makeSound();
    AnimalKind getKind();
    boolean isSameKind(AnimalKind kind);
}

// B 클래스 
public class Cat implements Animal {
    @Override
    public void makeSound() {
        System.out.println("야옹");
    }
    
    @Override
    public AnimalKind getKind() {
    	return AnimalKind.CAT;
    }
    
    @Override
    public boolean isSameKind(AnimalKind kind) {
    	return AnimalKind.CAT.equals(kind);
    }
}

// B 클래스 
public class Dog implements Animal {
    @Override
    public void makeSound() {
        System.out.println("멍멍");
    }

    @Override
    public AnimalKind getKind() {
    	return AnimalKind.DOG;
    }
    
    @Override
    public boolean isSameKind(AnimalKind kind) {
    	return AnimalKind.DOG.equals(kind);
    }
}

// C 클래스
// B 클래스의 역할을 위임받음.
public class AnimalDelegator {
	private final List<Animal> animals;
    
    public AnimalDelegator() {
    	animals = new ArrayList<>();
    	animals.add(new Cat());
        animals.add(new Dog());
    }

    public void makeSound(AnimalKind kind) {
        animals.stream()
                .filter(animal -> animal.isSameKind(kind))
                .forEach(Animal::makeSound);
    }
}

// A 클래스
// B 클래스를 직접 구현할 필요없이 C 클래스를 호출 
public class Client {
    public static void main(String[] args) {
        AnimalDelegator delegator = new AnimalDelegator();
        delegator.makeSound(AnimalKind.CAT); // "야옹" 출력
        delegator.makeSound(AnimalKind.DOG); // "멍멍" 출력
    }
}
```