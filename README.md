# Idiomatic Go 

If you learned some Go and want to learn how to write idiomatic Go code then my suggestion would be read the all the Go proverbs available here [Go Proverb](https://go-proverbs.github.io/). Each of them has deep philosophy which portrays Go's behavior and best practices.  Along the way of your learning spend sometime with each of them and get the idea what of actually each of them means. Most of the Go-proverbs are intiatiated by Rob Pike and he also gave talk on most of them. So watching/listening them would help a lot to learn deeply about Go.   

In this writing, I'm trying to made some bullet-points with little details, which are some practices that I follow and some of them are widely adopted/recommended by the Go community.  

* [Error Handling](#Error-Handling)
* [Project Structure](#Error-Handling)
* [HTTP Router](#Router)
* [Design Pattern](#Design-pattern-/-Best-practices)
* [ORM/Framework or Not](#No-ORM-or-Frameworks-is-a-better-choice)
* [Interface](#Interface)

**Errors are just values**  - Go Proverb

## Error Handling

A great talk by **Dave Cheney** on error handling, very consize and practical: 
* [GopherCon 2016: Dave Cheney - Dont Just Check Errors Handle Them Gracefully](https://www.youtube.com/watch?v=lsBF58Q-DnY) 

i.e. When returning error from a function and needs to add cintext to the error, use Wrap function like this: 

```go 
   if err != nil {
       return errors.Wrap(err, "Failed fetching data from users table")
   }
```

Don't inspect the error with  `error.Error()`, to check the error type. It's for the human, not for the code. Use  custom error type for checking error type. For example, when reading data from the sql database if it returns an error, we don't know error type but if we want to check if it's the error `not found rows`  or something we can check it like this: Design pattern / Best practices

In that avobe example ErrNoRows is a custom error data type defined by sql package which is to check if the error is: `no rows in result set`.  

Another good example for checking duplicate entry/ breaking unique key constraint could be something like this:

```go 
	if err != nil {
		if pgerr, ok := errors.Cause(err).(*pq.Error); ok {
			if pgerr.Code == "23505" {
	           
			}
		}

	   //handle error here
	}

```

More details on the customer error type wrapping and unwrapping could be found in this golang official blog:
* [Working with Errors in Go 1.13](https://blog.golang.org/go1.13-errors)


## Project Structure 

At the time I started learning or working with `Go`, the big issue was probably dependency management and  also Generics. The first one is solved and `go mod` works very smoothly for managing/vendoring dependency. They second one is on the processing of implementing.     

But there is a thrid one, which is Go project structure. Which could be confusing for those are just newcomer in `Go`. 

In the beginning I used global variable to store database instance, you know that's a bad practice, right? Even though it could just work fine!  

Later storing it into the struct was the solution, **Mat Ryer** gave/wrote a nice talk on this topic called 
* [How I write Go HTTP services after seven years](https://medium.com/@matryer/how-i-write-go-http-services-after-seven-years-37c208122831)  

All though my journy with Go like 3 year or so, I found quite similar pattern while making HTTP services. This approach is very clean and easy to implement. It works well for plain HTTP services.      

But if it's about project structure then this one is probably most mentioned and recommended layout for creating big Go project:
* [Standard Package Layout](https://medium.com/@benbjohnson/standard-package-layout-7cdbc8391fc1) written by **Ben Johnson**. Which is widely adopted in the gophers community. And this is the one I tend to follow these days.

## Router 

Find a router that is compatible with http handler method from the `net/http` package. 

At the very beginning I used Gin (not as a framework though) just for routing and middleware. But Gin have some bugs and some performance issues. 

Later used 
* [gorilla/mux](https://github.com/gorilla/mux), but it's a lot slower than a few other ones those came later like fasthttp, httprouter and that because of it uses regex handling pattern matching. whether other latest HTTP routers for Go are made by following Trie/Radix tree based data strcuture.

I have made a one called 
* [Track](https://github.com/monirz/track) using Trie data structure as well, it's pretty fast and light weight but it needs some heavy testing. Was working on the benchmarking and stuff, got busy later,  won't recommend to use it for production. 

For production, I use 
 * [Chi](https://github.com/go-chi/chi) for http routing, it's 100% compatible with net/http, fast and also uses less memory than other ones. I tested and ran the becnhmark personally. They also added the benchmark log in their doc. 

Final thought, I'll quote a comment from this 
 * [thread](https://www.reddit.com/r/golang/comments/a3qcid/httprouter_chi_gin_gorillamux/):
> router performance is less likely to be your bottleneck, but I still think if you can have something that's really fast, and has a great feature set (Chi), then why choose to use something slower with similar features?     


## Design pattern / Best practices

 People come from other languages to Go, they try write the way code they used write code in their favorite language. When you are learning a new langauge it's just not the syntax, it's the whole echo-system and best practices of that language. 
 
 * **[When in Go, do as the Gopher do](https://talks.golang.org/2014/readability.slide#1)**  

 Go is not a OOP language, people tend to do that a lot like implemnting pure OOP pattern whose are come from OOP language. Go is a C like language with the modern features and easier to writing code. Yet you can adapt some good parts/feature from OOP in Go.       

In idiomatic Go your interface{} should be small like they way it is used in the standard packages. So if you want to loosely couple your code for future modification or extendings by following some design pattern like clean-architecture, do it in the way that it looks like Go. Avoid writing the code same way you used to write in a pure OOP langauge. 

As someone I was talking to earlier, where he said clean-architecture `-- it's just two API level, One to actually do everything and one to hook it up to the outside world(http, gRPC,websockets, you could also maybe turn it into a CLI easily or any other protocol if you need). There is a third level below the business logic to handle things like talking to shipping companies, xlsx export, etc. People tend to not think about things once they find a term and become diehard fans using it everywhere even if it makes 0 sense or brings little benefit. Use it where you gain something.` Couldn't agree more. 
        

This is an another great talk by **Dave Cheney** on implementing SOLID principle in Go : 
* [Golang UK Conference 2016 - Dave Cheney - SOLID Go Design](https://www.youtube.com/watch?v=zzAdEt3xZ1M)  

* [Postel’s Law](https://en.wikipedia.org/wiki/Robustness_principle)
**"Be conservative with what you do, be liberal with you accept"**

In Go that becomes:

>**A great rule of thumb for Go is accept interfaces, return structs.**
                                                         –Jack Lindamood 

Design your function that way that it accepts interfaces and returns structs. It's like classic Go pattern. You will see that a lot standard or in third party libraries.    


## No ORM or Frameworks is a better choice 

The Go community discourage to not to use a Web framework mostly for Go.  

Since in Go the concept of Object is not completely present why do you try to use it to Object Relational Mapping. Where you'll also lose some control/freedom, most of the ORM's are slow as well (so far).   

SQL package is pretty much all you need for doing SQL stuff, I use 
* [sqlx](https://github.com/jmoiron/sqlx) package sometimes.

 It's just wrapper of SQL package, it has all the feature that `SQL` package has. By using it you can avoid manually scannings all the values from SQL to your data types.  \


## Interface

If you are confused about interface when to use it, you probably shouldn't use interface. You will know it when you need to use interfaces. 

A good article by **William Kennedy** called 
* [Avoid Interface Pollution](https://www.ardanlabs.com/blog/2016/10/avoid-interface-pollution.html) 

This is also another good artcile on how/when to use interface 
* [How To Use Go Interfaces](https://blog.chewxy.com/2018/03/18/golang-interfaces/) 

## Writing Unit Test 
A Good Go(Gopher) developer loves to write tests and don't want to avoid it. 
Go has a good echo-system for writing test. Thus it encourage you write unit test for most of your function. I think Go developers are currently way ahead of other developers if it's about how much test coverage they have. 
If you don't do TDD, write the test at least when you are testing your function, if that makes sense.


                                                                                                                   