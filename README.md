# Exception-guidelines - "Do’s and Don’ts for Exceptions"

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

