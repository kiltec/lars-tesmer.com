---
layout: post
title: "PHPUnit: Better Syntax for Expecting Exceptions"
date: 2011-08-29 18:32
comments: true
categories: [PHPUnit,PHP,Ruby,Unit Tests] 
---
Recently, while starting to learn Ruby, I've stumbled upon the following code snippet
which demonstrates how you can assert that a piece of code throws as exception in Ruby's Test::Unit:

{% codeblock Expecting exceptions in Test::Unit lang:ruby %}
def test_some_important_method()
	some_obj = new SomeObj
	# Potentially large amount of code

	assert_raise InvalidArgumentException do
		some_obj.someMethod
	end
end
{% endcodeblock %}
Check out lines 5-7 - isn't this a really elegant and readable way of expecting an exception?
Now let's contrast this with how it's done in PHPUnit:
<!--more-->
{% codeblock Current way of expecting exceptions in PHPUnit lang:php %}
<?php
/**
 * @expectedException InvalidArgumentException
 */
public function testSomeImportantMethod() {
	$some_obj = new SomeObj();
	// Potentially large amount of code

	$some_obj->someMethod();
}
{% endcodeblock %}
Well, that isn't too horrible but I've never been really happy with this.  
My main issues with this way of expecting exceptions are:

The expectation is pretty far away from the location you'd normally expect to find an assertion.
Usually, an assertion can be found at the bottom of each test function, whereas with the current method PHPUnit uses,
it's at the *top* of the test-function.

Additionally, it's an annotation "buried" in a comment which is easy to miss.

Finally, PHPUnit will watch for an exception thrown by *any* of the code inside the test-function. Normally,
it's the last line of the test-function, so it isn't hard to find - but what if the expected exception is thrown in
a line *before* the last line, maybe due to a bug?

So, is there a way to mimic the method of Test::Unit in PHPUnit?  
As you may have guessed, there is one, using anonymous functions and closures, which are available since PHP 5.3!

Here's the code for a simple function we can use to check if a given piece of code throws an exception:
{% codeblock lang:php %}
<?php
function assertThrowsException($exception_name, $code) {
	$e = null;
	try{
		$code();
	}catch (Exception $e) {
		// No more code, we only want to catch the exception in $e
	}

	if($e && $e instanceof $exception_name) {
		echo "\nCorrect exception thrown!\n";
	}else{
		echo "\nIncorrect exception thrown!\n";
	}
}
{% endcodeblock %}
Confused?  
Well, the most important line of that function is line 5.    
There, the anonymous function that was passed into the function as the second argument gets executed, which allows
us to catch any exception thrown by it.

Still confused? Let's check out an example:
{% codeblock Let's throw an exception, yay! lang:php %}
<?php
class ExceptionThrower {
	public function execute() {
		throw new \InvalidArgumentException("I'm an exception");
	}
}

$subject = new ExceptionThrower();

assertThrowsException('InvalidArgumentException', function () use ($subject) {
		$subject->execute();
	}
);
{% endcodeblock %}

The magic happens in lines 10-13:  
The *execute*-method of ExceptionThrower doesn't get executed immediately because it's wrapped in an anonymous function, i.e.
*execute* will only be invoked when we call the anonymous function contained in *$code* in line 5 of *assertThrowsException()*.

Now one question remains - can this be done in PHPUnit with a "proper" assertion?  
Yes, it can!

I won't post the full code into this blog, it would be a bit too much but you can find the code adding a new assertion
*assertThrowsException()* in the following commit to my fork of PHPUnit 3.5:  
[Link to Github commit](https://github.com/kiltec/phpunit/commit/dd5c7bd71d6eb8d4b58ce79b5ae069fbb0734354)  
Note that this code is not production-ready, it's a proof-of-concept with just enough code to get PHPUnit to run the
new assertion without throwing a fit, just so I could find out whether such a new assertion would be possible at all.

Anyway, with the new assertion the PHPUnit test-method from the beginning of this blog would look like this:
{% codeblock lang:php %}
<?php
public function testSomeImportantMethod() {
	$someClass = new SomeClass();

	$this->assertThrowsException('InvalidArgumentException', function () use($someClass) {
			$someClass->someMethod();
		}
	);
}
{% endcodeblock %}

Granted, you have more to type than in the current way but in my opinion, this offers more control and is more expressive and I'd prefer 
expressiveness over terseness any day.

What do you think, should I turn this into a real patch for PHPUnit?  
Or am I just crazy? ;)
