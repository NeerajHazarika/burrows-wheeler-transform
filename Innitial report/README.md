# Burrows-Wheeler Transform (BWT)

The Burrows-Wheeler transform (BWT) is an algorithm that takes blocks of data, such as strings, and rearranges them into runs of similar characters. After the transformation, the output block contains the same exact data elements before it had started, but differs in the ordering. The nature of the algorithm tends to put similar characters next to each other, making the resulting data order easier to compress. Hence it is used in many compression algorithms.

The Burrows-Wheeler transform algorithm is a relatively new algorithm invented in 1994 by Michael Burrows and David Wheeler and based on an unpublished transformation discovered by Wheeler in 1983, published in their paper “A Block-sorting Lossless Data Compression Algorithm.”

The algorithm is very simple, you just sort all the rotations lexicographically and take the last column of it. This last column is the Burrows-Wheeler transform of the string. An example will make it clear. Let our string be “PANAMABANANA”. We will add a $ at the end of the string, this end character should not be anywhere in the original string. (Considering $ to be the smallest character).

<p align="center"><img src="https://user-images.githubusercontent.com/72177954/132737030-e38d2694-1f02-4f02-a99d-c9bb3f3c91dd.png"></p>
<p align="center">Steps of BWT for PANAMABANANA</p>

The final transformed string we get is “ANMNNBPAAAAA$”. This is the BWT of our original string. Compression? Here, we can see that we get chunks of sthe ame characters, like 5 A’s. We can store it as “ANM2NBP5A$”. For some strings we can get even more characters together in a continuous run, resulting in better compression. Great! So, do we always get same characters together? NO. For example:

<p align="center"><img src="https://user-images.githubusercontent.com/72177954/132737158-387004b4-99b6-4ffb-90e1-8139a739433f.png"></p>
<p align="center">BWT with no characters together</p>

“AC$BCDABDBADC”. Here none of the characters is together. Nothing to compress. Worse than the original string. Let’s see another example.

<p align="center"><img src="https://user-images.githubusercontent.com/72177954/132737275-66e39c0d-233d-46bc-a5dc-d626ed3dece8.png"></p>
<p align="center">BWT resulting in good compression</p>

“3C$3A3B”. Isn’t it nice? Hmm… So when do we get repeated characters? Well, if there are substrings which occur often in the string (like in “ABCABCABC”), then there will be more characters together. 

# Inverting the Burrows-Wheeler Transform
All the information required for getting back the original string from it’s BWT is available in the BWT itself (the last column). First, let’s look at an important fact about the first column and last column(BWT) of the sorted rotations.

`For any character x, the ith x in the first column corresponds to the ith x in the last column!`

For example, in the “PANAMABANANA” example, the 2nd A in first column (3rd row) and the 2nd A (8th row) in last column are the same A. Similarly, the 3rd N in first column and 3rd N in last column is same.

<p align="center"><img src="https://user-images.githubusercontent.com/72177954/132735711-b8ca5aeb-0acd-4e7a-a83e-ab239ca98ae0.png"></p>

This will hold true for any character, always. You can verify yourself why that’s true.

Now to get back our string, we can start tracing character by character using last column(BWT) and first column. We can get the first column by sorting the last column (BWT of the string) (actually, you just need to count the occurrence of each character). We will start tracing from $ since we know that it’s the last character. For ith x in last column, we will go to ith x in first column. Record that character. Get the corresponding indexed character in last column and repeat. Keep doing till we hit the $.

Let’s do it on a small example, “RAGA”. First it’s transformation:

<p align="center"><img src="https://user-images.githubusercontent.com/72177954/132735872-52f08048-9145-4a15-b8f7-50a5dcdc899b.png"></p>
<p align="center">BWT of RAGA</p>

Now it’s inverse transformation. Start by $. This is the first $ in the last column, so we go to the first $ in the first column, i.e. first row. Refer to the figure below. We get an A here. This is the first A in last column, so we go to first A in first column. Corresponding to it we get G. This is the first G, so we go to first G in first column, for which we get an A. This is second A in last column, going to second A in first column, we get R. This is the first R in last column, we go to first R in first column. We get a $ and we stop. This image will make the steps clear:

