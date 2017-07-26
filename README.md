# Exception-guidelines

### When Should You Log Exceptions?
The short and sweet answer is that you should always log exceptions. For many developers, the general rule of thumb is to log exceptions when code could crash. The problem with this approach, however, is that even the simplest code can cause crashes in the application.

### Top five tips for handling .NET Exceptions.

1. Maintain your stack trace
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
