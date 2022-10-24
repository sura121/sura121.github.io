---
title: "Devlog: 깊이 우선 탐색 (DFS) / 너비 우선 탐색 (BFS)"
excerpt_separator: "<!--more-->"
categories:
  - Devlog
tags:
  - Devlog
  - Algorithm
---


깊이 우선 탐색 (DFS)
---
 Depth First Search의 약자로 깊이 우선 탐색을 의미합니다.
 stack 혹은 재귀함수(Recursion)을 이용해서 구연할 수 있습니다.
 
  - 경로를 탐색할 수 있는 방향으로 계속 가다가 갈 수 없을때 까지 탐색을 한뒤에 다른 방향으로 검색 시작
  - 모든 경우를 탐색할 때 이 방법을 사용한다.
  
  시간 복잡도 
  
  - 인접리스트 : O(V+E)
  - 인접 행렬 : O(V^2)
  
  ![](https://upload.wikimedia.org/wikipedia/commons/7/7f/Depth-First-Search.gif)
 
 **참고해야할 점**
 
 - 방문했던 정점을 재탐색하지 않기위, 따로 배열을 만들어서 방문한 곳인지 체크해야됩니다.
 - 완전 탐색 입니다.
 - 트리구조에서는 최단거리로 탐색이 가능합니다.

너비 우선 탐색 (BFS)
--

 Breath First Search로 **너비 우선 탐색**을 뜻하는 알고리즘 입니다.
 DFS와는 달리 현재 있는 정점을 다 갈 수 있습니다.
 
 BFS는 큐를 사용합니다. BFS의 구조가 큐와 동일 합니다.
 
 ![](https://cdn.filepicker.io/api/file/6sBaBZQVuci45KJTlGQ9)
 
 표에서 보는 바와 같이
 
 1. A에서 갈 수 있는 곳 X가 큐에들어가고,
 2. X에서 갈 수 있는 H,G가 큐에 들어갑니다.
 3. 그 뒤, H,G 해당 정점들이 갈 수 있는 정점들을 큐에 넣습니다.
 
 이런 식으로 정점들이 갈 수 있는 정점들을 탐색한 뒤, 정점들에서 갈 수 있는 정점을 탐색합니다.
 
 
 


    
 