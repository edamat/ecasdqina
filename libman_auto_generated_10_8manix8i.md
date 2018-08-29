---
layout: lib
title: Link/Cut Tree
permalink: data-structure/misc/LinkCutTree

---

森(もしくは根付き有向木(Arborescence)の集合)への更新とクエリを順不同で高速に．

根の変更クエリ(evert)には乗せるモノイドに対する要求があり，その条件は平衡二分探索木がreverseで要求するものと同じ([モノイド]()をみて)．

動的木の一種．

IOIで常勝できることはあまりにも有名．

参考になるものを準備しています…

TODO : 区間に対する更新に対し，leftを求めるの，ちょっとどうすればいいかわからないけど，  
やっている人がいるのでそのうち見てみようと思います．


```cpp
// when link(p, c) , c is root.
// use make(int index, Monoid::T x)
// lc[index] to access nodes
/// --- LinkCutTree Library {{"{{"}}{ ///

template < class Monoid, class M_act >
struct LinkCutTree {
  using X = typename Monoid::T;
  using M = typename M_act::M;

  // Splay sequence {{"{{"}}{
  struct Splay {
    Splay *ch[2] = {nullptr, nullptr}, *p = nullptr;
    X val, accum;
    M lazy = M_act::identity(); ///////
    // BSTの大きさ // 実際の部分木の大きさ，ではない
    int sz = 1;
    bool isRoot() { return !p || (p->ch[0] != this && p->ch[1] != this); }
    bool rev = false;
    // call before use
    void eval() {
      if(lazy != M_act::identity()) {
        val = M_act::actInto(lazy, -1, 1, val);
        accum = M_act::actInto(lazy, -1, sz, accum);
        if(ch[0]) ch[0]->lazy = M_act::op(lazy, ch[0]->lazy);
        if(ch[1]) ch[1]->lazy = M_act::op(lazy, ch[1]->lazy);
        lazy = M_act::identity();
      }
      if(rev) {
        swap(ch[0], ch[1]);
        if(ch[0]) ch[0]->rev ^= 1;
        if(ch[1]) ch[1]->rev ^= 1;
        // accum = reverse(accum, sz)
        rev = false;
      }
    }
    void evalDown() {
      vector< Splay * > b2t;
      Splay *t = this;
      for(; !t->isRoot(); t = t->p) b2t.emplace_back(t);
      t->eval();
      while(b2t.size()) b2t.back()->eval(), b2t.pop_back();
      // vector< Splay * >().swap(b2t);
    }
    X accumulated(Splay *a) { return !a ? Monoid::identity() : a->accum; }
    int size(Splay *a) { return !a ? 0 : a->sz; }
    // call after touch
    void prop() {
      if(ch[0]) ch[0]->eval(); ////
      if(ch[1]) ch[1]->eval(); ////
      sz = size(ch[0]) + 1 + size(ch[1]);
      accum =
          Monoid::op(Monoid::op(accumulated(ch[0]), val), accumulated(ch[1]));
    }
    Splay(const X &val) : val(val), accum(val) {}
    Splay *rotate(bool R) {
      Splay *t = ch[!R];
      if((ch[!R] = t->ch[R])) ch[!R]->p = this;
      t->ch[R] = this; ////
      if((t->p = p)) {
        if(t->p->ch[0] == this) t->p->ch[0] = t;
        if(t->p->ch[1] == this) t->p->ch[1] = t;
      }
      p = t;
      prop(), t->prop();
      return t;
    }
    // bottom-up
    void splay() {
      evalDown();
      while(!isRoot()) {
        Splay *q = p;
        if(q->isRoot()) {
          q->rotate(q->ch[0] == this);
        } else {
          Splay *r = q->p;
          bool V = r->ch[0] == q;
          if(q->ch[!V] == this)
            r->rotate(V);
          else
            q->rotate(!V);
          p->rotate(V); ///////
        }
      }
    }
  };
  // }}}

  vector< Splay * > data;
  LinkCutTree(int n) : data(n) {}
  Splay *operator[](int i) { return data[i]; }
  Splay *make(int i, const X &x = Monoid::identity()) {
    return data[i] = new Splay(x);
  }
  const X &get(Splay *x) {
    x->evalDown();
    return x->val;
  }
  void set(Splay *x, const X &val) {
    x->splay();
    x->val = val;
    x->prop();
  }
  Splay *expose(Splay *x) {
    Splay *prv = nullptr, *now = x;
    for(; now; prv = now, now = now->p) {
      now->splay();
      now->ch[1] = prv;
      now->prop();
    }
    x->splay(); /////
    return prv;
  }
  void cut(Splay *c) {
    expose(c);
#ifdef DEBUG
    static const struct CannotCutRoot {} ex;
    if(!c->ch[0]) throw ex;
#endif
    Splay *s = c->ch[0];
    c->ch[0] = nullptr;
    c->prop();
    s->p = nullptr;
  }
  void link(Splay *parent, Splay *child) {
#ifdef DEBUG
    static const struct CannotLinkSameNode {} ex;
    if(same(parent, child)) throw ex;
#endif
    expose(parent), expose(child);
    child->p = parent;
    parent->ch[1] = child;
    parent->prop();
  }
  void evert(Splay *x) {
    expose(x);
    x->rev = true;
  }
  bool same(Splay *a, Splay *b) {
    if(a == b) return true;
    expose(a), expose(b);
    return a->p != nullptr;
  }
  Splay *lca(Splay *a, Splay *b) {
#ifdef DEBUG
    static const struct CannotLCAAnotherNode {} ex;
    if(!same(a, b)) throw ex;
#endif
    expose(a), a = expose(b);
    return !a ? b : a;
  }
  void act(Splay *a, const M &m) { expose(a), a->lazy = m; }
  X query(Splay *a) {
    expose(a);
    return a->accum;
  }
};

/// }}}--- ///

/// --- Monoid, M_act examples {{"{{"}}{ ///

/// --- Monoid examples {{"{{"}}{ ///

struct Nothing {
  using T = char;
  using M = char;
  static constexpr T op(const T &, const T &) { return 0; }
  static constexpr T identity() { return 0; }
  template < class X >
  static constexpr X actInto(const M &, ll, ll, const X &x) {
    return x;
  }
};

struct RangeMin {
  using T = ll;
  static T op(const T &a, const T &b) { return min(a, b); }
  static constexpr T identity() { return numeric_limits< T >::max(); }
};

struct RangeMax {
  using T = ll;
  static T op(const T &a, const T &b) { return max(a, b); }
  static constexpr T identity() { return numeric_limits< T >::min(); }
};

struct RangeSum {
  using T = ll;
  static T op(const T &a, const T &b) { return a + b; }
  static constexpr T identity() { return 0; }
};

/// }}}--- ///

// MinAdd m + x
// MinSet m
// SumAdd m * n + x
// SumSet m * n

struct RangeMinAdd {
  using M = ll;
  using X = RangeMin::T;
  static M op(const M &a, const M &b) { return a + b; }
  static constexpr M identity() { return 0; }
  static X actInto(const M &m, ll, ll, const X &x) { return m + x; }
};

struct RangeMinSet {
  using M = ll;
  using X = RangeMin::T;
  static M op(const M &a, const M &) { return a; }
  static constexpr M identity() { return numeric_limits< M >::min(); }
  static X actInto(const M &m, ll, ll, const X &) { return m; }
};

struct RangeSumAdd {
  using M = ll;
  using X = RangeSum::T;
  static M op(const M &a, const M &b) { return a + b; }
  static constexpr M identity() { return 0; }
  static X actInto(const M &m, ll, ll n, const X &x) { return m * n + x; }
};

struct RangeSumSet {
  using M = ll;
  using X = RangeSum::T;
  static M op(const M &a, const M &) { return a; }
  static constexpr M identity() { return numeric_limits< M >::min(); }
  static X actInto(const M &m, ll, ll n, const X &) { return m * n; }
};

/// }}}--- ///

// LinkCutTree< RangeSum, RangeSumSet > lc(N);
```


# 検証

* RMQとRUQ - [AOJのなんか](https://onlinejudge.u-aizu.ac.jp/status/users/luma/submissions/1/DSL_2_F/judge/3092002/C++14){:target="_blank"}
  * 遅延セグ木でできることがevert付きでできる
* LCA - [AOJのなんか](https://onlinejudge.u-aizu.ac.jp/status/users/luma/submissions/1/GRL_5_C/judge/3092319/C++14){:target="_blank"}
* HL-Decomp(TLE) [E. The Number Games \| CF](https://codeforces.com/contest/980/submission/41594330){:target="_blank"}
* HL-Decomp [PCKの問題 \| AOJ]( https://onlinejudge.u-aizu.ac.jp/status/users/luma/submissions/1/0367/judge/3093506/C++14)
