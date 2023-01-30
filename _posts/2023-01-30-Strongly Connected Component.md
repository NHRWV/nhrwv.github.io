---
title: "강한 연결 요소(Strongly Connected Component)"
date: 2023-01-30 17:00:00 +0900
categories:
- PS
- 글쓰기
tags:
- 글쓰기
- 강한연결요소
- SCC
- 알고리즘
---

# 강한 연결 요소(Strongly Connected Component) 

2-sat에 대해 공부하기 전에 강한 연결 요소를 복습할 필요를 느껴서 이번에 정리하려고 합니다.  

SCC를 다루기 전에 필요한 개념 정리부터 하겠습니다.



### DFS(깊이 우선 탐색)에서 그래프 간선 분류

방향 그래프에서 DFS를 거쳐간 간선을 따라가면 DFS 스패닝 트리를 얻을 수 있습니다. 그리고 DFS 스패닝 트리가 속한 방향 그래프 간선들은 네 가지로 구분됩니다. **트리 간선을 제외한 나머지 간선들은 DFS 스패닝 트리에 속하지 않아요.**

- 트리 간선(Tree Edge) : 트리를 구성하는 간선
- 순방향 간선(Forward Edge) : DFS 스패닝 트리에 속한 노드 간에서 자손 노드로 향하는 간선
- 역방향 간선(Back Edge) : DFS 스패닝 트리에 속한 노드 간에서 조상 노드로 향하는 간선
- 교차 간선(Cross Edge) : 해당 간선으로 연결된 노드는 조상과 자손 관계가 아닌, 아무것도 아닌 간선


아래에 정점이 7개, 간선이 11개인 방향 그래프가 있는데요. 해당 그래프에서 1번 노드(정점)으로부터 DFS를 실시하면 두 번째 그림을 그려볼 수 있습니다.  

<img src="https://i.imgur.com/j3JV6kB.jpg" width="700" height="650"/>  
<img src="https://i.imgur.com/kWo4cKf.jpg" width="700" height="650"/>


위의 그림에서 회색 형광펜으로 칠해진 간선은 트리 간선(1-2, 2-4, 1-3, 3-5, 5-6, 5-7)입니다.  
푸른 형광펜으로 칠해진 간선은 순방향 간선(1-5)입니다. ~~보라색 같긴 한데~~  
붉은 형광펜으로 칠해진 간선은 역방향 간선(4-1, 6-5, 7-5)입니다. ~~이건 갈색 같네요~~  
노란 형광펜으로 칠해진 간선은 교차 간선(6-4)입니다.  

<br/>


### 강한 연결 요소

강한 연결 요소(Strongly Connected Component)는 임의의 방향 그래프 내에서 다음 조건을 만족하는 부분집합입니다.

1. SCC 그룹 내의 어떤 두 정점 사이에는 서로 이어진 경로가 존재한다. 이는 사이클이 존재한다는 것을 의미한다.
2. 서로 다른 SCC 그룹에 속한 정점끼리는 양방향으로 이어져 있지 않다.
3. SCC 그룹은 1번 조건을 만족하는 최대 부분집합이다. 즉, 가능한 커야 한다.

그래프에서 SCC 그룹을 하나의 정점이라고 생각한다면, 이렇게 SCC 그룹 정점으로 이루어진 그래프에는 싸이클이 존재하지 않습니다. 만약 SCC 그룹 A와 B 사이에 싸이클이 존재한다면 상대 SCC 그룹으로 언제든지 갈 수 있다는 것을 의미하며, 이는 SCC 그룹이 가능한 최대 크기로 생성되어야 한다는 성질을 위반함과 동시에 A와 B는 함께 SCC 그룹을 형성했어야 한다는 사실을 알 수 있습니다.

<br/>


### 알고리즘 설명

타잔(Tarjan)과 코사라주(Kosaraju)가 대표적인 강한 연결 요소 알고리즘입니다. 이 글에서는 타잔 알고리즘으로 설명하겠습니다.

<img src="https://i.imgur.com/aQl1BVF.jpg" width="700" height="650"/>  

위 그림은 기존 그래프에서 교차 간선을 제거한 모습입니다. 그리고 노드 왼쪽 상단에 DFS 방문 순서를 표기했습니다. 사실 순방향 간선은 지금부터 소개할 내용에서 필요 없기 때문에 파란~~푸른~~ 형광색으로 칠해진 간선은 신경 쓰지 않으셔도 됩니다.