<p align="center"><img src="https://user-images.githubusercontent.com/72177954/132736087-98126b91-2291-47c9-a8ed-a8b7477a03ae.jpg"></p>
<p align="center">Burrows-Wheeler Inverse</p>

Tada! We got back our original string “RAGA”!

BWT is used in bzip and bzip2 for compression. BWT is also used for read alignment, where you have a lot of short query strings (known as reads) and you have to find whether it’s a substring of a larger reference string. That’s called Burrows-Wheeler Alignment.

# C code implementation
```
// C program to find Burrows Wheeler transform
// of a given text

#include <stdio.h>
#include <stdlib.h>
#include <string.h>

// Structure to store data of a rotation
struct rotation {
	int index;
	char* suffix;
};

// Compares the rotations and
// sorts the rotations alphabetically
int cmpfunc(const void* x, const void* y)
{
	struct rotation* rx = (struct rotation*)x;
	struct rotation* ry = (struct rotation*)y;
	return strcmp(rx->suffix, ry->suffix);
}

// Takes text to be transformed and its length as
// arguments and returns the corresponding suffix array
int* computeSuffixArray(char* input_text, int len_text)
{
	// Array of structures to store rotations and
	// their indexes
	struct rotation suff[len_text];

	// Structure is needed to maintain old indexes of
	// rotations after sorting them
	for (int i = 0; i < len_text; i++) {
		suff[i].index = i;
		suff[i].suffix = (input_text + i);
	}

	// Sorts rotations using comparison
	// function defined above
	qsort(suff, len_text, sizeof(struct rotation),
		cmpfunc);

	// Stores the indexes of sorted rotations
	int* suffix_arr
		= (int*)malloc(len_text * sizeof(int));
	for (int i = 0; i < len_text; i++)
		suffix_arr[i] = suff[i].index;

	// Returns the computed suffix array
	return suffix_arr;
}

// Takes suffix array and its size
// as arguments and returns the
// Burrows - Wheeler Transform of given text
char* findLastChar(char* input_text,
				int* suffix_arr, int n)
{
	// Iterates over the suffix array to find
	// the last char of each cyclic rotation
	char* bwt_arr = (char*)malloc(n * sizeof(char));
	int i;
	for (i = 0; i < n; i++) {
		// Computes the last char which is given by
		// input_text[(suffix_arr[i] + n - 1) % n]
		int j = suffix_arr[i] - 1;
		if (j < 0)
			j = j + n;

		bwt_arr[i] = input_text[j];
	}

	bwt_arr[i] = '\0';

	// Returns the computed Burrows - Wheeler Transform
	return bwt_arr;
}

// Driver program to test functions above
int main()
{
	char input_text[] = "banana$";
	int len_text = strlen(input_text);

	// Computes the suffix array of our text
	int* suffix_arr
		= computeSuffixArray(input_text, len_text);

	// Adds to the output array the last char
	// of each rotation
	char* bwt_arr
		= findLastChar(input_text, suffix_arr, len_text);

	printf("Input text : %s\n", input_text);
	printf("Burrows - Wheeler Transform : %s\n",
		bwt_arr);
	return 0;
}
```

# Application of Burrows Wheeler Transform
BWT is used in bzip and bzip2 for compression. BWT turns out to be great for compressing strings with repeats, like the DNA sequences, where we have only 4 characters (A, C, G, T) and a lot of repeated patterns. There are lot of short query strings (known as reads) and you have to find whether it’s a substring of a larger reference string. That’s called Burrows-Wheeler Alignment.

# Reference 
* https://www.techopedia.com/definition/10467/burrows-wheeler-transform-bwt
* https://medium.com/@mr-easy/burrows-wheeler-transform-d475e0aacad6
* https://www.geeksforgeeks.org/burrows-wheeler-data-transform-algorithm/
