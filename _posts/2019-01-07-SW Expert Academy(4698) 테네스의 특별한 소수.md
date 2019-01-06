---
layout: post
title: 190107 SW Expert Academy(4698) 테네스의 특별한 소수
tags:
  - Algorithm
  - SW expert academy
  - Sieve of Eratosthenes
---

## 4698. 테네스의 특별한 소수

##### 문제 링크

https://www.swexpertacademy.com/main/code/problem/problemDetail.do?contestProbId=AWRuoqCKkE0DFAXt



##### 풀이

- 에라토스테네스의 체를 이용해 제한범위 내의 소수를 모두 구한다
- a,b 범위 내에서 특별한 소수를 구한다.
  - indexOf() 이용

```java
import java.io.*;
import java.util.ArrayList;

public class Main {
    public static ArrayList<Integer> getPrime(int start, int end){
        // 에라토스테네스의 체 방식으로 소수의 리스트를 얻는다.
        // 아래 방식은 필수로 암기할 것!
        ArrayList<Integer> list=new ArrayList<>();
        for(int i=start;i<=end;i++){
            boolean isPrime=true;
            for(int j=2;j<=(int)Math.sqrt(i);j++){ // i의 제곱근까지 루프
                if(i%j==0){
                    isPrime=false;
                    break;
                }
            }
            if(isPrime && i!=1)
                list.add(i);
        }
        return list;
    }
    public static void main(String[] args) throws IOException {
        BufferedReader br=new BufferedReader(new InputStreamReader(System.in));
        StringBuilder sb=new StringBuilder();
        int test=Integer.parseInt(br.readLine());
        ArrayList<Integer> list=getPrime(1,1000000);
        for(int i=0;i<test;i++){
            String[] dab=br.readLine().split(" ");
            int a=Integer.parseInt(dab[1]);
            int b=Integer.parseInt(dab[2]);
            int result=0;

            for(int l:list){
                if(l<a)
                    continue;
                if(l>b)
                    break;
                if(Integer.toString(l).indexOf(dab[0])!=-1)
                    // 소수가 D를 포함하고 있다면 count
                    result++;
            }

            sb.append("#"+(i+1)+" "+result+"\n");
        }
        System.out.println(sb);
    }
}
```

- 매번 테스트케이스마다 소수리스트를 구했더니 시간초과가 나와서 당황했다 ㅠ
- 제한범위 내의 소수를 모두 구한 다음에 범위 내의 특별한 소수를 구하는 방식으로 변경해서 pass