原问题：[Storing Objects in HTML5 localStorage](http://stackoverflow.com/questions/2010892/storing-objects-in-html5-localstorage)

提问者是问如何使用HTML5的`localStorage`来存储一个JavaScript对象问题。他碰到的问题是，除了primitive的对象之外，自定义的JavaScript对象存储到localStorage中之后，会被转换成字符串，无法再使用json对象方法来读取对象属性。代码示例：

```
var testObject = { 'one': 1, 'two': 2, 'three': 3 };
console.log('typeof testObject: ' + typeof testObject); // typeof testObject: object
// testObject properties:
//  one: 1
//  two: 2
//  three: 3
console.log('testObject properties:');
for (var prop in testObject) {
    console.log('  ' + prop + ': ' + testObject[prop]);
}

// Put the object into storage
localStorage.setItem('testObject', testObject);

// Retrieve the object from storage
var retrievedObject = localStorage.getItem('testObject');

console.log('typeof retrievedObject: ' + typeof retrievedObject);   // typeof retrievedObject: string
console.log('Value of retrievedObject: ' + retrievedObject);    // Value of retrievedObject: [object Object]
```
他在Safari, Chrome和Firefox试过，都是一样的结果。不知道这是[HTML5 Web Storage](http://www.w3.org/TR/webstorage/)规范的不足还是浏览器实现的缺陷。

##序列化
对象的存储，就是序列化问题。

对象一般是指一个结构化的数据。JavaScript中的对象是一个JSON实例：
```
obj {
    "key1": "value1",
    "key2": "value2"
}
```
**结构化**的意思是指数据是有从属/依存关系的，常见的结构有链、树、图。比如上面这个对象就是个树状结构。基于这些结构之上，可以定义一些方法，便于数据的存取。

存储之后的数据本身是二进制的，没有结构，数据本身与前后没有关联，是一串连续数据。

因此，如果想存储一个对象。需要：
* 在存储之前把对象按某种方式变成一串连续数据——序列化
* 取这个数据之后再逆向恢复成原始的对象结构——反序列化

###Java对象序列化
最早接触序列化是在Java中。

比如写一个贪吃蛇的游戏，暂停/退出之前需要保存当前游戏的进度，这时就需要做序列化，把游戏中各个状态值保存到物理文件。然后在下次启动游戏的时候，读取已保存的这些状态值。

对于一个需要被序列化的对象，其对应的类在定义时需要继承Serializable接口。另外：

* 需要给类声明一个`private static final long serialVersionUID`
* 需要给所有需要序列化的属性设置getter()/setter()方法
* 对于不需要序列化的属性，用`transient`修饰
* 可以重写`writeObject()`与`readObject()`方法来自定义默认序列化之外的动作

其他的交给Java来完成就行。

```
 public class Test implements Serializable {
    private static final long serialVersionUID = 1L;
    private String password = "pass";
    public String getPassword() {
        return password;
    }
    public void setPassword(String password) {
        this.password = password;
    }
    public Test(String pass) {
        this.password = pass;
    }
    public static void main(String[] args) {
        try {
            File file = new File("test.out");

            ObjectOutputStream oout = new ObjectOutputStream(new FileOutputStream(file));
            Test test = new Test("new pass");
            oout.writeObject(test);
            oout.close();

            ObjectInputStream oin = new ObjectInputStream(new FileInputStream(file));
            Test newTest = (Test) oin.readObject();
            System.out.println("password: " + newTest.getPassword());   // output: new pass
            oin.close();
        System.out.println(newTest);
        } catch (FileNotFoundException e) {
            e.printStackTrace();
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```
##JSON对象序列化
其实HTML5 Web Storage中的规范没有问题，浏览器的实现也没有问题。问题就是提问者没有把JavaScript进行序列化/反序列化。

JavaScript对象的序列化很简单，使用JSON的`stringify() && parse()`接口来进行序列化/反序列化。

```
var testObject = { 'one': 1, 'two': 2, 'three': 3 };

// 序列化
localStorage.setItem('testObject', JSON.stringify(testObject));


var retrievedObject = localStorage.getItem('testObject');
// 反序列化
console.log('retrievedObject: ', JSON.parse(retrievedObject));
```
###注意
* `JSON.stringify()`仅对enumerable的属性有效
* 重要的是：想清楚你要保存哪些“数据”（属性），而不是保存“业务逻辑”（函数）
