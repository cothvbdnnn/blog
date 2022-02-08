``` javascript
const sum = (a, b) => {
  console.log('start');
  const start = new Date();
  while (new Date() - start < 3000);
  console.log('end', a + b);
  return a + b;
}
```

```
start
// sau 3s
end 3
```

``` javascript
const number = useMemo(() => {
  return sum(a, b)
}, [])
```

``` javascript
const memoize = func => {
  const result = {}
  return (...params) => {
    const paramKey = JSON.stringify(params)
    if (!result[paramKey]) {
      result[paramKey] = func(...params)
    }
    return result[paramKey]
  }
}
```
``` javascript
const sum = memoize((a, b) => {
  console.log('start');
  const start = new Date();
  while (new Date() - start < 3000);
  console.log('end', a + b);
  return a + b;
})

```

```
{
  [1,2]: 3,
  [2,3]: 5
}
```