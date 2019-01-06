---
layout: post
title: 190106 SW Expert Academy(1204) 최빈수 구하기
tags:
  - Algorithm
  - SW expert academy
---
## 1204. 최빈수 구하기

##### 문제 링크

https://www.swexpertacademy.com/main/code/problem/problemDetail.do?contestProbId=AV13zo1KAAACFAYh&categoryId=AV13zo1KAAACFAYh&categoryType=CODE



##### 풀이 과정

- 인덱스로 0~100을 가지는 배열 만들고,  해당 점수=인덱스로 해서 수가 등장하면 값 증가

```java
import java.io.*;
import java.util.StringTokenizer;

public class Solution {
    public static void main(String[] args) throws IOException{
        BufferedReader bf=new BufferedReader(new InputStreamReader(System.in));
        StringBuilder sb=new StringBuilder();

        int test=Integer.parseInt(bf.readLine());
        for(int i=0;i<test;i++) {
            int count[]=new int[101]; // 0~100
            int idx=0;
            int test_case=Integer.parseInt(bf.readLine());
            StringTokenizer st=new StringTokenizer(bf.readLine()," ");
            while(st.hasMoreTokens()){
                int num=Integer.parseInt(st.nextToken());
                count[num]++;// 해당 점수 인덱스의 값을 1 증가
            }
            for(int j=0;j<101;j++){
                if(count[idx]<=count[j])
                    idx=j;
            }
            sb.append("#"+test_case+" "+idx+"\n");
        }

        System.out.println(sb);
    }
}
```

- 실행 시간 : 0.13933s
- BufferedReader, StringTokenizer, StringBuilder에 좀 더 익숙해져야겠다.