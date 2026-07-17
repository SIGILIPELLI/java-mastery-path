# 05 · Arrays & Basic Collections

## Arrays

An array is a fixed-size, ordered block of elements of the same type.

```java
// Declare and initialize
int[] numbers = {10, 20, 30, 40};

// Declare with a size, fill later
String[] names = new String[3];
names[0] = "Alice";
names[1] = "Bob";
names[2] = "Cara";

System.out.println(numbers[0]);      // 10
System.out.println(numbers.length);  // 4  -- length is a field, not a method
```

### Iterating arrays

```java
int[] scores = {90, 85, 77, 100};

// Classic indexed loop
for (int i = 0; i < scores.length; i++) {
    System.out.println("Index " + i + ": " + scores[i]);
}

// Enhanced for-each loop
for (int score : scores) {
    System.out.println(score);
}
```

### 2D arrays

```java
int[][] grid = {
    {1, 2, 3},
    {4, 5, 6}
};

System.out.println(grid[1][2]);   // 6 -- row 1, column 2

for (int[] row : grid) {
    for (int value : row) {
        System.out.print(value + " ");
    }
    System.out.println();
}
```

### Common array utilities

```java
import java.util.Arrays;

int[] nums = {5, 3, 8, 1};

Arrays.sort(nums);
System.out.println(Arrays.toString(nums));   // [1, 3, 5, 8]

int[] copy = Arrays.copyOf(nums, nums.length);
```

Arrays are fixed-size — once created, you can't add or remove elements, only
overwrite existing slots. For dynamic sizing, use `ArrayList`.

## ArrayList — a resizable list

`ArrayList<T>` is part of the Collections Framework (covered in depth in
[Level 2](../level-2/02-collections-framework.md)) and grows or shrinks as you
add or remove elements. It holds objects, so primitives are auto-boxed (see
[Module 2](02-variables-data-types.md)).

```java
import java.util.ArrayList;
import java.util.List;

List<String> todos = new ArrayList<>();

todos.add("Buy milk");
todos.add("Write report");
todos.add("Call Sam");

System.out.println(todos);              // [Buy milk, Write report, Call Sam]
System.out.println(todos.get(0));        // Buy milk
System.out.println(todos.size());        // 3

todos.remove("Call Sam");                 // remove by value
todos.remove(0);                           // remove by index -- careful, overload!
System.out.println(todos);               // [Write report]

todos.add("Write report");
for (String todo : todos) {
    System.out.println("- " + todo);
}
```

### List of numbers (boxed)

```java
List<Integer> scores = new ArrayList<>();
scores.add(95);
scores.add(80);

int sum = 0;
for (int score : scores) {   // auto-unboxed on each iteration
    sum += score;
}
System.out.println(sum);   // 175
```

| Feature | Array | ArrayList |
|---------|-------|-----------|
| Size | Fixed at creation | Grows/shrinks dynamically |
| Element type | Primitives or objects | Objects only (auto-boxing for primitives) |
| Syntax | `arr[i]` | `list.get(i)` |
| Declared as | `int[] arr` | `List<Integer> list` |

## Exercise

Write a program that stores five student names in an `ArrayList<String>` and
five corresponding scores in an `int[]` array. Print each name alongside its
score using an indexed loop, then compute and print the average score. Finally
add one more student to the `ArrayList` and print the updated size.
