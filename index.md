---
title: Eduardo or not?
---
# Starting with Spring

This resource is intended for people with no previous experience with Spring, and possibly little programming experience too. If you've been exposed to Spring before or have worked with other app development frameworks you're probably better off looking at other awesome resources, or even at Spring's own documentation.
If you've just finished learning how to program in Java, you're starting your software development career, or have just been dumped into a huge project sprinkled with lots of Spring magic that makes no sense, this is meant for you.    

You will become the next Spring Guru in your team!
## What you'll need

- Maven 3
- Java +8
- Git
- I recommend an IDE like IntelliJ (there's a free version) or alternatively a command line prompt

## Why Spring?

The best way I can think of advocating for Spring is not by demonstrating what it does but rather explaining what problems it solves.
Understanding why it'd be incredibly hard not to use it in a real world complex Java application is key. If right now you absolutely hate Spring for how complicated it is and how much harder your project is to understand because of it, the goal is to make you realise how it actually simplifies it and to feel grateful for it. Let's get to it.

## Dependency Injection

Imagine you want to build your own Amazon like company. For this you'll probably need a website that can accept customer orders. Since the website could be one of many customer entry points
(think about a mobile app) the actual handling can be delegated to a separate, central order processing unit. Such a unit needs to check the inventory for stock, charge the customer for the
price of the item and dispatch the item. In coding terms it might look like this.

{% github_sample_ref edu05/learnspring/blob/master/src/main/java/edu/learn/spring/dependencyinjection/vanilla/CustomerOrder.java %}
{% highlight java %}
{% github_sample edu05/learnspring/blob/master/src/main/java/edu/learn/spring/dependencyinjection/vanilla/CustomerOrder.java %}
{% endhighlight %}
{% github_sample_ref edu05/learnspring/blob/master/src/main/java/edu/learn/spring/dependencyinjection/vanilla/AmazonWebsite.java %}
{% highlight java %}
{% github_sample edu05/learnspring/blob/master/src/main/java/edu/learn/spring/dependencyinjection/vanilla/AmazonWebsite.java %}
{% endhighlight %}  
{% github_sample_ref edu05/learnspring/blob/master/src/main/java/edu/learn/spring/dependencyinjection/vanilla/OrderProcessor.java %}
{% highlight java %}
{% github_sample edu05/learnspring/blob/master/src/main/java/edu/learn/spring/dependencyinjection/vanilla/OrderProcessor.java %}
{% endhighlight %}
{% github_sample_ref edu05/learnspring/blob/master/src/main/java/edu/learn/spring/dependencyinjection/vanilla/CustomerWallet.java %}
{% highlight java %}
{% github_sample edu05/learnspring/blob/master/src/main/java/edu/learn/spring/dependencyinjection/vanilla/CustomerWallet.java %}
{% endhighlight %}
{% github_sample_ref edu05/learnspring/blob/master/src/main/java/edu/learn/spring/dependencyinjection/vanilla/Warehouse.java %}
{% highlight java %}
{% github_sample edu05/learnspring/blob/master/src/main/java/edu/learn/spring/dependencyinjection/vanilla/Warehouse.java %}
{% endhighlight %}  
{% github_sample_ref edu05/learnspring/blob/master/src/main/java/edu/learn/spring/dependencyinjection/vanilla/Application.java %}
{% highlight java %}
{% github_sample edu05/learnspring/blob/master/src/main/java/edu/learn/spring/dependencyinjection/vanilla/Application.java %}
{% endhighlight %}  

Note I have not fully implemented all of the methods, instead I have focused more on the structure of the code. 
All of the code is accessible via the [git repo](https://github.com/edu05/learnspring) so you don't have to waste your time copy pasting if you want to try this out for yourself. Here's where Maven, Java 8+, Git and the IDE come into play.

The problem with this approach is you have to dedicate a number of lines to construct all of the object instances your application needs, and to make sure they're declared in the correct order for
everything to compile. This quickly becomes a boring task as the number of object instances grows in your project. One of Spring's goals is to aid developers to focus in writing business specific
code, while reducing the amount of clutter. The Spring equivalent of this example only changes in its `Application` class as shown.

{% github_sample_ref edu05/learnspring/blob/master/src/main/java/edu/learn/spring/dependencyinjection/sprinkles/Application.java %}
{% highlight java %}
{% github_sample edu05/learnspring/blob/master/src/main/java/edu/learn/spring/dependencyinjection/sprinkles/Application.java %}
{% endhighlight %}  


Now, you might argue the Spring version takes up even more lines of code, and rightly so! But that's only because I'm going to use this example to explain Spring's core concept. If you understand this section
you'll have dominated the most important aspect of Spring, and everything will be downhill after that.

> A Spring application is formed of a Spring context. A Spring context is just a collection of Spring beans. A Spring bean is a long living* instance of an object that also has a name Spring can
> identify it by. Spring can only "sprinkle its magic" over object instances that are within its context.

And that's it!

Notice the `@SpringBootApplication` annotation on the `Application` class, this lets Java know this is a Spring* application. Notice also the first line of the `main()` method. It constructs the
Spring context by scanning the code for any methods that are annotated with `@Bean` and inserting those instances into it. It then returns the created context back to the caller.
To prove that a Spring context is a collection of named beans I grabbed the `AmazonWebsite` from the context by its name and submitted an order.

You're probably wondering how Spring could have created all the object instances in the correct order and wired them together, all outside of the `main()` method. I would resist the urge to understand
the inner workings of how this could be achieved unless you know about Java's Reflection - in which case...there you go...Spring does it via Java Reflection! Instead, suffice to say the
`SpringApplication.run()` call requests Spring to scan the code looking for special annotations (for now just `@Bean`), building a graph of object dependencies and subsequently creating them in the correct
order. Dependencies are indicated in the method signature of `@Bean` annotated methods. `AmazonWebsite` depends on an `OrderProcessor`, an `OrderProcessor` depends on a list of `Warehouse`s and a `CustomerWallet`, but
`Warehouse`s don't depend on anything for their construction, nor do `CustomerWallet`s, so Spring will create those first. Note Spring does not require you to declare these in any particular order, what's important
is that all dependencies are declared in their method signature. Another nice trick Spring does is aggregate all beans of type `Warehouse` before sending them for the construction of the `OrderProcessor`.

Now that you've got your head round this gimmick of object creation you're probably still regretting how surrounding object instantiations with an annotated method only increases the number
of lines in your code, and not the opposite. Spring simplifies the creation of single instance classes (we call these singletons) by offering us to simply annotate them with `@Component` at the class level, completely avoiding
the extra method. Have a look at [the improved Spring version for this section](https://github.com/edu05/learnspring/tree/master/src/main/java/edu/learn/spring/dependencyinjection/sprinkles_improved).

`AmazonWebsite`, `CustomerWallet`, and `OrderProcessor` are annotated with `@Component`, letting Spring know it needs to create a single instance for said classes. With this strategy the dependencies are declared
to Spring by the constructor arguments, and the name of the bean will be the camel cased name of the class. Note `CustomerOrder` is not annotated, as it simply a short lived object intended only to pass
data around. The previous classes don't hold any data, instead they describe business rules, making them appropriate to exist as a singleton. The `Warehouse` also
doesn't hold any data and it also only holds business logic, but we have 2 `Warehouse`s, not one, and for that reason it is not eligible for `@Component` annotation.

If you're still berating over the larger number of lines in the more concise Spring version think about how each project would fare against a growing number of single instance objects. The non-Spring
version would keep growing needing to manually create each new object instance whereas the concise Spring version would only require an `@Component` at the top of each new class declaration. While they might
take the same total amount of lines (after all `@Component` lines also count), I think we can agree the Spring version is simpler and allows us to focus on writing business specific code.

The problem we have solved in this section is also referred to as dependency injection or wiring.

## Containerization

If you run both flavours of the previous program you might have noticed the Spring variation did not just run and finish, but kept on going. This is tantamount for the majority of production grade
apps - after all, you don't want your website going down after taking its first order! When I was learning Java my programs were always expected to finish. Programs were viewed as a mechanism to obtain an answer to a question, and
a program running indefinitely was usually regarded as a bad thing. When building enterprise applications it's quite the opposite, you want to build a resilient program that can withstand the harshest
exceptions and still keep on running!

Programs that are long running in Spring are placed inside a container. A container is just a piece of software that wraps your application	and provides it with an execution environment that is
responsible for supplying capabilities such as security, logging or transaction management*.

In this section the Spring version remains unaltered but it's worth explaining where the container comes from at this point. The maven dependency `spring-boot-starter-web` indicates Spring* ours is going
to be a web application and so needs to run indefinitely. Spring places our program inside a container (Apache Tomcat by default) and then runs that. If you were to comment that dependency out the Spring
program would no longer run indefinitely. Try it!

To achieve something similar in plain Java you'd have to change the `Application` class to something like this.

{% github_sample_ref edu05/learnspring/blob/master/src/main/java/edu/learn/spring/containerization/vanilla/Application.java %}
{% highlight java %}
{% github_sample edu05/learnspring/blob/master/src/main/java/edu/learn/spring/containerization/vanilla/Application.java %}
{% endhighlight %}

It doesn't actually give us all the functionality a full container does, but it creates a `HttpServer` and starts it,
allowing the vanilla program to also run indefinitely. This is the first time both versions are neck and neck on number of lines, and it's only going to get better for Spring.

## Boilerplate

So far we've been ignoring the fact that the `AmazonWebsite` is not really a website (thanks for waiting patiently). It is unable to receive http requests, create java objects from the payload of the incoming
request (we call this process deserializing and the reverse process of converting a java object into a string representation that can be sent over the wire serializing), and to respond with
appropriate http status codes. As always lets have a look how we might achieve all of this with simple Java.

I moved the `HttpServer` creation to `AmazonWebsite`, after all dealing with http requests is something very specific to a website, and I don't like it when logic leaks from where it truly belongs. I had to
add a url to my server and a specification as to what should happen to the http exchange when one is initiated (the handler). The handler consists of the deserialization of the request body into a
`CustomerOrder`, which gets sent through to the `OrderProcessor` and then a response is sent back to the user acknowledging the order has been successfully submitted.

{% github_sample_ref edu05/learnspring/blob/master/src/main/java/edu/learn/spring/boilerplate/vanilla/Application.java %}
{% highlight java %}
{% github_sample edu05/learnspring/blob/master/src/main/java/edu/learn/spring/boilerplate/vanilla/Application.java %}
{% endhighlight %}

{% github_sample_ref edu05/learnspring/blob/master/src/main/java/edu/learn/spring/boilerplate/vanilla/AmazonWebsite.java %}
{% highlight java %}
{% github_sample edu05/learnspring/blob/master/src/main/java/edu/learn/spring/boilerplate/vanilla/AmazonWebsite.java %}
{% endhighlight %}

When running the program you can execute this in the command line to exercise your fully fledged website.

> `curl -X POST http://localhost:8080/accept-order -H 'content-type: application/json' -d '{"itemId":3,"customerId":1}'`

Notice all but one of the imported classes belongs to the JDK, and that's the `ObjectMapper`. The `ObjectMapper` has a very limited use, it's in charge of converting the request body into our `CustomerOrder`. That means there's no magic at all and
the code looks crystal clear right? Yay!

Maybe, but it's dull and boring. Why? Because there's much too much code for so little functionality. All that's special about this `AmazonWebsite` is its url (`accept-order`), what type of payload it
needs (`CustomerOrder`), what to with it (invoke the `acceptOrder()` method) and what to return. The `HttpServer` creation, http method check, payload deserialization, and output generation is generic code - you'd have to repeat those lines
for every url on your web application. If you're trying to beat Amazon, who have different urls for new orders, dispathed orders, past orders, recommended items, etc. those lines will quickly build up. Bear all of this
in mind while taking a look at the Spring version.

{% github_sample_ref edu05/learnspring/blob/master/src/main/java/edu/learn/spring/boilerplate/sprinkles/AmazonWebsite.java %}
{% highlight java %}
{% github_sample edu05/learnspring/blob/master/src/main/java/edu/learn/spring/boilerplate/sprinkles/AmazonWebsite.java %}
{% endhighlight %}

{% github_sample_ref edu05/learnspring/blob/master/src/main/java/edu/learn/spring/boilerplate/sprinkles/Application.java %}
{% highlight java %}
{% github_sample edu05/learnspring/blob/master/src/main/java/edu/learn/spring/boilerplate/sprinkles/Application.java %}
{% endhighlight %}
 

Notice how short `AmazonWebsite` is. The questions in your brain should match the issues in the plain Java equivalent I mentioned earlier. `@RestController` declares this is a class that'll receive http requests,
`@PostMapping("/accept-order")` declares the url and allowed http method, and `@RequestBody` indicates the type of payload to expect and to deserialize. You'll have guessed the method's body specifies what to do when a request is
received. Finally, the `Application` class needs the `@EnableWebMvc` annotation.

Hopefully you find this approach more concise than the pure Java one.

To illustrate how Spring's version can get away with being so succinct and how there's actually very little magic going on I have extracted the boilerplate of the vanilla's `AmazonWebsite` class into an
abstract `Wesbite` class that could be reused by any other future websites.

{% github_sample_ref edu05/learnspring/blob/master/src/main/java/edu/learn/spring/boilerplate/vanilla_improved/Website.java %}
{% highlight java %}
{% github_sample edu05/learnspring/blob/master/src/main/java/edu/learn/spring/boilerplate/vanilla_improved/Website.java %}
{% endhighlight %}

{% github_sample_ref edu05/learnspring/blob/master/src/main/java/edu/learn/spring/boilerplate/vanilla_improved/AmazonWebsite.java %}
{% highlight java %}
{% github_sample edu05/learnspring/blob/master/src/main/java/edu/learn/spring/boilerplate/vanilla_improved/AmazonWebsite.java %}
{% endhighlight %}



This time round, `AmazonWebsite` declares only the essence of what constitutes a class capable of receiving http requests. The url, method and the payload type go into the constructor and account for two of
the annotations in Spring's version. Extending the `Website` class is comparable to annotating with `@RestController`. Effectively, the code inside the `Website` class is all the boilerplate we save ourselves from writing by using Spring.

But Spring gives us so much more though. With Spring it'd be really simple to change the serialization schema from JSON to XML, to secure the website behind a username and a password, to map exceptions
to different http status codes, and more. Are you willing to give this a go using only plain Java?

This is where Spring really shines. It's able to elevate Java from a plain programming language to a tool-kit full of production grade capabilities.
Decorating the code with annotations may not look natural at first, but it's a small price to pay for all the free functionality we get. And, if you sit down to think about it, you can probably imagine
how an annotation could be mapped to some plain Java code.

Thank you for reading, let me know what you think :)

---
*It would have been more correct to say a Spring bean is an object instance with a lifecycle. While the lifecycle of a bean normally consists of being created at the beginning of the app and
shutdown at the end of the app, sometimes a bean might be shutdown mid app.  
*More precisely a Spring Boot application. Spring Boot is a relatively recent sub-project of Spring to further simplify how we work with Spring.  
*Not a direct quote but inspired by the definition in Server Component Patterns: Component Infrastructures Illustrated with EJB (Wiley Software Patterns Series)  
*Spring Boot  