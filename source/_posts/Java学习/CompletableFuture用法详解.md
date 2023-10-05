---
title: CompletableFuture用法详解
date: 2023-09-13 15:24:37
mathjax: true
categories: Java学习
tags: 
- Java
- 多线程
---

# 1 实例化CompletableFuture

<!--more-->

我们可以直接调用其构造方法直接创建一个`CompletableFuture`实例对象：

```java
/**
 * Creates a new incomplete CompletableFuture.
 */
public CompletableFuture() {
}
```

或者可以通过调用`CompletableFuture.completedFuture`静态方法，使用给定值来创建一个已经计算完成的`CompletableFuture`实例对象：

```java
/**
 * Returns a new CompletableFuture that is already completed with
 * the given value.
 *
 * @param value the value
 * @param <U> the type of the value
 * @return the completed CompletableFuture
 */
public static <U> CompletableFuture<U> completedFuture(U value);
```

另外，可以使用以下静态方法为**异步任务**创建`CompletableFuture`实例对象：

```java
/**
 * Returns a new CompletableFuture that is asynchronously completed
 * by a task running in the {@link ForkJoinPool#commonPool()} after
 * it runs the given action.
 *
 * @param runnable the action to run before completing the
 * returned CompletableFuture
 * @return the new CompletableFuture
 */
public static CompletableFuture<Void> runAsync(Runnable runnable);

/**
 * Returns a new CompletableFuture that is asynchronously completed
 * by a task running in the given executor after it runs the given
 * action.
 *
 * @param runnable the action to run before completing the
 * returned CompletableFuture
 * @param executor the executor to use for asynchronous execution
 * @return the new CompletableFuture
 */
public static CompletableFuture<Void> runAsync(Runnable runnable, Executor executor);

/**
 * Returns a new CompletableFuture that is asynchronously completed
 * by a task running in the {@link ForkJoinPool#commonPool()} with
 * the value obtained by calling the given Supplier.
 *
 * @param supplier a function returning the value to be used
 * to complete the returned CompletableFuture
 * @param <U> the function's return type
 * @return the new CompletableFuture
 */
public static <U> CompletableFuture<U> supplyAsync(Supplier<U> supplier);

/**
 * Returns a new CompletableFuture that is asynchronously completed
 * by a task running in the given executor with the value obtained
 * by calling the given Supplier.
 *
 * @param supplier a function returning the value to be used
 * to complete the returned CompletableFuture
 * @param executor the executor to use for asynchronous execution
 * @param <U> the function's return type
 * @return the new CompletableFuture
 */
public static <U> CompletableFuture<U> supplyAsync(Supplier<U> supplier, Executor executor);
```

上述两种方法的主要区别在于：

- `runAsync`方法：参数为`Runnable`类型，仅执行异步任务，无返回结果。
- `supplyAsync`方法：参数为`Supplier`类型，异步任务执行完毕之后，会返回对应的执行结果。

另外，对上面的两种方法，我们可以指定`Executor`参数，`CompletableFuture`会使用我们指定的线程池来执行异步任务。如果不指定的话，默认会使用`ForkJoinPool.commonPool()`这一公共线程池。

# 2 获取任务执行结果

`CompletableFuture`类实现了`Future`接口，因此我们还是可以像以前一样通过阻塞或轮询的方式获取结果，尽管这种方式不推荐使用。对于`CompletableFuture`，主要有以下获取结果的方法：

```java
/**
 * Waits if necessary for this future to complete, and then
 * returns its result.
 *
 * @return the result value
 * @throws CancellationException if this future was cancelled
 * @throws ExecutionException if this future completed exceptionally
 * @throws InterruptedException if the current thread was interrupted
 * while waiting
 */
public T get() throws InterruptedException, ExecutionException;

/**
 * Waits if necessary for at most the given time for this future
 * to complete, and then returns its result, if available.
 *
 * @param timeout the maximum time to wait
 * @param unit the time unit of the timeout argument
 * @return the result value
 * @throws CancellationException if this future was cancelled
 * @throws ExecutionException if this future completed exceptionally
 * @throws InterruptedException if the current thread was interrupted
 * while waiting
 * @throws TimeoutException if the wait timed out
 */
public T get(long timeout, TimeUnit unit) throws InterruptedException, ExecutionException, TimeoutException;

/**
 * Returns the result value (or throws any encountered exception)
 * if completed, else returns the given valueIfAbsent.
 *
 * @param valueIfAbsent the value to return if not completed
 * @return the result value, if completed, else the given valueIfAbsent
 * @throws CancellationException if the computation was cancelled
 * @throws CompletionException if this future completed
 * exceptionally or a completion computation threw an exception
 */
public T getNow(T valueIfAbsent);

/**
 * Returns the result value when complete, or throws an
 * (unchecked) exception if completed exceptionally. To better
 * conform with the use of common functional forms, if a
 * computation involved in the completion of this
 * CompletableFuture threw an exception, this method throws an
 * (unchecked) {@link CompletionException} with the underlying
 * exception as its cause.
 *
 * @return the result value
 * @throws CancellationException if the computation was cancelled
 * @throws CompletionException if this future completed
 * exceptionally or a completion computation threw an exception
 */
public T join();
```

