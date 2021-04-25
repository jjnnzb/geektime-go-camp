1.我们在数据库操作的时候，比如 dao 层中当遇到一个 sql.ErrNoRows 的时候，是否应该 Wrap 这个 error，抛给上层。为什么，应该怎么做请写出代码？

答：我认为应该Wrap此error，抛给上层。 dao层操作数据库，应该如实将执行情况报告给调用层（一般是service层），service层则负责统一的处理逻辑，例如参数校验、降级处理等都统一在 service层完成，这样的模式我认为是比较合理的。降级处理等，要视业务需求，由service层统一来做。有的时候，查不到数据会降级返回一个默认值， 而有的时候，没有数据就返回空，不报错；另外一些时候，则需要提示用户未曾提交相关的数据等。 

示例代码：

- dao层

```go
 return errors.Wrapf(business_code.NO_DATA, fmt.Sprintf("Sql is %s, fetched no data from database, %+v", err))
```



- service层

```go
   data, err = get_data()
   if errors.Is(err, business_code.NO_DATA) {
      	// 降级处理
   }
```

