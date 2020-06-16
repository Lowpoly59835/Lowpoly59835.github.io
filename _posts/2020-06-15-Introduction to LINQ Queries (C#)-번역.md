---
layout: post
title:  "Introduction to LINQ Queries (C#) (1) 번역"
author: "로우폴리"
comments: false
---

단순히 영어에 익숙해져야겠다 싶어서 내가 공부했던 것들중에서 기본적인 사항들을 정리해놓은 글들을 번역하는 중임
이번 글은 MS Docs에 있는 [Linq 쿼리 소개](https://docs.microsoft.com/ko-kr/dotnet/csharp/programming-guide/concepts/linq/introduction-to-linq-queries)를 번역해보았음
정보 전달은 링크에 번역이 되어있으니 거길 보는걸 추천함

**번역이라 이름붙은 글은 다 번역 연습임을 분명히 알림.**

-----------------------------------------

# Introduction to LINQ Queries (C#)

A query is an expression that retrieves data from a data source.

쿼리는 데이터 소스를 검색하는 표현식이다.

Queries are usually expressed in a specialized query language.  
쿼리들은 흔히 특별한 쿼리 언어로 표현된다. 

Different languages have been developed over time for the various types of data sources, for example SQL for relational databases and XQuery for XML.  
다른 언어들은 데이터베이스의 sql과 XML의 Xquery같은 데이터 소스의 다양한 타입으로 개발되었다.

Therefore, developers have had to learn a new query language for each type of data source or data format that they must support.   
그러므로, 개발자들은 그것들을 지원하는 데이터 포맷 혹은 데이터 소스의 각 타입을 위한 새로운 언어를 새로 배워야했다.

LINQ simplifies this situation by offering a consistent model for working with data across various kinds of data sources and formats.  
LINQ는 데이터 소스와 포맷의 다양한 종류를 동작시키기 위한 일관된 모델을 제공하여 이 상황을 단순화했다.

In a LINQ query, you are always working with objects.  
LINQ 쿼리에서, 당신은 항상 오브젝트와 함께한다.

You use the same basic coding patterns to query and transform data in XML documents, SQL databases, ADO.NET Datasets, .NET collections, and any other format for which a LINQ provider is available.

당신은 같은 쿼리 코딩 패턴을 사용함으로서 XML 문서, SQL 데이터 베이스, ADO.NET 데이터 베이스, .NET 콜렉션들, 그리고 LINQ 제공자가 할 수 있는 다른 포맷들을 데이터를 변환한다.

## Three Parts of a Query Operation (쿼리 명령의 세 파트)

All LINQ query operations consist of three distinct actions:  

모든 LINQ 쿼리 명령은 세가지 다른 액션으로 구성된다  

Obtain the data source.

데이터 소스 획득

Create the query.

쿼리 생성

Execute the query.

쿼리 실행

The following example shows how the three parts of a query operation are expressed in source code. The example uses an integer array as a data source for convenience; however, the same concepts apply to other data sources also. This example is referred to throughout the rest of this topic.

다음 예제는 어떻게 쿼리 명령의 세 파트가 소스 코드에서 표현되는지 보여준다. 예제는 간단한 정수 배열을 사용하지만, 같은 컨셉은 다른 데이터 소스 또한 적용된다. 이 예제는 이 토픽의 나머지를 통해서 참조된다.(?)


    class IntroToLINQ
    {
        static void Main()
        {
            // The Three Parts of a LINQ Query:
            // 1. Data source.
            int[] numbers = new int[7] { 0, 1, 2, 3, 4, 5, 6 };

            // 2. Query creation.
            // numQuery is an IEnumerable<int>
            var numQuery =
                from num in numbers
                where (num % 2) == 0
                select num;

            // 3. Query execution.
            foreach (int num in numQuery)
            {
                Console.Write("{0,1} ", num);
            }
        }
    }

The following illustration shows the complete query operation. In LINQ, the execution of the query is distinct from the query itself. In other words, you have not retrieved any data just by creating a query variable.  

다음 그림은 쿼리 명령의 수행을 보여준다. LINQ에서, 쿼리의 실행은 쿼리 자체와 다르다. 다른 말로, 당신은 쿼리 변수를 생성하기만으로는 어떠한 데이터도 검색할 수 없다.  


![쿼리수행](https://docs.microsoft.com/ko-kr/dotnet/csharp/programming-guide/concepts/linq/media/introduction-to-linq-queries/linq-query-complete-operation.png)

## The Data Source (데이터 소스)

In the previous example, because the data source is an array, it implicitly supports the generic IEnumerable<T> interface. This fact means it can be queried with LINQ. A query is executed in a foreach statement, and foreach requires IEnumerable or IEnumerable<T>. Types that support IEnumerable<T> or a derived interface such as the generic IQueryable<T> are called queryable types.

이전 예제에서, 데이터 소스가 배열이기 떄문에, 일반적인 IEnumerable<T>를 명시적으로 사용했다. 이것은 LINQ에 IEnumberable이 요구된다는 것을 의미한다. 쿼리는 foreach 명령에서 실행되고, foreach는 IEnumerable 혹은 IEnumerable<T>를 요구한다. IEnumerable<T> 혹은 인터페이스를 상속받은 각각 일반적인 IQueryable<T>같은 타입은 queryable한 타입이라 불려진다.

A queryable type requires no modification or special treatment to serve as a LINQ data source. If the source data is not already in memory as a queryable type, the LINQ provider must represent it as such. For example, LINQ to XML loads an XML document into a queryable XElement type:  

queryable한 타입은  수정불가능하거나 LINQ 데이터 소스같은 serve한 특별한 teratment를 요구한다. 만약 소스 데이터가 queryable한 타입의 메모리안에 준비되지 않다면, LINQ 제공자는 반드시 중단한다. 예를들어, LINQ to XML는 queryable한 XElement형식으로 XML 문서를 로드한다.

    // Create a data source from an XML document.
    // using System.Xml.Linq;
    XElement contacts = XElement.Load(@"c:\myContactList.xml");

With LINQ to SQL, you first create an object-relational mapping at design time either manually or by using the LINQ to SQL Tools in Visual Studio. You write your queries against the objects, and at run-time LINQ to SQL handles the communication with the database. In the following example, Customers represents a specific table in the database, and the type of the query result, IQueryable<T>, derives from IEnumerable<T>.    
    
LINQ to SQL을 사용하여 당신은 먼저 수동 혹은 비쥬얼 스튜디오에서 LINQ to SQL Tools를 사용하여 디자인 타임에서 객체 관계 매핑을 만든다. 당신은 개체에대해 쿼리를 작성하고, 런타임에 LINQ to SQL 핸들이 데이터베이스와 대화한다. 다음 예제에서, Customers는 데이터베이스에서 특정 테이블과 쿼리 결과의 IQueryable<T>, IEnumerable<T>를 상속받은 타입들을 나타낸다.
