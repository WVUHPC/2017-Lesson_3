---
title: "Advanced Shell/Python Scripting"
teaching: 45
exercises: 15
questions:
- "Key question"
objectives:
- "First objective."
keypoints:
- "First key point."
---

## Advanced Bash/Python Scripting

Here we will cover a few more advanced topics not covered on our introductory scripting session. The examples here are oriented to usual operations that need to be done with the text output from several research codes.

### Extracting text from output files (awk and grep)

Consider the data at \verb|Day3_AdvancedTopics/1.Scripting|.
There is a compressed file called \texttt{goldnano3cu.tgz}.
Uncompress the tar file with

~~~
tar -zxvf goldnano3cu.tgz
~~~
{: .source}

The folders and files are from actual simulations of gold nanoclusters. You can see a sample from the first 10 simulations. The actual simulation was done with actually several hundred files like those.

First, have a look to one of those files. For example \texttt{goldnano3cu/0/output.log}. Use \texttt{less} or your favorite text editor to get an idea about how the file actually looks. With less you can search for words typing \verb|/| and the pattern. Search for etot on that file.

The first challenge is to know the value of the total energy per atom. You can see that such information is contained on lines such as

~~~
Time step =    232 SCF step =   3 etot/atom =    -277.072096
~~~
{: .source}

So you can use grep to extract those lines from the output

~~~
grep "etot/atom" goldnano3cu/0/output.log
~~~
{: .source}

grep is a tool to extract lines from text output using patterns to identify the lines that we need to be shown.

The output will look like this

~~~
...
   Time step =    251 SCF step =   3 etot/atom =    -277.070767
   Time step =    252 SCF step =   3 etot/atom =    -277.070811
   Time step =    253 SCF step =   3 etot/atom =    -277.070817
   Time step =    254 SCF step =   4 etot/atom =    -277.070855
   Time step =    255 SCF step =   3 etot/atom =    -277.070890
   Time step =    256 SCF step =   3 etot/atom =    -277.070922
   Time step =    257 SCF step =   3 etot/atom =    -277.070953
   Time step =    258 SCF step =   3 etot/atom =    -277.070978
~~~
{: .source}

You can see the file again and noticing that after that line there are more information that could be interesting to see.
Execute for example:

~~~
grep -A 9 "etot/atom" goldnano3cu/0/output.log
~~~
{: .source}

Now, we are capturing the lines with \verb|'etot/atom'| and the next 9 lines AFTER. You can also use \texttt{-B} to get lines before the pattern.

~~~
grep -B 1 -A 9 "etot/atom" goldnano3cu/0/output.log
~~~
{: .source}

The pattern so far is just a fixed string. grep was created to use far more complex patterns called regular expressions.

Consider for example that we are searching for lines with the word "Time"

~~~
grep "Time" goldnano3cu/0/output.log
~~~
{: .source}

But, now we realize that we do not want those lines that start with EBS, so we can indicate grep that the pattern matches only the word "Time" at the beggining of the line with some arbitrary number of spaces.

~~~
grep "^ *Time" goldnano3cu/0/output.log
~~~
{: .source}

The following list shows the basic elements for regular expressions.

~~~
. (dot) - a single character.
?       - the preceding character matches 0 or 1 times only.
*       - the preceding character matches 0 or more times.
+       - the preceding character matches 1 or more times.
{n}     - the preceding character matches exactly n times.
{n,m}   - the preceding character matches at least n times        
          and not more than m times.
[agd]   - the character is one of those included
          within the square brackets.
[^agd]  - the character is not one of those included
          within the square brackets.
[c-f]   - the dash within the square brackets operates
          as a range. In this case it means either the
          letters c, d, e or f.
()      - allows us to group several characters to behave
          as one.
|       - the logical OR operation.
^       - matches the beginning of the line.
$       - matches the end of the line.
~~~
{: .source}

Regular expressions is whole subject by itself and widely used not only by grep, but also many other languages such as awk, perl and python. We will continue the exploration using AWK.

AWK is a programming language designed for text processing and typically used as a data extraction and reporting tool.
In the 1990s, Perl became very popular, competing with AWK in the niche of Unix text-processing languages. Nowadays is Python the most popular language for many purposes including text processing, however for some basic operations is far easier to use a awk line or small script rather than a python one.

Lets go back to our grep command for extracting the \verb|'etot/atom'|, lets use for example a line like this:

~~~
grep "^ *Time" goldnano3cu/0/output.log
~~~
{: .source}

Lets suppose that we would like to get just the number of the iteration and the energy, for example to plot the two columns later on. You can use grep and awk connected with a pipe to produce the result. For example

