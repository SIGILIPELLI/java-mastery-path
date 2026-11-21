---
title: "Final Quiz"
description: "Test what you have learned across all four levels of Java Mastery Path with a 19-question final quiz."
---

# Final Quiz

19 questions drawn from all four levels of this course. You need **70% (14 out of 19)** to pass —
you can retake it as many times as you like. Your best result is saved in this browser.

<div id="mp-quiz-root"></div>

<script type="application/json" id="mp-quiz-data">
[
  {
    "question": "What does this print?\n```java\npublic class Main {\n\npublic static void main(String[] args) throws Exception {\nint a = 7, b = 2;\nSystem.out.println(a / b + \" \" + (double) a / b);\n}\n}\n```\n",
    "options": [
      "3 3",
      "4 3.5",
      "3 3.5",
      "3.5 3.5"
    ],
    "correctIndex": 2,
    "explanation": "Integer division truncates. Verified by compiling and running it."
  },
  {
    "question": "What does this print?\n```java\npublic class Main {\n\npublic static void main(String[] args) throws Exception {\nInteger x = 127, y = 127, p = 1000, q = 1000;\nSystem.out.println((x == y) + \" \" + (p == q) + \" \" + p.equals(q));\n}\n}\n```\n",
    "options": [
      "true false false",
      "true true true",
      "true false true",
      "false false true"
    ],
    "correctIndex": 2,
    "explanation": "Integer caching: == compares references outside -128..127. Verified by compiling and running it."
  },
  {
    "question": "What does this print?\n```java\npublic class Main {\n\npublic static void main(String[] args) throws Exception {\nString a = \"hi\", c = new String(\"hi\");\nSystem.out.println((a == c) + \" \" + a.equals(c));\n}\n}\n```\n",
    "options": [
      "true false",
      "false false",
      "false true",
      "true true"
    ],
    "correctIndex": 2,
    "explanation": "Use equals for content. Verified by compiling and running it."
  },
  {
    "question": "What does this print?\n```java\npublic class Main {\n\npublic static void main(String[] args) throws Exception {\ntry { System.out.print(\"A\"); Integer.parseInt(\"x\"); System.out.print(\"B\"); } catch (NumberFormatException e) { System.out.print(\"C\"); } finally { System.out.print(\"D\"); }\n}\n}\n```\n",
    "options": [
      "AD",
      "ACD ",
      "ABD",
      "ACD"
    ],
    "correctIndex": 3,
    "explanation": "finally always runs. Verified by compiling and running it."
  },
  {
    "question": "What does this print?\n```java\nimport java.util.*;\npublic class Main {\n\npublic static void main(String[] args) throws Exception {\nList<Integer> l = new ArrayList<>(List.of(1, 2, 3, 4));\nl.removeIf(x -> x % 2 == 0);\nSystem.out.println(l);\n}\n}\n```\n",
    "options": [
      "Error",
      "[1, 2, 3]",
      "[1, 3]",
      "[2, 4]"
    ],
    "correctIndex": 2,
    "explanation": "removeIf filters in place. Verified by compiling and running it."
  },
  {
    "question": "What does this print?\n```java\nimport java.util.*;\npublic class Main {\n\npublic static void main(String[] args) throws Exception {\nMap<String,Integer> m = new HashMap<>();\nfor (String w : \"a b a c a b\".split(\" \")) m.merge(w, 1, Integer::sum);\nSystem.out.println(new TreeMap<>(m));\n}\n}\n```\n",
    "options": [
      "Error",
      "{a=3}",
      "{a=1, b=1, c=1}",
      "{a=3, b=2, c=1}"
    ],
    "correctIndex": 3,
    "explanation": "merge counts occurrences. Verified by compiling and running it."
  },
  {
    "question": "What does this print?\n```java\npublic class Main {\nstatic class Animal { String speak() { return \"Generic\"; } }\nstatic class Dog extends Animal { @Override String speak() { return \"Woof\"; } }\npublic static void main(String[] args) throws Exception {\nAnimal a = new Dog();\nSystem.out.println(a.speak());\n}\n}\n```\n",
    "options": [
      "Error",
      "Animal",
      "Woof",
      "Generic"
    ],
    "correctIndex": 2,
    "explanation": "Dynamic dispatch uses the runtime type. Verified by compiling and running it."
  },
  {
    "question": "What does this print?\n```java\npublic class Main {\nrecord P(int x, int y) {}\npublic static void main(String[] args) throws Exception {\nSystem.out.println(new P(1, 2) + \" \" + new P(1, 2).equals(new P(1, 2)));\n}\n}\n```\n",
    "options": [
      "Error",
      "P[x=1, y=2] true",
      "P[1,2] true",
      "P(1,2) false"
    ],
    "correctIndex": 1,
    "explanation": "Records generate toString, equals and hashCode. Verified by compiling and running it."
  },
  {
    "question": "What does this print?\n```java\nimport java.util.*; import java.util.stream.*;\npublic class Main {\n\npublic static void main(String[] args) throws Exception {\nList<String> w = List.of(\"apple\", \"bob\", \"avocado\");\nSystem.out.println(w.stream().filter(s -> s.startsWith(\"a\")).map(String::length).collect(Collectors.toList()));\n}\n}\n```\n",
    "options": [
      "[apple, avocado]",
      "[2]",
      "[5, 7]",
      "Error"
    ],
    "correctIndex": 2,
    "explanation": "Streams are lazy until a terminal operation. Verified by compiling and running it."
  },
  {
    "question": "What does this print?\n```java\npublic class Main {\n\npublic static void main(String[] args) throws Exception {\nint[] c = {0};\nObject lock = new Object();\nThread[] ts = new Thread[4];\nfor (int i = 0; i < 4; i++) { ts[i] = new Thread(() -> { for (int j = 0; j < 1000; j++) synchronized (lock) { c[0]++; } }); ts[i].start(); }\nfor (Thread t : ts) t.join();\nSystem.out.println(c[0]);\n}\n}\n```\n",
    "options": [
      "4000",
      "0",
      "Undefined",
      "Less than 4000"
    ],
    "correctIndex": 0,
    "explanation": "synchronized prevents lost updates. Verified by compiling and running it."
  },
  {
    "question": "What does this print?\n```java\nimport java.util.*; import java.util.concurrent.*;\npublic class Main {\n\npublic static void main(String[] args) throws Exception {\nExecutorService ex = Executors.newFixedThreadPool(3);\nint s = 0;\nList<Callable<Integer>> tasks = List.of(() -> 1, () -> 2, () -> 3);\nfor (Future<Integer> f : ex.invokeAll(tasks)) s += f.get();\nex.shutdown();\nSystem.out.println(s);\n}\n}\n```\n",
    "options": [
      "Error",
      "3",
      "6",
      "0"
    ],
    "correctIndex": 2,
    "explanation": "Pool tasks return results through Futures. Verified by compiling and running it."
  },
  {
    "question": "A HashMap lookup returns null for a key that was inserted. What's the likely bug?",
    "options": [
      "The key class overrides equals() without hashCode() (or mutates after insertion)",
      "GC",
      "Capacity",
      "Wrong type"
    ],
    "correctIndex": 0,
    "explanation": "Keep them consistent."
  },
  {
    "question": "A loop concatenates strings 100,000 times and is slow. What's the fix?",
    "options": [
      "Use arrays",
      "Use StringBuilder (strings are immutable)",
      "Synchronize",
      "Use +="
    ],
    "correctIndex": 1,
    "explanation": "Avoid quadratic copying."
  },
  {
    "question": "Two threads each lock account A then B, and B then A. What happens and what's the fix?",
    "options": [
      "Starvation; sleep",
      "Race; volatile",
      "Deadlock; always acquire locks in a consistent global order",
      "Nothing"
    ],
    "correctIndex": 2,
    "explanation": "Lock ordering."
  },
  {
    "question": "A REST endpoint concatenates user input into SQL. What's the fix?",
    "options": [
      "Use POST",
      "Trim input",
      "Escape quotes",
      "PreparedStatement/JPA parameters"
    ],
    "correctIndex": 3,
    "explanation": "Prevents injection."
  },
  {
    "question": "Passwords are stored with SHA-256. What should you use?",
    "options": [
      "MD5",
      "Plain",
      "Base64",
      "BCrypt/Argon2 via PasswordEncoder"
    ],
    "correctIndex": 3,
    "explanation": "Slow salted hashing."
  },
  {
    "question": "Spring Boot service is slow due to many queries for a list with children. What's this?",
    "options": [
      "GC",
      "Race",
      "The N+1 select problem; use fetch joins/batch fetching",
      "Deadlock"
    ],
    "correctIndex": 2,
    "explanation": "JPA tuning."
  },
  {
    "question": "A container gets OOM-killed though the heap seems small. What setting helps?",
    "options": [
      "Container-aware sizing (-XX:MaxRAMPercentage) and checking non-heap memory",
      "More threads",
      "Smaller image",
      "-server"
    ],
    "correctIndex": 0,
    "explanation": "JVM in containers."
  },
  {
    "question": "A consumer processes the same Kafka message twice after a rebalance. What must the handler be?",
    "options": [
      "Faster",
      "Stateless",
      "Idempotent",
      "Ordered"
    ],
    "correctIndex": 2,
    "explanation": "At-least-once delivery."
  }
]
</script>

