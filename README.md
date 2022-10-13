# Add_to_Cart
Cart shows the products selected for buyinh.

# NOTE: the names used in this guideline are from a different database and table. change the names where necessary to fit your records. 
## for the records not included in your table, be sure to add them 

# 1. STEP 1: Create an HTML file called Catt.html and paste the following code

```
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
      <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.1.1/dist/css/bootstrap.min.css" rel="stylesheet" integrity="sha384-F3w7mX95PdgyTmZZMECAngseQB83DfGTowi0iMjiWaeVhAn4FJkqJByhZMI3AhiU" crossorigin="anonymous">

</head>
<body>
 <br>
                                    {% with messages = get_flashed_messages(with_categories=true) %}
                                       {% if messages %}
                                          {% for category, message in messages %}
                                             <div class="alert alert-{{category}}" role="alert">
                                                 {{ message }}
                                             </div>
                                          {% endfor %}
                                       {% endif %}
                                    {% endwith %}
                                    <br>
 {% if 'cart_item' in session %}
           <p><a id="btnEmpty" href="{{ url_for('.empty_cart') }}" class="btn btn-danger">Empty Cart</a></p>
            <table class="table table-hover">
                <thead>
                    <tr>
                        <th>Product</th>
                        <th>Quantity</th>
                        <th class="text-center">Unit Price</th>
                        <th class="text-center">Price</th>
                        <th> </th>
                    </tr>
                </thead>
                <tbody>
                {% for key, val in session['cart_item'].items() %}
                 {% set quantity = session['cart_item'][key]['quantity'] %}
                 {% set product_cost = session['cart_item'][key]['product_cost'] %}
                 {% set item_price = session['cart_item'][key]['total_price'] %}
                    <tr>
                        <td class="col-sm-8 col-md-6">
                        <div class="media">
                            <a class="thumbnail pull-left" href="">
                                <img class="media-object" width="150" height="150"
                                     src="../static/img/{{ session['cart_item'][key]['image_url'] }}"
                                     > </a>
                             <div class="text-muted">
                               <i> <h5>
                                        {{ session['cart_item'][key]['product_name'] }}
                                        Brand <span class="text-info">{{ session['cart_item'][key]['product_brand'] }}</span>

                                </h5></i>
                            </div>
                        </div></td>
                        <td class=" col-md-3" style="text-align: center">
                        <input type="email" class="form-control col-sm" value="{{ quantity }}">
                        </td>
                        <td class="col-sm-1 col-md-1 text-center"><strong>KSH &nbsp{{ product_cost }} </strong></td>
                        <td class="col-sm-1 col-md-1 text-center"><strong>KSH &nbsp{{ item_price }} </strong></td>
                        <td class="col-sm-1 col-md-1">
                        <a href="{{ url_for('.delete_product', code=session['cart_item'][key]['product_id']) }}" class="btn btn-danger">
                            <span class="fa fa-trash"></span> Remove
                        </a></td>
                    </tr>
                    {% endfor %}
                    <tr>
                        <td colspan="4"><h5>Total Quantity</h5></td>
                        <td class="text-right"><h5><strong>{{ session['all_total_quantity'] }}</strong></h5></td>
                    </tr>
                    <tr>
                        <td colspan="3"><h4>Total</h4></td>
                        <td colspan="2" class="text-right"><h4><strong>$ {{ session['all_total_price'] }}</strong></h4></td>
                    </tr>
                    <tr>
                        <td colspan="4">
                        <a href="/" class="btn btn-info">
                            <span class="fa fa-shopping-cart"></span> Continue Shopping
                        </a>
                        </td>
                         <td>
                         {% if 'customer_id' in session %}
                               <form action="/proceed_checkout" method="post">
                                   <input type="text" name="mpesa_code" placeholder="Enter Your MPESA Code">
                                   <input type="submit" class="btn btn-info" value="Checkout">
                               </form>
                             <br><br>

                             <a href="/logout">Logout</a>


                         {% else %}
                            <a href="/customer_checkout" class="btn btn-info">
                                 Checkout <span class="fa fa-check"></span>
                            </a>
                         {% endif %}
                        </td>
                    </tr>
                </tbody>
            </table>
            <div class="row">
                <div class="col-md-6">

                </div>

            </div>
          {% else %}
            <br>
           <div class="no-records alert alert-info">Your Cart is Empty, Please continue shopping</div>
            <br>

            <a href="/" class="btn btn-info">
                            <span class="fa fa-shopping-cart"></span> Continue Shopping
            </a>

             <a href="/customer_account" class="btn btn-info">
                            <span class="fa fa-user"></span> My Profile
            </a>
          {% endif %}
        </div>
    </div>

</body>
</html>
```

