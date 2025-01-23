> [JavaScript中Class的this指向前言 JavaScript中的this指向问题本来是一个入门必会的问题，但 - 掘金](https://juejin.cn/post/6966860151311564836#heading-6)



1. **`this`的绑定优先级**
   
   - **new创建实例**：`this`指向新创建的实例。
     
     ```javascript
     class Cat {
       jump() { console.log('jump', this) }
     }
     const cat = new Cat();
     cat.jump(); // jump Cat {}
     ```
   
   - **显式绑定**：使用`call`、`apply`、`bind`方法。
     
     ```javascript
     function jump() { console.log(this.name) }
     const obj = { name: '豆芽', jump };
     jump = jump.bind(obj);
     jump(); // 豆芽
     ```
   
   - **对象中的方法绑定**：方法作为对象的一部分调用。
     
     ```javascript
     function jump() { console.log(this.name) }
     const obj = { name: '豆芽', jump };
     obj.jump(); // 豆芽
     ```
   
   - **默认绑定**：在严格模式下`this`是`undefined`，否则是全局对象。

2. **Class中属性与方法的绑定**
   
   - **实例方法**：挂载在实例的原型对象上。
     
     ```javascript
     class Cat {
       constructor(name) { this.name = name }
       jump() { console.log('jump', this) }
     }
     let cat = new Cat('豆芽');
     cat.jump(); // jump Cat { name: '豆芽' }
     ```
   
   - **静态方法**：不挂载在原型对象上，直接通过类调用。
     
     ```javascript
     class Cat {
       static go() { console.log(this) }
     }
     Cat.go(); // class Cat {}
     ```
   
   - **原型链上的方法**：所有实例共享。
     
     ```javascript
     class Cat {
       constructor(name) { this.name = name }
     }
     Cat.prototype.eat = function() { console.log('eat', this) }
     let cat = new Cat('豆芽');
     cat.eat(); // eat Cat { name: '豆芽' }
     ```
   
   - **箭头函数**：`this`在定义时绑定，继承自父级作用域。
     
     ```javascript
     class Cat {
       constructor(name) { this.name = name; this.jump = () => console.log('jump', this) }
     }
     let cat = new Cat('豆芽');
     cat.jump(); // jump Cat { name: '豆芽', jump: [Function] }
     ```

3. **Class中`this`的绑定**
   
   - **实例方法的`this`**：指向调用该方法的实例。
     
     ```javascript
     class Cat {
       constructor(name) { this.name = name }
       run() { console.log('run', this) }
     }
     let cat = new Cat('豆芽');
     cat.run(); // run Cat { name: '豆芽' }
     ```
   
   - **方法赋值后的`this`**：上下文变为全局对象（严格模式下为`undefined`）。
     
     ```javascript
     let run = cat.run;
     run(); // run undefined
     ```
   
   - **静态方法的`this`**：指向类本身。
     
     ```javascript
     class Cat {
       static go() { console.log(this) }
     }
     Cat.go(); // class Cat {}
     ```
   
   - **箭头函数的`this`**：继承自定义时的上下文。
     
     ```javascript
     class Cat {
       constructor(name) { this.jump = () => console.log('jump', this) }
     }
     let cat = new Cat('豆芽');
     cat.jump(); // jump Cat { name: '豆芽', jump: [Function] }
     ```

4. **注意点**
   
   - **严格模式**：类和模块内部默认是严格模式。
   - **`this`的指向**：类方法内部的`this`默认指向类的实例，但单独使用时需小心。
   - **静态方法**：不会被实例继承，直接通过类调用，`this`指向类本身。

通过这些内容，读者可以更好地理解JavaScript中`this`关键字在不同场景下的行为，特别是在类中的表现。
