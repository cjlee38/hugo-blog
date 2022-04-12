---
# author: cjlee
categories: data-mining
# cover: /assets/covers/data-mining.png
date: "2020-08-30T22:31:00Z"
tags: ['null']
title: '# Java로 구현하는 연관규칙분석(Association Rule) ( feat. Apriori Algorithm )'
---

# 0. 들어가며
: 2020년 1학기에 수강했던 종합설계 과목에서, 우리 조의 주제는 "시간표 추천 시스템" 이었다.

본인이 원하는 공강시간, 학점 범위, 그리고 내가 미리 골라놓은 듣고자 하는 수업을 선택하면, 이전에 내가 수강했던 과목들을 기반으로 "남는 시간에, 이걸 듣는건 어때?" 라는 개념으로 시간표를 구성해서, 제공해주는 시스템을 제작하고자 하였다.

그 과정에서, "비즈니스프로그래밍2" 수업에서 배웠던 "연관규칙"을 활용할 수 있을 것이라 판단했고, 이에 대해서 공부하기 시작하였다.

# 1. 연관규칙이란?
: 흔히, 장바구니 분석이라고도 한다. 경영학을 전공하고 있는 사람이라면, 마케팅 수업에서 **"맥주를 사는 사람은 대개 기저귀도 같이 산다"(혹은 반대로, 기저귀를 사는 사람은 대개 맥주도 사더라)**  라는 이야기를 들어본 적이 있을 것이다. 

전혀 연관이 없어 보이는 두 item이 왜 자주 같이 등장하는가? 를 살펴봤더니, 퇴근한 아빠가 집에서 아이를 돌보던 엄마로부터 "기저귀좀 사와라" 라는 말에 마트를 들리면서, 겸사겸사 맥주도 한 캔 산다는 이야기다.

즉, **"하나의 item을 샀을 때, 다른 item이 얼마나 자주 등장하는가?"** 를 다루는게 연관규칙 분석이다.

조금 어려운 말로는, **"연관규칙은 데이터베이스에서 Transaction(거래)의 동시출현성향에 대한 관계성을 표현한다"** 고 한다.

---
> Note. Transaction 이란?  
> Transaction이란 "하나의 거래"를 의미한다. 위 예시에서는, "맥주와 기저귀를 구매한 이력"이 된다.   
> 달리 말하면, 하나의 영수증이 Transaction이 된다.

---


# 2. 연관규칙의 평가척도
: 그렇다면, 단순하게 "A를 샀을 때, B가 자주 등장한다" 면, 이를 무조건 연관지어서 생각할 수 있을까? 이를 평가하는 방법에는 여러가지가 있을 것이다. 

이를 찾아보면 정말 다양한 분석 기법이 있지만, 이번 포스팅에서는 이 평가 척도를 **"지지도(Support), 신뢰도(Confidence), 향상도(Lift)"** 의 세가지만 다룬다.


### 1) 지지도(Support)
: 먼저, 지지도란 물건 X와 Y가 있을 때, **"X와 Y가 동시에 등장하는 비율"**을 의미한다. 가령, 전체 거래수가 1,000 건인데, 이 중에서 X와 Y가 동시에 등장하는 횟수가 200이라면, Support는 20%, 즉 0.2 가 된다. 

따라서, Support의 범위는 0 부터 1까지가 된다.

---
> Note 1. 간혹, Support를 "단순등장횟수"로, 즉 위 예시라면 200으로 표현하는 경우도 있긴 한데, 이는 비즈니스 전문가가 판단할 영역이라고 본다. 
>
> 그러나, 내가 진행하는 시간표 추천 시스템은 새로이 제작하는 시스템이고, 이 이용자의 수를 측정할 수 없기 때문에, 전체 Transaction 수로 나누어주었다.

---
> Note 2. "다른 물건과의 비교"가 필요한 다른 평가 척도와 달리, Support는 단독으로 측정될 수도 있다.  
>  즉, 물건 X에 대한 Support, 물건 Y에 대한 Support를 따로 구할 수 있다.
 
