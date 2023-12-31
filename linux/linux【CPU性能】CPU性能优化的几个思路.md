


---------
## 1. 性能优化方法论

 - 首先，既然要做性能优化，那要怎么判断它是不是有效呢？特别是优化后，到底能提升多少性能呢？
 - 第二，性能问题通常不是独立的，如果有多个性能问题同时发生，你应该先优化哪一个呢？
 - 第三，提升性能的方法并不是唯一的，当有多种方法可以选择时，你会选用哪一种呢？是不是总选那个最大程度提升性能的方法就行了呢？

如果你可以轻松回答这三个问题，那么二话不说就可以开始优化。

如，在前面的不可中断进程案例中，通过性能分析，我们发现是因为一个进程的直接 I/O  ，导致了 iowait 高达 90%。那是不是用“直接 I/O 换成缓存 I/O”的方法，就可以立即优化了呢？按照上面讲的，你可以先自己思考下那三点。如果不能确定，我们一起来看看。

 - 第一个问题，直接 I/O 换成缓存 I/O，可以把 iowait 从 90% 降到接近 0，性能提升很明显。
 - 第二个问题，我们没有发现其他性能问题，直接 I/O 是唯一的性能瓶颈，所以不用挑选优化对象。
 - 第三个问题，缓存 I/O 是我们目前用到的最简单的优化方法，而且这样优化并不会影响应用的功能。

好的，这三个问题很容易就能回答，所以立即优化没有任何问题。但是，很多现实情况，并不像我举的例子那么简单。性能评估可能有多重指标，性能问题可能会多个同时发生，而且，优化某一个指标的性能，可能又导致其他指标性能的下降。那么，面对这种复杂的情况，我们该怎么办呢？接下来，我们就来深入分析这三个问题。


##  2. 怎么评估性能优化的效果？

首先，来看第一个问题，怎么评估性能优化的效果。

我们解决性能问题的目的，自然是想得到一个性能提升的效果。为了评估这个效果，我们需要对系统的性能指标进行量化，并且要分别测试出优化前、后的性能指标，用前后指标的变化来对比呈现效果。我把这个方法叫做性能评估“三步走”。

 1. 确定性能的量化指标。
 2. 测试优化前的性能指标。
 3. 测试优化后的性能指标。

先看第一步，性能的量化指标有很多，比如 CPU 使用率、应用程序的吞吐量、客户端请求的延迟等，都可以评估性能。那我们应该选择什么指标来评估呢？

我的建议是不要局限在单一维度的指标上，你至少要从应用程序和系统资源这两个维度，分别选择不同的指标。比如，以 Web 应用为例：

 - 应用程序的维度，我们可以用吞吐量和请求延迟来评估应用程序的性能。
 - 系统资源的维度，我们可以用 CPU 使用率来评估系统的 CPU 使用情况。


之所以从这两个不同维度选择指标，主要是因为应用程序和系统资源这两者间相辅相成的关系。

 - **好的应用程序是性能优化的最终目的和结果，系统优化总是为应用程序服务的。所以，必须要使用应用程序的指标，来评估性能优化的整体效果**。
 - **系统资源的使用情况是影响应用程序性能的根源。所以，需要用系统资源的指标，来观察和分析瓶颈的来源**


至于接下来的两个步骤，主要是为了对比优化前后的性能，更直观地呈现效果。如果你的第一步，是从两个不同维度选择了多个指标，那么在性能测试时，你就需要获得这些指标的具体数值。还是以刚刚的 Web 应用为例，对应上面提到的几个指标，我们可以选择 `ab 等工具，测试 Web 应用的并发请求数和响应延迟。`而测试的同时，还可以用 **vmstat、pidstat 等性能工具，观察系统和进程的 CPU 使用率**。这样，我们就同时获得了应用程序和系统资源这两个维度的指标数值。

