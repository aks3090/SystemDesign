# Classic coding round: detailed revision (C#)

Each problem has: **Problem**, **Example**, **Idea**, **Code**, **Complexity**, **Edge cases / follow-ups**.
The order is by how often each problem is reported. If time runs out, stop after Part B.

`using System; using System.Collections.Generic; using System.Linq; using System.Text;`

**Habits for every problem (Staff signal):** clarify (size, duplicates, empty input, ties, whether you may mutate the input), name the brute force and its cost, code the optimal, narrate the invariant, walk the example by hand, then raise the follow-up yourself.

## Contents
- **Part A, most reported:** 716 Max Stack · 364 Nested List Weight Sum II · 341 Flatten Nested Iterator · 380 / 381 GetRandom · 432 All O`one · 146 LRU · 1004 Max Ones III · 244 / 245 Word Distance · 605 Flowers
- **Part B, LinkedIn custom:** Connection Distance · Mutual Connections · TaskList + cycles · LZ78 · Restaurant election · Mouse and cheese / Robot cleaner · Tallest tower
- **Part C, graphs:** 200 / 305 Islands · 1135 MST · 210 Course Schedule II · 277 Celebrity
- **Part D, DP:** 256 / 265 / 1473 Paint House · 53 · 152 · 121 · 135 Candy
- **Part E, trees and heaps:** 366 · 272 · 156 · 235 · 104 · 215 · 373
- **Part F, strings and easy problems:** 76 · 727 · 65 · 17 · 20 · 21 · 367

---

# Part A: most reported

## 716. Max Stack (24 reports, the top one)

**Problem.** Design a stack with `Push(x)`, `Pop()`, `Top()`, `PeekMax()`, and `PopMax()`. `PopMax` removes the max element. If several elements tie for max, it removes the **top-most** (most recently pushed) one.

**Example.**
```
Push(5) Push(1) Push(5)      stack (bottom→top): 5 1 5
Top()     → 5
PopMax()  → 5                removes the TOP 5 (most recent)   stack: 5 1
Top()     → 1
PeekMax() → 5
Pop()     → 1                stack: 5
Top()     → 5
```

**Idea.**
- With one structure you get either O(1) top (a stack) or O(log n) max (a heap), not both.
- So keep **two ordered views of the same elements**: one ordered by push time, one ordered by value.
- Give each push a unique `id` (a counter). Then `(id, val)` and `(val, id)` identify the element in both views, so you can delete it from the other view in O(log n).
- `(val, id)` ordering puts the most recent one last among equal values, which is exactly the tie rule.
- Say the brute force first: two stacks (main plus running max) make `PopMax` O(n), because you pop into a buffer until you reach the max and then push back.

**Code.**
```csharp
public class MaxStack {
    readonly SortedSet<(int id, int val)> byTime = new();   // Max = top of stack
    readonly SortedSet<(int val, int id)> byVal  = new();   // Max = largest val; on ties largest id (most recent)
    int nextId = 0;

    public void Push(int x) { byTime.Add((nextId, x)); byVal.Add((x, nextId)); nextId++; }

    public int Pop() {
        var t = byTime.Max;
        byTime.Remove(t); byVal.Remove((t.val, t.id));
        return t.val;
    }
    public int Top() => byTime.Max.val;
    public int PeekMax() => byVal.Max.val;

    public int PopMax() {
        var m = byVal.Max;
        byVal.Remove(m); byTime.Remove((m.id, m.val));
        return m.val;
    }
}
```

**Dry run.** After Push(5), Push(1), Push(5): `byTime = {(0,5),(1,1),(2,5)}` and `byVal = {(1,1),(5,0),(5,2)}`. PopMax takes `byVal.Max = (5,2)`, then removes `(2,5)` from byTime. Top is now `(1,1)`, so it returns 1. ✓

**Complexity.** O(log n) per operation, O(n) space.

**Follow-ups.**
- *"Do it without SortedSet."* Use a hand-rolled doubly linked list as the stack, plus a max-heap `PriorityQueue<(val,id)>`, plus `HashSet<int> deadIds`. On `Pop`, unlink the node and mark its id dead. On `PeekMax` and `PopMax`, first discard heap entries whose id is dead (lazy deletion). That gives amortized O(log n).
- *"Why not `SortedSet<int>`?"* It drops duplicates. The unique id is what makes duplicates work.

---

## 364. Nested List Weight Sum II (19 reports)

**Problem.** You get a nested list of integers, for example `[1,[4,[6]]]`. Depth starts at 1 at the top level. Each integer's weight is `maxDepth - depth + 1`, so **deeper integers get smaller weights**. Return the sum of `value * weight`.

```csharp
// LeetCode interface
public interface NestedInteger { bool IsInteger(); int GetInteger(); IList<NestedInteger> GetList(); }
```

**Examples.**
```
[[1,1],2,[1,1]]  → 8     maxDepth 2: four 1s at depth 2 (weight 1) = 4, the 2 at depth 1 (weight 2) = 4
[1,[4,[6]]]      → 17    maxDepth 3: 1*3 + 4*2 + 6*1
```

**Idea.**
- The obvious approach is two passes: one to find maxDepth, one to sum.
- One pass works with algebra: `Σ v·(maxDepth − d + 1) = (maxDepth + 1)·Σv − Σ(v·d)`. Collect three numbers in a single DFS: `flat = Σv`, `weighted = Σ v·d`, and `maxDepth`.
- **Only update maxDepth when you see an integer.** Otherwise `[1,[[]]]` would count the empty lists as levels.

**Code.**
```csharp
public class Solution {
    int maxDepth = 0, flat = 0, weighted = 0;

    public int DepthSumInverse(IList<NestedInteger> list) {
        Dfs(list, 1);
        return (maxDepth + 1) * flat - weighted;
    }
    void Dfs(IList<NestedInteger> list, int d) {
        foreach (var n in list) {
            if (n.IsInteger()) {
                int v = n.GetInteger();
                flat += v; weighted += v * d;
                maxDepth = Math.Max(maxDepth, d);
            } else Dfs(n.GetList(), d + 1);
        }
    }
}
```

**Dry run.** `[1,[4,[6]]]` gives flat = 11, weighted = 1·1 + 4·2 + 6·3 = 27, and maxDepth = 3. Then 4·11 − 27 = **17** ✓

**Complexity.** O(n) time over all elements, O(depth) stack.

**Edge cases / follow-ups.**
- **The well-known BFS trick:** go level by level, keep `running += levelIntegers`, and add `total += running` once per level. It's elegant, but **it overcounts `[1,[[]]]`**, because the trailing empty-list level adds `running` one extra time. Pointing that out is a strong Staff signal.
- **339 (weight = depth)** is just `weighted`.
- For very deep nesting, use an explicit stack of `(list, depth)`.

---

## 341. Flatten Nested List Iterator (Staff-tagged)

**Problem.** Implement `Next()` and `HasNext()` over a nested list so that it yields the integers in left-to-right, depth-first order.

**Example.** `[[1,1],2,[1,1]]` yields 1, 1, 2, 1, 1. `[1,[4,[6]]]` yields 1, 4, 6. `[[],[[]]]` yields nothing.

**Idea.**
- The eager approach flattens everything into a list in the constructor. That's simple, but it uses O(n) memory up front and isn't lazy, which is what the interviewer wants to see you avoid.
- The lazy approach keeps a **stack of enumerators**, one per open list.
- `HasNext` does the real work. It advances until it finds an integer, or until the stack is empty, and caches that integer. Calling it twice is safe because it's idempotent.

**Code.**
```csharp
public class NestedIterator {
    readonly Stack<IEnumerator<NestedInteger>> st = new();
    int? nextVal;

    public NestedIterator(IList<NestedInteger> list) => st.Push(list.GetEnumerator());

    public bool HasNext() {
        while (nextVal == null && st.Count > 0) {
            var it = st.Peek();
            if (!it.MoveNext()) { st.Pop(); continue; }          // this list is exhausted
            if (it.Current.IsInteger()) nextVal = it.Current.GetInteger();
            else st.Push(it.Current.GetList().GetEnumerator());  // descend
        }
        return nextVal != null;
    }

    public int Next() {
        HasNext();
        int v = nextVal!.Value; nextVal = null;
        return v;
    }
}
```

**Complexity.** Amortized O(1) per call. Space is O(depth).

**Follow-ups.**
- Empty lists `[[]]` are handled by the loop.
- If the list is modified during iteration, `MoveNext` throws `InvalidOperationException`. Say so.

---

## 380. Insert Delete GetRandom O(1) (10 reports)

**Problem.** Implement a set with `Insert(val)` and `Remove(val)`, which return a bool for whether the set changed, and `GetRandom()`, which returns each current element with equal probability. All operations must be **average O(1)**.

**Example.**
```
Insert(1) → true     Remove(2) → false     Insert(2) → true
GetRandom() → 1 or 2 (50/50)
Remove(1) → true     Insert(2) → false     GetRandom() → 2
```

**Idea.**
- A HashSet gives O(1) insert and remove but no O(1) random index. A List gives O(1) random index but O(n) remove.
- Use both: a `List<int> vals`, plus a `Dictionary<int,int> idx` mapping each value to its position in the list.
- **Remove trick:** move the last element into the hole, update its index, then pop the last slot, which is O(1).

**Code.**
```csharp
public class RandomizedSet {
    readonly List<int> vals = new();
    readonly Dictionary<int, int> idx = new();
    readonly Random rng = new();

    public bool Insert(int v) {
        if (idx.ContainsKey(v)) return false;
        idx[v] = vals.Count; vals.Add(v);
        return true;
    }

    public bool Remove(int v) {
        if (!idx.TryGetValue(v, out int i)) return false;
        int last = vals[^1];
        vals[i] = last; idx[last] = i;        // move last into the hole
        vals.RemoveAt(vals.Count - 1);
        idx.Remove(v);                        // AFTER idx[last] = i, so it works when v == last
        return true;
    }

