[toc]

---

# LockUtils

> 利用 ReentrantLock 实现排他锁

```java
import java.util.Iterator;
import java.util.Map;
import java.util.Set;
import java.util.concurrent.ConcurrentHashMap;
import java.util.concurrent.Executors;
import java.util.concurrent.ScheduledExecutorService;
import java.util.concurrent.TimeUnit;
import java.util.concurrent.locks.ReentrantLock;

public class LockUtils {
    private static final ConcurrentHashMap<String, LockInfo> LOCKS = new ConcurrentHashMap<>();

    private static final ScheduledExecutorService executorService = Executors.newScheduledThreadPool(1);

    static {
        executorService.scheduleAtFixedRate(() -> {
            long currentTime = System.currentTimeMillis();
            Iterator<Map.Entry<String, LockInfo>> iterator = LOCKS.entrySet().iterator();
            while (iterator.hasNext()) {
                Map.Entry<String, LockInfo> entry = iterator.next();
                if (currentTime > entry.getValue().getExpireIn()) {
                    unlock(entry.getKey());
                }
            }
        }, 1, 1, TimeUnit.SECONDS);
    }

    public static boolean tryLock(String key, long timeout, TimeUnit unit) {
        LockInfo lock = LOCKS.computeIfAbsent(key, k -> new LockInfo(timeout, unit));
        try {
            return lock.tryLock(1, TimeUnit.SECONDS);
        } catch (InterruptedException e) {
            return false;
        }
    }

    public static void unlock(String key) {
        ReentrantLock lock = LOCKS.get(key);
        if (lock != null && lock.isHeldByCurrentThread()) {
            lock.unlock();
            if (!lock.isLocked()) {
                LOCKS.remove(key);
            }
        }
    }

    private static final class LockInfo extends ReentrantLock {
        public LockInfo(Long expireIn, TimeUnit unit) {
            super();
            this.expireIn = System.currentTimeMillis() + unit.toMillis(expireIn);
        }

        public LockInfo(boolean fair, Long expireIn, TimeUnit unit) {
            super(fair);
            this.expireIn = System.currentTimeMillis() + unit.toMillis(expireIn);
        }

        private Long expireIn;

        public Long getExpireIn() {
            return expireIn;
        }

        public void setExpireIn(Long expireIn) {
            this.expireIn = expireIn;
        }
    }
}

```

