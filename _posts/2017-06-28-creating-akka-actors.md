---
title:  "Creating Akka actors"
date:   2017-06-28 08:31:12 +0200
---
#### **Akka**

[Akka][akka-link] is an awesome modular toolkit that enables us to build (event) message driven applications easily without worrying too much about boilerplate code, remoting, clustering etc. One of the patterns used in Akka is the inheritance structure of actors where each parent actor is responsible for the life cycle of its children. The supervisor strategies can be controlled and configured but let’s concentrate on the creation part.

#### **How to create an actor**

One of the standard ways of creating an actor is by using the default Props create method
{% highlight java %}
ActorRef nodeGuardian = system.actorOf(Props.create(NodeGuardianActor.class));
{% endhighlight %}
or with an optional actor name
{% highlight java %}
ActorRef nodeGuardian = system.actorOf(Props.create(NodeGuardianActor.class, "node-guardian"));
{% endhighlight %}

This is the default approach to actor creation and in most cases it’s good enough. But there are cases where you need to pass a parameter to the constructor of the actor class. One of the best ways is to define a static method (usually inside the actor class) that returns the Props object with a defined Creator for that actor class
{% highlight java %}
public static Props propsNodeGuardian(final Configuration configuration) {
    return Props.create(new Creator<NodeGuardianActor>() {
        @Override
        public NodeGuardianActor create() throws Exception {
            return new NodeGuardianActor(configuration);
        }
    });
}
{% endhighlight %}

Having multiple actor implementations that have the same constructor can either be done by repeating the previous example for each concrete class or can be implemented using generics
{% highlight java %}
public static <T extends AbstractActor> Props props(final Class<T> type, final Configuration configuration)
        throws NoSuchMethodException {
    Constructor<?> constructor = type.getConstructor(Configuration.class);
    return Props.create(new Creator<T>() {
        @Override
        public T create() throws Exception {
            return (T) constructor.newInstance(configuration);
        }
    });
}
{% endhighlight %}

With Props a method defined like this you can easily create an actor instance by calling
{% highlight java %}
ActorRef nodeGuardian = system.actorOf(ActorFactory.props(NodeGuardianActor.class, config));
{% endhighlight %}
where `ActorFactory` is a placeholder class for all generic actor Props definitions.

There are some corner cases where you need to create actors by their class name and this can also be done using a Props definition like this
{% highlight java %}
public static Props exampleProps(final String className, final Configuration configuration)
        throws ClassNotFoundException, NoSuchMethodException, InitializationException {
    Constructor<?> constructor = Class.forName(className).getConstructor(String.class, Configuration.class);
    return Props.create(ExampleActor.class, new Creator<ExampleActor>() {
        @Override
        public ExampleActor create() throws Exception {
            return (ExampleActor) constructor.newInstance(className, configuration);
        }
    });
}
{% endhighlight %}

This specific piece of code is for creating an instance of `ExampleActor` implementation where `ExampleActor` is an abstract class extending `AbstractActor`. You can also mix this with generics and passing class as an additional parameter depending on your requirements.

#### **The judgment**

I know that this can cause raised eyebrows with some people and I do agree that doing it this way can be marked as an anti-pattern since all these additional parameters can be sent to the actor via messages after its creation but there are some cases where you need to have the additional information at the point of creation. Also, creating actors using their class names can sound like a messy solution but if you are leveraging the distributed pub-sub mechanism and you really don’t care where the actual actor instance really is, this might be a good solution.

[akka-link]: http://akka.io/
