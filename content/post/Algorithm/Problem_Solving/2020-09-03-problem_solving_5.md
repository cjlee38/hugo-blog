---
# author: cjlee
categories: problem-solving
# cover: /assets/covers/coding.png
date: "2020-09-03T07:19:00Z"
tags: ['programmers']
title: '# 프로그래머스[Lv.2] - 오픈채팅방 ( java )'
---

[문제 링크](https://programmers.co.kr/learn/courses/30/lessons/42888)

# PROBLEM

오픈채팅방
카카오톡 오픈채팅방에서는 친구가 아닌 사람들과 대화를 할 수 있는데, 본래 닉네임이 아닌 가상의 닉네임을 사용하여 채팅방에 들어갈 수 있다.

신입사원인 김크루는 카카오톡 오픈 채팅방을 개설한 사람을 위해, 다양한 사람들이 들어오고, 나가는 것을 지켜볼 수 있는 관리자창을 만들기로 했다. 채팅방에 누군가 들어오면 다음 메시지가 출력된다.

[닉네임]님이 들어왔습니다.

채팅방에서 누군가 나가면 다음 메시지가 출력된다.

[닉네임]님이 나갔습니다.

채팅방에서 닉네임을 변경하는 방법은 다음과 같이 두 가지이다.

* 채팅방을 나간 후, 새로운 닉네임으로 다시 들어간다.
* 채팅방에서 닉네임을 변경한다.

닉네임을 변경할 때는 기존에 채팅방에 출력되어 있던 메시지의 닉네임도 전부 변경된다.  
예를 들어, 채팅방에 Muzi와 Prodo라는 닉네임을 사용하는 사람이 순서대로 들어오면 채팅방에는 다음과 같이 메시지가 출력된다.

Muzi님이 들어왔습니다.  
Prodo님이 들어왔습니다.  

채팅방에 있던 사람이 나가면 채팅방에는 다음과 같이 메시지가 남는다.

Muzi님이 들어왔습니다.  
Prodo님이 들어왔습니다.  
Muzi님이 나갔습니다.  

Muzi가 나간후 다시 들어올 때, Prodo 라는 닉네임으로 들어올 경우 기존에 채팅방에 남아있던 Muzi도 Prodo로 다음과 같이 변경된다.

Prodo님이 들어왔습니다.  
Prodo님이 들어왔습니다.  
Prodo님이 나갔습니다.   
Prodo님이 들어왔습니다.  

채팅방은 중복 닉네임을 허용하기 때문에, 현재 채팅방에는 Prodo라는 닉네임을 사용하는 사람이 두 명이 있다. 이제, 채팅방에 두 번째로 들어왔던 Prodo가 Ryan으로 닉네임을 변경하면 채팅방 메시지는 다음과 같이 변경된다.

Prodo님이 들어왔습니다.  
Ryan님이 들어왔습니다.  
Prodo님이 나갔습니다.  
Prodo님이 들어왔습니다.  

채팅방에 들어오고 나가거나, 닉네임을 변경한 기록이 담긴 문자열 배열 record가 매개변수로 주어질 때, 모든 기록이 처리된 후, 최종적으로 방을 개설한 사람이 보게 되는 메시지를 문자열 배열 형태로 return 하도록 solution 함수를 완성하라.

제한사항
* record는 다음과 같은 문자열이 담긴 배열이며, 길이는 1 이상 100,000 이하이다.
* 다음은 record에 담긴 문자열에 대한 설명이다.
    * 모든 유저는 [유저 아이디]로 구분한다.
    * [유저 아이디] 사용자가 [닉네임]으로 채팅방에 입장 - Enter [유저 아이디] [닉네임] (ex. Enter uid1234 Muzi)
    * [유저 아이디] 사용자가 채팅방에서 퇴장 - Leave [유저 아이디] (ex. Leave uid1234)
    * [유저 아이디] 사용자가 닉네임을 [닉네임]으로 변경 - Change [유저 아이디] [닉네임] (ex. Change uid1234 Muzi)
    * 첫 단어는 Enter, Leave, Change 중 하나이다.
    * 각 단어는 공백으로 구분되어 있으며, 알파벳 대문자, 소문자, 숫자로만 이루어져있다.
    * 유저 아이디와 닉네임은 알파벳 대문자, 소문자를 구별한다.
    * 유저 아이디와 닉네임의 길이는 1 이상 10 이하이다.
    * 채팅방에서 나간 유저가 닉네임을 변경하는 등 잘못 된 입력은 주어지지 않는다.


입출력 예
* record : ["Enter uid1234 Muzi", "Enter uid4567 Prodo","Leave uid1234","Enter uid1234 Prodo","Change uid4567 Ryan"]
* result :  ["Prodo님이 들어왔습니다.", "Ryan님이 들어왔습니다.", "Prodo님이 나갔습니다.", "Prodo님이 들어왔습니다."]

# SOLVE

위 문제에서, 신경써야 할 부분은 다음과 같다.
1. 닉네임을 몇 번을 변경을 해도, 최종적인 닉네임이 해당 user에게 모두 적용되어야 한다.
2. 닉네임은 중복이 가능하다.
3. 유저는 고유의 id값을 갖는다.

위 내용을 보았을 때, Map을 이용해야겠다는 생각이 들었는데, 그 이유는 다음과 같다.
1. Key-value structure에서, Key는 중복이 불가능, Value는 중복이 가능하다.
2. 따라서, user ID는 key로, 닉네임은 Value로 사용하면 될 것이다.
3. User ID를 tracking 하면, 닉네임을 알 수 있어야 한다.

따라서, 

세 개의 명령어 (Enter, Leave, Change) 중, 가장 먼저 신경써야 할 부분은 Enter와 Change 에 따른 최종 map을 우선 구성하는 것이고, 이를 Enter와 Leave에 따라 문자열을 list에 add 하면 된다.

---
> Note. 만약 이게 실시간 시스템이었다면, record의 list를 그대로 진행하면서 만약 Change가 생긴다면 그 앞에 등장한 Enter와 Leave에서, Change에 해당하는 ID를 찾아서 그 옆에 있는 닉네임 String을 다 새로 바꿔주고.. 해야할 수도 있지만  
> 일단은 문제에서 **최종적으로 방을 개설한 사람이 보게 되는 메시지** 가 관건이므로, Change를 모두 적용 시킨뒤, 그 다음 Enter와 Leave에 따라 문자열을 채워 넣는 방식을 택했다.

---

이를 코드로 작성하면 다음과 같다.
```java
import java.util.LinkedList;
import java.util.HashMap;

class Solution {
    public String[] solution(String[] record) {        
        HashMap<String, String> map = new HashMap<>();
        
        // Change 혹은 Enter인 경우 {userID - nickname}를 map에 등록
        for (int i = 0 ; i < record.length; i++) {
            String[] spt = record[i].split(" ");
            if (spt[0].equals("Change") || spt[0].equals("Enter")) {
                map.put(spt[1], spt[2]);
            }
        }
        
        // 다시 list를 돌면서, id 값에 해당하는 nickname을 list에 추가
        LinkedList<String> list = new LinkedList<>();
        for (int i = 0; i < record.length ; i++) {
            String[] spt = record[i].split(" ");
            if ( spt[0].equals("Enter") ) {
                list.add(map.get(spt[1]) + "님이 들어왔습니다.");
            } else if ( spt[0].equals("Leave")) {
                list.add(map.get(spt[1]) + "님이 나갔습니다.");
            } else {
                continue;
            }
        }
        
        // 최종 결과를 array로 바꿔서 return
        return list.toArray(new String[list.size()]);
    }
}

```