# Java Advanced Module-B: Executor framework and Fork Join pool

## 1. Executor Framework in Java

The **Executor Framework** in **Java** is part of the **java.util.concurrent** package, introduced in **Java 5**

It helps in managing threads efficiently through a set of high-level APIs for running tasks asynchronously

The framework simplifies task execution by abstracting thread creation, life cycle management, and other complexities of threading

Let's dive into some of the key components and see simple examples for each:

**Key Components**

**Executor**: This is the basic interface that supports launching new tasks

**ExecutorService**: A subinterface of Executor, it provides lifecycle methods to manage termination and methods producing Futures for tracking progress of tasks

**ScheduledExecutorService**: An interface that can schedule tasks to run after a delay or periodically

**Thread Pool**: A group of pre-instantiated reusable threads. Classes like ThreadPoolExecutor and Executors provide factory methods to create standardized thread pools

**Simple Examples**

**Example 1: Using ExecutorService**

This example uses an ExecutorService to run a simple task asynchronously.

```java
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;

public class SimpleExecutorServiceExample {
    public static void main(String[] args) {
        // Creating an ExecutorService with a fixed thread pool of 2 threads.
        ExecutorService executor = Executors.newFixedThreadPool(2);

        // Runnable task that prints the current thread name
        Runnable task = () -> {
            System.out.println("Current thread: " + Thread.currentThread().getName());
        };

        // Submit the task to the executor
        executor.execute(task);

        // Shutdown the executor
        executor.shutdown();
    }
}
```

**Example 2: Using ScheduledExecutorService**

This example demonstrates **scheduling a task with a delay**

```java
import java.util.concurrent.Executors;
import java.util.concurrent.ScheduledExecutorService;
import java.android.util.concurrent.TimeUnit;

public class ScheduledExecutorExample {
    public static void main(String[] args) {
        // Creating a ScheduledExecutorService
        ScheduledExecutorService scheduler = Executors.newScheduledThreadPool(1);

        // Runnable task that prints the current date
        Runnable task = () -> System.out.println("Executing at: " + new Date());

        // Schedule the task with a 5-second delay
        scheduler.schedule(task, 5, TimeUnit.SECONDS);

        // Shutdown the executor after some time
        scheduler.schedule(() -> scheduler.shutdown(), 10, TimeUnit.SECONDS);
    }
}
```

**Example 3: Future and Callable**

This example demonstrates how to **use a Callable task that returns a result**

The **Future** object can be used to retrieve this result

```java
import java.util.concurrent.Callable;
import java.util.concurrent.ExecutionException;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;
import java.util.concurrent.Future;

public class CallableAndFutureExample {
    public static void main(String[] args) throws InterruptedException, ExecutionException {
        // Create an ExecutorService
        ExecutorService executor = Executors.newCachedThreadPool();

        // Callable task that returns the current thread name
        Callable<String> task = () -> {
            return "Executed by: " + Thread.currentThread().getName();
        };

        // Submit the callable task
        Future<String> future = executor.submit(task);

        // Get the result of the callable
        String result = future.get();  // This line will block until the result is available
        System.out.println(result);

        // Shutdown the executor
        executor.shutdown();
    }
}
```

These examples illustrate basic usage of the Executor Framework in Java

They demonstrate how to set up an executor, run tasks asynchronously, schedule tasks, and retrieve results from tasks that compute values

This framework is highly useful for handling complex multithreading scenarios with relative ease

I can provide more examples to help you understand different features of the Java Executor Framework

Let's explore some practical use cases and advanced features of the framework, including handling multiple futures, combining results, and handling exceptions

**Example 4: Handling Multiple Futures**

This example demonstrates how to **manage a list of Future** objects, allowing you to **execute multiple tasks** and **process their results** as they become available

```java
import java.util.ArrayList;
import java.util.List;
import java.util.concurrent.Callable;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;
import java.util.concurrent.Future;

public class MultipleFuturesExample {
    public static void main(String[] args) throws Exception {
        ExecutorService executor = Executors.newFixedThreadPool(4);
        List<Future<String>> futures = new ArrayList<>();

        for (int i = 0; i < 10; i++) {
            int taskId = i;
            Callable<String> task = () -> "Task " + taskId + " executed";
            futures.add(executor.submit(task));
        }

        for (Future<String> future : futures) {
            System.out.println(future.get());  // Waits for the task to complete and prints the result
        }

        executor.shutdown();
    }
}
```

**Example 5: Combine Results from Callables**

This example shows how to **combine results from several Callable tasks** that return values, such as calculating the **sum of returned integers**

