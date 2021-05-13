Không nên dùng UnityEvent, lý do:

https://www.jacksondunstan.com/articles/3335

```
C# events beat UnityEvent for each number of arguments. The gap widens as more args are dispatched to the listeners. In the best case, UnityEvent takes 2.25x longer than C# events but in the worst case it takes almost 40x longer!
```