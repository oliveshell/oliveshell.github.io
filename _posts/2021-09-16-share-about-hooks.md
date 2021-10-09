---
layout: post
title: share about react hooks
categories: React
description: something about the best practice of react hooks.
keywords: react hook
---

Why Use React Hooks instead of Class?

# Share About React Hooks
## Agenda
1. Why Use React Hooks instead of Class
2. TypeScript experience
3. ~~Rxjs related~~
## Why use React Hooks?
### What are react hooks?
According to the official documentation, Hooks brings into a functional component all the powers previously accessible in class components and made them available in functional components. With React Hooks, you can now use state and other features of React outside of the construct of a class:
```javascript
import React, { useState } from 'react';
function Example() {
  // Declare a new state variable, which we'll call "count"
const [count, setCount] = useState(0);
  return (
    <div>
      <p>You clicked {count} times</p>
      <button onClick={() => setCount(count + 1)}>
        Click me
      </button>
    </div>
  );
}
```
* workflow 

![image info](/images/posts/react/hook-workflow.png)
- Dispatcher 
    The dispatcher is the shared object that contains the hook functions. It will be dynamically allocated or cleaned up based on the rendering phase of ReactDOM, and it will ensure that the user doesn’t access hooks outside a React component
- The hooks queue
- fiber
    upgrade from stack reconciler to queue reconciler, import the time slice. Impl: requestIdleCallback/requestAnimationCallback.
    Each render cycle, there are two workInProgressTree and nextWorkingProgressTree, realise the ping-pong operation.  
* life of a frame 

![image info](/images/posts/react/life-a-frame.png)

Accordingly, we need to rethink the way we view the a component’s state. So far we have thought about it as if it’s a plain object:
```javascript
{
  aa: 'a',
  bb: 'b',
  cc: 'c',
}
```
But when dealing with hooks it should be viewed as a queue, where each node represents a single model of the state and uses the linked-list structure:
```javascript
{
  memoizedState: 'a',
  next: {
    memoizedState: 'b',
    next: {
      memoizedState: 'c',
      next: null
    }
  }
}
```
**src**: ReactFiberHooks.new.js @828 @updateReducer()

### Advantages
With the introduction of Hooks, you can now do everything you would normally do in a class, inside a functional component. This is a game-changer because you have fewer concepts to learn and fewer lines of code to write too. 
#### Detailed Explanation:
* Lifecycle methods in functional components
    know that you cannot really escape using lifecycle methods if you use class components, componentDidMount is one of the most popular ones.
    useEffect
* common used
```javascript
const [count, changeCount] = useState(0);
    // trigger when the count is changed.
useEffect(() => {
    message.info(`the newest count: ${count}`);
}, [count])
```
* mount/unmount
```javascript
useEffect(() => {
    message.info('componentWillMount');
    return () => {
      message.info('componentWillUnmount');
    };
  }, []);
```
* componentDidUpdate
```javascript
useEffect(() => {
    // logic code
    message.info(`componentDidUpdate, count: ${count}`);
  });
```
* code refine with
   useCallback/useMemo
   **src**: ReactFiberHooks.new.js @1661 diff with useEffect@1428
