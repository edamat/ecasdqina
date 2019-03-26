---
layout: lib
title: AlienDP (Parametric Binary Search)
permalink: dynamic-programming/speedup/AlienDP

---


中国では WQS Binary Search と呼ばれている

AlienDPという名前は [IOI 2016 Aliens](https://www.ioi-jp.org/ioi/2016/tasks/day2-aliens-ISC.pdf){:target="_blank"}<!--_--> という問題から来ている

---

AlienDPの本質はDAG上の**2頂点対(単一パス)k辺最短経路問題**である

# 執筆中

解説書きます，しばしお待ちを！

# 実装


```cpp
// single pair k'-edge shortest path on DAG G
// k' can move in [k - droppable, k]
// from s to t; s, t is State
// dist(v, k) := k-edge shortest path from v to t
// dist(t, 0) = 0
// dist(s, k') is answer when k' is in [k - droppable, k]
// for every fixed v, dist(v, k) must be downward convex function regarding k as input
// (call this condition as Convexity)
//
// when G is perfect DAG,
// that the cost function w satisfies Convex QI is sufficient for Convexity
//
// O(T(Nf + r)) : T is times, N is number of nodes,
// calc, reset can be done in O(f), O(r), resp.
//
// hi is sufficient if it is the max of w

// GeneralPenalizeSpeedup<State>
// (to, from, k, droppable, reset, calc, lo, hi, times, comp?)
// {{"{{"}}{
#include <cassert>
#include <functional>
#include <map>
#include <tuple>
template < class State, class Float = double, class ResetF, class CalcF,
           class Compare = less< Float > >
Float GeneralPenalizeSpeedup(const State &to, const State &from, int k, int droppable,
                             ResetF reset, CalcF calc, Float lo, Float hi, int times,
                             Compare comp = Compare()) {
  assert(droppable >= 0);
  if(from == to) return Float(0);
  assert(k > 0);
  Float C;
  map< State, Float > memo;
  map< State, int > links;

  function< Float(State) > D = [&](State s) {
    if(memo.count(s)) return memo[s];
    Float val;
    State before;
    tie(val, before) = calc(D, s);
    links[s] = links[before] + 1;
    return memo[s] = val + C;
  };

  const auto solve = [&](const Float &_C) {
    C = _C;
    memo.clear();
    links.clear();

    memo[to] = Float(0);
    links[to] = 0;

    reset();

    D(from);

    // NOTE : acutualy, all nodes are added
    if(comp(C, 0)) {
      memo[from] += C * droppable;
      links[from] += droppable;
    }
  };

  if(comp(hi, lo)) swap(lo, hi); // for maximize

  // binary search
  for(int t = 0; t < times; t++) {
    Float mid = (lo + hi) / 2;
    solve(mid);
    if(links[from] <= k)
      hi = mid;
    else
      lo = mid;
  }

  // assume that we used just K edges
  solve(lo);

  return memo[from] - hi * k;
}
// }}}
// GeneralPenalizeSpeedup<State>(to, from, k, droppable, reset, calc, lo, hi, comp?) {{"{{"}}{
template < class State, class Float = double, class ResetF, class CalcF,
           class Compare = less< Float > >
Float GeneralPenalizeSpeedup(const State &to, const State &from, int k, int droppable,
                             const ResetF &reset, const CalcF &calc, Float lo, Float hi,
                             Compare comp = Compare()) {
  return GeneralPenalizeSpeedup< State, Float, ResetF, CalcF, Compare >(
      to, from, k, droppable, reset, calc, lo, hi, ceil(log2(abs(hi - lo) * 2 * k)) + 1,
      comp);
}
// }}}
// GeneralPenalizeSpeedup<State>(to, from, k, droppable, reset, calc, hi, comp?) {{"{{"}}{
template < class State, class Float = double, class ResetF, class CalcF,
           class Compare = less< Float > >
Float GeneralPenalizeSpeedup(const State &to, const State &from, int k, int droppable,
                             const ResetF &reset, const CalcF &calc, Float hi,
                             Compare comp = Compare()) {
  return GeneralPenalizeSpeedup< State, Float, ResetF, CalcF, Compare >(
      to, from, k, droppable, reset, calc, -hi, hi, comp);
}
// }}}

// when the dp is the folloing form
// d(i, k) = min(j < i, d(j, k - 1) + w[j + 1, i])
// and w is Convex QI

// NOTE : 1-indexed
// PenalizeSpeedup(int n, k, droppable, reset, calc, lo, hi, times, comp?) {{"{{"}}{
#include <cassert>
#include <functional>
#include <map>
#include <tuple>
#include <vector>
template < class Float = double, class ResetF, class CalcF,
           class Compare = less< Float > >
Float PenalizeSpeedup(int n, int k, int droppable, const ResetF &reset, const CalcF &calc,
                      Float lo, Float hi, int times, Compare comp = Compare()) {
  assert(droppable >= 0);
  if(n == 0) return 0;
  assert(k > 0 && n > 0);
  using State = int;
  const int from = n, to = 0;
  vector< Float > memo(n + 1);
  vector< int > links(n + 1);

  int now;
  function< Float(State) > D = [&](State s) {
    assert(0 <= s && s < now);
    return memo[s];
  };

  const auto solve = [&](const Float &C) {
    memo.assign(n + 1, Float(0));
    links.assign(n + 1, 0);

    memo[to] = Float(0);
    links[to] = 0;

    reset();

    for(now = 1; now <= n; now++) {
      Float val;
      State before;
      tie(val, before) = calc(D, now);
      assert(0 <= before && before < now);
      memo[now] = val + C;
      links[now] = links[before] + 1;
    }

    // NOTE : acutualy, all nodes (except "to") are added
    if(comp(C, 0)) {
      memo[from] += C * droppable;
      links[from] += droppable;
    }
  };

  if(comp(hi, lo)) swap(lo, hi); // for maximize

  // binary serach
  for(int t = 0; t < times; t++) {
    Float mid = (lo + hi) / 2;
    solve(mid);
    if(links[from] <= k)
      hi = mid;
    else
      lo = mid;
  }

  // assume that we used just K edges
  hi = (hi + lo) / 2;
  solve(hi);

  return memo[from] - hi * k;
}
// }}}
// PenalizeSpeedup(int n, k, droppable, reset, calc, lo, hi, comp?) {{"{{"}}{
template < class Float = double, class ResetF, class CalcF,
           class Compare = less< Float > >
Float PenalizeSpeedup(int n, int k, int droppable, const ResetF &reset, const CalcF &calc,
                      Float lo, Float hi, Compare comp = Compare()) {
  return PenalizeSpeedup< Float, ResetF, CalcF, Compare >(
      n, k, droppable, reset, calc, lo, hi, ceil(log2(abs(hi - lo) * k)) + 1, comp);
}
// }}}
// PenalizeSpeedup(int n, k, droppable, reset, calc, hi, comp?) {{"{{"}}{
template < class Float = double, class ResetF, class CalcF,
           class Compare = less< Float > >
Float PenalizeSpeedup(int n, int k, int droppable, const ResetF &reset, const CalcF &calc,
                      Float hi, Compare comp = Compare()) {
  return PenalizeSpeedup< Float, ResetF, CalcF, Compare >(
      n, k, droppable, reset, calc, -hi, hi, comp);
}
// }}}

// reset : () => void
// calc : (D : InpuType, state) => (value, before)

template < class State = int, class Float = double >
using InputType = function< Float(State) >;

using State = int;
using Float = double;
void reset() {}
// DO USE D function , DO NOT determine by yourself
// to be careful about TYPE (particular, use Float)
pair< Float, State > calc(const InputType< State, Float > &D, State i) {}
```


# 検証

* IOI2016 day2 Aliens
  * [PDF: 公式 statement](https://www.ioi-jp.org/ioi/2016/tasks/day2-aliens-ISC.pdf){:target="_blank"}<!--_-->
  * [PEG Online Judge (statement)](https://wcipeg.com/problem/ioi1623){:target="_blank"}<!--_-->
  * [PEG Online Judge (submission)](https://wcipeg.com/submissions/src/613303){:target="_blank"}<!--_--> ([mirror](https://gist.github.com/LumaKernel/e8c771f3ccc1c865112a157835915284#file-ioi2016-aliens-cpp){:target="_blank"}<!--_-->)

* ルーマニアIOI選抜2017 2 - Popcorn
  * [CSAcademy (statement)](https://csacademy.com/contest/romanian-ioi-2017-selection-2/task/popcorn/statement/){:target="_blank"}<!--_-->
  * [CSAcademy (code)](https://csacademy.com/code/J8qmEbff/){:target="_blank"}<!--_--> ([mirror](https://gist.github.com/LumaKernel/e8c771f3ccc1c865112a157835915284#file-romanian-ioi-2017-selection-2-popcorn-cpp){:target="_blank"}<!--_-->)

# 練習問題

* [#56 Or Problem - CSAcademy](https://csacademy.com/contest/round-56/task/or-problem/){:target="_blank"}<!--_-->
* [#351 C - Levels and Regions - codeforces](https://codeforces.com/contest/674/problem/C){:target="_blank"}<!--_-->
* [#381 div1 E - Gosha is hunting - codeforces](https://codeforces.com/contest/739/problem/E){:target="_blank"}<!--_-->
  * 下の参考にも書いたが，この問題がAlienDPを適用できる条件を満たしていることの証明がない

# 参考

* [Round #56 (Div. 1 + Div. 2) with prizes - codeforces](https://codeforces.com/blog/entry/55638){:target="_blank"}<!--_-->
* [Computing a Minimum Weight k-Link Path in Graphs with the Concave Monge Property](http://www.cs.ust.hk/mjg_lib/bibs/DPSu/DPSu.Files/sdarticle_204.pdf){:target="_blank"}<!--_-->
  * 完全DAG上での話だが，内容はAlienDPと捉えて問題ない
* [Incredibly beautiful DP optimization from N^3 to N log^2 N - codeforces](https://codeforces.com/blog/entry/49691){:target="_blank"}<!--_-->
  * やっていることはAlienDPだが，誰も $f(b) \coloneqq dp[i][a][b]$ が上に凸である事を証明できていない
  * 証明については，直感的に，得する順にソートするといったことを考えると良さそうだが，使えるbが増えたときに，他の組合せを変えて得することがあり，それも含めた証明は，そんなに自明なことではない
  * (コメントを見る限り) 誰も証明できていないが，おそらく上に凸だと思われる
* [競技プログラミングにおける動的計画法更新最適化まとめ - はまやんはまやんはまやん](https://www.hamayanhamayan.com/entry/2017/03/20/234711){:target="_blank"}<!--_-->