    public int GetRandom() => vals[rng.Next(vals.Count)];
}
```

**Dry run.** Remove(1) from `vals = [1,2]`: i = 0 and last = 2, so `vals = [2,2]` and `idx[2] = 0`. Pop gives `[2]`, then remove key 1 ✓

**Follow-up, 381 (duplicates allowed).** Map each value to the **set** of its indexes:
```csharp
public class RandomizedCollection {
    readonly List<int> vals = new();
    readonly Dictionary<int, HashSet<int>> idx = new();
    readonly Random rng = new();

    public bool Insert(int v) {
        if (!idx.TryGetValue(v, out var set)) idx[v] = set = new();
        set.Add(vals.Count); vals.Add(v);
        return set.Count == 1;                       // true if v was not present before
    }

    public bool Remove(int v) {
        if (!idx.TryGetValue(v, out var set)) return false;
        int i = set.First(); set.Remove(i);
        int lastIdx = vals.Count - 1, last = vals[lastIdx];
        vals[i] = last;
        idx[last].Add(i); idx[last].Remove(lastIdx); // order matters when i == lastIdx
        vals.RemoveAt(lastIdx);
        if (set.Count == 0) idx.Remove(v);
        return true;
    }

    public int GetRandom() => vals[rng.Next(vals.Count)];
}
```

**Staff talk.**
- Explain why the probability is uniform: the list holds exactly one slot per element.
- `Random` isn't thread-safe. Use `Random.Shared` on .NET 6+, or one `Random` per thread.

---

## 432. All O`one Data Structure (10 reports, hardest design problem)

**Problem.**
- `Inc(key)`: add 1 to key's count; a new key starts at 1.
- `Dec(key)`: subtract 1; remove the key when its count reaches 0.
- `GetMaxKey()` and `GetMinKey()`: return any key with the max or min count, or `""` if empty.

All four must be **O(1)**.

**Example.**
```
Inc("hello") Inc("hello")
GetMaxKey() → "hello"   GetMinKey() → "hello"
Inc("leet")
GetMaxKey() → "hello"   GetMinKey() → "leet"
```

**Idea.**
- A heap gives O(log n), which is too slow. The insight is that **counts only ever change by ±1**, so a key only moves to the *neighbouring* count.
- Keep a **doubly linked list of buckets** in ascending count order. Each bucket holds a set of keys with that count. Also keep `Dictionary<key, bucket>`.
- The min is `head.Next` and the max is `tail.Prev`.
- `Inc`: move the key to the bucket with count+1, creating it right after the current bucket if needed. Remove the old bucket if it's now empty.
- `Dec`: the mirror image, moving toward the front.

```
After the example:   head ⇄ [1: {leet}] ⇄ [2: {hello}] ⇄ tail
```

**Code.**
```csharp
public class AllOne {
    class Bucket { public int Count; public HashSet<string> Keys = new(); public Bucket Prev, Next; }

    readonly Bucket head = new(), tail = new();              // sentinels; head.Count = 0
    readonly Dictionary<string, Bucket> where = new();

    public AllOne() { head.Next = tail; tail.Prev = head; }

    Bucket InsertAfter(Bucket b, int count) {
        var n = new Bucket { Count = count, Prev = b, Next = b.Next };
        b.Next.Prev = n; b.Next = n;
        return n;
    }
    void RemoveIfEmpty(Bucket b) {
        if (b.Keys.Count == 0) { b.Prev.Next = b.Next; b.Next.Prev = b.Prev; }
    }

    public void Inc(string key) {
        var cur = where.TryGetValue(key, out var b) ? b : head;   // a new key behaves as if it were in head (count 0)
        var nxt = cur.Next;
        if (nxt == tail || nxt.Count != cur.Count + 1) nxt = InsertAfter(cur, cur.Count + 1);
        nxt.Keys.Add(key); where[key] = nxt;
        if (cur != head) { cur.Keys.Remove(key); RemoveIfEmpty(cur); }   // remove old AFTER inserting new
    }

    public void Dec(string key) {
        if (!where.TryGetValue(key, out var cur)) return;
        cur.Keys.Remove(key);
        if (cur.Count == 1) where.Remove(key);                    // count drops to 0: the key is gone
        else {
            var prv = cur.Prev;
            if (prv == head || prv.Count != cur.Count - 1) prv = InsertAfter(prv, cur.Count - 1);
            prv.Keys.Add(key); where[key] = prv;
        }
        RemoveIfEmpty(cur);
    }

    public string GetMaxKey() => tail.Prev == head ? "" : tail.Prev.Keys.First();
    public string GetMinKey() => head.Next == tail ? "" : head.Next.Keys.First();
}
```

**Dry run.**
- `Inc("hello")`: cur = head, and no bucket 1 exists, so create it: `head ⇄ [1:{hello}]`.
- Second `Inc("hello")`: cur = bucket 1. Create bucket 2 after it, move "hello" there, and bucket 1 is now empty so remove it: `head ⇄ [2:{hello}]`.
- `Inc("leet")`: cur = head, and head.Next has count 2, not 1, so insert bucket 1 between them: `head ⇄ [1:{leet}] ⇄ [2:{hello}]` ✓

**Complexity.** O(1) per operation (`HashSet.First()` is effectively O(1)).

**Edge cases to say out loud.**
- **Insert the new bucket before removing the old one.** If you remove first, you lose your position in the list.
- Calling `Dec` on a missing key is a no-op.
- **Strict O(1) follow-up:** after many deletes, `HashSet.First()` can scan empty slots. Use a `LinkedList<string>` per bucket and store the node in the map.

---

## 146. LRU Cache (Staff-tagged)

**Problem.** `LRUCache(capacity)`, `Get(key)` (returns -1 on a miss), `Put(key, value)`. When over capacity, evict the **least recently used** key. Both `Get` and `Put` count as a use. Everything is O(1).

**Example.**
```
cap = 2
Put(1,1) Put(2,2) Get(1) → 1       order (MRU→LRU): 1 2
Put(3,3)                            evicts 2      order: 3 1
Get(2) → -1
Put(4,4)                            evicts 1      order: 4 3
Get(1) → -1   Get(3) → 3   Get(4) → 4
```

**Idea.** A hashmap gives O(1) lookup, and a doubly linked list gives O(1) move-to-front and remove-tail. The map stores the **node**, so you can unlink it without searching.

**Code (hand-rolled, which interviewers often ask for).**
```csharp
public class LRUCache {
    class Node { public int Key, Val; public Node Prev, Next; }

    readonly Dictionary<int, Node> map = new();
    readonly Node head = new(), tail = new();      // head.Next = MRU, tail.Prev = LRU
    readonly int cap;

    public LRUCache(int capacity) { cap = capacity; head.Next = tail; tail.Prev = head; }

    void Unlink(Node n)   { n.Prev.Next = n.Next; n.Next.Prev = n.Prev; }
    void AddFront(Node n) { n.Next = head.Next; n.Prev = head; head.Next.Prev = n; head.Next = n; }

    public int Get(int key) {
        if (!map.TryGetValue(key, out var n)) return -1;
        Unlink(n); AddFront(n);
        return n.Val;
    }

    public void Put(int key, int value) {
        if (map.TryGetValue(key, out var n)) {             // update existing: no eviction
            n.Val = value; Unlink(n); AddFront(n); return;
        }
        if (map.Count == cap) {                            // evict the LRU node from BOTH structures
            var lru = tail.Prev; Unlink(lru); map.Remove(lru.Key);
        }
        n = new Node { Key = key, Val = value };
        map[key] = n; AddFront(n);
    }
}
```
With built-ins, the same thing is `LinkedList<(int k,int v)>` plus `Dictionary<int, LinkedListNode<(int k,int v)>>`, using `list.Remove(node)`, `list.AddFirst(node)`, and `list.RemoveLast()`.

**Complexity.** O(1) per operation.

**Staff follow-ups.**
- **Thread safety:** one lock around every public method, including reads, because `Get` mutates the order. Don't use `ConcurrentDictionary`, because the map and the list must change together.
- **Scaling:** shard into N caches by key hash, which gives approximate global LRU.
- **TTL:** store an expiry per node and treat `now >= expiry` as a miss, removing lazily.
- **Bug to avoid:** removing the evicted entry from the list but not the map, or the other way around.

---

## 1004. Max Consecutive Ones III (10 reports)

**Problem.** You have a binary array and may flip at most `k` zeros to ones. Return the longest run of consecutive 1s you can get.

**Examples.**
```
nums = [1,1,1,0,0,0,1,1,1,1,0], k = 2 → 6     flip indexes 5 and 10: [1,1,1,1,1,1] at 5..10
nums = [0,0,1,1,0,0,1,1,1,0,1,1,0,0,0,1,1,1,1], k = 3 → 10
```

**Idea.**
- Rephrase it: find the **longest window containing at most k zeros**.
- Use a sliding window. Expand on the right. While the window has more than k zeros, shrink from the left.
- Invariant: "the window always has at most k zeros."

**Code.**
```csharp
public int LongestOnes(int[] a, int k) {
    int left = 0, zeros = 0, best = 0;
    for (int r = 0; r < a.Length; r++) {
        if (a[r] == 0) zeros++;
        while (zeros > k) {
            if (a[left] == 0) zeros--;
            left++;
        }
        best = Math.Max(best, r - left + 1);
    }
    return best;
}
```

**Complexity.** O(n) time, because each index enters and leaves the window once. O(1) space.