```javascript
const [count1, changeCount1] = useState(0);
const [count2, changeCount2] = useState(10);

const calculateCount = useCallback(() => {
  if (count1 && count2) {
    return count1 * count2;
  }
  return count1 + count2;
}, [count1, count2])

useEffect(() => {
    const result = calculateCount(count, count2);
    message.info(`${result}`);
}, [calculateCount])

```
- Detailed Example With Async Request
```javascript
    useEffect(() => {
        const fetchData = async () => {
            const result =  await axios('https://google.com');
            setData(result.data);
        }
        fetchData();
    },[]);
    // better
    useEffect(() => {
        newClassService.fetchData(); // custom hook.
  }, []);
  // using useReducer to opt

const fetchDataReducer = (state, action) => {
    switch(action.type){
        case 'FETCH_INIT':
            return{
                ...state,
                isLoading: true,
                isError: false
            }
        case 'FETCH_SUCCESS':
            return {
                ...state,
                isLoading: false,
                isError: false,
                data: action.payload,
            }
        case 'FETCH_ERROR':
            return {
                ...state,
                isLoading: false,
                isError: false,
                data: action.payload,
            }
            break;
        default:
            return state;
    }
}
// custom hook
const useDataApi = (initUrl, initData) => {
    const [url, setUrl] = useState(initUrl);
    const [state, dispatch] = useReducer(fetchDataReducer,{
        data: initData,
        isLoading: false,
        isError: false
    })
    useEffect(() => {
        const fetchData = async () => {
            dispatch({type: 'FETCH_INIT'})
            try{
                const result =  await axios(url);
                dispatch({type: 'FETCH_SUCCESS', payload: result.data})
            }catch(error){
                dispatch({type: 'FETCH_ERROR'})
            }
        }
        fetchData();
    },[url]);

    return [state, setUrl];
}


const demoHooks = () => {
    const [search, setSearch] = useState('react')
    const [{data, isLoading,isError}, fetchData ] = useDataApi(
        'google.com',
        {hits: []});
    _renderItem = ({item}) => {
        return(
            <View>
            </View>
        )
    };
    _search = () => {
        fetchData(`url`)
    }
    return (
        <div >
            isLoading && <Loading>
            isError && <Error>
            <Normal>
        </div>
    );
};
```


#### State management in functional components
    useState

#### Fewer lines of code
#### Code Reused
    Currently we can use the HOC / Render Props to reuse the similar logical code. In this way easily lead to the wrapper hell and sometimes cannot be easily debugged. With the hooks, we can define the custom hooks to reuse the code.

## conclusion
| hook | usage | desc |
| :-----| ----: | :----: |
| useState | const [state, changeState] = useState(initialValue) |  |
| useEffect | useEffect(fn, [...relativeState]) |  |
| useCallback | useCallback(fn, [...relativeState]) | cache function |
| useMemo | useMemo(fn, [...relativeState]) | cache result,e.g. useCallback(fn, deps) equals useMemo(() => fn, deps) |
| useRef | const newRef = useRef(initialValue) |  |
| useReducer | useReducer(fn, initArg |  |
| useContext | const newRef = useRef(initialValue) |  |

## use Redux with Hooks
React Redux now includes its own useSelector and useDispatch Hooks that can be used instead of connect.

* useSelector is analogous to connect’s mapStateToProps. You pass it a function that takes the Redux store state and returns the pieces of state you’re interested in.

* useDispatch replaces connect’s mapDispatchToProps but is lighter weight. All it does is return your store’s dispatch method so you can manually dispatch actions. As binding action creators can be a little confusing sometimes.

Alright, so now let’s convert a React component that formerly used connect into one that uses Hooks.

Using **connect**:
```javascript
import React from "react";
import { connect } from "react-redux";
import { addCount } from "./store/counter/actions";

export const Count = ({ count, addCount }) => {
  return (
    <main>
      <div>Count: {count}</div>
      <button onClick={addCount}>Add to count</button>
    </main>
  );
};
const mapStateToProps = state => ({
  count: state.counter.count
});
const mapDispatchToProps = { addCount };
export default connect(mapStateToProps, mapDispatchToProps)(Count);
```
Now, with the new React Redux Hooks instead of connect:   
```javascript
import React from "react";
import { useDispatch, useSelector } from "react-redux";
import { addCount } from "./store/counter/actions";
export const Count = () => {
  const count = useSelector(state => state.counter.count);
  const dispatch = useDispatch();
  return (
    <main>
      <div>Count: {count}</div>
      <button onClick={() => dispatch(addCount())}>Add to count</button>
    </main>
  );
};
``` 
* example:
   HomeScreenConfigMainView.tsx
While the Redux Hooks have many benefits, there is one benefit of using connect instead of the Redux Hooks, and that is that it keeps your component decoupled from Redux.  Maybe it can also benefit the test case.
React Hooks are a useful new feature, and React Redux’s addition of Redux-specific hooks is a great step toward simplifying Redux development.  
Limited by ability, unable to study further. Let's dived into together.
## discussion
*  Need to cancel some request action in the specified condition. e.g. big file upload.
*  Ignore the impact of the multi-request response when one of encounter slowest network.


