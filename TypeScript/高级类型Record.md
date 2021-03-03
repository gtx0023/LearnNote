# typescript中高级类型Record

个人理解为 类型映射

ts文档上对Record的介绍不多，但却经常用到，Record是一个很好用的工具类型。
他会将一个类型的所有属性值都映射到另一个类型上并创造一个新的类型，先看下Record的源码。

 * ```typescript
   /**
    * Construct a type with a set of properties K of type T
    */
   type Record<K extends keyof any, T> = {
       [P in K]: T;
   };
   ```

   好像源码也比较简单，即将K中的每个属性([P in K]),都转为T类型。常用的格式如下：

```typescript
type proxyKType = Record<K,T>
```


会将K中的所有属性值都转换为T类型，并将返回的新类型返回给proxyKType，K可以是联合类型、对象、枚举…
看几个demo.
demo1:

```typescript
type petsGroup = 'dog' | 'cat' | 'fish';
interface IPetInfo {
    name:string,
    age:number,
}

type IPets = Record<petsGroup, IPetInfo>;

const animalsInfo:IPets = {
    dog:{
        name:'dogName',
        age:2
    },
    cat:{
        name:'catName',
        age:3
    },
    fish:{
        name:'fishName',
        age:5
    }
}
```



可以看到 IPets 类型是由 Record<petsGroup, IPetInfo>返回的。将petsGroup中的每个值(‘dog’ | ‘cat’ | ‘fish’)都转为 IPetInfo 类型。
当然也可以自己在第一个参数后追加额外的值，看下面demo2

```typescript
type petsGroup = 'dog' | 'cat' | 'fish';
interface IPetInfo {
    name:string,
    age:number,
}

type IPets = Record<petsGroup | 'otherAnamial', IPetInfo>;

const animalsInfo:IPets = {
    dog:{
        name:'dogName',
        age:2
    },
    cat:{
        name:'catName',
        age:3
    },
    fish:{
        name:'fishName',
        age:5
    },
    otherAnamial:{
        name:'otherAnamialName',
        age:10
    }
}
```


可以看到在demo1的基础上，demo2在
type IPets = Record<petsGroup | ‘otherAnamial’, IPetInfo>; 中除了petsGroup的值之外，还追加了 'otherAnamial’这个值。
下面看一个略复杂的例子，用axios将http的几个请求封装一下，使用Record定义每个请求方法的形状。

```typescript
enum IHttpMethods {
    GET = 'get',
    POST = 'post',
    DELETE = 'delete',
    PUT = 'put',
}

const methods = ["get", "post", "delete", "put"];

interface IHttpFn {
    <T = any>(url: string, config?: AxiosRequestConfig): Promise<T>
}

type IHttp = Record<IHttpMethods, IHttpFn>;

const httpMethods: IHttp = methods.reduce((map: any, method: string) => {
    map[method] = (url: string, options: AxiosRequestConfig = {}) => {
        const { data, ...config } = options;
        return (axios as any)[method](url, data, config)
            .then((res: AxiosResponse) => {
                if (res.data.errCode) {
                    //todo somethins
                } else {
                    //todo somethins
                }
            });
    }
    return map
}, {})

export default httpMethods;
```

上面这个demo就先枚举除了几个常见的http请求的方法名，而每个方法都接受请求的url以及可选参数config,然后每个方法返回的都是一个Promise。这种业务常见使用Record再合适不过了。使用下面的方式定义了每个方法的形状。

```typescript
type IHttp = Record<IHttpMethods, IHttpFn>;
```


最后只需要遍历一下几个方法，对每个方法有各自的具体实现即可。这里是用了reduce的特性，遍历了一下数据，然后将所有的方法体放在一个对象中，最终结果用 httpMethods接受，再将httpMethods对外暴露出去，那么外面就可直接调用了。这里把一些业务的部分抽离出去了(比如设置请求头、设置token之类的)，只是为了简单说明一个比较合适使用Record的业务场景。



