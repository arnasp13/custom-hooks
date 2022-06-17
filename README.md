# Most used custom React Hooks

### useIsMounted
If I'm using any async code, I make sure to check if my component is still mounted before doing anything like updating its state.

```
export const useIsMounted = () => {
  const isMounted = useRef(false)
  useEffect(() => {
      isMounted.current = true
      return () => isMounted.current = false
  }, [])
  return isMounted
}
```

### useIsInView
For triggering animations that start when a user scrolls to an element. 
```
const useIsInView = (margin="0px") => {
  const [isIntersecting, setIntersecting] = useState(false)
  const ref = useRef()
  useEffect(() => {
    const observer = new IntersectionObserver(([ entry ]) => {
      setIntersecting(entry.isIntersecting)
    }, { rootMargin: margin })
    if (ref.current) observer.observe(ref.current)
    return () => {
      observer.unobserve(ref.current)
    }
  }, [])
  return [ref, isIntersecting]
}
```

### useHash
For keeping the **hash** of the url in-sync with a local variable. This is helpful for storing, for example, a filtered view of a chart in the url, so that a visitor can share that specific view.
```
const useHash = (initialValue=null) => {
  const [storedValue, setStoredValue] = useState(() => {
    try {
      const item = window.location.hash
      return item ? item.slice(1) : initialValue
    } catch (error) {
      console.log(error)
      return initialValue
    }
  })
  const setValue = value => {
    try {
      setStoredValue(value)
      history.pushState(null, null, `#${value}`)
    } catch (error) {
      console.log(error)
    }
  }
  return [storedValue, setValue]
}
```

### useOnKeyPress
For triggering code when the user presses a specific key. The **isDebugging** variable is optional, but I find it helpful for figuring out the exact value for a key that I need to listen for.

```
const useOnKeyPress = (targetKey, onKeyDown, onKeyUp, isDebugging=false) => {
  const [isKeyDown, setIsKeyDown] = useState(false)
  const onKeyDownLocal = useCallback(e => {
    if (isDebugging) console.log("key down", e.key, e.key != targetKey ? "- isn't triggered" : "- is triggered")
    if (e.key != targetKey) return
    setIsKeyDown(true)
    if (typeof onKeyDown != "function") return
    onKeyDown(e)
  })
  const onKeyUpLocal = useCallback(e => {
    if (isDebugging) console.log("key up", e.key, e.key != targetKey ? "- isn't triggered" : "- is triggered")
    if (e.key != targetKey) return
    setIsKeyDown(false)
    if (typeof onKeyUp != "function") return
    onKeyUp(e)
  })
  useEffect(() => {
    addEventListener('keydown', onKeyDownLocal)
    addEventListener('keyup', onKeyUpLocal)
    return () => {
      removeEventListener('keydown', onKeyDownLocal)
      removeEventListener('keyup', onKeyUpLocal)
    }
  }, [])
  return isKeyDown
}
```

### useChartDimensions
This one is especially helpful! The way `<svg>` elements scale can be tricky, as well as maintaining consistent margin widths for a chart, so this helps me keep charts responsive, and automatically updates any **dimensions** when the window is resized.

```
const combineChartDimensions = dimensions => {
  let parsedDimensions = {
      marginTop: 40,
      marginRight: 30,
      marginBottom: 40,
      marginLeft: 75,
      ...dimensions,
  }
  return {
      ...parsedDimensions,
      boundedHeight: Math.max(parsedDimensions.height - parsedDimensions.marginTop - parsedDimensions.marginBottom, 0),
      boundedWidth: Math.max(parsedDimensions.width - parsedDimensions.marginLeft - parsedDimensions.marginRight, 0),
  }
}
export const useChartDimensions = passedSettings => {
  const ref = useRef()
  const dimensions = combineChartDimensions(passedSettings)
  const [width, changeWidth] = useState(0)
  const [height, changeHeight] = useState(0)
  useEffect(() => {
    if (dimensions.width && dimensions.height) return
    const element = ref.current
    const resizeObserver = new ResizeObserver(entries => {
      if (!Array.isArray(entries)) return
      if (!entries.length) return
      const entry = entries[0]
      if (width != entry.contentRect.width) changeWidth(entry.contentRect.width)
      if (height != entry.contentRect.height) changeHeight(entry.contentRect.height)
    })
    resizeObserver.observe(element)
    return () => resizeObserver.unobserve(element)
  }, [])
  const newSettings = combineChartDimensions({
    ...dimensions,
    width: dimensions.width || width,
    height: dimensions.height || height,
  })
  return [ref, newSettings]
}
```

### [useCookie](https://github.com/reactivestack/cookies/tree/master/packages/react-cookie/)
This seriously make getting & setting cookies a breeze. The only issue, at the moment, is that the hook value doesn't update when you **set**  it -- but this should be updated in the future.

### [useInterval](https://overreacted.io/making-setinterval-declarative-with-react-hooks/)
Especially useful for animations, or anything that needs to loop. Dan Abramov's blog post on this (linked) is especially great for understanding some gotchas with creating custom hooks.


### [useLocalStorage](https://usehooks.com/useLocalStorage/)
For keeping a localStorage value in-sync with a local variable.