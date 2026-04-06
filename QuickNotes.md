Why do we need useMemo and useCallback ?

- The answer is simple - memoization between re-renders.
- useCallback prevents unwanted re-triggering.
- ref : https://www.developerway.com/posts/how-to-use-memo-use-callback

useEffect and useCallback differnce

- useCallback = control when function changes
- useEffect = react to that change
