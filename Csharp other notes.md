# Csharp Notes
These are my notes for C#

- [Linq](#linq) and
- [Other](#other)


# LINQ

### AsNoTracking()

**code example**
``` csharp
 var product = _context.Products.Where(p => p.Id == model.Id).AsNoTracking().FirstOrDefault();
```

AsNoTracking means you turn off the connection to database from the layer(normally service layer) that code is written in

the main advantage is that it increases performance.

But when it involves saving changes to the database like **Delete or Update to the database** you should not add AsNoTracking because

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

So, it is best to use with getting (reading) data from the database.

### FirstOrDefault() and ToList()

There are used when using linq with **_context** 

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

**FirstOrDefault()** is used when only one row from the database is required 
while **ToList()** is used when we need to get the responses list(multiple rows) from the database.


# Other

