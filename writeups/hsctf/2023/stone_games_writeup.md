{\rtf1\ansi\ansicpg1252\cocoartf2580
\cocoatextscaling0\cocoaplatform0{\fonttbl\f0\fswiss\fcharset0 Helvetica;}
{\colortbl;\red255\green255\blue255;}
{\*\expandedcolortbl;;}
\margl1440\margr1440\vieww11520\viewh8400\viewkind0
\pard\tx720\tx1440\tx2160\tx2880\tx3600\tx4320\tx5040\tx5760\tx6480\tx7200\tx7920\tx8640\pardirnatural\partightenfactor0

\f0\fs24 \cf0 This is obviously a variant of the nim game, meaning that we have to put the Sprague-Grundy theorem to use. I will refer to \'93Grundy values\'94 as \'93nimbers\'94 here. We can consider each pile of stone separately, and take an xor of the nimber of each pile to find the nimber of the whole game. If this is `0`, the person who goes first loses, otherwise they win.\
\
We can first run a brute force algorithm to find the nimbers of certain pile sizes, to help find patterns. Here is a script to do so:\
```cpp\
#include <iostream>\
#include <vector>\
#include <set>\
using namespace std;\
\
const int MAX_N = 1000; // change to however high you want to brute force to, complexity O(n sqrt(n))\
\
vector<int> nimbers(MAX_N);\
\
int mex(set<int>& nims) \{\
  for (int i = 0; i <= nims.size(); ++i) \{\
    if (!nims.count(i)) \{\
      return i;\
    \}\
  \}\
  return -1;\
\}\
\
int main() \{\
  for (int i = 1; i < MAX_N; ++i) \{\
    set<int> nims;\
    for (int j = 1; j * j <= i; ++j) \{\
      nims.insert(nimbers[i - j]);\
    \}\
    nimbers[i] = mex(nims);\
  \}\
  for (int num : nimbers) cout << num << ' ';\
  cout << endl;\
  return 0;\
\}\
```\
\
This script results in the following sequence:\
```\
0 1 0 1 2 0 1 2 0 3 1 2 0 3 1 2 4 0 3 1 2 4 0 3 1 5 2 4 0 3 1 5 2 4 0 3 6 1 5 2 4 0 3 6 1 5 2 4 0 7 3 6 1 5 2 4 0 7 3 6 1 5 2 4 8 0 7 3 6 1 5 2 4 8 0 7 3 6 1 5 2 9 4 8 0 7 3 6 1 5 2 9 4 8 0 7 3 6 1 5 10 2 9 4 8\
```\
and putting a `0` on every new line results in:\
```\
\pard\tx720\tx1440\tx2160\tx2880\tx3600\tx4320\tx5040\tx5760\tx6480\tx7200\tx7920\tx8640\pardirnatural\partightenfactor0
\cf0 0 1\
0 1 2\
0 1 2\
0 3 1 2\
0 3 1 2 4\
0 3 1 2 4\
0 3 1 5 2 4\
0 3 1 5 2 4\
0 3 6 1 5 2 4\
0 3 6 1 5 2 4\
0 7 3 6 1 5 2 4\
0 7 3 6 1 5 2 4 8\
0 7 3 6 1 5 2 4 8\
0 7 3 6 1 5 2 9 4 8\
0 7 3 6 1 5 2 9 4 8\
0 7 3 6 1 5 10 2 9 4 8\
```\
Delete repeat lines for now (we can worry about them later).\
```\
0 1\
0 1 2\
0 3 1 2\
0 3 1 2 4\
0 3 1 5 2 4\
0 3 6 1 5 2 4\
0 7 3 6 1 5 2 4\
0 7 3 6 1 5 2 4 8\
0 7 3 6 1 5 2 9 4 8\
0 7 3 6 1 5 10 2 9 4 8\
```\
Each line is now a permutation, with one number inserted in every time (notice that the old numbers in the permutation stay in the same relative locations, this repetitiveness actually makes a lot of sense if you consider the \'93reach\'94 of each new line). Now we just have to find a pattern for how the new numbers are inserted. Here are the indices at which they are inserted:\
```\
1, 2, 1, 4, 3, 2, 1, 8, 7, 6, \'85\
```\
Aha! We first start inserting from thet end of the permutation, then decrease the index of insertion by one until we reach index `1`, and then we restart\'85\
\
Now we can construct the permutation of length about `sqrt(n)` in about `O(n)` time (in practice, this actually runs a LOT faster) in a trivial manner using this pattern (there may be other faster ways of constructions, but this one works).\
\
Now, we just have to find the size of each permutation in the list of nimbers to find the size of the permutation we are currently on with a certain pile size, so that we can calculate the nimber of our pile. We notice that the length of the permutation increases when we reach a new square number.\
```cpp\
vector<int> sizes(1, 2);\
int sum = 2, sz = 2;\
while (sum < MAX_X) \{\
  if (sum + sz >= sz * sz) ++sz;\
  sizes.push_back(sz);\
  sum += sz;\
\}\
```\
\
Now, we can find the size of the permutation of the current pile, and also the index of the current pile. We can iterate through the big permutation and ignore all numbers that are too big, and find the nimber. Use the following script to solve:\
```cpp\
#include <iostream>\
#include <fstream>\
#include <vector>\
#include <algorithm>\
using namespace std;\
\
const int MAX_X = 1000000000;\
\
ifstream in("input.txt");\
\
vector<int> perm(1), sizes(1, 2);\
string ans = "";\
\
void solve() \{\
  int n, x = 0;\
  in >> n;\
  for (int i = 0; i < n; ++i) \{\
    int a, sum = 0, idx = 0;\
    in >> a;\
    while (sum + sizes[idx] <= a) sum += sizes[idx++];\
    int left = a - sum;\
    for (int j = 0; j < perm.size(); ++j) \{\
      if (perm[j] >= sizes[idx]) continue;\
      if (left == 0) \{\
        x ^= perm[j];\
        break;\
      \}\
      --left;\
    \}\
  \}\
  ans += to_string(x == 0);\
\}\
\
int main() \{\
  int sum = 2, sz = 2;\
  while (sum < MAX_X) \{\
    if (sum + sz >= sz * sz) ++sz;\
    sizes.push_back(sz);\
    sum += sz;\
  \}\
  int cur = 0;\
  while (perm.size() <= sz) \{\
    if (cur == 0) cur = perm.size();\
    perm.insert(perm.begin() + cur--, perm.size());\
  \}\
  int t;\
  in >> t;\
  for (int i = 0; i < t; ++i) solve();\
  reverse(ans.begin(), ans.end());\
  cout << "flag\{" << ans << "\}" << endl;\
  return 0;\
\}\
```\
which prints the correct flag.}