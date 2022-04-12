---
# author: cjlee
categories: problem-solving
# cover: /assets/covers/coding.png
date: "2020-11-18T11:52:00Z"
tags: ['programmers']
title: '# 프로그래머스[Lv.3] - 순위 ( Java )'
---

[문제 링크](https://programmers.co.kr/learn/courses/30/lessons/17677)

# Problem

**문제 설명**  
n명의 권투선수가 권투 대회에 참여했고 각각 1번부터 n번까지 번호를 받았습니다. 권투 경기는 1대1 방식으로 진행이 되고, 만약 A 선수가 B 선수보다 실력이 좋다면 A 선수는 B 선수를 항상 이깁니다. 심판은 주어진 경기 결과를 가지고 선수들의 순위를 매기려 합니다. 하지만 몇몇 경기 결과를 분실하여 정확하게 순위를 매길 수 없습니다.

선수의 수 n, 경기 결과를 담은 2차원 배열 results가 매개변수로 주어질 때 정확하게 순위를 매길 수 있는 선수의 수를 return 하도록 solution 함수를 작성해주세요.

**제한사항**  
- 선수의 수는 1명 이상 100명 이하입니다.  
- 경기 결과는 1개 이상 4,500개 이하입니다.  
- results 배열 각 행 [A, B]는 A 선수가 B 선수를 이겼다는 의미입니다.  
- 모든 경기 결과에는 모순이 없습니다.  

**입출력 예**

|n|results|return|
|:--:|:--:|:--:|
|5|[[4, 3], [4, 2], [3, 2], [1, 2], [2, 5]]|2|

**입출력 예 설명**  
- 2번 선수는 [1, 3, 4] 선수에게 패배했고 5번 선수에게 승리했기 때문에 4위입니다.
- 5번 선수는 4위인 2번 선수에게 패배했기 때문에 5위입니다.

# Solve

## 문제에 대한 이해.
: 나만 그렇게 느끼는 건진 모르겠는데, 코딩 테스트의 문제가 어려워질수록 문제 설명도 굉장히 불친절해진다. 이게 단순히 문제의 난이도를 높이기 위해서 말을 아끼는 것이라면 그러려니 하겠는데, 애초에 설명 자체가 부족하니 참 이해하기가 어렵다.

잠깐 잡소리를 하자면, 대학 교수님들 중에 간혹 그런 분이 있다. 본질적으로 강의라는 것은 학생들에게 지식을 전파하기 위함이고, 시험은 이를 검증하는 것이 목표이다. 따라서, 학생들이 이해할 만한 선에서, 최대한의 깊고 다양한 지식을 학생들에게 전수하면서, 동시에 시험을 변별력 있게 내는 것은 타당하다. 그러나, 거꾸로 시험을 변별력 있게 내기 위해서, 강의를 하면서 일부러 말을 아끼고, 설명해야 할 것도 설명하지 않는 교수님을 몇 번 본적이 있다. 

잡소리는 이만하면 됐고, 어쨌든 이번 순위 문제도 이러한 문제에 해당하는 것 같은데, 문제 내용 중 **"만약 A 선수가 B 선수보다 실력이 좋다면 A 선수는 B 선수를 항상 이깁니다."** 라는 명제는, **만약 A 선수가 B 선수를 이겼다면, A 선수는 B 선수에 비해 항상 실력이 좋다는 것을 의미합니다.** 정도로 바꿔줬으면 좀 잘 와닿지 않았을까 싶다.

또한, 이 문제에는 실제로 일어난 경기가 몇 번 일어났는지에 대해서 설명하고 있지 않다.

무슨말인가 하면, 문제에서는 **몇몇 경기 결과를 분실하였다.** 라고 이야기 하였는데, 이는 곧 주어진 배열보다 더 많은 경기가 있었음을 의미한다. 그리고 이 경기가 몇 번 일어났는지, 매치 방식은 어떻게 되는지에 대한 정보가 전혀 없다. 밑에서 설명할 방법이 아니라, 토너먼트 방식일 수도 있지 않은가? 

차라리 이런 식의 문제 설명보다는, **서바이벌 방식의 권투 대회가 아직 진행중인데, 현재까지의 경기 결과를 가지고 순서가 이미 확정된 선수의 수를 구하려고 합니다.** 정도의 설명이었다면 더욱 명료했을 것 같다.

아무튼 정리하자면, 각 선수는 다른 선수들과 경기를 하는데, 그 대상은 자기 자신을 제외한 나머지 모두가 될 수 있고, A 가 B 를 이겼다는 것은, B 가 이긴 C, D 또한 A가 **항상** 이긴다. 그리고 이러한 가정은, 한 선수가 모든 선수와 게임을 직접 해 봐야 그 선수의 순위가 결정되는 것이 아니라, **대충 몇 번 싸워만 봐도 그 선수에 대한 순위를 알 수 있다는 결론**으로 이어진다.

## 접근 방법

: 문제의 유형이 그래프로 명시되어 있다 보니, 각 선수를 그래프의 Node 로 생각하고, 각 경기의 결과를 방향이 있는 간선으로 생각할 수 있다. 문제의 예시를 그림으로 그리면 다음과 같다.

![](/assets/images/2020-11-19-01-14-52_2020-11-18-problem_solving_19.md.png)



또한, 문제 설명을 기반으로 다음과 같이 정리할 수 있다.

1. 경기 결과에 모순이 없다는 말은, 곧 방향 그래프의 Cycle이 없다는 의미가 된다.
2. 따라서, 특정 Node가 **가리킬 수 있는** 다른 Node들의 **리스트를 만든다.**

그런데, 이렇게만 하면 정보가 부족하다. 가령, 2번 Node의 경우, 5번 Node로부터 승리했기 때문에 4위라는 순위를 얻을 수 있는데, 위 그래프만으로는 이를 설명할 수 없다. 따라서, **방향을 뒤집어서도**, 특정 Node를 가리킬 수 있는 Node들의 리스트 또한 만들어 줘야 한다.

![](/assets/images/2020-11-19-01-03-25_2020-11-18-problem_solving_19.md.png)

즉, 바꿔 말하면, **정방향과 역방향의 그래프 두 개를 바탕**으로, **내가(Node가) 직, 간접적으로 나갈 수 있는 Node 리스트의 합집합**을 만들어 줘야 한다.

위 그림을 바탕으로, 하나씩 살펴보자.

**Node 1**
- 첫 번째 그림에서, Node 1 이 갈 수 있는 Node는 없다.
- 두 번째 그림에서, Node 1 이 갈 수 있는 Node는 2, 5 이다.

-> count = 2

**Node 2**
- 첫 번째 그림에서, Node 2 가 갈 수 있는 Node는 1, 3, 4 이다.
- 두 번째 그림에서, Node 2 가 갈 수 있는 Node는 5 이다.

-> count = 4

**Node 3**
- 첫 번째 그림에서, Node 3 가 갈 수 있는 Node는 4 이다.
- 두 번째 그림에서, Node 3 가 갈 수 있는 Node는 2, 5 이다.
  
-> count = 3

**Node 4**
- 첫 번째 그림에서, Node 4 가 갈 수 있는 Node는 없다
- 두 번째 그림에서, Node 4 가 갈 수 있는 Node는 2, 3, 5 이다.

-> count = 3

**Node 5**
- 첫 번째 그림에서, Node 5 가 갈 수 있는 Node는 1, 2, 3, 4 이다.
- 두 번째 그림에서, Node 5 가 갈 수 있는 Node는 없다.
  
-> count = 4

이렇게 정리했을 때, 노드의 개수 `n` 에서, 자기 자신을 제외한 나머지의 개수는 `n-1` 이므로, 4가 되는 Node 2, Node 5 의 개수 = 2 가 정답이 된다.

## 구현.
: 우선, 문제를 쉽게 하기 위해 `Map<Integer, List<Integer>>` 를 만들어 주었다. 그리고 위에서 이야기했던 것 처럼, 역방향도 고려해줘야 하기 때문에, `winnerMap` 과 `loserMap` 이라는 이름으로 두 개를 만들어 주었다.

```java
Map<Integer, List<Integer>> winnerMap = initGames(n, results, true);
Map<Integer, List<Integer>> loserMap = initGames(n, results, false);
// winnerMap = {1=[2], 2=[5], 3=[2], 4=[3, 2], 5=[]}
// loserMap = {1=[], 2=[4, 3, 1], 3=[4], 4=[], 5=[2]}
```

```java
private Map<Integer, List<Integer>> initGames(int n, int[][] results, boolean isWin) {
    int WINNER = isWin ? 0 : 1;
    int LOSER = isWin ? 1 : 0;
    Map<Integer, List<Integer>> map = new HashMap<>();

    // 한 번의 for문으로 해결하려 하면, 비어 있는 List를 표현할 수 없다.
    for (int i = 1; i <= n; i++) {
        map.put(i, new ArrayList<>());
    }

    for (int[] result : results) {
        int winner = result[WINNER];
        int loser = result[LOSER];
        map.get(winner).add(loser);
    }

    return map;
}
```

그리고 나서, 각 Node (1 ~ n이하) 의 iteration을 돌면서, 합집합을 구하기 위해 `boolean[] visited = new boolean[n];` 를 만들고, 재귀함수를 통해 visited 를 채워나갔다.

배열을 채우고 나서는, 해당 배열에 true 값이 몇개가 들어있는지 파악한 후, `n-1` 개 이면 int형 변수 answer를 1 증가시켰다.

```java
int answer = 0;

for (int i = 1; i <= n; i++) {
    boolean[] visited = new boolean[n];
    recursive(winnerMap, visited, i);
    recursive(loserMap, visited, i);
    // 배열 채우기.

    int battleCount = countBattle(visited);
    if (battleCount == (n - 1)) answer++;
}
```

재귀 함수 내부는, **내가 더이상 나갈 수 없을 때 까지**, 즉 List가 비어있을 때까지 를 종료 조건으로 주었다.

```java
private void recursive(Map<Integer, List<Integer>> map, boolean[] visited, int enemy) {
    if (map.get(enemy).isEmpty()) return;

    for (Integer val : map.get(enemy)) {
        if (!visited[val - 1]) { // 이를 확인하지 않으면, 너무 깊게 들어갈 수 있다.
            visited[val - 1] = true;
            recursive(map, visited, val);
        }
    }
}
```

이렇게 구한 answer 를 return 해주면 된다. 전체 코드는 다음과 같다.

```java
package programmers.lv3;

import java.util.*;

public class p49191 {
    public static void main(String[] args) {
        int n = 5;
        int[][] results = {
                {4, 3},
                {4, 2},
                {3, 2},
                {1, 2},
                {2, 5}
        };

        p49191 p = new p49191();
        int result = p.solution(n, results);
        System.out.println("result = " + result);
    }

    public int solution(int n, int[][] results) {
        Map<Integer, List<Integer>> winnerMap = initGames(n, results, true);
        Map<Integer, List<Integer>> loserMap = initGames(n, results, false);

        int answer = 0;

        for (int i = 1; i <= n; i++) {
            boolean[] visited = new boolean[n];
            recursive(winnerMap, visited, i);
            recursive(loserMap, visited, i);

            int battleCount = countBattle(visited);
            if (battleCount == (n - 1)) answer++;
        }



        return answer;
    }

    private void recursive(Map<Integer, List<Integer>> map, boolean[] visited, int enemy) {
        if (map.get(enemy).isEmpty()) return;

        for (Integer val : map.get(enemy)) {
            if (!visited[val - 1]) {
                visited[val - 1] = true;
                recursive(map, visited, val);
            }
        }
    }

    private int countBattle(boolean[] visited) {
        int count = 0;

        for (int i = 0; i < visited.length; i++)
            if (visited[i]) count++;

        return count;
    }

    private Map<Integer, List<Integer>> initGames(int n, int[][] results, boolean isWin) {
        int WINNER = isWin ? 0 : 1;
        int LOSER = isWin ? 1 : 0;

        Map<Integer, List<Integer>> map = new HashMap<>();

        for (int i = 1; i <= n; i++) {
            map.put(i, new ArrayList<>());
        }

        for (int[] result : results) {
            int winner = result[WINNER];
            int loser = result[LOSER];
            map.get(winner).add(loser);
        }

        return map;
    }


}
```