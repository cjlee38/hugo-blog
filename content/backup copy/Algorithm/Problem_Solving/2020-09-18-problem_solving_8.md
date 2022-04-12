---
# author: cjlee
categories: problem-solving
# cover: /assets/covers/coding.png
date: "2020-09-18T20:12:00Z"
tags: ['baekjoon']
title: '# 백준[No.17140] - 이차원 배열과 연산 ( Java )'
---

[문제 링크](https://www.acmicpc.net/problem/17140)

# Problem
![17140번-이차원-배열과-연산](/assets/images/2020-09-18-20-00-00_2020-09-18-problem_solving_8.png){: .alignCenter}

# Solve
: 문제 자체를 푸는데도 꽤 걸렸지만, 쓸데 없는 부분에서 고생했던 문제라서, 살짝 열받아서 가져왔다.

문제의 지문 자체가 그리 어렵지는 않지만, 시뮬레이션해서 이를 만들어줘야 하기 때문에, 이 부분에서 난이도가 좀 오르는 것 같다.

## 접근 방법
: 다음의 순서를 반복하면 된다

1. 행과 열 중, size가 더 긴 곳(같다면 행)을 선택한다.
2. 더 긴 곳을 기준으로 돌면서(즉, 행이라면 하나의 행을 가져와서) 정렬을 수행한다.  
   1) 정렬은 **"숫자의 등장 횟수를 기준으로 오름차순"**  
   2) 숫자의 등장 횟수가 같다면, **숫자 자체의 크기를 기준으로 오름차순**  
3. 각 정렬을 수행하면서, 달라진 최대 길이를 구한다.
4. 최대 길이만큼, 짧은 길이를 0으로 채워서 늘려준다.
5. 1로 가서 반복한다. 만약, 반복 횟수가 100 이상이면 -1을 리턴한다.


## 구현

### 첫 번째 설계 - 이중배열

: 주어진 이중배열을 다루기 위해, 두 가지 방법이 존재한다.
1. 이중배열의 크기를 100,100으로 고정시키고, 이를 계속 변경시키는 방법
2. 반복이 진행될때마다, 배열을 새로 만들어서 갈아끼우는 방법

두 가지 방법 중, 1번 방법을 선택했는데, 그 이유는  
1) 최대 100번 반복하는 동안, 배열을 새로 만드는 작업은 **자원의 낭비**라고 생각했다.  
2) 배열의 최대 크기가 100,100 이고 나머지는 버린다고 문제에서 제시했으므로, 이를 따르지 않을 이유가 없었다.  

1번 방법을 택했으므로, 계속해서 배열의 **"실제 사이즈"** 를 track 할 수 있는 변수인 rSize와 cSize를 만들었다.

```java
int rSize = 3;
int cSize = 3;
```

그리고, 최대 길이를 구하고 나면, 한번의 반복이 끝났을 때 최대 길이로 기존의 길이를 바꿔줘야 한다.

```java
int newCSize = 0; // 새로 구한 최대 길이는 기존의 길이보다 짧을 수 있기 때문에, cSize로 초기화 하면 안된다.
newCSize = PQ.size() * 2 > newCSize ? Math.min(PQ.size() * 2, 100) : newCSize;
```

PriorityQueue에서 두개의 요소를 꺼낼 것 이므로 * 2 를 해줘야 한다.  
또한, 최대 길이는 100을 넘으면 안되므로,  
현재 line의 길이 vs 지금까지 나온 최대 길이 vs 100 을 비교해야 한다.


### 두 번째 설계 - 정렬
: 정렬을 할 때, 어떤 방법으로 정렬할지에 대해서 고민했다.  
우선순위큐라는 지금 문제에 딱 걸맞는 자료구조가 있으므로, 이를 사용하지 않을 이유가 없었다.  

문제를 풀고 나서 다른 사람들의 코드를 보니, 그냥 List를 정렬하는 사람도 있었는데, 이는 취향의 차이라고 생각한다.

어쨌든, 우선순위큐를 만드려면, 이를 비교할 수 있도록 Comparable 인터페이스를 구현해야 하고,  
우리는 1. 숫자의 등장 횟수, 2. 숫자의 크기 순서대로 비교할 것이므로,  
Comparable 인터페이스를 구현하는 Number 클래스를 만들었다.

```java
class Number implements Comparable<Number> {
    private int num;
    private int count;

    public Number(int num, int count) {
        this.num = num;
        this.count = count;
    }

    public int getNum() {
        return num;
    }

    public int getCount() {
        return count;
    }

    @Override
    public int compareTo(Number number) {
        if (this.count == number.count) {
            return this.num > number.num ? 1 : -1;
        } else {
            return this.count > number.count ? 1 : -1;
        }
    }
}
```