不过，在进行性能测试时，有两个特别重要的地方你需要注意下。**第一，要避免性能测试工具干扰应用程序的性能**。通常，对 Web 应用来说，性能测试工具跟目标应用程序要在不同的机器上运行。比如，在之前的 Nginx 案例中，我每次都会强调要用两台虚拟机，其中一台运行 Nginx 服务，而另一台运行模拟客户端的工具，就是为了避免这个影响。**第二，避免外部环境的变化影响性能指标的评估。**这要求优化前、后的应用程序，都运行在相同配置的机器上，并且它们的外部依赖也要完全一致。比如还是拿 Nginx 来说，就可以运行在同一台机器上，并用相同参数的客户端工具来进行性能测试。


##  3. 多个性能问题同时存在，要怎么选择？
在性能测试的领域，流传很广的一个说法是“二八原则”，**也就是说 80% 的问题都是由 20% 的代码导致的。只要找出这 20% 的位置，你就可以优化 80% 的性能**。所以，我想表达的是，并不是所有的性能问题都值得优化。

##  4. 有多种优化方法时，要如何选择?

一般情况下，我们当然想选能最大提升性能的方法，这其实也是性能优化的目标。性能优化并非没有成本

##  4.1 CPU 优化

###  4.2 应用程序优化

降低 CPU 使用率的最好方法当然是，排除所有不必要的工作，只保留最核心的逻辑。比如`减少循环的层次、减少递归、减少动态内存分配`等等。

用程序的性能优化也包括很多种方法:

 - **编译器优化**：很多编译器都会提供优化选项，适当开启它们，在编译阶段你就可以获得编译器的帮助，来提升性能。比如， gcc 就提供了优化选项-O2，开启后会自动对应用程序的代码进行优化。
 - **算法优化**：使用复杂度更低的算法，可以显著加快处理速度。比如，在数据比较大的情况下，可以用 O(nlogn 的排序算法（如快排、归并排序等），代替 O(n^2) 的排序算法（如冒泡、插入排序等）。
 - **异步处理**：使用异步处理，可以避免程序因为等待某个资源而一直阻塞，从而提升程序的并发处理能力。比如，把轮询替换为事件通知，就可以避免轮询耗费 CPU 的问题。
 - **多线程代替多进程**：前面讲过，相对于进程的上下文切换，线程的上下文切换并不切换进程地址空间，因此可以降低上下文切换的成本。
 - **善用缓存**：经常访问的数据或者计算过程中的步骤，可以放到内存中缓存起来，这样在下次用时就能直接从内存中获取，加快程序的处理速度。

###  4.3 系统优化

优化 CPU 的运行，一方面要充分利用 CPU 缓存的本地性，加速缓存访问；另一方面，就是要控制进程的 CPU 使用情况，减少进程间的相互影响

具体来说，系统层面的 CPU 优化方法也有不少，这里我同样列举了最常见的一些方法，方便你记忆和使用。

 - **CPU 绑定**：把进程绑定到一个或者多个 CPU 上，可以提高 CPU 缓存的命中率，减少跨 CPU 调度带来的上下文切换问题。
 - **CPU 独占**：跟 CPU 绑定类似，进一步将 CPU 分组，并通过 CPU 亲和性机制为其分配进程。这样，这些 CPU就由指定的进程独占，换句话说，不允许其他进程再来使用这些 CPU。
 - **优先级调整**：使用 nice 调整进程的优先级，正值调低优先级，负值调高优先级。优先级的数值含义前面我们提到过，忘了的话及时复习一下。在这里，适当降低非核心应用的优先级，增高核心应用的优先级，可以确保核心应用得到优先处理。
 - **为进程设置资源限制**：使用 Linux cgroups  来设置进程的 CPU 使用上限，可以防止由于某个应用自身的问题，而耗尽系统资源。
 - **NUMA（Non-Uniform Memory Access）优化**：支持 NUMA 的处理器会被划分为多个 node，每个 node都有自己的本地内存空间。NUMA 优化，其实就是让 CPU 尽可能只访问本地内存。
 - **中断负载均衡**：无论是软中断还是硬中断，它们的中断处理程序都可能会耗费大量的 CPU。开启 irqbalance 服务或者配置smp_affinity，就可以把中断处理过程自动负载均衡到多个 CPU 上。


## 5.  千万避免过早优化
**过早优化是万恶之源”**
针对当前情况进行的优化，很可能并不适应快速变化的新需求。这样，在新需求出现时，这些复杂的优化，反而可能阻碍新功能的开发。
