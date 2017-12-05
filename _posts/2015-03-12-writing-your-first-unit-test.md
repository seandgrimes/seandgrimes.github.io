---
layout: post
title:  "Writing Your First Unit Test"
date:   2015-03-12 09:00:00 -0600
categories: csharp testing
---

For the rest of the articles in this series, we'll be using the excellent 
[xUnit.net](http://xunit.github.io) unit testing framework to write tests. This is the unit testing 
framework with the greatest amount of mindshare at the moment, with Microsoft having recently adopted 
it for the various ASP.NET projects that they have open sourced. 

Before we begin, we'll need to cover some simple concepts around how to write your tests. When 
writing a unit test, the thing being tested is called the *System Under Test* (or SUT for short). 
The SUT can be just be about anything but is typically a method in a particular class. 
Since a unit test should only test a single thing then it follows that each of your tests should 
only have a single SUT.

We typically write multiple tests to test the behaviors of the SUT that we are testing. We usually 
group these tests together into a single class in order to keep them organized into a single logical unit. 
These multiple tests are what are referred to as a *test suite* for that particular component.

#### Arrange, Act, Assert

When writing a unit test, we follow the *Arrange, Act, Assert* pattern. Each word in the pattern names 
a particular stage of the test.

During the *arrange* stage, the SUT is setup for testing. This generally involves instantiating the 
SUT and setting up any *fakes* or dependencies that the SUT may rely on. We'll cover the concept of 
fakes and when to use them in a later blog post, but for now it should be enough to say that they are 
something that can serve in place of one of the SUT's dependencies. 

During the *act* stage, some action is invoked on the SUT that you wish to test. This is typically a 
method call where the method called is the SUT. When writing the test, we are actually testing against 
the result of this action. This result is typically the method's return value or some other side effect 
of the method call. 

During the *assert* stage, we make some assertion about the result from the *act stage* of the test. 
If the assertion made about the result of the test doesn't pass, then we say that the test has failed. 
xUnit provides multiple built-in assertion types that make writing our tests easier.

#### Writing a Unit Test

As an example, we're going to write some unit tests for the simple contrived class below that performs 
some common mathematical operations on two numbers.

{% highlight csharp %}
public class MathService
{
	public int Add(int a, int b)
	{
		return a + b;
	}
	
	public int Multiply(int a, int b)
	{
		return a * b;
	}
}
{% endhighlight %}
  
I like to use a pattern where we have an outer class that groups all the tests together and then an 
inner class for each SUT that we are testing. Each of these inner classes can have multiple tests for 
each SUT under test. I believe this makes our tests much more organized and makes it so that our tests 
read like a sentence.

{% highlight csharp %}
public class MathServiceFacts
{
	public class TheAddMethod
	{
		[Fact]
		public void ReturnsTenForEightAndTwo()
		{
			// Arrange
			var service = new MathService();
			
			// Act
			var result = service.Add(8, 2);
			
			// Assert
			Assert.Equal(10, result);
		}
		
		[Fact]
		public void IsCommutative()
		{
			// Arrange
			var service = new MathService();
			
			// Act
			var result1 = service.Add(8, 2);
			var result2 = service.Add(2, 8);
			
			// Assert
			Assert.Equal(result1, result2);
		}
	}
}
{% endhighlight %}

You may have noticed the use of the `[Fact]` attribute on our test methods, this attribute is necessary 
to signal to xUnit that the method is a test. Otherwise the method won't be executed by xUnit whenever 
our tests are run. 

For the most part our tests are simple and straight forward. We are adhering closely to the *arrange, 
act, assert* pattern that was described earlier, and we are using one of the assertion types that is 
built in to xUnit in order to test the result of our test. 

In the next blog post, we'll cover how to design components with testability in mind. 