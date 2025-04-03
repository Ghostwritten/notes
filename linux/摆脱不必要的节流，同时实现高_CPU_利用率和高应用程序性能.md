CPU Burst：摆脱不必要的节流，同时实现高 CPU 利用率和高应用程序性能 | CPU Burst: Getting Rid of Unnecessary Throttling, Achieving High CPU Utilization and Application Performance at the Same Time - Huaixin Chang & Tianchen Ding, Alibaba

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/44a19454fe1f0ecfd060be3fb0f417fe.png)


长期以来，CPU 节流一直是一个令人头疼的问题。即使当 pods 的 CPU 利用率远低于其 CPU 限制时，仍然存在许多长尾情况。因此，开发人员很难选择合适的硬限制。到目前为止，对于这一问题的答案始终是增加 CPU 硬限制或关闭 CPU 硬限制。然而，这一方法带来了一些新问题，如潜在的 难应付的性能影响、低 CPU 利用率和高 TCO（总拥有成本）。最近，融入到 Linux 5.14 中的 CPU Burst 的特性，成为了彻底解决不必要的 CPU 节流问题的一个新选择。注意到 CPU 节流是由 100 毫秒级的突发 CPU 使用引起的后，CPU Burst 的特性可允许平均 CPU 利用率低于 CPU 限制情况下可能的突发使用。应用 CPU Burst 后，用户可以同时获得高 CPU 利用率和高应用程序性能。在本会话中，我们将介绍执行 CPU 限制的内核机制，CPU Burst 更改的内容及其影响，以及如何评估此种更改。在本会话结束时，您可确切地了解到是否在您的 pods 上使用 CPU Burst。

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/237313aa9f43a074f9a0e5c0ab97dcda.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/8a01503cb259a7af2b3665a8aee67c67.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/5d987d88972922517fdfcf618d6d9c3a.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/706858b37aae9cd959424cd0e6a569e3.png)

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/14026c857927ad242d633688550555cd.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/15a507f4e041d12677a0c32533f70203.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/4df397cd5c7e73e169f1eb3e089180e1.png)

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/fbd82eebf2dfc84a5cb784c25945e402.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/489a05dde2d26b03ba6526899cc18f45.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/b3b345c6c53a2d749e5290dcafca2935.png)

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/cdc81411f9765eaf2bfdec370bd33c92.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/d22c253b51e5a0bb2c2f8933a1b3d08f.png)

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/e750b883021753da7808158c43e23467.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/24d6c4225f761161728813b4f31e4c2d.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/8a6e913a2cc86fbc2c47d26b27f53fb1.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/e83b58d07f1efdf3482e9ac21a0f8608.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/5f06c3b9c97653bd30f15767d75510c4.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/4e5bc5c7d0de5997249480d11defd623.png)



![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/b2ea68ff6a6216bba7f9b9a939545b51.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/d67418a3f0e9f11d5a3af24d1cf8dad8.png)