```python
# v = 정점 개수

stack = []

num = [0] * (v+1) # 노드 방문 순서
count = 1
check = [0] * (v+1) # 노드 방문 & 스택에서 꺼내어져 어느 SCC 그룹에 속하는지 확인 완료 여부.
answer = []


def sol(x):
    global count
    
    num[x] = count # DFS 방문 순서 기록과 방문 여부 확인. count 값은 1부터 시작.
    stack.append(x)
    
    count += 1
    group = num[x]
    # group은 현재 노드의 num 배열 값인데, 이 값과 자기 자손의 group 값(sol 함수 결과값) 중에서 가장 작은 값을 선택하게 될 것이다.
    # 같은 SCC 그룹에 속한다면 최종적으로 같은 값을 가지게 될 것이다.
    
    for i in edge[x]:
        if not num[i]: # 이번 노드가 첫 방문인지?
            group = min(group, sol(i))
            
        elif not check[i]: # 방문한 적이 있으나 아직 처리 중(스택에서 꺼내지 않은 상태)인 노드인 경우
            group = min(group, num[i])
            
    
    # 현재 정점과 그 밑에 딸린 자손 정점들이 트리 상에서 더 이상 올라갈 다른 곳이 없어서 현재 정점에서 멈춘 경우.
    if group == num[x]:
        temp = []
        
        # 같은 그룹이 될 아이들을 스택에서 꺼내기 시작. 스택에서 현재 정점부터 시작해서 그 위에 쌓여 있는 정점을 모두 꺼낸다.
        while True:
            t = stack.pop()
            temp.append(t)
            check[t] = 1 # 모든 작업 완료 표시. 해당 정점은 이제 신경 쓰지 않아도 된다.
            
            if t == x:
                break
            
        answer.append(sorted(temp)) # 추출한 SCC 그룹을 담는다.
        
    
    return group
        
```

바로 위의 코드는 타잔 알고리즘의 정수인 DFS 함수입니다. 여기서 눈 여겨 봐야 하는 배열이 두 개 있는데요, num과 check 배열입니다.  
num 배열은 처음 값이 0인데, 방문 순서는 1부터 시작하기에 방문 여부 확인 역할도 수행합니다.  
check 배열은 해당 인덱스의 정점이 속한 SCC 그룹 확인까지 끝내서 모든 작업이 완료되었는지에 대한 여부를 기록합니다.  

코드를 보면 핵심 과정은 다음과 같습니다.  

1. 각 정점을 sol 함수로 방문하는 동시에 스택에 넣어줍니다. 
2. 그리고 해당 정점과 이어진 정점을 살펴서 첫 방문 여부 또는 check 배열 값을 확인합니다. 자신뿐만 아니라 자손 노드가 더 이상 방문할 곳이 없어야 하기에 역방향 간선으로 방문할 수 있다면 여전히 방문할 조상 노드도 check 배열 값이 0입니다. 
3. 현재 정점과 그 밑에 딸린 자손 정점들이 트리 상에서 더 이상 방문할 수 있는 다른 조상 정점이 없다면 하나의 SCC 그룹을 찾은 것이기 때문에 스택에서 현재 정점부터 시작해서 그 위에 쌓여 있는 정점을 모두 추출합니다.
  

글만 보면 잘 안 와닿으니 이제 그림으로 보아요.

<img src="https://i.imgur.com/eYTfotk.jpg" width="700" height="650"/>  

위 그림에서 보이는 것처럼 1번 노드부터 DFS를 실행하면 1-2-4 순서로 4번 노드까지 쭉 내려옵니다. 노드에 첫 방문할 때는 스택에 해당 노드를 넣어줍니다. 4번 노드에 이르렀을 때 다음에 갈 수 있는 곳은 역방향 간선을 통한, 이미 방문한 적이 있는 조상 노드 1번이에요. 1번 노드는 이미 방문했지만, 여전히 check 배열 상태가 0입니다.

<br/>

<img src="https://i.imgur.com/EWaf2W0.jpg" width="700" height="650"/>  

4번에서 1번 노드로 올라가면, 다시 트리 간선을 통해 1-3-5-6 순서로 6번 노드로 내려갈 거예요. 그리고 6번 노드에 이르렀다면 역방향 간선을 통해 조상 노드인 5번 노드로 다시 갈 수 있습니다.

<br/>

<img src="https://i.imgur.com/Wlvuadn.jpg" width="700" height="650"/>  

