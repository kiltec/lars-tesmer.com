---
layout: post
title: "What My Co-Workers and I Learned When Trying to Write Unit Tests for PHPUnit"
date: 2011-09-08 23:06
comments: true
published: true
categories: [PHPUnit, Unit Testing, Code Quality, Clean Code] 
---
Today a bunch of co-workers and me got together right after work to hone our skills, more specifically, our unit testing skills.  
A couple of weeks before, I had discovered that the code coverage of [PHPUnit](https://github.com/sebastianbergmann/phpunit/) is only at ~55%, so I thought it would be a great exercise for our after work hacking to help increase its code coverage by writing unit tests for it.

The plan was to try and write as many tests as we could for the [Constraint classes](https://github.com/sebastianbergmann/phpunit/tree/master/PHPUnit/Framework/Constraint) PHPUnit uses to implement its assertions.  
Those Constraint classes are pretty small, fairly easy to understand and not entirely covered by tests - in short, very well suited for our group, a mix of programmers having quite some experience in unit testing as well as others just having started to learn unit testing.

Well, our plan didn't work out that way, we didn't really succeed in writing a considerable amount of unit tests.  
However, it still was a valuable experience, as it turned out the unit tests of the Constraints are a good example of how not to unit test.

So here's what we learned (or were reminded of):
<!-- more -->
### 1. Don't use one single test case class to test several different classes  ###
The unit tests for the constraint classes of PHPUnit are lumped together [in one large file](https://github.com/sebastianbergmann/phpunit/blob/3.5/Tests/Framework/ConstraintTest.php). I think the reasoning behind that might have been that all constraints derive from the same parent class, *PHPUnit_Framework_Constraint*.  
Don't do that.  
It renders it quite hard to find your way to the tests for one of the constraints, it's quite annoying to navigate through that class.  
If each class has its own test case, it's much easier to find its test, too.

### 2. Name your tests well ###
You should name your tests such that they convey the intent of the test, describe what the test is about and how they're different from other tests.   
The tests in [ConstraintTest.php](https://github.com/sebastianbergmann/phpunit/blob/3.5/Tests/Framework/ConstraintTest.php) wear names like:

* testConstraintIsEmpty
* testConstraintPCREMatch
* testConstraintClassNotHasStaticAttribute
* testConstraintStringMatches
* testConstraintStringMatches2
* testConstraintStringMatches3
* testConstraintStringMatches4
* testConstraintStringMatches5
* testConstraintStringMatches6 (sic!)

It's really hard to understand what's going on in tests named in such a way.

### 3. Avoid to test more than one behaviour in one single test 
The tests in ConstraintTest.php test a lot of behavior in one single test, here's an example:

``` php Whoa! A lot of stuff going on in here! 
    /**
     * @covers PHPUnit_Framework_Constraint_IsType
     * @covers PHPUnit_Framework_Assert::isType
     * @covers PHPUnit_Framework_Constraint::count
     * @covers PHPUnit_Framework_TestFailure::exceptionToString
     */
    public function testConstraintIsType()
    {
        $constraint = PHPUnit_Framework_Assert::isType('string');

        $this->assertFalse($constraint->evaluate(0, '', TRUE));
        $this->assertTrue($constraint->evaluate('', '', TRUE));
        $this->assertEquals('is of type "string"', $constraint->toString());
        $this->assertEquals(1, count($constraint));

        try {
            $constraint->evaluate(new stdClass);
        }

        catch (PHPUnit_Framework_ExpectationFailedException $e) {
            $this->assertEquals(<<<EOF
Failed asserting that stdClass Object () is of type "string".

EOF
              ,
              self::trimnl(PHPUnit_Framework_TestFailure::exceptionToString($e))
            );

            return;
        }

        $this->fail();
    }
```
Especially in conjunction with #2, this makes it really hard to figure out what a given test is supposed to test. In our practice group we sat in pairs in front of our laptops and yet it still took us quite some time to find out what those tests were actually doing.

If you find yourself writing more than one assertion in a single test, take your hands from the keyboard and find out whether you're about to test more than a single behavior.  
Otherwise you will probably curse yourself when you come back to that test code in, say, 6 months...

### Conclusion
If you plan to introduce others to unit testing by letting them loose on a real world codebase, I'd suggest not to pick PHPUnit, as ironic as it may sound.  
PHPUnit itself is a very valuable tool and I'm very grateful for its existence, I'm using it daily at work. Yet, its test are no good example of how to write good unit tests, working on those tests will cause quite some confusion especially among programmers new to unit testing.

Anyways, for our next practice group meeting we'll choose something more appropriate, we're thinking about trying our luck on [Symfony's Assetic](https://github.com/kriswallsmith/assetic).  

However, we will very probably come back to PHPUnit at some point in the future in order to try and refactor its tests.
