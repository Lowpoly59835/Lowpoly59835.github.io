---
layout: post
title:  "Introduction to LINQ Queries (C#) (2) 번역"
author: "로우폴리"
comments: false
---

단순히 영어에 익숙해져야겠다 싶어서 아무 생각없이 아무 글이나 번역하는 중임  
이번 글은 MS Docs에 있는 [Linq 쿼리 소개](https://docs.microsoft.com/ko-kr/dotnet/csharp/programming-guide/concepts/linq/introduction-to-linq-queries)를 번역해보았음  
정보 전달은 링크에 번역이 되어있으니 거길 보는걸 추천함  

-----------------------------------------

# Introduction to LINQ Queries (C#) (2)

LINQ to SQL을 사용하여 당신은 먼저 수동 혹은 비쥬얼 스튜디오에서 LINQ to SQL Tools를 사용하여 디자인 타임에서 객체 관계 매핑을 만든다. 당신은 개체에대해 쿼리를 작성하고, 런타임에 LINQ to SQL 핸들이 데이터베이스와 대화한다. 다음 예제에서, Customers는 데이터베이스에서 특정 테이블과 쿼리 결과의 IQueryable<T>, IEnumerable<T>를 상속받은 타입들을 나타낸다.  

    Northwnd db = new Northwnd(@"c:\northwnd.mdf");  
    
    // Query for customers in London.  
    IQueryable<Customer> custQuery =  
        from cust in db.Customers  
        where cust.City == "London"  
        select cust; 

For more information about how to create specific types of data sources, see the documentation for the various LINQ providers. However, the basic rule is very simple: a LINQ data source is any object that supports the generic IEnumerable<T> interface, or an interface that inherits from it.

어떻게 데이터 소스의 특정 타입을 만드는지에 대한 더 자세한 정보를 알고싶다면, LINQ 제공자 변수를 위한 문서를 봐라. 하지만, 기본 룰은 매우 간단하다 : LINQ 데이터 소스는 제네릭 IEnumerable<T> 인터페이스를 돕는 어떠한 객체이거나 이것을 상속받는다.

## The Query (쿼리)

The query specifies what information to retrieve from the data source or sources. Optionally, a query also specifies how that information should be sorted, grouped, and shaped before it is returned. A query is stored in a query variable and initialized with a query expression. To make it easier to write queries, C# has introduced new query syntax.  

쿼리는 데이터 소스 혹은 소스로부터 검색한 정보를 지정한다. 선택적으로, 쿼리는 또한 리턴되기전에 어떻게 정보가 저장되고, 그룹짓고, 공유되는지 지정한다. 쿼리는 쿼리 변수에 저장되고 쿼리 표현식과 함께 초기화된다. 더 쉽게 쿼리를 작성하기위해서, C#은 새로운 쿼리 문맥을 소개한다.  


The query in the previous example returns all the even numbers from the integer array. The query expression contains three clauses: from, where and select. (If you are familiar with SQL, you will have noticed that the ordering of the clauses is reversed from the order in SQL.) The from clause specifies the data source, the where clause applies the filter, and the select clause specifies the type of the returned elements. These and the other query clauses are discussed in detail in the Language Integrated Query (LINQ) section. For now, the important point is that in LINQ, the query variable itself takes no action and returns no data. It just stores the information that is required to produce the results when the query is executed at some later point. For more information about how queries are constructed behind the scenes, see Standard Query Operators Overview (C#).

이전 예제에서 쿼리는 정수 배열로부터 모든 짝수를 배열한다. 쿼리 표현식은 세 개의 계산을 포함한다. from, where 그리고 select. 
(만약 당신이 SQL과 익숙하다면, 당신은 명령의 순서가 SQL과 반대된다는 것에 주목해야할 것이다.) from 순서가 데이터 소스를 특징하고, where 명령가 필터를 어플라이하고, select가 리턴될 항목의 타입을 특징한다. 이것들과 다른 쿼리 명령들은 언어가 쿼리(LINQ) 세션을 통합하는 디테일을 야기한다. 현재 LINQ에서 중요한 포인트는 쿼리 변수 자체로는 액션이 없고, 데이터를 리턴하지 않는다. 쿼리는 어떤 시점에서 실행될 때에 결과를 생산하기위해 요구되는 정보만을 저장한다. 어떻게 쿼리가 뒷면(scene?)에서 구성되는지에대한 정보를 알고 싶다면, [표준 쿼리 명령 오버뷰](https://docs.microsoft.com/ko-kr/dotnet/csharp/programming-guide/concepts/linq/standard-query-operators-overview)를 보라.

 참고

Queries can also be expressed by using method syntax. For more information, see Query Syntax and Method Syntax in LINQ.  

쿼리가 또한 메소드 문맥을 사용함으로서 표현된다. 더 많은 정보를 알고싶다면, [LINQ에서 쿼리 문맥과 함수 문맥](https://docs.microsoft.com/ko-kr/dotnet/csharp/programming-guide/concepts/linq/query-syntax-and-method-syntax-in-linq)을 보라

## Query Execution (쿼리 실행)

### Deferred Execution (연기된 실행)

As stated previously, the query variable itself only stores the query commands. The actual execution of the query is deferred until you iterate over the query variable in a foreach statement. This concept is referred to as deferred execution and is demonstrated in the following example:

시작하기이전에, 쿼리 변수 자체로는 쿼리 명령을 저장한다. 쿼리의 실제 실행은 당신이 foreach 명령에서 쿼리 변수를 참조할 때까지 연기된다. 연기된 실행이 참조되는고 이것은 다음 예제로 증명된다.

    //  Query execution.
    foreach (int num in numQuery)
    {
        Console.Write("{0,1} ", num);
    }


The foreach statement is also where the query results are retrieved. For example, in the previous query, the iteration variable num holds each value (one at a time) in the returned sequence.

foreach 명령은 쿼리 결과에서 또한 검색된다. 예를들어, 이전 쿼리에서, iteration 변수는 리턴된 시퀀스에서 각 값이 고정된다. 

Because the query variable itself never holds the query results, you can execute it as often as you like. For example, you may have a database that is being updated continually by a separate application. In your application, you could create one query that retrieves the latest data, and you could execute it repeatedly at some interval to retrieve different results every time.

쿼리 변수 자체는 절대 쿼리 결과를 고정하지 못하기 때문에, 당신은 종종 이렇게 실행해야한다. 예를들어, 당신은 아마도 분할한 어플리케이션에 의해 연속적으로 업데이트되는 데이터베이스를 가진다. 당신의 어플리케이션에서 가장 최근 데이터를 검색하는 하나의 쿼리를 만들고, 당신은 조금의 간격을 두고 되풀이하며 매번 다른 결과를 검색한다.

## Forcing Immediate Execution (즉시 실행을 주목하라)

Queries that perform aggregation functions over a range of source elements must first iterate over those elements. Examples of such queries are Count, Max, Average, and First. These execute without an explicit foreach statement because the query itself must use foreach in order to return a result. Note also that these types of queries return a single value, not an IEnumerable collection. The following query returns a count of the even numbers in the source array:

소스 항목의 범위에대해 집계 함수를 수행하는 쿼리는 각 항목에대해 먼저 반복해야한다. 각 예제의 각 쿼리들은 Count, Max, Average와 First다. 쿼리 자체는 결과를 리턴하기위해 foreach를 사용해야하기 때문에 명시적 foreach 명령 없이 실행한다. 그런 쿼리들의 타입들이 IEnumerable 콜렉션이 아닌 하나의 값을 리턴하는데 주목하라. 소스 배열에서 짝수의 갯수를 반환하는 다음 예제를 보라.

    var evenNumQuery =
        from num in numbers
        where (num % 2) == 0
        select num;

    int evenNumCount = evenNumQuery.Count();

To force immediate execution of any query and cache its results, you can call the ToList or ToArray methods.  

쿼리의 즉시 실행하고 결과를 캐시하기위해, 당신은 ToList 혹은 ToArray를 호출한다.

    List<int> numQuery2 =
        (from num in numbers
        where (num % 2) == 0
        select num).ToList();

    // or like this:
    // numQuery3 is still an int[]

    var numQuery3 =
        (from num in numbers
        where (num % 2) == 0
        select num).ToArray();

You can also force execution by putting the foreach loop immediately after the query expression. However, by calling ToList or ToArray you also cache all the data in a single collection object.

당신은 쿼리 표현식 이후에 바로 foreach 반복을 실행할 수 있다. 하지만, ToList 혹은 ToArray를 호출하여 당신은 하나의 콜렉션 객체에 모든 데이터를 캐시했다.


------------------------------------

다음글은 이전의 Lambda를 디스어셈블리해서 확인한 것처럼 LINQ 또한 디스어셈블로 확인하는 글을 쓰고 마치고자한다.