```java
import java.util.ArrayList;
import java.util.List;
import java.util.concurrent.Callable;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;
import java.util.concurrent.Future;

public class CombineResultsExample {
    public static void main(String[] args) throws Exception {
        ExecutorService executor = Executors.newCachedThreadPool();
        List<Future<Integer>> futures = new ArrayList<>();

        for (int i = 0; i < 5; i++) {
            int value = i * 10;
            Callable<Integer> task = () -> {
                Thread.sleep(100); // Simulate some computation
                return value + 10;
            };
            futures.add(executor.submit(task));
        }

        int sum = 0;
        for (Future<Integer> future : futures) {
            sum += future.get();
        }

        System.out.println("Total sum: " + sum);

        executor.shutdown();
    }
}
```

**Example 6: Handling Exceptions in Callable Tasks**

This example illustrates how to **handle exceptions** that occur within Callable tasks

It demonstrates proper **error handling** without terminating the program abruptly

```java
import java.util.concurrent.Callable;
import java.util.concurrent.ExecutionException;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;
import java.util.concurrent.Future;

public class ExceptionHandlingExample {
    public static void main(String[] args) {
        ExecutorService executor = Executors.newSingleThreadExecutor();
        Callable<Integer> task = () -> {
            throw new IllegalStateException("I throw an exception!");
        };

        Future<Integer> future = executor.submit(task);

        try {
            future.get();
        } catch (ExecutionException ee) {
            System.err.println("Task threw an exception!");
            System.err.println(ee.getCause());
        } catch (InterruptedException ie) {
            Thread.currentThread().interrupt();  // set the interrupt flag
            System.err.println("Task was interrupted!");
        }

        executor.shutdown();
    }
}
```

**Example 7: Scheduled Task with Fixed Rate**

This example uses **ScheduledExecutorService** to **schedule a periodic task that executes repeatedly at fixed intervals**, regardless of the task's execution duration

```java
import java.util.concurrent.Executors;
import java.util.concurrent.ScheduledExecutorService;
import java.util.concurrent.TimeUnit;

public class ScheduledTaskFixedRateExample {
    public static void main(String[] args) {
        ScheduledExecutorService scheduler = Executors.newScheduledThreadPool(1);

        Runnable task = () -> {
            System.out.println("Executing Task at " + System.nanoTime());
        };

        // Initialize Delay - 0, Period - 2 seconds
        scheduler.scheduleAtFixedRate(task, 0, 2, TimeUnit.SECONDS);

        // Scheduler will keep running. To stop, you might use:
        // scheduler.shutdown();
    }
}
```

These examples demonstrate how to use the Executor Framework effectively to handle multiple tasks, manage asynchronous computation, and deal with exceptions in Java, providing a robust foundation for concurrent programming in Java applications.

## 2. Fork/Join Framework in Java

The Fork/Join Framework is a part of Java's java.util.concurrent package, specifically designed to help leverage multi-core processors effectively

It is ideal for tasks that can be broken down into smaller, independent subtasks, which can be executed concurrently, and then combined to produce a final result

This is achieved using a divide-and-conquer approach

**Key Components**

**ForkJoinPool**: The heart of the Fork/Join Framework. It is an implementation of ExecutorService that manages worker threads

**RecursiveAction**: A recursive resultless ForkJoinTask (typically for side effects, such as array sorting)

**RecursiveTask**: A recursive result-bearing ForkJoinTask

**Principles of the Fork/Join Framework**

Work-Stealing Algorithm: Worker threads that run out of tasks can "steal" tasks from other threads that are still busy

Task Splitting/Forking: Large tasks are split (forked) into smaller tasks until the task size is small enough to be executed directly

Joining Tasks: Results of subtasks are joined together to form the final result

**Simple Examples**

**Example 1: Using RecursiveAction**

Here's an example using **RecursiveAction** for performing a simple task, like **incrementing all elements of an array**

```java
import java.util.concurrent.ForkJoinPool;
import java.util.concurrent.RecursiveAction;

public class SimpleRecursiveActionExample extends RecursiveAction {
    private long[] array;
    private int low;
    private int high;
    private static final int THRESHOLD = 1000; // This threshold can be adjusted based on the task size and system

    public SimpleRecursiveActionExample(long[] array, int low, int high) {
        this.array = array;
        this.low = low;
        this.high = high;
    }

    @Override
    protected void compute() {
        if (high - low < THRESHOLD) {
            for (int i = low; i < high; ++i) {
                array[i]++; // Incrementing each element by one
            }
        } else {
            int mid = (low + high) >>> 1;
            SimpleRecursiveActionExample left = new SimpleRecursiveActive(array, low, mid);
            SimpleRecursiveActionExample right = new SimpleRecursiveActive(array, mid, high);
            invokeAll(left, right);
        }
    }

    public static void main(String[] args) {
        long[] array = new long[2000];
        ForkJoinPool pool = new ForkJoinPool();
        SimpleRecursiveActionExample task = new SimpleRecursiveActionExample(array, 0, array.length);
        pool.invoke(task);
    }
}
```

