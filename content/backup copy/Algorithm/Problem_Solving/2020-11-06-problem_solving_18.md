---
# author: cjlee
categories: problem-solving
# cover: /assets/covers/coding.png
date: "2020-11-06T09:05:00Z"
tags: ['baekjoon']
title: '# 백준[No.10227] - 삶의 질 ( Java )'
---

[문제 링크](https://www.acmicpc.net/problem/10227)

# Problem

![](/assets/images/2020-11-06-10-56-56_2020-11-06-problem_solving_18.md.png)


# Solve
: 처음에는 한 5분 정도는 문제가 무슨 뜻인지 잘 이해가 되지 않았다.

문제의 예제를 기준으로 설명하자면,  
1. R * C 크기의 2차원 배열에서, 각 배열의 요소는 1 ~ R * C 까지의 중복되지 않는 값을 가진다.  
2. H * W (H와 W는 홀수) 의 크기만큼의 영역이 여러 개 존재하는데, 그 중 중앙값이 가장 낮은 값을 골라야 한다.

즉, 문제의 예제를 기준으로 설명하자면,  

5 11 12  
17 18 2  
4 23 20  

이 하나의 sub-area 이고, 여기서   

2 4 5 11 12 17 18 20 23 

의 중앙값은 12 이다.

이런식으로 계속 반복해서, 모든 sub-area의 중앙값을 구한 뒤,  
그 중 최소가 되는 값(즉, Quality rank가 가장 좋은)을 출력하면 된다.

문제를 이해하고 나니 간단해보였지만, 시간복잡도로 인해 전혀 간단하지 않았던 문제였다.  
한참을 고민하다 안되서, 결국 구글링을 통해 답지를 보고 어떻게 풀어야할지 이해했다.

### 첫 번째 시도(실패)
: 처음에는, 문제에서 요구하는 그대로 시뮬레이션 했다.

sub-area를 구하고, 이를 array로 만들고, 중앙값을 찾아서, 최소값인지 비교했다.  

```java
package BOJ;

import java.io.BufferedReader;
import java.io.IOException;
import java.io.InputStreamReader;
import java.util.Arrays;
import java.util.StringTokenizer;

public class bj10227 {

    static Solution10227 init() throws IOException {
        BufferedReader br = new BufferedReader(new InputStreamReader(System.in));
        StringTokenizer st = new StringTokenizer(br.readLine());
        int r = Integer.parseInt(st.nextToken());
        int c = Integer.parseInt(st.nextToken());
        int h = Integer.parseInt(st.nextToken());
        int w = Integer.parseInt(st.nextToken());

        int[][] map = new int[r][c];
        for (int i = 0; i < r; i++) {
            st = new StringTokenizer(br.readLine());
            for (int j = 0; j < c; j++) {
                map[i][j] = Integer.parseInt(st.nextToken());
            }
        }

        return new Solution10227(r, c, h, w, map);
    }

    public static void main(String[] args) throws IOException {
        Solution10227 s = init();
        int answer = s.run();
        System.out.println(answer);
    }
}

class Solution10227 {
    private int r;
    private int c;
    private int h;
    private int w;
    private int[][] map;

    public Solution10227(int r, int c, int h, int w, int[][] map) {
        this.r = r;
        this.c = c;
        this.h = h;
        this.w = w;
        this.map = map;
    }

    public int run() {
        int min = Integer.MAX_VALUE;

        for (int i = 0; i <= r - h; i++) {
            for (int j = 0; j <= c - w; j++) {
                int[] subArray = createSubArray(i, j);
                min = Math.min(min, getMedian(subArray));
            }
        }

        return min;
    }


    public int[] createSubArray(int i, int j) {
        int[] sublist = new int[h*w];
        for (int k = 0; k < h; k++) {
            for (int l = 0; l < w; l++) {
                sublist[(k*h)+l] = map[k+i][l+j];
            }
        }
        return sublist;
    }

    public int getMedian(int[] sublist) {
        Arrays.sort(sublist);
        return sublist[(h*w) / 2];
    }
}
```

시간초과가 떴으니 답이 맞았는지도 잘 모르겠다.  
아무튼, 코드가 그리 길지 않으니 이해하는데에는 큰 어려움이 없을 것이라 생각한다.  
이렇게 제출하고 나니까 곧바로 시간초과가 떴다.  

문제의 조건을 다시 보니, R과 C는 최대 3000 까지 간다는 것을 볼 수 있었다.  
만약 H와 W가 1500 이라면, 한 번의 Array를 만드는데 1500 * 1500 이 소요될 것이고,  
이를 정렬하는데에 또 N*logN 이 걸릴 것이고,  
또 이 Array를 만들고 정렬하는 총 횟수는 1500 * 1500 번이 필요하다.  

대충 생각만해도 끔찍하다. 

### 두 번째 방법
: [다른 사람의 풀이](https://luckytoilet.wordpress.com/2010/08/18/ioi-2010-quality-of-living/) 를 보고 이해했다.

해당 풀이는, 두 단계의 접근법으로 문제를 풀었다.

1. 이분 탐색
2. 누적 합

#### 1) 이분 탐색
: 이분 탐색은 여기서 설명할 내용은 아니니, 뭔지 잘 모른다면 다른 글을 참고하자.
이분 탐색을 이용하는 방법은 기존의 방법과 궤를 달리한다.  

간단한 Array에 대한 이분 탐색을 하려면 정렬이 되어 있어야 하는데, 이는 이 문제에 적용해 바꿔 말하면, **내가 갖고 있는 값을 기준으로, 위로 가야 하는가 아래로 가야 하는가에 대한 판단**이 가능해야 한다.   
다음의 내용을 차분히 따라가보자.


우리가 찾고자 하는 값은 "중앙값" 중 "최소값" 이다.  
만약 어떤 숫자(m)를 기준으로 삼고, 

* m 보다 크면 -> 1
* m 보다 작으면 -> -1
* m 이면 -> 0

으로 sub-area 를 만들어준다고 가정해보자.  
문제의 예시를 기준으로 설명하자면, 

![](/assets/images/2020-11-06-11-23-06_2020-11-06-problem_solving_18.md.png)


이 상태에서, m을 (임의로) 12로 둔다고 해보자. 그러면,  

![](/assets/images/2020-11-06-11-24-50_2020-11-06-problem_solving_18.md.png)

이와 같은 그림이 될 것이다. 그리고 이중에서, 임의의 sub-area를 고른 뒤, sub-area의 합계를 구한다.

이 때,  

* sub-area의 합이 0인 경우 -> m 이 해당 sub-area의 중앙값임.  
* sub-area의 합이 음수인 경우 -> m 이 중앙값보다 큼.
* sub-area의 합이 양수인 경우 -> m 이 중앙값보다 작음.

을 의미한다. 정말일까?

![](/assets/images/2020-11-06-11-28-31_2020-11-06-problem_solving_18.md.png)

위에서 했던 내용이다. 빨간 박스 부분의 합은 0이 된다.  
그리고, 원본의 Array인 2 4 5 11 12 17 18 20 23 의 중앙값은 12이다.  
따라서, sub-area의 합이 0이라면, m 이 중앙값이 맞다.

![](/assets/images/2020-11-06-11-30-56_2020-11-06-problem_solving_18.md.png)

하지만 빨간 박스를 위 그림과 같이 옮기게 되면, 합은 -2 가 된다.  
원본 Array를 오름차순 정렬을 해보면 1 2 3 7 10 12 16 20 25 이므로,  
m(12)은 중앙값(10)보다 크다.

어찌보면 되게 당연해보인다. **그런데 이걸로 무엇을 할 수 있을까?**  
사실 이 부분이 이해하기 너무 어려웠다.  
문제에서 요구하는 내용과, 이러한 접근법이 머릿속에서 mapping 되지 않았기 때문이다.

m을 바꿔서, 1로 설정했다고 가정해보자.   
그러면 다음 그림과 같이, 값 1에 해당하는 부분은 0이 되고, 나머지는 1이 된다.

![](/assets/images/2020-11-06-11-36-52_2020-11-06-problem_solving_18.md.png)

이는 **어떠한 sub-area를 그려도 m이 중앙값이 될 수 없다는 의미**이기도 하지만,  
동시에 **sub-area가 모두 양수이므로, m이 너무 작다는 것을 의미**한다.

반대로, m을 25로 설정했다고 가정해보자.  
역시, 값 25에 해당하는 부분은 0 이 되고, 나머지는 -1이 된다.

![](/assets/images/2020-11-06-11-47-51_2020-11-06-problem_solving_18.md.png)  

이 또한, **어떠한 sub-area를 그려도 m이 중앙값이 될 수 없다는 의미** 이면서,  
동시에 **sub-area가 모두 음수이므로, m이 너무 크다는 것을 의미**한다.  

이 쯤되면, 어떠한 규칙을 발견할 수 있지 않은가?   
즉, **우리가 설정한 m이 중앙값이 되면서, 다시 말해 sub-area의 합이 0이면서,**  
**동시에 이외의 모든 sub-area들은 양수의 값을 가져야 한다는 것을 의미**한다!

따라서, 위 문제의 정답인 9를 넣은 경우의 모습은 다음과 같다.

![](/assets/images/2020-11-06-11-52-24_2020-11-06-problem_solving_18.md.png)

위 빨간 박스의 영역을 제외하고는, **어떠한 sub-area를 계산해봐도 그 합은 모두 양수**가 나온다.(아직도 의심이 된다면, 위에서 m이 12일 때, 그리고 빨간 박스가 우측 상단에 있었을 때 합이 -2의 음수가 나왔다는 것을 떠올려보자.)

이에 더불어, **우리가 원하는 9 라는 정답은, 1 ~ 25 의 중앙값(13) 에서 그리 멀지 않을 것이므로**, 13에서 출발해서 위/아래로 이분탐색 하듯이 계산해나가면 된다.

이렇게 계산했을 때의 장점은, **"내가 구한 Median이 좋은 Median인가?"** 에 대한 판단이 가능해져서, Median을 증가시킬 것인지, 감소 시킬 것인지에 대한 구분이 가능해지고, 이에 따라 이분 탐색을 사용할 수 있어진다는 것이다.

#### 2) 누적 합.
누적 합은 위 이분 탐색 처럼 어렵지는 않다.  
단순하게, "계산을 미리 해놓는 것" 이라고 생각하면 편하다.  
m이 9일때의 모습을 다시 가져와보자.

![](/assets/images/2020-11-06-12-10-08_2020-11-06-problem_solving_18.md.png)

해당 빨간 박스 부분을, 행 단위로 합을 미리 구해놓으면 순서대로  

3  
-1  
-1  
2  
1

이 된다.

처음 [3, -1, -1] 부분을 가져와서 합쳐보면(즉, 3*3 sub-area의 합) 1이 된다.  

3을 빼고, 2를 집어넣어서 만든 [-1, -1, 2] 를 합치면, 0이 된다.  
즉, 해당 sub-area 에서, m은 중앙값이다.

기존의 방식이었다면, -1`(-1 - 1 + 1)`, -1`(1 - 1 - 1)` 의 두 개를, 중복해서 계산(3이 있을 때 한번, 2가 있을 때 한번) 해야 했지만, 누적 합을 이용할 경우 그럴 필요가 없다. 가로 연산은 한번만 해놓고, 세로 연산할 때 가져와서 쓰기만 하면 되기 때문이다.

---

자, 이제 이론적인 부분은 다 끝났다. 이를 코드로 구현해보자.

```java
package BOJ;

import java.io.BufferedReader;
import java.io.IOException;
import java.io.InputStreamReader;
import java.util.StringTokenizer;

public class bj10227 {

    static Solution10227 init() throws IOException {
        BufferedReader br = new BufferedReader(new InputStreamReader(System.in));
        StringTokenizer st = new StringTokenizer(br.readLine());
        int r = Integer.parseInt(st.nextToken());
        int c = Integer.parseInt(st.nextToken());
        int h = Integer.parseInt(st.nextToken());
        int w = Integer.parseInt(st.nextToken());

        int[][] map = new int[r][c];
        for (int i = 0; i < r; i++) {
            st = new StringTokenizer(br.readLine());
            for (int j = 0; j < c; j++) {
                map[i][j] = Integer.parseInt(st.nextToken());
            }
        }

        return new Solution10227(r, c, h, w, map);
    }

    public static void main(String[] args) throws IOException {
        Solution10227 s = init();
        int answer = s.run();
        System.out.println(answer);
    }
}

class Solution10227 {
    private int R;
    private int C;
    private int H;
    private int W;
    private int[][] map;

    public Solution10227(int r, int c, int h, int w, int[][] map) {
        R = r;
        C = c;
        H = h;
        W = w;
        this.map = map;
    }

    public int run() {
        int min = 1;
        int max = R * C;
        int mid;

        while (true) {
            mid = (max + min) / 2;
            int result = tryMedian(mid);

            if (result == 0) break;
            else if (result == 1) min = mid + 1;
            else max = mid - 1;
        }
        return mid;
    }

    private int tryMedian(int mid) {
        boolean exists = false;

        int[] rowSum = new int[R];
        for (int i = 0; i <= C - W; i++) {
            getRowSumOfTernary(rowSum, mid, i);
            int mapSum = 0;
            for (int j = 0; j <= R - H; j++) {
                mapSum += getMapSum(rowSum, j);

                if (mapSum == 0) exists = true;
                else if (mapSum < 0) return -1;
            }
        }

        if (exists) return 0;
        return 1;
    }


    private void getRowSumOfTernary(int[] rowSum, int mid, int x) {
        if (x == 0) {
            for (int i = 0; i < R; i++) {
                int sum = 0;
                for (int j = x; j < x + W; j++) {
                    sum += getTernary(mid, map[i][j]);
                }
                rowSum[i] = sum;
            }
        } else {
            for (int i = 0; i < R; i++) {
                rowSum[i] += getTernary(mid, map[i][x+W-1])- getTernary(mid, map[i][x-1]);
            }
        }
    }

    private int getTernary(int mid, int value) {
        if (value == mid) return 0;
        else if (value > mid) return 1;
        else return -1;
    }

    private int getMapSum(int[] rowSum, int x) {
        int mapSum = 0;
        if (x == 0) {
            for (int i = 0; i < H; i++) {
                mapSum += rowSum[i];
            }
        } else {
            mapSum += rowSum[x+H-1] - rowSum[x-1];
        }

        return mapSum;
    }
}
```

정답 코드는 위에서 설명한 내용과 조금 다른 부분이 있는데,  
`rowSum` 과 `mapSum` 변수를 매 번 새로 구하니 여전히 시간초과가 났다.

가령, mapSum의 경우, rowSum이 다음과 같이 있다고 할 때  

`3  -1  -1  2  1`

`3 - 1 - 1` 를 계산해서 `1` 이라는 결과를 내놓고,  
그 다음에 다시 `-1 - 1 + 2` 를 계산해서 `0` 이라는 결과를 내놓는 방식을 사용했었다.

그런데, 이렇게 할 경우 -1, -1 의 중복 연산(위에서 언급했던, "-1을 구하기 위한" `(-1 - 1 + 1)` 과 `(1 - 1 - 1)` 연산과는 다르다. 이 연산의 **결과물인 -1의 연산**이다.) 이 발생하므로, 이를 제거하기 위해 Sliding window 기법을 이용해서,  

* 첫 rowSum (혹은 mapSum)을 구하는 경우 -> 순수하게 계산
* 첫 번째가 아닌 경우 -> 가장 첫 번째 열(또는 값)을 빼고, 새로운 열(또는 값)을 추가

하는 방식으로 수정했다.(그 덕에 코드가 조금 난잡해졌다.)

# 마치며
다른 문제들과 다르게, 진짜 너무너무 어려웠다..  
Internatinal Olympiad in Informatics(처음 들어봤다.) 문제라는데, 이게 어떻게 40% 의 정답률인지 이해가 안될 정도였다. 

그러다보니 기존 PS 들은 그냥, 내가 어떻게 풀었는지 설명하고 코드를 보여주는 식으로 끝났는데, 이 문제는 어쩌다보니 각잡고 쓰게 되었다..

풀고나서, 다른 사람의 풀이를 보니 Dynamic Programming을 이용하는 방법도 있었다.  
더 이상 머리아프기 싫어서 쳐다보기를 그만두었는데, 혹시 궁금한 사람은 

[이 동영상](https://www.youtube.com/watch?v=J05ERPxrahE&ab_channel=IOIKOREA)을 참고하면 될 것 같다.