---
layout: lib
title: 普通のセグメントツリー
permalink: data-structure/SegmentTree/SegmentTree

---



```cpp
// NOTE : query in range!
/// --- SegmentTree {{"{{"}}{ ///

#include <iostream>
#include <vector>

template < class Monoid >
struct SegmentTree {
private:
  using T = typename Monoid::T;
  int n;
  vector< T > data;
  // call after touch data[i]
  void prop(int i) { data[i] = Monoid::op(data[2 * i], data[2 * i + 1]); }

public:
  SegmentTree() : n(0) {}
  SegmentTree(int n, T initial = Monoid::identity()) : n(n) {
    data.resize(n * 2, initial);
    for(int i = n - 1; i > 0; i--) data[i] = Monoid::op(data[i * 2], data[i * 2 + 1]);
  }
  template < class InputIter, class = typename iterator_traits< InputIter >::value_type >
  SegmentTree(InputIter first, InputIter last) : SegmentTree(distance(first, last)) {
    copy(first, last, begin(data) + n);
    // fill from deep
    for(int i = n - 1; i > 0; i--) prop(i);
  }
  void set(int i, const T &v) {
    data[i += n] = v;
    while(i >>= 1) prop(i); // propUp
  }
  T get(int i) { return data[i + n]; }
  T query(int l, int r) {
    T tmpL = Monoid::identity(), tmpR = Monoid::identity();
    for(l += n, r += n; l < r; l >>= 1, r >>= 1) {
      if(l & 1) tmpL = Monoid::op(tmpL, data[l++]);
      if(r & 1) tmpR = Monoid::op(data[--r], tmpR);
    }
    return Monoid::op(tmpL, tmpR);
  }
  inline void dum(int r = -1) {
#ifdef DEBUG
    if(r < 0) r = n;
    DEBUG_OUT << "{";
    for(int i = 0; i < min(r, n); i++) DEBUG_OUT << (i ? ", " : "") << get(i);
    DEBUG_OUT << "}" << endl;
#endif
  }
};

/// }}}--- ///

/// --- Monoid examples {{"{{"}}{ ///

#include <algorithm>

constexpr long long inf = 1e18 + 100;

struct Nothing {
  using T = char;
  using M = T;
  static constexpr T op(const T &, const T &) { return T(); }
  static constexpr T identity() { return T(); }
  template < class X >
  static constexpr X actInto(const M &, ll, ll, const X &x) {
    return x;
  }
};

struct RangeMin {
  using T = ll;
  static T op(const T &a, const T &b) { return min(a, b); }
  static constexpr T identity() { return inf; }
};

struct RangeMax {
  using T = ll;
  static T op(const T &a, const T &b) { return max(a, b); }
  static constexpr T identity() { return -inf; }
};

struct RangeSum {
  using T = ll;
  static T op(const T &a, const T &b) { return a + b; }
  static constexpr T identity() { return 0; }
};

/// }}}--- ///

using Seg = SegmentTree< RangeMin >;
Seg seg(N);
```


# 関連

[Li-Chao Tree]({{ "dynamic-programming/convex-hull-trick/LiChaoTree" | absolute_url }}) はセグメントツリーを利用したCHTです

# 参考

* [データ構造と代数(前編)](https://tomcatowl.github.io/post/ds-and-alg-1/){:target="_blank"}