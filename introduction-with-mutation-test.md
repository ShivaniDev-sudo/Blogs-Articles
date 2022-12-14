
## Introduction to Mutation Testing

In recent years, countless examples show that poorly tested software increase cost, uncertainties in project release, customer dissatisfaction and reflects poorly on the company. Hence, practices such as Test Driven Development or Continuous Integration became an essential part of the software life cycle.

A high-quality test suite will guarantee the stability of the software.

For example, if we modify the logic of code, then at least one test case should fail, signalling a semantic change. 

Hence question is, how to create a quality test suite? And how do we make sure that we're not just executing every instruction, but also checking it? This is what mutation testing is, which we will discuss in detail in the following sections.

## Quality Of Test Suite

```
"A program test can be used to show the presence of bugs, but never to show their absence" - Dijkstra (1970)
```
Considering that the effectiveness of unit testing is well established, this subject becomes interesting if we are developing a tested project.

Often the first and only indicator of quality, we use the *code coverage metric*. Hypothetically, if the tests verify the program, it means the application is functioning correctly.

Also, it is not the only code coverage metric, but its ease to integrate with other projects, IDE etc. makes it a preferred choice. Thanks to the presence of numerous plug-and-play tools such as :

  - JaCoCo
  - Sonar
  - JCov

The closer this metric reaches 98-100 per cent, the more confident we are that no regression will happen. Unfortunately, there is no practical evidence to support this assertion and we will see how.

## Does 100% Code Coverage Work?

Unfortunately, code coverage metrics can sometimes be completely inefficient, because code coverage metric 100% only means that all lines have been executed at least once, but says nothing about the accuracy of the tests. 

Code coverage usually misses the behaviour of code on loops, recursion, method execution order etc.


## Problem of Code Coverage Metrics 

### Lack Of Edge Testing

Consider the following java code snippet:
```
public static String foo(int input) {
  if (i >= 0) {
    return "Yes";
  } else {
    return "No";
  }
}
```
The function "foo" performs a simple operation i.e. check if the given integer is greater than zero or not. It's easy to test: all we need to do is check that the function returns the correct value if the number is +ve or -ve.


We can test this code with two integer numbers such as 1 and -1 as input. To cover all this code, this step would be sufficient.


```
@Test
void shouldReturnYesWhenGivenOne() {
  assertThat("Yes", foo(1));
}

@Test
void shouldReturnNoWhenGivenMinus1() {
  assertThat("No", foo(-1));
}
```
This test provides:

  - Statement coverage is 100% 
  - Branch coverage is 100% 

The present assertions are correct and significant but the behaviour when (i == 0) is not verified at all. What will happen if in the future a developer accidentally changed the if condition as below:

```
public static String foo(int input) {
  if (i > 0) {
    return "Yes";
  } else {
    return "No";
  }
}
```

But even though the code is visibly broken, thanks to a missing boundary case, the tests will pass always. Hence, these test does not guarantee stability, and errors may go unnoticed.


### Poor Quality Testcase

Consider another example, where the method "reverseString" reverses any string passed to it. A simple java code will as follow:

```
public static String reverseString(String inp) {
  char inpArray[] = inp.toCharArray();
  String revseredString = "";
  for (int i = inpArray.length - 1; i >= 0; i--) {
    revseredString += inpArray[i];
  }
  return revseredString;
}
```
A JUnit test case will as follow:

```
assertEquals ( reverseString("wow") , "wow" )
```
If in future reverseString() logic is changed to simply returning the same string then our JUnit will never catch it. 

This example shows, how to tweak the validation and assertions that code coverage cannot catch.

# Mutation Tests: What are they?

Mutation testing is an approach for measuring the effectiveness of a test suite. To put it simply, this technique gives us more confidence in our tests. It works on a fairly simple concept. The idea is to modify and manipulate the original source code to see if the connected tests fail accordingly.


Hence, Flaws (or mutations) are automatically seeded into our code, and then tests are run. If the tests are green(pass), then the mutation is killed. If the tests are red(fails), then the mutation survived.