其中，`get`方法为`Future`接口中方法的实现，在此不再多做介绍。下面介绍一下另外两个`CompletableFuture`类中新增的方法：

- `getNow`方法：如果任务已经执行完成，则返回结果或者抛出异常；否则返回给定的`valueIfAbsent`值。
- `join`方法：该方法与不带超时时间的`get`方法类似，都是阻塞获取任务执行结果。不同之处在于`join`方法抛出的是非受检异常`CompletionException`，而`get`方法抛出的则是受检异常。

使用不同方式创建的实例对象，其调用`get`以及`join`方法得到的结果也不尽相同：

- 对于使用`completedFuture`静态方法创建的实例对象，调用`get`或`join`方法可以直接获取到设置的`valud`值，不会发生阻塞（因为该实例对象创建出来之后就已经处于任务完成的状态了）。
- 对于使用`runAsync`或者`supplyAsync`静态方法创建的实例对象，当异步任务执行完成之后，`get`或`join`方法才可以获取到对应的任务执行结果。
- 对于使用构造方法创建的实例对象，如果不执行相关操作，`get`或`join`方法会一直阻塞等待下去（如果不执行相关操作，构造方法创建的实例对象会一直处于任务未完成的状态）。

对于使用构造方法创建的实例对象，我们可以通过调用下面的方法来令其进入任务完成的状态，从而解除`get`或`join`方法的等待：

```java
/**
 * If not already completed, sets the value returned by {@link
 * #get()} and related methods to the given value.
 *
 * @param value the result value
 * @return {@code true} if this invocation caused this CompletableFuture
 * to transition to a completed state, else {@code false}
 */
public boolean complete(T value);

/**
 * If not already completed, causes invocations of {@link #get()}
 * and related methods to throw the given exception.
 *
 * @param ex the exception
 * @return {@code true} if this invocation caused this CompletableFuture
 * to transition to a completed state, else {@code false}
 */
public boolean completeExceptionally(Throwable ex);
```

其中，调用`complete(T value)`方法表示任务正常执行，`get`方法会获取到传递的`value`值。调用`completeExceptionally(Throwable ex)`方法表示任务执行发生异常，`get`方法会抛出传递的`ex`异常。

# 3 任务执行完成时的处理

我们可以为`CompletableFuture`注册回调方法，当任务执行完毕之后即调用该回调方法。我们可以使用以下两类方法来实现回调方法的注册：

- `whenComplete`方法：注册回调方法。当任务执行完成时，将任务执行结果result以及执行异常exception传递到回调方法中。**回调方法无返回结果**。
- `handle()`方法：注册回调方法。当任务执行完成时，将任务执行结果result以及执行异常exception传递到回调方法中。**回调方法有返回结果**。

二者的方法声明如下：

```java
/**
 * Returns a new CompletionStage with the same result or exception as
 * this stage, that executes the given action when this stage completes.
 *
 * <p>When this stage is complete, the given action is invoked with the
 * result (or {@code null} if none) and the exception (or {@code null}
 * if none) of this stage as arguments.  The returned stage is completed
 * when the action returns.  If the supplied action itself encounters an
 * exception, then the returned stage exceptionally completes with this
 * exception unless this stage also completed exceptionally.
 *
 * @param action the action to perform
 * @return the new CompletionStage
 */
public CompletionStage<T> whenComplete(BiConsumer<? super T, ? super Throwable> action);

/**
 * Returns a new CompletionStage that, when this stage completes
 * either normally or exceptionally, is executed with this stage's
 * result and exception as arguments to the supplied function.
 *
 * <p>When this stage is complete, the given function is invoked
 * with the result (or {@code null} if none) and the exception (or
 * {@code null} if none) of this stage as arguments, and the
 * function's result is used to complete the returned stage.
 *
 * @param fn the function to use to compute the value of the
 * returned CompletionStage
 * @param <U> the function's return type
 * @return the new CompletionStage
 */
public <U> CompletionStage<U> handle(BiFunction<? super T, Throwable, ? extends U> fn);
```

