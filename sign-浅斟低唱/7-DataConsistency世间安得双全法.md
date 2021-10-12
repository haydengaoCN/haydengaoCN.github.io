> 缓存适用于读多写少（8-2）的场景，通过访问缓存而非数据库提升了处理读请求的速度。然而，却对数据库的一致性带来了挑战。这篇文章主要探讨数据不一致的原因以及可能的解决办法。
>
> 此外，也试图整理下计算机科学领域最难的两个问题之一的[缓存失效问题](https://skeptics.stackexchange.com/questions/19836/has-phil-karlton-ever-said-there-are-only-two-hard-things-in-computer-science)。
>
> There are only two hard things in Computer Science: cache invalidation and naming things.
>
> -- Phil Karlton

# 1-数据一致性

[数据一致性](https://note.youdao.com/s/SJ85I3UA)

<figure>
  <img src="7-DataConsistency世间安得双全法.assets/image-20211012155544713.png" alt="img" style="zoom: 50%;">
  <figcaption>Fig.1 数据一致性。</figcaption>
</figure>


# 2-缓存失效

[缓存三大问题](https://note.youdao.com/s/OglRzOSR)

<figure>
  <img src="7-DataConsistency世间安得双全法.assets/image-20211012155045333.png" alt="img" style="zoom: 50%;">
  <figcaption>Fig.2 缓存三大问题以及解决办法。</figcaption>
</figure>
