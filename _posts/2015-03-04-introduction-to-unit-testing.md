---
layout: post
title:  "Introduction to Unit Testing"
date:   2015-03-04 09:00:00 -0600
categories: csharp testing
---

Let's be honest, manually testing software sucks. When developing a new feature, you'll often have to manually test your changes multiple times to ensure everything is working. If you are a web developer, this usually means jumping through hoops in order to test your code through the UI. What you really want is a way to automate these tests and run them whenever you make a change. Even better would be a way to run these tests everytime *someone else* makes a change to the software as well. This is where unit testing and automated testing in general comes in.

#### What is a Unit Test? 

According to Roy Osherove, a unit test is "an automated piece of code that invokes a *unit of work* in the system and then checks a single assumption about the behavior of that unit of work." 

That's a good definition, but there is one other important criteria for what is a unit test:  It has full control over all components that are being tested. So if your code is hitting a third-party webservice, it's not a unit test but an integration test. We'll cover integration tests in a later post.

Now would be a good time to explain exactly what a unit of work is. I like to think of it as the collection of all code in the system that must be executed in order to accomplish a single task. This could be a single method or it could be multiple methods spanning multiple different classes. A good example would be placing an order in an e-commerce application. Consider the code below:

{% highlight csharp %}
public class OrderService : IOrderService
{
	private IOrderRepository _OrderRepository;
	private IShippingService _ShippingService;
	
	public OrderService(IShippingService shippingService, IOrderRepository repo)
	{
		_OrderRepository = repo;
		_ShippingService = shippingService;
	}
	
	public void PlaceOrder(Order order)
	{
		if (_ShippingService.AddressIsValid(order.Address)) 
		{
			order.Shipping = _ShippingService.CalculateShipping(order.Address);
			order.Status = OrderStatus.Pending;
			_OrderRepository.Add(order);
		}
		else
		{
			throw new OrderException("The shipping address is invalid");
		}
	}
}
{% endhighlight %}

So if we wanted to test placing an order, our unit of work would be the `PlaceOrder` method as well as every other method that the `PlaceOrder` method executes to accomplish this task. This unit of work would then also span the `CalculateShipping` and `Add` methods.

In order to write a unit test for placing an order, we would need to write a test that checks a *single behavior* of that unit of work. So for example, we may want to write a unit test that checks that an order is only placed when the shipping address is valid. Or we might want to write a unit test that checks that an order is always created with a status of `Pending` when it is initially placed. 

It's important to keep in mind that when testing the logic for placing an order, we *only* want to test that specific logic. We don't want to test the specifics of how `IShippingService` validates an address as part of these tests, just that an order is only placed if the address is valid. For testing address validation, we'd want to write an entirely different set of tests that tests only that unit of work.

#### What Is A Good Unit Test?

This is a very subjective question, but I think there are some general guidelines that everyone can agree on. 

*  A good unit test is fast

Ideally you want to run your suite of unit tests every time you make a change to verify that your code is working correctly, and almost as importantly, that you haven't inadvertantly broken some other part of the system with your changes. If your unit tests take forever to run then you're much less likely to run them on a regular basis. There are even some Visual Studio plugins that will run your unit tests continuously as you develop. 

* A good unit test is trustworthy

The last thing you want is for your unit tests to fail when there isn't a problem with your code. When this happens on a regular basis, you'll get into a complacent mindset where you'll start ignoring failing tests due to false positives. This can cause you to ignore tests that are showing genuine breakages in functionality. Plus why would you spend all this time writing tests that don't provide any value?

* A good test is maintainable

Nothing in life is free, writing good tests takes time and effort. As the system evolves, you'll need to update the tests you have written already. Writing tests already intorduces some friction into the development process, if you write unmaintainable tests then this friction will turn into actual pain as time goes on. There are some strategies you can employ to reduce the amount of changes you'll have to make to your tests as the system evolves, I'll cover those in a separate blog post.

* A good unit test has no side effects

A good unit test should run entirely in memory and shouldn't touch things like the database or the filesystem at all. In fact, I would argue that if there are any permanent side effects of running a unit test then it isn't a unit test at all but an integration test. Again, we'll have more on integration tests in a different post.

* A good unit test can be run in isolation

In other words, a good unit test doesn't depend on other unit tests to have been run before it. You can run just that one unit test or you can run all the unit tests together in any order and it shouldn't matter. This also harkens back to the rule about a good unit test having no side effects, running one unit test shouldn't have an effect on another.

* A good unit test is completely automated

A unit test should be able to completely test some aspect of the system without any human intervention. If some part of the system is too difficult to test in some automated fashion, it's likely that this part of the system needs to be refactored in order to improve testability. In other words, it's time to improve your system architecture ;)

#### Why Would You Want To Write Unit Tests?

I would hope that this would be obvious by this point, but there are several benefits to writing automated tests.

* Reduces the amount of time spent manually testing

As you develop new features for the system, you can write a test once to test some aspect of that feature over and over again as you continue working on it. This means you will catch coding mistakes earlier and spend less time tracking down what caused them. 

* Improves the refactorability of your code

As developers, we're often scared to refactor our code. We worry about inadvertantly breaking functionality or introducing new bugs somewhere in the system as a result. A good set of unit tests give developers instant feedback on whether a refactoring broke something and gives us the confidence we need to make continual improvements to existing code. 

* Reduces the chances of introducing a regression

Whenever we make a bug fix to the system, we should write a test to prove that the bug has been eliminated. This test then becomes a permanent addition to the test suite, if any change ever reintroduces that bug then we should be able to catch it instantly. 

* Reduces the amount of time spent analyzing the impact of a change

With enough tests covering enough of the system (usually around 95% of your code base), you can be reasonably assured that your changes haven't broken anything in some other part of the system. I have found that doing a manual impact analysis on a change made to a big system can take dramatically longer than it took to actually make the change in the first place. Anything that can significantly reduce the amount of time spent doing this type of analysis is a huge win.

#### Unit Testing Isn't A Magic Bullet

It can be easy to be lulled into a false sense of security when writing unit tests. It's important to remember that your unit tests only prove that your system is absent of the defects you have written tests for. Just because your system has 98% code coverage with your tests doesn't mean that there aren't glaring problems with the system. 