---

### 2) 신뢰도(Confidence)
: 다음으로, 신뢰도란, 물건 X와 Y가 있을 때, **"X를 구매한 Transaction 중, Y가 얼마나 포함되는가?"** 를 이야기 한다.

즉, 전체 Transaction이 1,000 건인데, X를 구매한 Transaction이 200건이고, 이 중 Y가 포함된 거래가 100건이라면, 100/200 인 0.5가 된다.

따라서, Support와 마찬가지로 Range는 0부터 1까지가 된다.

---
> Note. 위와 같이 "X를 구매했을 때, Y를 얼마나 구매한다." 라고 할 때,   
> "X를 구매했을 때" 부분을 조건절(Antecedent)  
> "Y를 얼마나 구매한다" 부분을 결과절(Consequent) 이라고 칭한다.

---

### 3) 향상도(Lift)
: 마지막으로 향상도란, 물건 X와 Y가 있을 때, "Y가 없을 때 X를 구매한 횟수 vs Y가 있을 때 X를 구매한 횟수" 가 된다. 팀원은 이를 기울기로 비유했다.

마찬가지로, 전체 Transaction이 1,000 건인데, X를 구매한 Transaction이 200건, Y를 구매한 Transaction이 400건이라고 해보자.  
이 때, Y를 구매했을 때 X가 50건, Y를 구매하지 않았을 때 X가 150건이라면,

((50/1000) / (200/1000) * (400/1000)) = (0.05 / 0.08) = 0.625가 된다.

따라서, Lift 값이 1이라는 것은 서로 연관이 없다는 것을 의미한다.

이를 수식으로 표현하면 다음과 같다.


![assess](/assets/images/2020-08-30-22-19-25_datamining_1.md.png){: .alignCenter}


이러한 평가 척도는 비즈니스 전문가가 이를 섞어서 사용한다. 즉, 상황에 맞춰서 여러 평가척도를 결합하여 활용해야 한다는 것이다.

가령, 무조건 신뢰도가 높은 itemset만을 활용한다면, 전체 Transaction에서 얼마 등장하지도 않은(극단적으로, 한 번만 등장한) itemset에 대해서 유의미하다고 판단할 것이다. 

혹은, 높은 지지도, 낮은 신뢰도의 itemset에 집중하는 것이, 낮은 지지도, 높은 신뢰도의 itemset에 집중하는 것 보다 더욱 높은 수익을 안겨줄 수도 있다.


# 3. Apriori Algorithm

### 1) Assocation Rule as Vanilla
: 위와 같이 평가척도가 존재하는 것을 알았다. 그렇다면, 이에 대해서 연관규칙을 분석 하려면 얼마나 많은 경우의 수가 생길까? 

가령, 다루고 있는 item이 A,B,C,D의 4개가 있다고 해보자. 그러면, 이에 대한 집합은   
Level 1. [(A),(B),(C),(D)] --> 4개  
Level 2. [(A,B), (A,C), (A,D), (B,C), (B,D), (C,D)]  --> 6개  
Level 3. [(A,B,C), (A,B,D), (A,C,D), (B,C,D)] --> 4개  
Level 4. [(A,B,C,D)] --> 1개  

총 15개, 즉 2<sup>4</sup> -1 개가 생긴다.
다루고 있는 item이 10개만 되더라도 2<sup>10</sup>-1 , 즉 1023개가 생긴다.

---
> Note. (A), (B), (C) ... (A,B,C,D)와 같은 집합을, 연관규칙에선 항목집합(Itemset) 이라고 부른다.


---

또한, 이 Itemset들에 대한 연관규칙을 분석하는 것도 상당히 많다.  
가령, (A,B,C) 라는 항목집합에 대해서 Confidence를 구한다고 생각해보자.

조건절 A & 결과절 B,C  
조건절 B & 결과절 A,C  
조건절 C & 결과절 A,B  

즉, A를 구매했을 때, B,C 가 같이 구매될 확률을 구하는 것이다.

