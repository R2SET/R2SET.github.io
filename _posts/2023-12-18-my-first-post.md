---
layout: post
title:  "Dijkstra 알고리즘"
excerpt: "다익스트라의 기본 설정과 간단한 문제들의 풀이를 배워보자"

categories:
  - Algorithm
  
toc: true
toc_sticky: true
 
date: 2023-12-18
last_modified_at: 2023-12-28
---
#Dijkstra(다익스트라) 알고리즘
###**간단설명:** BFS(넓이우선탐색)과 heapq 자료 구조를 이용하여 모든 노드까지 가는 최단 경로를 구하는 알고리즘
---
###목차
>1.다익스트라의 아이디어
>1-1 사전지식
>2.코드 구현
>3.코드 상세 설명
>4.마무리

___
## 다익스트라의 아이디어
최단 경로 알고리즘의 아이디어는 다음과 같다
자료 구조로는 graph와 heapq를 사용하며, 입력을 바탕으로 그래프에 해당하는 인덱스에 도착노트와 가중치를 포함한 집합을 표현한 후, 그 그래프를 바탕으로 다익스트라를 진행함

>출발 노드와, 도착 노드, 거리값을 바탕으로 그래프생성 (1)
출발 노드부터 시작하여, 인접한 노드까지의 거리를 계산한 다음, 
현재 알고있는 거리보다 짧으면 해당 값으로 갱신 
현재 노드에 인접한 모든 미방문 노드까지의 거리를 계산했다면, 
현재 노드는 방문한 것이므로, 미방문 집합에서 제거 (2)