### Mutation testing concepts

In order to understand mutation testing better, let's explore some concepts first:

1)  **Mutants**: This is simply the mutated version of the source code. This is the code that contains minute changes. When the test data is run through the mutant, it should ideally give us different results than the original source code. Mutants are also called mutant programs.


    There are different types of mutants. These are the following:

      - *Surviving mutants*: As mentioned, these are the mutants that are still alive after running test data through the original and mutated variants of the source code. These must be killed. They are also known as Living Mutants.

       - *Mutants killed*: These are mutants that are killed after a mutation test. We get them when we get different results from the original and mutated versions of the source code.

       - *Equivalent mutants*: These are closely related to living mutants, in that they are "alive" even after running test data through them. What differentiates them from others is that they have the same meaning as the original source code, even though they may have a different syntax.

    *Mutant Example:*

    Consider the below method as the original source code:

    ```
    public static String foo(int i) {
      if (i >= 0) {
        return "true";
      } else {
        return "false";
      }
    }
    ```

    After mutation, this will be modified as below:

    ```
    public static String foo(int i) {
      if (i == 0) {
        return "true";
      } else {
        return "false";
      }
    }
    ```
2)  **Mutators** : This is what makes mutations possible, they are in the “driver's seat”. They essentially define the type of alteration or change to be made to the source code to have a mutant version. They may be called defects or mutation rules.

      There are several types of mutators:

       - *Value Mutation*: The value of a constant or a parameter in the condition is adjusted. e.g. changing the while loop condition by +2/-2 etc.
       - *Statements Mutation*: Some statements are deleted or the processing order is changed in a code block. e.g. remove the else state from an if-else block.
       - *Decision Mutation*: The conditions present in code logic is altered e.g. if condition_1 < condition_2 throw error)

3) **Mutation Score**: Using the ratio of surviving mutants vs total mutants as a metric, the final score is generated. The higher the score of mutants killed, the better the quality of tests.


## Mutation Test With PIT

Manually introducing mutations into the code is not a viable practice. There are tools in each language that independently create mutants for each of these cases. In the end, the goal is to identify how many mutants survive the test suite.

