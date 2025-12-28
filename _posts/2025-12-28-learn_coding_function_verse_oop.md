在研究zustand里创建create的方式，他是基于函数闭包，高级函数等技巧构建了一个函数可以接受另一个函数作为参数，然后这个作为参数被接受的函数甚至也可以带另一个函数作为参数去对最初构建的函数里的数据进行操作。突然发现这和python里面向对象好像--有用函数作为参数做初始化（利用闭包和高级函数），又有自己内部保存的数据（利用闭包），然后又有api可以调用。问了下几个AI，其中Gemini提到了一个非常有意思的画，在编程界有一句名言：“闭包是穷人的类，类是穷人的闭包。”（Closures are a poor man's objects, and vice versa.）
我刚开始不太理解，我理解闭包是穷人的类是说有了闭包，就可以构建一些像简单的类一样的函数了。那类是穷人的闭包又怎么说？Gemini意思是前半句像我所说的，后半句的情况是以前像早期java或者c++没有闭包概念的语言，只能创建一个对象来达到闭包的效果。然后总的说来，用闭包函数式编程；或者用类面向对象式编程各有千秋，闭包是方便快捷我不用投入额外资源去构建一个类，但是类的好处是逻辑清晰数据实实在在的的抓取而不是藏在堆上只能找到他的引用，另外好理解心智负担也低些，但是当然资源占用和代码行数可能相对会比闭包相对多些。另外通过面向对象去理解学习闭包是一个有效的方法。

我觉得比较一下不同编程语言的特点也挺有意思。还有考古一下不同编程语言的历史等等。


function create(createState) {
  // 闭包：存储状态和监听器
  let state
  const listeners = new Set()
  
  // 核心方法：设置状态并通知监听器
  const setState = (partial, replace) => {
    const nextState = typeof partial === 'function'
      ? partial(state)
      : partial
    
    if (!Object.is(nextState, state)) {
      const previousState = state
      
      // replace 为 true 时完全替换，否则合并
      state = replace ?? typeof nextState !== 'object'
        ? nextState
        : Object.assign({}, state, nextState)
      
      // 通知所有监听器
      listeners.forEach(listener => listener(state, previousState))
    }
  }
  
  // 获取当前状态
  const getState = () => state
  
  // 订阅状态变化
  const subscribe = (listener) => {
    listeners.add(listener)
    // 返回取消订阅函数
    return () => listeners.delete(listener)
  }
  
  // 销毁 store
  const destroy = () => listeners.clear()
  
  // 初始化状态
  const api = { setState, getState, subscribe, destroy }
  state = createState(setState, getState, api)
  
  return api
}