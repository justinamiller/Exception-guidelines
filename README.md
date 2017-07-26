# Exception-guidelines

### When Should You Log Exceptions?
The short and sweet answer is that you should always log exceptions. For many developers, the general rule of thumb is to log exceptions when code could crash. The problem with this approach, however, is that even the simplest code can cause crashes in the application.

### Top five tips for handling .NET Exceptions.

***1. Maintain your stack trace***

When you first start programming in .NET you might think that you catch the exception, try some stuff but rethrow that exception if you couldn’t handle it.  Or, you may have defined your own exception class.  It might have looked like the following code.

In both these cases, you will find that the stack trace provided with your exception will go no further down the call stack than the method PartialStackTrace.
````
private double PartialStackTrace()
{
  try
  {
       return DivideANumber(1, 0);
  }
  catch (Exception ex)
  {
       throw ex;
  }
}

OR

private double PartialStackTrace()
{
  try
  {
       return DivideANumber(1, 0);
  }
  catch (Exception ex)
  {
       throw new MyAppException(“Ooops!”);
  }
}
````
You really would like a full stack trace, that reveals that the problem exists in DivideANumber (or further down the call stack).

If you are just re-throwing the exception use throw by itself, and the full stack trace is preserved.

If you define your own exception class, provide the exception you just caught as the InnerException in the constructor.  Job done!
````
private double FullStackTrace()
{
  try
  {
       return DivideANumber(1, 0);
  }
  catch (Exception ex)
  {
       throw;
  }
}

OR

private double FullStackTrace()
{
  try
  {
       return DivideANumber(1, 0);
  }
  catch (Exception ex)
  {
       throw new MyAppException(“Ooops!”, ex);
  }
}
````

***2. Keep exceptions exceptional…***

This sounds a bit trite, but sometimes you won’t need exceptions as you know there will be times that you will receive data or be in a situation where an exception might occur, and you know how to handle it.

As an example, it is trivial to verify that a key exists in a dictionary prior to attempting to access a value, and save having to wrap dictionary access in a try … catch block.

````
ar alphabet = new Dictionary<int, string>() { { 1, "A" }, { 2, "B" } };

// throws KeyNotFoundException

try
{
    Console.WriteLine($"27th letter of alphabet {alphabet[27]}");
}
catch (KeyNotFoundException kex)
{
    Console.WriteLine("27th letter of alphabet : <not present>");
}

// check first, no try ... catch required
var letter = alphabet.ContainsKey(27) ? alphabet[27] : "<not present>";
Console.WriteLine($"27th letter of alphabet : { letter }");k
````
Similarly, you might want to check for the existence of a file with the same name before an attempt to copying a new file.  This could be gracefully handled by changing the resulting file name and providing the new file name back as a result.

***3. Avoid generic, catch-all exception handling***

Not only should you not throw exceptions for conditions that are expected to occur regularly, you should also always aim to catch specific exceptions, rather than generic exceptions.  This has become finer grained in C# 6 as it supports filtering using catch ... when and supplying a suitable predicate.

````
// handle specific exception differently

try
{
    double result = DivideByZeroCheckForSpecificException();
    Console.WriteLine($"Divide by zero results : {result}\n");
}
catch (DivideByZeroException dex)
{
    Console.WriteLine("Divide by zero results : <cannot divide by 0>\n");
}
catch (Exception ex)
{
    Console.WriteLine("Unknown exception : \n\n" + ex.ToString() + "\n");
}

// handle exception using a catch ... when predicate 

try
{
    double result = DivideByZeroCheckExceptionPredicate();
    Console.WriteLine($"Divide by zero results : {result}");
}
catch (Exception ex) when (ex.InnerException != null)
{
    Console.WriteLine("An InnerException exists : \n\n" + ex.ToString() + "\n");
}
````

***4. Async and exception handling***

The world of async has given us a few more things to think about regarding exceptions.  The first is that you will be expected to handle the AggregateException much more often than you may have done previously.  This wrapper could be wrapping several exceptions, or just a single exception.  You have to examine the InnerExceptions property to see if there are underlying exceptions which you could handle.

It should also be noted that methods which are decorated with async will not throw exceptions unless you await them.  This can be easy to miss when creating unit tests with mocks and stubs, and can lead to much frustration as tests suddenly fail due to exceptions not being thrown and handled as expected.

````
internal static async Task ThrowAnException()
{
    throw new Exception("Throwing from an async decorated method.");
}
private static async Task TestAsyncExceptions()
{

  // No exception will be thrown, we omitted await

  try
  {
    ThrowAnException();
    Console.WriteLine("No exception thrown.");
  }
  catch (Exception ex)
  {
    Console.WriteLine("Exception thrown : \n\n" + ex.ToString() + "\n");
  }

  // Exception will be thrown because we have used await

  try
  {
    await ThrowAnException();
    Console.WriteLine("No exception thrown.");
  }
  catch (Exception ex)
  {
    Console.WriteLine("Exception thrown : \n\n" + ex.ToString() + "\n");
  } 
}
````

***5. Don’t rely on exception message content, it could be localised***

Even when you do capture on specific exception types, you might find that the exception is ambiguous and doesn’t really inform you of the underlying issue.  It can be tempting to check for a given exception message to really confirm that the exception is something you think you can handle.

````
try
{  
  getDocument = DocStore.SingleOrDefault(d => d.SelfLink = doc.SelfLink); 
}
catch (System.InvalidOperationException ex)
{    
  if (ex.Message == “Sequence contains more than one matching element”)  
  {     
    CleanUpDuplicates(doc.SelfLink);  
  }  
  else  
  {    
   throw;  
  }
}
````


### "Do’s and Don’ts for Exceptions"
1. Don’t just wrap an entire method with one try-catch. Place try-catch around specific code.

**Wrong**
````
public ActionResult Index()
{
    try
    {
       // all of your code here

    } catch (Exception e)
   {
      // do something
   }
}
````

**Right**
````
public ActionResult Index()
{
  int myNumber1 = 0;
  int myNumber2 = 1;
    try
    {
       int result = DivideNumbers(3, 4);
    } catch (Exception e)
   {
      // do something
   }
       try
    {
       int result2 = DivideNumbers(myNumber1, myNumber2);
    } catch (Exception e)
   {
      // do something
   }
   
   
 

}
````

2. Do be descriptive. Your logs should contain specific information that helps you identify the root of an error. Don’t just log “error.” Log “Critical error in logging class” and then log the exception information.

3. Don’t have an empty catch statement. In other words, don’t just “do nothing” with the caught exception. You should log the information and then either redirect the user or alert the user that something went wrong.

4. Do inherit from the Exception class for custom exceptions. You can define your own exceptions for specific programming scenarios. For instance, you might want to define a specific error for your data layers to catch connection issues and bugs.

5. Don’t throw errors that are actually not errors. For instance, if we query a database for a list of users and no users are returned, this isn’t an error. An empty data set is not an error even if logically it shouldn’t happen. Handle this type of scenario using logic workflow.
