[toc]

---

# 前言

最近在做登录授权，因为需要保存用户密码到数据库，那此时引发了一个思考：

1. 密码明文存入，那任何人员登录数据库后，都能轻而易举的拿到用户密码，并进行登录。也就是明文存入数据库，极其容易泄露。

2. 既然密码不能明文存入数据，那只能使用密文。
   1. 如果只是对密码进行加密后存入会导致什么问题呢？同一个加密出来的密文是一致的。
   2. 密文是否可逆？通过密文是不是能反编译出明文？
3. 那针对加密的基础上，是不是可以通过额外的一个变量（salt），来进行加密，这样同样的密码+salt，得出来的密文就是不一样的了。

基于以上几点思考，这篇文章会着重讲述一下随机数生成的几种方式，主要是用来生成种子。

# Salt生成

Salt就是一串随机数，只是我们都知道会分为伪随机和真随机，那么用真随机和伪随机产生的随机数就会达成不同的效果。

## Random

我们最常用的方式如下：

```java
Random random = new Random();
int num = random.nextInt(10);
```

那现在简单的看一下 Random 的源码。

```java
public Random() {
    this(seedUniquifier() ^ System.nanoTime());
}   
public Random(long seed) {
    if (getClass() == Random.class)
        this.seed = new AtomicLong(initialScramble(seed));
    else {
        // subclass might have overriden setSeed
        this.seed = new AtomicLong();
        setSeed(seed);
    }
}
private static long seedUniquifier() {
    // L'Ecuyer, "Tables of Linear Congruential Generators of
    // Different Sizes and Good Lattice Structure", 1999
    for (;;) {
        long current = seedUniquifier.get();
        long next = current * 181783497276652981L;
        if (seedUniquifier.compareAndSet(current, next))
            return next;
    }
}
private static final AtomicLong seedUniquifier
    = new AtomicLong(8682522807148012L);
```

从这里可以看出 Random类，默认使用了 当前系统时间(System.nanoTime()) 和 静态AtomicLong 两者的异或结果 作为种子。

也就是说当我们同时并发的时候，seed 是一样时，产生出来的随机数也是一样的。因为他们基于的随机数算法也是一样的，所以这是可以预测出来的。

我们再来看一下常用的方法 nextInt() 。

```java
public int nextInt() {
    return next(32);
}
protected int next(int bits) {
    long oldseed, nextseed;
    AtomicLong seed = this.seed;
    do {
        oldseed = seed.get();
        nextseed = (oldseed * multiplier + addend) & mask;
    } while (!seed.compareAndSet(oldseed, nextseed));
    return (int)(nextseed >>> (48 - bits));
}
```

我们看到了这里使用了 AtomicLong 作为seed，Atomic原子类虽然保证了数据的原子性，但是其底层的CAS机制，使得在多线程并发下，性能并不算乐观。

简单写了个demo 跑了下并发

```java
  Random random = new Random();

        new Thread(() -> {
            while (true) {
                int i = random.nextInt(1000);
                System.out.println(System.currentTimeMillis() + "--" + i);
            }
        }).start();
        new Thread(() -> {
            while (true) {
                int i = random.nextInt(1000);
                System.out.println(System.currentTimeMillis() + "--" + i);
            }
        }).start();

        new Thread(() -> {
            while (true) {
                int i = random.nextInt(1000);
                System.out.println(System.currentTimeMillis() + "--" + i);
            }
        }).start();

        new Thread(() -> {
            while (true) {
                int i = random.nextInt(1000);
                System.out.println(System.currentTimeMillis() + "--" + i);
            }
        }).start();

```

从执行结果上来看，4线程执行下，平均每1秒可以生成280个随机数。那有什么办法来解决多线程下的随机数生成呢？

## ThreadLocalRandom

ThreadLocalRandom 是继承 Random 类，并在其基础上解决了并发生成的问题。

每一个线程有一个独立的**随机数生成器**，在并发的时候不需要跟别的线程竞争。**效率更高！** 

