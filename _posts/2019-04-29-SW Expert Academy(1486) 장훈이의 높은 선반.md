---
layout: post
title: 190429 SW Expert Academy(1486) 장훈이의 높은 선반
tags:
  - Algorithm
  - SW expert academy
  - DFS

---

## 1486. 장훈이의 높은 선반

##### 문제 링크

<https://www.swexpertacademy.com/main/code/problem/problemDetail.do?contestProbId=AV2b7Yf6ABcBBASw>



##### 풀이 과정

- 각 점원의 키를 합하거나-합하지 않거나 두 가지 경우의 수가 있다.
- 모든 경우의 수를 수행해 본다->DFS
  - 직원 수가 20명이므로 모든 경우의 수를 수행해 보아도 시간초과X

```java
import java.io.*;
import java.util.*;

class Solution{
    static int answer;
    static int[] arr;
    static int m,n,b;
    public static void sum(int now, int res){
        if(res>=b){
            if(answer>res-b)
                answer=res-b;
            return;
        }// 높이가 b이상인 탑 중 가장 높이가 낮은 경우를 구한다.
        if(now==n)
            return;
		sum(now+1, res+arr[now]);
        sum(now+1, res);
    }
    
	public static void main(String[] args) throws IOException {
        BufferedReader br=new BufferedReader(new InputStreamReader(System.in));
        m=Integer.parseInt(br.readLine());
        for(int j=1;j<=m;j++){
            String[] tmp=br.readLine().split(" ");
            n=Integer.parseInt(tmp[0]);
            b=Integer.parseInt(tmp[1]);
            answer=0;
            
            arr=new int[n];
            tmp=br.readLine().split(" ");
            for(int i=0;i<n;i++){
                arr[i]=Integer.parseInt(tmp[i]);
                answer+=arr[i];
            }
            sum(0, 0);
            System.out.println("#"+j+" "+answer);
        }
    }
}
```