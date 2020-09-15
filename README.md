
- [Guide to PriorityBlockingQueue in Java](http://www.baeldung.com/java-priority-blocking-queue)

---------------------------------------------------------------------------------------------------
1. Introduction
---------------------------------------------------------------------------------------------------
In this article, we'll focus on the PriorityBlockingQueue class and go over some practical examples.

Starting with the assumption that we already know what a Queue is, we will first demonstrate how elements in the PriorityBlockingQueue are ordered by priority.

Following this, we will demonstrate how this type of queue can be used to block a thread.

Finally, we will show how using these two features together can be useful when processing data across multiple threads.

---------------------------------------------------------------------------------------------------
2. Priority of Elements
---------------------------------------------------------------------------------------------------
Unlike a standard queue, you can't just add any type of element to a PriorityBlockingQueue. There are two options:

	Adding elements which implement Comparable
	Adding elements which do not implement Comparable, on the condition that you provide a Comparator as well

By using either the Comparator or the Comparable implementations to compare elements, the PriorityBlockingQueue will always be sorted.

The aim is to implement comparison logic in a way in which the highest priority element is always ordered first. Then, when we remove an element from our queue, it will always be the one with the highest priority.

To begin with, let's make use of our queue in a single thread, as opposed to using it across multiple ones. By doing this, it makes it easy to prove how elements are ordered in a unit test:
	 
	PriorityBlockingQueue<Integer> queue = new PriorityBlockingQueue<>();
	ArrayList<Integer> polledElements = new ArrayList<>();
	  
	queue.add(1);
	queue.add(5);
	queue.add(2);
	queue.add(3);
	queue.add(4);
	 
	queue.drainTo(polledElements);	 
	assertThat(polledElements).containsExactly(1, 2, 3, 4, 5);

As we can see, despite adding the elements to the queue in a random order, they will be ordered when we start polling them. This is because the Integer class implements Comparable, which will, in turn, be used to make sure we take them out from the queue in ascending order.

It's also worth noting that when two elements are compared and are the same, there's no guarantee of how they will be ordered.

---------------------------------------------------------------------------------------------------
3. Using the Queue to Block
---------------------------------------------------------------------------------------------------
If we were dealing with a standard queue, we would call poll() to retrieve elements. However, if the queue was empty, a call to poll() would return null.

The PriorityBlockingQueue implements the BlockingQueue interface, which gives us some extra methods that allow us to block when removing from an empty queue. Let's try using the take() method, which should do exactly that:
	 
	PriorityBlockingQueue<Integer> queue = new PriorityBlockingQueue<>();
	 
	new Thread(() -> {
	  System.out.println("Polling...");
	 
	  try {
		  Integer poll = queue.take();
		  System.out.println("Polled: " + poll);
	  } catch (InterruptedException e) {
		  e.printStackTrace();
	  }
	}).start();
	 
	Thread.sleep(TimeUnit.SECONDS.toMillis(5));
	System.out.println("Adding to queue");
	queue.add(1);

Although using sleep() is a slightly brittle way of demonstrating things, when we run this code we will see:
	 
	Polling...
	Adding to queue
	Polled: 1

This proves that take() blocked until an item was added:

	The thread will print “Polling” to prove that it's started
	The test will then pause for around five seconds, to prove the thread must have called take() by this point
	We add to the queue, and should more or less instantly see “Polled: 1” to prove that take() returned an element as soon as it become available

It's also worth mentioning that the BlockingQueue interface also provides us with ways of blocking when adding to full queues.

However, a PriorityBlockingQueue is unbounded. This means that it will never be full, thus it will always possible to add new elements.

---------------------------------------------------------------------------------------------------
4. Using Blocking and Prioritization Together
---------------------------------------------------------------------------------------------------
Now that we've explained the two key concepts of a PriorityBlockingQueue, let's use them both together. We can simply expand on our previous example, but this time add more elements to the queue:
	 
	Thread thread = new Thread(() -> {
		System.out.println("Polling...");
		while (true) {
			try {
				Integer poll = queue.take();
				System.out.println("Polled: " + poll);
			} 
			catch (InterruptedException e) { 
				e.printStackTrace();
			}
		}
	});
	 
	thread.start();
	 
	Thread.sleep(TimeUnit.SECONDS.toMillis(5));
	System.out.println("Adding to queue");
	 
	queue.addAll(newArrayList(1, 5, 6, 1, 2, 6, 7));
	Thread.sleep(TimeUnit.SECONDS.toMillis(1));

Again, while this is a little brittle because of the use of sleep(), it still shows us a valid use case. We now have a queue which blocks, waiting for elements to be added. We're then adding lots of elements at once, and then showing that they will be handled in priority order. The output will look like this:
	 
	Polling...
	Adding to queue
	Polled: 1
	Polled: 1
	Polled: 2
	Polled: 5
	Polled: 6
	Polled: 6
	Polled: 7

---------------------------------------------------------------------------------------------------
5. Conclusion
---------------------------------------------------------------------------------------------------
In this guide, we've demonstrated how we can use a PriorityBlockingQueue in order to block a thread until some items have been added to it, and also that we are able to process those items based on their priority.

The implementation of these examples can be found over on GitHub. This is a Maven-based project, so should be easy to run as is.

---------------------------------------------------------------------------------------------------
