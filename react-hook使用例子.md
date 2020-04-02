```javascript
  import React, {
    PureComponent,
    useState,
    useEffect,
    useRef,
    useDebugValue,
    useCallback,
    createContext,
    useContext,
    useMemo,
    useReducer,
    useImperativeHandle,
    forwardRef
} from 'react';
import cx from 'classnames';

import S from './index.m.less';

const HookCn = createContext(null);

const initialState = { disCount: 0 };

function reducer(state, action) {
    switch (action.type) {
        case 'increment':
            return { disCount: state.disCount + 1 };
        case 'decrement':
            return { disCount: state.disCount - 1 };
        default:
            throw new Error();
    }
}

function usePrevious(value) {
    const ref = useRef();
    useEffect(() => {
        ref.current = value;
    });
    return ref.current;
}

function MeasureExample(props, ref) {
    const [height, setHeight] = useState(0);
    const { count, dispatch } = useContext(HookCn);

    const measuredRef = useCallback(node => {
        if (node !== null) {
            console.log(6666666);
            setHeight(node.getBoundingClientRect().height);
        }
    }, []);

    useEffect(() => {
        console.log(height);
        ref.current.height = height;
    });

    useImperativeHandle(ref, () => ({
        focus: () => {
            console.log(ref);
            console.log(1234567);
        },
    }));

    // 改变content
    function changeCn() {
        dispatch({ type: 'increment' });
    }

    return (
        <div>
            <h1 ref={measuredRef}>Hello, world</h1>
            <h2>
                The above header is
                {' '}
                {Math.round(height)}
                px tall
            </h2>
            <button className={cx(S.button, S.dispatch)} onClick={changeCn}>
                点击发送dispatch按钮
            </button>
            <h2>
                从父组件传递下来的count：
                {count}
            </h2>
        </div>
    );
}

function HookLink(props) {
    const [count, setCount] = useState(0);
    const [val, setValue] = useState('');
    const inputEl = useRef(null);
    const testRef = useRef('');
    const MeasureExampleF = forwardRef(MeasureExample);
    const [state, dispatch] = useReducer(reducer, initialState);
    useDebugValue(count ? 'Online' : 'Offline');

    const prevCount1 = usePrevious(count);

    useEffect(() => {
        document.title = `You clicked ${count} times`;
        inputEl.current.focus();
    });

    useEffect(() => {
        testRef.current.focus();
        console.log(testRef.current.height);
        console.log('*******************');
        console.log(testRef.current);
        console.log('*******************');
    });

    const expensive = useMemo(() => {
        console.log('compute');
        let sum = 0;
        for (let i = 0; i < count * 100; i++) {
            sum += i;
        }
        return sum;
    }, [count]);

    return (
        <div>
            <HookCn.Provider value={{ count, state, dispatch }}>
                <div className={S.button} onClick={() => setCount(props.initialCount)}>
                    Reset
                </div>
                <div className={S.button} onClick={() => setCount(prevCount => prevCount + 1)}>
                    +添加
                </div>
                <div className={S.button} onClick={() => setCount(prevCount => prevCount - 1)}>
                    -减少
                </div>
                <h2>
                    {count}
                    -
                    {expensive}
                    -
                    {val}
                </h2>
                <input className={S.input} ref={inputEl} type="text" onChange={event => setValue(event.target.value)} />
                <div>
                    <span className={S.spanCount}>
                        上一个count值：
                        <i>{prevCount1}</i>
                    </span>
                    {' '}
                    <span className={S.spanCount}>
                        当前的值：
                        <i>{count}</i>
                    </span>
                </div>
                <MeasureExampleF ref={testRef} />
                <h1>
                    dispatch的值:
                    {state.disCount}
                </h1>
            </HookCn.Provider>
            {/* <NoContent /> */}
        </div>
    );
}

function Parent() {
    const [count, setCount] = useState(1);
    const [val, setVal] = useState('');

    // useEffect(() => {
    //     // setCount(prevCount => prevCount + 1);
    //     const id = setTimeout(() => {
    //         setCount(prevCount => prevCount + 1);
    //         console.log(count);
    //     }, 1000);
    //     if (count > 5) {
    //         window.clearTimeout(id);
    //     }
    // });

    useEffect(() => {
        const id1 = setInterval(() => {
            setCount(prevCount => {
                console.log(prevCount);
                if (prevCount > 2) {
                    console.log(99999);
                    window.clearInterval(id1);
                }
                return prevCount + 1;
            });
        }, 1000);
    }, []);

    const callback = useCallback(() => count, [count]);
    return (
        <div>
            <h4>
                {count}
                -
                {val}
            </h4>
            <Child callback={callback} />
            <div>
                <button className={S.button} onClick={() => setCount(count + 1)}>
                    +
                </button>
                <input className={S.input} value={val} onChange={event => setVal(event.target.value)} />
            </div>
        </div>
    );
}

function Child({ callback }) {
    const [count, setCount] = useState(() => callback());
    useEffect(() => {
        console.log('childNew');
        setCount(callback());
    }, [callback]);

    return (
        <div>
            <h2>
                childValue:
                {count}
            </h2>
        </div>
    );
}

export default class Cheap extends PureComponent {
    componentDidMount() {}

    a = false;

    render() {
        return (
            <div>
                <HookLink hook="我是hook" initialCount={10} />
                <br />
                <p className={S.fenge} />
                <Parent />
                <br />
            </div>
        );
    }
}


```
