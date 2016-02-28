Java Generics wildcards has been bothering me for some time, a long time in fact. It is not something that I could not understand or use it to my advantage in my daily work. I just could not explain why some of the Generics errors I encountered. But not this time, I must get to bottom of it, at least, the easy part of it.

##Class Diagram
Apple and Orange inherit from Fruit. Further on, Fruit inherits from Plant i.e. Apple and Orange is-a Fruit is-a Plant

![Apple/Orange is-a Fruit is-a Plant][1]

##Getting It Out of The Way
Let it be clear that `List<Apple>` and `List<Orange>` are not the children of `List<Fruit>`. For example, you simply cannot do this,

    List<Apple> applesBasket = new ArrayList<>(); 
    fill(applesBasket); // compilation error 
    ...
    
    List<Orange> orangesBasket = new ArrayList<>(); 
    fill(orangesBasket); // compilation error 
    ...
    
    // add more fruits to the basket
    public void fill(List<Fruit> basket) { // add fruits to basket }

but it is alright to do this,

    List<Fruit> fruitsBasket = new ArrayList<>();
    fruitsBasket.add(new Apple());  // compiles ok
    fruitsBasket.add(new Orange()); // compiles ok

So, we must change `fill(...)` method signature to 

    public void fill(List<? super Fruit> basket) { // add fruits to basket }
    
to accept both `List<Apple>` and `List<Orange>`.

Why does the parameter `basket` is not a Covariance type parameter `List<? extends Fruit>`?. After all,  `Apple` and `Orange` are extended from `Fruit`. By the way, `List<? super Fruit>` is a Contravariance type parameter.

##PECS Rule And Generics Wildcards

The mistake was trying to add instances of supertype of Fruit to the `List<? super Fruit>` container e.g. add `Plant` instances to `List<? super Fruit>` in the `fill(...)` method. It was *wrong* to think that `super` was referring to the class parent-child relationship. `super` refers to the container behaviour. The `basket` in the method `fill(List<? super Fruit> basket)` ***consumes*** instances of type `Fruit` or subtypes of `Fruit`. So, according to PECS (Producer `extends` and Consumer `super`) rule, if the container is a consumer, use `? super T` otherwise use `? extends T`, the type parameter is `? super Fruit` for this case.

If those  methods used in the container are limited to those which take parameters of that type parameter, the container is a consumer. On the contrary, if the methods used by the container return instances of that type parameter, the container is a producer. Lastly, if both type of methods are used, the container is both producer and consumer. In this case, the wildcard (?) cannot be used. Examples of consumer methods are,

    void public add(T o, int i);
    void public remove(T o);
    void public invoke(T o, int i);

and examples of producer methods are,

    void T get();
    void T removeAt(int i);
    
It is all make sense now once we have determined the `super` and `extends` type parameters and view from the caller angle. The method requires the caller to provide the right type of value to invoke the method as shown in this example,

    List<Plant> plantsBasket = new ArrayList<>();
    List<Fruit> fruitsBasket = new ArrayList<>();
    List<Apple> applesBasket = new ArrayList<>();
    ...
    fill(plantsBasket); // compiles ok
    fill(fruitsBasket); // compiles ok
    fill(applesBasket); // compilation error

It requires thinking from inside out.    

Source code illustrating this article is available [here][2]

Please refer to [Effective Java by Joshua Bloch][3] for more details on PECS and Generics.

Finally, `List<Object>` and `List<?>` are not the same.

  [1]: http://thlim.files.wordpress.com/2013/12/dcff8f91.png
  [2]: https://gist.github.com/sshark/7925397
  [3]: http://uet.vnu.edu.vn/~chauttm/e-books/java/Effective.Java.2nd.Edition.May.2008.3000th.Release.pdf