其实从方法声明中也可以看出，`whenComplete`方法传递的参数类型为`BiConsumer`，而`handle`方法传递的参数为`BiFunction`，因此前者无返回结果而后者有。

`whenComplete`方法的使用示例如下：

```java
CompletableFuture<String> completableFuture = CompletableFuture.completedFuture("hello");
completableFuture.whenComplete(new BiConsumer<String, Throwable>() {
    @Override
    public void accept(String result, Throwable throwable) {
        System.out.println(result);
    }
});
```

`handle`方法的使用示例如下：

```java
CompletableFuture<String> completableFuture = CompletableFuture.completedFuture("hello");
completableFuture.handle(new BiFunction<String, Throwable, Integer>() {
    @Override
    public Integer apply(String s, Throwable throwable) {
        return s.length();
    }
});
```

需要注意的是，`whenComplete`方法以及`handle`方法有着三种变形，分别是：

- `xxx(callback)`：同步方法，使用当前线程执行`callback`回调。
- `xxxAsync(callback)`：异步方法，使用预设的线程池（`ForkJoinPool.commonPool()`）执行`callback`回调。
- `xxxAsync(callback, executor)`：异步方法，使用指定的`executor`线程池执行`callback`回调。

如果只专注于异常处理的话，可以试下`exceptionally`方法，该方法的声明如下：

```java
/**
 * Returns a new CompletableFuture that is completed when this
 * CompletableFuture completes, with the result of the given
 * function of the exception triggering this CompletableFuture's
 * completion when it completes exceptionally; otherwise, if this
 * CompletableFuture completes normally, then the returned
 * CompletableFuture also completes normally with the same value.
 * Note: More flexible versions of this functionality are
 * available using methods {@code whenComplete} and {@code handle}.
 *
 * @param fn the function to use to compute the value of the
 * returned CompletableFuture if this CompletableFuture completed
 * exceptionally
 * @return the new CompletableFuture
 */
public CompletableFuture<T> exceptionally(Function<Throwable, ? extends T> fn);
```

调用`exceptionally`之后，如果任务执行过程中发生异常，则会执行该方法中注册的异常处理回调方法；如果任务正常执行结束，则会跳过此处注册的回调方法。值得一提的是，`exceptionally`方法能实现的功能，`handle`方法都能实现，`exceptionally`方法的优势在于仅专注于异常处理。

# 4 串联

`CompletableFuture`允许我们将一系列操作串联起来，常用的方法有以下几种：

- `thenRun`方法：对应`Runnable`类型，无入参，无返回值。
- `thenAccept`方法：对应`Consumer`类型，有入参，无返回值。
- `thenApply`方法：对应`Function`类型，有入参，有返回值。
- `thenCompose`方法：对应`Function`类型，有入参，有返回值。

其对应的方法声明分别为：

```java
/**
 * Returns a new CompletionStage that, when this stage completes
 * normally, executes the given action.
 *
 * See the {@link CompletionStage} documentation for rules
 * covering exceptional completion.
 *
 * @param action the action to perform before completing the
 * returned CompletionStage
 * @return the new CompletionStage
 */
public CompletionStage<Void> thenRun(Runnable action);

/**
 * Returns a new CompletionStage that, when this stage completes
 * normally, is executed with this stage's result as the argument
 * to the supplied action.
 *
 * See the {@link CompletionStage} documentation for rules
 * covering exceptional completion.
 *
 * @param action the action to perform before completing the
 * returned CompletionStage
 * @return the new CompletionStage
 */
public CompletionStage<Void> thenAccept(Consumer<? super T> action);

/**
 * Returns a new CompletionStage that, when this stage completes
 * normally, is executed with this stage's result as the argument
 * to the supplied function.
 *
 * See the {@link CompletionStage} documentation for rules
 * covering exceptional completion.
 *
 * @param fn the function to use to compute the value of
 * the returned CompletionStage
 * @param <U> the function's return type
 * @return the new CompletionStage
 */
public <U> CompletionStage<U> thenApply(Function<? super T,? extends U> fn);

/**
 * Returns a new CompletionStage that, when this stage completes
 * normally, is executed with this stage as the argument
 * to the supplied function.
 *
 * See the {@link CompletionStage} documentation for rules
 * covering exceptional completion.
 *
 * @param fn the function returning a new CompletionStage
 * @param <U> the type of the returned CompletionStage's result
 * @return the CompletionStage
 */
public <U> CompletionStage<U> thenCompose(Function<? super T, ? extends CompletionStage<U>> fn);
```

