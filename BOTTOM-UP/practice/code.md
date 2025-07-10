---
page:
  headHtml: |
    <snippet var="js.highlightjs" />

---

> [!warning] 
> Bear in mind that when using highlight.js you must manually add language support. The above snippet includes Haskell and [Nix](https://nixos.asia) by default; otherwise, it is normally added as:
>
> ```yaml
> page:
>   headHtml: |
>     <snippet var="js.highlightjs" />
>     <with var="js">
>     <script src="https://cdnjs.cloudflare.com/ajax/libs/highlight.js/${value:highlightjs-ver}/languages/haskell.min.js"></script>
>     </with>
> ```

## Example (highlight.js)

### Python

```python
def fib(n):
    a, b = 0, 1
    while a < n:
        print(a, end=' ')
        a, b = b, a+b
    print()
fib(1000)
```

### Haskell

```haskell
fib 0 = 0
fib 1 = 1
fib n = fib (n-1) + fib (n-2)
```