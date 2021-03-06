## This is an add-in for [Fody](https://github.com/Fody/Fody/) 

Compile time decorator pattern via IL rewriting.

[Introduction to Fody](http://github.com/Fody/Fody/wiki/SampleUsage)

This version is fork of [Fody/MethodDecorator](https://github.com/Fody/MethodDecorator) with changes I found useful

Differences from original Fody/MethodDecorator:
* No attributes or interfaces in root namespace (actually without namespace) required
* Interceptor attribute can be declared and implemented in separate assembly
* Init method called before any method and receiving method reference and method args 
* OnEntry/OnExit/OnException methods don't receiving method reference anymore

### Your Code
	//Atribute should be "registred" by adding as module or assembly custom attribute
	[module: Interceptor]
	
	//Any attribute which provide OnEntry/OnExit/OnException with proper args
	[AttributeUsage(AttributeTargets.Method | AttributeTargets.Constructor | AttributeTargets.Assembly | AttributeTargets.Module)]
	public class InterceptorAttribute : Attribute, IMethodDecorator	{
	    public void Init(MethodBase method, object[] args) {
			TestMessages.Record(string.Format("Init: {0} [{1}]", method.DeclaringType.FullName + "." + method.Name, args.Length));
		}
		public void OnEntry() {
	        TestMessages.Record("OnEntry");
	    }
	
	    public void OnExit() {
	        TestMessages.Record("OnExit");
	    }
	
	    public void OnException(Exception exception) {
	        TestMessages.Record(string.Format("OnException: {0}: {1}", exception.GetType(), exception.Message));
	    }
	}
	
	public class Sample	{
		[Interceptor]
		public void Method()
		{
		    Debug.WriteLine("Your Code");
		}
	}

### What gets compiled
	
	public class Sample
	{
		public void Method(int value)
		{
		    MethodBase method = methodof(Sample.Method, Sample);
		    InterceptorAttribute attribute = (InterceptorAttribute) method.GetCustomAttributes(typeof(InterceptorAttribute), false)[0];
		    object[] args = new object[1] { (object) value };
			attribute.Init(methodFromHandle, args);

			attribute.OnEntry();
		    try
		    {
		        Debug.WriteLine("Your Code");
		        attribute.OnExit();
		    }
		    catch (Exception exception)
		    {
		        attribute.OnException(exception);
		        throw;
		    }
		}
	}

NuGet: https://www.nuget.org/packages/MethodDecoratorEx.Fody/
	
In plans:
* Make Init method optional
* Add "this" as parameter to Init method if method is not static
* Pass return value to "OnExit" if method returns any

