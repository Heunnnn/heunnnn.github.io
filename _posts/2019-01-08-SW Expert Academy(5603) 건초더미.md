---
layout: post
title: 190108 SW Expert Academy(5603) 건초더미
tags:
  - Algorithm
  - SW expert academy
---

## 5603. 건초더미

##### 문제 링크

https://www.swexpertacademy.com/main/code/problem/problemDetail.do?contestProbId=AWXGEbd6cjMDFAUo



##### 풀이 과정

- 건초더미의 크기는 모두 같았다-> 모든 건초더미 크기를 합해 평균을 구하고, 평균값보다 큰 쪽의 건초를 작은쪽으로 옮긴다.

```java
import java.io.BufferedReader;
import java.io.IOException;
import java.io.InputStreamReader;

public class Main {
    public static void main(String[] args) throws IOException {
        BufferedReader br=new BufferedReader(new InputStreamReader(System.in));
        StringBuilder sb=new StringBuilder();
        int T=Integer.parseInt(br.readLine());
        for(int i=1;i<=T;i++){
            int N=Integer.parseInt(br.readLine());
            int result=0;
            int avg=0;
            int s[]=new int[N];

            // 평균을 구한다
            for(int j=0;j<N;j++){
                s[j]=Integer.parseInt(br.readLine());
                avg+=s[j];
            }
            avg/=N;

            // 평균보다 많은 양을 가졌으면 옮긴다
            for(int j=0;j<N;j++) {
                if(s[j]-avg>0) {
                    int tmp = s[j] - avg;
                    result += tmp;
                }
            }

            sb.append("#"+i+" "+result+"\n");
        }
        System.out.println(sb);
    }
}
```

- 이제 D3난이도는 무난하게 푸는것 같다.