## Coding challenges

Try these on your own computer, then compare your output. Nothing you write is sent anywhere.

### Reports with records and streams

Implement revenueByCategory (total per category, sorted by name) and topCustomers (top n by total spend, ties by name, formatted name=12.34) with the Streams API, using the Order record. The program also prints summary statistics.

**Starter code**

```java
import java.util.*;
import java.util.stream.*;

public class Main {
    record Order(String customer, String category, int qty, double price) {
        double total() { return qty * price; }
    }

    static Map<String, Double> revenueByCategory(List<Order> orders) {
        // total revenue per category, sorted by category name (use groupingBy with a TreeMap + summingDouble)
        return new TreeMap<>();
    }

    static List<String> topCustomers(List<Order> orders, int n) {
        // the n customers with the highest total spend (ties by name), formatted "name=12.34"
        return List.of();
    }

    public static void main(String[] args) {
        List<Order> orders = List.of(
                new Order("ada", "books", 2, 12.50), new Order("bo", "games", 1, 59.99),
                new Order("ada", "games", 1, 30.00), new Order("cy", "books", 4, 9.99),
                new Order("bo", "books", 1, 12.50));
        System.out.println(revenueByCategory(orders));
        System.out.println(topCustomers(orders, 2));
        System.out.println(orders.stream().mapToInt(Order::qty).summaryStatistics());
    }
}
```

