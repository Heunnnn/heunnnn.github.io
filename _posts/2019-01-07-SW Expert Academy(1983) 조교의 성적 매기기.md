---
layout: post
title: 190107 SW Expert Academy(1983) 조교의 성적 매기기
tags:
  - Algorithm
  - SW expert academy
---
## 1983. 조교의 성적 매기기

##### 문제 링크

https://www.swexpertacademy.com/main/code/problem/problemDetail.do?contestProbId=AV5PwGK6AcIDFAUq



##### 풀이 과정

- 총점이 같은 경우는 없으므로 총점으로 학생의 석차를 찾는다.
- 전체 학생의 성적을 내림차순으로 정렬해 석차를 찾고, 석차에 따른 성적을 판정한다.

```java
import java.io.*;
import java.util.ArrayList;
import java.util.Collections;

public class Solution {
    public static void main(String[] args) throws IOException {
        BufferedReader bf=new BufferedReader(new InputStreamReader(System.in));
        StringBuilder sb=new StringBuilder();
        String[] grade={"A+","A0","A-","B+","B0","B-","C+","C0","C-","F"};

        int test=Integer.parseInt(bf.readLine());
        for(int i=0;i<test;i++){
            String[] info=bf.readLine().split(" ");
            int n=Integer.parseInt(info[0]); // 학생 수
            int k=Integer.parseInt(info[1]); // 성적을 찾고싶은 학생의 번호
            ArrayList<Double> list=new ArrayList<>(); // 전체 학생의 성적을 내림차순 리스트로 구성
            double find_score=0.0;// 총점이 같은 경우는 없으므로 성적을 찾고싶은 학생의 총점을 저장, 검색

            for(int j=0;j<n;j++) {
                String[] scores = bf.readLine().split(" ");// 학생 점수 읽어오기
                double total_scores=Double.parseDouble(scores[0])*0.35+Double.parseDouble(scores[1])*0.45+Double.parseDouble(scores[2])*0.2;// 점수 계산
                list.add(total_scores);// 리스트에 점수 추가

                if(j+1==k)
                    find_score=total_scores;
            }

            list.sort(Collections.reverseOrder()); // 성적을 내림차순으로 정렬

            int idx=list.indexOf(find_score);
            idx/=(n/10);// 원하는 학생의 석차를 찾아 성적 계산
            sb.append("#"+(i+1)+" "+grade[idx]+"\n");
        }
        System.out.println(sb);
    }
}
```

- Collections.reverseOrder() -> 내림차순으로 정렬!!!!(큰수->작은수)
- 아직도 java Collection과 입/출력부터 헤메고 있다. 이러면 안댕...