앞서, 2<sup>n</sup> -1 개의 경우의 수에, 위와 같은 경우의 수가 추가된다면, 이를 계산하는 것은 대단한 미친짓이다.

[여기](https://rfriend.tistory.com/192?category=706118)를 참고하면, 이러한 Rule의 개수는 3<sup>n</sup>-2<sup>n+1</sup>+1 개가 된다고 한다.


### 2) Association Rule with Apriori Algorithm
: 위와 같은 참사가 발생하는 것을 막기 위해, Apriori Algorithm이 고안되었다.   

---
> Note 1. 연관규칙에서, 연산을 줄이는 전략은 Apriori Algorithm뿐만이 아니라, FP-growth, DHP 등의 다른 알고리즘이 있다. 그러나, Apriori의 장점은, 구현하기가 용이하고, 다른 사람에게 설명하기 좋다는 장점이 있다.


---

> Note 2. Apriori Algorithm과 Association Rule을 동일시하는 글도 본 적이 있는데, 내가 봤을 때에는 엄연히 다르다. Association Rule은 여러 평가 척도를 기반으로 한 분석 기법이고, Apriori는 Association Rule의 비효율성을 방지하기 위한 Algorithm이다.

---

이 알고리즘은 "한 항목이 비빈발(infrequent)집합이라면, 이 항목집합(subset)을 포함하는 모든 항목집합(superset)은 비빈발 항목집합이다" 라는 원칙을 갖고 있다.

무슨말이냐 하면, 위와 같은 A,B,C,D 의 예시에서, 만약 (A) 라는 Itemset이 자주 등장하지 않는다면, 이 (A) 를 포함하는 Itemset[ (A,B), (A,C) ... (A,B,C,D) ]은 쳐다보지도 않는 것이다.

또한, (A)와 같은 단일항목집합에만 해당하는 것이 아니고, (A,B)와 같은 여러개의 item을 포함하고 있는 Itemset에서 출발할 수도 있다.

그렇다면, 비빈발(infrequent)의 기준은 어떻게 정할까? 이 또한 비즈니스 전문가가 정한다. 이를 **"최소지지도"** 라 한다.
(어찌보면, 도메인 지식을 갖고 있는 사람이 이 기준을 정하는 것은 당연하기도 하다.)  
그러나, **"통상적"** 으로, 0.5를 기준으로 둔다고 **"카더라."**


따라서, Apriori 알고리즘의 순서는 다음과 같다.
1. 최소 지지도는 0.5 이다.
2. Level.1의 Itemset[(A),(B),(C),(D)]를 만든다. (이를 C1 이라 한다.)
3. Level.1 에서, 최소지지도를 만족하지 못하는 Itemset을 **쳐낸다.** 여기선, **A와 B가 살아남았다**고 해보자.  
4. 다음으로, 살아남은 녀석들에 대한 Level.2 의 Itemset [(A,B), (A,C), (A,D), (B,C), (B,D)] 를 만든다. (C2)
5. Level.2에서, 최소지지도를 만족하지 못하는 Itemset을 **쳐낸다.**
6. Level이 item의 개수(A,B,C,D의 4개)가 될 때까지 계속해서 반복.

이를 그림으로 표현하면 다음과 같다.

![apriori](/assets/images/2020-08-30-23-31-35_2020-08-30-datamining_1.md.png){: .alignCenter}

[사진 출처](https://blog.naver.com/winipe/150162405537?proxyReferer=https%3A%2F%2Fwww.google.com%2F)


위 그림에서, 우리가 앞으로 "연관규칙분석 할 대상"은 L1, L2, L3 이다.

# 4. 코드로 구현해보자.
: 자바에 익숙치 않아, 클래스 설계를 이렇게 하는 것이 맞는지는 모르겠다. 

Apriori.java
```java
import com.google.common.collect.Sets;
import java.util.*;

/**
 * Apriori Algorithm implementation 
 * @author  cjlee
 * @see     cjlee38.github.io
 */
public class Apriori {
    /**
     * Example :
     * 
     * Set<Set<String>> transactions = getTransactions(); // superset : set of transactions, subset : set of items
     * Float minSupport = (float) 0.5;
     * Apriori apriori = new Apriori(minSupport, transactions)
     * apriori.run();
     */
    private Float minSupport;
    private Set<Set<String>> dataset;
    private Map<Set<String>, Float> result;

    public Apriori(Float minSupport, Set<Set<String>> dataset) {
        this.minSupport = minSupport;
        this.dataset = dataset;
        this.result = new HashMap<>();
    }
    public Map<Set<String>, Float> getResult() {
        return result;
    }

    public void run() {
        Set<String> itemset = createSet(dataset);
        Integer level = 1;
        Integer maxLevel = itemset.size();

        while (level < maxLevel) {
            Set<Set<String>> combinations = getCombinations(itemset, level);
            Map<Set<String>, Float> supportedItemset = getSupportedItemset(combinations);
            result.putAll(supportedItemset);
            itemset = createSet(supportedItemset.keySet());
            level++;
            if (itemset.size() <= 1) { // means current itemsets are all pruned
                break;
            }
        }

    }

    public Set<Set<String>> getCombinations(Set<String> set, Integer level) {
        return Sets.combinations(set, level);
    }

    // 중복없는 데이터를 구하는 함수
    public Set<String> createSet(Set<Set<String>> data) {
        Set<String> set = new HashSet<>();
        for (Set<String> transaction : data) {
            for ( String item : transaction) {
                set.add(item);
            }
        }
        return set;
    }

    public Float calcSupport(Set<String> itemset) {
        Integer occurence = 0;
        for( Set<String> transaction : dataset) {
            if (transaction.containsAll(itemset)) {
                occurence++;
            }
        }
        return occurence / ((float)dataset.size());
    }

    public Map<Set<String>, Float> getSupportedItemset(Set<Set<String>> itemset) {
        Map<Set<String>, Float> map = new HashMap<>();
        for (Set<String> comb : itemset) {
            Float support = calcSupport(comb);
            if (support > minSupport) {
                map.put(comb, support)
            }
        }

        return map;
    }
}
```

AssociationRule.java
```java

import com.google.common.collect.Sets;
import java.util.*;


/*
* 참고문헌
* Association Rule implementation
* @author  cjlee
* @see     cjlee38.github.io
* @refer   김용, Apriori 알고리즘 기반의 개인화 정보 추천 시스템 설계 및 구현에 관한 연구(한국비블리아학회지 VOL.23. NO.4 (2012):283-308,)
* */
public class AssociationRule {
    /**
     * Example : 
     *
     * String metric = "lift";
     * Float minLift = (float)1.0;
     * AssociationRule associationRule = new AssociationRule(apriori.getResult(), metric, minLift); // apriori initialized
     * associationRule.run();
     */
    private Map<Set<String>, Float> itemset;
    private String metric;
    private Float min_threshold;
    private List<AssociationRuleObj> rules;


    public AssociationRule(Map<Set<String>, Float> itemset, String metric, Float min_threshold) {
        this.itemset = itemset;
        this.metric = metric;
        this.min_threshold = min_threshold;
        this.rules = new LinkedList<>();
    }

    public List<AssociationRuleObj> getRules() {
        return rules;
    }

    private Float scoreByMetric(String metric, Float sAC, Float sA, Float sC) {
        Float score;
        if (metric.equals("support")) { score = getSupport(sAC); }
        else if (metric.equals("confidence")) { score = getConfidence(sAC, sA); }
        else if(metric.equals("lift")) { score = getLift(sAC, sA, sC); }
        else { score = null; }

        return score;
    }
    private Float getSupport(Float sAC) { return sAC; }
    private Float getConfidence(Float sAC, Float sA) { return sAC / sA; }
    private Float getLift(Float sAC, Float sA, Float sC) {
        return (sAC/sA) / sC;
    }
    private void filterItemset() {
        itemset.entrySet().removeIf(x -> x.getKey().size() > 2);
    }

    public void run() {
        filterItemset();
        for (Set<String> item : itemset.keySet()) {
            Float sAC = itemset.get(item);
            for (Integer idx = item.size()-1; idx > 0; idx--) {
                Set<Set<String>> combination = Sets.combinations(item, idx);
                for (Set<String> comb : combination) {
                    Set<String> antecedent  = comb;
                    Set<String> consequent = Sets.difference(item, comb);

                    Float sA = itemset.get(antecedent);
                    Float sC = itemset.get(consequent);
                    Float score = scoreByMetric(metric, sAC, sA, sC);
                    if (score >= min_threshold) {
                        rules.add(new AssociationRuleObj(antecedent, consequent, sAC, getConfidence(sAC, sA), getLift(sAC, sA, sC)));
                    }
                }
            }
        }
    }


}
```

AssociationRuleObj.java
```java
import lombok.Getter;
import java.util.Set;

/**
 * Apriori Algorithm implementation 
 * @author  cjlee
 * @see     cjlee38.github.io
 */
@Getter
public class AssociationRuleObj {
    private Set<String> antecedent;
    private Set<String> consequent;
    private Float support;
    private Float confidence;
    private Float lift;

    public AssociationRuleObj(Set<String> antecedent, Set<String> consequent,
                              Float support, Float confidence, Float lift) {
        this.antecedent = antecedent;
        this.consequent = consequent;
        this.support = support;
        this.confidence = confidence;
        this.lift = lift;
    }
}
```

사용법은 코드 내에 작성하였다.  
또한, AssociationRule은 python의 mlxtend 라이브러리를 참고하였다.  


# 5. 활용
: 이렇게 구현한 연관 규칙을, 어떻게 활용할까?

우리 시스템 상에서의 목적은, 연관규칙을 통해, "해당 사용자"에게 걸맞는 "과목별 점수"를 부여하는 것이다.  
따라서, 최소 지지도를 0.5, 향상도가 1 이상인 연관규칙에 대하여, 신뢰도를 점수로 부여하였다.

또한, [참고논문](https://academic.naver.com/article.naver?doc_id=56754379)에 근거하여, 한 itemset의 item이 2인 경우로 한정하였다.


그 결과의 예시는 다음과 같다.(지난 발표자료에서 가져왔다.)

![1](/assets/images/2020-08-31-00-21-10_2020-08-30-datamining_1.md.png){: .alignCenter}

위와 같이 1,2,3,4의 인물이 기존 사용자로 존재한다. 그리고, **5번의 새로운 인물이 새로이 들어왔다**고 해보자. 이에 대한 결과는 다음과 같다.

![2](/assets/images/2020-08-31-00-22-10_2020-08-30-datamining_1.md.png){: .alignCenter}

위와 같은 가중치를 기반으로, "수강가능 과목"에 대한 List를 정렬한 뒤, 앞쪽에서부터 시작하여 Backtracking 기법을 활용해 시간표를 구성하고, 이를 추천하였다.

# 6. 마치며
: Association Rule, Apriori의 library가 존재하기는 하지만, 대부분 Database와 연동해서 가져오는 것으로 보았다.  
또한, 우리 입맛에 맞게 Customizing을 할 수 없었기 때문에, 직접 구현하는.. 사서 고생을 했다.


또한, "가중치"라는 결과로 내뱉기 위한 metric을 어떻게 설정할 것인가에 대해서도 팀원과 논의하면서 어려움을 좀 겪었다.   
혹여나 이 글, 그리고 이러한 방법론이 누군가에게 도움이 된다면 뿌듯하겠다.

### Reference  
- [https://ratsgo.github.io/machine%20learning/2017/04/08/apriori/](https://ratsgo.github.io/machine%20learning/2017/04/08/apriori/)    
- [https://rfriend.tistory.com/191](https://rfriend.tistory.com/191)
- 김용, Apriori 알고리즘 기반의 개인화 정보 추천 시스템 설계 및 구현에 관한 연구  
한국비블리아학회지 VOL.23. NO.4 (2012):283-308