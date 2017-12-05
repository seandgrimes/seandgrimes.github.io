---
layout: post
title:  "Designing With Testability In Mind"
date:   2015-03-12 12:00:00 -0600
categories: csharp testing
---

Although there are some special considerations, designing for testability largely involves following well established software design principles. In this blog post I'll cover these design principles and how they help with testability and also cover any special considerations that you'll need to keep in mind.

#### Useful Software Design Principles

* Single Responsiblity Principle

For those not familiar with the [Single Responsiblity Principle](http://en.wikipedia.org/wiki/Single_responsibility_principle) (SRP), it was originally proposed by Uncle Bob Martin and states that a class should have a single responsibility or reason to change. The most important thing about this principle is that it promotes the separation of concerns. Consider the example below:

{% highlight csharp %}
public class StateRespository
{
    private static IDictionary<string, Object> _Cache = 
        new ConcurrentDictionary<string, Object>();
            
    // ...
        
    public IList<State> GetStates()
    {
        const string cacheKey = "StateRepository_GetStates";
        if (_Cache.HasKey(cacheKey))
        {
            var states = _Cache[cacheKey] as List<State>;
            return states;
        }
            
        var states = _DbContext.States
        	.OrderBy(s => s.Abbreviation)
            .ToList();
                
		_Cache[cacheKey] = states;
        return states;
    }
}
{% endhighlight %}
    
This class has two separate responsiblities, retrieving data from a database and caching query results in memory. If we wanted to test only that caching is working correctly in GetStates(), we couldn't do that without testing the caching and querying logic together. By separating the caching and querying concerns into two different classes, we could test both independently and make testing easier.

* Loose Coupling
                
[Loose coupling](http://en.wikipedia.org/wiki/Loose_coupling) occurs in a software system when one component has very little knowledge about the implementation of another component. One of the easiest ways to accomplish loose coupling between components is to have components rely on interfaces for other components instead of concrete implementations. Not only does this help hide implementation details but it also makes it easy to have a component use an entirely different implementation. 

Let's revisit the previous example, we'll fix the violation of the SRP by moving the caching logic to a separate class. 

{% highlight csharp %}
public interface ICachingService
{
	bool TryGetValue<TValue>(string key, out TValue val);
	void Set<TValue>(string key, TValue val);
}
    
public class DictionaryCachingService : ICachingService
{
	private IDictionary<string, object> _Cache = 
		new ConcurrentDictionary<string, object>();
		
	public bool TryGetValue<TValue>(string key, out TValue val)
	{
		if (_Cache.HasKey(key))
		{
			val = (TValue)_Cache[key];
			return true;
		}
		
		val = default(TValue);
		return false;
	}
	
	public void Set<TValue>(string key, TValue val)
	{
		_Cache[key] = val;
	}
}
{% endhighlight %}
    
Next we'll ensure loose coupling between the StateRepository and the caching service by having the StateRepository only reference *DictionaryCachingService* by its interface.

{% highlight csharp %}
public class StateRespository
{
	private ICachingService _CachingService;
		
	// ...
	
	public StateRepository(ICachingService cachingService)
	{
		_CachingService = cachingService;
		// ...
	}
	
	public IList<State> GetStates()
	{
		const string cacheKey = "StateRepository_GetStates";
		
		List<State> states = null;
		if (_CacheService.TryGetValue(cacheKey, out states)
		{
			return states;
		}
		
		states = _DbContext.States
			.OrderBy(s => s.Abbreviation)
			.ToList();
			
		_CacheService.Set(cacheKey, states);
		return states;
	}
}
{% endhighlight %}
    
Now we can test the query and caching logic separately:

{% highlight csharp %}
public class StateRepositoryFacts
{
	public class TheGetStatesMethod
	{
		[Fact]
		public void CachesDatabaseQueries()
		{
			// Arrange
			ICacheService cacheService = new DictionaryCacheService();
			var sut = new StateRepository(cacheService);
			const string cacheKey = "StateRepository_GetStates";
			List states = null;
			
			// Act
			sut.GetStates();
			bool cached = cacheService.TryGetValue(cacheKey, out states);
			
			// Assert
			Assert.True(cached);
		}
	}
}

public class DictionaryCacheServiceFacts
{
	public class TheTryGetValueMethod
	{
		[Fact]
		public void ReturnsCorrectValueWhenKeyExists()
		{
			// Arrange
			const string cacheKey = "TestKey";
			const string testValue = "TestValue";
			var sut = new DictionaryCacheService();
			sut.Set(cacheKey, testValue);
			
			// Act
			string actual = null;
			sut.TryGetValue(cacheKey, out actual);
			
			// Assert
			Assert.Equal(testValue, actual);
		}
	}
}
{% endhighlight %}
         
#### Special Considerations

Using the above software design principles will allow you to test the vast majority of the code in your system, but there is one thing that can't be tested with this approach: non-public methods.

A strong argument can be made that you should only test the public interface of your classes and ignore testing the various implementation details like private and protected methods. One reason is that the public interface of your class is much less likely to change than its internal implementation details. Tests that make assumptions about how the internals of a class work are more brittle than those that don't and require more maintenance in the long run. 

That being said, there are a couple of ways to test the behavior of private and protected methods that don't require making these methods part of the class's contract. The simplest would be to use the [InternalsVisibleTo Attribute](https://msdn.microsoft.com/en-us/library/system.runtime.compilerservices.internalsvisibletoattribute%28v=vs.110%29.aspx) to make these methods visible to the assembly that your tests reside in. You would then be able to write tests directly against these methods but I would suggest against doing this.

A slightly better way would be to rely on testing the side effects of the non-public methods in your class being called, this will still lead to brittle tests but they will be less brittle than relying on the *InternalsVisbileTo* attribute. 

Arguably the best solution would be to extract these non-public methods to a new class entirely. If you can do this in such a way that this logic can be shared by multiple classes then you've already improved the maintainability of your system as a whole. The only major downside to this approach is that you may wind up with classes whose only reason for existing is to make the code they contain testable. 

In the next blog post, we'll discuss the differences between unit and integration tests and cover the use of test doubles to convert integration tests to unit tests. 
                