---
layout: post
title: 190108 SW Expert Academy(4789) 성공적인 공연 기획
tags:
  - Algorithm
  - SW expert academy
---

## 4789. 성공적인 공연 기획

##### 문제 링크

https://www.swexpertacademy.com/main/code/problem/problemDetail.do?contestProbId=AWS2dSgKA8MDFAVT



##### 풀이 과정

- 만약 `claper`의 수가 현재 박수를 쳐야 하는 사람보다 작은 경우 `result`에 더 필요한 사람의 수(고용하는 아르바이트생 수)만큼 계속 더한다. 
-  `claper`=현재 박수치고 있는 사람의 수

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
            String s = br.readLine();
            int result=0;
            int claper=0;
            for(int j=0;j<s.length();j++){
                if(claper<j){
                    result+=j-claper;
                    claper+=j-claper;
                }
                claper+=Integer.parseInt(Character.toString(s.charAt(j)));
            }

            sb.append("#"+i+" "+result+"\n");
        }
        System.out.println(sb);
    }
}
```

