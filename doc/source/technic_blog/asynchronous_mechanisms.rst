多线程.多进程.异步通信
======================

最近的项目里面碰到多线程的问题，有些问题莫名其妙。 归根结底都是多线程导致的，任何技术，如果他带来巨大的好处，一定会有一些副作用。识别并且管控这些副作用，对软件的质量非常重要。


std::thread::join
-------------------

Blocks the current thread until the thread identified by *this* finishes its execution.


.. code-block:: c++

	#include <iostream>
	#include <thread>



	void g(int n, std::string taskName)
	{
	    for(int i=0; i<n; i++)
	    {
		std::cout << taskName << " is ongoing i = " << i << std::endl;
	    }
	    std::cout << taskName << " is exit." << std::endl; 
	}

	int main()
	{
	    std::thread t1(g, 1000, "task1");
	    std::thread t2(g, 300, "task2");

	    std::cout << "main thread" << std::endl;

	    t2.join();
	    std::cout << "start to t1.join" << std::endl;
	    t1.join();

	    std::cout << "t1.join done" << std::endl;
	}


std::thread::detach
---------------------

	Separates the thread of execution from the thread object, allowing execution to continue independently. Any allocated resources will be freed once the thread exits.

	After calling detach *this* no longer owns any thread. 

**example**

.. code-block:: c++

	#include <iostream>
	#include <chrono>
	#include <thread>
	 
	void independentThread() 
	{
	    std::cout << "Starting concurrent thread.\n";
	    std::this_thread::sleep_for(std::chrono::seconds(2));
	    std::cout << "Exiting concurrent thread.\n";
	}
	 
	void threadCaller() 
	{
	    std::cout << "Starting thread caller.\n";
	    std::thread t(independentThread);
	    t.detach();
	    std::this_thread::sleep_for(std::chrono::seconds(1));
	    std::cout << "Exiting thread caller.\n";
	}
	 
	int main() 
	{
	    threadCaller();
	    std::this_thread::sleep_for(std::chrono::seconds(5));
	}

**output**

.. code-block:: console

	yinwu@yinwu-ThinkPad-E475:~/learning/cplus$ ./a.out 
	Starting thread caller.
	Starting concurrent thread.
	Exiting thread caller.
	Exiting concurrent thread.



std::async
-----------

The template function async runs the function f asynchronously (potentially in a separate thread which may be part of a thread pool) and returns a std::future that will eventually hold the result of that function call.

* proto type 1

.. code-block::c++

	template< class Function, class... Args>

	std::future<std::result_of_t<std::decay_t<Function>(std::decay_t<Args>...)>>
	    async( Function&& f, Args&&... args );

* proto type 2

.. code-block::c++

	template< class Function, class... Args >

	std::future<std::result_of_t<std::decay_t<Function>(std::decay_t<Args>...)>>
	    async( std::launch policy, Function&& f, Args&&... args );


**lanch policy**

	* std::launch::async
	* std::launch::deferred

**return value**

	std::future referring to the shared state created by this call to std::async

**Example**

.. code-block:: c++ 

    std::future<void> handler = std::async(std::launch::deferred, [&](){asyncFunction(a);});


**std::launch::deferred** -- enable lazy evaluation

**std::launch::deferred** -- enable asynchronous evaluation


**note**

You have to add **-lpthread** while compile above example.


std::future
------------

