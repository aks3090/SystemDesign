# Classic coding round: 15-minute brush-up (C#)

Every problem gets one line of key idea, then the code. Read the code, not the prose.

**Habits for every problem:** clarify (size, duplicates, empty input, ties), name the brute force and its cost, code the optimal, narrate the invariant, walk one example by hand, then raise the follow-up yourself.

---

### 716 Max Stack (24 reports): two sorted sets that share a push id, every op O(log n)
```csharp
SortedSet<(int id, int val)> byTime = new();   // Max = top of stack
SortedSet<(int val, int id)> byVal  = new();   // Max = largest val, most recent on ties
int nextId = 0;
public void Push(int x) { byTime.Add((nextId, x)); byVal.Add((x, nextId)); nextId++; }
public int Pop()     { var t = byTime.Max; byTime.Remove(t); byVal.Remove((t.val, t.id)); return t.val; }
public int Top()     => byTime.Max.val;
public int PeekMax() => byVal.Max.val;
public int PopMax()  { var m = byVal.Max; byVal.Remove(m); byTime.Remove((m.id, m.val)); return m.val; }
```
Mention the two-stack version first (popMax is O(n)).

### 364 Nested List Weight Sum II (19 reports): `(maxDepth+1)·Σv − Σ(v·depth)`, one DFS
```csharp
int maxDepth = 0, flat = 0, weighted = 0;
public int DepthSumInverse(IList<NestedInteger> list) { Dfs(list, 1); return (maxDepth + 1) * flat - weighted; }
void Dfs(IList<NestedInteger> list, int d) {
    foreach (var n in list)
        if (n.IsInteger()) { int v = n.GetInteger(); flat += v; weighted += v * d; maxDepth = Math.Max(maxDepth, d); }
        else Dfs(n.GetList(), d + 1);
}
```
The level-by-level BFS trick overcounts on `[1,[[]]]`. Point that out.

### 341 Flatten Nested List Iterator: a stack of enumerators, lazy
```csharp
Stack<IEnumerator<NestedInteger>> st = new(); int? nextVal;
public NestedIterator(IList<NestedInteger> l) => st.Push(l.GetEnumerator());
public bool HasNext() {
    while (nextVal == null && st.Count > 0) {
        var it = st.Peek();
        if (!it.MoveNext()) { st.Pop(); continue; }
        if (it.Current.IsInteger()) nextVal = it.Current.GetInteger();
        else st.Push(it.Current.GetList().GetEnumerator());
    }
    return nextVal != null;
}
public int Next() { HasNext(); int v = nextVal!.Value; nextVal = null; return v; }
```

### 380 Insert Delete GetRandom O(1): list + value-to-index map, swap with last on remove
```csharp
List<int> vals = new(); Dictionary<int, int> idx = new(); Random rng = new();
public bool Insert(int v) { if (idx.ContainsKey(v)) return false; idx[v] = vals.Count; vals.Add(v); return true; }
public bool Remove(int v) {
    if (!idx.TryGetValue(v, out int i)) return false;
    int last = vals[^1]; vals[i] = last; idx[last] = i;      // works even if v == last
    vals.RemoveAt(vals.Count - 1); idx.Remove(v); return true;
}
public int GetRandom() => vals[rng.Next(vals.Count)];
```
Follow-up 381 (duplicates allowed): `Dictionary<int, HashSet<int>>`.

### 146 LRU Cache: LinkedList + map from key to node
```csharp
Dictionary<int, LinkedListNode<(int k, int v)>> map = new(); LinkedList<(int k, int v)> list = new(); int cap;
public int Get(int k) {
    if (!map.TryGetValue(k, out var n)) return -1;
    list.Remove(n); list.AddFirst(n); return n.Value.v;
}
public void Put(int k, int v) {
    if (map.TryGetValue(k, out var n)) list.Remove(n);
    else if (map.Count == cap) { map.Remove(list.Last!.Value.k); list.RemoveLast(); }
    map[k] = list.AddFirst((k, v));
}
```