其中，`thenApply`与`thenCompose`的返回值类型相同，都是`CompletionStage<U>`类型。不同之处在于`Function`接口的返回值，`thenApply`中`Funciton`接口的返回值可以为任意值，而`thenCompose`中`Function`接口的返回值只能为`CompletionStage`类型的子类。

总的来说，`thenApply`是获取上一个`CompletionStage`的执行结果，并对该结果进行一定的转换。而`thenCompose`方法则是将上一个`CompletionStage`与现在返回的`CompletionStage`组合成一个新的`CompletionStage`，相当于`CompletionStage`之间的链式调用。

需要注意的是，上述方法同样具有`xxx()`、`xxxAsync(callback)`、`xxxAsync(callback, executor)`等三个版本。最后需要强调一点，上面讲的串联是多个操作之间串行执行，只有在前一个操作执行完毕之后，后一个操作才会执行。

# 5 并联

类似于串联，`CompletableFuture`也可以实现串联操作，即多个操作并行执行。可使用以下方法实现`CompletableFuture`之间的**两两并联**：

- `runAfterBoth`：当前`CompletionStage`与给定`ComplationStage`均正常执行完毕之后，执行指定动作。对应`Runnable`类型，无入参，无返回值。
- `runAfterEither`：当前`CompletionStage`与给定`ComplationStage`有一个正常执行完毕之后，执行指定动作。对应`Runnable`类型，无入参，无返回值。
- `thenAcceptBoth`：当前`CompletionStage`与给定`ComplationStage`均正常执行完毕之后，执行指定动作。对应`Consumer`类型，有入参，无返回值。
- `acceptEither`：当前`CompletionStage`与给定`ComplationStage`有一个正常执行完毕之后，执行指定动作。对应`Consumer`类型，有入参，无返回值。
- `thenCombine`：当前`CompletionStage`与给定`ComplationStage`均正常执行完毕之后，执行指定动作。对应`Function`类型，有入参，有返回值。
- `applyToEither`：当前`CompletionStage`与给定`ComplationStage`有一个正常执行完毕之后，执行指定动作。对应`Function`类型，有入参，有返回值。

