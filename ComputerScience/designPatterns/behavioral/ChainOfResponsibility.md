### 责任链模式  
避免请求发送者与接收者耦合在一起，让多个对象都有可能接收请求，将这些对象连接成一条链，并且沿着这条链传递请求，直到有对象处理它为止。  
职责链模式是一种对象行为型模式。  
 一个纯的职责链模式要求一个具体处理者对象只能在两个行为中选择一个：要么承担全部责任，要么将责任推给下家；  
 
 OkHttpClient  
 ```
public interface Interceptor {

    Request intercept(Chain chain);

    interface Chain {
        Request request();

        Request proceed(Request request);

    }
}

public class MyInterceptor implements Interceptor{
    public Request intercept(Chain chain) {
         return chain.proceed(chain.request());
    }
}

 ```
参考  
http://blog.csdn.net/qq_25827845/article/details/51959801  
