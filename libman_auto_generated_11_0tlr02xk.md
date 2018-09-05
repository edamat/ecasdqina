---
layout: lib
title: Union Find (DSU)
permalink: data-structure/misc/UnionFind

---


Union Find; UF  
Disjoint Set Union; DSU

kruskal法や矛盾検知，他いろいろつかえる便利ですごい．

アッカーマン関数の逆関数はこの宇宙に書き下せるどのような数字(おそらく十進数を想定)をぶち込んでも4以下を返す関数なので，クエリの係数は4程度．(参考は英語wiki)


```cpp
/// --- Union Find Library {{"{{"}}{ ///

struct UF {
  int n;
  vector< int > par, rank;
  UF(int n) : n(n), par(n, -1), rank(n, 0) {}
  int find(int x) { return par[x] < 0 ? x : par[x] = find(par[x]); }
  int size(int x) { return -par[find(x)]; }
  bool same(int a, int b) { return find(a) == find(b); }
  void unite(int a, int b) {
    a = find(a);
    b = find(b);
    if(a == b) return;
    if(rank[a] > rank[b]) swap(a, b);
    par[b] += par[a];
    par[a] = b;
    if(rank[a] == rank[b]) rank[b]++;
  }
};

/// }}}--- ///
```


# 検証

* [B - Union Find - AC](https://beta.atcoder.jp/contests/atc001/submissions/2147616){:target="_blank"}<!--_-->
