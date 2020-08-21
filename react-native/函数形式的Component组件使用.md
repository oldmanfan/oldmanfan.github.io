# 函数形式的Componet使用

class和function两种形式的Component都是RN中常见的组件方式, 在一些无状态的组件中, function形式的component用的更多一些(并不是说不能有状态).

## 1. class和function的两种component的定义方式

* class方式

```typescript
interface MyComponentProps {
  pA: string;
  pB: number;
}

interface MyComponentStates {
  sA: string;
  sB: number;
}

class MyComponent extends React.Component<
	MyComponentProps,
	MyComponentStates
> {
  // 正常逻辑代码
}
```

示例代码中定义的`MyComponentProps`和`MyComponentStates`分别定义了MyComponent的Props和States.

* function方式

```typescript
interface MyComponentProps {
  pA: string;
  pB: number;
}

const MyComponent: React.FC<MyComponentProps> = ({pA, pB}) => {
  
  // 其它逻辑
  const [stateA, setStateA] = React.useState('MMMMM');
  
  return (
  	<View>
    	// ...
    </View>
  );
}
```

## 2. 在function方式中使用状态及生命周期方法

### React.useState

`React.useState`方法用来定义一个状态参数, 如上面例子中的`const [stateA, setStateA] = React.useState('MMMMM');`就定义了一个`string`类型的状态`stateA`, 它等同于class方式中的`this.state = {stateA: ‘MMMM’}`; 而`setStateA`是一个方法, 用来设置`stateA`的值, 例如: `setStateA(‘XXX’)`的作用和class中的`this.setState({stateA: ‘XXX’})`的作用是一样的.

### React.useEffect

关于`useEffect`的详细说明, 可以参考[这个文档](https://zh-hans.reactjs.org/docs/hooks-effect.html).

总结来说, `useEffect`类似于组件生命周期中的`componentDidMount`，`componentDidUpdate` 和 `componentWillUnmount` 这三个函数的组合.  使用useEffect, 就等同于在生命周期中执行对应的操作. 这非常有助于一些需要进行设置和移除的操作, 比如设置和移除EventListener.

```typescript
const MyComponent: React.FC<MyComponentProps> = ({pA, pB}) => {
  
  // 其它逻辑
  const [stateA, setStateA] = React.useState('MMMMM');
  
  const [stateB, setStateB] = React.useState('NNNN');
  
  React.useEffect(()=> {
    // ComponentDidMount时, 会执行这里的操作
    setStateA('stateA set when Component mounted');
    
    // 返回的函数, 将在ComponentWillUnmount时执行.
    return () => {
      setStateB('stateA cleared when Component will unmount.');
    }
  });
  
  // 可以使用多个useEffect, 来分离代码, 以使得逻辑更清晰
  React.useEffect(() => {
    setStateB('stateB set when Component mounted');
    
    return () => {
      setStateB('stateB cleared when Component will unmount.');
    }
  });
  return (
  	<View>
    	// ...
    </View>
  );
}
```

在上面的示例中, useEffect()方法中return了另一个方法, 以便component unmount时执行里面的相关操作. 如果不需要在umount时做动作, 则useEffect无需return.

可以使用多个useEffect来分离操作, 以使得代码逻辑更清晰.

### React.useRef

`useRef`用来引用DOM中的某个element, 和class中的`ref`属性作用类似.

```typescrit
// class方式的ref
<Button
	ref={(r) => {this.button = r}}
	...
	/>
```

```typescript
// funciton方式的ref
const buttonRef = useRef();

//...

useEffect(() => {
  // 使用这个ref来做一些事情
  buttonRef.current?.title = 'AAAAA'
})

<Button
	ref={buttonRef}
...
/>
```

## 3. 更多的Hook方法

更多关于React中的Hook方法, 参考[React的官方文档](https://zh-hans.reactjs.org/docs/hooks-intro.html).