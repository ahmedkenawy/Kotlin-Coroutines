# Kotlin Coroutines


##  What exactly Coroutines are?

|Co|Routines|
|--|--|
| Cooperation |Function  |

**It means that when functions cooperate with each other, we call it Coroutines.**

For Example 

	 

	

    fun functionA(case: Int) {
	     when (case) {
		      1 -> { taskA1() functionB(1) } 
		      2 -> { taskA2() functionB(2) } 
		      3 -> { taskA3() functionB(3) }
		      4 -> { taskA4() functionB(4) } 
		      }
		  }
		

	
    fun functionB(case: Int) {
	     when (case) { 
		     1 -> { taskB1() functionA(2) } 
		     2 -> { taskB2() functionA(3) } 
		     3 -> { taskB3() functionA(4) } 
		     4 -> { taskB4() }
		    }
		 }
			
**Then, we can call the functionA as below:**

    functionA(1)
Here, **functionA** will do the **taskA1** and give control to the **functionB** to execute the **taskB1**.

Then, **functionB** will do the **taskB1** and give the control back to the **functionA** to execute the **taskA2** and so on.

The important thing is that functionA and functionB are cooperating with each other.

With Kotlin Coroutines, the above cooperation can be done very easily which is without the use of  **when**  or  **switch**  **case** which I have used in the above example for the sake of understanding.

Now that, we have understood what are coroutines when it comes to cooperation between the functions. There are endless possibilities that open up because of the cooperative nature of functions.

**Coroutines**  are available in many languages. Basically, there are two types of Coroutines:

-   Stackless
-   Stackful

Kotlin implements stackless coroutines — it means that the coroutines don’t have their own stack, so they don’t map on the native thread.

## **Why there is a need for Kotlin Coroutines?**
-   Fetch User from the server.
-   Show the User in the UI.

				

	    fun fetchUser(): User {
	     // make network call  
	     // return user 
	     }
	    fun showUser(user: User) {
	     // show user
	      }
	    fun fetchAndShowUser() {
	        val user = fetchUser()
	        showUser(user) 
	      }
	   
	
	When we call the  **fetchAndShowUser**  function, it will throw the  **NetworkOnMainThreadException** as the network call is not allowed on the main thread.

	There are many ways to solve that. A few of them are as follows:

 1. **Using Callback:** Here, we run the fetchUser in the background thread and we pass the result with the callback.
 2. **Using RxJava:** Reactive world approach. This way we can get rid of the nested callback.
 3. **Using Coroutines:** Yes, coroutines.
 
 **We have introduced two things here as follows:**
 -   **Dispatchers**: Dispatchers help coroutines in deciding the thread on which the work has to be done. There are majorly three types of Dispatchers which are as  **IO, Default, and Main**. IO dispatcher is used to do the network and disk-related work. Default is used to do the CPU intensive work. Main is the UI thread of Android. In order to use these, we need to wrap the work under the  **async**  function. Async function looks like below.

-   **suspend**: Suspend function is a function that could be started, paused, and resume.
	- https://s3.ap-south-1.amazonaws.com/mindorks-server-uploads/suspend-coroutines.png
	Suspend functions are only allowed to be called from a coroutine or another suspend function. You can see that the  **async**  function which includes the keyword  **suspend**. So, in order to use that, we need to make our function  **suspend**  too.

		So, the  **fetchAndShowUser**  can only be called from another suspend function or a coroutine. We can't make the onCreate function of an activity  **suspend**, so we need to call it from the coroutines like below:

		

		    override fun onCreate(savedInstanceState: Bundle?) { 
		    super.onCreate(savedInstanceState)
		    GlobalScope.launch(Dispatchers.Main) {
		     fetchAndShowUser() 
			     }
		      }

		

 ##  **Launch vs Async in Kotlin Coroutines**
 -   `launch`: fire and forget
-   `async`: perform a task and return a result
- `withContext` Like async but without  `await()`

**The thumb-rules:**
-   Use  `withContext`  when you do  **not**  need the parallel execution.
-   Use  `async`  only when you need the parallel execution.
-   Both  `withContext`  and  `async`  can be used to get the result which is not possible with the  `launch`.
-   Use  `withContext`  to return the result of a single task.
-   Use  `async`  for results from multiple tasks that run in parallel.

## Scopes in Kotlin Coroutines

Scopes in Kotlin Coroutines are very useful because we need to cancel the background task as soon as the activity is destroyed.

**There are basically 3 scopes in Kotlin coroutines:**
-   Global Scope.
-   LifeCycle Scope.
-   ViewModel Scope.

##  Exception Handling in Kotlin Coroutines

**When Using launch**

    GlobalScope.launch(Dispatchers.Main) { 
	    try { fetchUserAndSaveInDatabase() 
		    // do on IO thread and back to UI Thread
		    }
		catch (exception: Exception) { 
			Log.d(TAG, "$exception handled !") 
			}
		 }
	

Another way is to use a handler:
	

    val handler = CoroutineExceptionHandler { _, exception -> 
	    Log.d(TAG, "$exception handled !")
	   }
	  
Then, we can attach the handler like below:

    GlobalScope.launch(Dispatchers.Main + handler) {
     fetchUserAndSaveInDatabase()
     // do on IO thread and back to UI Thread
      }
**what if we want to make the network calls in parallel. We can write the code like below using `async`**

    launch {
	     try {
		      val usersDeferred = async { getUsers() } 
		      val moreUsersDeferred = async { getMoreUsers() } 
		      val users = usersDeferred.await() 
		      val moreUsers = moreUsersDeferred.await() 
		      } 
		  catch (exception: Exception) {
			   Log.d(TAG, "$exception handled !") 
			   }
			 }
Here, we will face one problem, if any network error comes, the application will **crash!**, it will **NOT** go to the `catch` block.

**To solve this, we will have to use the `coroutineScope` like below:**

    launch { 
	    try { 
		    coroutineScope { 
			    val usersDeferred = async { getUsers() } 
			    val moreUsersDeferred = async { getMoreUsers() } 
			    val users = usersDeferred.await() 
			    val moreUsers = moreUsersDeferred.await() 
			    } 
			} 
		catch (exception: Exception) { Log.d(TAG, "$exception handled !") 
			}
		 }
		 
Now, if any network error comes, it will go to the `catch` block.
