---
title: 'Initialize C Structure'
date: 2020-04-04 01:15:54
tags: [c]
published: true
hideInList: false
feature: 
isTop: false
---
Define C structure and initialize it.

```c
#include <stdio.h>

typedef struct _date {
    int year;
    int month;
    int day;
} Date;


typedef struct _time {
    int hour;
    int minute;
    int second;
} Time;

typedef struct _datetime {
    Date date;
    Time time;
} Datetime;


void print_date(Date date) {
    printf("Year: %d, Month: %d, Day: %d\n", date.year, date.month, date.day);
}

void print_datetime(Datetime dt) {
    printf("%d-%d-%d %d:%d:%d\n", 
           dt.date.year, dt.date.month, dt.date.day,
           dt.time.hour, dt.time.minute, dt.time.second);
}


int main() {
    Date today = {2020, 4, 4};
    Date tomorrow = {.year=2020, .month=4, .day=5};

    print_date(today);
    print_date(tomorrow);

    today = (Date) {2020, 4, 3};
    print_date(today);

    Datetime now = {
        {2020, 4, 4},
        {.hour=13, .minute=37, .second=40}
    };
    print_datetime(now);

    return 0;
}
```

Compile and run:
```
Year: 2020, Month: 4, Day: 4
Year: 2020, Month: 4, Day: 5
Year: 2020, Month: 4, Day: 3
2020-4-4 13:37:40
```