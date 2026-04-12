### NeetCode版 (https://neetcode.io/problems/count-connected-components/question) を解いています
## Step.0
まずは自力で解く  
グラフのノード(点)の個数とエッジ(辺)の情報が与えられて、グラフの個数を答える  
実際にグラフの関係を何かにまとめられたら一番良いのだろうが、やり方が分からない  
ノードが属するグループをリストで表現したら良いのか  
リスト`node_group`で保持しておく  
一旦思いついたアイデア↓  
エッジ情報`edges`の要素の一つを`edge = [a, b]`として、  
 - (i) `a`または`b`が`node_group`のどこかにあり、もう片方がない場合  
        `a`または`b`が存在しているグループに、`b`または`a`を追加する  
 - (ii) `a`と`b`が、すでに同じグループに存在する場合  
        特に何もしない  
 - (iii) `a`と`b`が、すでに別々のグループに存在する場合  
        `a`, `b`が存在するグループを結合する  

ただ、このままではノードの生成・条件分岐が多くて面倒  
書こうとしたけど面倒すぎてこのやり方はまずいと判断  

今回、ノードは`0` ~ `n-1`までの`n`個登場することは分かっているから、  
ノードのみのグラフを`0`から`n-1`まで一通り作り、  
エッジ情報をもとにグループの統合を行えば良い！  

これなら`edge = [a, b]`として、 `a`のグループと`b`のグループの統合のみを順番に行えば大丈夫  

統合は、`a`のグループに`b`のグループを追加(updateで可能)  
した後に`b`グループを削除してばできる (removeで可能)  

`update`メソッドは、二つのグループを一つのセットに入れたものをリストが返すような感じで自分でも作れる  

コードに起こしてみる  

## 書いたコード1
```python
class Solution:
    def countComponents(self, n: int, edges: List[List[int]]) -> int:
        node_groups = [{i} for i in range(n)]

        for a, b in edges:
            group_a = None
            group_b = None

            for group in node_groups:
                if a in group:
                    group_a = group
                if b in group:
                    group_b = group

            if group_a != group_b:
                group_a.update(group_b)
                node_groups.remove(group_b)

        return len(node_groups)
```
`AC`  
ノードの数を`V`, エッジ情報の個数を`E`とおくと、  
時間計算量：  
 - `a`, `b`の所属グループの検索に`O(V)`
 - `update`による結合、`remove`による削除は要素数に比例なので`O(V)`
 - これをエッジ分おこなうから合計`O(V*E)`

空間計算量：
 - 保持するデータは、ノード部分のみであり、所属グループが重複することはないから、`O(V)`
  
今回の制約が  
`1 <= n <= 2000`, `1 <= edges.length <= 5000`  なので、`V * E = 10^7`  

ちょっと時間かけすぎてしまったかな...  
25分くらいかかってしまった  

## Step.1
他の方々のPRを見て修正していく  

自分のコードで気になっていたのが、このままだと接続(到達可能)しているグループが分かるが、連結関係が分からないということ  
このままだとグラフがどのような形になっているのかが全然わからない    
実際にグラフを作ったうえで答えを出力するのがベストなのだろう  

よく見かけたのが、`Union-Find`と呼ばれるメソッドを使用した手法と、DFS,BFSを使用した手法  

`Union-Find`とは？  
...要素を素集合(要素の重複がない集合)に分割するデータ構造で用いられるアルゴリズム  
 - `Union`: 2つの集合を1つに併合する
 - `Find`: ある要素がどの集合に属しているかを判定する  
    2つの要素が同じ集合に属しているかの判定も可能

`unionfind`クラスに`union`関数と`find`関数を実装することで解いたバージョン  
(初めて知ったのでほぼ写経)  

はじめはノードが`n`個のグループに分かれている(つながっていない)として、  
つながりがあったらグループ数を減らしていく  

### 書いたコード2  
```python
class Solution:
    def countComponents(self, n: int, edges: list[list[int]]) -> int:
        parent = list(range(n))
        group_num = n

        def find(i):
            if parent[i] == i:
                return i
            parent[i] = find(parent[i])
            return parent[i]

        def union(i, j):
            root_i = find(i)
            root_j = find(j)
            if root_i != root_j:
                parent[root_i] = root_j
                group_num -= 1

        for a, b in edges:
            union(a, b)
            
        return group_num
```
時間計算量がほぼ`O(E)`で済むようになった  

また、DFS, BFSで実際にグラフを構築していけば、グループ数だけでなく、各ノード間の距離なども分かり応用が効きそう  
訪問済みかを確認するリスト`visited`と、ノードの隣接関係を記録するリスト`adjacent`を用意する  

DFSの場合  
### 書いたコード3
```python
class Solution:
    def countComponents(self, n: int, edges: list[list[int]]) -> int:
        adjacent = [[] for _ in range(n)]
        for a, b in edges:
            adjacent[a].append(b)
            adjacent[b].append(a)
        
        visited = [False] * n

        def dfs(node):
            visited[node] = True
            for neighbor in adjacent[node]:
                if not visited[neighbor]:
                    dfs(neighbor)

        group_num = 0

        for i in range(n):
            if not visited[i]:
                group_num += 1
                dfs(i)
        
        return group_num
```

BFSの場合  

### 書いたコード4
```python
from collections import deque

class Solution:
    def countComponents(self, n: int, edges: list[list[int]]) -> int:
        adjacent = [[] for _ in range(n)]
        for a, b in edges:
            adjacent[a].append(b)
            adjacent[b].append(a)
        
        visited = [False] * n
        group_num = 0

        for i in range(n):
            if not visited[i]:
                group_num += 1
                queue = deque([i])
                visited[i] = True
                while queue:
                    node = queue.popleft()
                    for neighbor in adj[node]:
                        if not visited[neighbor]:
                            visited[neighbor] = True
                            queue.append(neighbor)
        
        return group_num
```
ネストが深くなってしまった  
見た目はDFSの方が好み  

時間計算量：`O(V + E)`頂点をもとに訪問済みかを`V`回おこなうのと、  
            エッジ情報をもとに隣接箇所を両端見るので、`2*E`回おこなう、合計で`O(V + E)`

## Step.2
書いたコードを、見ずにミスなく3回書く
