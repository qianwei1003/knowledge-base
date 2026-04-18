# 阿里 Java 开发手册（嵩山版）

> **来源**: 阿里巴巴 Java 开发手册 嵩山版
> **特点**: 业界最权威，覆盖编程规约、异常日志、单元测试、安全规约

---

## 核心要点速查

| 类别 | 规则 |
|------|------|
| **命名** | 类 UpperCamelCase，方法/变量 lowerCamelCase，常量 CAPS |
| **缩进** | 4 空格 |
| **行长度** | 单行不超过 120 字符 |
| **OOP** | 避免 `Object.equals()` 比较字符串常量 |
| **集合** | 注意 `null`，避免在 foreach 中 add/remove |
| **并发** | 线程池命名，使用后 `remove()` ThreadLocal |

---

## 一、命名风格

### 基本命名

| 类型 | 规则 | 示例 |
|------|------|------|
| 类名 | UpperCamelCase | `UserController` |
| 方法名/变量 | lowerCamelCase | `getUserById()` |
| 常量 | 全大写+下划线 | `MAX_COUNT` |
| 抽象类 | Abstract/Base 开头 | `AbstractUserService` |
| 异常类 | Exception 结尾 | `BusinessException` |
| 枚举类 | - | `enum Status { RUNNING, SUCCESS }` |

### 命名禁止

```java
// ❌ 禁止拼音与英文混合
// ❌ 禁止中文命名
// ❌ 禁止下划线（除常量外）
// ❌ 禁止 `$` 符号
// ❌ 禁止单字符命名（除临时变量）
```

---

## 二、常量定义

### 禁止魔法值

```java
// ❌ 禁止硬编码魔法值
if (type == 1) { }
new HashMap<>(16);

// ✅ 定义常量
private static final int TYPE_NORMAL = 1;
private static final int HASHMAP_DEFAULT_CAPACITY = 16;

if (type == TYPE_NORMAL) { }
new HashMap<>(HASH_MAP_DEFAULT_CAPACITY);
```

### 常量分组

```java
// ✅ 相关常量归类枚举
public enum Status {
    RUNNING(1, "运行中"),
    SUCCESS(0, "成功"),
    FAILED(-1, "失败");

    private final int code;
    private final String desc;

    Status(int code, String desc) {
        this.code = code;
        this.desc = desc;
    }
}
```

---

## 三、代码格式

### 基本格式

```java
// ✅ 4 空格缩进
// ✅ 左大括号不单独占一行
public class UserService {
    public void login(String name, String password) {
        if (name == null || name.isEmpty()) {
            return;
        }
        // ...
    }
}

// ✅ 单行不超过 120 字符
// ✅ 运算符两侧加空格
int result = a + b;
boolean flag = a == b && c > d;

// ✅ 不同职责用空行分隔
public void method() {
    // 初始化
    init();

    // 业务逻辑
    process();

    // 收尾
    cleanup();
}
```

---

## 四、OOP 规约

### 基本原则

```java
// ✅ 使用组合替代继承（优先使用组合）
public class UserDao {
    private DbConnector connector;  // ✅ 组合

    public UserDao(DbConnector connector) {
        this.connector = connector;
    }
}

// ✅ 使用 lombok 简化 POJO
@Data
@NoArgsConstructor
@AllArgsConstructor
public class User {
    private Long id;
    private String name;
}

// ❌ 避免 Object.equals() 比较字符串常量
String name = "admin";
if (name.equals("admin")) { }  // ❌ 可能 NPE
if ("admin".equals(name)) { }  // ✅ 安全

// ✅ 使用 Objects.equals() 更安全
if (Objects.equals(name, "admin")) { }  // ✅
```

### 构造函数

```java
// ❌ 构造函数中不要执行可能失败的操作
public class UserService {
    private final UserDao userDao;

    public UserService() {
        this.userDao = new UserDao();  // ❌ 可能在构造时失败
    }

    // ✅ 依赖注入
    public UserService(UserDao userDao) {
        this.userDao = Objects.requireNonNull(userDao);  // ✅
    }
}
```

### 循环中的变量

```java
// ❌ 禁止在 for/while 循环中修改循环变量
for (int i = 0; i < list.size(); i++) {
    if (list.get(i) == null) {
        list.remove(i);  // ❌ 可能漏掉元素
    }
}

// ✅ 使用迭代器
for (Iterator<T> it = list.iterator(); it.hasNext(); ) {
    if (it.next() == null) {
        it.remove();  // ✅
    }
}

// ✅ 或使用 removeIf (Java 8+)
list.removeIf(Objects::isNull);
```

---

## 五、日期时间

### 推荐用法