**Follow-ups.**
- **487 (k = 1)** is the same code.
- **Streaming input** (too big for memory, so you can't index back): keep a `Queue<int>` with the positions of the last k+1 zeros. When a new zero arrives and the queue is full, `left = queue.Dequeue() + 1`.
- The same template works for "longest substring with at most k distinct characters".

---

## 244. Shortest Word Distance II (7 reports)

**Problem.** The constructor receives `string[] words`. `Shortest(word1, word2)` returns the minimum `|i − j|` where `words[i] == word1` and `words[j] == word2`. It will be called **many times**.

**Example.**
```
words = ["practice","makes","perfect","coding","makes"]
Shortest("coding","practice") → 3      indexes 3 and 0
Shortest("makes","coding")    → 1      "makes" at 1 and 4, "coding" at 3 → |4-3| = 1
```

**Idea.**
- Preprocess each word into its **sorted** list of positions. The lists come out sorted for free because you scan left to right.
- For a query, merge the two sorted lists with two pointers: always advance the pointer at the smaller value, since that's the only move that can shrink the gap.

**Code.**
```csharp
public class WordDistance {
    readonly Dictionary<string, List<int>> pos = new();

    public WordDistance(string[] words) {
        for (int i = 0; i < words.Length; i++) {
            if (!pos.TryGetValue(words[i], out var l)) pos[words[i]] = l = new();
            l.Add(i);
        }
    }

    public int Shortest(string a, string b) {
        var x = pos[a]; var y = pos[b];
        int i = 0, j = 0, best = int.MaxValue;
        while (i < x.Count && j < y.Count) {
            best = Math.Min(best, Math.Abs(x[i] - y[j]));
            if (x[i] < y[j]) i++; else j++;
        }
        return best;
    }
}
```

**Complexity.** The constructor is O(n). Each query is O(|x| + |y|).

**Variants.**
- **243 (a single query)** and **245 (word1 may equal word2)** need only one scan with no preprocessing:
```csharp
public int ShortestWordDistance(string[] words, string a, string b) {
    int last = -1, best = int.MaxValue;              // last index where a or b appeared
    for (int i = 0; i < words.Length; i++) {
        if (words[i] != a && words[i] != b) continue;
        if (last != -1 && (a == b || words[i] != words[last])) best = Math.Min(best, i - last);
        last = i;
    }
    return best;
}
```

**Staff follow-ups.**
- Memoize `(a, b)` results in a dictionary for repeated queries.
- If one list is tiny and the other huge, binary search each small position in the big list for O(s · log L).
- Memory is O(n) total positions.

---

## 605. Can Place Flowers (7 reports): do it in 5 minutes with no bugs

**Problem.** In a 0/1 flowerbed, no two 1s may be adjacent. Can you plant `n` more flowers?

**Examples.** `[1,0,0,0,1]` with n = 1 → true (plant at index 2). The same bed with n = 2 → false.

**Idea.** Scan greedily. Plant at i if it's 0 and both neighbours are 0, treating out-of-bounds as 0. Planting as early as possible never hurts.

**Code.**
```csharp
public bool CanPlaceFlowers(int[] f, int n) {
    for (int i = 0; i < f.Length && n > 0; i++) {
        bool leftEmpty  = i == 0 || f[i - 1] == 0;
        bool rightEmpty = i == f.Length - 1 || f[i + 1] == 0;
        if (f[i] == 0 && leftEmpty && rightEmpty) { f[i] = 1; n--; }
    }
    return n <= 0;
}
```

**Complexity.** O(n) time, O(1) space.

**Edge cases.**
- This mutates the input. Say so, or copy the array.
- n = 0 → true.
- `[0]` with n = 1 → true.

---

# Part B: LinkedIn custom problems (most on-theme for Marketplace Integrity)

## LinkedIn Connection Distance (Staff)

**Problem.** Given a social graph, return the degree of separation (the shortest path length) between two users. Then: *"How do you handle the search-space explosion from influencer nodes with thousands of connections while keeping complete accuracy?"*

**Example.**
```
Edges: 1-2, 2-3, 3-4, 1-5, 5-9, 9-[1000 other users]   (9 is an influencer)
Distance(1, 4) → 3        1 → 2 → 3 → 4
Distance(1, 1) → 0
Distance(1, 99) → -1      not connected
```

**Idea.**
- Plain BFS from s visits about b^d nodes, where b is the branching factor and d the distance. If one hop goes through an influencer, the frontier explodes.
- **Bidirectional BFS** grows from both ends and stops when they meet. That's about 2·b^(d/2) nodes, which is a huge saving.
- **Always expand the smaller frontier.** If one side is about to expand an influencer, the other side is probably smaller and gets expanded instead. Nothing is pruned, so **the answer stays exact**. That's the "complete accuracy" part.
- Expand one whole layer at a time, and check each neighbour against the other side's visited set.

**Code.**
```csharp
public int Distance(Dictionary<int, List<int>> g, int s, int t) {
    if (s == t) return 0;
    var front = new HashSet<int> { s }; var back = new HashSet<int> { t };
    var seenF = new HashSet<int> { s }; var seenB = new HashSet<int> { t };
    int depth = 0;

    while (front.Count > 0 && back.Count > 0) {
        if (front.Count > back.Count) {                       // always expand the smaller side
            (front, back) = (back, front);
            (seenF, seenB) = (seenB, seenF);
        }
        depth++;
        var next = new HashSet<int>();
        foreach (var u in front) {
            if (!g.TryGetValue(u, out var nbrs)) continue;
            foreach (var v in nbrs) {
                if (seenB.Contains(v)) return depth;          // the frontiers met
                if (seenF.Add(v)) next.Add(v);
            }
        }
        front = next;
    }
    return -1;
}
```

**Dry run (path 1-2-3-4).**
- `front={1}` and `back={4}`.
- depth 1: expand {1} and get {2}.
- depth 2: expand {2} and get {3}.
- depth 3: expanding {3} sees 4, which is in seenB, so it returns **3** ✓

**Complexity.** O(b^(d/2)) nodes and edges touched, versus O(b^d) for plain BFS.

**Staff talking points.**
- **Degree cap.** LinkedIn shows 1st, 2nd and 3rd degree, so stop when `depth == 3` and return "3+".
- **Precompute.** Store each user's 1st and 2nd-degree sets offline. Then "is t within 3?" becomes "does my 2nd-degree set intersect t's 1st-degree set?", which is a set intersection.
- **Distributed graph.** Each BFS layer is one batched fan-out per shard. Latency is driven by the number of round trips, so bidirectional search also halves the round trips.
- **Path, not just distance.** Keep a parent map on each side and stitch the two halves together at the meeting node.
- **Integrity angle.** Fake accounts often connect to many strangers with few mutual connections. Distance and mutual counts are standard features for that.

---

## Mutual Connections in a Social Graph

**Problem.** Given user connections (undirected), implement:
1. Mutual friends of A and B.
2. Second-degree connections of A, ranked by number of mutual friends ("People You May Know").
3. The shortest path between two users (BFS).

**Example.**
```
A: B C D        B: A C E        C: A B E F
D: A            E: B C          F: C

Mutual(A, B)       → [C]
SecondDegree(A)    → [(E, 2), (F, 1)]     E via B and C; F via C. B and C are excluded (direct friends).
ShortestPath(D, F) → [D, A, C, F]
```

**Code.**
```csharp
public class SocialGraph {
    readonly Dictionary<string, HashSet<string>> adj = new();

    HashSet<string> Friends(string u) {
        if (!adj.TryGetValue(u, out var s)) adj[u] = s = new();
        return s;
    }
    public void Connect(string a, string b) { Friends(a).Add(b); Friends(b).Add(a); }

    // 1. Iterate the SMALLER set and probe the larger one: O(min(|A|, |B|))
    public List<string> Mutual(string a, string b) {
        var fa = Friends(a); var fb = Friends(b);
        var (small, big) = fa.Count <= fb.Count ? (fa, fb) : (fb, fa);
        return small.Where(big.Contains).ToList();
    }

    // 2. Friends of friends, minus me, minus my direct friends; the score is the number of mutual friends
    public List<(string User, int Mutuals)> SecondDegree(string me) {
        var mine = Friends(me);
        var score = new Dictionary<string, int>();
        foreach (var f in mine)
            foreach (var ff in Friends(f))
                if (ff != me && !mine.Contains(ff))
                    score[ff] = score.GetValueOrDefault(ff) + 1;
        return score.OrderByDescending(kv => kv.Value)
                    .Select(kv => (kv.Key, kv.Value)).ToList();
    }

    // 3. BFS with a parent map to rebuild the path
    public List<string>? ShortestPath(string s, string t) {
        var parent = new Dictionary<string, string?> { [s] = null };
        var q = new Queue<string>(); q.Enqueue(s);
        while (q.Count > 0) {
            var u = q.Dequeue();
            if (u == t) {
                var path = new List<string>();
                for (string? x = t; x != null; x = parent[x]) path.Add(x);
                path.Reverse();
                return path;
            }
            if (!adj.TryGetValue(u, out var nbrs)) continue;
            foreach (var v in nbrs)
                if (parent.TryAdd(v, u)) q.Enqueue(v);     // TryAdd doubles as the "visited" check
        }
        return null;
    }
}
```

**Complexity.**
- Mutual is O(min degree).
- SecondDegree is O(Σ degree of my friends), plus O(m log m) for sorting. For a top-k only, use a size-k heap.
- ShortestPath is O(V + E).

**Staff follow-ups.**
- **Influencer friends:** a friend with 30k connections blows up SecondDegree. Cap the fan-out per friend, or sample. Precompute offline, because PYMK runs as a batch job.
- **Directed (follow) graph vs undirected (connection) graph:** clarify which one you have.
- For a faster shortest path, use the bidirectional BFS above.
- **Integrity signal:** an account sending many invites to people with zero mutual connections is a common spam pattern.

---

## Implement a TaskList with Prerequisites + Cycle Detection

**Problem.** Each `Task` has `sub_tasks` (prerequisites that must complete first) and an action.
- `AddTask(task)` adds the task **and all its subtasks recursively**. It throws if that would create a circular dependency.
- `PollNext()` returns the next task whose prerequisites are all complete, or `null` when everything is finished.

**Clarify first.**
1. Does `PollNext` run and complete the task, or is there a separate `Complete(task)`? I'll assume it runs and completes.
2. Can the same task be added twice? I'll make it idempotent.
3. Are tasks compared by reference? I'll assume yes.

**Example.**
```
C has no subtasks. B.subTasks = [C]. A.subTasks = [B, C].
AddTask(A)          registers A, B, C
PollNext() → C      (no prerequisites)
PollNext() → B      (C done)
PollNext() → A      (B and C done)
PollNext() → null

X.subTasks = [Y], Y.subTasks = [X]
AddTask(X) → throws (cycle)
```

**Idea.**
- **Cycle check:** run a DFS with an `onPath` set (the gray nodes) and a `cleared` set (the black nodes). Seeing a node that's already on the path is a back edge, which means a cycle. **Validate before mutating**, so a failed add leaves nothing half-registered.
- **Scheduling:** use Kahn's algorithm incrementally.
  - `pending[t]` counts t's unfinished prerequisites.
  - `dependents[p]` lists the tasks waiting on p.
  - When p completes, decrement the count of each of its dependents. When a count hits 0, that task goes into the `ready` queue.

**Code.**
```csharp
public class TaskItem {
    public string Id = "";
    public List<TaskItem> SubTasks = new();
    public Action? Run;
}

public class TaskList {
    readonly Dictionary<TaskItem, int> pending = new();               // unfinished prerequisite count
    readonly Dictionary<TaskItem, List<TaskItem>> dependents = new(); // prereq → tasks waiting on it
    readonly HashSet<TaskItem> known = new(), done = new();
    readonly Queue<TaskItem> ready = new();

    public void AddTask(TaskItem t) {
        if (HasCycle(t, new HashSet<TaskItem>(), new HashSet<TaskItem>()))
            throw new InvalidOperationException($"Cycle detected via {t.Id}");
        Register(t);                                                  // only after validation
    }

    bool HasCycle(TaskItem t, HashSet<TaskItem> onPath, HashSet<TaskItem> cleared) {
        if (onPath.Contains(t)) return true;                          // back edge → cycle
        if (cleared.Contains(t)) return false;                        // already proven acyclic
        onPath.Add(t);
        foreach (var s in t.SubTasks)
            if (HasCycle(s, onPath, cleared)) return true;
        onPath.Remove(t);
        cleared.Add(t);
        return false;
    }

    void Register(TaskItem t) {
        if (!known.Add(t)) return;                                    // idempotent
        int waiting = 0;
        foreach (var s in t.SubTasks) {
            Register(s);                                              // subtasks first
            if (done.Contains(s)) continue;
            waiting++;
            if (!dependents.TryGetValue(s, out var list)) dependents[s] = list = new();
            list.Add(t);
        }
        pending[t] = waiting;
        if (waiting == 0) ready.Enqueue(t);
    }

    public TaskItem? PollNext() {
        if (ready.Count == 0) return null;
        var t = ready.Dequeue();
        t.Run?.Invoke();
        done.Add(t);
        if (dependents.TryGetValue(t, out var waiters))
            foreach (var w in waiters)
                if (--pending[w] == 0) ready.Enqueue(w);
        return t;
    }
}
```

**Dry run (A → [B, C], B → [C]).**
- Register(A) calls Register(B), which calls Register(C). C has `pending = 0`, so `ready = [C]`.
- Back in B: it waits on C, so `pending[B] = 1` and `dependents[C] = [B]`.
- Back in A: it waits on B and C, so `pending[A] = 2`, `dependents[B] = [A]`, and `dependents[C] = [B, A]`.
- Poll returns C. That decrements B to 0 (B becomes ready) and A to 1.
- Poll returns B, and A drops to 0.
- Poll returns A. Poll returns null ✓

**Complexity.** `AddTask` is O(V + E) over the new subgraph. `PollNext` is O(out-degree).

**Staff follow-ups.**
- **Parallel workers:** split `PollNext` into `Claim()` (dequeue without completing) and `Complete(t)`, guard the state with a lock, and let N workers pull from `ready`.
- **A task throws:** decide whether to retry it, or to mark it failed and cancel its dependents transitively.
- **Deep chains:** recursion can overflow, so use an iterative DFS with an explicit stack.
- **Course Schedule II (210)** is the same Kahn's algorithm run once over the whole graph.

---

## Implement LZ78 Compression and Decompression

**Problem.** Compress a string into `(index, char)` tuples using an LZ78 dictionary, then decompress the tuples back into the original string.

**LZ78 rules.**
- The dictionary starts with entry 0 = "" (the empty phrase).
- Read the **longest prefix w that's already in the dictionary**, plus the next char c.
- Emit `(index(w), c)` and add `w + c` as a new entry.
- If the input ends while w is non-empty and already known, emit `(index(w), null)`.

**Example (small).** `"ABAABABA"`

| Step | Longest known w | Next c | Emit | New entry |
|---|---|---|---|---|
| 1 | "" | A | (0, A) | 1 = A |
| 2 | "" | B | (0, B) | 2 = B |
| 3 | "A" (1) | A | (1, A) | 3 = AA |
| 4 | "B" (2) | A | (2, A) | 4 = BA |
| 5 | "BA" (4) | end | (4, null) | - |

Decompressing gives A + B + AA + BA + BA = `ABAABABA` ✓

A larger test: `"ABBCBCABABCAABCAAB"` → `(0,A)(0,B)(2,C)(3,A)(2,A)(4,A)(6,B)`.

**Idea.**
- The naive version uses `Dictionary<string,int>` and builds `w + c` strings, which is O(n·L).
- **Use a trie instead.** Each dictionary entry is a trie node id, and `(nodeId, char)` maps to the child id. Walking the input is then O(1) per char, so O(n) total.

**Code.**
```csharp
public static class LZ78 {
    public static List<(int Index, char? Ch)> Compress(string s) {
        var child = new Dictionary<(int node, char c), int>();       // trie edges
        var output = new List<(int, char?)>();
        int node = 0, nextId = 1;                                    // 0 = the empty phrase
        foreach (char c in s) {
            if (child.TryGetValue((node, c), out int nx)) { node = nx; continue; }   // extend w
            output.Add((node, c));                                   // emit (index(w), c)
            child[(node, c)] = nextId++;                             // add w + c
            node = 0;                                                // restart from the empty phrase
        }
        if (node != 0) output.Add((node, null));                     // leftover phrase is already known
        return output;
    }

    public static string Decompress(List<(int Index, char? Ch)> codes) {
        var phrases = new List<string> { "" };                       // phrases[i] = dictionary entry i
        var sb = new StringBuilder();
        foreach (var (idx, ch) in codes) {
            string p = ch is char c ? phrases[idx] + c : phrases[idx];
            sb.Append(p);
            phrases.Add(p);
        }
        return sb.ToString();
    }
}
```

**Complexity.**
- Compress is O(n).
- Decompress is O(output length), plus the cost of building the phrase strings. To avoid copying, store `(parentIndex, char)` per entry and rebuild each phrase by walking back.

**Edge cases.** Empty string → empty list. A single char → `[(0, c)]`. Input that ends inside a known phrase → the `null` case. Always round-trip test: `Decompress(Compress(s)) == s`.

**Follow-ups.**
- **Bounded dictionary:** when it's full, either freeze it (stop adding entries) or reset it.
- **Bit packing:** encode each index with ceil(log2(dictSize)) bits.
- **LZW** is a variant that emits only indexes, by pre-seeding the dictionary with all single chars.

---

## Design an Algorithm to Elect a Restaurant (instant-runoff voting)

**Problem.** Each person ranks restaurants in order. If an option has **more than 50%** of first-choice votes, it wins. Otherwise, remove the option with the fewest first-choice votes from every ballot and repeat.

**Example (from the prompt).**
```
Ballots: [A,B,C] [A,B,C] [B,C,A] [B,A,C] [C,A,B]
Round 1 first choices: A A B B C  → A=2, B=2, C=1. Nobody has > 2.5, so eliminate C.
Round 2 first choices: A A B B A  → A=3 > 2.5, so A wins.
```

**Clarify first.**
1. Is ">50%" measured against **all ballots** or only **non-exhausted** ones (ballots that still have a live choice)? I'll assume non-exhausted.
2. **Tie for the lowest:** eliminate all of them, or break the tie somehow (fewest second-choice votes, alphabetical)? I'll eliminate all, unless that would eliminate everyone.
3. All remaining options tie: return null, or apply a tie-break rule?

**Idea.**
- Simulate it. Each round, count every ballot's top **still-alive** choice.
- Include zero-vote candidates in the tally, so a restaurant nobody ranked first gets eliminated early.

**Code.**
```csharp
public static string? Elect(List<List<string>> ballots) {
    var alive = ballots.SelectMany(b => b).ToHashSet();
    while (alive.Count > 0) {
        var tally = alive.ToDictionary(c => c, _ => 0);          // include zero-vote candidates
        int active = 0;
        foreach (var b in ballots) {
            var top = b.FirstOrDefault(alive.Contains);          // first choice still alive
            if (top != null) { tally[top]++; active++; }
        }
        var leader = tally.MaxBy(kv => kv.Value);
        if (leader.Value * 2 > active || alive.Count == 1) return leader.Key;   // strict majority

        int min = tally.Values.Min();
        var lowest = tally.Where(kv => kv.Value == min).Select(kv => kv.Key).ToList();
        if (lowest.Count == alive.Count) return null;            // everyone tied: ask for the rule
        alive.ExceptWith(lowest);
    }
    return null;
}
```

**Complexity.** For R rounds (at most C candidates), B ballots and L choices per ballot, it's O(R · B · L).

**Staff optimization.**
- Keep `Dictionary<candidate, List<ballotId>>` (who currently counts for whom) and a pointer into each ballot.
- When you eliminate X, re-route only X's ballots, advancing each pointer to the next alive choice.
- Total work becomes O(total ballot entries + C log C).
- `2 * votes > active` avoids floating point. Mention it.

---

## Help Mouse Find the Cheese / 489 Robot Room Cleaner

**Problem (mouse).** You can't see the grid. The only API is `Move(direction)` (moves and returns true if possible, otherwise false) and `IsCheese()`. There's exactly one cheese and it's reachable. Return true once the mouse is on it.

**Problem (489, the Staff version).** The robot has `Move()` (forward), `TurnLeft()`, `TurnRight()` and `Clean()`. Clean every reachable cell.

**Example.**
```
. . #
# . C        mouse starts top-left (0,0). Coordinates are RELATIVE to the start: (0,0), (0,1), (1,1), then C at (1,2)
```

**Idea.**
- Without a map, run DFS on coordinates relative to the start `(0,0)`, with a `HashSet<(int,int)>` of visited cells.
- **Backtracking is physical.** After exploring from a neighbour, move back the opposite way, so that the mouse's real position always matches the DFS frame.
- Add walls to `seen` too, so you never bump into them twice.

**Code (mouse).**
```csharp
public enum Dir { Up, Right, Down, Left }
public interface IMouse { bool Move(Dir d); bool IsCheese(); }

public class CheeseFinder {
    static readonly (Dir d, int dx, int dy)[] Moves =
        { (Dir.Up, -1, 0), (Dir.Right, 0, 1), (Dir.Down, 1, 0), (Dir.Left, 0, -1) };
    static Dir Opposite(Dir d) => (Dir)(((int)d + 2) % 4);

    readonly HashSet<(int, int)> seen = new();

    public bool Find(IMouse m) => Dfs(m, 0, 0);

    bool Dfs(IMouse m, int x, int y) {
        if (m.IsCheese()) return true;
        seen.Add((x, y));
        foreach (var (d, dx, dy) in Moves) {
            var nxt = (x + dx, y + dy);
            if (seen.Contains(nxt)) continue;
            if (!m.Move(d)) { seen.Add(nxt); continue; }          // wall: remember it
            if (Dfs(m, x + dx, y + dy)) return true;              // found it: stay there
            m.Move(Opposite(d));                                  // physically backtrack
        }
        return false;
    }
}
```

**Code (489 robot, which faces a direction).**
```csharp
static readonly (int dx, int dy)[] D = { (-1, 0), (0, 1), (1, 0), (0, -1) };   // up, right, down, left (clockwise)
readonly HashSet<(int, int)> visited = new();

public void CleanRoom(Robot robot) => Dfs(robot, 0, 0, 0);

void Dfs(Robot r, int x, int y, int dir) {
    visited.Add((x, y)); r.Clean();
    for (int i = 0; i < 4; i++) {
        int nd = (dir + i) % 4, nx = x + D[nd].dx, ny = y + D[nd].dy;
        if (!visited.Contains((nx, ny)) && r.Move()) {
            Dfs(r, nx, ny, nd);
            GoBack(r);                                            // return to (x,y), facing nd
        }
        r.TurnRight();                                            // the next direction clockwise
    }
}
void GoBack(Robot r) { r.TurnRight(); r.TurnRight(); r.Move(); r.TurnRight(); r.TurnRight(); }
```

**Complexity.** O(cells) moves. Each cell is entered once, and each edge is used twice (out and back).

**Follow-ups.**
- **Shortest path to the cheese:** first explore the whole reachable map with DFS, recording it, then run BFS on the recorded map.
- **Deep recursion on huge grids:** use an iterative DFS with a stack of `(cell, nextDirIndex)`.
- The `GoBack` sequence returns the robot to the same cell, facing the same direction as before the move. That's why the outer loop's `TurnRight` still works.

---

## Tallest Tower by Stacking Blocks (Staff, 1 report)

**Problem.** Block types are given as `(count, weight, maxSupported)`. Build the tallest tower (the most blocks) such that for every block, the **total weight of the blocks above it plus its own weight** is at most its `maxSupported`.

**Clarify.**
- I'm reading "including itself" as the load on a block being its own weight plus everything above it.
- If the load excludes itself, use `cap = maxSupported + weight` in the code below.

**Example.**
```
types: (count 1, w 3, cap 3), (count 2, w 1, cap 2), (count 1, w 5, cap 10)
Answer 3, top → bottom: [w1] load 1 ≤ 2, [w1] load 2 ≤ 2, [w5] load 7 ≤ 10.
The w3 block can't fit: it needs load ≤ 3, so it must be on top, but then the 1s below it carry 4 > 2.
```

**Idea.**
- Read the tower from the top down. The load on block i is the sum of the weights from the top through i, which is a **prefix sum**.
- That's scheduling: weight = processing time, cap = deadline, and "load ≤ cap" means "finishes by its deadline". Maximizing the number of blocks is maximizing the number of on-time jobs, which is the **Moore-Hodgson** algorithm.
  1. Sort blocks by `cap` ascending (the top of the tower has the smallest cap).
  2. Add each block and keep its weight in a max-heap.
  3. If the total exceeds the current cap, drop the **heaviest** block chosen so far.
  4. The answer is the heap size.

**Code.**
```csharp
public int TallestTower(List<(int Count, int Weight, int Cap)> types) {
    var blocks = types.SelectMany(t => Enumerable.Repeat((t.Weight, t.Cap), t.Count))
                      .OrderBy(b => b.Cap).ToList();
    var heaviest = new PriorityQueue<int, int>();                    // max-heap via negative priority
    long total = 0;
    foreach (var (w, cap) in blocks) {
        heaviest.Enqueue(w, -w); total += w;
        if (total > cap) total -= heaviest.Dequeue();                // drop the heaviest one chosen
    }
    return heaviest.Count;
}
```

**Dry run.**
- Sorted by cap: (1,2), (1,2), (3,3), (5,10).
- Add 1 (total 1 ≤ 2). Add 1 (total 2 ≤ 2). Add 3 (total 5 > 3), so drop 3 and total is back to 2. Add 5 (total 7 ≤ 10).
- The heap holds 3 blocks, so the answer is **3** ✓

**Complexity.** O(N log N), where N is the total number of blocks.

**Follow-ups.**
- **Huge counts:** keep `(weight, count)` groups in the heap. Add a whole group at once, then remove the heaviest units in bulk using `ceil(excess / w)`.
- **Proof sketch** (exchange argument): if adding a block breaks the deadline, removing the heaviest block reduces the load the most, and the remaining set is still feasible.
- If the interviewer means **height** (blocks with different heights), the problem becomes a knapsack-style DP. Ask which one they mean.

---

# Part C: graphs and union-find

## 200. Number of Islands (5 reports, reported as recently as November 2026)

**Problem.** Count the islands in a grid of '1' (land) and '0' (water), where land connects 4-directionally.

**Example.**
```
1 1 0 0 0
1 1 0 0 0
0 0 1 0 0
0 0 0 1 1      → 3
```

**Idea.** For each unvisited '1', count one island and flood-fill all of its connected land. Use an **explicit stack**: a recursive DFS on a 300×300 all-land grid recurses 90k deep and overflows C#'s 1 MB stack, and a `StackOverflowException` can't be caught.

**Code.**
```csharp
public int NumIslands(char[][] grid) {
    int rows = grid.Length, cols = grid[0].Length, count = 0;
    var dirs = new (int dr, int dc)[] { (1, 0), (-1, 0), (0, 1), (0, -1) };
    for (int r = 0; r < rows; r++)
        for (int c = 0; c < cols; c++) {
            if (grid[r][c] != '1') continue;
            count++;
            var st = new Stack<(int, int)>();
            st.Push((r, c)); grid[r][c] = '0';                       // mark when PUSHING
            while (st.Count > 0) {
                var (x, y) = st.Pop();
                foreach (var (dr, dc) in dirs) {
                    int nx = x + dr, ny = y + dc;
                    if (nx >= 0 && ny >= 0 && nx < rows && ny < cols && grid[nx][ny] == '1') {
                        grid[nx][ny] = '0'; st.Push((nx, ny));
                    }
                }
            }
        }
    return count;
}
```

**Complexity.** O(m·n) time, and O(m·n) worst-case for the stack.

**Follow-ups.**
- This mutates the grid. Use a `bool[,] visited` if that's not allowed.
- 8-directional connectivity just needs 8 entries in `dirs`.
- Land being added over time is problem 305.

## Union-find template (used by 305 and 1135)
```csharp
public class DSU {
    readonly int[] parent, rank;
    public DSU(int n) { parent = new int[n]; rank = new int[n]; for (int i = 0; i < n; i++) parent[i] = i; }

    public int Find(int x) {
        while (parent[x] != x) { parent[x] = parent[parent[x]]; x = parent[x]; }   // path halving
        return x;
    }
    public bool Union(int a, int b) {                    // returns false if a and b were already connected
        a = Find(a); b = Find(b);
        if (a == b) return false;
        if (rank[a] < rank[b]) (a, b) = (b, a);
        parent[b] = a;
        if (rank[a] == rank[b]) rank[a]++;
        return true;
    }
}
```
With both path compression and union by rank, operations are effectively O(1) amortized (O(α(n))).

## 305. Number of Islands II

**Problem.** Start with an m×n grid of all water. Each position in `positions` turns land. After each addition, return the island count.

**Example.** m = 3, n = 3, positions = [[0,0],[0,1],[1,2],[2,1]] → **[1, 1, 2, 3]**. (0,1) joins (0,0). (1,2) and (2,1) are each new islands, because diagonals don't connect.

**Idea.** For each new land cell: `count++`, then union it with each land neighbour, doing `count--` for **each successful union**. Skip duplicate additions.

**Code.**
```csharp
public IList<int> NumIslands2(int m, int n, int[][] positions) {
    var dsu = new DSU(m * n); var land = new bool[m * n];
    var res = new List<int>(); int count = 0;
    var dirs = new (int dr, int dc)[] { (1, 0), (-1, 0), (0, 1), (0, -1) };
    foreach (var p in positions) {
        int id = p[0] * n + p[1];
        if (!land[id]) {                                   // duplicate add: the count doesn't change
            land[id] = true; count++;
            foreach (var (dr, dc) in dirs) {
                int r = p[0] + dr, c = p[1] + dc;
                if (r < 0 || c < 0 || r >= m || c >= n) continue;
                int nid = r * n + c;
                if (land[nid] && dsu.Union(id, nid)) count--;
            }
        }
        res.Add(count);
    }
    return res;
}
```
**Complexity.** O(m·n + k·α), where k is the number of additions.

## 1135. Connecting Cities With Minimum Cost (MST)

**Problem.** Given n cities (1-indexed) and weighted undirected edges, return the minimum cost to connect all of them, or -1 if that's impossible.

**Examples.**
- n = 3, edges [[1,2,5],[1,3,6],[2,3,1]] → **6**. Take 2-3 (cost 1) and 1-2 (cost 5).
- n = 4, edges [[1,2,3],[3,4,4]] → **-1**.

**Idea.** Kruskal's algorithm: sort edges by cost and take each edge that joins two different components. You're done after n − 1 unions.

**Code.**
```csharp
public int MinimumCost(int n, int[][] connections) {
    Array.Sort(connections, (a, b) => a[2].CompareTo(b[2]));
    var dsu = new DSU(n + 1);                           // cities are 1-indexed
    int cost = 0, used = 0;
    foreach (var e in connections)
        if (dsu.Union(e[0], e[1])) {
            cost += e[2];
            if (++used == n - 1) return cost;
        }
    return n == 1 ? 0 : -1;
}
```
**Complexity.** O(E log E). **Alternative:** Prim's algorithm with a heap, which is better for dense graphs.

## 210. Course Schedule II

**Problem.** Given `numCourses` and pairs `[a, b]` meaning "b before a", return any valid order, or `[]` if there's a cycle.

**Example.** n = 4, pairs [[1,0],[2,0],[3,1],[3,2]] → [0,1,2,3] (or [0,2,1,3]).

**Idea.** Kahn's algorithm: compute in-degrees and start a queue with every course whose in-degree is 0. Pop a course, append it to the order, and decrement its neighbours. **If the order is shorter than n, there's a cycle.**

**Code.**
```csharp
public int[] FindOrder(int n, int[][] prereq) {
    var adj = new List<int>[n]; var indeg = new int[n];
    for (int i = 0; i < n; i++) adj[i] = new();
    foreach (var p in prereq) { adj[p[1]].Add(p[0]); indeg[p[0]]++; }
    var q = new Queue<int>();
    for (int i = 0; i < n; i++) if (indeg[i] == 0) q.Enqueue(i);
    var order = new List<int>();
    while (q.Count > 0) {
        int u = q.Dequeue(); order.Add(u);
        foreach (var v in adj[u]) if (--indeg[v] == 0) q.Enqueue(v);
    }
    return order.Count == n ? order.ToArray() : Array.Empty<int>();
}
```
**Complexity.** O(V + E).

## 277. Find the Celebrity (4 reports)

**Problem.** People are labeled 0..n−1, and `Knows(a, b)` is an API call. A celebrity is known by everyone and knows nobody. Return the celebrity, or −1. Minimize the number of API calls.

**Example.** `graph = [[1,1,0],[0,1,0],[1,1,1]]`, where `graph[a][b] = 1` means a knows b → **1**. Person 1 knows nobody else, and both 0 and 2 know person 1.

**Idea.**
- **Elimination pass:** each `Knows(cand, i)` call rules someone out. If cand knows i, cand isn't the celebrity, so i becomes the candidate. Otherwise i isn't the celebrity, because cand doesn't know them.
- After n−1 calls, one candidate is left.
- **Verify pass:** check that the candidate knows nobody and that everybody knows the candidate.

**Code.**
```csharp
public int FindCelebrity(int n) {
    int cand = 0;
    for (int i = 1; i < n; i++)
        if (Knows(cand, i)) cand = i;
    for (int i = 0; i < n; i++)
        if (i != cand && (Knows(cand, i) || !Knows(i, cand))) return -1;
    return cand;
}
```

**Dry run.** cand = 0. Knows(0,1) is true, so cand = 1. Knows(1,2) is false, so it stays 1. The verify pass succeeds, so the answer is 1 ✓

**Complexity.** At most 3(n−1) calls and O(1) space.

**Follow-up.**
- Cache the calls from the first pass, or skip `Knows(cand, i)` for i > cand in the verify pass, since the first pass already showed cand doesn't know them. That gets you close to 2n calls.

---

# Part D: dynamic programming

## 256. Paint House / 265. Paint House II (4 + 2 reports)

**Problem.** `costs[i][c]` is the cost to paint house i with color c. Adjacent houses can't share a color. Return the minimum total cost. Problem 256 has k = 3 colors; problem 265 has any k.

**Examples.**
- 256: `[[17,2,17],[16,16,5],[14,3,19]]` → **10** (2 + 5 + 3).
- 265: `[[1,5,3],[2,9,4]]` → **5** (1 + 4, or 3 + 2).

**Idea.**
- `dp[i][c] = costs[i][c] + min over c' ≠ c of dp[i−1][c']`. Done naively, that's O(n·k²).
- **Trick:** for the previous row, keep only the **lowest** cost (`min1`), **its color** (`idx1`), and the **second-lowest** cost (`min2`).
- Then for color c: if `c == idx1`, add `min2`; otherwise add `min1`. That makes it O(n·k) time and O(1) space.

**Code (265, which also solves 256).**
```csharp
public int MinCostII(int[][] costs) {
    int min1 = 0, min2 = 0, idx1 = -1;
    foreach (var row in costs) {
        int n1 = int.MaxValue, n2 = int.MaxValue, nIdx = -1;
        for (int c = 0; c < row.Length; c++) {
            int cost = row[c] + (c == idx1 ? min2 : min1);
            if (cost < n1) { n2 = n1; n1 = cost; nIdx = c; }
            else if (cost < n2) n2 = cost;
        }
        (min1, min2, idx1) = (n1, n2, nIdx);
    }
    return min1;
}
```

**Dry run (265 example).**
- Row 0: the previous row is all 0, so the costs are 1, 5, 3. That gives min1 = 1 (color 0) and min2 = 3.
- Row 1: color 0 is the same color as min1, so its cost is 2 + 3 = 5. Color 1 costs 9 + 1 = 10, and color 2 costs 4 + 1 = 5.
- min1 = **5** ✓

**Edge cases.**
- k = 1 with more than one house is impossible, and `min2` stays `int.MaxValue`, which overflows when added. Guard that case.
- No houses → 0.

## 1473. Paint House III (1 report, stretch)

**Problem.**
- `houses[i]` is 0 if the house is unpainted, otherwise its fixed color (1..n).
- Paint the unpainted houses so that there are **exactly `target` neighbourhoods** (maximal runs of the same color), at minimum cost.
- Return −1 if that's impossible.

**Example.** houses = [0,0,0,0,0], cost = [[1,10],[10,1],[10,1],[1,10],[5,1]], m = 5, n = 2, target = 3 → **9**. Paint [1,2,2,1,1]: 1+1+1+1+5.

**Idea.**
- The state is `dp[color][groups]` for the previous house.
- For house i with color c: if it's the same color as the previous house, the group count stays the same. If it's a different color, the group count goes up by 1.

**Code.**
```csharp
public int MinCost(int[] houses, int[][] cost, int m, int n, int target) {
    const int INF = int.MaxValue / 2;
    var dp = new int[n + 1, target + 1];
    for (int c = 0; c <= n; c++) for (int g = 0; g <= target; g++) dp[c, g] = INF;
    for (int c = 1; c <= n; c++)
        if (houses[0] == 0 || houses[0] == c) dp[c, 1] = houses[0] != 0 ? 0 : cost[0][c - 1];

    for (int i = 1; i < m; i++) {
        var nd = new int[n + 1, target + 1];
        for (int c = 0; c <= n; c++) for (int g = 0; g <= target; g++) nd[c, g] = INF;
        for (int c = 1; c <= n; c++) {
            if (houses[i] != 0 && houses[i] != c) continue;             // a fixed color can't change
            int paint = houses[i] != 0 ? 0 : cost[i][c - 1];
            for (int g = 1; g <= target; g++) {
                int best = dp[c, g];                                      // same color as the previous house
                for (int pc = 1; pc <= n; pc++)
                    if (pc != c) best = Math.Min(best, dp[pc, g - 1]);    // a new neighbourhood
                if (best < INF) nd[c, g] = best + paint;
            }
        }
        dp = nd;
    }
    int ans = INF;
    for (int c = 1; c <= n; c++) ans = Math.Min(ans, dp[c, target]);
    return ans >= INF ? -1 : ans;
}
```
**Complexity.** O(m · n² · target). Use the same min/second-min trick from 265 to bring it down to O(m · n · target).

## 53. Maximum Subarray (3 reports)

**Problem.** Return the maximum sum of a contiguous, non-empty subarray.

**Example.** [-2,1,-3,4,-1,2,1,-5,4] → **6**, from [4,-1,2,1].

**Idea (Kadane).** At each element, either extend the previous subarray or start fresh: `cur = max(x, cur + x)`.

**Code.**
```csharp
public int MaxSubArray(int[] nums) {
    int cur = nums[0], best = nums[0];
    for (int i = 1; i < nums.Length; i++) {
        cur = Math.Max(nums[i], cur + nums[i]);
        best = Math.Max(best, cur);
    }
    return best;
}
```

**Follow-ups.**
- **All values negative:** the answer is the largest single element, which this code handles because it starts from `nums[0]`.
- **Return the indexes too:** track `start` whenever you restart.
- **Divide and conquer (the asked follow-up):** the answer is max(best of the left half, best of the right half, best subarray crossing the middle). The crossing part is the best suffix of the left half plus the best prefix of the right half. That's O(n log n).

## 152. Maximum Product Subarray (2 reports)

**Problem.** Return the maximum product of a contiguous subarray.

**Examples.** [2,3,-2,4] → **6** ([2,3]). [-2,0,-1] → **0**.

**Idea.**
- A negative number flips the biggest product into the smallest, and the smallest into the biggest. So track both `hi` (the max product ending here) and `lo` (the min product ending here).
- On a negative x, swap hi and lo before updating.
- A zero resets both, because `max(x, ...)` restarts at x.

**Code.**
```csharp
public int MaxProduct(int[] nums) {
    int hi = nums[0], lo = nums[0], best = nums[0];
    for (int i = 1; i < nums.Length; i++) {
        int x = nums[i];
        if (x < 0) (hi, lo) = (lo, hi);
        hi = Math.Max(x, hi * x);
        lo = Math.Min(x, lo * x);
        best = Math.Max(best, hi);
    }
    return best;
}
```

**Dry run ([2,3,-2,4]).**
- Start: hi = lo = 2.
- x = 3: hi = 6, lo = 3, best = 6.
- x = −2: swap, so hi = 3 and lo = 6. Then hi = max(−2, −6) = −2 and lo = −12.
- x = 4: hi = 4. best stays **6** ✓

## 121. Best Time to Buy and Sell Stock

**Problem.** Make one buy followed by one later sell, and return the maximum profit (or 0).

**Example.** [7,1,5,3,6,4] → **5** (buy at 1, sell at 6).

**Code.**
```csharp
public int MaxProfit(int[] prices) {
    int minSoFar = int.MaxValue, best = 0;
    foreach (var p in prices) {
        minSoFar = Math.Min(minSoFar, p);
        best = Math.Max(best, p - minSoFar);
    }
    return best;
}
```

## 135. Candy

**Problem.** Every child gets at least 1 candy. A child with a higher rating than a neighbour must get more candy than that neighbour. Return the minimum total.

**Examples.** [1,0,2] → **5** (2,1,2). [1,2,2] → **4** (1,2,1).

**Idea.**
- Use two passes. Left to right: if the rating goes up, give `left + 1`. Right to left: if the rating goes up toward the left, set the count to `max(current, right + 1)`.
- Each pass enforces one neighbour rule. Taking the max keeps both satisfied.

**Code.**
```csharp
public int Candy(int[] r) {
    int n = r.Length; var c = new int[n]; Array.Fill(c, 1);
    for (int i = 1; i < n; i++) if (r[i] > r[i - 1]) c[i] = c[i - 1] + 1;
    for (int i = n - 2; i >= 0; i--) if (r[i] > r[i + 1]) c[i] = Math.Max(c[i], c[i + 1] + 1);
    return c.Sum();
}
```
**Complexity.** O(n) time and O(n) space. An O(1)-space version counts up and down slopes; mention it, but it's rarely needed.

---

# Part E: trees and heaps

```csharp
public class TreeNode { public int val; public TreeNode? left, right; }   // LeetCode shape
```

## 366. Find Leaves of Binary Tree (2 reports)

**Problem.** Repeatedly remove all the leaves and collect them per round, until the tree is empty.

**Example.**
```
      1
     / \
    2   3
   / \
  4   5          → [[4,5,3],[2],[1]]
```

**Idea.**
- A node is removed in round `height(node)`, where a leaf has height 0 and each parent is 1 + the max of its children.
- Compute heights with a post-order traversal and append each node to `res[height]`. There's no need to actually delete anything.

**Code.**
```csharp
public IList<IList<int>> FindLeaves(TreeNode root) {
    var res = new List<IList<int>>();
    int Height(TreeNode? n) {
        if (n == null) return -1;
        int h = 1 + Math.Max(Height(n.left), Height(n.right));
        if (h == res.Count) res.Add(new List<int>());
        res[h].Add(n.val);
        return h;
    }
    Height(root);
    return res;
}
```
**Complexity.** O(n). The post-order traversal guarantees that `res[h]` exists by the time you add to it.

## 272. Closest Binary Search Tree Value II (3 reports)

**Problem.** Given a BST, a real-valued target and an integer k, return the k values closest to the target.

**Example.** Tree [4,2,5,1,3], target = 3.714286, k = 2 → **[4,3]**.

**Idea.**
- **Simple, O(n):** walk the tree in order, which visits values in sorted order, and keep a sliding window of size k. When the window is full and the new value is closer to the target than the window's oldest value, drop the oldest. Otherwise, stop.
- **Staff, O(log n + k):** use two stacks. One yields the predecessors (values below the target) in descending order. The other yields the successors in ascending order. Merge them like two pointers, taking whichever is closer to the target each time.

**Code (two stacks).**
```csharp
public IList<int> ClosestKValues(TreeNode root, double target, int k) {
    var pred = new Stack<TreeNode>(); var succ = new Stack<TreeNode>();
    for (var n = root; n != null; )
        if (n.val < target) { pred.Push(n); n = n.right; }
        else                { succ.Push(n); n = n.left; }

    var res = new List<int>();
    while (res.Count < k) {
        bool takePred = succ.Count == 0 ||
                        (pred.Count > 0 && target - pred.Peek().val <= succ.Peek().val - target);
        res.Add(takePred ? NextPred(pred) : NextSucc(succ));
    }
    return res;
}
int NextPred(Stack<TreeNode> st) {          // pop, then push the rightmost path of the left subtree
    var n = st.Pop(); int v = n.val;
    for (var x = n.left; x != null; x = x.right) st.Push(x);
    return v;
}
int NextSucc(Stack<TreeNode> st) {          // pop, then push the leftmost path of the right subtree
    var n = st.Pop(); int v = n.val;
    for (var x = n.right; x != null; x = x.left) st.Push(x);
    return v;
}
```

**Dry run.**
- Initial walk: 4 ≥ 3.71 goes to succ (then go left), 2 goes to pred (then go right), 3 goes to pred (then go right). So pred = [2,3] with 3 on top, and succ = [4].
- Step 1: the predecessor gap is 0.71 and the successor gap is 0.29, so take **4**, and 5 goes onto succ.
- Step 2: 0.71 vs 1.29, so take **3**. The result is [4,3] ✓

## 156. Binary Tree Upside Down

**Problem.** Every right node either has a left sibling or is empty. Flip the tree so that the leftmost node becomes the root. Each original left child gets its original parent as its right child, and its original sibling as its left child.

**Example.**
```
    1                4
   / \              / \
  2   3     →      5   2
 / \                  / \
4   5                3   1
```

**Idea.** Walk down the left spine, rewiring as you go. At each node, its new left is the parent's old right child, and its new right is the parent.

**Code (iterative).**
```csharp
public TreeNode? UpsideDownBinaryTree(TreeNode? root) {
    TreeNode? cur = root, parent = null, parentRight = null;
    while (cur != null) {
        var left = cur.left;            // save the next spine node
        cur.left = parentRight;         // new left = the parent's old right
        parentRight = cur.right;        // save before overwriting
        cur.right = parent;             // new right = the parent
        parent = cur;
        cur = left;
    }
    return parent;                      // the old leftmost node
}
```
**Complexity.** O(h) time and O(1) space.

## 235. Lowest Common Ancestor of a BST

**Problem.** Find the deepest node that is an ancestor of both p and q. A node counts as its own ancestor.

**Example.** Tree [6,2,8,0,4,7,9,null,null,3,5]. p = 2 and q = 8 → **6**. p = 2 and q = 4 → **2**.

**Code.**
```csharp
public TreeNode LowestCommonAncestor(TreeNode root, TreeNode p, TreeNode q) {
    var n = root;
    while (true) {
        if (p.val < n.val && q.val < n.val) n = n.left!;
        else if (p.val > n.val && q.val > n.val) n = n.right!;
        else return n;                  // p and q are on different sides, or one of them is n
    }
}
```
**Complexity.** O(h). **Follow-up:** for a general binary tree (236), use a recursive approach: if both the left and right subtrees return non-null, the current node is the answer.

## 104. Maximum Depth of Binary Tree
```csharp
public int MaxDepth(TreeNode? n) => n == null ? 0 : 1 + Math.Max(MaxDepth(n.left), MaxDepth(n.right));
```
For a tree with 10^4 nodes in one chain, mention the recursion depth and offer an iterative BFS that counts levels.

## 215. Kth Largest Element in an Array (4 reports)

**Problem.** Return the k-th largest element (duplicates count).

**Examples.** [3,2,1,5,6,4] with k = 2 → **5**. [3,2,3,1,2,4,5,5,6] with k = 4 → **4**.

**Idea.**
- **Heap:** keep a min-heap of the k largest seen so far. Its top is the answer. That's O(n log k).
- **Quickselect (Staff follow-up):** partition around a random pivot and recurse only into the side that contains the answer. That's O(n) on average.
- Use a **3-way partition**, so that inputs with many duplicates don't degrade to O(n²).

**Code.**
```csharp
public int FindKthLargest(int[] nums, int k) {          // heap version
    var pq = new PriorityQueue<int, int>();
    foreach (var x in nums) { pq.Enqueue(x, x); if (pq.Count > k) pq.Dequeue(); }
    return pq.Peek();
}

public int FindKthLargestQS(int[] a, int k) {           // quickselect, 3-way partition
    int target = a.Length - k, lo = 0, hi = a.Length - 1;
    var rng = new Random();
    while (true) {
        int pivot = a[rng.Next(lo, hi + 1)];
        int lt = lo, i = lo, gt = hi;                    // [lo,lt) < pivot, [lt,gt] == pivot, (gt,hi] > pivot
        while (i <= gt) {
            if (a[i] < pivot)      { (a[lt], a[i]) = (a[i], a[lt]); lt++; i++; }
            else if (a[i] > pivot) { (a[i], a[gt]) = (a[gt], a[i]); gt--; }
            else i++;
        }
        if (target < lt) hi = lt - 1;
        else if (target > gt) lo = gt + 1;
        else return pivot;
    }
}
```

## 373. Find K Pairs with Smallest Sums (2 reports)

**Problem.** Given two sorted arrays, return the k pairs `(u, v)`, one from each array, with the smallest sums.

**Example.** nums1 = [1,7,11], nums2 = [2,4,6], k = 3 → **[[1,2],[1,4],[1,6]]**.

**Idea.**
- Think of each `i` as defining a sorted row: `nums1[i] + nums2[0..]`. That makes this a k-way merge.
- Seed the heap with `(i, 0)` for `i < k`. When you pop `(i, j)`, push `(i, j+1)`.
- Each pop is the next smallest pair overall.

**Code.**
```csharp
public IList<IList<int>> KSmallestPairs(int[] nums1, int[] nums2, int k) {
    var res = new List<IList<int>>();
    var pq = new PriorityQueue<(int i, int j), long>();
    for (int i = 0; i < Math.Min(k, nums1.Length); i++) pq.Enqueue((i, 0), (long)nums1[i] + nums2[0]);
    while (res.Count < k && pq.Count > 0) {
        var (i, j) = pq.Dequeue();
        res.Add(new List<int> { nums1[i], nums2[j] });
        if (j + 1 < nums2.Length) pq.Enqueue((i, j + 1), (long)nums1[i] + nums2[j + 1]);
    }
    return res;
}
```
**Complexity.** O(k log k). Using `long` sums avoids overflow.

---

# Part F: strings and easy problems

## 76. Minimum Window Substring

**Problem.** Find the smallest substring of s that contains every char of t, including duplicates. Return "" if there's none.

**Example.** s = "ADOBECODEBANC", t = "ABC" → **"BANC"**.

**Idea.**
- Use a sliding window. `need[c]` is how many more of c the window still needs, and `missing` is the total still needed.
- Expand to the right. When `missing == 0`, record the window and shrink from the left until it's invalid again.

**Code.**
```csharp
public string MinWindow(string s, string t) {
    var need = new int[128];
    foreach (var c in t) need[c]++;
    int missing = t.Length, left = 0, bestL = 0, bestLen = int.MaxValue;
    for (int r = 0; r < s.Length; r++) {
        if (need[s[r]]-- > 0) missing--;                 // this char was still needed
        while (missing == 0) {
            if (r - left + 1 < bestLen) { bestLen = r - left + 1; bestL = left; }
            if (++need[s[left++]] > 0) missing++;        // dropping a needed char breaks the window
        }
    }
    return bestLen == int.MaxValue ? "" : s.Substring(bestL, bestLen);
}
```
**Complexity.** O(|s| + |t|).

## 727. Minimum Window Subsequence

**Problem.** Find the shortest substring of S such that T is a **subsequence** of it (in order, not necessarily contiguous). If there's a tie, return the leftmost.

**Example.** S = "abcdebdde", T = "bde" → **"bcde"**. ("bdde" is also valid, but it comes later and has the same length.)

**Idea.**
- **Scan forward** to match all of T and find the window's end.
- Then **scan backward** from that end to match T in reverse. That finds the latest possible start, which gives the tightest window ending there.
- Record it, then restart the search at `start + 1`.

**Code.**
```csharp
public string MinWindow(string s, string t) {
    int bestStart = -1, bestLen = int.MaxValue, i = 0;
    while (i < s.Length) {
        int j = 0;
        while (i < s.Length) {                            // forward: match t
            if (s[i] == t[j] && ++j == t.Length) break;
            i++;
        }
        if (i == s.Length) break;
        int end = i;
        j = t.Length - 1;
        while (j >= 0) {                                  // backward: shrink
            if (s[i] == t[j]) j--;
            i--;
        }
        int start = i + 1;
        if (end - start + 1 < bestLen) { bestLen = end - start + 1; bestStart = start; }
        i = start + 1;                                    // look for the next window
    }
    return bestStart == -1 ? "" : s.Substring(bestStart, bestLen);
}
```

**Dry run.**
- Forward from 0 matches b(1), d(3), e(4), so the end is 4.
- Backward from 4 matches e, d, then skips c, then matches b at 1. The window is "bcde" (length 4).
- Restarting from 2 finds "bdde", which is length 4 but not shorter. The answer is "bcde" ✓

**Complexity.** O(|S|·|T|) in the worst case. The alternative is a DP: `next[i][c]` jumps to the next occurrence of c.

## 65. Valid Number (2 reports)

**Problem.**
- A valid number is an optional sign, then an integer or a decimal (`"1."`, `".5"`, `"1.5"`), then optionally `e`/`E` followed by an optional sign and an integer.

**Valid examples.** "2", "0089", "-0.1", "+3.14", "4.", "-.9", "2e10", "-90E3", "3e+7", "53.5e93"
**Invalid examples.** "abc", "1a", "1e", "e3", "99e2.5", "--6", "-+3", "95a54e53", ".", "+."

**Idea.**
- Make one pass with three flags: `digit` (seen a digit in the current part), `dot`, and `exp`.
- **A sign** is only allowed at index 0, or right after `e`.
- **An `e`** needs a digit before it, and it can appear only once. After it, reset `digit = false`, because the exponent needs digits of its own.
- **A dot** is allowed once, and only before the `e`.
- At the end, the string is valid only if `digit` is true.

**Code.**
```csharp
public bool IsNumber(string s) {
    bool digit = false, dot = false, exp = false;
    for (int i = 0; i < s.Length; i++) {
        char c = s[i];
        if (c >= '0' && c <= '9') digit = true;
        else if (c == '+' || c == '-') { if (i > 0 && s[i - 1] != 'e' && s[i - 1] != 'E') return false; }
        else if (c == 'e' || c == 'E') { if (exp || !digit) return false; exp = true; digit = false; }
        else if (c == '.') { if (dot || exp) return false; dot = true; }
        else return false;
    }
    return digit;
}
```
**Note.** Don't use `char.IsDigit`. It accepts Unicode digits such as Arabic-Indic numerals.

## 17. Letter Combinations of a Phone Number (5 reports)

**Example.** "23" → ["ad","ae","af","bd","be","bf","cd","ce","cf"]. "" → [].

**Code (backtracking).**
```csharp
static readonly string[] Map = { "", "", "abc", "def", "ghi", "jkl", "mno", "pqrs", "tuv", "wxyz" };

public IList<string> LetterCombinations(string digits) {
    var res = new List<string>();
    if (digits.Length == 0) return res;
    var sb = new StringBuilder();
    void Go(int i) {
        if (i == digits.Length) { res.Add(sb.ToString()); return; }
        foreach (var ch in Map[digits[i] - '0']) {
            sb.Append(ch); Go(i + 1); sb.Length--;      // choose, explore, un-choose
        }
    }
    Go(0);
    return res;
}
```
**Complexity.** O(4ⁿ · n).

## 20. Valid Parentheses

**Example.** "()[]{}" → true. "(]" → false. "([)]" → false. "{[]}" → true.

**Code.**
```csharp
public bool IsValid(string s) {
    var st = new Stack<char>();
    foreach (var c in s) {
        if (c == '(') st.Push(')');
        else if (c == '[') st.Push(']');
        else if (c == '{') st.Push('}');
        else if (st.Count == 0 || st.Pop() != c) return false;
    }
    return st.Count == 0;
}
```
The trick is to push the **expected closer**, which makes the comparison trivial.

## 21. Merge Two Sorted Lists

**Example.** [1,2,4] + [1,3,4] → [1,1,2,3,4,4].

**Code.**
```csharp
public ListNode? MergeTwoLists(ListNode? a, ListNode? b) {
    var dummy = new ListNode(); var tail = dummy;
    while (a != null && b != null) {
        if (a.val <= b.val) { tail.next = a; a = a.next; }
        else                { tail.next = b; b = b.next; }
        tail = tail.next;
    }
    tail.next = a ?? b;
    return dummy.next;
}
```
**Follow-up (23, merge k lists).** Use a heap of the k list heads, for O(N log k) total.

## 367. Valid Perfect Square

**Example.** 16 → true. 14 → false. Don't use `Math.Sqrt`.

**Code (binary search, using `long` to avoid overflow).**
```csharp
public bool IsPerfectSquare(int num) {
    long lo = 1, hi = num;
    while (lo <= hi) {
        long mid = lo + (hi - lo) / 2, sq = mid * mid;
        if (sq == num) return true;
        if (sq < num) lo = mid + 1; else hi = mid - 1;
    }
    return false;
}
```
**Complexity.** O(log n). Newton's method is the faster alternative: `x = (x + num / x) / 2`.

---

## C# gotchas (read last)
- `PriorityQueue<TElement, TPriority>` is a **min**-heap. For a max-heap, use the negated priority `-x`. It has **no Remove and no decrease-key**, so use lazy deletion or a `SortedSet`.
- `SortedSet<T>` **drops duplicates**, so store `(val, uniqueId)`. `Min` and `Max` are O(log n).
- Value tuples work as `Dictionary` and `HashSet` keys and compare lexicographically, with no custom comparer needed.
- `dict.TryGetValue(k, out var v)` does a single lookup. `dict.GetValueOrDefault(k)` works for counters. `list[^1]` is the last element.
- **Recursion depth:** at 10^4 to 10^5 levels, a `StackOverflowException` kills the process and can't be caught. Say "I'll go iterative".
- `int` overflow: `mid * mid`, and sums of two values near 10^9. Use `long`.
- **`string + char` in a loop is O(n²).** Use `StringBuilder`.