다익스트라 알고리즘은 결국 크게 위에서의 (1),(2)번 단계로 구분할 수 있다
___
## 사전지식
우선 이 게시글에서 BFS와 heapq에 대해서는 
자세히 설명하지 않을 예정이며 이에 대한 자세한 설명이 필요할 경우에는 아래 링크를 참고
[BFS](https://itholic.github.io/python-bfs-dfs/#google_vignette)
[heapq](https://www.daleseo.com/python-heapq/)
전반적인 코드의 느낌은 BFS의 느낌과 흡사하나, 기존의 BFS 알고리즘이 큐 자료구조를 이용하는 것과 다르게 다익스트라는 heapq 자료구조를 이용하여 알고리즘을 구성한다
이를 설명하기 위해서는 시간복잡도에 대한 기초개념이 필요하나 이에 대한 설명은 생략하며 설명이 필요할 경우엔 아래 링크를 참고
[시간복잡도(Big-O)](https://velog.io/@gillog/%EC%8B%9C%EA%B0%84%EB%B3%B5%EC%9E%A1%EB%8F%84#:~:text=%EA%B0%84%20%EB%B3%B5%EC%9E%A1%EB%8F%84%EB%8A%94%20%EB%AC%B8%EC%A0%9C%EB%A5%BC%20%ED%95%B4%EA%B2%B0%ED%95%98%EB%8A%94%EB%8D%B0%20%EA%B1%B8%EB%A6%AC%EB%8A%94%20%EC%8B%9C%EA%B0%84%EA%B3%BC%20%EC%9E%85%EB%A0%A5%EC%9D%98%20%ED%95%A8%EC%88%98,%EB%B9%85-%EC%98%A4%20%ED%91%9C%EA%B8%B0%EB%B2%95%EC%9D%80%20%EA%B3%84%EC%88%98%EC%99%80%20%EB%82%AE%EC%9D%80%20%EC%B0%A8%EC%88%98%EC%9D%98%20%ED%95%AD%EC%9D%84%20%EC%A0%9C%EC%99%B8%EC%8B%9C%ED%82%A4%EB%8A%94%20%EB%B0%A9%EB%B2%95%EC%9D%B4%EB%8B%A4.)
만약 우선순위 큐를 사용하지 않고 코드를 작성하게 된다면, 시간복잡도는 ***O(V^2)*** 로 표현된다. 여기서 V(Vertex)는 주어진 간선의 개수이고, 만일 V의 개수가 10,000개가 넘는 상황이라면, 시간초과에 걸리지 않고 코드를 실행하기 어려운 상황에 직면할 가능성이 존재한다
**하지만**, 만약 우선순위 큐를 사용하여 구현하게 된다면, 노드를 순환하며 그 노드와 인접한 노드를 같은 넓이인 노드를 먼저 순환하게 된다
그 후에 기준이 되는 노드는 우선순위 큐로 인해 정렬된, 가장 거리가 가까운 노드가 되므로 이에 대한 시간복잡도는 ***O(ElogV)*** 로 나타낼 수 있고 시간적으로 훨씬 개선된 모습을 보인다
코드 내에 등장하는 distance내의  INF는 무한을 의미하며 int(10e9)으로 표현하였고, 그 이유는 가중치 값으로 10e9보다 큰 정수가 들어오는 경우가 많지 않기 때문이라고 이해하면 된다
___
## 전체 코드
우선순위 큐를 활용한 파이썬 코드는 다음과 같다
```
import heapq
import sys
input = sys.stdin.readline
INF = int(1e9)
n, e = map(int, input().split())
start = int(input())
graph = [[] for i in range(n+1)]
distance = [INF] * (n+1) #사전작업으로써 빈 그래프, INF, 시작노드 설정

for _ in range(e):
    a, b, cost = map(int, input().split())
    graph[a].append((b, cost))

def dijkstra(start):
    pq = []
    heapq.heappush(pq, (0, start))
    distance[start] = 0
    while pq:
        dist, n_now = heapq.heappop(pq)
        if distance[n_now] < dist:
            continue
        for i in graph[n_now]:
            c = dist + i[1]
            if c < distance[i[0]]:
               distance[i[0]] = c
               heapq.heappush(pq, (c, i[0]))
dijkstra(start)
for i in range(1, n+1):
    if distance[i] == INF:
        print("INF")
    else: 
        print(distance[i])
```
위의 내용을 위에서 언급한 대로 쪼개어 각각 설명하면 다음과 같다
___
## 코드 상세 설명
####아래의 코드는 (1)번 과정에 해당하는 코드이다
```
import heapq
import sys
input = sys.stdin.readline
INF = int(1e9)
n, e = map(int, input().split())
start = int(input())
graph = [[] for i in range(n+1)]
distance = [INF] * (n+1)
for _ in range(e):
    a, b, cost = map(int, input().split())
    graph[a].append((b, cost))
```
*(*참고: sys.stdin.readine을 사용하는 이유는 입력을 할 때의 걸리는 시간을 최소하기 위함)*
1. sys와 heapq를 import 해준 뒤
2. 10e9를 INF(무한)으로 설정*(이유는 주로 10e9보다 큰 입력값이 주어지지 않는 경우가 많기 때문이다)*
3. 노드의 수와 간선의 개수를 받아준 뒤엔 시작 노드와 빈 그래프를 생성
4. 간선의 개수 e만큼 a,b,cost를 받아서 그래프의 a번째 리스트에 집합을 append해줌
####2번에 대한 코드와 내용은 다음과 같다
```
def dijkstra(start):
    pq = []
    heapq.heappush(pq, (0, start))
    distance[start] = 0
    while pq:
        dist, n_now = heapq.heappop(pq)
        if distance[n_now] < dist:
            continue
        for i in graph[n_now]:
            c = dist + i[1]
            if c < distance[i[0]]:
               distance[i[0]] = c
               heapq.heappush(pq, (c, i[0]))
dijkstra(start)
for i in range(1, n+1):
    if distance[i] == INF:
        print("INF")
    else: 
        print(distance[i])
```
우선순위 큐 pq를 생성한 후에 (0,start)를 push하고 distance의 start번째의 숫자를 0으로 처리한다(*자기자신과의 거리 = 0)
이후엔 pq에서 가장 거리가 가까운 노드와 그 거리를 pop하고
>if 현재 distance 숫자가 빼온 dist보다 작을 경우:
방문처리가 되었다 or 할 필요가 없다 간주

>else:
방문처리가 되지 않았다고 간주 & 현재 처리중인 노드와 인접한 노드 조사
if 현재 노드를 거쳐 다른 노드로 가는 거리가 더 짧음:
distance 갱신, pq에 (거리, 노드정보) push

해준 뒤, pq가 완전히 빌 때까지 이 작업을 반복한다
___
####마무리
최단 경로 알고리즘(다익스트라)은 지하철 노선도, 네비게이션 등 다방면에 사용되는 알고리즘이며 실생활에서 거리를 계산하는 대다수의 어플에서 사용하고 있는 알고리즘이다. 조금 난도가 있는 알고리즘이라고 생각하나(내 기준이다) 파이썬으로 한번 구현을 해보는 연습을 하는 것도 좋을 것이라 생각한다
