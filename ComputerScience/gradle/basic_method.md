遍历 task  
```
project.tasks.each {
    logV it.name
}
```

```
task invokeMethod << {
    method1(1,2)
    method1 1,2
}

def method1(int a,int b){
    println a+b
}

task printMethodReturn << {
    def add1 = method2 1,2
    def add2 = method2 5,3
    println "add1:${add1},add2:${add2}"
}

def method2(int a,int b){
    if(a>b){
        a
    }else{
        b
    }
}
```

```
task helloClosure << {
    //使用我们自定义的闭包
    customEach {
        println it
    }

    //多个参数
    eachMap {k,v ->
        println "${k} is ${v}"
    }
}

def customEach(closure){
    //模拟一个有10个元素的集合，开始迭代
    for(int i in 1..10){
        closure(i)
    }
}

def eachMap(closure){
    def map1 = ["name":"张三","age":18]
    map1.each {
        closure(it.key,it.value)
    }
}
task helloDelegate << {
    new Delegate().test {
        println "thisObject:${thisObject.getClass()}"
        println "owner:${owner.getClass()}"
        println "delegate:${delegate.getClass()}"
        method1()
        it.method1()
    }
}

def method1(){
    println "Context this:${this.getClass()} in root"
    println "method1 in root"
}
class Delegate {
    def method1(){
        println "Delegate this:${this.getClass()} in Delegate"
        println "method1 in Delegate"
    }

    def test(Closure<Delegate> closure){
        closure(this)
    }
}

task configClosure << {
    person {
        personName = "张三"
        personAge = 20
        dumpPerson()
    }
}

class Person {
    String personName
    int personAge

    def dumpPerson(){
        println "name is ${personName},age is ${personAge}"
    }
}

def person(Closure<Person> closure){
    Person p = new Person();
    closure.delegate = p
    //委托模式优先
    closure.setResolveStrategy(Closure.DELEGATE_FIRST);
    closure(p)
}


```