**Expected output**

```text
{books=77.46, games=89.99}
[bo=72.49, ada=55.00]
IntSummaryStatistics{count=5, sum=9, min=1, average=1.800000, max=4}
```

<details><summary>Show solution and explanation</summary>

<pre><code>import java.util.*;
import java.util.stream.*;

public class Main {
    record Order(String customer, String category, int qty, double price) {
        double total() { return qty * price; }
    }

    static Map&lt;String, Double&gt; revenueByCategory(List&lt;Order&gt; orders) {
        return orders.stream().collect(Collectors.groupingBy(Order::category, TreeMap::new,
                Collectors.collectingAndThen(Collectors.summingDouble(Order::total), v -&gt; Math.round(v * 100) / 100.0)));
    }

    static List&lt;String&gt; topCustomers(List&lt;Order&gt; orders, int n) {
        return orders.stream()
                .collect(Collectors.groupingBy(Order::customer, Collectors.summingDouble(Order::total)))
                .entrySet().stream()
                .sorted(Map.Entry.&lt;String, Double&gt;comparingByValue().reversed().thenComparing(Map.Entry.comparingByKey()))
                .limit(n)
                .map(e -&gt; e.getKey() + &quot;=&quot; + String.format(&quot;%.2f&quot;, e.getValue()))
                .collect(Collectors.toList());
    }

    public static void main(String[] args) {
        List&lt;Order&gt; orders = List.of(
                new Order(&quot;ada&quot;, &quot;books&quot;, 2, 12.50), new Order(&quot;bo&quot;, &quot;games&quot;, 1, 59.99),
                new Order(&quot;ada&quot;, &quot;games&quot;, 1, 30.00), new Order(&quot;cy&quot;, &quot;books&quot;, 4, 9.99),
                new Order(&quot;bo&quot;, &quot;books&quot;, 1, 12.50));
        System.out.println(revenueByCategory(orders));
        System.out.println(topCustomers(orders, 2));
        System.out.println(orders.stream().mapToInt(Order::qty).summaryStatistics());
    }
}
</code></pre>

