紧接上文，和AI聊了半天react里的useState和useEffect，今早又看到一处奇怪的代码。主要是useTimerStore里面传了一个函数state => state.syncTime和state => state.isRunning让我大为费解，追着Gemini聊了一天，竟然聊到了设计模式。

首先为什么要传函数大致解释如下：这里useTimerStore对象需要一个“地图”去追踪这个对应的变量，当变量变化时才可以触发后面对应的useEffect。如果传一个state.syncTime则仅仅是一个写死的数据则没有效果（突然想到传一个地址指针是不是也可以？但是如何保证这个地址始终指向对应的数据？）

然后怎么联系上设计模式的，Gemini说这种写法是观察者模式里的常用手段。具体做法是用subscribe维护一个数组[]里面填充selector函数，用new调用subscribe保证每次新建某个相关对象时他的selector都会被subscribed，最后用update调用notify遍历subscribe的数组保证每次更新都会notify相关的对象，在notify里通过执行selector函数就可以方便的找到对应的变量。

再然后Gemini又给提了好几个相关的设计模式：除了观察者模式，还有控制反转/依赖注入，中间件/洋葱模式，代理模式，策略模式。
其中中间件Gemini给了个有关zustand并涉及了函数闭包的复杂例子。明天再详细探讨。



  const syncTime = useTimerStore(state => state.syncTime);
  const isRunning = useTimerStore(state => state.isRunning);

export const useTimerStore = create((set, get) => ({
  // ===== 状态 =====
  durationSeconds: 25 * 60, // 默认25分钟
  seconds: 25 * 60,
  isRunning: false,

  startTimestamp: null,
  endTimestamp: null,

...

  // ===== 同步时间（前台刷新）=====
  syncTime: () => {
    const { isRunning, endTimestamp } = get();
    if (!isRunning || !endTimestamp) return;

    const remaining = Math.max(
      0,
      Math.floor((endTimestamp - Date.now()) / 1000)
    );

    set({
      seconds: remaining,
      isRunning: remaining > 0,
    });
  },
}));

