# You don’t know getters

In class, using `getter` gives us a way to evaluate a property in runtime. There are some limitations that are interesting to know.

- getters are like functions, which are not enumerable. If we use getters to define properties, it’s not shown if we do Object.keys(), and will be lost when we do stringify.
    
    ```tsx
    class Test {
        get name() {
            return 'alice';
        }
    }
    const testObj = new Test();
    console.log({
        testObj,
        testObj_name: testObj.name,
        serialized_testObj_name: JSON.parse(JSON.stringify(testObj)).name,
    });
    ```
    
    `serialized_testObj_name` will be undefined.
    
    
- To work around this, we need to redefined as **enumerable properties** on the instance, which override the ones on the prototype.
    
    ```tsx
    class SerialisableTest {
        constructor() {
            const propertyDescriptors = Object.getOwnPropertyDescriptors(
                Object.getPrototypeOf(this)
            );
            Object.entries(propertyDescriptors).forEach(([property, descriptor]) => {
                // Methods do not need to be overridden on the instance.
                if (typeof this[property] === 'function') {
                    return;
                }
                Object.defineProperty(this, property, {
                    ...descriptor,
                    enumerable: true,
                });
            });
        }
        get name() {
            return 'alice';
        }
    }
    const serialisableTestObj = new SerialisableTest();
    console.log({
    		keys: Object.keys(serialisableTestObj),    
    		testObj_name: serialisableTestObj.name,
        serialized_TestObj_name: JSON.parse(JSON.stringify(serialisableTestObj)).name,
    });
    ```
    
- References:
    - runkit for try out [https://runkit.com/xiaosha/62a3566144f4850007863558](https://runkit.com/xiaosha/62a3566144f4850007863558)