# Burrows-Wheeler Transform (BWT)

The Burrows-Wheeler transform (BWT) is an algorithm that takes blocks of data, such as strings, and rearranges them into runs of similar characters. After the transformation, the output block contains the same exact data elements before it had started, but differs in the ordering. The nature of the algorithm tends to put similar characters next to each other, making the resulting data order easier to compress. Hence it is used in many compression algorithms.


The Burrows-Wheeler transform algorithm is a relatively new algorithm invented in 1994 by Michael Burrows and David Wheeler and based on an unpublished transformation discovered by Wheeler in 1983, published in their paper “A Block-sorting Lossless Data Compression Algorithm.”

In the most basic, BWT takes a block of data such as a string, adding an EOF character and then sorting all rotations of that string into lexicographic order. The following pseudocode or steps illustrate the algorithm:

1. Create a table that contains rows that represent all possible one-increment rotations of the string.
2. Sort all rows alphabetically.
3. Output the last column of the table.

For example: the word “banana”; adding an EOF character turns it into “banana$” then we apply the algorithm:

1. Create a table with rows representing all possible rotations:
```
banana$
anana$b
nana$ba
ana$ban
na$bana
a$banan
$banana
```
2. Sort the rows alphabetically/lexicographically based on the first column:
```
$banana
a$banan
ana$ban
anana$b
banana$
nana$ba
na$bana
```
3.Return the last column as the BWT output: `annb$aa`

The resulting string is easier to compress because repeated characters are bunched up next to each other. But there needs to be additional data stored with the transformed data so that a reverse transformation can be done. Even though the resulting transformed data is larger than its original form but its compressibility characteristic is increased manyfold, essentially making it a “free” method of improving efficiency of compression methods.
