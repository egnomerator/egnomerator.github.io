## Scenario
This topic is of course not limited in application to only C#, but this is the language I was working with when I decided to spend a little time with this scenario:
  - I have a big collection (~10 million MyClass instances) and a small collection (~5 MyClass instances)
    - I'll use the term "list" instead of collection since I'm using the .NET Type `List` in the code sample
  - I want to set a `bool` property to `true` on any items in one list that *match* any in the other list

### Match Criteria
```cs
var isMatch = instanceA.Path.Equals(instanceB.Path, StringComparison.OrdinalIgnoreCase)
```

### Approach
I'll iterate over one list once and the second list many times (once for each item in the first list).
- For each item in one list (single iteration over first list), I need to know if that `item.Path` matches the `Path` property of any item in the second list (one iteration over second list for each item in first list).

### Implementation Options
These are the options I'm considering in this scenario:
1. Iterate over the large list once, iterating over the small list once per item in the large list
2. Iterate over the small list once, iterating over the large list once per item in the small list

## What's Faster?
There will be an outer loop for iterating over one list and an inner loop for iterating over the other list. Do we want the large list on the outside (iterate over it once) or on the inside (iterate over it multiple times)?

### Side Note
Any performance difference probably won't be noticeable until at least one of the lists begins to approach a large enough size depending on computing power (and there would still have to be a significant difference between each list length, of course).
- E.g. in my scenario with a list of 10 million and a list of 5 items, there was only a difference of about a second either way

## Answer
Option 2 (from above) iterate over the small list once, iterating over the large list once per item in the small list

### Explanation
- My first thought was that I'd want to iterate over the large list as few times as possible, so option 1 from above
- However, it's probably faster (at least it is in my simple scenario) to iterate 1 time over the small list, as the outer loop, and a few times over the large list, as the inner loop

**From a math perspective:**
The number of inner loops will be the same regardless of which list is the inner loop
```
var myBigListSize = 10000000;
var mySmlListSize = 5;
var innerLoops = myBigListSize * mySmlListSize;
```
So reducing the number of outer loops is the key difference. We can have 10 million outer loops or 5 for a total number of inner and outer loops of ~60 million or ~50 million.

### Side Note: LINQ `.FirstOrDefault`
LINQ `.FirstOrDefault` can potentially yield a significant performance improvement. In my sample code I've set the test data up so that my usage of `.FirstOrDefault` has minimal effect. But I left a commented line of code that would really highlight the potential performance effect of `.FirstOrDefault`.

```cs
using System;
using System.Collections.Generic;
using System.Diagnostics;
using System.Linq;

namespace Sandbox
{
    class Program
    {
        static void Main(string[] args)
        {
            var myBigList = new List<MyClass>();
            var mySmlList = new List<MyClass>();
            for (var i = 0; i < 10000000; i++)
            {
                myBigList.Add(new MyClass
                {
                    Id = i,
                    Path = string.Format(Retrieve.PathPattern, i)
                });
            }
            // for (var i = 0; i < 5; i++) // use this `for` statement to see LINQ .FirstOrDefault shine
            for (var i = 9999995; i < 10000000; i++)
            {
                mySmlList.Add(new MyClass
                {
                    Id = i,
                    Path = string.Format(Retrieve.PathPattern, i)
                });
            }

            var manyItrOverSmlOuterLoopCount = 0;
            var fewItrOverBigOuterLoopCount = 0;
            var manyItrOverSmlInnerLoopCount = 0;
            var fewItrOverBigInnerLoopCount = 0;

            var sw = new Stopwatch();

            // first timed loop structure

            sw.Start();

            myBigList.ForEach(c =>
            {
                manyItrOverSmlOuterLoopCount++;
                var matchingtrackedDataPath = mySmlList.FirstOrDefault(tdp =>
                {
                    manyItrOverSmlInnerLoopCount++;
                    return c.Path.Equals(tdp.Path, StringComparison.OrdinalIgnoreCase);
                });
                if (matchingtrackedDataPath == null) return;

                c.MatchYesNo = true;
            });

            sw.Stop();
            var swElapsedOne = sw.Elapsed;
            sw.Reset();

            // second timed loop structure

            sw.Start();

            mySmlList.ForEach(c =>
            {
                fewItrOverBigOuterLoopCount++;
                var matchingtrackedDataPath = myBigList.FirstOrDefault(tdp =>
                {
                    fewItrOverBigInnerLoopCount++;
                    return c.Path.Equals(tdp.Path, StringComparison.OrdinalIgnoreCase);
                });
                if (matchingtrackedDataPath == null) return;

                c.MatchYesNo = true;
            });

            sw.Stop();
            var swElapsedTwo = sw.Elapsed;

            Console.WriteLine("Multiple iterations over small list:");
            Console.WriteLine("Seconds: {0}", swElapsedOne.TotalSeconds);
            Console.WriteLine("     ms: {0}", swElapsedOne.TotalMilliseconds);
            Console.WriteLine("  Ticks: {0}", swElapsedOne.Ticks);
            Console.WriteLine("Outer Loops: {0}", manyItrOverSmlOuterLoopCount);
            Console.WriteLine("Inner Loops: {0}", manyItrOverSmlInnerLoopCount);
            Console.WriteLine("_______________________________________");
            Console.WriteLine("Multiple iterations over big list:");
            Console.WriteLine("Seconds: {0}", swElapsedTwo.TotalSeconds);
            Console.WriteLine("     ms: {0}", swElapsedTwo.TotalMilliseconds);
            Console.WriteLine("  Ticks: {0}", swElapsedTwo.Ticks);
            Console.WriteLine("Outer Loops: {0}", fewItrOverBigOuterLoopCount);
            Console.WriteLine("Inner Loops: {0}", fewItrOverBigInnerLoopCount);
        }
    }

    public class MyClass
    {
        public long Id { get; set; }
        public string Path { get; set; }
        public bool MatchYesNo { get; set; }
    }

    public static class Retrieve
    {
        public static string PathPattern => @"\\server\share\my\path\destination\folder{0}";
    }
}

```
## Output
![Iterations Few Over Many VS Many Over Few](Images/iterations-few-over-many-vs-many-over-few.png)