In Java, [PIT](https://pitest.org/) is the most widely used standard mutation testing tool. PIT is a maven plugin that applies mutations to the bytecode generated by the compilation and executes the test suite for each mutation.

To convince the of the power of mutation tests we will create a simple example in java with unit tests written using the JUnit framework.

## Mutation Test: Hands-On With Real Example.

Suppose we need to implement a very simple class that validates if a given number is two digit number or not. The specification is very simple:

  - A null or empty string is not accepted as input
  - Number must be greater than 10 and less than 99
  - number must be +ve for simplicity

### Setup PIT 
Installing the PIT is simple. We can simply add the maven plugin in the pom.xml.
```
// other omitted code 
<plugins>
  <plugin>
    <groupId>org.pitest</groupId>
    <artifactId>pitest-maven</artifactId>
    <version>1.7.3</version>
    <dependencies>
      <dependency>
        <groupId>org.pitest</groupId>
        <artifactId>pitest-junit5-plugin</artifactId>
        <version>0.12</version>
      </dependency>
    </dependencies>
    <configuration>
      <mutators>
        <mutator>DEFAULTS</mutator>
      </mutators>
      <outputFormats>
        <outputFormat>XML</outputFormat>
        <outputFormat>HTML</outputFormat>
      </outputFormats>
      <targetClasses>
        <param>en.demo.*</param>
      </targetClasses>
      <targetTests>
        <param>en.demo.*</param>
      </targetTests>
      <failWhenNoMutations>false</failWhenNoMutations>
      <exportLineCoverage>true</exportLineCoverage>
      <timestampedReports>true</timestampedReports>
    </configuration>
  </plugin>
</plugins>
// other omitted code 
```
By default, PIT introduces mutations throughout the code. Since the execution of the mutation test is an expensive operation, we can also configure PIT to insert mutations only in a specific package or class. Currently for simplicity, we will configure all the classes.

```
<targetClasses>
  <param>en.demo.*</param>
</targetClasses>
<targetTests>
  <param>en.demo.*</param>
</targetTests>
```

Below is a simple java implementation of code:

```
public class SimpleMath {
  boolean isTwoDigitNumber(int input) {
    return input > 9 && input <= 99;
  }
}
```


We also propose the SimpleMathTest JUnit test class as follows:

```
class SimpleMathTest {

  private SimpleMath myobj;

  @BeforeEach
  public void setUp() {
    myobj = new SimpleMath();
  }

  @Test
  @DisplayName("Should return true given 50")
  void fifty_isTwoDigitNumber_returnsTrue() {
    assertTrue(myobj.isTwoDigitNumber(50));
  }

  @Test
  @DisplayName("Should return false given 200")
  void twoHundred_isTwoDigitNumber_returnsFalse() {
    assertFalse(myobj.isTwoDigitNumber(200));
  }

  @Test
  @DisplayName("Should return false given -10")
  void minus10_isTwoDigitNumber_returnsFalse() {
    assertFalse(myobj.isTwoDigitNumber(-10));
  }
}
```
So let's now try to run the SimpleMathTest and the result is the following:

```
[INFO] -------------------------------------------------------
[INFO]  T E S T S
[INFO] -------------------------------------------------------
[INFO] Running en.demo.pittest.SimpleMathTest
[INFO] Tests run: 3, Failures: 0, Errors: 0, Skipped: 0, Time elapsed: 0.068 s - in en.demo.pittest.SimpleMathTest
[INFO] 
[INFO] Results:
[INFO]
[INFO] Tests run: 3, Failures: 0, Errors: 0, Skipped: 0
[INFO]
[INFO] 
[INFO] --- jacoco-maven-plugin:0.8.5:report (report) @ mutation-test-getting-started ---
[INFO] Loading execution data file C:\Users\tech-blog-articles\code\mutation-test-getting-started\target\jacoco.exec
[INFO] Analyzed bundle 'mutation-test-getting-started' with 1 classes
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time:  3.158 s
[INFO] Finished at: 2022-09-10T23:14:10+05:30
[INFO] ------------------------------------------------------------------------

```

It shows : 

 + Green: all tests passed successfully
100% code coverage: 
+ 100% line and instruction coverage by JaCoCo

![](./img/jococo_coverage.png)


###  Lets Run Mutation Test

To perform the mutation coverage we must first have the tests compiled using the "mvn test" and then run the following command:

```
mvn test pitest:mutationCoverage
```

Pitest will generate a report in HTML format in the target/pit-reports folder, here is an extract:

![](./img/mutation_pit_report.png)

We can see that five mutants were generated by PIT and only 3 out of 5 failed the unit tests (thus validating that the test case worked properly). However, 2 mutants survived therefore our test suite is not robust. We learn more about the surviving mutant by clicking individual class.

From these reports, it can be deduced that although we have 3 tests in place to test the algorithm. But the tests are not robust and a possible bug in this condition could not be detected by our JUnit tests. Hence, the developer of the test case should the report as the follow-up action item and re-create the unit test so all the mutants will be killed.

## Mutation Test Challenges

Unfortunately, the mutation test has an inherited flaw. It can become expensive in terms of computation. If the project is very large then mutation testing sometimes generates a large number of mutants. As a result, it increases test running time and requires higher computation power. Hence, we cannot continuously run mutation tests as we do with unit tests in TDD. 

In addition, false positives can also occur. Although it is not limited to mutation tests, it is necessary to check all the contents of the test results.


## Conclusion

In this article, we addressed the direct relationship between mutation tests and the quality of an existing test suite and also observed the correlation with code coverage.
Since code coverage is still an important metric, sometimes it is not satisfactory enough to ensure a well-tested code. So mutation test provides another layer of safety to the quality of our tests.

In my thought, a mutation test adds an excellent deal of value to an application because it compels developers to reduce redundancy, remove unused code and thoroughly test every aspect of the code.
