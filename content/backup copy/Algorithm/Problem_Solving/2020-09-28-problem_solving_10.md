---
# author: cjlee
categories: problem-solving
# cover: /assets/covers/coding.png
date: "2020-09-28T21:00:00Z"
tags: ['baekjoon']
title: '# 백준[No.17144] - 미세먼지 안녕! ( Java )'
---

# Problem
![Problem](/assets/images/2020-09-28-21-00-14_2020-09-28-problem_solving_10.md.png){: .alignCenter}

# Answer
: 문제가 좀 복잡한데, 어쨌든 시뮬레이션으로 풀었다.  
이 문제에서 신경써야 할 부분은, 보다시피 딱 두 가지다.
- 미세먼지의 확산
- 공기청정기의 작동

처음에는 공기청정기가 로봇청소기처럼 돌아다니면서 청소하는 개념인줄 알았더니,   
그게 아니고 제자리 Fix된 상태에서 공기를 순환시키는 방식으로 동작하는 것이었다.

아무튼, 하나씩 풀어보자.

## 미세먼지의 확산
: 미세먼지는 인접한 네 방향으로만 확산하고,   
벽 혹은 공기청정기에 가로막혀 있으면 확산되지 않는다. 

또한, 본래 자리에 있던 미세먼지는 **빠져나간 미세먼지 만큼만**  
감소하므로, 이를 유의해야 한다.  

이에 더해, **미세먼지의 확산은 동시에 일어나므로,**  
확산되는 숫자를 모아놨다가 한번에 더해줘야 한다.

![init](/assets/images/2020-09-28-21-05-51_2020-09-28-problem_solving_10.md.png){: .alignCenter}


예를 들자면, 그림의 **(1, 8)** 에 위치한 9 에서 확산이 벌어지면,  
두 방향으로 1씩 나눠주므로, 본래 위치의 값은 **7**이 된다.  
**바꿔말하면, -2 라는 값을 받아야 한다.**

그림의 **(2, 8)** 에 위치한 8 에서 확산이 벌어지면,  
세 방향으로 1씩 나눠주므로, 본래 위치의 값은 **5**가 된다.  

그러나, (원래 9에서 원래 8)로 1 만큼, (원래 8에서 원래 9로) 1 만큼  
주기 때문에, 확산이 일어난 후 값은 7과 5가 아닌 **8과 6**이 된다. 

![after](/assets/images/2020-09-28-21-31-56_2020-09-28-problem_solving_10.md.png){: .alignCenter}


기존의 board 에다가 그리면 이러한 동시다발적인 효과를 구현하기 어려우므로,  
emptyMap을 하나 만들어서, 해당 Map에다가 증감의 값을 기록해놓은 뒤,  
한번에 더해주는 방식으로 작성하였다.

```java

private void dustSpread() {
    int[][] newMap = createEmptyMap(); // empty Map
    for (int i=0; i<R; i++) {
        for (int j=0; j<C; j++) {
            if (map[i][j] != 0 && map[i][j] != -1) { // 원래의 map에서 미세먼지를 발견한다면
                spread(newMap, i, j, map[i][j]); // 미세먼지 확산
            }
        }
    }

    addMap(newMap); // 기록해놓은 값을 한번에 더함
}

private void spread(int[][] newMap, int i, int j, int target) {
    int spreadCount = 0;
    if (i != 0 && !isCleaner(i-1, j)) {
        newMap[i-1][j] += target / 5;
        spreadCount++;
    }
    if (i != R-1 && !isCleaner(i+1, j)) {
        newMap[i+1][j] += target / 5;
        spreadCount++;
    }
    if (j != 0 && !isCleaner(i, j-1)) {
        newMap[i][j-1] += target / 5;
        spreadCount++;
    }
    if (j != C-1 && !isCleaner(i, j+1)) {
        newMap[i][j+1] += target / 5;
        spreadCount++;
    }

    newMap[i][j] -= ((target / 5) * spreadCount);
}
```

## 공기청정기의 동작
: 미세먼지의 확산보다, 좀 더 어려웠던 부분이 공기청정기였던 것 같다.

공기청정기는 두 칸을 차지하고, 또 동작하는 방향도 다르므로,   
마치 1x1의 공기청정기가 두 개 있는 것 처럼 생각해서 작성하였다.

그리고, 시계방향 혹은 반시계방향으로 회전할 때에는,  
마치 배열을 한칸씩 뒤로 미룬뒤 삽입할 때 처럼,  
prev와 cur를 사용해서 동작하면 된다.

*p.s.1 코드를 다시 보니까 리팩토링할 부분이 한두군데가 아닌 것 같다.*  
*p.s.2 문제를 풀 때 뭔 생각이었는진 모르겠는데, x와 y가 바뀌어 있다.*


