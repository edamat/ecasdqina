---
layout: lib
title: 通常のCHT
permalink: dynamic-programming/convex-hull-trick/CHT

---


直線の集合 $L = \\{f_i(x) = a_i x + b_i\\}$ に対し，以下のクエリを高速に行えます
* add(a, b) : 直線 $f(x) = ax + b$ を追加するクエリ
* query(x) : $x$ に対して，最小値を取るような直線に対するその最小値を返すクエリ
  * $\displaystyle\min\_{f\in L}{f(x)}$

DPにも使える (ページ下部の参考も参照してください)

直線の数を$N$，クエリの数を$Q$ とする

# 実装

使い方はコードの下．オンラインクエリ可能


```cpp
// CHT<T, x-increasing?, Comp>
// - maximize : let Comp = greater<T>
// .add(a, b) : f(x) = ax + b
// - minimize : (a) desc
// - maximize : (a) asc
/// --- Convex Hull Trick Library {{"{{"}}{ ///

#include <cassert>
#include <functional>
#include <utility>
#include <vector>

template < class T = long long, bool xIncreasing = false, class Comp = less< T > >
struct CHT {
  static T EPS;
  static Comp comp;

private:
  using Line = pair< T, T >;

public:
  vector< Line > lines;
  // is l2 unnecessary ?
  bool check(Line l1, Line l2, Line l3) {
    if(l2.first == l3.first) return 1;
    // cp(l2, l3).x <= cp(l2, l1).x
    return (l2.first - l1.first) * (l3.second - l2.second) + EPS >=
           (l3.first - l2.first) * (l2.second - l1.second);
  }
  T f(int i, const T &x) { return lines[i].first * x + lines[i].second; }
  void add(const T &a, const T &b) {
    assert("add monotonic" && (lines.empty() || !comp(lines.back().first, a)));
    if(lines.size() && lines.back().first == a && !comp(b, lines.back().second)) return;
    while((int) lines.size() >= 2 &&
          check(lines[lines.size() - 2], lines.back(), Line(a, b)))
      lines.pop_back();
    lines.emplace_back(a, b);
  }
  T query(const T &x) {
    Line p = get(x);
    return p.first * x + p.second;
  }
  pair< T, T > get(const T &x) {
    assert(lines.size());
    if(xIncreasing) {
      static size_t head = 0;
      if(head >= lines.size()) head = lines.size() - 1;
      while(head + 1 < lines.size() && comp(f(head + 1, x), f(head, x))) head++;
      return lines[head];
    } else {
      int ok = lines.size() - 1, ng = -1;
      while(ok - ng > 1) {
        int mid = (ok + ng) >> 1;
        if(comp(f(mid, x), f(mid + 1, x)))
          ok = mid;
        else
          ng = mid;
      }
      return lines[ok];
    }
  }
};

template < class T, bool xIncreasing, class Comp >
T CHT< T, xIncreasing, Comp >::EPS = 1e-19;

template < class T, bool xIncreasing, class Comp >
Comp CHT< T, xIncreasing, Comp >::comp;

/// }}}--- ///
```


* `CHT<T, bool xIncreasing, Comp>`
  * `xIncreasing` : クエリ (get, query) の `x` が広義単調増加かどうか
  * デフォルトで最小値．`Comp = greater<T>` とすれば最大値
* `add(a, b)` : ならし $O(1)$
  * `f(x) = ax + b` を追加する
  * 最小化 : `a` の広義降順，`b` は自由
  * 最大化 : `a` の広義昇順，`b` は自由
  * 違反すると `assert` に引っかかる
  * この順序を任意の順で行うこともできる ([追加順序任意のCHT]({{ "dynamic-programming/convex-hull-trick/CHT-Ex" | absolute_url }}))
* `get(x) : (a, b)`
  * 単調 (x-increasing) なら 全体 $O(N + Q)$．そうでないなら 1回 $O(\log N)$
  * `x` を代入したときの値が最小 (最大) となる直線を `(a, b)` の形で返す
* `query(x)` : getと同様の計算量
  * `x` での最小 (最大) 値

# 検証

* [C - スペースエクスプローラー高橋君 - AtCoder](https://beta.atcoder.jp/contests/colopl2018-final-open/submissions/2171456){:target="_blank"}<!--_-->
  * このときの実装は少々バグがあります
* [D - Computer Game - codeforces](https://codeforces.com/contest/1067/submission/45446448){:target="_blank"}<!--_-->

# メモ

直線が不必要な条件

$\displaystyle-\frac{b_3-b_2}{a_3-a_2} \leq -\frac{b_2-b_1}{a_2-a_1}$

について，最小化なら $a_3 \gt a_2 \gt a_1$ という条件が成り立っているので，
最小・最大化どちらでも $(a_3-a_2)(a_2-a_1) \gt 0$ となり，

$(b_3-b_2)(a_2-a_1) \geq (b_2-b_1)(a_3-a_2)$ へ一意に変形できる

# EPSについて

`(aの正の最小値)(bの正の最小値)` より小さな正の値を採用するといい

たとえばデフォルトでは `1e-19` になっているが，これは両方が小数9桁ずつを想定している

# 参考

* [Convex-Hull Trick - SATANIC++](http://satanic0258.hatenablog.com/entry/2016/08/16/181331){:target="_blank"}<!--_-->
* [Dynamic Programming Optimizations - codeforcesの記事](https://codeforces.com/blog/entry/8219){:target="_blank"}<!--_-->