```java
new Thread(() -> {
    ThreadLocalRandom current = ThreadLocalRandom.current();
    while (true) {
        int i = current.nextInt();
        System.out.println(System.currentTimeMillis() + "--" + i);
    }
}).start();
```

ThreadLocalRandom 通过 current 方法获取当前线程所持有的 随机数生成器实例。

```java
public static ThreadLocalRandom current() {
    if (UNSAFE.getInt(Thread.currentThread(), PROBE) == 0)
        localInit();
    return instance;
}

static final void localInit() {
    int p = probeGenerator.addAndGet(PROBE_INCREMENT);
    int probe = (p == 0) ? 1 : p; // skip 0
    long seed = mix64(seeder.getAndAdd(SEEDER_INCREMENT));
    Thread t = Thread.currentThread();
    UNSAFE.putLong(t, SEED, seed);
    UNSAFE.putInt(t, PROBE, probe);
}
```

> localInit 方法是用来生成当前线程的属性。仅在 Thread.threadLocalRandomProbe为0时调用，表示需要一个线程本地seed值需要生成。注意：尽管初始化是 ThreadLocal 的，但我们需要依赖一个(静态)原子生成器来初始化值。

## SecureRandom

我们知道 Random 实现的算法是伪随机，也就是有规律的随机算法。进行随机时，在随机算法的种子seed 的基础上进行一定的变换，从而产生随机数字。

当不可预测性至关重要时， 如大多数对安全性要求较高的环境可以使用密码学的 PRNG。 不管选择了哪一种 PRNG， 都要始终使用带有充足熵的数值作为该算法的种子。 （诸如当前时间之类的数值只提供很小的熵， 因此不应该使用（Random的实现）。 ） 

1. SecureRandom 也是继承于 Random

2. SecureRandom提供加密的强随机生成器(RNG)。

3. SecureRandom 和 Random 都是种子一样时，生成出来的随机数也是一样的。二者区别在于：

   - SecureRandom 的种子必须是不可预测的。
   - Random 的种子是基于系统时间的

注意:根据实现的不同，{@code generateSeed}和{@code nextBytes}方法可能会因为熵被收集而阻塞，例如，如果它们需要从各种类unix操作系统上的/dev/random读取。 

SecureRandom，实现强随机的核心方法在于：SecureRandomSpi.engineNextBytes 方法

```java
public SecureRandom() {
    /*
         * This call to our superclass constructor will result in a call
         * to our own {@code setSeed} method, which will return
         * immediately when it is passed zero.
         */
    super(0);
    getDefaultPRNG(false, null);
}

private void getDefaultPRNG(boolean setSeed, byte[] seed) {
    String prng = getPrngAlgorithm();
    if (prng == null) {
        // bummer, get the SUN implementation
        prng = "SHA1PRNG";
        this.secureRandomSpi = new sun.security.provider.SecureRandom();
        this.provider = Providers.getSunProvider();
        if (setSeed) {
            this.secureRandomSpi.engineSetSeed(seed);
        }
    } else {
        try {
            SecureRandom random = SecureRandom.getInstance(prng);
            this.secureRandomSpi = random.getSecureRandomSpi();
            this.provider = random.getProvider();
            if (setSeed) {
                this.secureRandomSpi.engineSetSeed(seed);
            }
        } catch (NoSuchAlgorithmException nsae) {
            // never happens, because we made sure the algorithm exists
            throw new RuntimeException(nsae);
        }
    }
    // JDK 1.1 based implementations subclass SecureRandom instead of
    // SecureRandomSpi. They will also go through this code path because
    // they must call a SecureRandom constructor as it is their superclass.
    // If we are dealing with such an implementation, do not set the
    // algorithm value as it would be inaccurate.
    if (getClass() == SecureRandom.class) {
        this.algorithm = prng;
    }
}

```

### 使用方法

配置 -Djava.security参数来修改调用的算法 

```java
// 没有配置时，默认与SHA1PRNG
SecureRandom secureRandom = new SecureRandom();

SecureRandom secureRandom1 = SecureRandom.getInstance("SHA1PRNG");

// 通常可以用来作为其它算法的种子
secureRandom.generateSeed(20);
```

