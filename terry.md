# Prepare

## 手画架构图


## 遇到的问题


## Quick3Way

```
package com.alibaba.ttl;

import java.util.ArrayList;
import java.util.Collections;
import java.util.List;

public class Quick {

    public static void main(String[] args) {
        List<Integer> data = new ArrayList<>();
        for (int i = 0; i < 100; i++) {
            data.add(i + 1);
        }
        Collections.shuffle(data);
        Comparable[] array = data.toArray(new Comparable[0]);
        sort(array, 0, data.size() - 1);
        for (Comparable comparable : array) {
            System.out.println(comparable);
        }
    }

    public static void sort(Comparable[] a, int lo, int hi) {
        if (lo >= hi) return;
        int lt = lo;
        int gt = hi;
        Comparable v = a[lo];
        int i = lo;
        while (i <= gt) {
            int cmp = a[i].compareTo(v);
            if (cmp < 0) {
                exchange(a, i++, lt++);
            } else if (cmp > 0) {
                exchange(a, i, gt--);
            } else {
                i++;
            }
        }
        sort(a, lo, lt -1);
        sort(a, gt + 1, hi);
    }

    private static boolean less(Comparable a, Comparable b) {
        return a.compareTo(b) < 0;
    }

    private static void exchange(Object[] a, int i, int j) {
        Object temp = a[i];
        a[i] = a[j];
        a[j] = temp;
    }
}
```