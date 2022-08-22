# CS50 Week 3 Problem Set Writeup - Plurality

## Problem Description
> Elections come in all shapes and sizes. In the UK, the Prime Minister is officially appointed by the monarch, who generally chooses the leader of the political party that wins the most seats in the House of Commons. The United States uses a multi-step Electoral College process where citizens vote on how each state should allocate Electors who then elect the President.

>Perhaps the simplest way to hold an election, though, is via a method commonly known as the “plurality vote” (also known as “first-past-the-post” or “winner take all”). In the plurality vote, every voter gets to vote for one candidate. At the end of the election, whichever candidate has the greatest number of votes is declared the winner of the election.

[Source](https://cs50.harvard.edu/x/2022/psets/3/plurality/#background)

## Specification

> Complete the implementation of `plurality.c` in such a way that the program simulates a plurality vote election.

> Complete the `vote` function.
> - `vote` takes a single argument, a `string` called `name`, representing the name of the candidate who was voted for.
>  - If `name` matches one of the names of the candidates in the election, then update that candidate’s vote total to account for the new vote. The `vote` function in this case should return `true` to indicate a successful ballot.
> - If `name` does not match the name of any of the candidates in the election, no vote totals should change, and the `vote` function should return `false` to indicate an invalid ballot.
> - You may assume that no two candidates will have the same name.

> Complete the `print_winner` function.
> - The function should print out the name of the candidate who received the most votes in the election, and then print a newline.
> - It is possible that the election could end in a tie if multiple candidates each have the maximum number of votes. In that case, you should output the names of each of the winning candidates, each on a separate line.

> You should not modify anything else in `plurality.c` other than the implementations of the `vote` and `print_winner` functions (and the inclusion of additional header files, if you’d like).

[Source](https://cs50.harvard.edu/x/2022/psets/3/plurality/#specification)

## Setup

Upon using `wget https://cdn.cs50.net/2021/fall/psets/3/plurality.zip` to pull the challenge files from Harvard's server and unzipping its contents we are presented with a file called `plurality.c`. Opening it in VSCode we can see the following boilerplate 

```c
#include <cs50.h>
#include <stdio.h>
#include <string.h>

// Max number of candidates
#define MAX 9

// Candidates have name and vote count
typedef struct
{
    string name;
    int votes;
}
candidate;

// Array of candidates
candidate candidates[MAX];

// Number of candidates
int candidate_count;

// Function prototypes
bool vote(string name);
void print_winner(void);

int main(int argc, string argv[])
{
    // Check for invalid usage
    if (argc < 2)
    {
        printf("Usage: plurality [candidate ...]\n");
        return 1;
    }

    // Populate array of candidates
    candidate_count = argc - 1;
    if (candidate_count > MAX)
    {
        printf("Maximum number of candidates is %i\n", MAX);
        return 2;
    }
    for (int i = 0; i < candidate_count; i++)
    {
        candidates[i].name = argv[i + 1];
        candidates[i].votes = 0;
    }

    int voter_count = get_int("Number of voters: ");

    // Loop over all voters
    for (int i = 0; i < voter_count; i++)
    {
        string name = get_string("Vote: ");

        // Check for invalid vote
        if (!vote(name))
        {
            printf("Invalid vote.\n");
        }
    }

    // Display winner of election
    print_winner();
}

// Update vote totals given a new vote
bool vote(string name)
{
    // TODO
    return false;
}

// Print the winner (or winners) of the election
void print_winner(void)
{
    // TODO
    return;
}
```

## Code Analysis

The `main` function can be divided into a few steps:
 1. Validate the command line arguments passed into the application.
 2. Store theses command line arguments into an array called `candidates`, alongside declaring a variable called `candidate_count` storing the number of candidates.
 3. Ask the user how many voters there are.
 4. For every voter:
    1. Ask the user for a vote.
    2. Validate this vote with  the `vote` function, passing in the user provided input as the parameter.
 5. Calculate the winner and show it on the screen with `print_winner`

We can see that the `vote` function is meant to take a user provided input (`name`), verify its validity and update the vote tally if it is a valid vote, it must then return `true` or `false` based on whether the update was successful or not. The `print_winner` function must simply find the candidate or candidates with the most votes, and print them to the screen.

## `vote` Function

To implement the vote function, we must first validate that the user provided (which will henceforth be referred to as `name`). The first step should therefore be to check if `name` exists in the `candidates` array. For the sake of keeping my code readable I decided to create a function to help with this called `candidate_index`. 

`candidate_index` takes a string parameter (which we will call `n`) and returns an `int` representing the index of `n` in the `candidates` array, if `n` could not be found in the array, we simply return a value of `-1`

```c
int candidate_index(string n)
{
    for (int i = 0; i < candidate_count; i++)
    {
        if (strcmp(candidates[i].name, n) == 0)
        {
            return i;
        }
    }
    return -1;
}
```

The `vote` function simply gets the index of `name`, if it is `-1`, it simply returns `false` to indicate that the vote tally could not be updated, as the input was invalid, if the index is anything other than `-1`, it increases that candidates vote tally by one and returns `true`

```c
bool vote(string name)
{
    int index = candidate_index(name);
    if (index != -1)
    {
        candidates[index].votes++;
        return true;
    }
    return false;

}
```

## `print_winner` Function

The `print_winner` must calculate the winner or winners and print them out to the screen, while I am sure that there are many ways that one can approach this problem, the solution I came up with is as follows:
 1. Take the list of candidates and sort them in descending order.
 2. We'll create a variable `n` with value 0.
 3. Take the `n`th element of this list and print it to the screen, we are certain this is ***a*** winner, but we are not sure if it is the ***only*** winner.
 4. If `n` is equal to `candidate_count - 1`, stop the execution, as this means we have reached the end of the array.
 5. To make sure there aren't any other candidates tied, we check the `n + 1`th element ...
    * If it is equal to the `n`th element, we set `n` to `n + 1` and repeat step 3.
    * If it is not, we know that the `n`th element is the final winner, and we stop there.

To implement this process I created 2 additional functions:
 - `sort_candidates`, a simple implementation of bubble sort which sorts the `candidates` array.    
 
```c
void sort_candidates(void)
{
    for (int i = 0; i < candidate_count; i++)
    {
        for (int j = 0; j < candidate_count - 1; j++)
        {
            if (candidates[j].votes < candidates[j + 1].votes)
            {
                candidate temp = candidates[j];
                candidates[j] = candidates[j + 1];
                candidates[j + 1] = temp;
            }
        }
    }
}
```

 - `print_winners_from_array`, a function that implements steps two through five, this function takes an `index` parameter which specifies the current value of `n`.

 ```c
void print_winners_from_array(int index)
{
    // Step 3
    printf("%s\n", candidates[index].name);
    
    // Step 4
    if (index == candidate_count - 1)
    {
        return;
    }

    // Step 5
    if (candidates[index].votes == candidates[index + 1].votes)
    {
        print_winners_from_array(index + 1);
    }
}
```

The `print_winner`  function simply calls `sort_candidates` and `print_winners_from_array` with an initial value of 0.

```c
void print_winner(void)
{
    sort_candidates();
    print_winners_from_array(0);
    return;
}
```

# Source Code

```c
#include <cs50.h>
#include <stdio.h>
#include <string.h>

// Max number of candidates
#define MAX 9

// Candidates have name and vote count
typedef struct
{
    string name;
    int votes;
}
candidate;

// Array of candidates
candidate candidates[MAX];

// Number of candidates
int candidate_count;

// Function prototypes
bool vote(string name);
void sort_candidates(void);
void print_winner(void);

int main(int argc, string argv[])
{
    // Check for invalid usage
    if (argc < 2)
    {
        printf("Usage: plurality [candidate ...]\n");
        return 1;
    }

    // Populate array of candidates
    candidate_count = argc - 1;
    if (candidate_count > MAX)
    {
        printf("Maximum number of candidates is %i\n", MAX);
        return 2;
    }
    for (int i = 0; i < candidate_count; i++)
    {
        candidates[i].name = argv[i + 1];
        candidates[i].votes = 0;
    }

    int voter_count = get_int("Number of voters: ");

    // Loop over all voters
    for (int i = 0; i < voter_count; i++)
    {
        string name = get_string("Vote: ");

        // Check for invalid vote
        if (!vote(name))
        {
            printf("Invalid vote.\n");
        }
    }

    // Display winner of election
    print_winner();
}

int candidate_index(string n)
{
    for (int i = 0; i < candidate_count; i++)
    {
        if (strcmp(candidates[i].name, n) == 0)
        {
            return i;
        }
    }
    return -1;
}

// Update vote totals given a new vote
bool vote(string name)
{
    int index = candidate_index(name);
    if (index != -1)
    {
        candidates[index].votes++;
        return true;
    }
    return false;

}

void sort_candidates(void)
{
    for (int i = 0; i < candidate_count; i++)
    {
        for (int j = 0; j < candidate_count - 1; j++)
        {
            if (candidates[j].votes < candidates[j + 1].votes)
            {
                candidate temp = candidates[j];
                candidates[j] = candidates[j + 1];
                candidates[j + 1] = temp;
            }
        }
    }
}

void print_winners_from_array(int index)
{
    printf("%s\n", candidates[index].name);
    if (index == candidate_count - 1)
    {
        return;
    }

    if (candidates[index].votes == candidates[index + 1].votes)
    {
        print_winners_from_array(index + 1);
    }
}

// Print the winner (or winners) of the election
void print_winner(void)
{
    sort_candidates();
    print_winners_from_array(0);
    return;
}
```
