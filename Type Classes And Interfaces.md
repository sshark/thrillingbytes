What `interface` in Java is what  `trait`  in Scala. Types are not classes. Types are interfaces and they are abstract. As such,

 - A **type** is an abstract interface.
 - A **class** is an implementation of a type or interface.
 - An **instance** or **object** is the state a class holds at any point in time.

Say, I have a **Driveable** (interface / trait)  for anything that can move forth and back. Next, classes **Car** and **Bus** implement **Driveable**. Each has similar and distinct properties but both can move forth and back. Finally, we **C200**, **CX9** and **Tata** which are **Car** and **Bus** instances  that hold the state for each class separately at any given time.

However, this article is not about `trait`, it is about the *Type Class* pattern which uses `trait` and `implicit` to achieve its objective.

##Type Class Pattern
Type class is a pattern, an alternative to Strategy and Adapter patterns. It has its root in Haskell. Scala uses  `implicit` and `trait` as shown [here][scala-example] to implement type class pattern.

    case class Alcohol(liters:Double)
    case class Water(liters:Double)
    
    case class Fire(heat:Double)
    trait Flammable[A] {
      def burn(fuel:A): Fire
    }
    
    implicit object AlcoholIsFlammable extends Flammable[Alcohol] {
      def burn(fuel:Alcohol) = Fire(120.0)
    }
    
    def setFire[T](fuel:T)(implicit f:Flammable[T]) = f.burn(fuel)
    
    setFire(Alcohol(1.0)) // ok
    setFire(Water(1.0)) // FAIL

Using `import` creatively I could have have different `Flammable` without recompiling `setFire`.

There is a Java version which I mimicked the Scala version,

    // AlcoholIsFlammable.java
    public class AlcoholIsFlammable implements Flammable<Alcohol> {
	    public Fire burn(Alcohol fuel) {
		    return new Fire(120.0);
	    }
    }

    // Alcohol.java
    public class Alcohol {
	    public double liters;

	    public Alcohol(double liters) {
		    this.liters = liters;
	    }	
    }

	// Fire.java 
    public class Fire {
	    public double heat;

	    public Fire(double heat) {
		    this.heat = heat;
	    }	
    }

	/* Flammable.java */
    public interface Flammable<A> {
	    Fire burn(A fuel);
    }

    /** uncomment this to make water flammable :)
    // WaterIsFlammable.java
    public class WaterIsFlammable implements Flammable<Water> {
	    public Fire burn(Water fuel) {
		    return new Fire(60.0);
	    }
    }

	// Water.java
    public class Water {
	    public double liters;

	    public Water(double liters) {
		    this.liters = liters;
	    }	
    }
    */

	// RunMe.java
    public class RunMe {
	    public static void main(String[] args) {	
		    System.out.println(setFire(new Alcohol(1.0), 
		        new AlcoholIsFlammable()).heat);
		    // compilation will fail if the next line is uncommented
		    // System.out.println(setFire(new Water(1.0), 
		    //    new AlcoholIsFlammable()).heat);  		
     	}

     	public static <T> Fire setFire(T fuel, Flammable<T> f) {
     		return f.burn(fuel);
     	}
    }

Without `implicit` it make no sense in Java because all types and classes must be present during compilation unless dynamic class injection like *Guice* or *Spring IoC* is used. Therefore, *IoC* makes less sense in Scala.

## Summary
This article is a summary from a few random resources including [Wikipedia][wiki]. Out of them, I find this [paragraph][stackoverflow],

> A crucial distinction between *type classes* and *interfaces* is that for class A to be a "member" of an interface it must declare so at the site of its own definition. By contrast, any type can be added to a type class at any time, provided you can provide the required definitions, and so the members of a type class at any given time are dependent on the current scope. Therefore we don't care if the creator of A anticipated the type class we want it to belong to; if not we can simply create our own definition showing that it does indeed belong, and then use it accordingly. So this not only provides a better solution than adapters, in some sense it obviates the whole problem adapters were meant to address.

 
and [this][debasishg],

> Type classes define a set of contracts that the adaptee type needs to implement. Many people misinterpret type classes synonymously with interfaces in Java or other programming languages. With interfaces the focus is on subtype polymorphism, with type classes the focus changes to parametric polymorphism. You implement the contracts that the type class publishes across unrelated types.

stands out because it is clear and simple to understand

[wiki]: https://en.wikipedia.org/wiki/Class_%28computer_programming%29#Class_vs._type
[stackoverflow]: http://stackoverflow.com/a/5409272/185514
[scala-example]: http://www.azavea.com/blogs/labs/2011/06/scalas-numeric-type-class-pt-1/
[debasishg]: http://debasishg.blogspot.com/2010/06/scala-implicits-type-classes-here-i.html