---
layout: post
title: Find amount of greater elements in the array
---

The original problem description: 
Given an array of elements, return an array of values pertaining to how many elements are greater than that element remaining in the array. 

[Source task](http://www.careercup.com/question?id=5631660689195008) 
[Git Repo](http://www.careercup.com/question?id=5631660689195008) 

The brute force solution with the O(n^2) is obvious, just create two nested loops and count all the greater elements. Is it possible to solve the problem faster? Let's think about sorting in descending order for just a second. Intuitively, exchanging elements while sorting contains the required information already. If we move some element forward, we incrementing the counter for all elements, that we have  "outruned" in the array:

![Sorting 1]({{ site.baseurl }}/images/2015-12-15/sorting1.png).

So, we need only to modify some existing sorting algorithm witn O(nlogn) complexity in order to store the required information. We'll take the merge algorithm, because it is very easy to implement and modify for this problem.

Below is the easy merge sort implementation in C#:

    public class MergeSort
    {
        public void DoMerge(int[] arr, int start, int middle, int end)
        {
            // create temp array to store sorted subarray
            int[] temp = new int[end - start + 1];
            int l = start;
            int m = middle + 1;

            int k = 0;

            // run a marker until at least one source subarrays is not empty
            while (l <= middle && m <= end)
            {
                if (arr[l] > arr[m])
                    temp[k++] = arr[l++];
                else
                    temp[k++] = arr[m++];
            }

            // take the remaining elements from the first subarray is any
            while (l <= middle)
                temp[k++] = arr[l++];

            // take the remaining elements from the second subarray is any
            while (m <= end)
                temp[k++] = arr[m++];

            // copy sorted subarray back to the array
            for (int pos = 0; pos < end - start + 1; pos++)
                arr[start + pos] = temp[pos];

        }

        public void Merge(int[] arr, int start, int end)
        {
            // if array is long enough
            if (start < end)
            {
                // divide array into two parts
                int m = (start + end) / 2;

                // sort the first half
                Merge(arr, start, m);

                // sort the second half
                Merge(arr, m + 1, end);
                
                // Merge sorted subarrays
                DoMerge(arr, start, m, end);
            }
        }
    }

First step is to rewrite the algorithms in order to preserve the source array. We do this by introducing the new int[] arrindices array, where arrindices[i] shows which element of the source array stays at position i at the moment. 

    public class MergeSortPreserveArray
    {
        public void DoMerge(int[] arr, int[] arrindices, int start, int middle, int end)
        {
            // create temp array to store sorted subarray
            int[] temp = new int[end - start + 1];
            int l = start;
            int m = middle + 1;

            int k = 0;

            // run a marker until at least one source subarrays is not empty
            while (l <= middle && m <= end)
            {
                if (arr[arrindices[l]] >= arr[arrindices[m]])
                    temp[k++] = arrindices[l++];
                else
                    temp[k++] = arrindices[m++];
            }

            // take the remaining elements from the first subarray is any
            while (l <= middle)
                temp[k++] = arrindices[l++];

            // take the remaining elements from the second subarray is any
            while (m <= end)
                temp[k++] = arrindices[m++];

            // copy sorted subarray of indices back to the indices array
            for (int pos = 0; pos < end - start + 1; pos++)
                arrindices[start + pos] = temp[pos];
        }

        public void Merge(int[] arr, int[] arrindices, int start, int end)
        {
            // if array is long enough
            if (start < end)
            {
                // divide array into two parts
                int m = (start + end) / 2;

                // sort the first half
                Merge(arr, arrindices, start, m);

                // sort the second half
                Merge(arr, arrindices, m + 1, end);

                // Merge sorted subarrays
                DoMerge(arr, arrindices, start, m, end);
            }
        }
    }

At the last step, we introduce int[] resarray to store the number of elements, greater than that element remaining in the array.
The last modification is beautiful, but a little bit tricky. Let's consier take a single Merge step above:

            // run a marker until at least one source subarrays is not empty
            while (l <= middle && m <= end)
            {
                if (arr[arrindices[l]] > arr[arrindices[m]])
                    temp[k++] = arrindices[l++];
                else
                    temp[k++] = arrindices[m++];
            }

If we take the element from the second array (else clause), it is by default greater then all remaining elements from the first array. So, we have to increment the resulting array for all such elements by one. To avoid introducing another loops in code, we'll introduce the counter of the elements, that had been moved from the second array, and will add this counter to each element of the first array, when it is taken to the result:

            // store the amount of grea
            int gr = 0;

            // run a marker until at least one source subarrays is not empty
            while (l <= middle && m <= end)
            {
                if (arr[arrindices[l]] >= arr[arrindices[m]])
                {
                    temp[k] = arrindices[l];
                    
                    // increment the result array by the number of already found greater elements
                    resarray[arrindices[l]] += gr;
                    l++;
                    k++;
                }
                else
                {
                    // increment the amount of "greater" elements
                    gr++;
                    temp[k] = arrindices[m];
                    m++;
                    k++;
                }
            }

Please note, we do not need to make anything with the elements of the second subarray. Since it is already sorted, all such increments has been already added to the resarray. 

Then not to forget to increment the result array when we take the remaining elements from the first array and we are ready:


            // take the rest of the first subarray is any
            while (l <= middle)
            {
                temp[k] = arrindices[l];

                // increment the result array by the number of already found greater elements
                resarray[arrindices[l]] += gr;
                k++;
                l++;
            }

The complete code can be found below:

        public void DoMerge(int[] arr, int[] arrindices, int[] resarray, int start, int middle, int end)
        {
            // create temp array to store sorted subarray
            int[] temp = new int[end - start + 1];
            int l = start;
            int m = middle + 1;

            int k = 0;

            // store the amount 
            int gr = 0;

            // run a marker until at least one source subarrays is not empty
            while (l <= middle && m <= end)
            {
                if (arr[arrindices[l]] >= arr[arrindices[m]])
                {
                    temp[k] = arrindices[l];

                    // increment the result array by the number of already found greater elements
                    resarray[arrindices[l]] += gr;
                    l++;
                    k++;
                }
                else
                {
                    // increment the amount of "greater" elements
                    gr++;
                    temp[k] = arrindices[m];
                    m++;
                    k++;
                }
            }

            // take the remaining elements from the first subarray is any
            while (l <= middle)
            {
                temp[k] = arrindices[l];

                // increment the result array by the number of already found greater elements
                resarray[arrindices[l]] += gr;
                k++;
                l++;
            }

            // take the remaining elements from the second subarray is any
            while (m <= end)
            {
                temp[k++] = arrindices[m++];
            }

            // copy sorted subarray of indices back to the indices array
            for (int pos = 0; pos < end - start + 1; pos++)
                arrindices[start + pos] = temp[pos];
        }

        public void Merge(int[] arr, int[] arrindices, int[] resarray, int start, int end)
        {
            // if array is long enough
            if (start < end)
            {  
                // divide array into two parts
                int m = (start + end) / 2;

                // sort the first half
                Merge(arr, arrindices, resarray, start, m);

                // sort the second half
                Merge(arr, arrindices, resarray, m + 1, end);

                // Merge sorted subarrays
                DoMerge(arr, arrindices, resarray, start, m, end);
            }

