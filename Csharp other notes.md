# Csharp Notes
These are my notes for C#

- [Linq](#linq)
- [LifeTime](#lifetime)
- [Parameter with HTTPGet and Post](#parameter-with-get-and-post)
- [classname new classname()](#classname-new-classname())
- [AsNoTracking before Select](#asnotracking-before-select)
- [Return Same Type](#return-same-type)
- [Other](#other)


# LINQ

### AsNoTracking()

**code example**
``` csharp
 var product = _context.Products.Where(p => p.Id == model.Id).AsNoTracking().FirstOrDefault();
```

AsNoTracking means you turn off the connection to database from the layer(normally service layer) that code is written in.

The main advantage is that it increases performance.

But when it involves saving changes to the database like **Delete or Update to the database** you should not add **AsNoTracking** because

**code example:**
``` csharp
 public UpdateProductByIdResponseModel UpdateProductById(UpdateProductByIdRequestModel model)
{
    var product = _context.Products.Where(p=> p.Id ==model.Id).AsNoTracking().FirstOrDefault();
    product.ItemName = model.ItemName;
    product.Quantity = model.Quantity;
    product.Price = model.Price;
    _context.Products.Update(product);
    _context.SaveChanges();

    UpdateProductByIdResponseModel updateResponse = new UpdateProductByIdResponseModel();
    return updateResponse;
}
```

what this does is **AsNoTracking** closes the database and then **_context.Product.Update(product)** open the database
again which leads to decrease the performance.

Same for the Delete in CRUD.

So, it is best to use with getting(reading) data from the database.


### FirstOrDefault() and ToList()

They are used when using linq with **_context**.

**code example:**
``` csharp
public GetProductByIdResponseModel GetProductById(GetProductByIdRequestModel model)
{
   
    var product = _context.Products.Where(p => p.Id == model.Id).AsNoTracking().FirstOrDefault();

    GetProductByIdResponseModel result = new GetProductByIdResponseModel()
    {
        Id = product.Id,
        ItemName = product.ItemName,
        Quantity = product.Quantity,
        Price = product.Price,

    };
    return result;
}

public GetProductListResponseModel GetProductList(GetProductListRequestModel model)
{
    var products = new GetProductListResponseModel();
    products.Products = _context.Products.AsNoTracking().ToList();

    return products;
}
```

**FirstOrDefault()** is used when only one row from the database is required <br/>
while **ToList()** is used when we need to get the responses list(multiple rows) from the database.

# LifeTime

In ASP.NET Core, the *AddTransient*, *AddScoped*, and *AddSingleton* methods define the lifetime of a service in the Dependency Injection (DI) container. <br/>
The key difference is how long the service instance lives and how often it is created or reused.

# Parameter with Get and Post

In controller when we use HttpGet method, the parameter for the controller cannot be model while for HttpPost parameter can be model

# Classname new classname()

``` csharp
GetAuthorResponseModel result = new GetAuthorResponseModel();
result.Id = "123";
result.Name = "J.K. Rowling";

Console.WriteLine($"Author ID: {result.Id}, Author Name: {result.Name}");
```
or 
``` csharp
GetAuthorResponseModel result = new GetAuthorResponseModel()
{
    Id = model.Id,
    Name = model.Name,
    RealName = model.RealName,
    Bio = model.Bio
};
```
can be write like this and 

The purpose of the code is to create an instance of the **GetAuthorResponseModel** class

# AsNoTracking before Select 

### How `.AsNoTracking()` and `.Select()` Work Together

- **`.AsNoTracking()`**: Ensures that EF does not track the entities involved in the query. It applies globally to all subsequent parts of the query.
- **`.Select()`**: Projects the data into a new shape (e.g., a custom model like `BookModel`) or extracts specific columns.

When you project data into a new object in `.Select()` (like your `BookModel`), EF doesn’t need to track anything because the result of the query is no longer an EF entity. However, **`.AsNoTracking()` is still helpful to avoid the overhead of tracking intermediate entities** (e.g., `BookDbSet`, `AuthorDbSet`, `CategoryDbSet`) during the query.

---

### Should You Use `.AsNoTracking()` Before `.Select()`?
**Yes**, it's a good practice to apply `.AsNoTracking()` **before** `.Select()` when:
1. The query involves entities (`_context.BookDbSet`, `_context.AuthorDbSet`, etc.).
2. You are projecting the result into custom objects like `BookModel`, not modifying entities.
3. The operation is read-only.

Example:
```csharp
var booklist = _context.BookDbSet
    .AsNoTracking() // Applies to the whole query
    .Join(...) // Join logic
    .Where(...) // Filtering logic
    .Select(book => new BookModel // Projection
    {
        Id = book.book.Id,
        Name = book.book.Name,
        AuthorId = book.author.Id,
        AuthorName = book.author.Name,
        CategoryId = book.category.Id,
        CategoryName = book.category.Name,
        PublishedYear = book.book.PublishedYear,
        TotalQuantity = book.book.TotalQuantity,
        AvailableQuantity = book.book.AvailableQuantity
    })
    .ToList();
```

### After `.ToList()`
Once you call `.ToList()`, the query is executed, and the result is loaded into memory. At this point:
- The query stops being a deferred LINQ query.
- All entities or objects (like `BookModel`) are materialized into a list.
- If you’ve applied `.AsNoTracking()` before `.ToList()`, no tracking occurs, and the result is purely read-only.

# Return Same Type

The return type of a method must match the declared return type of the method because it ensures type safety and consistency in the code. 

### Correct 
```csharp
public GetBookByIdResponseModel GetBookById(GetBookByIdRequestModel model)
{
    var book = _context.BookDbSet.Where(a => a.Id == model.Id).AsNoTracking().FirstOrDefault();

    BookModel bookmodel = new BookModel()
    {
        Id = book.Id,
        Name = book.Name,
        PublishedYear = book.PublishedYear,
        TotalQuantity = book.TotalQuantity,
        AvailableQuantity = book.AvailableQuantity
    };

    GetBookByIdResponseModel result = new GetBookByIdResponseModel()
    {
        BookRes = bookmodel
    };

    return result;

    //return new GetBookByIdResponseModel
    //{
    //    Book = result
    //};
}
```

### Wrong 

As you see here, `BookModel` is not the same type with `GetBookByIdResponseModel` and IDE will show you the error that `result` is not null.

```csharp
public GetBookByIdResponseModel GetBookById(GetBookByIdRequestModel model)
{
    var book = _context.BookDbSet.Where(a => a.Id == model.Id).AsNoTracking().FirstOrDefault();

    BookModel result = new BookModel()
    {
        Id = book.Id,
        Name = book.Name,
        PublishedYear = book.PublishedYear,
        TotalQuantity = book.TotalQuantity,
        AvailableQuantity = book.AvailableQuantity
    };

    return result;
}
```

# Other

