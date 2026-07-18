# Conditions & Loops

## 79. Why should you always use braces around an if statement?

**Interview Answer**

Even if an `if` block contains only one statement, always use braces
`{}`.

Benefits: - Prevents bugs when adding new statements later. - Improves
readability. - Follows Java coding standards.

``` java
// Bad
if (isValid)
    process();

// Good
if (isValid) {
    process();
}
```

------------------------------------------------------------------------

## 80. Guess the output

The original question depends on a code snippet that is not included.

**Interview Tip:** Read carefully for: - Assignment (`=`) vs comparison
(`==`) - Integer overflow - Pre/post increment (`++i` vs `i++`) - Scope
of variables

------------------------------------------------------------------------

## 81. Guess the output

The code snippet is missing.

Typical concepts tested: - Operator precedence - Short-circuit operators
(`&&`, `||`) - Increment operators - Variable scope

------------------------------------------------------------------------

## 82. Guess the output of this switch block

The code snippet is missing.

Common interview points: - Missing `break` causes **fall-through**. -
`default` executes if no case matches.

------------------------------------------------------------------------

## 83. Guess the output of this switch block

Again, output depends on the code.

Interviewers usually test: - Fall-through - Duplicate case labels (not
allowed) - Constant expressions

------------------------------------------------------------------------

## 84. Should `default` be the last case in a switch?

No.

`default` can appear anywhere.

``` java
switch(day){
    default:
        System.out.println("Invalid");
        break;

    case 1:
        System.out.println("Monday");
}
```

It is convention---not a rule---to place it last.

------------------------------------------------------------------------

## 85. Can switch be used with a String?

Yes.

Supported since **Java 7**.

``` java
String role = "ADMIN";

switch(role){
    case "ADMIN":
        System.out.println("Admin");
        break;

    case "USER":
        System.out.println("User");
        break;

    default:
        System.out.println("Unknown");
}
```

Modern Java also supports **switch expressions**.

``` java
String result = switch(role){
    case "ADMIN" -> "A";
    case "USER" -> "U";
    default -> "N";
};
```

------------------------------------------------------------------------

## 86. Output of the for loop

The referenced code is missing.

Typical interview checks: - Loop initialization - Condition evaluation -
Increment step - Infinite loops

------------------------------------------------------------------------

## 87. What is an enhanced for loop?

Also called the **for-each loop**.

Used to iterate over arrays and collections.

``` java
int[] nums = {1,2,3};

for(int num : nums){
    System.out.println(num);
}
```

Advantages: - Cleaner syntax - No index handling - Less error-prone

Limitations: - Cannot modify collection structure while iterating. - No
direct access to index.

------------------------------------------------------------------------

## 88. Output of the for loop

Cannot determine without the original code.

Common topics: - Nested loops - `break` - `continue` - Variable scope

------------------------------------------------------------------------

## 89. Output of the program

Code snippet not provided.

Interviewers often test: - Loop termination - Integer overflow -
Break/continue - Scope

------------------------------------------------------------------------

## 90. Output of the program

Cannot determine without the program.

Always simulate execution line by line during interviews.

------------------------------------------------------------------------

# Common Loop Keywords

## break

Terminates the nearest loop.

``` java
for(int i=1;i<=5;i++){

    if(i==3){
        break;
    }

    System.out.println(i);
}
```

Output

``` text
1
2
```

------------------------------------------------------------------------

## continue

Skips the current iteration.

``` java
for(int i=1;i<=5;i++){

    if(i==3){
        continue;
    }

    System.out.println(i);
}
```

Output

``` text
1
2
4
5
```

------------------------------------------------------------------------

# Interview Cheat Sheet

  Topic              Key Point
  ------------------ -------------------------------
  if                 Always use braces
  switch             `break` prevents fall-through
  default            Can appear anywhere
  String in switch   Supported since Java 7
  Enhanced for       Cleaner iteration, no index
  break              Exit loop
  continue           Skip current iteration

## Interview Tip

Whenever an interviewer asks **"What's the output?"**, don't guess.

Explain your reasoning line by line. Even if you make a mistake,
demonstrating your thought process usually leaves a better impression
than giving a random answer.