**Example 2: Using RecursiveTask**

Here's an example using **RecursiveTask** to compute a value, such as finding the **sum of an array**

```java
import java.util.concurrent.ForkJoinPool;
import java.util.concurrent.RecursiveTask;

public class SimpleRecursiveTaskExample extends RecursiveTask<Long> {
    private long[] array;
    private int low;
    private int high;
    private static final int THRESHOLD = 1000;

    public SimpleRecursiveTaskExample(long[] array, int low, int high) {
        this.array = array;
        this.low = low;
        this.high = high;
    }

    @Override
    protected Long compute() {
        if (high - low < THRESHOLD) {
            long sum = 0;
            for (int i = low; i < high; ++i) {
                sum += array[i];
            }
            return sum;
        } else {
            int mid = (low + high) >>> 1;
            SimpleRecursiveTaskExample left = new SimpleRecursiveTaskExample(array, low, mid);
            SimpleRecursiveTaskExample right = new SimpleRecursiveTaskExample(array, mid, high);
            left.fork();
            long rightResult = right.compute();
            long leftResult = left.join();
            return leftResult + rightResult;
        }
    }

    public static void main(String[] args) {
        long[] array = new long[4000]; // large array initialization with values
        ForkJoinPool pool = new ForkJoinPool();
        SimpleRecursiveTaskExample task = new SimpleRecursiveTaskExample(array, 0, array.length);
        long sum = pool.invoke(task);
        System.out.println("Sum: " + sum);
    }
}
```

**Example 3: RecursiveTask for Parallel Array Search**

This example demonstrates how to use **RecursiveTask** to perform a parallel **search in an array to find the maximum value**

```java
import java.util.concurrent.ForkJoinPool;
import java.util.concurrent.RecursiveTask;

public class ParallelMaxFinder extends RecursiveTask<Integer> {
    private static final int THRESHOLD = 1000;
    private final int[] array;
    private final int start;
    private final int end;

    public ParallelMaxFinder(int[] array, int start, int end) {
        this.array = array;
        this.start = start;
        this.end = end;
    }

    @Override
    protected Integer compute() {
        if (end - start < THRESHOLD) {
            int max = array[start];
            for (int i = start + 1; i < end; i++) {
                if (array[i] > max) {
                    max = array[i];
                }
            }
            return max;
        } else {
            int mid = (start + end) / 2;
            ParallelMaxFinder leftTask = new ParallelMaxFinder(array, start, mid);
            ParallelMaxFinder rightTask = new ParallelMaxFinder(array, mid, end);
            leftTask.fork();
            int rightResult = rightTask.compute();
            int leftResult = leftTask.join();
            return Math.max(leftResult, rightResult);
        }
    }

    public static void main(String[] args) {
        int[] array = new int[10000];
        // Populate the array with random values
        for (int i = 0; i < array.length; i++) {
            array[i] = (int) (Math.random() * 10000);
        }

        ForkJoinPool pool = new ForkJoinPool();
        ParallelMaxFinder task = new ParallelMaxFinder(array, 0, array.length);
        int max = pool.invoke(task);
        System.out.println("Max value: " + max);
    }
}
```

**Example 4: RecursiveAction for Matrix Multiplication**

This example demonstrates how to use RecursiveAction for parallel matrix multiplication

java
import java.util.concurrent.ForkJoinPool;
import java.util.concurrent.RecursiveAction;

public class ParallelMatrixMultiplication extends RecursiveAction {
    private static final int THRESHOLD = 64;
    private final double[][] A;
    private final double[][] B;
    private final double[][] C;
    private final int row;
    private final int col;
    private final int size;

    public ParallelMatrixMultiplication(double[][] A, double[][] B, double[][] C, int row, int col, int size) {
        this.A = A;
        this.B = B;
        this.C = C;
        this.row = row;
        this.col = col;
        this.size = size;
    }

    @Override
    protected void compute() {
        if (size <= THRESHOLD) {
            for (int i = 0; i < size; i++) {
                for (int j = 0; j < size; j++) {
                    for (int k = 0; k < size; k++) {
                        C[row + i][col + j] += A[row + i][k] * B[k][col + j];
                    }
                }
            }
        } else {
            int newSize = size / 2;
            invokeAll(
                new ParallelMatrixMultiplication(A, B, C, row, col, newSize),
                new ParallelMatrixMultiplication(A, B, C, row, col + newSize, newSize),
                new ParallelMatrixMultiplication(A, B, C, row + newSize, col, newSize),
                new ParallelMatrixMultiplication(A, B, C, row + newSize, col + newSize, newSize)
            );
        }
    }

