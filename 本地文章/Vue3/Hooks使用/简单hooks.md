```ts
export const useTotal = () => {
  const total = ref(0)
  const setTotal = (num: number) => {
    total.value = num
  } 

  return { total, setTotal }
}

const { total, setTotal } = useTotal()
```