~~~
grep "^ *Time" goldnano3cu/0/output.log | awk '{print $4, $11}'
~~~
{: .source}

What we are doing here is letting grep to extract the lines that we want and using awk to just print the 4th and 11th column from those lines. Notice the \verb|","| after \verb|$4| otherwise the two numbers will be stick together.

This is a very popular combination, using grep for line extraction and awk to print the output needed.

AWK is able to reproduce the grep execution by itself, for example:

~~~
awk '(/etot\/atom/) {print $0}' goldnano3cu/0/output.log
~~~
{: .source}

And the extraction of the iteration number and energy can be done with:

~~~
awk '(/etot\/atom/) {print $4, $11}' goldnano3cu/0/output.log
~~~
{: .source}

One of the advantages of AWK is that variables are converted on the fly based on use. That allow us to change the value of the energy if we need for example converted to different units. For example:

~~~
awk '(/etot\/atom/) {n=$11; print n*0.0367493, "Ha"}' goldnano3cu/0/output.log
~~~
{: .source}

This line also shows how you can create variables and use it them to manipulate the output.

At this point is good to see AWK as an old fashioned C-like scripting language. Take for example this script:

~~~
#!/bin/awk -f
BEGIN {
# Print the squares from 1 to 10 the first way
    i=1;
    printf("Squares:\n")
    while (i <= 10) {
	printf("The square of %3d is %3d\n", i, i*i);
	i = i+1;
    }
# do it again, using more concise code
    printf("Cubes:\n")
    for (i=1; i <= 10; i++) {
	printf("The cube of %3d is %4d\n", i, i**3);
    }
# now end
    exit;
}
~~~
{: .source}

Lets go back to the output of the simulation. Imagine now that you would like to know the minimum, maximum and average energy per atom. The next script in AWK will do that:

~~~
#/usr/bin/awk -f
BEGIN {
    emin=10000;
    emax=-10000;
    eavg=0.0
    nele=0
}
(/etot\/atom/){
    etot_atom = $11
    if ( etot_atom < emin ) {
	emin = etot_atom;
    }
    if ( etot_atom > emax ) {
	emax = etot_atom;
    }
    eavg+=etot_atom;
    nele+=1
}
END {
    printf("Minimum: %f Maximum: %f Average: %f\n", emin, emax, eavg/nele);
}
~~~
{: .source}

You can use it like this:

~~~
awk -f awk_min_max_avg.awk goldnano3cu/0/output.log
~~~
{: .source}

In fact you can go further and process all the outputs from the 10 simulations like this:

