---
layout: post
title: 190510 SW Expert Academy(3074) 입국심사
tags:
  - Algorithm
  - SW expert academy
  - Binary Search
---

## 3074. 입국심사

##### 문제 링크

<https://www.swexpertacademy.com/main/code/problem/problemDetail.do?contestProbId=AV_XEokaAEcDFAX7&categoryId=AV_XEokaAEcDFAX7&categoryType=CODE>



##### 풀이 과정

- left=1, right=가장 오래 걸릴 때의 시간(모든 사람이 가장 오래 걸리는 심사대에서 심사받을 때)에서 이분탐색 시작, left가 right보다 커질 때까지 계속 탐색

- (최종 걸린시간)=(사람)*(심사대당 걸리는 시간)이므로 (사람)=(최종 걸린시간)/(심사대당 걸리는 시간) 으로 표현할 수 있다.
- 만약 계산한 사람 수가 총 사람수보다 작은 경우는 left를 mid값보다 크게 해야 하고, 반대의 경우는 right를 하나 줄여 최종 걸린 시간의 범위를 좁힌다.



```java
import java.io.BufferedReader;
import java.io.IOException;
import java.io.InputStreamReader;

class Solution {
    public static void main(String[] args) throws IOException{
        BufferedReader br=new BufferedReader(new InputStreamReader(System.in));
        int T=Integer.parseInt(br.readLine());
        for(int i=1;i<=T;i++){
            String[] tmp=br.readLine().split(" ");
            int N=Integer.parseInt(tmp[0]); // 심사대 수
            int M=Integer.parseInt(tmp[1]); // 사람수
            int[] times=new int[N];
            int max=0;
            for(int j=0;j<N;j++){
                times[j]=Integer.parseInt(br.readLine());
                max=Math.max(max,times[j]);
            }

            long left=1;
            long right=max*(long)M;
            long total=0, mid=0;
            while(left<=right){
                mid=(left+right)/2;
                total=0;
                for(int k=0;k<N;k++)
                    total+=mid/times[k];
                if(total<M)
                    left=mid+1;
                else
                    right=mid-1;
            }
            System.out.println("#"+i+" "+left);
        }
    }
}
```