**전체 코드**
```java
package BOJ;

import java.io.BufferedReader;
import java.io.IOException;
import java.io.InputStreamReader;
import java.util.StringTokenizer;

// 백준 - 미세먼지 안녕!
public class bj17144 {

    static Solution17144 init() throws IOException {
        BufferedReader br = new BufferedReader(new InputStreamReader(System.in));
        StringTokenizer st = new StringTokenizer(br.readLine());
        int R = Integer.parseInt(st.nextToken());
        int C = Integer.parseInt(st.nextToken());
        int T = Integer.parseInt(st.nextToken());

        int[][] map = new int[R][C];

        for (int i = 0; i < R; i++) {
            st = new StringTokenizer(br.readLine());
            for (int j = 0; j < C; j++) {
                map[i][j] = Integer.parseInt(st.nextToken());
            }
        }

        return new Solution17144(R, C, T, map);
    }

    public static void main(String[] args) throws IOException {
        Solution17144 s = init();
        int answer = s.run();
        System.out.println(answer);
    }
}

class Solution17144 {
    final private int R;
    final private int C;
    final private int T;

    private int[][] map;

    public Solution17144(int R, int C, int T, int[][] map) {
        this.R = R;
        this.C = C;
        this.T = T;
        this.map = map;
    }

    public int run() {
        int answer = 0;
        AirCleaner posA = findClenaerPos(0);
        AirCleaner posB = findClenaerPos(1);

        for (int i=0; i<T; i++) {
            dustSpread();
            posA.clean(map);
            posB.clean(map);
        }

        answer = getResult();

        return answer;
    }

    private AirCleaner findClenaerPos(int number) {
        AirCleaner AC = null;
        for (int i=0; i<R; i++) {
            for (int j=0; j<C; j++) {
                if (map[i][j] == -1) {
                    if (number == 0) {
                        AC = new AirCleaner(i, j, AirCleaner.CLOCKWISE);
                    } else {
                        AC = new AirCleaner(i+1, j, AirCleaner.COUNTER_CLOCKWISE);
                    }
                    return AC;
                }
            }
        }

        return AC;
    }

    private void dustSpread() {
        int[][] newMap = createEmptyMap();
        for (int i=0; i<R; i++) {
            for (int j=0; j<C; j++) {
                if (map[i][j] != 0 && map[i][j] != -1) {
                    spread(newMap, i, j, map[i][j]);
                }
            }
        }

        addMap(newMap);
    }

    private void spread(int[][] newMap, int i, int j, int target) {
        int spreadCount = 0;
        if (i != 0 && !isCleaner(i-1, j)) {
            newMap[i-1][j] += target / 5;
            spreadCount++;
        }
        if (i != R-1 && !isCleaner(i+1, j)) {
            newMap[i+1][j] += target / 5;
            spreadCount++;
        }
        if (j != 0 && !isCleaner(i, j-1)) {
            newMap[i][j-1] += target / 5;
            spreadCount++;
        }
        if (j != C-1 && !isCleaner(i, j+1)) {
            newMap[i][j+1] += target / 5;
            spreadCount++;
        }

        newMap[i][j] -= ((target / 5) * spreadCount);
    }

    private int[][] createEmptyMap() {
        int[][] newMap = new int[R][C];
        for (int i=0; i<R; i++) {
            for (int j=0; j<C; j++) {
                newMap[i][j] = 0;
            }
        }

        return newMap;
    }

    private boolean isCleaner(int i, int j) {
        if (map[i][j] == -1) {
            return true;
        } else {
            return false;
        }
    }

    private void addMap(int[][] newMap) {
        for (int i=0; i<R; i++) {
            for (int j=0; j<C; j++) {
                map[i][j] += newMap[i][j];
            }
        }
    }

    public int getResult() {
        int result = 0;
        for (int i=0; i<R; i++) {
            for (int j=0; j<C; j++) {
                if (map[i][j] ==-1) {
                    continue;
                } else {
                    result += map[i][j];
                }
            }
        }

        return result;
    }

}

class AirCleaner {
    final static int CLOCKWISE = 0;
    final static int COUNTER_CLOCKWISE = 1;


    final private int X;
    final private int Y;
    final private int clockwise;
    public AirCleaner(int x, int y, int clockwise) {
        X = x;
        Y = y;
        this.clockwise = clockwise;
    }

    public void clean(int[][] map) {
        int x = X;
        int y = Y+1;
        int R = map.length;
        int C = map[0].length;
        int cur;
        int prev = map[x][y-1];

        // go right
        for ( ; y<C-1; y++) {
            cur = map[x][y];
            map[x][y] = map[x][y-1] == -1 ? 0 : prev;
            prev = cur;
        }

        // go up or down
        if (clockwise == CLOCKWISE) {
            for ( ; x<R-1; x++) {
                cur = map[x][y];
                map[x][y] = prev;
                prev = cur;
            }
        } else {
            for ( ; x>0; x--) {
                cur = map[x][y];
                map[x][y] = prev;
                prev = cur;
            }
        }

        // go left
        for ( ; y>0; y--) {
            cur = map[x][y];
            map[x][y] = prev;
            prev =cur;
        }

        // go up or down
        if (clockwise == CLOCKWISE) {
            for ( ; x>X ; x--) {
                cur = map[x][y];
                map[x][y] = prev;
                prev = cur;
            }
        } else {
            for ( ; x<X ; x++) {
                cur = map[x][y];
                map[x][y] = prev;
                prev = cur;
            }
        }
    }
}
```