### 432 All O`one (10 reports): doubly linked list of count buckets + map from key to bucket
```csharp
class B { public int C; public HashSet<string> Keys = new(); public B Prev, Next; }
B head = new(), tail = new(); Dictionary<string, B> at = new();   // ctor: head.Next = tail; tail.Prev = head;
B After(B b, int c) { var n = new B { C = c, Prev = b, Next = b.Next }; b.Next.Prev = n; b.Next = n; return n; }
void Drop(B b) { if (b.Keys.Count == 0) { b.Prev.Next = b.Next; b.Next.Prev = b.Prev; } }

public void Inc(string k) {
    var cur = at.TryGetValue(k, out var b) ? b : head;
    var nx = cur.Next.C == cur.C + 1 && cur.Next != tail ? cur.Next : After(cur, cur.C + 1);
    nx.Keys.Add(k); at[k] = nx;
    if (cur != head) { cur.Keys.Remove(k); Drop(cur); }      // create the new bucket BEFORE dropping the old
}
public void Dec(string k) {
    if (!at.TryGetValue(k, out var cur)) return;
    cur.Keys.Remove(k);
    if (cur.C == 1) at.Remove(k);
    else { var pv = cur.Prev != head && cur.Prev.C == cur.C - 1 ? cur.Prev : After(cur.Prev, cur.C - 1); pv.Keys.Add(k); at[k] = pv; }
    Drop(cur);
}
public string GetMaxKey() => tail.Prev == head ? "" : tail.Prev.Keys.First();
public string GetMinKey() => head.Next == tail ? "" : head.Next.Keys.First();
```

### 1004 Max Consecutive Ones III: sliding window with at most k zeros
```csharp
int l = 0, zeros = 0, best = 0;
for (int r = 0; r < a.Length; r++) {
    if (a[r] == 0) zeros++;
    while (zeros > k) if (a[l++] == 0) zeros--;
    best = Math.Max(best, r - l + 1);
}
```

### 244 Shortest Word Distance II: precompute positions, two-pointer merge per query
```csharp
// ctor: pos[word].Add(i) for each index (the lists are already sorted)
public int Shortest(string a, string b) {
    var x = pos[a]; var y = pos[b]; int i = 0, j = 0, best = int.MaxValue;
    while (i < x.Count && j < y.Count) {
        best = Math.Min(best, Math.Abs(x[i] - y[j]));
        if (x[i] < y[j]) i++; else j++;
    }
    return best;
}
```
Follow-ups: cache `(a, b)` pairs. For 245 (same word allowed), compare adjacent positions in one list.

### 605 Can Place Flowers
```csharp
for (int i = 0; i < f.Length && n > 0; i++)
    if (f[i] == 0 && (i == 0 || f[i - 1] == 0) && (i == f.Length - 1 || f[i + 1] == 0)) { f[i] = 1; n--; }
return n <= 0;
```

---

## LinkedIn-custom problems (most on-theme)

### Connection Distance: bidirectional BFS, always expand the smaller frontier
```csharp
int Distance(Dictionary<int, List<int>> g, int s, int t) {
    if (s == t) return 0;
    HashSet<int> front = new() { s }, back = new() { t }, seenF = new() { s }, seenB = new() { t };
    int depth = 0;
    while (front.Count > 0 && back.Count > 0) {
        if (front.Count > back.Count) { (front, back) = (back, front); (seenF, seenB) = (seenB, seenF); }
        depth++;
        var next = new HashSet<int>();
        foreach (var u in front)
            foreach (var v in g.TryGetValue(u, out var nb) ? nb : new())
            {
                if (seenB.Contains(v)) return depth;
                if (seenF.Add(v)) next.Add(v);
            }
        front = next;
    }
    return -1;
}
```
Influencer nodes: expanding the smaller side handles them while staying exact (b^d becomes 2·b^(d/2)). Cap at 3 degrees, and precompute 2nd-degree sets offline.