```java
/**
 * Returns a new CompletionStage that, when this and the other
 * given stage both complete normally, executes the given action.
 *
 * See the {@link CompletionStage} documentation for rules
 * covering exceptional completion.
 *
 * @param other the other CompletionStage
 * @param action the action to perform before completing the
 * returned CompletionStage
 * @return the new CompletionStage
 */
public CompletionStage<Void> runAfterBoth(CompletionStage<?> other, Runnable action);

/**
 * Returns a new CompletionStage that, when either this or the
 * other given stage complete normally, executes the given action.
 *
 * See the {@link CompletionStage} documentation for rules
 * covering exceptional completion.
 *
 * @param other the other CompletionStage
 * @param action the action to perform before completing the
 * returned CompletionStage
 * @return the new CompletionStage
 */
public CompletionStage<Void> runAfterEither(CompletionStage<?> other, Runnable action);

/**
 * Returns a new CompletionStage that, when this and the other
 * given stage both complete normally, is executed with the two
 * results as arguments to the supplied action.
 *
 * See the {@link CompletionStage} documentation for rules
 * covering exceptional completion.
 *
 * @param other the other CompletionStage
 * @param action the action to perform before completing the
 * returned CompletionStage
 * @param <U> the type of the other CompletionStage's result
 * @return the new CompletionStage
 */
public <U> CompletionStage<Void> thenAcceptBoth(CompletionStage<? extends U> other, BiConsumer<? super T, ? super U> action);

/**
 * Returns a new CompletionStage that, when either this or the
 * other given stage complete normally, is executed with the
 * corresponding result as argument to the supplied action.
 *
 * See the {@link CompletionStage} documentation for rules
 * covering exceptional completion.
 *
 * @param other the other CompletionStage
 * @param action the action to perform before completing the
 * returned CompletionStage
 * @return the new CompletionStage
 */
public CompletionStage<Void> acceptEither(CompletionStage<? extends T> other, Consumer<? super T> action);

/**
 * Returns a new CompletionStage that, when either this or the
 * other given stage complete normally, is executed with the
 * corresponding result as argument to the supplied function.
 *
 * See the {@link CompletionStage} documentation for rules
 * covering exceptional completion.
 *
 * @param other the other CompletionStage
 * @param fn the function to use to compute the value of
 * the returned CompletionStage
 * @param <U> the function's return type
 * @return the new CompletionStage
 */
public <U> CompletionStage<U> applyToEither(CompletionStage<? extends T> other, Function<? super T, U> fn);

/**
 * Returns a new CompletionStage that, when this and the other
 * given stage both complete normally, is executed with the two
 * results as arguments to the supplied function.
 *
 * See the {@link CompletionStage} documentation for rules
 * covering exceptional completion.
 *
 * @param other the other CompletionStage
 * @param fn the function to use to compute the value of
 * the returned CompletionStage
 * @param <U> the type of the other CompletionStage's result
 * @param <V> the function's return type
 * @return the new CompletionStage
 */
public <U,V> CompletionStage<V> thenCombine(CompletionStage<? extends U> other, BiFunction<? super T,? super U,? extends V> fn);
```

需要注意的是，上述方法同样具有`xxx()`、`xxxAsync(callback)`、`xxxAsync(callback, executor)`等三个版本。

除了上述提到的两两并联的方法之外，`CompletableFuture`还提供了一些静态方法来实现多个future之间的并联：

- `allOf`方法：返回一个`CompletableFuture`，其中所有并联的future全部完成之后，该future才算完成。
- `anyOf`方法：返回一个`CompletableFuture`，其中任何一个并联的future完成之后，该future就算完成。

上述方法的声明如下：

```java
/**
 * Returns a new CompletableFuture that is completed when all of
 * the given CompletableFutures complete.  If any of the given
 * CompletableFutures complete exceptionally, then the returned
 * CompletableFuture also does so, with a CompletionException
 * holding this exception as its cause.  Otherwise, the results,
 * if any, of the given CompletableFutures are not reflected in
 * the returned CompletableFuture, but may be obtained by
 * inspecting them individually. If no CompletableFutures are
 * provided, returns a CompletableFuture completed with the value
 * {@code null}.
 *
 * <p>Among the applications of this method is to await completion
 * of a set of independent CompletableFutures before continuing a
 * program, as in: {@code CompletableFuture.allOf(c1, c2,
 * c3).join();}.
 *
 * @param cfs the CompletableFutures
 * @return a new CompletableFuture that is completed when all of the
 * given CompletableFutures complete
 * @throws NullPointerException if the array or any of its elements are
 * {@code null}
 */
public static CompletableFuture<Void> allOf(CompletableFuture<?>... cfs);

/**
 * Returns a new CompletableFuture that is completed when any of
 * the given CompletableFutures complete, with the same result.
 * Otherwise, if it completed exceptionally, the returned
 * CompletableFuture also does so, with a CompletionException
 * holding this exception as its cause.  If no CompletableFutures
 * are provided, returns an incomplete CompletableFuture.
 *
 * @param cfs the CompletableFutures
 * @return a new CompletableFuture that is completed with the
 * result or exception of any of the given CompletableFutures when
 * one completes
 * @throws NullPointerException if the array or any of its elements are
 * {@code null}
 */
public static CompletableFuture<Object> anyOf(CompletableFuture<?>... cfs);
```

参考资料：

[CompletableFuture](https://popcornylu.gitbooks.io/java_multithread/content/async/cfuture.html)

[CompletableFuture用法详解](https://zhuanlan.zhihu.com/p/344431341)

[Java CompletableFuture 详解](https://colobu.com/2016/02/29/Java-CompletableFuture/)

[CompletableFuture使用大全，简单易懂](https://juejin.cn/post/6844904195162636295)

