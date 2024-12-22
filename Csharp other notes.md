# Csharp Notes
These are my notes for C#

- [Linq](#linq)
- [LifeTime](#lifetime)
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

# Other