### Mutual friends / 2nd degree ("People You May Know")
```csharp
var mutual = (a.Count < b.Count ? a : b).Where((a.Count < b.Count ? b : a).Contains);   // iterate the smaller set

var score = new Dictionary<int, int>();                    // 2nd degree, ranked by number of mutuals
foreach (var f in g[me]) foreach (var ff in g[f])
    if (ff != me && !g[me].Contains(ff)) score[ff] = score.GetValueOrDefault(ff) + 1;
```

### TaskList with prerequisites + cycle detection: DFS cycle check first, then incremental Kahn
```csharp
Dictionary<TaskItem, int> pending = new(); Dictionary<TaskItem, List<TaskItem>> deps = new();
HashSet<TaskItem> known = new(), done = new(); Queue<TaskItem> ready = new();

public void AddTask(TaskItem t) {
    if (Cycle(t, new(), new())) throw new InvalidOperationException("cycle");   // validate before mutating
    Register(t);
}
bool Cycle(TaskItem t, HashSet<TaskItem> path, HashSet<TaskItem> ok) {
    if (path.Contains(t)) return true; if (ok.Contains(t)) return false;
    path.Add(t); foreach (var s in t.SubTasks) if (Cycle(s, path, ok)) return true;
    path.Remove(t); ok.Add(t); return false;
}
void Register(TaskItem t) {
    if (!known.Add(t)) return;
    int w = 0;
    foreach (var s in t.SubTasks) {
        Register(s);
        if (done.Contains(s)) continue;
        w++; if (!deps.TryGetValue(s, out var l)) deps[s] = l = new(); l.Add(t);
    }
    pending[t] = w; if (w == 0) ready.Enqueue(t);
}
public TaskItem? PollNext() {
    if (ready.Count == 0) return null;
    var t = ready.Dequeue(); t.Run?.Invoke(); done.Add(t);
    if (deps.TryGetValue(t, out var ws)) foreach (var x in ws) if (--pending[x] == 0) ready.Enqueue(x);
    return t;
}
```
Ask first: does `pollNext` also mark the task complete? For parallel workers, split it into `Claim()` + `Complete(t)`.

### LZ78: trie dictionary keyed by `(nodeId, char)`
```csharp
var child = new Dictionary<(int, char), int>(); var outp = new List<(int, char?)>();
int node = 0, next = 1;
foreach (char c in s) {
    if (child.TryGetValue((node, c), out int nx)) { node = nx; continue; }
    outp.Add((node, c)); child[(node, c)] = next++; node = 0;
}
if (node != 0) outp.Add((node, null));                    // input ended inside a known phrase

// decompress
var phrases = new List<string> { "" }; var sb = new StringBuilder();
foreach (var (i, ch) in codes) { var p = ch is char c ? phrases[i] + c : phrases[i]; sb.Append(p); phrases.Add(p); }
```
`"ABBCBCABABCAABCAAB"` → `(0,A)(0,B)(2,C)(3,A)(2,A)(4,A)(6,B)`

### Restaurant election (instant runoff)
```csharp
var alive = ballots.SelectMany(b => b).ToHashSet();
while (alive.Count > 0) {
    var tally = alive.ToDictionary(c => c, _ => 0); int active = 0;   // include zero-vote candidates
    foreach (var b in ballots) { var top = b.FirstOrDefault(alive.Contains); if (top != null) { tally[top]++; active++; } }
    var lead = tally.MaxBy(kv => kv.Value);
    if (lead.Value * 2 > active || alive.Count == 1) return lead.Key;
    int min = tally.Values.Min();
    var low = tally.Where(kv => kv.Value == min).Select(kv => kv.Key).ToList();
    if (low.Count == alive.Count) return null;                        // full tie: ask how to break it
    alive.ExceptWith(low);
}
```
Faster version: keep a list of ballots per candidate and re-route only the eliminated candidate's ballots.

---

## Templates

