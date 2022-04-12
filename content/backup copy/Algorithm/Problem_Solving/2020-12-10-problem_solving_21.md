---
# author: cjlee
categories: problem-solving
# cover: /assets/covers/coding.png
date: "2020-12-10T13:39:00Z"
tags: ['programmers']
title: '# 알고스팟 [ID:PICNIC] 소풍 ( Java )'
---

[문제 링크](https://algospot.com/judge/problem/read/PICNIC)

# Problem

**문제**  
안드로메다 유치원 익스프레스반에서는 다음 주에 율동공원으로 소풍을 갑니다. 원석 선생님은 소풍 때 학생들을 두 명씩 짝을 지어 행동하게 하려고 합니다. 그런데 서로 친구가 아닌 학생들끼리 짝을 지어 주면 서로 싸우거나 같이 돌아다니지 않기 때문에, 항상 서로 친구인 학생들끼리만 짝을 지어 줘야 합니다.

각 학생들의 쌍에 대해 이들이 서로 친구인지 여부가 주어질 때, 학생들을 짝지어줄 수 있는 방법의 수를 계산하는 프로그램을 작성하세요. 짝이 되는 학생들이 일부만 다르더라도 다른 방법이라고 봅니다. 예를 들어 다음 두 가지 방법은 서로 다른 방법입니다.

(태연,제시카) (써니,티파니) (효연,유리)  
(태연,제시카) (써니,유리) (효연,티파니)  

**입력**  
입력의 첫 줄에는 테스트 케이스의 수 C (C <= 50) 가 주어집니다. 각 테스트 케이스의 첫 줄에는 학생의 수 n (2 <= n <= 10) 과 친구 쌍의 수 m (0 <= m <= n*(n-1)/2) 이 주어집니다. 그 다음 줄에 m 개의 정수 쌍으로 서로 친구인 두 학생의 번호가 주어집니다. 번호는 모두 0 부터 n-1 사이의 정수이고, 같은 쌍은 입력에 두 번 주어지지 않습니다. 학생들의 수는 짝수입니다.

**출력**  
각 테스트 케이스마다 한 줄에 모든 학생을 친구끼리만 짝지어줄 수 있는 방법의 수를 출력합니다.

**예제 입력**  
3   
2 1   
0 1   
4 6   
0 1 1 2 2 3 3 0 0 2 1 3   
6 10   
0 1 0 2 1 2 1 3 1 4 2 3 2 4 3 4 3 5 4 5  

**예제 출력**  
1
3
4

# Solve

: 책의 카테고리도 그렇지만, 문제 자체의 내용도 읽어보면, 완전탐색을 통해서 구할 수 있음을 알 수 있다.

즉, 아직 짝이 맺어지지 않은 학생을 두 명 (A, B) 고른 다음에, 그 둘이 친구인지 확인해보고, 맞다면 짝을 맺어주는 방식을 계속 반복하면 된다.

따라서, 자연스럽게 재귀함수와, 짝이 맺어졌는지 확인하는 boolean 배열(대개 visited 배열로 표현되는) 이 필요하단 것을 머릿속에 떠올릴 수 있다. 

```java
private int countPairings(boolean[] paired) {

    int target = findTarget(paired);
    if (target == -1) return 1;

    int ret = 0;
    for (int i = target + 1; i < n; i++) {
        if (!paired[i] && areFriends[target][i]) {
            paired[target] = paired[i] = true;
            ret += countPairings(paired);
            paired[target] = paired[i] = false;
        }
    }

    return ret;
}
```

위 함수는 paried 라는 boolean 배열을 받아서, 짝을 찾아서 맺어주는 역할을 하는 재귀함수이다. 그런데 여기서 눈여겨보아야 할 것이, 바로 `target` 변수다. `target` 변수는 **1.** 아직 짝을 구하지 못한 학생 A 를 나타내기도 하지만, **2.** 동시에 재귀 함수의 호출을 종료하는 Base case이기도 하며, **3.** 또한 **짝의 쌍을 중복으로 찾지 않도록 해주는 역할** 까지 해준다.

이게 무슨 말일까? 우선, `target` 변수를 할당해주는 `findTarget()` 함수를 살펴보자.

```java
public int findTarget(boolean[] paired) {
    int target = -1;
    for (int i = 0; i < n; ++i) {
        if (!paired[i]) {
            target = i;
            break;
        }

    }

    return target;
}
```

특별할 것 없이, target을 -1로 설정해놓은 뒤에 for 문을 돌면서, false인, 즉 다시 말해 아직 짝이 맺어지지 않은 학생을 발견하면, 해당 학생을 target으로 return 해주는 함수이다. 따라서, 1번에서 언급한 아직 짝을 구하지 못한 학생 A를 나타낸다. 

만약 짝이 맺어지지 않은 학생을 발견하지 못한다면, 이는 모든 학생이 짝이 맺어졌다는 뜻이므로 target은 -1 로 return 될 것이고, 따라서 `if (target == -1) return 1;` 라는 Base case의 조건에 부합하게 된다.

다시 `countPairings()` 로 돌아가서, for 문을 살펴보자.

`for (int i = target + 1; i < n; i++)` 

짝이 맺어지지 않은 학생 B(코드 상 i)를 찾는 for문이, target + 1 부터 시작한다. 0부터 시작하지 않은 이유가 바로 3번의 **짝의 쌍을 중복으로 찾지 않기 위함** 이다.

잠깐 다른 이야기를 하자면, 수학의 순열과 조합을 떠올려보자.   
순열은 "서로 다른 n 개의 원소를 가진 집합에서, r 개를 **순서를 따져서** 나열" 한다.   
반면 조합은 "서로 다른 n 개의 원소를 가진 집합에서, r 개를 **순서를 따지지 않고** 나열" 한다.

[1, 2, 3] 이라는 배열에서, 

3P2 는 (1, 2), (2, 1), (1, 3), (3, 1), (2, 3), (3, 2)  이 되고,  
3C2 는 (1, 2), (1, 3), (2, 3) 이 된다.

이쯤되면 아마 눈치를 챘을텐데, 우리가 구하고자 하는 것은 **조합** 이지, 순열이 아니다. 즉, (1, 2)와 (2, 1)은 같은 취급을 한다.

다시 코드로 돌아가서, 만약 for 문이 target + 1 이 아닌, 0 부터 시작했다고 해보자. 

0번 학생과 5번 학생이 짝이라고 하고, 재귀 함수를 호출하는 과정 속에서 5번이 짝이 맺어지지 않아 A 로 선정되었다고 할 때, 0 번 학생은 B로 선정되어 그 둘을 짝으로 맺어줄 것이다. 이는 우리가 원하는 결과가 아니다. 왜냐면, 그 전에 이미 A 가 0 이었을 때 B 가 5 임을 발견해서, 짝을 맺어주었을 것이기 때문이다.

즉, 정리하자면, (5, 0) 이라는 쌍을 생성하게 되는 꼴이고, 이는 조합으로 고려하겠다는 의미가 아닌, 순열로 고려하겠다는 의미이므로, 중복된 쌍을 만들게 된다. 이는 우리가 원하는 것이 아니다.

---

핵심이 되는 target 을 이해하고 나면, 나머지 코드는 쉽게 따라갈 수 있다. 전체 코드는 다음과 같다.

```java
package algospot;

/*

3
2 1
0 1
4 6
0 1 1 2 2 3 3 0 0 2 1 3
6 10
0 1 0 2 1 2 1 3 1 4 2 3 2 4 3 4 3 5 4 5

*/

import java.io.BufferedReader;
import java.io.IOException;
import java.io.InputStreamReader;
import java.util.*;

// 소풍
public class Picnic {
    public static void main(String[] args) throws IOException {
        List<Solution> problems = new ArrayList<>();
        BufferedReader br = new BufferedReader(new InputStreamReader((System.in)));
        StringTokenizer st = new StringTokenizer(br.readLine());
        int c = Integer.parseInt(st.nextToken());

        for (int i = 0; i < c; ++i) {
            st = new StringTokenizer(br.readLine());
            int n = Integer.parseInt(st.nextToken());
            int m = Integer.parseInt(st.nextToken());

            st = new StringTokenizer(br.readLine());
            boolean[][] areFriends = new boolean[n][n];

            for (int j = 0; j < m; j++) {
                int a = Integer.parseInt(st.nextToken());
                int b = Integer.parseInt(st.nextToken());

                areFriends[a][b] = true;
                areFriends[b][a] = true;
            }


            problems.add(new Solution(n, m, areFriends));

        }


        for (Solution problem : problems) {
            int answer = problem.run();
            System.out.println(answer);
        }
    }

    static class Solution {
        private int n;
        private int m;
        private boolean[][] areFriends;

        public Solution(int n, int m, boolean[][] areFriends) {
            this.n = n;
            this.m = m;
            this.areFriends = areFriends;
        }

        public int run() {

            boolean[] paired = new boolean[n];

            int ret = countPairings(paired);

            return ret;
        }

        private int countPairings(boolean[] paired) {

            int target = findTarget(paired);
            if (target == -1) return 1;

            int ret = 0;
            for (int i = target + 1; i < n; i++) {
                if (!paired[i] && areFriends[target][i]) {
                    paired[target] = paired[i] = true;
                    ret += countPairings(paired);
                    paired[target] = paired[i] = false;
                }
            }

            return ret;
        }

        public int findTarget(boolean[] paired) {
            int target = -1;
            for (int i = 0; i < n; ++i) {
                if (!paired[i]) {
                    target = i;
                    break;
                }

            }

            return target;
        }
    }
}
```