    public static void main(String[] args) {
        int n = 512; // Matrix size (must be a power of 2)
        double[][] A = new double[n][n];
        double[][] B = new double[n][n];
        double[][] C = new double[n][n];

        // Initialize matrices A and B with random values
        for (int i = 0; i < n; i++) {
            for (int j = 0; j < n; j++) {
                A[i][j] = Math.random();
                B[i][j] = Math.random();
            }
        }

        ForkJoinPool pool = new ForkJoinPool();
        ParallelMatrixMultiplication task = new ParallelMatrixMultiplication(A, B, C, 0, 0, n);
        pool.invoke(task);

        // Print the result matrix C
        for (int i = 0; i < n; i++) {
            for (int j = 0; j < n; j++) {
                System.out.printf("%8.2f ", C[i][j]);
            }
            System.out.println();
        }
    }
}
```

**Example 5: RecursiveTask for Parallel Merge Sort**

This example demonstrates how to use RecursiveTask for implementing parallel merge sort

```java
import java.util.concurrent.ForkJoinPool;
import java.util.concurrent.RecursiveTask;
import java.util.Arrays;

public class ParallelMergeSort extends RecursiveTask<int[]> {
    private static final int THRESHOLD = 1000;
    private final int[] array;

    public ParallelMergeSort(int[] array) {
        this.array = array;
    }

    @Override
    protected int[] compute() {
        if (array.length < THRESHOLD) {
            Arrays.sort(array);
            return array;
        } else {
            int mid = array.length / 2;
            ParallelMergeSort leftTask = new ParallelMergeSort(Arrays.copyOfRange(array, 0, mid));
            ParallelMergeSort rightTask = new ParallelMergeSort(Arrays.copyOfRange(array, mid, array.length));
            invokeAll(leftTask, rightTask);
            int[] leftResult = leftTask.join();
            int[] rightResult = rightTask.join();
            return merge(leftResult, rightResult);
        }
    }

    private int[] merge(int[] left, int[] right) {
        int[] result = new int[left.length + right.length];
        int i = 0, j = 0, k = 0;
        while (i < left.length && j < right.length) {
            if (left[i] <= right[j]) {
                result[k++] = left[i++];
            } else {
                result[k++] = right[j++];
            }
        }
        while (i < left.length) {
            result[k++] = left[i++];
        }
        while (j < right.length) {
            result[k++] = right[j++];
        }
        return result;
    }

    public static void main(String[] args) {
        int[] array = new int[10000];
        // Populate the array with random values
        for (int i = 0; i < array.length; i++) {
            array[i] = (int) (Math.random() * 10000);
        }

        ForkJoinPool pool = new ForkJoinPool();
        ParallelMergeSort task = new ParallelMergeSort(array);
        int[] sortedArray = pool.invoke(task);

        // Print the sorted array
        for (int value : sortedArray) {
            System.out.print(value + " ");
        }
    }
}
```

**Example 6: Using Work-Stealing Algorithm**

This example demonstrates the work-stealing algorithm by simulating a simple workload distribution scenario

```java
import java.util.concurrent.ForkJoinPool;
import java.util.concurrent.RecursiveAction;
import java.util.concurrent.TimeUnit;

public class WorkStealingDemo extends RecursiveAction {
    private static final int THRESHOLD = 10;
    private final int[] array;
    private final int start;
    private final int end;

    public WorkStealingDemo(int[] array, int start, int end) {
        this.array = array;
        this.start = start;
        this.end = end;
    }

    @Override
    protected void compute() {
        if (end - start < THRESHOLD) {
            for (int i = start; i < end; i++) {
                array[i] = process(array[i]);
            }
        } else {
            int mid = (start + end) / 2;
            WorkStealingDemo leftTask = new WorkStealingDemo(array, start, mid);
            WorkStealingDemo rightTask = new WorkStealingDemo(array, mid, end);
            invokeAll(leftTask, rightTask);
        }
    }

    private int process(int value) {
        try {
            TimeUnit.MILLISECONDS.sleep(value);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        return value * 2;
    }

    public static void main(String[] args) {
        int[] array = new int[100];
        // Populate the array with values ranging from 1 to 100
        for (int i = 0; i < array.length; i++) {
            array[i] = i + 1;
        }

        ForkJoinPool pool = new ForkJoinPool();
        WorkStealingDemo task = new WorkStealingDemo(array, 0, array.length);
        pool.invoke(task);

        // Print the processed array
        for (int value : array) {
            System.out.print(value + " ");
        }
    }
}
```

These examples illustrate the versatility of the Fork/Join Framework, showing how it can be used for parallel search, matrix multiplication, merge sort, and even demonstrating the work-stealing algorithm. By leveraging these capabilities, developers can efficiently utilize multi-core processors for complex and large-scale tasks.

These examples demonstrate the basic usage of the Fork/Join Framework, which is very effective for tasks that can be broken down recursively

By understanding how to create and manage tasks with RecursiveAction and RecursiveTask, developers can effectively utilize multicore processors to enhance performance for suitable tasks