### Union-find (200, 305, 1135)
```csharp
int[] p, r;  // p[i] = i
int Find(int x) { while (p[x] != x) { p[x] = p[p[x]]; x = p[x]; } return x; }
bool Union(int a, int b) {
    a = Find(a); b = Find(b); if (a == b) return false;
    if (r[a] < r[b]) (a, b) = (b, a);
    p[b] = a; if (r[a] == r[b]) r[a]++; return true;
}
```
- 305: on each add-land, `count++`, then `count--` for each successful union. Skip duplicate adds.
- 1135: sort edges and union them, summing costs. You need n−1 unions, otherwise return −1.

### 265 Paint House II: keep the best, second best, and the best's color
```csharp
int m1 = 0, m2 = 0, i1 = -1;
foreach (var row in costs) {
    int n1 = int.MaxValue, n2 = int.MaxValue, ni = -1;
    for (int c = 0; c < row.Length; c++) {
        int cost = row[c] + (c == i1 ? m2 : m1);
        if (cost < n1) { n2 = n1; n1 = cost; ni = c; } else if (cost < n2) n2 = cost;
    }
    (m1, m2, i1) = (n1, n2, ni);
}
return m1;
```

### 277 Find the Celebrity
```csharp
int cand = 0;
for (int i = 1; i < n; i++) if (Knows(cand, i)) cand = i;
for (int i = 0; i < n; i++) if (i != cand && (Knows(cand, i) || !Knows(i, cand))) return -1;
return cand;
```

### Robot Room Cleaner / Mouse finds cheese: DFS on relative coordinates, physically move back
```csharp
(int dx, int dy)[] D = { (-1, 0), (0, 1), (1, 0), (0, -1) };  // clockwise
void Dfs(Robot r, int x, int y, int d) {
    seen.Add((x, y)); r.Clean();
    for (int i = 0; i < 4; i++) {
        int nd = (d + i) % 4, nx = x + D[nd].dx, ny = y + D[nd].dy;
        if (!seen.Contains((nx, ny)) && r.Move()) { Dfs(r, nx, ny, nd); r.TurnRight(); r.TurnRight(); r.Move(); r.TurnRight(); r.TurnRight(); }
        r.TurnRight();
    }
}
// Mouse: if (mouse.Move(d)) { if (Dfs(...)) return true; mouse.Move(Opposite(d)); }   stop as soon as IsCheese()
```

---

## One-liners

| Problem | Key idea |
|---|---|
| 200 Islands | Flood-fill with an explicit `Stack<T>`, because recursion overflows the stack on 300x300 |
| 53 Max Subarray | `cur = max(x, cur + x); best = max(best, cur)` |
| 152 Max Product | On a negative x, swap `hi` and `lo`, then `hi = max(x, hi*x)`, `lo = min(x, lo*x)` |
| 121 Stock | `best = max(best, p - minSoFar)` |
| 135 Candy | Left pass for rising ratings, then right pass with `max(c[i], c[i+1] + 1)` |
| 366 Find Leaves | Height from the bottom (`leaf = 0`) is the bucket index |
| 272 Closest BST II | Predecessor stack + successor stack, merge k values. O(log n + k) |
| 215 Kth Largest | Min-heap of size k. Quickselect for O(n) average |
| 373 K Pairs | Seed the heap with `(i, 0)` for i < k. When you pop `(i, j)`, push `(i, j+1)` |
| 76 Min Window | Need-count map plus a `missing` counter. Expand right, shrink left while `missing == 0` |
| 727 Min Window Subsequence | Scan forward to match T, then scan backward to shrink. Restart at `start + 1` |
| 210 Course Schedule II | Kahn's algorithm. If the output count is less than n, there's a cycle |
| 156 Upside Down | On the left spine: `left.left = right`, `left.right = parent` |
| 235 LCA BST | Both smaller: go left. Both larger: go right. Otherwise this node is the answer |
| 65 Valid Number | Flags `digit`, `dot`, `e`. A sign is only allowed at index 0 or after `e`. Need a digit after `e` |

**C# gotchas:** `PriorityQueue` has no Remove (skip stale entries when you pop them). `SortedSet` drops duplicates, so store `(val, id)`. Tuples work as dictionary and set keys. Use `TryGetValue`. `^1` is the last element.