### 세 번째 설계 - 정렬을 위한 Counter
: Python에서는 collections 모듈에 Counter라는 편리한 방법이 있지만, 자바에서는 딱히 없으므로 이를 흉내내기 위해 HashMap을 사용했다. 
```java
Map<Integer, Integer> KV = new HashMap<>();
for(int i=0; i<size; i++) {
    if (values[i] != 0) {
         KV.compute(values[i], (k, v) -> v == null ? 1 : v + 1);
    }
}
```

이쯤 되면, 거의 다 끝났다. 이제 계산을 해서 PriorityQueue를 만든 후, 하나씩 꺼내서 배열의 요소를 하나하나 바꿔주기만 하면 된다.

이 때, **바꾼 요소 이외의 나머지 요소들은 0으로 초기화 해야 함**을 잊지 말자.(길이가 기존보다 짧아질 수 있으므로)  
가령, 1 1 1 이라는 line이 들어왔다면, 1 3 으로 바꿔주고, 나머지는 0으로 초기화 해야 한다.  
그렇지 않는다면, 1 3 1 ... 이런식으로 되기 때문에, 문제가 된다.


코드 전체는 다음과 같다.

```java
package BOJ;

import java.io.BufferedReader;
import java.io.IOException;
import java.io.InputStreamReader;
import java.util.*;

public class bj17140 { // need to be renamed as main

    static Solution init() throws IOException{


        BufferedReader br = new BufferedReader(new InputStreamReader(System.in));
        StringTokenizer st = new StringTokenizer(br.readLine());
        int r = Integer.parseInt(st.nextToken()) - 1;
        int c = Integer.parseInt(st.nextToken()) - 1;
        int k = Integer.parseInt(st.nextToken());

        int[][] map = new int[101][101];
        for (int i = 0; i < 3; i++) {
            st = new StringTokenizer(br.readLine());
            for (int j = 0; j < 3; j++) {
                map[i][j] = Integer.parseInt(st.nextToken());
            }
        }

        return new Solution(r, c, k, map);
    }

    public static void main(String[] args) throws IOException {
        Solution s = init();
        int answer = s.run();
        System.out.println(answer);

    }
}

class Solution {
    private int r;
    private int c;
    private int k;
    private int[][] map;
    
    public Solution(int r, int c, int k, int[][] map) {
        this.r = r;
        this.c = c;
        this.k = k;
        this.map = map;
    }

    public int run() {
        int time = 0;
        int rSize = 3;
        int cSize = 3;
        while(true) {
            if (time > 100) {
                time = -1;
                break;
            }
            else if (map[r][c] == k) {
                break;
            } else {
                if (rSize >= cSize) {
                    cSize = operateR(rSize, cSize);
                } else {
                    rSize = operateC(rSize, cSize);
                }
            }
            time++;
        }
        return time;
    }

    private PriorityQueue<Number> countValues(int[] values, int size) {
        Map<Integer, Integer> KV = new HashMap<>();
        PriorityQueue<Number> Q = new PriorityQueue<>();
        for(int i=0; i<size; i++) {
            if (values[i] != 0) {
                KV.compute(values[i], (k, v) -> v == null ? 1 : v + 1);
            }
        }

        for(Integer key : KV.keySet()) {
            Q.offer(new Number(key, KV.get(key)));
        }

        return Q;

    }

    private int operateR(int rSize, int cSize) {
        int newCSize = 0;
        for (int i = 0; i < rSize; i++) {
            PriorityQueue<Number> PQ = countValues(map[i], cSize);
            newCSize = PQ.size() * 2 > newCSize ? Math.min(PQ.size() * 2, 100) : newCSize;
            int idx = 0;
            while(idx < 100) {
                if (!PQ.isEmpty()) {
                    Number number = PQ.poll();
                    map[i][idx++] = number.getNum();
                    map[i][idx++] = number.getCount();
                } else {
                    map[i][idx++] = 0;
                }
            }
        }

        return newCSize;
    }


    private int operateC(int rSize, int cSize) {
        int newRSize = 0;
        for(int i=0; i<cSize; i++) {
            int[] col = new int[rSize];
            for(int j=0; j<rSize; j++) {
                col[j] = map[j][i];
            }
            PriorityQueue<Number> PQ = countValues(col, rSize);
            newRSize = PQ.size()*2 > newRSize ? Math.min(PQ.size() * 2, 100) : newRSize;
            int idx = 0;
            while(idx < 100) {
                if (!PQ.isEmpty()) {
                    Number number = PQ.poll();
                    map[idx++][i] = number.getNum();
                    map[idx++][i] = number.getCount();
                } else {
                    map[idx++][i] = 0;
                }
            }

        }
        return newRSize;
    }


}


class Number implements Comparable<Number> {
    private int num;
    private int count;

    public Number(int num, int count) {
        this.num = num;
        this.count = count;
    }

    public int getNum() {
        return num;
    }

    public int getCount() {
        return count;
    }

    @Override
    public int compareTo(Number number) {
        if (this.count == number.count) {
            return this.num > number.num ? 1 : -1;
        } else {
            return this.count > number.count ? 1 : -1;
        }
    }
}


```