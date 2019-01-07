---
layout: post
title: 190107 SW Expert Academy(6692) 다솔이의 월급 상자
tags:
  - Algorithm
  - SW expert academy
---

## 6692. 다솔이의 월급 상자

##### 문제 링크

https://www.swexpertacademy.com/main/code/problem/problemDetail.do?contestProbId=AWdXofhKFkADFAWn



##### 풀이 과정

- pi*xi값을 모두 더하면 끝.
- 이 문제는 왜 난이도가 D3인 것인가...?

```java
import java.io.BufferedReader;
import java.io.IOException;
import java.io.InputStreamReader;

public class Solution {
    public static void main(String[] args) throws IOException {
        BufferedReader br=new BufferedReader(new InputStreamReader(System.in));
        StringBuilder sb=new StringBuilder();
        int T=Integer.parseInt(br.readLine());
        for(int i=1;i<=T;i++){
            int N=Integer.parseInt(br.readLine());
            double result=0.0;
            for(int j=1;j<=N;j++){
                String[] value=br.readLine().split(" ");
                result+=Double.parseDouble(value[0])*Double.parseDouble(value[1]);
            }
            sb.append("#"+i+" "+result+"\n");
        }
        System.out.println(sb);
    }
}
```