~~~
awk -f awk_min_max_avg.awk goldnano3cu/*/output.log
~~~
{: .source}

AWK is actually reading more than 7 million lines of output, extracting the relevant lines and computing the maximum, minimum and average from them.

### Regular Expressions with python

Consider the following challenge.
We have the output from a simulation with some data that we would like to process, the problem now is that the data is not on a single like, so a simple grep will not work. The data we want to parse looks like this:

~~~
...
     14       7.7300      0.00000
     15       7.9145      0.00000
     16       8.7421      0.00000

 k-point 115 :       0.4444    0.3636    0.4286
  band No.  band energies     occupation
      1      -6.7076      1.00000
      2      -6.6256      1.00000
      3      -3.8932      1.00000
      4      -3.8031      1.00000
      5       0.1344      1.00000
      6       0.4871      1.00000
      7       0.7520      1.00000
      8       1.1131      1.00000
      9       3.2272      1.00000
     10       3.3574      1.00000
     11       7.4689      0.00000
     12       7.4905      0.00000
     13       7.7325      0.00000
     14       7.9343      0.00000
     15       8.3742      0.00000
     16       8.7648      0.00000

 k-point 116 :       0.0000    0.4545    0.4286
  band No.  band energies     occupation
      1      -7.0118      1.00000
      2      -6.8668      1.00000
      3      -4.4179      1.00000
...
~~~
{: .source}

This is quite complex set of data and we would like to take the different elements in such a way that we can manipulate them later on.

There are several ways of solving this problem, for example, knowing that each block of data starts with "k-point"
 and spans 17 rows. Such task could be done using just grep

~~~
grep -A 17 k-point OUTCAR
~~~
{: .source}

The argument "-A 17" will tell grep to show 17 rows after each occurrence of the line k-point.
We extract the line of information that we need but still is just a piece of text that is not so simple to manipulate.

With python we can achieve this task with just 4 lines, using the so called regular expressions, a way to explain a computer that we want extract text with some format by indicating the kind of data that we expect on the text.

This following script will extract the pieces still as text but, we will work on the conversion to text later.

~~~
import re
rf = open('OUTCAR')
data = rf.read()
kp = re.findall('k-point([\d\s]*):([\d\s.]*)band[\.\s\w]*occupation([\s\d:\-\.]*)\n\n', data)
~~~
{: .source}

The most cryptic part of this small script is understanding the meaning of all those symbols used as arguments for the findall function.
Lets start with a simpler version of the findall line an we will understand it piece by piece.

Using IPython lets start with executing the first 3 lines and we will explore the findall function step by step

~~~
import re
rf = open('OUTCAR')
data = rf.read()
~~~
{: .source}

Now, we start exploring this line

~~~
re.findall('k-point[\d\s]*:', data)
~~~
{: .source}

The output will look like this:

~~~
['k-point  1 :',
 'k-point  2 :',
 'k-point  3 :',
 'k-point  4 :',
~~~
{: .source}

The line in findall can be read like this: Search for text that start with `k-point` followed by 0 or more (that is the meaning of `*`) groups of characters (what is enclose by `[` and `]`) that can be either numbers `\d` or characters that looks like spaces `\s`

Now, if we just need the number, we can enclose the information that findall will return by enclosing it in parenthesis, like this:

~~~
re.findall('k-point([\d\s]*):', data)
~~~
{: .source}

At this point could be interesting to show how we can convert the list of strings returned by findall into actual numbers.
This could be done like this:

~~~
[int(x) for x in re.findall('k-point([\d\s]*):', data)]
~~~
{: .source}

Now lets move forward and get the next piece of information, the three numbers after colon, the numbers before the word 'band'

~~~
re.findall('k-point([\d\s]*):([\d\s.]*)band', data)
~~~
{: .source}

As you can see we are getting more information this time

~~~
[('   1 ', '       0.0000    0.0000    0.0000\n  '),
 ('   2 ', '       0.1111    0.0000    0.0000\n  '),
 ('   3 ', '       0.2222    0.0000    0.0000\n  '),
 ('   4 ', '       0.3333    0.0000    0.0000\n  '),
 ('   5 ', '       0.4444    0.0000    0.0000\n  '),
 ('   6 ', '       0.0000    0.0909    0.0000\n  '),
 ('   7 ', '       0.1111    0.0909    0.0000\n  '),
 ('   8 ', '       0.2222    0.0909    0.0000\n  '),
...
~~~
{: .source}

The output is a list of tuples, each tuple consisting of two strings. There is just one extra character on the regular expression, dot `.` is added to cover the existence of that character in the 3 numbers after colon. In regular expressions "dot" is used to match any character except a newline. Inside the `[]` special characters lose their special meaning, so `dot` here means just a `.`.

Now we can go to our final version of the findall function

~~~
re.findall('k-point([\d\s]*):([\d\s.]*)band[\s\w.]*occupation([\s\d:.\-]*)\n\n', data)
~~~
{: .source}

The meaning of all this cryptic code become far more clear now, the only notice here is that the character minus `-` needs still to be escaped like `\-` because it has a meaning for ranges inside `[]`. We close the regular expression with a double `\n\n`, indicating that each block is separated by a double newline.

Out final task is to convert the output from findall into actual numbers such that we can manipulate them for whatever purpose we need.

Lets do first a more simple exercise by storing correctly the k-point number and the three numbers after colon, they are the positions but their actual meaning is not important here.

~~~
ret=[]
for ikp in kp:
    entry={}
    entry['number']= int(ikp[0])
    entry['position']= [float(x) for x in ikp[1].split()]
    entry['values']= len(ikp[2].split())
    ret.append(entry)
~~~
{: .source}

What we are doing here is creating a list called \textbf{ret}
and for each element in our list kp we will create a python dictionary, converting the elements from the tuple into numbers, the first one will be integer, the second one is a set of three floating point numbers and for the third one we will just split the string into words and count the elements.

~~~
[{'number': 1, 'position': [0.0, 0.0, 0.0], 'values': 48},
 {'number': 2, 'position': [0.1111, 0.0, 0.0], 'values': 48},
 {'number': 3, 'position': [0.2222, 0.0, 0.0], 'values': 48},
 {'number': 4, 'position': [0.3333, 0.0, 0.0], 'values': 48},
 {'number': 5, 'position': [0.4444, 0.0, 0.0], 'values': 48},
 {'number': 6, 'position': [0.0, 0.0909, 0.0], 'values': 48},
...
~~~
{: .source}

For the position we use a list comprehension, a syntactic construct available in some programming languages for creating a list based on existing lists.

The conversion of values is a bit more elaborated. First, notice that the final element contain 49 elements due to a final string with several dashes. We would like to extract the numbers that are really relevant the floating point numbers. Lets consider just the final element from kp

~~~
In [44]: kp[-1]
Out[44]:
(' 120 ',
 '       0.4444    0.4545    0.4286\n  ',
 ' \n      1      -6.6226      1.00000\n      2      -6.5937      1.00000\n      3      -3.7536      1.00000\n      4      -3.7218      1.00000\n      5      -0.0498      1.00000\n      6       0.0803      1.00000\n      7       0.5565      1.00000\n      8       0.6896      1.00000\n      9       3.3146      1.00000\n     10       3.3603      1.00000\n     11       7.6585      0.00000\n     12       7.6740      0.00000\n     13       7.9721      0.00000\n     14       8.0356      0.00000\n     15       8.8014      0.00000\n     16       8.9382      0.00000\n\n\n--------------------------------------------------------------------------------------------------------\n')
~~~
{: .source}

Using Numpy we can easily get the information converted ready easily. Consider this line

~~~
import numpy
np.array(kp[-1][2].split()[:48], dtype=float).reshape(-1,3)
~~~
{: .source}

This line can be readed like this. Take the last element in kp (kp[-1]). Now take the third element of the tuple (`kp[-1][2]`). Split the string in words and make a list with the first 48 words encountered

~~~
kp[-1][2].split()[:48]
~~~
{: .source}

The final step is to convert those 48 strings into numbers as floating point numbers and reshape the whole array in 3 columns.

The final version of this part of the script will look like this:

~~~
ret=[]
for ikp in kp:
    entry={}
    entry['number']= int(ikp[0])
    entry['position']= [float(x) for x in ikp[1].split()]
    entry['values']= np.array(ikp[2].split()[:48], dtype=float).reshape(-1,3)
    ret.append(entry)
~~~
{: .source}

For reasons that will become clearer later we would like to keep everything as simple lists of numbers rather than numpy arrays. So we will serialize the numpy array into a list of lists

~~~
ret=[]
for ikp in kp:
    entry={}
    entry['number']= int(ikp[0])
    entry['position']= [float(x) for x in ikp[1].split()]
    entry['values']= np.array(ikp[2].split()[:48], dtype=float).reshape(-1,3).tolist()
    ret.append(entry)
~~~
{: .source}

It is time for us to save the data that we parse in something that allow us to recover later. There are several ways to store python objects into files. One way is using JSON, another is using pickle

Right now, the variable ret is a list of dictionaries where each of them contains either single numbers, lists or lists of lists. We can store that in a JSON file such that we can recover that information easily.

JSON is a lightweight data interchange format inspired by JavaScript object literal syntax.
The JSON module in python offers convenient functions to convert simple variables such as list and dictionaries into strings that could be stored in text files such that their contents could be easily retrieved.

Try first executing something like this

~~~
import json
json.dumps(ret)
~~~
{: .source}

Not easy to read for a human but that long string can be easily understood by a computer to recover the information you stored in it. Try this version for something clearer to read

~~~
import json
json.dumps(ret, sort_keys=True, indent=4, separators=(',', ': '))
~~~
{: .source}

Lets now store ret into a file and read it again to test we can recover the file.

~~~
wf = open('k-points.json','w')
dp = json.dump(ret, wf, sort_keys=True, indent=4, separators=(',', ': '))
wf.close()
~~~
{: .source}

Now lets test recovering the data from the file.

~~~
rf2=open('k-points.json')
json.load(rf2)
~~~
{: .source}

Finally, lets summarize all that we learn with this example.
The whole script will be listed here:

~~~
#!/usr/bin/env python

import re
import numpy as np
import json

rf = open('OUTCAR')
data = rf.read()

# Parsing of the data
kp=re.findall('k-point([\d\s]*):([\d\s.]*)band[\s\w.]*occupation([\s\d:.\-]*)\n\n', data)

# Giving structure to the data
ret=[]
for ikp in kp:
    entry={}
    entry['number']= int(ikp[0])
    entry['position']= [float(x) for x in ikp[1].split()]
    entry['values']= np.array(ikp[2].split()[:48], dtype=float).reshape(-1,3).tolist()
    ret.append(entry)

# Storing the results into a JSON file
wf = open('k-points.json','w')
dp = json.dump(ret, wf, sort_keys=True, indent=4, separators=(',', ': '))
wf.close()
~~~
{: .source}


{% include links.md %}
