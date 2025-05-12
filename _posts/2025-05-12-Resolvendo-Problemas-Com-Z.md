---
layout: post
title: "Resolvendo problemas com Z"
date: 2025-05-12
categories: [algoritmos, c++, programação]
---

Continuando nossa sequência de post sobre algoritmos de string. Achei um problema interessante: 

[Codeforces: B. Password](https://codeforces.com/problemset/problem/126/B).

Descobri esse problema enquanto estudava sobre *Z*, nesse post no codeforces: [Z Algorithm](https://codeforces.com/blog/entry/3107). (Caso queira ver na integra sobre a solução).

De maneira direta, temos que o problema é o seguinte: Dado um string s, queremos encontrar o maior sufixo que também é prefixo, mas que também, de maneira disjunta tambem aparece no meio da string s.

Podemos entender isso da seguinte forma, `s` tem solução quando : `s = t + ... + t + ... + t`, onde t representa uma a substring de s. Sabendo que t existe, precisamos do maior t que podemos encontrar que satisfaz essas condições.

## Solução

Primeira ideia para esse problema é enxergar que Z que pela natureza de Z, parcialmente já temos uma resposta para o subproblema: achar um sufixo que também é prefixo.

Para isso, basta que `z[i] + i == s.size()`.

Até agora temos esse código:

```cpp
vector<int> z_function(string s) {
    int n = s.size();
    vector<int> z(n);
    int l = 0, r = 0;
    for(int i = 1; i < n; i++) {
        if(i < r) {
            z[i] = min(r - i, z[i - l]);
        }
        while(i + z[i] < n && s[z[i]] == s[i + z[i]]) {
            z[i]++;
        }
        if(i + z[i] > r) {
            l = i;
            r = i + z[i];
        }
    }
    return z;
}
signed main() {
    string s; cin >> s;
    vector<int> z = z_function(s);
    int n = s.size();
    for(int i = 1; i < n; i++){

        if(z[i] == n - i){
          //achei o sufixo que é prefixo.
        }
    }
    return 0;
}
```

Agora precisamos verificar se quando eu tenho um prefixo que também é sufixo ele aparece no meio da minha string.

E para isso, enxerguei que:

`Seja s = fixprefixsuffix` e
`z(s) = [0, 0, 0, 0, 0, 0, 3, 0, 0, 0, 0, 1, 3, 0, 0]`

Resposta: `fix`, pois `Seja s = [fix]pre[fix]suf[fix]`

De maneira intuitiva percemos que `Z[12] = 3` é solução para nossa primeira condição e pelo numero de ocorrências (`ocorrencias de (Z[12]) > 1`).

Primeira ideia que tive foi contar as ocorrências de `Z[i]` e sendo maior que 1, indica que encontrei ela antes de chegar no final.

```cpp

    ....
    vector<int> ocorrencias(n, 0);
    for(int i = 1; i < n; i++){
        ocorrencias[z[i]]++;
        if(z[i] == n - i and ocorrencias[z[i]] > 1){
          //achei um candidato a solução do problema
        }
    }
    ...
```

Contudo, encontrei alguns problemas. Para essa string de exemplo, funcionou muito bem. Mas para o caso:

`s = 'aaaaaa'` e
`z = [0, 5, 4, 3, 2, 1]`

O número de ocorrências de qualquer `Z[i]` é exatamente 1, e falha nossa primeira tentativa de solução.

Mas, tendo esse caso em mente, conseguimos visualizar que:

- Se `z[i] + i == s.size()`.
- E, as ocorrências de  `qualquer Z[x], com x, estando entre 1 até i - 1 maiores que Z[i]`, também podem ser solução.

E isso é a chave do problema. E funciona pois se `Z[x] >= Z[i]`, sabemos que a substring formada por `s(x,x+ Z[x]) contem s(i,i + Z[i])`.

Para checarmos isso de maneira inteligente, podemos fazer como é exposto no blog aqui : [Z Algorithm](https://codeforces.com/blog/entry/3107). Basta para cada iteração i, checarmos o maior Z[i] que encontramos até o momento.

O código ficaria assim:

```cpp

    ....
    int maxz = 0;
    for(int i = 1; i < n; i++){
        ocorrencias[z[i]]++;
        if(z[i] == n - i and maxz >= z[i]){
          //achei a solução do problema
          break; // o break é importante, pois como ele quer o maior, se achamos uma solução em i, todas as possiveis solução de i+1 até n, serão inferiores a i.
        }
    }
    ...
```

E finalizando, temos a solução final:

```cpp
#include <bits/stdc++.h>
#include <ext/pb_ds/assoc_container.hpp>
#include <ext/pb_ds/tree_policy.hpp>
using namespace __gnu_pbds;

using namespace std;

#define _ ios_base::sync_with_stdio(0); cin.tie(0);
#define endl '\n'
#define int long long

#ifdef DEBUG
    #define dbg(x) cerr << #x << " = "; _print(x); cerr << endl;
#else
    #define dbg(x)
#endif

void _print(long long x) { cerr << x; }
void _print(string x) { cerr << '"' << x << '"'; }
void _print(char x) { cerr << '\'' << x << '\''; }
void _print(bool x) { cerr << (x ? "true" : "false"); }
void _print(double x) { cerr << x; }

template<typename T, typename V> void _print(pair<T, V> p) { cerr << "{"; _print(p.first); cerr << ", "; _print(p.second); cerr << "}"; }
template<typename T> void _print(vector<T> v) { cerr << "["; for (auto &i : v) { _print(i); cerr << ", "; } cerr << "]"; }
template<typename T> void _print(set<T> s) { cerr << "{"; for (auto &i : s) { _print(i); cerr << ", "; } cerr << "}"; }
template<typename T, typename V> void _print(map<T, V> m) { cerr << "{"; for (auto &i : m) { _print(i); cerr << ", "; } cerr << "}"; }


// Função para calcular o array Z
vector<int> z_function(string s) {
    int n = s.size();
    vector<int> z(n);
    int l = 0, r = 0;
    for(int i = 1; i < n; i++) {
        if(i < r) {
            z[i] = min(r - i, z[i - l]);
        }
        while(i + z[i] < n && s[z[i]] == s[i + z[i]]) {
            z[i]++;
        }
        if(i + z[i] > r) {
            l = i;
            r = i + z[i];
        }
    }
    return z;
}

signed main() {
    string s; cin >> s;
    vector<int> z = z_function(s);
    int n = s.size();
    int maxz = 0, ans = 0;
    for(int i = 1; i < s.size(); i++){

        if(z[i] == n - i and maxz >= n - i){
            ans = n - i;
            break;
        }
        maxz = max(maxz, z[i]);
    }
    if(ans == 0){
        cout << "Just a legend" << endl;
        return 0;
    }
    cout << s.substr(n - ans, n) << endl;
    return 0;
}
```
