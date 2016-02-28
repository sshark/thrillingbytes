# To Checked or to Unchecked Exceptions
## Introduction
Java classes `java.lang.Error` and `java.lang.Exception` manage errors and exceptions. They are direct subclasses of `java.lang.Throwable`. Errors are thrown when serious problems occur and the application is not supposed to capture them and let the JVM terminates. Exceptions, on the other hand, are recoverable errors most of the time. Developers should catch them and try an alternate logic flow to mitigate the errors. Exceptions are divided into 2 groups of checked and unchecked exceptions, `java.lang.Exception` and `java.lang.RuntimeException` respectively. Future exceptions `extends java.lang.Exception` are checked exceptions and likewise, unchecked exceptions *a.k.a* runtime exceptions `extends java.lang.RuntimeException`. Ironically, `java.lang.RuntimeException` extends `java.lang.Exception`. 

This article is not about handling exceptions but to show why unchecked exceptions are preferred over checked exceptions. There are lots of material available for those who wants to learn more in-depth about Java exceptions.

## Checked Exceptions
Java introduced checked exceptions to make sure that exceptions are properly addressed to during compilation and developers are obliged to catch them with the *try-catch* block or re-throw them to the caller to manage. A typical Java class  handling checked exceptions,

**Example 1**: *`foo(...)` should throw an exception if `r` is more 60 and `bar(...)` should throw an exception if `r` is more than 70. If `r` is less than 60, the "main" method shall print the sum of `bar(...)` and `foo(...)` and 20*

	public class Checked {
	    public static void main(String[] args) {
	        int r = Integer.parseInt(args[0]);

	        if (r > 60) {  // preflight check
		        System.out.println(20);
		    } else { 
		        Checked handling = new Checked();
		        System.out.println(handling.fuzz(r));
		    }
	    }
	
	    public int fuzz(int r) {
	        try {
	            return bar(r) + foo(r) + 20;
	        } catch (Exception e) {
	            return 20;
	        }
	    }
		
	    public int foo(int r) throws Exception {
	        if (r > 60) throw new Exception("foo value more than 60");
	        return 100 / r;
	    }
	
	    public int bar(int r) throws Exception {
	        if (r > 70) throw new Exception("bar value more than 70");
	        return 200 / r;
	    }
	}

Although there is a *preflight* check, `fuzz(...)` has to include the unnecessary *try-catch* boilerplate or otherwise, it will not compile.

## Runtime Exceptions
Developers are not obliged to catch runtime exceptions. The technique to catch runtime exceptions is the same as catching checked exceptions. Uncaught runtime exceptions behave like checked exceptionsm they get thrown from one method to another until it reaches the "top" and the JVM terminates e.g. the infamous uncaught `NullPointerException`. Runtime exception is *implicitly* thrown from one method to another unless there is a *try-catch* block waiting to catch it.

**Example 2**: *Same requirements as Example 1*

	public class Unchecked {
	    public static void main(String[] args) {
	        int r = Integer.parseInt(args[0]);
	
	        if (r > 60) {  // preflight check
	          System.out.println(20);
	      } else { 
	          Unchecked handling = new Unchecked();
	          System.out.println(handling.fuzz(r));
	      }
	    }
	
	    public int fuzz(int r) {
	        return bar(r) + foo(r) + 20;
	
	        /* previous try-block is made obsolete because of preflight check
	        try {...}
	        */
	    }
	  
	    public int foo(int r) {
	        if (r > 60) throw new RuntimeException("foo value more than 60");
	        return 100 / r;
	    }
	
	    public int bar(int r) {
	        if (r > 70) throw new RuntimeException("bar value more than 70");
	        return 200 / r;
	    }
	}

`fuzz(...)` method has become much simple. The *preflight* check removes the necessity of the cumbersome *try-catch* block if the input is already verified and usable. *Preflight* check are classes that deal with external sources and vet the input before passing on to the other internal methods. 

*Can you spot the bug in Examples 1 and 2?*

## Conclusion
Using runtime exceptions frees the developer from repeating the *try-catch* blocks when the input is known to be safe and usable. 

Methods that deal directly with external sources can use the `try` block as one of the many ways to manage random inputs from external sources.
