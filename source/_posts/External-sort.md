---
title: External sort
date: 2020-03-30 18:46:00
tags: Algorithm
---

## Comparison sorts
|Name|Best|Average|Worst|Memory|Stable|
|---|---|---|---|---|---|
|Bubble sort|$n$|$n^2$|$n^2$|$1$|Yes|
|Insertion sort|$n$|$n^2$|$n^2$|$1$|Yes|
|Merge sort|$n\log n$|$n\log n$|$n\log n$|$n$|Yes|
|Heapsort|$n\log n$|$n\log n$|$n\log n$|$1$|No|
|Quicksort|$n\log n$|$n\log n$|$n^2$|$\log n$|No|

## External sort
[External sorting](https://en.wikipedia.org/wiki/External_sorting)

`external.cpp`:
```cpp
#include <bits/stdc++.h>
#define DBG(x) cerr << #x << " = " << x << endl

using namespace std;
typedef long long LL;

const int MAX_MEM = 128 * 1024 * 1024; // 128MB
const int BUF_SIZE = 64;

string write_temp_file(int file_no, const vector<string> &strings) {
    string temp_file = to_string(file_no) + ".txt";
    FILE *f_temp = fopen(temp_file.c_str(), "w");

    for (const auto &s : strings) {
        fputs(s.c_str(), f_temp);
    }
    fclose(f_temp);

    return temp_file;
}

vector<string> sort_part_to_disk(const string &input_file) {
    vector<string> temp_files;

    FILE *f_input = fopen(input_file.c_str(), "r");

    int temp_size = 0;
    int file_no = 0;
    char buf[BUF_SIZE];
    vector<string> strings;

    while (fgets(buf, sizeof(buf) - 1, f_input) != NULL) {
        string str = buf;
        temp_size += str.size();
        strings.push_back(str);

        if (temp_size >= MAX_MEM) {
            sort(strings.begin(), strings.end());
            string temp_file = write_temp_file(file_no, strings);
            temp_files.push_back(temp_file);

            file_no++;
            strings.clear();
            temp_size = 0;
        }
    }

    if (!strings.empty()) {
        sort(strings.begin(), strings.end());
        string temp_file = write_temp_file(file_no, strings);
        temp_files.push_back(temp_file);
    }

    fclose(f_input);

    return temp_files;
}

void external_sort(const string &input_file, const string &output_file) {
    vector<string> temp_files = sort_part_to_disk(input_file);
    vector<FILE *> files;
    for (const auto &file_name : temp_files) {
        FILE *temp_file = fopen(file_name.c_str(), "r");
        files.push_back(temp_file);
    }

    FILE *f_output = fopen(output_file.c_str(), "w");

    priority_queue<pair<string, string>, vector<pair<string, string>>, greater<pair<string, string>>> pri_que;

    bool have_input = false;
    do {
        have_input = false;

        for (int i = 0; i < temp_files.size(); i++) {
            char buf[BUF_SIZE];
            if (fgets(buf, sizeof(buf) - 1, files[i]) != NULL) {
                have_input = true;
                pri_que.push(pair<string, string>(buf, temp_files[i]));
            }
        }

        if (!pri_que.empty()) {
            pair<string, string> top = pri_que.top();
            pri_que.pop();
            fputs(top.first.c_str(), f_output);
        }

    } while (have_input || !pri_que.empty());

    for (FILE *f : files) {
        fclose(f);
    }
    fclose(f_output);
}

int main(int argc, char **argv) {
    external_sort("raw.txt", "output.txt");
    return 0;
}
```
```
 torapture@localhost ~/SSD/Codes/acm ls -lh raw.txt
-rwxrwxrwx  1 torapture  staff   685M  3 30 18:02 raw.txt

 torapture@localhost ~/SSD/Codes/acm time ./external
./external  617.04s user 17.50s system 94% cpu 11:10.99 total

 torapture@localhost ~/SSD/Codes/acm time sort ./raw.txt > sort.txt
sort ./raw.txt > sort.txt  496.36s user 87.25s system 81% cpu 11:58.01 total

 torapture@localhost ~/SSD/Codes/acm ls -lh raw.txt output.txt sort.txt
-rwxrwxrwx  1 torapture  staff   685M  3 30 18:13 output.txt
-rwxrwxrwx  1 torapture  staff   685M  3 30 18:02 raw.txt
-rwxrwxrwx  1 torapture  staff   685M  3 30 18:28 sort.txt
 torapture@localhost ~/SSD/Codes/acm diff output.txt sort.txt
```