<p>groupingBy with a TreeMap and summingDouble builds the category totals; the customer ranking sorts map entries by value descending then key. Verified by running the solution with java Main.java.</p>
</details>

### A deadlock-free thread-safe transfer

Implement transfer(from, to, amount): lock both accounts (always the smaller id first), refuse overdrafts, move the money, and unlock in finally blocks. Run 2,000 concurrent opposite-direction transfers on a pool and check the total is preserved.

**Starter code**

```java
import java.util.*;
import java.util.concurrent.*;
import java.util.concurrent.locks.ReentrantLock;

public class Main {
    static class Account {
        final int id;
        private long balance;
        final ReentrantLock lock = new ReentrantLock();
        Account(int id, long balance) { this.id = id; this.balance = balance; }
        long balance() { return balance; }
        void adjust(long d) { balance += d; }
    }

    /** Move money atomically between two accounts and refuse overdrafts (return false). Always lock the
     *  account with the smaller id first so opposite-direction transfers cannot deadlock. */
    static boolean transfer(Account from, Account to, long amount) {
        return false;
    }

    public static void main(String[] args) throws Exception {
        Account a = new Account(1, 1_000), b = new Account(2, 1_000);
        ExecutorService pool = Executors.newFixedThreadPool(8);
        List<Future<Boolean>> results = new ArrayList<>();
        for (int i = 0; i < 2_000; i++) {
            final boolean forward = i % 2 == 0;
            results.add(pool.submit(() -> forward ? transfer(a, b, 7) : transfer(b, a, 5)));
        }
        for (Future<Boolean> f : results) f.get();
        pool.shutdown();
        System.out.println("total preserved: " + (a.balance() + b.balance() == 2_000));
        System.out.println("overdraft refused: " + !transfer(a, b, 1_000_000));
    }
}
```

**Expected output**

```text
total preserved: true
overdraft refused: true
```

<details><summary>Show solution and explanation</summary>

<pre><code>import java.util.*;
import java.util.concurrent.*;
import java.util.concurrent.locks.ReentrantLock;

public class Main {
    static class Account {
        final int id;
        private long balance;
        final ReentrantLock lock = new ReentrantLock();
        Account(int id, long balance) { this.id = id; this.balance = balance; }
        long balance() { return balance; }
        void adjust(long d) { balance += d; }
    }

    /** Move money atomically. Locks are always taken in id order, so opposite transfers can&#x27;t deadlock. */
    static boolean transfer(Account from, Account to, long amount) {
        Account first = from.id &lt; to.id ? from : to;
        Account second = first == from ? to : from;
        first.lock.lock();
        try {
            second.lock.lock();
            try {
                if (from.balance() &lt; amount) return false;
                from.adjust(-amount);
                to.adjust(amount);
                return true;
            } finally {
                second.lock.unlock();
            }
        } finally {
            first.lock.unlock();
        }
    }

    public static void main(String[] args) throws Exception {
        Account a = new Account(1, 1_000), b = new Account(2, 1_000);
        ExecutorService pool = Executors.newFixedThreadPool(8);
        List&lt;Future&lt;Boolean&gt;&gt; results = new ArrayList&lt;&gt;();
        for (int i = 0; i &lt; 2_000; i++) {
            final boolean forward = i % 2 == 0;
            results.add(pool.submit(() -&gt; forward ? transfer(a, b, 7) : transfer(b, a, 5)));
        }
        for (Future&lt;Boolean&gt; f : results) f.get();
        pool.shutdown();
        System.out.println(&quot;total preserved: &quot; + (a.balance() + b.balance() == 2_000));
        System.out.println(&quot;overdraft refused: &quot; + !transfer(a, b, 1_000_000));
    }
}
</code></pre>

<p>Taking locks in a consistent global order removes the circular wait that causes deadlock, and finally blocks guarantee unlocking. The invariant (total balance) holds because each transfer is atomic. Verified by running the solution with java Main.java.</p>
</details>