```java
// ✅ 使用 LocalDateTime（Java 8+）
LocalDateTime now = LocalDateTime.now();
LocalDateTime endTime = LocalDateTime.of(2024, 12, 31, 23, 59, 59);

// ❌ 避免使用 java.util.Date
// ❌ 避免使用 SimpleDateFormat（线程不安全）
// ✅ 使用 DateTimeFormatter（线程安全）
DateTimeFormatter formatter = DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss");

// ✅ 日期格式化
String dateStr = now.format(formatter);
```

---

## 六、集合处理

### ArrayList

```java
// ✅ 创建时预估容量
ArrayList<User> users = new ArrayList<>(100);

// ❌ 避免频繁扩容
// 每次扩容约 1.5 倍，有性能开销
```

### Map 操作

```java
// ✅ HashMap key 允许 null
HashMap<String, Object> map = new HashMap<>();
map.put(null, null);  // ✅

// ❌ ConcurrentHashMap 不允许 null
ConcurrentHashMap<String, Object> map = new ConcurrentHashMap<>();
map.put(null, "value");  // ❌ 抛出 NPE

// ✅ 遍历 Map 使用 entrySet
for (Map.Entry<String, Object> entry : map.entrySet()) {
    System.out.println(entry.getKey() + ":" + entry.getValue());
}

// ✅ 或使用 Java 8+ forEach
map.forEach((k, v) -> System.out.println(k + ":" + v));
```

### foreach 遍历

```java
// ❌ 不要在 foreach 中 add/remove
List<String> list = new ArrayList<>();
for (String item : list) {
    if (item.equals("remove")) {
        list.remove(item);  // ❌ ConcurrentModificationException
    }
}

// ✅ 使用迭代器
Iterator<String> it = list.iterator();
while (it.hasNext()) {
    if (it.next().equals("remove")) {
        it.remove();  // ✅
    }
}
```

---

## 七、并发处理

### 线程池

```java
// ❌ 禁止使用 Executors 创建线程池
ExecutorService executor = Executors.newFixedThreadPool(10);  // ❌
ExecutorService executor = Executors.newCachedThreadPool();  // ❌

// ✅ 使用 ThreadPoolExecutor 明确参数
ThreadPoolExecutor executor = new ThreadPoolExecutor(
    2,                              // corePoolSize
    10,                             // maximumPoolSize
    60L,                            // keepAliveTime
    TimeUnit.SECONDS,               // unit
    new LinkedBlockingQueue<>(100), // workQueue
    new ThreadFactoryBuilder()      // ✅ 命名线程
        .setNameFormat("user-task-%d")
        .build()
);

// ✅ 线程必须有有意义的名字
```

### ThreadLocal

```java
// ✅ 使用后必须 remove
private static final ThreadLocal<User> currentUser = new ThreadLocal<>();

public void process() {
    try {
        currentUser.set(getCurrentUser());
        // 业务逻辑
    } finally {
        currentUser.remove();  // ✅ 必须
    }
}

// ❌ 不 remove 会导致内存泄漏
```

### 同步控制

```java
// ✅ 同步范围要明确
synchronized(this) {  // ❌ 范围过大
    // 大量无关代码
}

// ✅ 缩小同步范围
private final Object lock = new Object();

public void method() {
    synchronized(lock) {  // ✅ 精确控制
        // 仅必要的同步代码
    }
}
```

---

## 八、异常处理

### 基本原则

```java
// ✅ 抛出具体异常类型
throw new IllegalArgumentException("用户名不能为空");
throw new BusinessException("用户不存在", 1001);

// ❌ 不要吞掉异常
try {
    process();
} catch (Exception e) {
    // ❌ 静默忽略
}

// ✅ 至少记录日志
try {
    process();
} catch (Exception e) {
    log.error("处理失败", e);
    throw e;  // ✅ 重新抛出或包装
}

// ✅ 包装异常保留原因
throw new BusinessException("业务处理失败", e);  // ✅
```

### 返回值

```java
// ✅ 有返回值的方法用 Optional 或抛异常
public Optional<User> findById(Long id) {
    return Optional.ofNullable(userMap.get(id));
}

// ❌ 不要返回 null 表示不存在
public User findById(Long id) {
    return userMap.get(id);  // ❌ 返回 null
}
```

---

## AI 编程调用示例

```markdown
请遵循阿里 Java 开发手册嵩山版：
- 类名 UpperCamelCase，方法/变量 lowerCamelCase
- 禁止硬编码魔法值，定义常量或枚举
- 4 空格缩进，单行不超过 120 字符
- 避免 Object.equals() 比较字符串，用 Objects.equals()
- 使用 LocalDateTime 处理日期时间
- HashMap key 允许 null，ConcurrentHashMap 不允许
- 不要在 foreach 中 add/remove
- 使用 ThreadPoolExecutor 而非 Executors
- 使用 ThreadLocal 后必须 remove()
- 异常要记录日志并重新抛出
```