다시 방문한 5번 노드에서 이제 우리는 트리 간선을 통해 7번 노드로 갈 수 있습니다. 그런데 7번 노드는 리프 노드이기 때문에 역방향 간선으로 조상 노드인 5번으로 다시 돌아갈 수밖에 없네요.

<br/>

<img src="https://i.imgur.com/kKSCdRZ.jpg" width="700" height="650"/>  

위 그림에서 보이는 바와 같이 5번 노드에 세 번째 방문했는데, 이제 5번 노드는 새로 방문할 다른 곳도 없으며 SCC 추출 조건을 만족하는 상태입니다. 즉, 5번 노드와 그 자손 노드가 도달 가능한 최고 조상 노드가 5번 노드인 상황이에요.

따라서 스택의 5번 노드부터 그 위에 쌓인 노드를 모두 추출합니다. 추출된 노드로 새로운 SCC 그룹을 이루게 되며, 동시에 해당 노드들의 check 배열 값을 1로 만들면 됩니다.

<br/>

<img src="https://i.imgur.com/X9c29Aq.jpg" width="700" height="730"/>  

다시 3번 노드 로 돌아가면 3번 노드도 방금 5번 노드의 상황처럼 SCC 추출 조건을 만족합니다. 3번 노드는 홀로 SCC 그룹을 생성하게 되네요. 스택에서 3번 노드를 추출함과 동시에 check 배열 상태를 1로 설정해줍니다.

<br/>

<img src="https://i.imgur.com/kT94mZu.jpg" width="700" height="730"/>  

이제 1번 노드로 돌아갔는데, 또 다시 바로 SCC 추출 조건을 만족하는 상황입니다. 아까 한 것처럼 스택에서 1, 2, 4로 이루어진 새로운 SCC 그룹을 추출합니다.  

<br/>

<img src="https://i.imgur.com/QfeaJNR.jpg" width="700" height="730"/>  

루트 노드인 1번까지 스택에서 비워지면서 작업이 완료됩니다. 임의의 서로 다른 두 정점이 연결된 연결 그래프가 아니라면 아직 남아 있는 정점에서 새롭게 sol 함수를 실행하면 되겠죠.


<br/>



### 전체 예시 코드

아래는 [Strongly Connected Component](https://www.acmicpc.net/problem/2150) 문제의 소스 코드입니다.

```python
import sys
input = sys.stdin.readline
sys.setrecursionlimit(int(1e6))

v, e = map(int, input().split())

edge = [[] for _ in range(v+1)]
for i in range(e):
    a, b = map(int, input().split())
    edge[a].append(b)
    
    
stack = []

num = [0] * (v+1) # 노드 방문 순서
count = 1
check = [0] * (v+1) # 노드 방문 & 스택에서 꺼내어져 어느 SCC 그룹에 속하는지 확인 완료 여부.
answer = []


def sol(x):
    global count
    
    num[x] = count # DFS 방문 순서 기록과 방문 여부 확인. count 값은 1부터 시작.
    stack.append(x)
    
    count += 1
    group = num[x]
    # group은 현재 노드의 num 배열 값인데, 이 값과 자기 자손의 group 값(sol 함수 결과값) 중에서 가장 작은 값을 선택하게 될 것이다.
    # 같은 SCC 그룹에 속한다면 최종적으로 같은 값을 가지게 될 것이다.
    
    for i in edge[x]:
        if not num[i]: # 이번 노드가 첫 방문인지?
            group = min(group, sol(i))
            
        elif not check[i]: # 방문한 적이 있으나 아직 처리 중(스택에서 꺼내지 않은 상태)인 노드인 경우
            group = min(group, num[i])
            
    
    # 현재 정점과 그 밑에 딸린 자손 정점들이 트리 상에서 더 이상 올라갈 다른 곳이 없어서 현재 정점에서 멈춘 경우.
    if group == num[x]:
        temp = []
        
        # 같은 그룹이 될 아이들을 스택에서 꺼내기 시작. 스택에서 현재 정점부터 시작해서 그 위에 쌓여 있는 정점을 모두 꺼낸다.
        while True:
            t = stack.pop()
            temp.append(t)
            check[t] = 1 # 모든 작업 완료 표시. 해당 정점은 이제 신경 쓰지 않아도 된다.
            
            if t == x:
                break
            
        answer.append(sorted(temp)) # 추출한 SCC 그룹을 담는다.
        
    
    return group
        
        
for i in range(1, v+1):
    if not num[i]:
        sol(i)
        
print(len(answer))

for i in sorted(answer):
    print(*i, -1)
    
```