# 2.STEP 2: inside app.py create a route for adding items to the cart ('/add') and pasete the following code

```
@app.route('/add', methods=['POST'])
def add_product_to_cart():
        _quantity = int(request.form['quantity'])
        _code = request.form['code']
        # validate the received values
        if _quantity and _code and request.method == 'POST':
            cursor = connection.cursor(pymysql.cursors.DictCursor)
            cursor.execute("SELECT * FROM products WHERE product_id= %s", _code)
            row = cursor.fetchone()
            #An array is a collection of items stored at contiguous memory locations. The idea is to store multiple items of the same type together

            itemArray = {row['product_id']: {'product_name': row['product_name'], 'product_id': row['product_id'], 'quantity': _quantity, 'product_cost': row['product_cost'],
                              'image_url': row['image_url'], 'total_price': _quantity * row['product_cost'],
                                             'product_brand': row['product_brand']}}
            print((itemArray))


            all_total_price = 0
            all_total_quantity = 0
            session.modified = True
            #if there is an item already
            if 'cart_item' in session:
                #check if the product you are adding is already there
                if row['product_id'] in session['cart_item']:

                    for key, value in session['cart_item'].items():
                        #check if product is there
                        if row['product_id'] == key:
                            #take the old quantity  which is in session with cart item and key quantity
                            old_quantity = session['cart_item'][key]['quantity']
                            #add it with new quantity to get the total quantity and make it a session
                            total_quantity = old_quantity + _quantity
                            session['cart_item'][key]['quantity'] = total_quantity
                            #now find the new price with the new total quantity and add it to the session
                            session['cart_item'][key]['total_price'] = total_quantity * row['product_cost']

                else:
                    #a new product added in the cart.Merge the previous to have a new cart item with two products
                    session['cart_item'] = array_merge(session['cart_item'], itemArray)
                for key, value in session['cart_item'].items():
                    individual_quantity = int(session['cart_item'][key]['quantity'])
                    individual_price = float(session['cart_item'][key]['total_price'])
                    all_total_quantity = all_total_quantity + individual_quantity
                    all_total_price = all_total_price + individual_price

            else:
                #if the cart is empty you add the whole item array
                session['cart_item'] = itemArray
                all_total_quantity = all_total_quantity + _quantity
                #get total price by multiplyin the cost and the quantity
                all_total_price = all_total_price + _quantity * row['product_cost']


            #add total quantity and total price to a session
            session['all_total_quantity'] = all_total_quantity
            session['all_total_price'] = all_total_price
            return redirect(url_for('.cart'))
        else:
            return 'Error while adding item to cart'
``

# 3. STEP 3: create another route for the cart
```
@app.route('/cart')
def cart():
    return render_template('cart.html')
```    
@app.route('/customer_checkout')
def customer_checkout():
    if check_customer():
            return redirect('/cart')
    else:
        return redirect('/customer_login')
```
# 4. Create another route for empting the cart
```
@app.route('/empty')
def empty_cart():
    try:
        if 'cart_item' in session or 'all_total_quantity' in session or 'all_total_price' in session:
            session.pop('cart_item', None)
            session.pop('all_total_quantity', None)
            session.pop('all_total_price', None)
            return redirect(url_for('.cart'))
        else:
            return redirect(url_for('.cart'))

    except Exception as e:
        print(e)
```
# 5. STEP 5: Create a delete route for individual items in the cart
```
@app.route('/delete/<string:code>')
def delete_product(code):
    try:
        all_total_price = 0
        all_total_quantity = 0
        session.modified = True
        for item in session['cart_item'].items():
            if item[0] == code:
                session['cart_item'].pop(item[0], None)
                if 'cart_item' in session:
                    for key, value in session['cart_item'].items():
                        individual_quantity = int(session['cart_item'][key]['quantity'])
                        individual_price = float(session['cart_item'][key]['total_price'])
                        all_total_quantity = all_total_quantity + individual_quantity
                        all_total_price = all_total_price + individual_price
                break

        if all_total_quantity == 0:
            session.clear()
        else:
            session['all_total_quantity'] = all_total_quantity
            session['all_total_price'] = all_total_price

        # return redirect('/')
        return redirect(url_for('.cart'))
    except Exception as e:
        print(e)
```

#Function for merging arrays
```
def array_merge( first_array , second_array ):
     if isinstance( first_array , list) and isinstance( second_array , list ):
      return first_array + second_array
     #takes the new product add to the existing and merge to have one array with two products
     elif isinstance( first_array , dict ) and isinstance( second_array , dict ):
      return dict( list( first_array.items() ) + list( second_array.items() ) )
     elif isinstance( first_array , set ) and isinstance( second_array , set ):
      return first_array.union( second_array )
     return False

```
