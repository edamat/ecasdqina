---
layout: lib
title: Modulo Integer
permalink: misc/ModuloInteger

---

最強時短エンバグ回避ライブラリ


```cpp
/// --- ModInt Library {{"{{"}}{ ///
template < ll mod = (ll) 1e9 + 7 >
struct ModInt {
  ll extgcd(ll a, ll b, ll &x, ll &y) {
    ll d;
    return b == 0 ? (x = 1, y = 0, a)
                  : (d = extgcd(b, a % b, y, x), y -= a / b * x, d);
  }
  ll modinv(ll a) {
    ll x = 0, y = 0;
    extgcd(a, mod, x, y);
    return (x + mod) % mod;
  }
  ll val;
  ModInt() : val(0) {}
  ModInt(ll val) : val((val % mod + mod) % mod) {}
  ModInt operator+(ModInt const &rhs) const { return ModInt(val + rhs.val); }
  ModInt operator-(ModInt const &rhs) const { return ModInt(val - rhs.val); }
  ModInt operator*(ModInt const &rhs) const { return ModInt(val * rhs.val); }
  ModInt operator/(ModInt const &rhs) const {
    return ModInt(val * rhs.inv().val);
  }
  ModInt &operator+=(ModInt const &rhs) {
    val = ((val + rhs.val) % mod + mod) % mod;
    return *this;
  }
  ModInt &operator-=(ModInt const &rhs) {
    val = ((val - rhs.val) % mod + mod) % mod;
    return *this;
  }
  ModInt &operator*=(ModInt const &rhs) {
    val = (val * rhs.val % mod + mod) % mod;
    return *this;
  }
  ModInt &operator/=(ModInt const &rhs) {
    val = (val * rhs.inv().val % mod + mod) % mod;
    return *this;
  }
  ModInt operator++(int) {
    ModInt tmp = *this;
    val = val + 1;
    if(val >= mod) val = 0;
    return tmp;
  }
  ModInt operator--(int) {
    ModInt tmp = *this;
    val = val == 0 ? mod - 1 : val - 1;
    return tmp;
  }
  ModInt &operator++() {
    val = val + 1;
    if(val >= mod) val = 0;
    return *this;
  }
  ModInt &operator--() {
    val = val == 0 ? mod - 1 : val - 1;
    return *this;
  }
  template < typename T >
  ModInt operator+(T const &rhs) const {
    return ModInt(val + rhs % mod);
  }
  template < typename T >
  ModInt operator-(T const &rhs) const {
    return ModInt(val - rhs % mod);
  }
  template < typename T >
  ModInt operator*(T const &rhs) const {
    return ModInt(val * (rhs % mod));
  }
  template < typename T >
  ModInt operator/(T const &rhs) const {
    return ModInt(val * modinv(rhs));
  }
  template < typename T >
  ModInt &operator+=(T const &rhs) {
    val = ((val + rhs % mod) % mod + mod) % mod;
    return *this;
  }
  template < typename T >
  ModInt &operator-=(T const &rhs) {
    val = ((val - rhs % mod) % mod + mod) % mod;
    return *this;
  }
  template < typename T >
  ModInt &operator*=(T const &rhs) {
    val = (val * (rhs % mod) % mod + mod) % mod;
    return *this;
  }
  template < typename T >
  ModInt &operator/=(T const &rhs) {
    val = (val * modinv(rhs, mod) % mod + mod) % mod;
    return *this;
  }
  ModInt inv() const { return ModInt(modinv(val, mod)); }
  friend ostream &operator<<(ostream &os, ModInt const &mv) {
    os << mv.val;
    return os;
  }
  friend constexpr ModInt operator+(ll a, ModInt const &mv) {
    return ModInt(a % mod + mv.val);
  }
  friend constexpr ModInt operator-(ll a, ModInt const &mv) {
    return ModInt(a % mod - mv.val);
  }
  friend constexpr ModInt operator*(ll a, ModInt const &mv) {
    return ModInt(a % mod * mv.val);
  }
  friend constexpr ModInt operator/(ll a, ModInt const &mv) {
    return ModInt(a % mod * mv.inv().val);
  }
};
/// }}}--- ///

using Int = ModInt<>;
```
